# 06 — Entity Relationship Diagram (ERD)

[← Kembali ke daftar isi](./README.md)

---

## 1. Konvensi Database

| Konvensi | Aturan |
|---|---|
| Primary key | `id UUID` (UUIDv7, terurut waktu); dibuat aplikasi/perangkat agar aman untuk offline |
| Tenant | Semua tabel milik tenant memiliki `tenant_id UUID NOT NULL` + **RLS policy** `tenant_id = current_setting('app.tenant_id')::uuid`; indeks diawali `tenant_id` |
| Tabel global | `plans`, `permissions`, `users`, `member_logins`, `coa_templates`, `rule_packs`, `report_templates` (versi platform) — tanpa RLS tenant, akses dibatasi peran platform |
| Uang | `NUMERIC(20,2)`; tarif/persen `NUMERIC(9,6)`; qty `NUMERIC(18,4)` |
| Waktu | `TIMESTAMPTZ` (UTC) untuk waktu kejadian; `DATE` untuk tanggal akuntansi (`entry_date`, `value_date`) |
| Audit kolom | `created_at`, `created_by`, `updated_at`, `updated_by`, `version` (optimistic locking) |
| Hapus | *Soft delete* (`deleted_at`) hanya untuk data master. **Transaksi keuangan tidak pernah dihapus**; dibatalkan dengan pembalik (`reversed_by_id`) |
| Penomoran dokumen | Tabel `document_sequences` per tenant, jenis dokumen, dan periode; nomor perangkat POS memakai prefiks perangkat |
| Enum | Disimpan sebagai `TEXT` + `CHECK` constraint (mudah migrasi) |
| JSONB | Hanya untuk data semi-terstruktur: definisi rule, snapshot, custom field, payload event |
| Partisi | `journal_lines` & `rule_executions` per rentang tanggal (tahunan/bulanan); `audit_logs` per bulan |
| Data sensitif | `nik_enc`, `kk_no_enc`, `bank_account_enc` terenkripsi di aplikasi (AES-256-GCM, kunci KMS per tenant) + `nik_hash` (HMAC) untuk pencarian & keunikan |

Diagram di bawah menampilkan **kolom kunci** saja; kolom lengkap untuk tabel inti ada di §11 Kamus Data.

---

## 2. Platform, Tenant & IAM

```mermaid
erDiagram
    PLANS ||--o{ PLAN_FEATURES : "berisi"
    PLANS ||--o{ TENANT_SUBSCRIPTIONS : "dipakai"
    TENANTS ||--o{ TENANT_SUBSCRIPTIONS : "berlangganan"
    TENANTS ||--o{ TENANT_FEATURE_OVERRIDES : "mengatur"
    TENANTS ||--o{ TENANT_USERS : "memiliki tim"
    USERS ||--o{ TENANT_USERS : "bergabung di"
    TENANTS ||--o{ ROLES : "mendefinisikan"
    ROLES ||--o{ ROLE_PERMISSIONS : "memiliki"
    PERMISSIONS ||--o{ ROLE_PERMISSIONS : "diberikan"
    TENANT_USERS ||--o{ TENANT_USER_ROLES : "diberi"
    ROLES ||--o{ TENANT_USER_ROLES : "ditugaskan"
    TENANTS ||--o{ DEVICES : "mendaftarkan"
    TENANTS ||--o{ AUDIT_LOGS : "mencatat"
    USERS ||--o{ AUDIT_LOGS : "melakukan"
    USERS ||--o{ USER_SESSIONS : "memiliki"

    TENANTS {
        uuid id PK
        string code UK "subdomain"
        string legal_name
        string coop_type "KONSUMEN KSP KSU KOPDES PRODUSEN"
        string pattern "KONVENSIONAL SYARIAH"
        string accounting_framework "SAK_EP SAK_EMKM"
        string service_loop "CLOSED OPEN"
        string isolation_mode "POOL SILO"
        string db_route "null bila POOL"
        string timezone "Asia/Jakarta dst"
        string status "TRIAL ACTIVE SUSPENDED CLOSED"
    }
    PLANS {
        uuid id PK
        string code UK
        string name
        decimal price_monthly
        jsonb limits "kuota anggota, terminal, storage"
    }
    PLAN_FEATURES {
        uuid plan_id FK
        string feature_code
        bool enabled
    }
    TENANT_SUBSCRIPTIONS {
        uuid id PK
        uuid tenant_id FK
        uuid plan_id FK
        date period_start
        date period_end
        string status
    }
    TENANT_FEATURE_OVERRIDES {
        uuid tenant_id FK
        string feature_code
        bool enabled
    }
    USERS {
        uuid id PK
        string email UK
        string phone UK
        string password_hash "argon2id"
        string mfa_secret_enc
        string status
        timestamptz last_login_at
    }
    TENANT_USERS {
        uuid id PK
        uuid tenant_id FK
        uuid user_id FK
        uuid officer_id FK "jabatan pengurus bila ada"
        string status
    }
    ROLES {
        uuid id PK
        uuid tenant_id FK "null = peran sistem"
        string code
        string name
        bool is_system
    }
    PERMISSIONS {
        uuid id PK
        string code UK "acc.journal.post"
        string module
    }
    ROLE_PERMISSIONS {
        uuid role_id FK
        uuid permission_id FK
    }
    TENANT_USER_ROLES {
        uuid tenant_user_id FK
        uuid role_id FK
        string scope_type "ALL BRANCH UNIT OUTLET WAREHOUSE"
        uuid scope_id
    }
    DEVICES {
        uuid id PK
        uuid tenant_id FK
        uuid outlet_id FK
        string device_code UK "prefiks nomor struk"
        string public_key
        string status
        timestamptz last_sync_at
        string bundle_version
    }
    USER_SESSIONS {
        uuid id PK
        uuid user_id FK
        uuid tenant_id FK
        string refresh_token_hash
        timestamptz expires_at
        string ip
    }
    AUDIT_LOGS {
        uuid id PK
        uuid tenant_id FK
        uuid user_id FK
        string action
        string entity_type
        uuid entity_id
        jsonb before_data
        jsonb after_data
        string ip
        string prev_hash
        string hash "rantai hash anti-ubah"
        timestamptz created_at
    }
```

## 3. Organisasi, Keanggotaan & Loyalty

```mermaid
erDiagram
    TENANTS ||--o{ BRANCHES : "memiliki"
    TENANTS ||--o{ BUSINESS_UNITS : "memiliki"
    TENANTS ||--o{ FISCAL_YEARS : "memiliki"
    FISCAL_YEARS ||--|{ FISCAL_PERIODS : "terdiri dari"
    TENANTS ||--o{ OFFICERS : "dipimpin"
    MEMBERS ||--o{ OFFICERS : "menjabat"
    BRANCHES ||--o{ MEMBERS : "terdaftar di"
    MEMBER_GROUPS ||--o{ MEMBERS : "mengelompokkan"
    MEMBERS ||--o{ MEMBER_BENEFICIARIES : "menunjuk"
    MEMBERS ||--o{ MEMBER_DOCUMENTS : "melampirkan"
    MEMBERS ||--o{ MEMBER_STATUS_HISTORIES : "berubah status"
    MEMBERS ||--o{ MEMBER_CARDS : "memegang"
    MEMBERS ||--o{ MEMBER_CONSENTS : "memberi"
    MEMBERSHIP_TIERS ||--o{ LOYALTY_ACCOUNTS : "mengklasifikasi"
    MEMBERS ||--o| LOYALTY_ACCOUNTS : "memiliki"
    CUSTOMERS ||--o| LOYALTY_ACCOUNTS : "memiliki"
    CUSTOMERS }o--o| MEMBERS : "dikonversi menjadi"
    LOYALTY_ACCOUNTS ||--o{ POINT_LEDGER : "mencatat"
    LOYALTY_ACCOUNTS ||--o{ POINT_BUCKETS : "menyimpan"
    MEMBER_LOGINS ||--o{ MEMBER_LOGIN_LINKS : "tertaut"
    MEMBERS ||--o{ MEMBER_LOGIN_LINKS : "diakses via"

    BRANCHES {
        uuid id PK
        uuid tenant_id FK
        string code
        string name
        string address
    }
    BUSINESS_UNITS {
        uuid id PK
        uuid tenant_id FK
        string code
        string name
        string unit_type "SIMPAN_PINJAM TOKO APOTEK JASA LOGISTIK"
    }
    FISCAL_YEARS {
        uuid id PK
        uuid tenant_id FK
        int year
        date start_date
        date end_date
        string status "OPEN CLOSED"
    }
    FISCAL_PERIODS {
        uuid id PK
        uuid tenant_id FK
        uuid fiscal_year_id FK
        int period_no
        date start_date
        date end_date
        string status "OPEN SOFT_CLOSED CLOSED"
        timestamptz closed_at
    }
    OFFICERS {
        uuid id PK
        uuid tenant_id FK
        uuid member_id FK
        string position "KETUA SEKRETARIS BENDAHARA PENGAWAS MANAJER"
        date term_start
        date term_end
        string decree_no
    }
    MEMBERS {
        uuid id PK
        uuid tenant_id FK
        string member_no UK "unik per tenant"
        string status "CALON AKTIF PASIF DIBERHENTIKAN KELUAR MENINGGAL DITOLAK"
        string full_name
        string nik_enc
        string nik_hash "unik per tenant"
        string kk_no_enc
        date birth_date
        string phone
        uuid branch_id FK
        uuid member_group_id FK
        decimal monthly_income
        date joined_at
        date exited_at
        jsonb custom_fields
    }
    MEMBER_GROUPS {
        uuid id PK
        uuid tenant_id FK
        string name
        string group_type "INSTANSI DUSUN KELOMPOK_TANI LAINNYA"
        string payroll_contact
    }
    MEMBER_BENEFICIARIES {
        uuid id PK
        uuid member_id FK
        string name
        string relation
        decimal share_pct
    }
    MEMBER_DOCUMENTS {
        uuid id PK
        uuid member_id FK
        string doc_type "KTP KK FOTO TTD LAINNYA"
        uuid attachment_id FK
    }
    MEMBER_STATUS_HISTORIES {
        uuid id PK
        uuid member_id FK
        string from_status
        string to_status
        string reason
        uuid rule_version_id FK
        timestamptz changed_at
    }
    MEMBER_CARDS {
        uuid id PK
        uuid member_id FK
        string card_no UK
        string qr_token_hash
        string status "AKTIF HILANG DIGANTI"
    }
    MEMBER_CONSENTS {
        uuid id PK
        uuid member_id FK
        string consent_type
        string policy_version
        timestamptz granted_at
        timestamptz revoked_at
    }
    CUSTOMERS {
        uuid id PK
        uuid tenant_id FK
        string name
        string phone
        uuid converted_member_id FK
    }
    MEMBERSHIP_TIERS {
        uuid id PK
        uuid tenant_id FK
        string code
        string name
        int rank
    }
    LOYALTY_ACCOUNTS {
        uuid id PK
        uuid tenant_id FK
        uuid member_id FK "salah satu terisi"
        uuid customer_id FK "salah satu terisi"
        uuid tier_id FK
        decimal points_balance
    }
    POINT_LEDGER {
        uuid id PK
        uuid tenant_id FK
        uuid loyalty_account_id FK
        string entry_type "EARN REDEEM EXPIRE ADJUST REVERSE"
        decimal points
        decimal value_amount
        string source_type
        uuid source_id
        uuid rule_version_id FK
        uuid journal_entry_id FK
    }
    POINT_BUCKETS {
        uuid id PK
        uuid loyalty_account_id FK
        decimal earned_points
        decimal remaining_points
        date expires_at
    }
    MEMBER_LOGINS {
        uuid id PK
        string phone UK "global lintas tenant"
        string pin_hash
        string status
    }
    MEMBER_LOGIN_LINKS {
        uuid member_login_id FK
        uuid tenant_id FK
        uuid member_id FK
    }
```

## 4. Simpanan

```mermaid
erDiagram
    SAVING_PRODUCTS ||--o{ SAVING_ACCOUNTS : "jenis"
    MEMBERS ||--o{ SAVING_ACCOUNTS : "memiliki"
    SAVING_ACCOUNTS ||--o{ SAVING_TRANSACTIONS : "mutasi"
    SAVING_ACCOUNTS ||--o{ SAVING_INTEREST_ACCRUALS : "akrual jasa"
    MEMBERS ||--o{ SAVING_BILLS : "ditagih"
    SAVING_PRODUCTS ||--o{ SAVING_BILLS : "untuk"
    SAVING_BILLS ||--o{ SAVING_BILL_PAYMENTS : "dilunasi"
    SAVING_TRANSACTIONS ||--o{ SAVING_BILL_PAYMENTS : "dialokasikan"
    MEMBER_GROUPS ||--o{ PAYROLL_BATCHES : "ditagihkan"
    PAYROLL_BATCHES ||--|{ PAYROLL_BATCH_ITEMS : "berisi"
    MEMBERS ||--o{ PAYROLL_BATCH_ITEMS : "dipotong"
    SAVING_TRANSACTIONS }o--o| JOURNAL_ENTRIES : "dijurnal"

    SAVING_PRODUCTS {
        uuid id PK
        uuid tenant_id FK
        string code
        string name
        string kind "POKOK WAJIB SUKARELA BERJANGKA KHUSUS"
        string accounting_class "EQUITY LIABILITY"
        decimal min_deposit
        decimal min_balance
        bool withdrawable
        jsonb settings
        string status
    }
    SAVING_ACCOUNTS {
        uuid id PK
        uuid tenant_id FK
        string account_no UK
        uuid member_id FK
        uuid product_id FK
        string status "AKTIF DIBEKUKAN DITUTUP"
        decimal balance "cache, sumber kebenaran = mutasi"
        decimal hold_amount
        date opened_at
        date closed_at
        int term_months "berjangka"
        decimal fixed_rate "berjangka"
        date maturity_date "berjangka"
        bool auto_rollover
    }
    SAVING_TRANSACTIONS {
        uuid id PK
        uuid tenant_id FK
        uuid saving_account_id FK
        string txn_no UK
        string txn_type "DEPOSIT WITHDRAW INTEREST TAX FEE TRANSFER_IN TRANSFER_OUT SHU REVERSAL"
        string channel "TELLER POS PAYROLL VA QRIS COLLECTOR AUTODEBET SHU"
        decimal amount
        decimal balance_after
        date value_date
        string reference_type
        uuid reference_id
        uuid journal_entry_id FK
        uuid reversed_by_id FK
        string client_txn_id UK "idempotensi"
    }
    SAVING_INTEREST_ACCRUALS {
        uuid id PK
        uuid saving_account_id FK
        date accrual_date
        decimal base_balance
        decimal rate
        decimal amount
        uuid rule_version_id FK
        bool capitalized
    }
    SAVING_BILLS {
        uuid id PK
        uuid tenant_id FK
        uuid member_id FK
        uuid product_id FK
        string period "YYYY-MM"
        decimal amount
        decimal paid_amount
        decimal penalty_amount
        date due_date
        string status "TERBUKA SEBAGIAN LUNAS DIHAPUS"
    }
    SAVING_BILL_PAYMENTS {
        uuid saving_bill_id FK
        uuid saving_transaction_id FK
        decimal amount
    }
    PAYROLL_BATCHES {
        uuid id PK
        uuid tenant_id FK
        uuid member_group_id FK
        string period
        string status "DRAFT DIKIRIM REALISASI_DIIMPOR SELESAI"
        decimal billed_total
        decimal realized_total
        uuid export_file_id FK
        uuid import_file_id FK
    }
    PAYROLL_BATCH_ITEMS {
        uuid id PK
        uuid payroll_batch_id FK
        uuid member_id FK
        string item_type "SIMPANAN_WAJIB ANGSURAN KREDIT_BELANJA LAINNYA"
        string reference_type
        uuid reference_id
        int priority
        decimal billed_amount
        decimal realized_amount
    }
```

## 5. Pinjaman

```mermaid
erDiagram
    LOAN_PRODUCTS ||--o{ LOAN_APPLICATIONS : "diajukan"
    MEMBERS ||--o{ LOAN_APPLICATIONS : "mengajukan"
    LOAN_APPLICATIONS ||--o| LOANS : "menjadi"
    LOAN_APPLICATIONS ||--o| WORKFLOW_INSTANCES : "disetujui via"
    LOANS ||--|{ LOAN_SCHEDULES : "dijadwalkan"
    LOANS ||--o{ LOAN_PAYMENTS : "dibayar"
    LOAN_PAYMENTS ||--o{ LOAN_PAYMENT_ALLOCATIONS : "dialokasikan"
    LOAN_SCHEDULES ||--o{ LOAN_PAYMENT_ALLOCATIONS : "menerima"
    LOANS ||--o{ LOAN_COLLATERALS : "dijamin"
    LOANS ||--o{ LOAN_COLLECTIBILITY_SNAPSHOTS : "dinilai"
    LOANS ||--o{ LOAN_RESTRUCTURES : "direstrukturisasi"
    LOANS ||--o{ LOAN_WRITEOFFS : "dihapus buku"

    LOAN_PRODUCTS {
        uuid id PK
        uuid tenant_id FK
        string code
        string name
        string interest_method "FLAT EFFECTIVE ANNUITY"
        decimal default_rate_annual
        int min_tenor
        int max_tenor
        string rate_period "BULAN TAHUN"
        jsonb fee_config
        bool members_only
        string status
    }
    LOAN_APPLICATIONS {
        uuid id PK
        uuid tenant_id FK
        string application_no UK
        uuid member_id FK
        uuid product_id FK
        decimal amount_requested
        int tenor_months
        string purpose
        string channel "TELLER APP"
        string status "DRAFT DIAJUKAN DITINJAU DISETUJUI DITOLAK DIBATALKAN DICAIRKAN"
        decimal plafond_calculated
        jsonb eligibility_trace
        jsonb rule_versions "snapshot versi rule"
        uuid workflow_instance_id FK
    }
    LOANS {
        uuid id PK
        uuid tenant_id FK
        string loan_no UK
        uuid application_id FK
        uuid member_id FK
        uuid product_id FK
        decimal principal
        decimal rate_annual
        string interest_method
        int tenor_months
        date disbursed_at
        date first_due_date
        date maturity_date
        decimal outstanding_principal
        decimal accrued_interest
        int dpd
        string collectibility
        string status "AKTIF LUNAS RESTRUKTURISASI HAPUS_BUKU"
        uuid disbursement_journal_id FK
        uuid contract_attachment_id FK
    }
    LOAN_SCHEDULES {
        uuid id PK
        uuid loan_id FK
        int installment_no
        date due_date
        decimal principal_due
        decimal interest_due
        decimal principal_paid
        decimal interest_paid
        decimal penalty_due
        decimal penalty_paid
        string status "BELUM JATUH_TEMPO SEBAGIAN LUNAS"
    }
    LOAN_PAYMENTS {
        uuid id PK
        uuid tenant_id FK
        uuid loan_id FK
        string payment_no UK
        date paid_at
        decimal amount
        string channel "TELLER PAYROLL AUTODEBET VA QRIS COLLECTOR"
        uuid journal_entry_id FK
        uuid reversed_by_id FK
        string client_txn_id UK
    }
    LOAN_PAYMENT_ALLOCATIONS {
        uuid loan_payment_id FK
        uuid loan_schedule_id FK
        decimal penalty_amount
        decimal interest_amount
        decimal principal_amount
    }
    LOAN_COLLATERALS {
        uuid id PK
        uuid loan_id FK
        string collateral_type
        string description
        decimal appraised_value
        string status "DITAHAN DIKEMBALIKAN DIEKSEKUSI"
    }
    LOAN_COLLECTIBILITY_SNAPSHOTS {
        uuid id PK
        uuid loan_id FK
        date as_of_date
        int dpd
        string collectibility "LANCAR DPK KURANG_LANCAR DIRAGUKAN MACET"
        decimal impairment_rate
        decimal impairment_amount
        uuid rule_version_id FK
    }
    LOAN_RESTRUCTURES {
        uuid id PK
        uuid loan_id FK
        date effective_date
        jsonb old_terms
        jsonb new_terms
        uuid workflow_instance_id FK
    }
    LOAN_WRITEOFFS {
        uuid id PK
        uuid loan_id FK
        date writeoff_date
        decimal principal_amount
        uuid journal_entry_id FK
    }
```

## 6. POS, Inventori & Pembelian

```mermaid
erDiagram
    OUTLETS ||--o{ CASHIER_SHIFTS : "membuka"
    DEVICES ||--o{ CASHIER_SHIFTS : "dipakai"
    CASHIER_SHIFTS ||--o{ SALES : "mencatat"
    OUTLETS ||--o{ SALES : "terjadi di"
    MEMBERS ||--o{ SALES : "membeli"
    CUSTOMERS ||--o{ SALES : "membeli"
    SALES ||--|{ SALE_ITEMS : "berisi"
    SALES ||--|{ SALE_PAYMENTS : "dibayar"
    SALES ||--o{ SALE_RETURNS : "diretur"
    SALE_RETURNS ||--|{ SALE_RETURN_ITEMS : "berisi"
    PRODUCTS ||--o{ SALE_ITEMS : "terjual"
    PRODUCT_CATEGORIES ||--o{ PRODUCTS : "mengelompokkan"
    PRODUCTS ||--o{ PRODUCT_BARCODES : "memiliki"
    PRODUCTS ||--o{ PRODUCT_UOMS : "dikonversi"
    PRICE_LISTS ||--o{ PRICE_LIST_ITEMS : "berisi"
    PRODUCTS ||--o{ PRICE_LIST_ITEMS : "dihargai"
    WAREHOUSES ||--o{ STOCK_BALANCES : "menyimpan"
    PRODUCTS ||--o{ STOCK_BALANCES : "tersedia"
    WAREHOUSES ||--o{ STOCK_MOVEMENTS : "mutasi"
    PRODUCTS ||--o{ STOCK_MOVEMENTS : "bergerak"
    OUTLETS }o--|| WAREHOUSES : "mengambil stok dari"
    SUPPLIERS ||--o{ PURCHASE_ORDERS : "menerima"
    PURCHASE_ORDERS ||--|{ PURCHASE_ORDER_ITEMS : "berisi"
    PURCHASE_ORDERS ||--o{ GOODS_RECEIPTS : "diterima"
    GOODS_RECEIPTS ||--|{ GOODS_RECEIPT_ITEMS : "berisi"
    SUPPLIERS ||--o{ SUPPLIER_INVOICES : "menagih"
    SUPPLIER_INVOICES ||--o{ SUPPLIER_PAYMENT_LINES : "dibayar"
    SUPPLIERS ||--o{ PRODUCTS : "menitipkan konsinyasi"
    MEMBERS ||--o| SUPPLIERS : "dapat menjadi pemasok"
    MEMBERS ||--o{ RECEIVABLES : "berutang belanja"
    SALES ||--o{ RECEIVABLES : "menimbulkan"
    WAREHOUSES ||--o{ STOCK_OPNAMES : "dihitung"
    STOCK_OPNAMES ||--|{ STOCK_OPNAME_ITEMS : "berisi"

    OUTLETS {
        uuid id PK
        uuid tenant_id FK
        uuid branch_id FK
        uuid business_unit_id FK
        uuid warehouse_id FK
        string code
        string name
        jsonb receipt_settings
    }
    CASHIER_SHIFTS {
        uuid id PK
        uuid tenant_id FK
        uuid outlet_id FK
        uuid device_id FK
        uuid cashier_user_id FK
        timestamptz opened_at
        decimal opening_cash
        timestamptz closed_at
        decimal expected_cash
        decimal counted_cash
        decimal difference
        jsonb denomination_count
        string status "OPEN CLOSED APPROVED"
        uuid journal_entry_id FK
    }
    SALES {
        uuid id PK
        uuid tenant_id FK
        uuid outlet_id FK
        uuid shift_id FK
        uuid device_id FK
        string receipt_no UK "prefiks perangkat"
        string client_txn_id UK "UUIDv7 idempotensi"
        uuid member_id FK
        uuid customer_id FK
        string buyer_type "ANGGOTA NON_ANGGOTA"
        timestamptz sold_at
        decimal subtotal
        decimal discount_total
        decimal tax_total
        decimal grand_total
        decimal cost_total
        string status "SELESAI VOID RETUR_SEBAGIAN RETUR_PENUH"
        bool created_offline
        timestamptz synced_at
        string rule_bundle_version
        uuid journal_entry_id FK
    }
    SALE_ITEMS {
        uuid id PK
        uuid sale_id FK
        uuid product_id FK
        uuid uom_id FK
        decimal qty
        decimal unit_price
        uuid price_list_id FK
        decimal discount_amount
        decimal tax_amount
        decimal line_total
        decimal unit_cost
        bool is_consignment
        jsonb applied_rules
    }
    SALE_PAYMENTS {
        uuid id PK
        uuid sale_id FK
        string method "CASH QRIS EDC TRANSFER SAVING_BALANCE MEMBER_CREDIT POINTS VOUCHER"
        decimal amount
        string external_ref
        uuid saving_account_id FK
        decimal points_used
        string status "BERHASIL PENDING GAGAL"
    }
    SALE_RETURNS {
        uuid id PK
        uuid sale_id FK
        string return_no UK
        decimal amount
        string refund_method
        uuid approved_by FK
        uuid journal_entry_id FK
    }
    SALE_RETURN_ITEMS {
        uuid sale_return_id FK
        uuid sale_item_id FK
        decimal qty
    }
    RECEIVABLES {
        uuid id PK
        uuid tenant_id FK
        uuid member_id FK
        string source_type "SALE PAYROLL_SHORTFALL OFFLINE_CONFLICT"
        uuid source_id
        decimal amount
        decimal paid_amount
        date due_date
        string status
    }
    PRODUCTS {
        uuid id PK
        uuid tenant_id FK
        string sku UK
        string name
        uuid category_id FK
        uuid base_uom_id FK
        bool is_stock_item
        bool is_consignment
        uuid consignor_supplier_id FK
        decimal consignment_commission_pct
        string tax_code
        decimal min_stock
        bool track_batch
        jsonb custom_fields
    }
    PRODUCT_CATEGORIES {
        uuid id PK
        uuid tenant_id FK
        uuid parent_id FK
        string code
        string name
    }
    PRODUCT_BARCODES {
        uuid id PK
        uuid product_id FK
        string barcode UK
        uuid uom_id FK
    }
    PRODUCT_UOMS {
        uuid product_id FK
        uuid uom_id FK
        decimal factor_to_base
    }
    PRICE_LISTS {
        uuid id PK
        uuid tenant_id FK
        string code "ANGGOTA UMUM GROSIR"
        int priority
        date valid_from
        date valid_to
    }
    PRICE_LIST_ITEMS {
        uuid price_list_id FK
        uuid product_id FK
        uuid uom_id FK
        decimal min_qty
        decimal price
    }
    WAREHOUSES {
        uuid id PK
        uuid tenant_id FK
        uuid branch_id FK
        string code
        string name
    }
    STOCK_BALANCES {
        uuid warehouse_id FK
        uuid product_id FK
        decimal qty_on_hand
        decimal avg_cost
        timestamptz updated_at
    }
    STOCK_MOVEMENTS {
        uuid id PK
        uuid tenant_id FK
        uuid warehouse_id FK
        uuid product_id FK
        string movement_type "SALE SALE_RETURN PURCHASE PURCHASE_RETURN TRANSFER_IN TRANSFER_OUT ADJUSTMENT"
        decimal qty
        decimal unit_cost
        decimal total_cost
        string batch_no
        date expiry_date
        string reference_type
        uuid reference_id
        timestamptz moved_at
    }
    SUPPLIERS {
        uuid id PK
        uuid tenant_id FK
        string code
        string name
        uuid member_id FK "bila pemasok adalah anggota"
        int payment_terms_days
        string npwp
    }
    PURCHASE_ORDERS {
        uuid id PK
        uuid tenant_id FK
        string po_no UK
        uuid supplier_id FK
        date order_date
        string status
        decimal total
    }
    PURCHASE_ORDER_ITEMS {
        uuid id PK
        uuid purchase_order_id FK
        uuid product_id FK
        decimal qty
        decimal unit_price
    }
    GOODS_RECEIPTS {
        uuid id PK
        uuid tenant_id FK
        string grn_no UK
        uuid purchase_order_id FK
        uuid warehouse_id FK
        date received_at
        uuid journal_entry_id FK
    }
    GOODS_RECEIPT_ITEMS {
        uuid id PK
        uuid goods_receipt_id FK
        uuid product_id FK
        decimal qty
        decimal unit_cost
        string batch_no
        date expiry_date
    }
    SUPPLIER_INVOICES {
        uuid id PK
        uuid tenant_id FK
        uuid supplier_id FK
        string invoice_no
        date invoice_date
        date due_date
        decimal amount
        decimal tax_amount
        decimal paid_amount
        uuid journal_entry_id FK
    }
    SUPPLIER_PAYMENT_LINES {
        uuid id PK
        uuid supplier_invoice_id FK
        decimal amount
        uuid journal_entry_id FK
    }
    STOCK_OPNAMES {
        uuid id PK
        uuid tenant_id FK
        uuid warehouse_id FK
        date count_date
        string status "DRAFT DIHITUNG DISETUJUI"
        uuid journal_entry_id FK
    }
    STOCK_OPNAME_ITEMS {
        uuid stock_opname_id FK
        uuid product_id FK
        decimal system_qty
        decimal counted_qty
        decimal difference_value
    }
```

## 7. Akuntansi

```mermaid
erDiagram
    COA_TEMPLATES ||--|{ COA_TEMPLATE_ACCOUNTS : "berisi"
    COA_TEMPLATE_ACCOUNTS ||--o{ ACCOUNTS : "menjadi dasar"
    ACCOUNTS ||--o{ ACCOUNTS : "induk dari"
    ACCOUNTS ||--o{ ACCOUNT_ROLE_MAPPINGS : "dipetakan"
    ACCOUNTS ||--o{ JOURNAL_LINES : "didebet/dikredit"
    JOURNAL_ENTRIES ||--|{ JOURNAL_LINES : "terdiri dari"
    FISCAL_PERIODS ||--o{ JOURNAL_ENTRIES : "dalam"
    JOURNAL_ENTRIES ||--o| JOURNAL_ENTRIES : "dibalik oleh"
    ACCOUNTS ||--o{ ACCOUNT_BALANCES : "saldo"
    FISCAL_PERIODS ||--o{ ACCOUNT_BALANCES : "per periode"
    REPORT_TEMPLATES ||--|{ REPORT_LINES : "tersusun"
    REPORT_LINES ||--o{ ACCOUNTS : "menampung"
    ACCOUNTS ||--o{ CASH_TRANSACTIONS : "kas/bank"
    ACCOUNTS ||--o{ BANK_STATEMENTS : "rekening"
    BANK_STATEMENTS ||--|{ BANK_STATEMENT_LINES : "berisi"
    BANK_STATEMENT_LINES }o--o| JOURNAL_LINES : "dicocokkan"
    FIXED_ASSETS ||--o{ DEPRECIATION_LINES : "disusutkan"
    DEPRECIATION_RUNS ||--|{ DEPRECIATION_LINES : "berisi"
    BUDGETS ||--|{ BUDGET_LINES : "berisi"
    ACCOUNTS ||--o{ BUDGET_LINES : "dianggarkan"
    FISCAL_PERIODS ||--o{ PERIOD_CLOSE_CHECKS : "diperiksa"
    TAX_CODES }o--|| ACCOUNTS : "dicatat ke"

    COA_TEMPLATES {
        uuid id PK
        string code "KONSUMEN KSP KSU KOPDES"
        string version
        string status
    }
    COA_TEMPLATE_ACCOUNTS {
        uuid id PK
        uuid coa_template_id FK
        string code
        string name
        int level
        string report_line_code
        string cashflow_class
        bool extendable
    }
    ACCOUNTS {
        uuid id PK
        uuid tenant_id FK
        string code UK "unik per tenant"
        string name
        string alias_name
        uuid parent_id FK
        int level "1 sampai 5"
        string account_type "ASSET LIABILITY EQUITY REVENUE COGS EXPENSE OTHER_INCOME OTHER_EXPENSE TAX"
        string normal_balance "D K"
        bool is_postable
        bool is_standard "terkunci"
        bool is_contra
        string report_line_code
        string cashflow_class
        string subledger_type "MEMBER SAVING LOAN AR AP INVENTORY CONSIGNMENT null"
        bool manual_posting_allowed
        string status "AKTIF NONAKTIF"
    }
    ACCOUNT_ROLE_MAPPINGS {
        uuid id PK
        uuid tenant_id FK
        string role_code "KAS_GERAI PENJUALAN_ANGGOTA"
        string scope_type "TENANT BRANCH OUTLET PRODUCT SAVING_PRODUCT LOAN_PRODUCT PAYMENT_PROVIDER"
        uuid scope_id
        uuid account_id FK
        date effective_from
    }
    JOURNAL_ENTRIES {
        uuid id PK
        uuid tenant_id FK
        string entry_no UK
        date entry_date
        uuid fiscal_period_id FK
        string journal_type "UMUM OTOMATIS PENYESUAIAN PENUTUP PEMBALIK SALDO_AWAL"
        string source_type
        uuid source_id
        string description
        string status "DRAFT DIAJUKAN POSTED DIBALIK"
        decimal total_debit
        decimal total_credit
        uuid posting_template_version_id FK
        string idempotency_key UK
        uuid reversal_of_id FK
        uuid created_by FK
        uuid approved_by FK
        timestamptz posted_at
    }
    JOURNAL_LINES {
        uuid id PK
        uuid tenant_id FK
        uuid journal_entry_id FK
        date entry_date "kunci partisi"
        int line_no
        uuid account_id FK
        decimal debit
        decimal credit
        string memo
        uuid branch_id FK
        uuid business_unit_id FK
        uuid project_id FK
        string participant_type "ANGGOTA NON_ANGGOTA UMUM"
        uuid member_id FK
        uuid supplier_id FK
        string subledger_type
        uuid subledger_id
    }
    ACCOUNT_BALANCES {
        uuid tenant_id FK
        uuid account_id FK
        uuid fiscal_period_id FK
        uuid branch_id FK
        uuid business_unit_id FK
        decimal opening_balance
        decimal period_debit
        decimal period_credit
        decimal closing_balance
    }
    REPORT_TEMPLATES {
        uuid id PK
        uuid tenant_id FK "null = template platform"
        string code "NERACA LHU LPE ARUS_KAS_LANGSUNG ARUS_KAS_TIDAK_LANGSUNG"
        string version
        string framework "SAK_EP SAK_EMKM"
    }
    REPORT_LINES {
        uuid id PK
        uuid report_template_id FK
        string code UK
        string label
        uuid parent_id FK
        int sort_order
        string formula "SUM anak atau ekspresi"
        int sign
    }
    CASH_TRANSACTIONS {
        uuid id PK
        uuid tenant_id FK
        string doc_no UK
        string txn_type "MASUK KELUAR TRANSFER"
        uuid cash_account_id FK
        uuid counter_account_id FK
        date txn_date
        decimal amount
        string counterparty
        uuid journal_entry_id FK
    }
    BANK_STATEMENTS {
        uuid id PK
        uuid tenant_id FK
        uuid account_id FK
        date period_start
        date period_end
        decimal opening_balance
        decimal closing_balance
    }
    BANK_STATEMENT_LINES {
        uuid id PK
        uuid bank_statement_id FK
        date txn_date
        string description
        decimal amount
        uuid matched_journal_line_id FK
        string match_status
    }
    FIXED_ASSETS {
        uuid id PK
        uuid tenant_id FK
        string asset_no UK
        string name
        string category
        date acquisition_date
        decimal cost
        decimal residual_value
        int useful_life_months
        string method "GARIS_LURUS SALDO_MENURUN"
        decimal accumulated_depreciation
        string status
    }
    DEPRECIATION_RUNS {
        uuid id PK
        uuid tenant_id FK
        uuid fiscal_period_id FK
        uuid journal_entry_id FK
    }
    DEPRECIATION_LINES {
        uuid depreciation_run_id FK
        uuid fixed_asset_id FK
        decimal amount
    }
    BUDGETS {
        uuid id PK
        uuid tenant_id FK
        uuid fiscal_year_id FK
        string name
        string status
    }
    BUDGET_LINES {
        uuid budget_id FK
        uuid account_id FK
        uuid business_unit_id FK
        int period_no
        decimal amount
    }
    PERIOD_CLOSE_CHECKS {
        uuid id PK
        uuid fiscal_period_id FK
        string check_code
        string status "LULUS GAGAL DIABAIKAN"
        jsonb detail
        timestamptz checked_at
    }
    TAX_CODES {
        uuid id PK
        uuid tenant_id FK
        string code
        string name
        decimal rate
        uuid account_id FK
        date effective_from
    }
```

## 8. SHU & RAT

```mermaid
erDiagram
    FISCAL_YEARS ||--o| SHU_PERIODS : "menghasilkan"
    SHU_PERIODS ||--|{ SHU_ALLOCATIONS : "dialokasikan"
    SHU_PERIODS ||--o{ SHU_MEMBER_SHARES : "dibagikan"
    MEMBERS ||--o{ SHU_MEMBER_SHARES : "menerima"
    SHU_PERIODS ||--o{ SHU_SCENARIOS : "disimulasikan"
    SHU_PERIODS ||--o| RAT_MEETINGS : "disahkan di"
    SHU_ALLOCATIONS }o--|| ACCOUNTS : "dicatat ke"
    SHU_PERIODS ||--o{ FUND_USAGES : "dana dipakai"

    SHU_PERIODS {
        uuid id PK
        uuid tenant_id FK
        uuid fiscal_year_id FK
        decimal shu_before_tax
        decimal income_tax
        decimal shu_net
        uuid allocation_rule_version_id FK
        uuid capital_basis_rule_version_id FK
        uuid business_basis_rule_version_id FK
        string status "DRAFT DISIMULASIKAN DISAHKAN_RAT DIJURNAL DIDISTRIBUSIKAN"
        uuid distribution_journal_id FK
    }
    SHU_ALLOCATIONS {
        uuid id PK
        uuid shu_period_id FK
        string component "CADANGAN JASA_MODAL JASA_USAHA PENGURUS KARYAWAN PENDIDIKAN SOSIAL PDK"
        decimal percentage
        decimal amount
        uuid account_id FK
    }
    SHU_MEMBER_SHARES {
        uuid id PK
        uuid shu_period_id FK
        uuid member_id FK
        decimal capital_basis
        decimal business_basis
        jsonb business_basis_detail "belanja, jasa pinjaman, penjualan"
        decimal capital_share
        decimal business_share
        decimal gross_amount
        decimal tax_amount
        decimal net_amount
        string payout_method "TUNAI TRANSFER SIMPANAN"
        timestamptz paid_at
        uuid saving_transaction_id FK
    }
    SHU_SCENARIOS {
        uuid id PK
        uuid shu_period_id FK
        string name
        jsonb allocation
        jsonb summary
    }
    RAT_MEETINGS {
        uuid id PK
        uuid tenant_id FK
        uuid fiscal_year_id FK
        date meeting_date
        int attendees
        bool quorum_met
        uuid minutes_attachment_id FK
        string decision_no
    }
    FUND_USAGES {
        uuid id PK
        uuid tenant_id FK
        string fund_component
        date usage_date
        decimal amount
        string description
        uuid journal_entry_id FK
    }
```

## 9. Rule Engine & Workflow

```mermaid
erDiagram
    RULE_PACKS ||--|{ RULE_PACK_ITEMS : "berisi"
    RULE_PACK_ITEMS ||--o{ RULE_DEFINITIONS : "disalin menjadi"
    RULE_DEFINITIONS ||--|{ RULE_VERSIONS : "berversi"
    RULE_DEFINITIONS ||--o{ RULE_TEST_CASES : "diuji dengan"
    RULE_VERSIONS ||--o{ RULE_TEST_RUNS : "dijalankan uji"
    RULE_VERSIONS ||--o{ RULE_APPROVALS : "disetujui"
    RULE_VERSIONS ||--o{ RULE_EXECUTIONS : "dieksekusi"
    TENANTS ||--o{ CUSTOM_FIELD_DEFINITIONS : "mendefinisikan"
    TENANTS ||--o{ REFERENCE_TABLES : "memiliki"
    REFERENCE_TABLES ||--|{ REFERENCE_TABLE_ROWS : "berisi"
    RULE_VERSIONS ||--o{ WORKFLOW_INSTANCES : "menjalankan"
    WORKFLOW_INSTANCES ||--|{ WORKFLOW_TASKS : "terdiri dari"
    TENANTS ||--o{ RULE_BUNDLES : "membangun untuk POS"

    RULE_PACKS {
        uuid id PK
        string code "KONSUMEN KSP KSU KOPDES"
        string version
        string status
    }
    RULE_PACK_ITEMS {
        uuid id PK
        uuid rule_pack_id FK
        string rule_code
        string hook_code
        string rule_type
        jsonb default_definition
    }
    RULE_DEFINITIONS {
        uuid id PK
        uuid tenant_id FK
        string code UK "unik per tenant"
        string name
        string hook_code
        string rule_type "PARAMETER FORMULA DECISION_TABLE ECA WORKFLOW POSTING_TEMPLATE"
        string category "OPERASIONAL FINANSIAL AD_ART AKUNTANSI"
        jsonb scope "cabang unit gerai produk kelompok"
        int priority
        uuid source_pack_item_id FK
        string status "AKTIF NONAKTIF"
    }
    RULE_VERSIONS {
        uuid id PK
        uuid tenant_id FK
        uuid rule_definition_id FK
        int version_no
        jsonb definition "AST JSON"
        string natural_text "kalimat bahasa Indonesia"
        string definition_hash
        date effective_from
        date effective_to
        bool retroactive
        string status "DRAFT VALIDATED IN_REVIEW APPROVED SCHEDULED ACTIVE SUPERSEDED RETIRED"
        bool shadow_mode
        string change_note
        uuid created_by FK
    }
    RULE_APPROVALS {
        uuid id PK
        uuid rule_version_id FK
        uuid approver_user_id FK
        string decision "SETUJU TOLAK"
        string comment
        timestamptz decided_at
    }
    RULE_TEST_CASES {
        uuid id PK
        uuid rule_definition_id FK
        string name
        jsonb input
        jsonb expected
    }
    RULE_TEST_RUNS {
        uuid id PK
        uuid rule_version_id FK
        string run_type "TEST_CASE REPLAY SHADOW"
        jsonb parameters
        jsonb summary "dampak"
        timestamptz started_at
        timestamptz finished_at
    }
    RULE_EXECUTIONS {
        uuid id PK
        uuid tenant_id FK
        uuid rule_version_id FK
        string hook_code
        string subject_type
        uuid subject_id
        string input_hash
        jsonb input_snapshot "terbatas"
        jsonb output
        jsonb trace
        int duration_ms
        timestamptz executed_at "kunci partisi"
    }
    CUSTOM_FIELD_DEFINITIONS {
        uuid id PK
        uuid tenant_id FK
        string entity "MEMBER PRODUCT LOAN"
        string field_key
        string label
        string data_type
        jsonb options
        bool required
    }
    REFERENCE_TABLES {
        uuid id PK
        uuid tenant_id FK
        string code
        string name
        jsonb columns
    }
    REFERENCE_TABLE_ROWS {
        uuid id PK
        uuid reference_table_id FK
        string row_key
        jsonb row_values
        date effective_from
    }
    WORKFLOW_INSTANCES {
        uuid id PK
        uuid tenant_id FK
        uuid rule_version_id FK
        string subject_type "LOAN_APPLICATION JOURNAL RULE_VERSION STOCK_ADJUSTMENT PURCHASE_ORDER MEMBER"
        uuid subject_id
        int current_step
        string status "BERJALAN DISETUJUI DITOLAK DIBATALKAN"
    }
    WORKFLOW_TASKS {
        uuid id PK
        uuid workflow_instance_id FK
        int step_no
        string assignee_type "ROLE POSITION USER"
        string assignee_ref
        uuid acted_by FK
        string action "SETUJU TOLAK REVISI DELEGASI"
        string comment
        timestamptz due_at
        timestamptz acted_at
    }
    RULE_BUNDLES {
        uuid id PK
        uuid tenant_id FK
        string bundle_version
        jsonb rule_version_ids
        string storage_key
        string signature
        timestamptz built_at
    }
```

## 10. Infrastruktur Aplikasi (Outbox, Sinkron, Notifikasi, Integrasi)

```mermaid
erDiagram
    TENANTS ||--o{ OUTBOX_EVENTS : "menghasilkan"
    OUTBOX_EVENTS ||--o{ PROCESSED_EVENTS : "dikonsumsi"
    DEVICES ||--o{ SYNC_BATCHES : "mengunggah"
    SYNC_BATCHES ||--o{ SYNC_EXCEPTIONS : "menghasilkan"
    TENANTS ||--o{ NOTIFICATION_TEMPLATES : "memiliki"
    NOTIFICATION_TEMPLATES ||--o{ NOTIFICATIONS : "dipakai"
    TENANTS ||--o{ PAYMENT_GATEWAY_TXNS : "menerima"
    TENANTS ||--o{ ATTACHMENTS : "menyimpan"
    TENANTS ||--o{ DOCUMENT_SEQUENCES : "menomori"
    TENANTS ||--o{ IMPORT_JOBS : "mengimpor"

    OUTBOX_EVENTS {
        uuid id PK
        uuid tenant_id FK
        string aggregate_type
        uuid aggregate_id
        string event_type "sale.completed saving.deposited loan.disbursed"
        jsonb payload
        timestamptz occurred_at
        timestamptz published_at
        int attempts
        string last_error
    }
    PROCESSED_EVENTS {
        string consumer
        uuid event_id FK
        timestamptz processed_at
    }
    SYNC_BATCHES {
        uuid id PK
        uuid tenant_id FK
        uuid device_id FK
        string client_batch_id UK
        int txn_count
        int accepted_count
        int duplicate_count
        string status
        timestamptz received_at
    }
    SYNC_EXCEPTIONS {
        uuid id PK
        uuid tenant_id FK
        uuid sync_batch_id FK
        string client_txn_id
        string exception_type "SALDO_KURANG POIN_KURANG ANGGOTA_NONAKTIF STOK_NEGATIF BUNDLE_KADALUARSA PERIODE_TERTUTUP"
        jsonb detail
        string status "TERBUKA DITANGANI"
        uuid resolved_by FK
    }
    NOTIFICATION_TEMPLATES {
        uuid id PK
        uuid tenant_id FK
        string code
        string channel "WA PUSH EMAIL SMS"
        string body
        string provider_template_id
    }
    NOTIFICATIONS {
        uuid id PK
        uuid tenant_id FK
        uuid template_id FK
        string recipient_type
        uuid recipient_id
        string channel
        string status "ANTRE TERKIRIM GAGAL DIBACA"
        timestamptz sent_at
    }
    PAYMENT_GATEWAY_TXNS {
        uuid id PK
        uuid tenant_id FK
        string provider
        string external_id UK
        string txn_type "QRIS VA"
        decimal amount
        decimal fee_amount
        string status "PENDING PAID EXPIRED SETTLED"
        string reference_type
        uuid reference_id
        jsonb raw_payload
        timestamptz settled_at
    }
    ATTACHMENTS {
        uuid id PK
        uuid tenant_id FK
        string owner_type
        uuid owner_id
        string storage_key
        string mime_type
        int size_bytes
        string sha256
    }
    DOCUMENT_SEQUENCES {
        uuid tenant_id FK
        string doc_type
        string period_key
        string prefix
        bigint last_no
    }
    IMPORT_JOBS {
        uuid id PK
        uuid tenant_id FK
        string import_type "ANGGOTA SALDO_SIMPANAN PINJAMAN SALDO_AWAL PRODUK STOK"
        string status "VALIDASI DRY_RUN SIAP DIPROSES GAGAL SELESAI"
        jsonb error_report
        uuid file_id FK
    }
```

## 11. Kamus Data — Tabel Inti

### 11.1 `journal_entries`

| Kolom | Tipe | Null | Keterangan |
|---|---|---|---|
| id | uuid | N | PK (UUIDv7) |
| tenant_id | uuid | N | RLS |
| entry_no | text | N | Unik per tenant; format `JU-2027-03-000123` |
| entry_date | date | N | Tanggal akuntansi |
| fiscal_period_id | uuid | N | FK; harus berstatus OPEN (atau SOFT_CLOSED untuk PENYESUAIAN) |
| journal_type | text | N | `UMUM`, `OTOMATIS`, `PENYESUAIAN`, `PENUTUP`, `PEMBALIK`, `SALDO_AWAL` |
| source_type / source_id | text / uuid | Y | Dokumen sumber (mis. `SALE`, `SAVING_TXN`, `LOAN_PAYMENT`) |
| description | text | N | |
| status | text | N | `DRAFT`, `DIAJUKAN`, `POSTED`, `DIBALIK` |
| total_debit / total_credit | numeric(20,2) | N | `CHECK (total_debit = total_credit)` saat POSTED |
| posting_template_version_id | uuid | Y | Versi rule template posting (jurnal otomatis) |
| idempotency_key | text | Y | Unik per tenant; = `event_id` untuk jurnal otomatis |
| reversal_of_id | uuid | Y | Jurnal yang dibalik |
| created_by / approved_by | uuid | N / Y | Maker-checker: `approved_by <> created_by` |
| posted_at | timestamptz | Y | |

**Constraint & trigger:** jurnal `POSTED` tidak dapat di-`UPDATE` kecuali kolom `status` menjadi `DIBALIK`;
`DELETE` ditolak untuk semua status selain `DRAFT`.

### 11.2 `journal_lines` (partisi per tahun `entry_date`)

| Kolom | Tipe | Null | Keterangan |
|---|---|---|---|
| id | uuid | N | PK |
| tenant_id | uuid | N | RLS |
| journal_entry_id | uuid | N | FK |
| entry_date | date | N | Salinan dari header untuk partisi & indeks |
| line_no | int | N | |
| account_id | uuid | N | Harus `is_postable = true` dan `status = AKTIF` |
| debit / credit | numeric(20,2) | N | `CHECK (debit >= 0 AND credit >= 0 AND (debit = 0 OR credit = 0))` |
| memo | text | Y | |
| branch_id / business_unit_id / project_id | uuid | Y | Dimensi |
| participant_type | text | Y | `ANGGOTA`, `NON_ANGGOTA`, `UMUM` |
| member_id / supplier_id | uuid | Y | Pihak terkait |
| subledger_type / subledger_id | text / uuid | Y | Referensi buku pembantu (rekening simpanan, pinjaman, piutang) |

Indeks: `(tenant_id, account_id, entry_date)`, `(tenant_id, subledger_type, subledger_id)`, `(tenant_id, member_id, entry_date)`.

### 11.3 `accounts`

| Kolom | Tipe | Keterangan |
|---|---|---|
| code | text | 1/2/4/6 digit standar atau `NNNNNN.NNN` sub-akun tenant; unik per tenant |
| level | int | 1–5 |
| is_standard | bool | `true` untuk akun dari template (tidak dapat dihapus/diubah kodenya) |
| is_postable | bool | Hanya level 4 tanpa anak, atau level 5 |
| report_line_code | text | Pos laporan (diwarisi sub-akun) |
| cashflow_class | text | `OPERASI_*`, `INVESTASI_*`, `PENDANAAN_*`, `KAS` |
| subledger_type | text | Bila akun kontrol buku pembantu |
| manual_posting_allowed | bool | `false` untuk akun kontrol buku pembantu |

### 11.4 `sales`

| Kolom | Tipe | Keterangan |
|---|---|---|
| client_txn_id | text | UUIDv7 dari perangkat; **unik per tenant** → idempotensi sinkronisasi |
| receipt_no | text | `{device_code}-{yyMMdd}-{seq}`; dicetak di struk saat offline |
| buyer_type | text | Snapshot status pembeli saat transaksi (menentukan akun penjualan & dasar SHU) |
| rule_bundle_version | text | Versi bundle rule yang dipakai menghitung harga/promo/poin |
| created_offline | bool | Ditandai untuk analisis & audit |
| cost_total | numeric(20,2) | Total HPP (barang milik sendiri; konsinyasi tidak termasuk) |

### 11.5 `saving_transactions`

| Kolom | Tipe | Keterangan |
|---|---|---|
| txn_type | text | Lihat diagram §4 |
| amount | numeric(20,2) | Selalu positif; arah ditentukan `txn_type` |
| balance_after | numeric(20,2) | Saldo berjalan; diperbarui dalam transaksi dengan `SELECT … FOR UPDATE` pada rekening |
| value_date | date | Tanggal efektif untuk perhitungan jasa |
| journal_entry_id | uuid | Wajib terisi untuk transaksi selesai (kecuali mode ringkas → diisi saat jurnal ringkas dibuat) |
| reversed_by_id | uuid | Koreksi melalui transaksi pembalik |

### 11.6 `rule_versions`

| Kolom | Tipe | Keterangan |
|---|---|---|
| definition | jsonb | AST JSON sesuai skema `kop.rule/v1` (lihat dokumen 03 §8) |
| definition_hash | text | SHA-256 definisi kanonik; kunci cache kompilasi |
| effective_from / effective_to | date | Tidak boleh tumpang tindih antar versi ACTIVE/SCHEDULED pada definisi yang sama |
| status | text | Siklus hidup (dokumen 03 §9) |
| natural_text | text | Kalimat yang ditinjau penyetuju |

**Exclusion constraint:** `EXCLUDE USING gist (rule_definition_id WITH =, daterange(effective_from, effective_to) WITH &&) WHERE status IN ('SCHEDULED','ACTIVE')`.

## 12. Relasi Lintas Domain (Ringkas)

| Entitas sumber | Entitas tujuan | Makna |
|---|---|---|
| `members` | `saving_accounts`, `loans`, `sales`, `loyalty_accounts`, `shu_member_shares`, `receivables`, `journal_lines.member_id` | Satu identitas anggota di semua modul |
| `sales`, `saving_transactions`, `loan_payments`, `goods_receipts`, `cashier_shifts`, … | `journal_entries` (`source_type`/`source_id` & `journal_entry_id`) | Ketertelusuran dua arah dokumen ↔ jurnal |
| Semua transaksi yang dievaluasi rule | `rule_versions` (snapshot ID/versi) | Reproduksibilitas keputusan |
| `outbox_events` | Posting Engine, notifikasi, loyalty | Integrasi antar modul tanpa kehilangan event |
| `sales.buyer_type = ANGGOTA` + `loan_payments` (jasa) + pembelian dari anggota | `shu_member_shares.business_basis` | Dasar jasa usaha SHU |
| `saving_accounts` (pokok, wajib, opsional sukarela) | `shu_member_shares.capital_basis` | Dasar jasa modal SHU |

[← 05 Flowchart](./05-flowchart.md) · [Lanjut ke 07 — Wireframe →](./07-wireframe.md)
