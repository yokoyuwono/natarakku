# 05 — Flowchart

[← Kembali ke daftar isi](./README.md)

---

Diagram arsitektur tingkat tinggi ada di [01 §8.1](./01-ringkasan-analisa-dan-tech-stack.md#81-gaya-arsitektur);
siklus hidup rule (state diagram) dan urutan evaluasi rule ada di [03 §9 & §11](./03-rule-engine.md#9-siklus-hidup-rule).

| No | Alur | Modul |
|---|---|---|
| 5.1 | Onboarding tenant | PLT |
| 5.2 | Pendaftaran & aktivasi anggota | MBR, SAV, RUL |
| 5.3 | Setoran & penarikan simpanan | SAV, RUL, ACC |
| 5.4 | Siklus hidup pinjaman | LON, RUL, ACC |
| 5.5 | Transaksi POS end-to-end | POS, MBR, LOY, SAV, ACC |
| 5.6 | Shift kasir | POS, ACC |
| 5.7 | Sinkronisasi POS offline | POS, SYNC |
| 5.8 | Posting Engine (event → jurnal) | ACC, RUL |
| 5.9 | Potong gaji bulanan | SAV, LON, POS, INT |
| 5.10 | Pembelian sampai pembayaran supplier | INV, ACC |
| 5.11 | Tutup buku bulanan | ACC |
| 5.12 | Tutup buku tahunan, SHU & RAT | ACC, SHU |
| 5.13 | Pembuatan & publikasi rule | RUL |
| 5.14 | Anggota keluar | MBR, SAV, LON, ACC |

---

## 5.1 Onboarding Tenant

```mermaid
flowchart TD
    A(["Calon koperasi daftar / Super Admin buat tenant"]) --> B["Isi profil badan hukum<br/>nama, no. akta/SK, NIK koperasi, NPWP"]
    B --> C{"Jenis koperasi?"}
    C -->|Konsumen/Karyawan| D1["Template COA Konsumen<br/>+ Rule Pack Konsumen"]
    C -->|KSP/USP| D2["Template COA Simpan Pinjam<br/>+ Rule Pack KSP"]
    C -->|Serba Usaha| D3["Template COA KSU<br/>+ Rule Pack KSU"]
    C -->|Kopdes Merah Putih| D4["Template COA Kopdes<br/>+ Rule Pack Kopdes"]
    D1 --> E["Pilih paket langganan & aktifkan feature flag"]
    D2 --> E
    D3 --> E
    D4 --> E
    E --> F["Sistem membuat tenant:<br/>RLS context, peran bawaan, tahun buku & periode,<br/>COA, account role default, rule pack versi aktif"]
    F --> G["Atur unit usaha, cabang, gerai, gudang"]
    G --> H["Konfirmasi rule AD/ART wajib<br/>simpanan pokok/wajib, syarat anggota, alokasi SHU"]
    H --> I{"Migrasi dari sistem lama?"}
    I -->|Ya| J["Impor data: anggota, saldo simpanan,<br/>pinjaman berjalan, saldo awal akun, produk & stok"]
    J --> K{"Dry run valid?<br/>neraca seimbang & buku pembantu cocok"}
    K -->|Tidak| J
    K -->|Ya| L["Posting jurnal saldo awal"]
    I -->|Tidak| M["Saldo awal nol / input manual"]
    L --> N["Undang pengguna & tetapkan peran"]
    M --> N
    N --> O["Daftarkan perangkat POS & printer"]
    O --> P(["Tenant aktif — transaksi pertama"])
```

## 5.2 Pendaftaran & Aktivasi Anggota

```mermaid
flowchart TD
    A(["Calon anggota datang / daftar via petugas"]) --> B["Input data: NIK, KK, identitas, pekerjaan,<br/>kelompok, penghasilan, ahli waris, foto KTP, tanda tangan"]
    B --> C["Catat persetujuan pemrosesan data pribadi"]
    C --> D{"NIK valid & belum terdaftar<br/>di tenant ini?"}
    D -->|Tidak| D1["Tampilkan anggota yang sudah ada / perbaiki data"]
    D1 --> B
    D -->|Ya| E["Evaluasi rule member.before_register"]
    E --> F{"Lolos?"}
    F -->|Diblokir| F1(["Tolak dengan alasan"])
    F -->|Perlu persetujuan| G
    F -->|Lolos| G["Status: CALON ANGGOTA<br/>buat rekening simpanan pokok & wajib"]
    G --> H["Tagih simpanan pokok<br/>nominal dari rule saving.required_amount"]
    H --> I["Pembayaran simpanan pokok<br/>teller / transfer / VA"]
    I --> J["Jurnal: D Kas / K Simpanan Pokok"]
    J --> K["Evaluasi rule member.activation_requirements"]
    K --> L{"Semua syarat terpenuhi?"}
    L -->|Belum| L1["Tampilkan syarat yang belum dipenuhi"]
    L1 --> I
    L -->|Ya| M["Ajukan persetujuan pengurus"]
    M --> N{"Disetujui?"}
    N -->|Tidak| N1["Status DITOLAK<br/>proses pengembalian simpanan pokok"]
    N -->|Ya| O["Status: AKTIF<br/>terbitkan nomor anggota & kartu QR"]
    O --> P["Event member.activated<br/>→ rule member.after_activated: WA selamat datang, poin sambutan"]
    P --> Q(["Anggota aktif"])
```

## 5.3 Setoran & Penarikan Simpanan

```mermaid
flowchart TD
    A(["Teller pilih anggota<br/>scan QR / cari nomor"]) --> B["Tampil profil, rekening, tagihan wajib tertunggak"]
    B --> C{"Jenis transaksi?"}
    C -->|Setoran| D["Pilih rekening & nominal<br/>wajib: alokasikan ke tagihan tertua"]
    C -->|Penarikan| E["Pilih rekening sukarela & nominal"]
    D --> F["Evaluasi saving.before_deposit"]
    E --> G["Evaluasi saving.before_withdraw<br/>saldo minimal, limit harian, rekening dibekukan"]
    F --> H{"Hasil rule?"}
    G --> H
    H -->|block| H1(["Tampilkan alasan, transaksi batal"])
    H -->|require_approval| H2["Minta PIN/persetujuan supervisor"]
    H2 --> H3{"Disetujui?"}
    H3 -->|Tidak| H1
    H3 -->|Ya| I
    H -->|lolos / warn dikonfirmasi| I["Simpan transaksi simpanan<br/>+ update saldo buku pembantu"]
    I --> J["Posting Engine: jurnal otomatis<br/>dalam transaksi DB yang sama"]
    J --> K["Update posisi kas teller"]
    K --> L["Cetak slip / kirim bukti WA"]
    L --> M["Event saving.deposited / withdrawn<br/>→ rule ASYNC: notifikasi, poin"]
    M --> N(["Selesai"])
```

## 5.4 Siklus Hidup Pinjaman

```mermaid
flowchart TD
    A(["Anggota ajukan pinjaman<br/>teller atau aplikasi anggota"]) --> B["Pilih produk, nominal, tenor, tujuan<br/>simulasi jadwal angsuran"]
    B --> C["Evaluasi loan.eligibility"]
    C --> D{"Layak?"}
    D -->|Tidak| D1(["Tolak + jejak alasan rule"])
    D -->|Ya| E["Hitung plafon loan.plafond<br/>tarif loan.rate & biaya loan.fees"]
    E --> F{"Nominal ≤ plafon?"}
    F -->|Tidak| F1["Sarankan nominal = plafon"]
    F1 --> B
    F -->|Ya| G["Unggah dokumen & data agunan<br/>simpan snapshot versi rule"]
    G --> H["Tentukan rute persetujuan<br/>loan.approval_route"]
    H --> I["Langkah persetujuan<br/>serial / paralel, SLA, eskalasi"]
    I --> J{"Keputusan?"}
    J -->|Ditolak| J1(["Status DITOLAK + notifikasi"])
    J -->|Revisi| B
    J -->|Disetujui| K["Generate akad PDF dari template"]
    K --> L["Tanda tangan akad & unggah"]
    L --> M["Pencairan maker-checker<br/>tunai / transfer / ke simpanan sukarela"]
    M --> N["Jurnal L1 + jadwal angsuran aktif"]
    N --> O["Job harian: hitung DPD, denda loan.penalty.daily,<br/>kolektibilitas loan.collectibility.classify"]
    O --> P["Pengingat H-3 & hari jatuh tempo via WA"]
    P --> Q["Pembayaran angsuran<br/>teller / potong gaji / autodebet / VA / kolektor"]
    Q --> R["Alokasi: denda → jasa → pokok<br/>jurnal L3"]
    R --> S{"Lunas?"}
    S -->|Belum| O
    S -->|Ya| T(["Status LUNAS<br/>kembalikan agunan"])
    O --> U{"DPD melewati ambang macet?"}
    U -->|Ya| V{"Tindakan?"}
    V -->|Restrukturisasi| W["Jadwal ulang dengan persetujuan"]
    W --> O
    V -->|Hapus buku| X["Jurnal L7 hapus buku<br/>tetap ditagih, pemulihan L8"]
    U -->|Tidak| O
    O --> Y["Job akhir bulan: CKPN loan.impairment.rate<br/>jurnal L5/L6"]
```

## 5.5 Transaksi POS End-to-End

```mermaid
flowchart TD
    A(["Kasir memulai transaksi<br/>shift harus terbuka"]) --> B{"Pembeli anggota?"}
    B -->|Ya| C["Scan QR kartu / input no. anggota"]
    C --> C1{"Anggota aktif?"}
    C1 -->|Ya| C2["Muat: tier, poin, saldo sukarela,<br/>sisa limit kredit, batas offline"]
    C1 -->|Tidak| C3["Peringatan status — pakai harga umum"]
    B -->|Tidak| D["Pembeli umum / pelanggan terdaftar"]
    C2 --> E
    C3 --> E
    D --> E["Scan / cari produk, atur qty & satuan"]
    E --> F["pos.price.resolve per item<br/>harga anggota / umum / grosir"]
    F --> G["pos.discount.apply<br/>promo, diskon tier, strategi gabung"]
    G --> H["Tampilkan total & estimasi poin"]
    H --> I{"Tambah item?"}
    I -->|Ya| E
    I -->|Tidak| J["pos.before_checkout<br/>kuota, larangan, otorisasi diskon"]
    J --> K{"Lolos?"}
    K -->|Diblokir| K1["Tampilkan alasan, ubah keranjang"]
    K1 --> E
    K -->|Lolos| L["Pilih pembayaran — boleh split"]
    L --> M{"Metode"}
    M -->|Tunai| M1["Input uang diterima, hitung kembalian"]
    M -->|QRIS| M2["QR dinamis via PG<br/>tunggu callback; offline: QR statis + verifikasi manual"]
    M -->|EDC| M3["Input ref approval EDC"]
    M -->|Saldo sukarela| M4["Cek saldo online atau batas offline"]
    M -->|Kredit anggota| M5["Cek pos.credit_limit"]
    M -->|Poin| M6["Konversi poin ke Rupiah<br/>loyalty.points.redeem_value"]
    M1 --> N{"Total terbayar?"}
    M2 --> N
    M3 --> N
    M4 --> N
    M5 --> N
    M6 --> N
    N -->|Belum| L
    N -->|Ya| O["Simpan transaksi lokal<br/>client_txn_id UUIDv7 + nomor struk perangkat"]
    O --> P["Cetak struk / kirim struk WA"]
    P --> Q["Antrekan ke outbox perangkat"]
    Q --> R{"Online?"}
    R -->|Ya| S["Upload ke Sync Gateway"]
    R -->|Tidak| S1["Tunggu koneksi — lihat 5.7"]
    S1 --> S
    S --> T["Server: kurangi stok & hitung HPP,<br/>debet saldo / buat piutang / kurangi poin"]
    T --> U["Posting Engine: jurnal P1–P4<br/>per transaksi atau ringkas per shift"]
    U --> V["Rule ASYNC pos.after_checkout<br/>poin, notifikasi"]
    V --> W(["Selesai"])
```

## 5.6 Shift Kasir

```mermaid
flowchart TD
    A(["Kasir login di perangkat terdaftar"]) --> B["Buka shift: input modal awal kas"]
    B --> C["Transaksi penjualan, retur, void<br/>void & retur butuh PIN supervisor"]
    C --> D{"Akhir shift?"}
    D -->|Tidak| C
    D -->|Ya| E["Pastikan semua transaksi tersinkron"]
    E --> F["Hitung fisik uang per pecahan"]
    F --> G["Sistem hitung kas seharusnya =<br/>modal awal + tunai masuk - kembalian - refund tunai"]
    G --> H{"Ada selisih?"}
    H -->|Tidak| J
    H -->|Ya| I["Rule pos.cash_difference.policy<br/>beban / piutang kasir / perlu persetujuan"]
    I --> I1{"Perlu persetujuan?"}
    I1 -->|Ya| I2["Supervisor meninjau & menyetujui"]
    I2 --> J
    I1 -->|Tidak| J["Tutup shift — jurnal selisih P9/P10"]
    J --> K["Cetak laporan shift: per metode bayar,<br/>anggota vs non-anggota"]
    K --> L["Setor kas ke kas besar / bank<br/>jurnal P11 kirim"]
    L --> M["Penerima konfirmasi setoran<br/>jurnal P11 terima"]
    M --> N(["Shift selesai"])
```

## 5.7 Sinkronisasi POS Offline

```mermaid
sequenceDiagram
    autonumber
    participant K as Kasir
    participant D as POS Device - IndexedDB
    participant G as Sync Gateway
    participant S as Core Services
    participant X as Antrean Pengecualian

    Note over D: Saat online, perangkat menarik delta data:<br/>produk, harga, anggota terbatas, rule bundle bertanda tangan
    G-->>D: Delta master data + rule bundle v-N
    D->>D: Verifikasi tanda tangan bundle

    Note over K,D: Koneksi terputus
    K->>D: Transaksi penjualan
    D->>D: Evaluasi harga/promo/poin dengan rule bundle lokal
    D->>D: Cek batas offline anggota untuk saldo/kredit/poin
    D->>D: Simpan sale + client_txn_id UUIDv7 ke outbox lokal
    D-->>K: Cetak struk nomor perangkat

    Note over K,D: Koneksi kembali
    loop Setiap batch maks 100 transaksi
        D->>G: POST batch dengan idempotency key per transaksi
        G->>G: Autentikasi perangkat & tenant
        G->>S: Proses transaksi baru, lewati yang sudah ada
        S->>S: Stok, HPP, saldo, piutang, poin, jurnal
        alt Konflik: saldo/poin tidak cukup, anggota nonaktif, stok negatif
            S->>X: Catat pengecualian, transaksi tetap diterima
            Note over S: Kekurangan saldo menjadi piutang anggota
        end
        S-->>G: Hasil per transaksi
        G-->>D: Ack daftar client_txn_id
        D->>D: Tandai tersinkron, hapus dari outbox
    end
    G-->>D: Delta terbaru + saldo anggota terbaru
    Note over X: Supervisor menindaklanjuti pengecualian di back-office
```

## 5.8 Posting Engine (Event → Jurnal)

```mermaid
flowchart TD
    A(["Transaksi modul tersimpan<br/>mis. sale.completed, saving.deposited"]) --> B["Tulis event ke Outbox<br/>dalam transaksi DB yang sama"]
    B --> C{"Mode posting?"}
    C -->|Sinkron: teller, kas| D["Posting Engine dipanggil langsung"]
    C -->|Asinkron: sinkron POS, job massal| E["Worker mengambil event dari Outbox"]
    E --> D
    D --> F{"Sudah pernah diposting?<br/>idempotency key = event_id"}
    F -->|Ya| F1(["Lewati"])
    F -->|Tidak| G["Ambil template posting aktif<br/>sesuai tanggal transaksi & lingkup"]
    G --> H["Evaluasi baris: nominal, sisi D/K, role, dimensi"]
    H --> I["Resolusi account role → akun<br/>lingkup paling spesifik"]
    I --> J{"Validasi:<br/>seimbang, akun posting aktif,<br/>periode terbuka?"}
    J -->|Periode tertutup| K["Rule period.late_transaction.policy"]
    K -->|Post ke periode berjalan| L
    K -->|Tahan| M
    J -->|Gagal lain| M["Antrean Posting Gagal<br/>+ notifikasi akuntan"]
    M --> M1["Akuntan perbaiki pemetaan/template"]
    M1 --> G
    J -->|Lolos| L["Simpan journal_entry POSTED + journal_lines<br/>tautan source_type/source_id"]
    L --> N["Update saldo akun inkremental<br/>account_balances per periode & dimensi"]
    N --> O["Tandai event diproses"]
    O --> P(["Jurnal tersedia di buku besar"])
```

## 5.9 Potong Gaji Bulanan (Koperasi Karyawan)

```mermaid
flowchart TD
    A(["Tanggal cut-off bulanan"]) --> B["Bendahara pilih instansi & periode"]
    B --> C["Sistem kumpulkan tagihan per anggota:<br/>simpanan wajib, angsuran jatuh tempo,<br/>kredit belanja POS, lainnya"]
    C --> D["Urutkan & batasi sesuai rule<br/>maks. persentase gaji"]
    D --> E["Buat batch potong gaji & ekspor file<br/>Excel/CSV format instansi"]
    E --> F["Kirim ke bagian payroll instansi"]
    F --> G["Instansi transfer dana + file realisasi"]
    G --> H["Impor file realisasi"]
    H --> I{"Cocok per anggota?"}
    I -->|Sesuai| J["Alokasi ke tagihan sesuai prioritas"]
    I -->|Kurang| K["Alokasi sebagian, sisa jadi tunggakan"]
    I -->|Lebih| L["Kelebihan ke titipan anggota"]
    I -->|Anggota tidak dikenal| M["Daftar selisih untuk ditindaklanjuti"]
    J --> N["Posting jurnal S8 / L3 / P12"]
    K --> N
    L --> N
    N --> O["Rekonsiliasi dengan mutasi bank"]
    O --> P["Notifikasi rincian potongan ke anggota"]
    P --> Q(["Batch selesai"])
```

## 5.10 Pembelian sampai Pembayaran Supplier

```mermaid
flowchart TD
    A(["Stok di bawah minimum / kebutuhan"]) --> B["Buat Purchase Order"]
    B --> C{"Perlu persetujuan?<br/>rule purchase.approval"}
    C -->|Ya| D["Persetujuan manajer/bendahara"]
    D --> E
    C -->|Tidak| E["Kirim PO ke supplier"]
    E --> F["Penerimaan barang GRN<br/>cek qty, batch, kedaluwarsa"]
    F --> G["Stok bertambah, biaya rata-rata diperbarui"]
    G --> H{"Faktur sudah ada?"}
    H -->|Ya| I["Jurnal B1: Persediaan + PPN Masukan / Utang Dagang"]
    H -->|Belum| J["Jurnal ke Utang Penerimaan Belum Difaktur"]
    J --> K["Faktur diterima → reklasifikasi ke Utang Dagang"]
    I --> L["Jadwal jatuh tempo pembayaran"]
    K --> L
    L --> M["Pembayaran supplier<br/>jurnal B2"]
    M --> N(["Selesai"])
    F --> O{"Barang rusak/kurang?"}
    O -->|Ya| P["Retur pembelian — jurnal B4"]
```

## 5.11 Tutup Buku Bulanan

```mermaid
flowchart TD
    A(["Akhir bulan"]) --> B["Job otomatis: akrual jasa simpanan,<br/>akrual jasa pinjaman, denda, kolektibilitas & CKPN,<br/>penyusutan, kedaluwarsa poin"]
    B --> C["Akuntan membuka checklist tutup buku"]
    C --> D{"Antrean posting gagal = 0?"}
    D -->|Tidak| D1["Selesaikan posting gagal"]
    D1 --> D
    D -->|Ya| E{"Semua shift kasir & kas teller ditutup?"}
    E -->|Tidak| E1["Tutup shift yang tertinggal"]
    E1 --> E
    E -->|Ya| F{"Semua perangkat POS tersinkron?"}
    F -->|Tidak| F1["Hubungi gerai / tandai perangkat"]
    F1 --> F
    F -->|Ya| G{"Buku pembantu = akun kontrol?"}
    G -->|Tidak| G1["Investigasi & jurnal koreksi"]
    G1 --> G
    G -->|Ya| H{"Rekonsiliasi bank selesai?"}
    H -->|Tidak| H1["Cocokkan mutasi bank"]
    H1 --> H
    H -->|Ya| I["Jurnal penyesuaian manual bila ada"]
    I --> J["Rule period.before_close: cek tambahan tenant"]
    J --> K{"Lolos?"}
    K -->|Tidak| I
    K -->|Ya| L["Status SOFT_CLOSED<br/>hanya jurnal penyesuaian akuntan"]
    L --> M["Review laporan: neraca saldo, neraca, LHU"]
    M --> N["Persetujuan bendahara"]
    N --> O["Status CLOSED + snapshot laporan"]
    O --> P["Kirim laporan bulanan ke pengurus & pengawas"]
    P --> Q(["Periode terkunci"])
```

## 5.12 Tutup Buku Tahunan, SHU & RAT

```mermaid
flowchart TD
    A(["Desember ditutup — 12 periode CLOSED"]) --> B["Jurnal penyesuaian tahunan<br/>audit/KAP bila ada"]
    B --> C["Jurnal penutup Y1–Y3:<br/>pendapatan & beban → Ikhtisar → SHU Tahun Berjalan"]
    C --> D["Buka tahun buku baru, saldo neraca dibawa"]
    D --> E["Jurnal Y4: SHU Tahun Berjalan → SHU Belum Dibagi"]
    E --> F["Kalkulator SHU: ambil SHU bersih"]
    F --> G["Terapkan rule shu.allocation<br/>validasi total 100%"]
    G --> H["Hitung dasar jasa modal per anggota<br/>rule shu.capital_basis"]
    G --> I["Hitung dasar jasa usaha per anggota<br/>belanja POS, jasa pinjaman, penjualan ke koperasi<br/>rule shu.business_basis"]
    H --> J["SHU per anggota = jasa modal + jasa usaha<br/>pembulatan & selisih"]
    I --> J
    J --> K["Simulasi skenario & bandingkan"]
    K --> L["Susun paket laporan RAT:<br/>laporan keuangan, keanggotaan, usaha, usulan SHU, RAPB"]
    L --> M(["RAT dilaksanakan"])
    M --> N{"RAT mengesahkan?"}
    N -->|Revisi alokasi| O["Ubah versi rule shu.allocation<br/>sesuai keputusan RAT"]
    O --> G
    N -->|Disahkan| P["Unggah notulen/berita acara RAT<br/>persetujuan pengurus"]
    P --> Q["Jurnal Y5: pembagian SHU & dana-dana"]
    Q --> R["Distribusi: tunai / transfer / ke simpanan sukarela<br/>potong pajak rule shu.tax — jurnal Y6"]
    R --> S["Slip SHU per anggota: PDF, WA, aplikasi"]
    S --> T(["SHU selesai dibagikan"])
```

## 5.13 Pembuatan & Publikasi Rule

```mermaid
flowchart TD
    A(["Pengurus/akuntan buka Rule Builder"]) --> B{"Mulai dari?"}
    B -->|Rule pack| C["Salin rule default"]
    B -->|Kosong| D["Pilih hook & tipe rule"]
    C --> E
    D --> E["Susun rule: mode Sederhana atau formula"]
    E --> F["Sistem menampilkan kalimat bahasa Indonesia<br/>dari rule"]
    F --> G["Validasi otomatis:<br/>skema, tipe variabel, aksi diizinkan,<br/>guardrail, celah tabel, siklus ECA"]
    G --> H{"Valid?"}
    H -->|Tidak| E
    H -->|Ya| I["Jalankan test case & uji cepat"]
    I --> J{"Lulus?"}
    J -->|Tidak| E
    J -->|Ya| K["Replay historis & laporan dampak"]
    K --> L["Tentukan tanggal berlaku & catatan perubahan"]
    L --> M["Ajukan persetujuan sesuai kategori"]
    M --> N{"Disetujui semua penyetuju?"}
    N -->|Tidak| E
    N -->|Ya| O{"Shadow mode?"}
    O -->|Ya| P["Jalankan paralel N hari, tinjau perbedaan"]
    P --> Q
    O -->|Tidak| Q["Aktif pada effective_from<br/>versi lama SUPERSEDED"]
    Q --> R["Rule bundle POS diperbarui & didistribusikan<br/>bila hook pos/loyalty"]
    R --> S(["Rule berlaku, log eksekusi tercatat"])
```

## 5.14 Anggota Keluar

```mermaid
flowchart TD
    A(["Permohonan keluar / diberhentikan / meninggal"]) --> B["Hitung hak anggota:<br/>simpanan pokok, wajib, sukarela, berjangka,<br/>SHU terutang"]
    B --> C["Hitung kewajiban:<br/>sisa pinjaman + jasa + denda,<br/>piutang belanja, tunggakan lain"]
    C --> D{"Hak ≥ kewajiban?"}
    D -->|Ya| E["Kompensasi kewajiban dari hak"]
    D -->|Tidak| F["Kompensasi sebagian,<br/>sisa kewajiban ditagih / ke ahli waris / hapus buku sesuai keputusan"]
    E --> G["Persetujuan pengurus"]
    F --> G
    G --> H["Jurnal S9: reklasifikasi pokok & wajib<br/>ekuitas → utang pengembalian"]
    H --> I["Jurnal S10: kompensasi"]
    I --> J{"Meninggal?"}
    J -->|Ya| K["Verifikasi ahli waris & dokumen"]
    K --> L
    J -->|Tidak| L["Pembayaran sisa hak — jurnal S11<br/>sesuai jadwal pengembalian AD/ART"]
    L --> M["Tutup semua rekening, nonaktifkan kartu & akun poin"]
    M --> N["Status KELUAR / MENINGGAL<br/>data disimpan sesuai retensi"]
    N --> O(["Selesai"])
```

[← 04 Bagan Akun](./04-bagan-akun-dan-posting.md) · [Lanjut ke 06 — ERD →](./06-erd.md)
