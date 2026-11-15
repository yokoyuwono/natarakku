# 09 — Penanganan Error & Edge Case

[← Kembali ke daftar isi](./README.md)

---

Dokumen ini adalah **kontrak wajib** bagi seluruh modul: format error API, katalog kode error, kebijakan retry,
idempotensi, konkurensi, penanganan error di UI, serta daftar *edge case* beserta perilaku yang diharapkan.
Setiap fitur baru harus menyebut kode error yang dipakainya dan menambah *edge case* yang relevan ke dokumen ini.

## 1. Prinsip

| # | Prinsip | Penerapan |
|---|---|---|
| E1 | **Gagal secara aman, tidak pernah diam-diam** | Transaksi keuangan yang ragu dihentikan/ditahan, bukan diloloskan. Tidak ada `catch` kosong |
| E2 | **Atomik** | Satu perintah bisnis = satu transaksi DB. Transaksi bisnis + jurnal + event outbox tersimpan bersama atau tidak sama sekali |
| E3 | **Idempoten** | Semua perintah yang mengubah uang/stok/poin dapat diulang tanpa efek ganda |
| E4 | **Tidak ada panggilan eksternal di dalam transaksi DB** | Payment gateway, WA, dan file storage dipanggil sebelum/sesudah transaksi via outbox |
| E5 | **Error dapat ditindaklanjuti** | Pesan menjelaskan apa yang terjadi, mengapa, dan apa yang harus dilakukan; disertai kode referensi untuk dukungan |
| E6 | **Pisahkan pesan pengguna dan detail teknis** | Pengguna melihat pesan bahasa Indonesia; detail teknis (stack, SQL) hanya di log |
| E7 | **Tidak membocorkan data tenant lain** | Akses ke sumber daya tenant lain mengembalikan `404`, bukan `403` |
| E8 | **Setiap error tercatat dengan korelasi** | `trace_id` mengalir dari klien → API → worker → log; ditampilkan sebagai kode referensi pendek |

## 2. Format Error API (RFC 9457 Problem Details)

Semua respons error memakai `Content-Type: application/problem+json`:

```json
{
  "type": "https://docs.koperasios.id/errors/SAV_INSUFFICIENT_BALANCE",
  "title": "Saldo tidak mencukupi",
  "status": 422,
  "code": "SAV_INSUFFICIENT_BALANCE",
  "detail": "Saldo simpanan sukarela Rp 120.000, penarikan Rp 150.000. Saldo minimal setelah penarikan Rp 50.000.",
  "instance": "/api/v1/savings/accounts/0193.../withdrawals",
  "trace_id": "7f3a9c2e1b",
  "retryable": false,
  "errors": [
    { "field": "amount", "code": "MAX_EXCEEDED", "message": "Maksimal Rp 70.000" }
  ],
  "rule_trace": {
    "rule_code": "SAV-MIN-01", "version": 2, "message": "Saldo setelah penarikan minimal Rp 50.000"
  },
  "context": { "available": "120000.00", "requested": "150000.00", "min_balance": "50000.00" }
}
```

| Field | Wajib | Keterangan |
|---|---|---|
| `code` | Ya | Kode stabil dari katalog §4; klien bercabang berdasarkan `code`, **bukan** `title`/`detail` |
| `title` | Ya | Ringkasan bahasa Indonesia (dapat diterjemahkan) |
| `detail` | Ya | Penjelasan spesifik kasus, aman ditampilkan ke pengguna |
| `status` | Ya | Kode HTTP |
| `trace_id` | Ya | Korelasi log; UI menampilkan 8–10 karakter pertama sebagai "Kode ref." |
| `retryable` | Ya | `true` bila klien boleh mengulang otomatis (dengan idempotency key yang sama) |
| `errors[]` | Untuk validasi | Error per field (path JSON), dipetakan ke input form |
| `rule_trace` | Bila diblokir rule | Rule, versi, pesan; UI menampilkan tautan "Lihat aturan" bagi pengguna berizin |
| `context` | Opsional | Data terstruktur untuk UI (nilai uang sebagai string desimal) |

## 3. Klasifikasi Error & Pemetaan HTTP

| Kelas | HTTP | Contoh kode | Retryable | Level log | Alert |
|---|---|---|---|---|---|
| Format/permintaan rusak | 400 | `VAL_MALFORMED_JSON` | Tidak | WARN | Tidak |
| Validasi input | 422 | `VAL_FIELD_INVALID` | Tidak | INFO | Tidak |
| Aturan bisnis / rule engine | 422 | `SAV_INSUFFICIENT_BALANCE`, `RUL_BLOCKED` | Tidak | INFO | Tidak |
| Butuh persetujuan | 202 | `WF_APPROVAL_REQUIRED` (bukan error; respons berisi `workflow_instance_id`) | — | INFO | Tidak |
| Tidak terautentikasi | 401 | `AUTH_TOKEN_EXPIRED` | Ya (setelah refresh) | INFO | Tidak |
| Tidak berwenang | 403 | `IAM_PERMISSION_DENIED` | Tidak | WARN | Bila berulang |
| Tidak ditemukan / tenant lain | 404 | `SYS_NOT_FOUND` | Tidak | INFO | Bila pola enumerasi |
| Konflik status | 409 | `LON_INVALID_STATE`, `ACC_PERIOD_CLOSED` | Tidak | INFO | Tidak |
| Konflik konkurensi | 409 / 412 | `SYS_VERSION_CONFLICT` | Tidak (muat ulang) | INFO | Tidak |
| Idempotensi | 409 | `SYS_IDEMPOTENCY_MISMATCH`, `SYS_REQUEST_IN_PROGRESS` | `IN_PROGRESS`: Ya | WARN | Tidak |
| Sumber daya terkunci | 423 | `POS_DEVICE_LOCKED` | Tidak | INFO | Tidak |
| Kuota / rate limit | 429 | `SYS_RATE_LIMITED`, `PLT_QUOTA_EXCEEDED` | Ya (`Retry-After`) | WARN | Bila > ambang |
| Dependensi eksternal | 502 / 503 / 504 | `PAY_PROVIDER_UNAVAILABLE` | Ya | ERROR | Ya (circuit open) |
| Error sistem tak terduga | 500 | `SYS_INTERNAL` | Ya (sekali) | ERROR | Ya |
| Pemeliharaan | 503 | `SYS_MAINTENANCE` | Ya (`Retry-After`) | INFO | Tidak |

## 4. Katalog Kode Error

Format: `<DOMAIN>_<ALASAN>`. Kode **tidak boleh diubah** setelah rilis; kode baru ditambahkan di sini lewat PR.

### 4.1 Umum, Auth, Platform

| Kode | HTTP | Pesan pengguna (ringkas) |
|---|---|---|
| `VAL_MALFORMED_JSON` | 400 | Format permintaan tidak valid |
| `VAL_FIELD_INVALID` | 422 | Ada isian yang belum benar (lihat `errors[]`) |
| `SYS_NOT_FOUND` | 404 | Data tidak ditemukan |
| `SYS_VERSION_CONFLICT` | 409 | Data sudah diubah pengguna lain. Muat ulang lalu ulangi |
| `SYS_IDEMPOTENCY_MISMATCH` | 409 | Permintaan ganda dengan isi berbeda terdeteksi |
| `SYS_REQUEST_IN_PROGRESS` | 409 | Permintaan yang sama sedang diproses |
| `SYS_RATE_LIMITED` | 429 | Terlalu banyak permintaan. Coba lagi sebentar lagi |
| `SYS_INTERNAL` | 500 | Terjadi kesalahan sistem. Sebutkan kode ref. ke dukungan |
| `SYS_MAINTENANCE` | 503 | Sistem sedang dalam pemeliharaan |
| `AUTH_INVALID_CREDENTIALS` | 401 | Email/no. HP atau kata sandi salah |
| `AUTH_ACCOUNT_LOCKED` | 423 | Akun terkunci sementara karena terlalu banyak percobaan |
| `AUTH_MFA_REQUIRED` | 401 | Masukkan kode 2FA |
| `AUTH_MFA_INVALID` | 401 | Kode 2FA salah atau kedaluwarsa |
| `AUTH_TOKEN_EXPIRED` | 401 | Sesi berakhir |
| `AUTH_OTP_INVALID` / `AUTH_OTP_EXPIRED` | 401 | Kode OTP salah / kedaluwarsa |
| `IAM_PERMISSION_DENIED` | 403 | Anda tidak memiliki izin untuk tindakan ini |
| `IAM_SCOPE_DENIED` | 403 | Data ini di luar cabang/unit yang dapat Anda akses |
| `IAM_MAKER_CHECKER_VIOLATION` | 409 | Pembuat tidak boleh menyetujui transaksinya sendiri |
| `PLT_TENANT_SUSPENDED` | 403 | Layanan koperasi sedang ditangguhkan. Hubungi admin |
| `PLT_FEATURE_DISABLED` | 403 | Fitur tidak tersedia pada paket Anda |
| `PLT_QUOTA_EXCEEDED` | 429 | Kuota paket terlampaui (mis. jumlah anggota) |

### 4.2 Keanggotaan & Loyalty

| Kode | HTTP | Pesan pengguna (ringkas) |
|---|---|---|
| `MBR_NIK_DUPLICATE` | 409 | NIK sudah terdaftar sebagai anggota {no_anggota} |
| `MBR_NIK_FORMAT` | 422 | NIK harus 16 digit angka |
| `MBR_NOT_ACTIVE` | 409 | Anggota tidak berstatus aktif ({status}) |
| `MBR_ACTIVATION_REQUIREMENTS_UNMET` | 409 | Syarat aktivasi belum terpenuhi (daftar di `context`) |
| `MBR_HAS_OBLIGATIONS` | 409 | Anggota masih memiliki kewajiban yang belum diselesaikan |
| `MBR_CARD_INVALID` | 404 | Kartu anggota tidak dikenali atau sudah diganti |
| `MBR_CARD_TOKEN_EXPIRED` | 401 | QR kartu kedaluwarsa, minta anggota memperbarui |
| `LOY_INSUFFICIENT_POINTS` | 422 | Poin tidak mencukupi |
| `LOY_BELOW_MIN_REDEEM` | 422 | Minimal penukaran {n} poin |

### 4.3 Simpanan & Pinjaman

| Kode | HTTP | Pesan pengguna (ringkas) |
|---|---|---|
| `SAV_ACCOUNT_NOT_ACTIVE` | 409 | Rekening dibekukan/ditutup |
| `SAV_INSUFFICIENT_BALANCE` | 422 | Saldo tidak mencukupi |
| `SAV_BELOW_MIN_DEPOSIT` | 422 | Setoran minimal Rp {n} |
| `SAV_NOT_WITHDRAWABLE` | 409 | Simpanan pokok/wajib hanya dapat diambil saat keluar anggota |
| `SAV_DAILY_LIMIT_EXCEEDED` | 422 | Batas penarikan harian terlampaui |
| `SAV_TERM_NOT_MATURED` | 409 | Simpanan berjangka belum jatuh tempo (pencairan dini dikenai penalti) |
| `SAV_PAYROLL_FILE_INVALID` | 422 | Format file potong gaji tidak sesuai (baris error di `context`) |
| `LON_NOT_ELIGIBLE` | 422 | Pengajuan tidak memenuhi syarat (alasan di `rule_trace`) |
| `LON_EXCEEDS_PLAFOND` | 422 | Nominal melebihi plafon Rp {plafon} |
| `LON_TENOR_OUT_OF_RANGE` | 422 | Tenor harus {min}–{max} bulan |
| `LON_INVALID_STATE` | 409 | Tindakan tidak dapat dilakukan pada status {status} |
| `LON_ALREADY_DISBURSED` | 409 | Pinjaman sudah dicairkan |
| `LON_OVERPAYMENT` | 422 | Pembayaran melebihi sisa kewajiban (kelebihan dapat dititipkan) |
| `LON_CONTRACT_NOT_SIGNED` | 409 | Akad belum diunggah/ditandatangani |

### 4.4 POS, Inventori & Sinkronisasi

| Kode | HTTP | Pesan pengguna (ringkas) |
|---|---|---|
| `POS_SHIFT_NOT_OPEN` | 409 | Buka shift terlebih dahulu |
| `POS_SHIFT_ALREADY_OPEN` | 409 | Perangkat ini sudah memiliki shift terbuka |
| `POS_SHIFT_UNSYNCED` | 409 | Masih ada {n} transaksi belum tersinkron |
| `POS_DEVICE_NOT_REGISTERED` | 401 | Perangkat belum terdaftar |
| `POS_DEVICE_LOCKED` | 423 | Perangkat terkunci: belum sinkron lebih dari {jam} jam |
| `POS_PRODUCT_NOT_FOUND` | 404 | Barcode tidak dikenali |
| `POS_PRICE_NOT_SET` | 422 | Produk belum memiliki harga |
| `POS_PAYMENT_INCOMPLETE` | 422 | Pembayaran belum lunas |
| `POS_CREDIT_LIMIT_EXCEEDED` | 422 | Melebihi sisa limit kredit Rp {sisa} |
| `POS_OFFLINE_LIMIT_EXCEEDED` | 422 | Melebihi batas transaksi non-tunai saat offline Rp {batas} |
| `POS_SUPERVISOR_REQUIRED` | 403 | Butuh otorisasi supervisor |
| `POS_RETURN_EXCEEDS_SOLD` | 422 | Jumlah retur melebihi jumlah terjual |
| `POS_VOID_NOT_ALLOWED` | 409 | Void hanya untuk transaksi pada shift yang sama |
| `INV_INSUFFICIENT_STOCK` | 422 | Stok tidak cukup (bila stok negatif tidak diizinkan) |
| `INV_UOM_CONVERSION_MISSING` | 422 | Konversi satuan belum diatur |
| `INV_OPNAME_IN_PROGRESS` | 409 | Gudang sedang stok opname |
| `SYNC_BATCH_TOO_LARGE` | 413 | Batch melebihi 100 transaksi |
| `SYNC_BUNDLE_SIGNATURE_INVALID` | 422 | Paket aturan perangkat tidak valid, sinkron ulang |
| `SYNC_CLOCK_SKEW` | 422 | Jam perangkat berbeda lebih dari 10 menit dari server |

### 4.5 Akuntansi, Posting, Rule, Workflow, SHU

| Kode | HTTP | Pesan pengguna (ringkas) |
|---|---|---|
| `ACC_UNBALANCED` | 422 | Total debit dan kredit tidak sama (selisih Rp {n}) |
| `ACC_ACCOUNT_NOT_POSTABLE` | 422 | Akun {kode} adalah akun induk/nonaktif |
| `ACC_CONTROL_ACCOUNT_MANUAL` | 422 | Akun buku pembantu tidak dapat dipakai di jurnal manual |
| `ACC_PERIOD_CLOSED` | 409 | Periode {bulan} sudah ditutup |
| `ACC_PERIOD_SOFT_CLOSED` | 409 | Periode dalam tutup sementara; hanya jurnal penyesuaian |
| `ACC_JOURNAL_IMMUTABLE` | 409 | Jurnal yang sudah diposting tidak dapat diubah; gunakan jurnal pembalik |
| `ACC_CLOSE_CHECK_FAILED` | 409 | Checklist tutup buku belum lengkap (daftar di `context`) |
| `ACC_ACCOUNT_IN_USE` | 409 | Akun sudah memiliki mutasi, hanya dapat dinonaktifkan |
| `PST_TEMPLATE_NOT_FOUND` | — (internal) | Template posting tidak ada untuk event → antrean posting gagal |
| `PST_ROLE_UNMAPPED` | — (internal) | Account role {role} belum dipetakan |
| `PST_UNBALANCED` | — (internal) | Hasil template tidak seimbang |
| `RUL_BLOCKED` | 422 | Diblokir aturan {rule}: {pesan} |
| `RUL_EVALUATION_TIMEOUT` | 503 / tahan | Evaluasi aturan melebihi batas waktu (perilaku fail-safe dok. 03 §12) |
| `RUL_VALIDATION_FAILED` | 422 | Aturan tidak valid (detail per node di `errors[]`) |
| `RUL_TEST_FAILED` | 422 | {n} test case gagal |
| `RUL_EFFECTIVE_OVERLAP` | 409 | Tanggal berlaku bertumpang tindih dengan versi lain |
| `RUL_CYCLE_DETECTED` | 422 | Rantai aturan membentuk siklus |
| `WF_APPROVAL_REQUIRED` | 202 | Transaksi menunggu persetujuan |
| `WF_ALREADY_DECIDED` | 409 | Tugas persetujuan sudah diputuskan |
| `WF_NOT_ASSIGNEE` | 403 | Anda bukan penyetuju langkah ini |
| `SHU_ALLOCATION_NOT_100` | 422 | Total alokasi SHU harus 100% |
| `SHU_YEAR_NOT_CLOSED` | 409 | Tahun buku belum ditutup |
| `SHU_ALREADY_DISTRIBUTED` | 409 | SHU tahun ini sudah dibagikan |

### 4.6 Integrasi & Impor

| Kode | HTTP | Pesan pengguna (ringkas) |
|---|---|---|
| `PAY_PROVIDER_UNAVAILABLE` | 503 | Layanan pembayaran sedang gangguan; gunakan metode lain |
| `PAY_TIMEOUT` | 504 | Status pembayaran belum diketahui; sedang diperiksa otomatis |
| `PAY_WEBHOOK_SIGNATURE_INVALID` | 401 | (internal, tidak ditampilkan ke pengguna) |
| `PAY_AMOUNT_MISMATCH` | 422 | Nominal pembayaran tidak sesuai tagihan |
| `PAY_EXPIRED` | 409 | QR/VA sudah kedaluwarsa, buat ulang |
| `NTF_PROVIDER_FAILED` | — (internal) | Pengiriman gagal; dicoba kanal cadangan |
| `IMP_FILE_UNREADABLE` | 422 | File tidak dapat dibaca (gunakan template .xlsx/.csv) |
| `IMP_ROWS_INVALID` | 422 | {n} baris tidak valid (unduh laporan error) |
| `IMP_OPENING_BALANCE_MISMATCH` | 422 | Saldo awal tidak seimbang / buku pembantu tidak cocok |

## 5. Idempotensi

| Aspek | Aturan |
|---|---|
| Cakupan | Semua `POST`/`PATCH` yang mengubah uang, stok, poin, status pinjaman/anggota, dan semua endpoint sinkron POS |
| Header | `Idempotency-Key: <uuid>` dibuat klien **saat form dibuka** (bukan saat klik), sehingga klik ganda dan retry jaringan memakai kunci yang sama |
| Penyimpanan | Tabel `idempotency_keys(tenant_id, key, request_hash, status, response, created_at)`; retensi 7 hari. Untuk transaksi keuangan, keunikan permanen dijamin kolom `client_txn_id` |
| Kunci sama + isi sama + selesai | Kembalikan respons asli (status & body sama) |
| Kunci sama + isi berbeda | `409 SYS_IDEMPOTENCY_MISMATCH` |
| Kunci sama + masih diproses | `409 SYS_REQUEST_IN_PROGRESS` + `Retry-After: 2` |
| Event konsumen | `processed_events(consumer, event_id)` dalam transaksi yang sama dengan efeknya |
| Jurnal otomatis | `journal_entries.idempotency_key = event_id` (unik) |
| Webhook pembayaran | Unik per `(provider, external_id, status)` |

## 6. Konkurensi

| Situasi | Strategi |
|---|---|
| Ubah data master (anggota, produk, rule draft) | *Optimistic locking*: kolom `version`, header `If-Match` (ETag). Tidak cocok → `412`/`409 SYS_VERSION_CONFLICT`; UI menampilkan perbandingan & tombol muat ulang |
| Saldo simpanan, stok, poin, sisa pinjaman | *Pessimistic lock* `SELECT … FOR UPDATE` pada baris saldo; **urutan kunci tetap** (berdasarkan `id` naik) untuk mencegah deadlock |
| Deadlock / serialization failure | Retry otomatis maks. 3× dengan jitter 20–200 ms; gagal → `500 SYS_INTERNAL` (retryable) |
| Penomoran dokumen | `UPDATE document_sequences … RETURNING` dalam transaksi; nomor boleh melompat bila transaksi gagal (dicatat), tidak boleh ganda |
| Dua kasir menjual stok terakhir | Server otoritatif; penjualan kedua tetap diterima bila kebijakan stok negatif diizinkan (default untuk penjualan offline) dan masuk antrean pengecualian; bila tidak diizinkan dan online → `INV_INSUFFICIENT_STOCK` |
| Job akhir bulan berjalan saat transaksi masuk | Job memproses berdasarkan `value_date ≤ cut-off` dan *snapshot*; transaksi setelah cut-off masuk periode berikutnya |
| Tutup buku saat ada transaksi berjalan | Status `SOFT_CLOSED` diset dulu (menolak transaksi operasional baru ke periode itu), tunggu transaksi aktif selesai, baru checklist dijalankan |

## 7. Retry, Timeout & Circuit Breaker

| Komponen | Timeout | Retry | Setelah gagal |
|---|---|---|---|
| Panggilan API dari web/app | 15 s | GET: 2× otomatis; POST: hanya bila `retryable` & dengan idempotency key yang sama | Tampilkan error + tombol "Coba lagi" |
| Sinkron POS | 30 s per batch | Backoff 5 s, 15 s, 60 s, lalu tiap 5 menit tanpa batas | Indikator "antrean sinkron: n"; alarm ke supervisor bila > 2 jam |
| Worker job (posting, akrual, notifikasi) | 60 s per job | 5× backoff eksponensial (1 m, 5 m, 15 m, 1 j, 6 j) | *Dead letter queue* + notifikasi admin + UI "Job gagal" dengan tombol jalankan ulang |
| Job massal per tenant (akhir bulan) | — | Diproses per tenant & per batch 500 entitas dengan *checkpoint*; dapat dilanjutkan | Kegagalan satu tenant tidak menghentikan tenant lain |
| Payment gateway | Connect 3 s, total 10 s | 2× untuk GET status; **tidak** untuk create charge (pakai idempotency key penyedia) | Circuit breaker terbuka setelah 50% gagal dalam 1 menit (min. 20 panggilan); tutup percobaan tiap 30 s |
| WhatsApp/SMS/email | 10 s | 3× | Kanal cadangan (WA → SMS → email); status `GAGAL` tercatat |
| Object storage | 10 s | 3× | Upload ditandai gagal; transaksi bisnis tidak tergantung pada upload lampiran |

## 8. Penanganan per Area Kritis

### 8.1 Posting Engine

| Penyebab | Kode | Perilaku |
|---|---|---|
| Template tidak ada / role belum dipetakan / hasil tidak seimbang | `PST_*` | Event masuk **Antrean Posting Gagal**; transaksi operasional **tetap sah**; notifikasi akuntan; tutup buku diblokir sampai antrean kosong |
| Periode tertutup | `ACC_PERIOD_CLOSED` | Ikuti rule `period.late_transaction.policy` |
| Error transien (DB) | — | Retry otomatis (§7) |

> Untuk transaksi teller/kas yang diposting **sinkron**, kegagalan posting karena konfigurasi (role belum dipetakan)
> **membatalkan** transaksi dan menampilkan pesan "Hubungi akuntan: pemetaan akun belum lengkap". Ini mencegah
> transaksi kas tanpa jurnal di jalur online. Sinkron POS & job massal memakai antrean (tidak membatalkan).

### 8.2 POS Offline

| Skenario | Perilaku yang diharapkan |
|---|---|
| Koneksi putus di tengah pembayaran QRIS dinamis | Status "menunggu konfirmasi"; kasir dapat cek status manual, ganti metode, atau tahan transaksi. Bila pembayaran ternyata berhasil setelah transaksi diganti metode → rekonsiliasi membuat refund/titipan dan pengecualian |
| Unggah batch terputus | Ulang dengan `client_txn_id` sama; server mengembalikan hasil transaksi yang sudah diterima (duplikat diabaikan) |
| Saldo sukarela anggota di server kurang dari yang dipakai offline | Transaksi diterima; kekurangan → `receivables` (piutang anggota) + pengecualian `SALDO_KURANG` |
| Poin dipakai di dua perangkat bersamaan | Server menerima keduanya; poin negatif dikonversi menjadi piutang senilai poin + pengecualian `POIN_KURANG` |
| Anggota keluar/pasif saat transaksi offline | Transaksi diterima dengan harga yang tercatat; pengecualian `ANGGOTA_NONAKTIF` untuk ditinjau |
| Bundle rule perangkat kedaluwarsa (> versi aktif) | Transaksi diterima sesuai bundle lama (harga yang sudah dibayar pembeli tidak diubah); pengecualian `BUNDLE_KADALUARSA` bila selisih material |
| Jam perangkat salah | Perangkat menyimpan `device_time` + `monotonic_offset`; server memakai waktu terima bila selisih > 10 menit dan menandai `SYNC_CLOCK_SKEW` |
| IndexedDB penuh / rusak | Perangkat berhenti menerima transaksi baru dengan pesan jelas; ekspor darurat transaksi tertunda ke file terenkripsi |
| Printer gagal | Transaksi tetap tersimpan; tombol "cetak ulang"; struk digital WA sebagai alternatif |
| Perangkat hilang | Admin mencabut perangkat; token perangkat ditolak (`POS_DEVICE_NOT_REGISTERED`); transaksi yang belum sinkron dianggap hilang dan dilaporkan sebagai selisih kas shift |

### 8.3 Pembayaran Digital (Webhook)

| Skenario | Perilaku |
|---|---|
| Webhook duplikat | Diabaikan (idempoten), respons `200` |
| Webhook datang sebelum transaksi tercatat (race) | Disimpan di `payment_gateway_txns` status `UNMATCHED`; dicocokkan ulang tiap menit selama 1 jam |
| Signature tidak valid | `401`, tidak diproses, alert keamanan bila berulang |
| Nominal berbeda | Status `AMOUNT_MISMATCH`, tidak diposting otomatis, masuk antrean rekonsiliasi |
| Webhook tidak pernah datang | *Poller* memeriksa status transaksi `PENDING` > 5 menit; VA/QR kedaluwarsa ditutup |
| Pembayaran untuk VA anggota tanpa tagihan | Dikreditkan ke titipan anggota + notifikasi admin |

### 8.4 Impor Data

- Selalu **validasi penuh dulu (dry run)**, lalu proses dalam satu transaksi per jenis data; gagal sebagian = tidak ada yang tersimpan.
- Laporan error per baris (nomor baris, kolom, nilai, alasan) dapat diunduh sebagai Excel.
- File > 50.000 baris diproses sebagai job asinkron dengan progres.

## 9. Penanganan Error di Antarmuka (Web, POS, Mobile)

| Jenis | Pola UI | Contoh |
|---|---|---|
| Validasi field | Pesan inline di bawah field, fokus ke field pertama yang salah, ringkasan di atas form bila > 3 error | "NIK harus 16 digit angka" |
| Aturan bisnis | Dialog/banner dengan penjelasan + tindakan yang disarankan + "Lihat aturan" | "Saldo setelah penarikan minimal Rp 50.000. Maksimal yang dapat ditarik Rp 70.000. [Tarik Rp 70.000]" |
| Butuh persetujuan | Dialog PIN supervisor (POS) atau info "dikirim untuk persetujuan" + tautan status | — |
| Konflik versi | Dialog perbandingan "Versi Anda vs versi terbaru" | — |
| Jaringan (online app) | Toast tidak menghalangi + tombol coba lagi; form tidak hilang | — |
| Sesi berakhir | Modal login ulang **tanpa** kehilangan isi form (draf disimpan lokal) | — |
| Error sistem | Halaman/dialog ramah + kode ref. + tombol salin & hubungi dukungan | "Terjadi kesalahan. Kode ref. 7f3a9c2e1b" |
| Halaman crash | *Error boundary* per area (bukan seluruh aplikasi), laporan otomatis ke Sentry | — |
| Offline (POS) | Banner tetap "OFFLINE — transaksi disimpan & akan disinkron", tidak memakai dialog | — |

**Aturan tambahan UI:**

1. Tombol aksi finansial dinonaktifkan dan menampilkan *spinner* setelah diklik sampai respons diterima (cegah klik ganda; idempotency key tetap jadi pengaman utama).
2. Tidak ada *optimistic update* untuk transaksi keuangan; UI menunggu konfirmasi server (kecuali POS offline yang menyimpan lokal dahulu).
3. Penjaga "perubahan belum disimpan" pada form panjang (pendaftaran anggota, jurnal, rule).
4. Setiap halaman daftar punya 4 status: **memuat** (skeleton), **kosong** (ilustrasi + tindakan pertama), **error** (pesan + coba lagi), **berisi**.
5. Pesan error mengikuti pola: **apa yang terjadi → mengapa → apa yang bisa dilakukan**. Hindari kata "gagal" tanpa penjelasan, jargon teknis, dan menyalahkan pengguna.

## 10. Logging, Monitoring & Alert

| Aspek | Aturan |
|---|---|
| Format log | JSON terstruktur: `timestamp`, `level`, `trace_id`, `tenant_id`, `user_id`, `module`, `code`, `message`, `duration_ms` |
| PII | NIK, No. KK, no. rekening, token, kata sandi, OTP **tidak pernah** dicatat; nomor HP dimasking (`0812****7788`) |
| Error 5xx | Dikirim ke Sentry dengan tag tenant (tanpa PII) |
| Metrik bisnis | `posting_failed_queue_size`, `sync_pending_transactions`, `sync_exceptions_open`, `rule_eval_errors`, `job_dlq_size`, `payment_unmatched` per tenant |
| Alert on-call | 5xx > 1% selama 5 menit; antrean posting gagal > 0 selama 24 jam; DLQ > 0; circuit breaker terbuka; lag replikasi > 60 s; disk DB > 80% |
| Audit vs log | Audit trail (dok. 06 `audit_logs`) adalah catatan bisnis permanen; log aplikasi untuk diagnosis dengan retensi 30–90 hari |

## 11. Katalog Edge Case per Domain

| # | Domain | Edge case | Perilaku yang diharapkan |
|---|---|---|---|
| EC-01 | Anggota | NIK sama mendaftar di koperasi lain | Diizinkan (identitas per tenant); tidak ada pengecekan lintas tenant |
| EC-02 | Anggota | Calon anggota mendaftar ulang setelah ditolak | Data lama dipakai ulang dengan status baru; riwayat tetap |
| EC-03 | Anggota | Anggota meninggal dengan pinjaman aktif | Proses keluar via ahli waris; sisa kewajiban sesuai keputusan pengurus (kompensasi/tagih/hapus buku) |
| EC-04 | Anggota | Anggota keluar lalu masuk kembali | Nomor anggota baru (default) atau lama (rule); simpanan pokok disetor ulang |
| EC-05 | Anggota | Kartu hilang | Token QR dirotasi; kartu lama ditolak `MBR_CARD_INVALID` |
| EC-06 | Simpanan | Setoran wajib melebihi tagihan | Kelebihan dialokasikan ke tagihan berikutnya (default) atau ke sukarela (rule) |
| EC-07 | Simpanan | Penarikan saat rekening punya `hold_amount` | Saldo tersedia = saldo − hold |
| EC-08 | Simpanan | Pembalikan setoran yang sudah dipakai untuk penarikan | Ditolak bila saldo jadi negatif; harus koreksi berurutan |
| EC-09 | Simpanan | Jasa simpanan bernilai < Rp 1 | Dibulatkan ke bawah; akumulasi pecahan tidak dibawa (default) |
| EC-10 | Simpanan | Potongan gaji untuk anggota yang sudah keluar | Masuk titipan + daftar selisih untuk dikembalikan ke instansi |
| EC-11 | Pinjaman | Jatuh tempo tanggal 31 di bulan 30 hari | Pakai hari terakhir bulan; Februari = 28/29 |
| EC-12 | Pinjaman | Jatuh tempo pada hari libur | Rule: tetap / mundur ke hari kerja berikutnya tanpa denda |
| EC-13 | Pinjaman | Pembayaran sebelum pencairan selesai diposting | Ditolak `LON_INVALID_STATE` |
| EC-14 | Pinjaman | Bayar lebih dari total sisa | Kelebihan ke titipan anggota; pinjaman lunas |
| EC-15 | Pinjaman | Pembayaran dibalik setelah status LUNAS | Status kembali AKTIF; jadwal & DPD dihitung ulang |
| EC-16 | Pinjaman | Rule plafon berubah saat pengajuan sedang ditinjau | Pengajuan tetap memakai snapshot versi saat diajukan |
| EC-17 | Pinjaman | Penyetuju adalah keluarga/pemohon sendiri | Diblokir constraint workflow; otomatis dialihkan ke penyetuju lain |
| EC-18 | POS | Harga berubah saat item sudah di keranjang | Harga di keranjang tetap sampai transaksi selesai |
| EC-19 | POS | Retur barang yang dibeli dengan poin & tunai | Pengembalian proporsional per metode; poin dikembalikan sebagai poin baru (masa berlaku baru default) |
| EC-20 | POS | Retur oleh anggota yang kini berstatus non-anggota | Akun retur mengikuti `buyer_type` transaksi asli |
| EC-21 | POS | Barang dengan konversi satuan (jual per pcs, stok per dus) | Stok selalu dalam satuan dasar; pecahan diperbolehkan sesuai pengaturan produk |
| EC-22 | POS | Transaksi bernilai Rp 0 (100% diskon/promo) | Diizinkan dengan otorisasi supervisor; tetap mengurangi stok & membentuk HPP |
| EC-23 | POS | Shift lupa ditutup lewat tengah malam | Shift tetap milik kasir; tanggal akuntansi transaksi = tanggal transaksi; peringatan ke supervisor |
| EC-24 | POS | Sinkron transaksi bulan lalu setelah periode ditutup | Rule `period.late_transaction.policy` |
| EC-25 | Inventori | Stok negatif akibat penjualan offline | Diizinkan; biaya rata-rata memakai biaya terakhir; dikoreksi saat penerimaan berikutnya; laporan stok negatif |
| EC-26 | Inventori | GRN lebih dari PO | Diizinkan dengan toleransi (rule) dan persetujuan |
| EC-27 | Inventori | Opname saat ada transaksi berjalan | Snapshot qty sistem saat hitung dimulai; transaksi selama opname dicatat terpisah dan diperhitungkan |
| EC-28 | Akuntansi | Akun dinonaktifkan tapi masih dipetakan ke role | Publikasi pemetaan/penonaktifan ditolak sampai role dipindah |
| EC-29 | Akuntansi | Jurnal pembalik untuk periode tertutup | Pembalik diposting di periode berjalan dengan referensi ke jurnal asli |
| EC-30 | Akuntansi | Selisih pembulatan akibat alokasi | Diposting ke akun `SELISIH_PEMBULATAN` (810105), tidak pernah dibuang |
| EC-31 | Akuntansi | Membuka kembali periode tertutup | Hanya Bendahara + Ketua, wajib alasan, tercatat audit, snapshot laporan lama tetap tersimpan |
| EC-32 | Akuntansi | Tahun buku pertama dimulai di tengah tahun | Saldo awal per tanggal mulai; periode sebelum mulai berstatus `CLOSED` tanpa transaksi |
| EC-33 | SHU | SHU negatif (rugi) | Tidak ada pembagian; kebijakan penutupan rugi mengikuti keputusan RAT (cadangan / dibawa ke tahun berikut) |
| EC-34 | SHU | Anggota keluar di tengah tahun | Berhak SHU sesuai jasa selama menjadi anggota (default); dapat diubah rule |
| EC-35 | SHU | Total pembagian per anggota ≠ alokasi karena pembulatan | Selisih ke akun pembulatan / dana cadangan (rule) |
| EC-36 | Rule | Dua rule diskon berlaku bersamaan | Strategi gabung hook (default `BEST_FOR_CUSTOMER`) |
| EC-37 | Rule | Rule merujuk parameter yang dinonaktifkan | Validasi publikasi menolak; rule aktif yang terdampak memakai fallback + alert |
| EC-38 | Rule | Custom field dihapus padahal dipakai rule | Penghapusan ditolak sampai rule diperbarui |
| EC-39 | Platform | Tenant disuspend saat POS offline | POS tetap jalan sampai sinkron berikutnya; setelah itu terkunci dengan pesan |
| EC-40 | Platform | Pengguna di 2 tenant membuka 2 tab | Token per tab mengikat tenant; tab tidak boleh saling menimpa konteks tenant |
| EC-41 | Waktu | Tenant WIT melakukan transaksi 23:30 | Tanggal akuntansi mengikuti zona waktu tenant, bukan UTC/server |
| EC-42 | Uang | Nominal sangat besar (≥ Rp 1 triliun) | Didukung `NUMERIC(20,2)`; UI memformat tanpa notasi ilmiah |

## 12. Kriteria Penerimaan untuk Setiap Fitur

Sebelum fitur dianggap selesai (melengkapi DoD di dokumen 08):

- [ ] Semua jalur error memakai kode dari katalog §4 (atau kode baru yang ditambahkan ke katalog)
- [ ] Ada tes untuk setiap kode error yang dapat dihasilkan endpoint tersebut
- [ ] Edge case relevan dari §11 memiliki tes otomatis
- [ ] Endpoint yang mengubah uang/stok/poin teruji idempoten (permintaan ganda → satu efek)
- [ ] UI menampilkan status memuat, kosong, error, dan berhasil

[← 08 Task Todo](./08-task-todo.md) · [Lanjut ke 10 — Spesifikasi Teknis Detail →](./10-spesifikasi-teknis-detail.md)
