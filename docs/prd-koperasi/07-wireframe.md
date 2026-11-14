# 07 — Konsep Wireframe

[← Kembali ke daftar isi](./README.md)

---

Wireframe di dokumen ini adalah **konsep tata letak berfidelitas rendah** (low-fidelity) untuk menyamakan
pemahaman alur dan isi layar. Desain visual (warna, tipografi, ikon) dibuat di Figma pada Fase 0 berdasarkan
prinsip di bawah. Notasi: `[Tombol]`, `[Pilihan v]` = dropdown, `( )`/`(o)` = radio, `[ ]`/`[x]` = checkbox,
`[____]` = isian, `>` = buka detail.

## 1. Prinsip UI/UX

| Prinsip | Penerapan |
|---|---|
| **Bahasa awam dulu, istilah akuntansi kemudian** | Label "Uang masuk/keluar" dengan istilah akuntansi sebagai keterangan; mode "Akuntan" menampilkan kode akun & debit/kredit penuh |
| **Satu anggota, satu layar** | Profil 360° menjadi pusat: semua transaksi dapat dimulai dari profil anggota |
| **Keyboard-first di kasir & teller** | Pintasan F-key, fokus otomatis ke kolom scan, tanpa wajib mouse |
| **Status selalu terlihat** | Indikator online/offline, antrean sinkron, periode akuntansi aktif, versi rule |
| **Aman dari kesalahan** | Konfirmasi untuk aksi finansial, maker-checker, pratinjau jurnal sebelum posting, tombol "Batalkan dengan pembalik" alih-alih hapus |
| **Dapat ditelusuri** | Setiap angka di laporan dapat diklik sampai dokumen sumber |
| **Responsif** | Back-office: desktop ≥ 1280 px, tablet 768 px; POS: tablet/desktop landscape & Android 6"; aplikasi anggota: mobile |
| **Aksesibel** | Kontras AA, ukuran sentuh ≥ 44 px, teks dapat diperbesar, tidak bergantung warna saja |

## 2. Sitemap


```text
KoperasiOS
├── Konsol Super Admin (platform)
│   ├── Tenant: daftar, onboarding, paket, feature flag, impersonasi
│   ├── Template: COA, laporan, rule pack, dokumen
│   ├── Billing & penggunaan
│   └── Monitoring & audit platform
├── Back-office Koperasi (per tenant)
│   ├── Beranda (dashboard)
│   ├── Keanggotaan: daftar anggota, pendaftaran, persetujuan, kelompok, kartu, tier, pelanggan
│   ├── Simpanan: teller, rekening, tagihan wajib, potong gaji, jasa simpanan, berjangka
│   ├── Pinjaman: pengajuan, persetujuan, pencairan, angsuran, kolektibilitas, agunan, laporan
│   ├── Toko & Inventori: produk, harga, stok, pembelian, supplier, opname, konsinyasi, laporan POS
│   ├── Loyalty: program poin, tier, voucher
│   ├── Akuntansi: jurnal, buku besar, buku pembantu, kas & bank, rekonsiliasi, aset tetap,
│   │              anggaran, pajak, tutup buku, antrean posting gagal
│   ├── Laporan: keuangan (neraca, LHU, ekuitas, arus kas, CaLK), operasional, rasio
│   ├── SHU & RAT: kalkulator SHU, distribusi, dana-dana, paket RAT
│   ├── Aturan (Rule Engine): katalog, editor, uji & simulasi, persetujuan, log keputusan,
│   │                         tabel referensi, custom field, template posting, account role
│   ├── Persetujuan (inbox)
│   └── Pengaturan: profil koperasi, cabang & unit, pengguna & peran, perangkat, notifikasi,
│                   integrasi, periode akuntansi, impor data
├── POS (PWA/Android): login perangkat, shift, kasir, pembayaran, retur, laporan shift, sinkron
└── Aplikasi Anggota (Android/iOS): beranda, kartu, simpanan, pinjaman, belanja, poin, SHU, profil
```

## 3. Back-office

### 3.1 Kerangka Layout (Shell)


```text
┌─ Shell Back-office ──────────────────────────────────────────────────────────────────────────────┐
├────────────────────────┬─────────────────────────────────────────────────────────────────────────┤
│ KoperasiOS             │ [Cari anggota, produk, dokumen (Ctrl+K)]     Periode: Mar 2027 (OPEN)   │
│ Kopdes Sukamaju  [v]   │ Cabang: [Semua v]  Unit: [Semua v]    [Lonceng 3]  Sri (Bendahara) [v]  │
│                        │ ----------------------------------------------------------------------- │
│ > Beranda              │ Beranda > Akuntansi > Buku Besar                                        │
│ > Keanggotaan          │                                                                         │
│ > Simpanan             │                        AREA KONTEN                                      │
│ > Pinjaman             │                                                                         │
│ > Toko & Inventori     │   - Judul halaman + aksi utama di kanan atas                            │
│ > Loyalty              │   - Filter di atas tabel, simpan sebagai tampilan favorit               │
│ > Akuntansi            │   - Tabel dengan kolom dapat diatur, ekspor Excel/PDF                   │
│ > Laporan              │   - Panel samping (drawer) untuk detail tanpa pindah halaman            │
│ > SHU & RAT            │                                                                         │
│ > Aturan               │                                                                         │
│ > Persetujuan (4)      │ Status: Online | Posting gagal: 0 | Rule pack v7 | App 1.4.2            │
│ > Pengaturan           │                                                                         │
│                        │                                                                         │
│ [?] Bantuan            │                                                                         │
└────────────────────────┴─────────────────────────────────────────────────────────────────────────┘
```


### 3.2 Konsol Super Admin — Daftar Tenant


```text
┌─ Super Admin > Tenant ───────────────────────────────────────────────────────────────────────────┐
│ Tenant (1.284)                                                  [+ Tenant Baru]  [Ekspor]        │
│ Cari [______________________]  Jenis [Semua v]  Paket [Semua v]  Status [Aktif v]                │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Kode        Nama Koperasi                 Jenis      Paket       Anggota  Status    Aksi         │
│ ----------  ----------------------------  ---------  ----------  -------  --------  ------       │
│ kds-skmj    Kopdes Merah Putih Sukamaju    KOPDES     Dasar           412  AKTIF     [...]       │
│ kkpt-maju   Kop. Karyawan PT Maju Jaya     KONSUMEN   Usaha         2.531  AKTIF     [...]       │
│ ksp-sejht   KSP Sejahtera Bersama          KSP        Enterprise   18.204  AKTIF     [...]       │
│ ksu-tani    KSU Tani Makmur                KSU        Usaha           980  TRIAL     [...]       │
│                                                                                                  │
│ Menu [...]: Lihat detail | Ubah paket | Feature flag | Minta impersonasi | Suspend | Ekspor data │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Detail (drawer): profil badan hukum, langganan, kuota (anggota 412/1.000, terminal 2/3),         │
│ versi template COA & rule pack, riwayat audit platform, kontak admin tenant.                     │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```


### 3.3 Wizard Onboarding Tenant


```text
┌─ Onboarding > Jenis & Template ──────────────────────────────────────────────────────────────────┐
│ Langkah: (1) Profil  (2) Jenis & Template  (3) Unit & Cabang  (4) Aturan AD/ART  (5) Saldo Awal  │
│                      ^^^^^^^^^^^^^^^^^^^^                                                        │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Pilih jenis koperasi — menentukan Bagan Akun Standar, template laporan, dan rule pack awal:      │
│                                                                                                  │
│ ┌ Konsumen / Karyawan ┐ ┌ Simpan Pinjam ──────┐ ┌ Serba Usaha ────────┐ ┌ Kopdes Merah Putih ─┐  │
│ │ (o) Toko, simpan    │ │ ( ) KSP / USP       │ │ ( ) Multi unit      │ │ ( ) Gerai, simpan   │  │
│ │     pinjam, potong  │ │     SAK EP wajib    │ │     usaha +         │ │     pinjam, apotek  │  │
│ │     gaji            │ │                     │ │     konsinyasi      │ │                     │  │
│ │ [Lihat template]    │ │ [Lihat template]    │ │ [Lihat template]    │ │ [Lihat template]    │  │
│ └─────────────────────┘ └─────────────────────┘ └─────────────────────┘ └─────────────────────┘  │
│                                                                                                  │
│ Kerangka akuntansi: (o) SAK EP   ( ) SAK EMKM        Status PKP: ( ) Ya  (o) Tidak               │
│ Tahun buku: [01/01] s/d [31/12]   Mulai menggunakan sistem per: [01/04/2027]                     │
│                                                                                                  │
│ Template terpilih: COA Konsumen v3 (148 akun)  |  Rule pack Konsumen v7 (36 rule)                │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ [< Kembali]                                                      [Simpan Draft]  [Lanjut >]      │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```


### 3.4 Beranda / Dashboard Koperasi


```text
┌─ Beranda ────────────────────────────────────────────────────────────────────────────────────────┐
│ Selamat pagi, Bu Sri               Periode: Maret 2027 (OPEN)       [Unit: Semua v] [Ekspor]     │
│                                                                                                  │
│ ┌ Anggota Aktif ─────┐  ┌ Total Simpanan ────┐  ┌ Pinjaman Beredar ──┐  ┌ Penjualan Hari Ini ┐   │
│ │ 2.531              │  │ Rp 8,42 M          │  │ Rp 6,15 M          │  │ Rp 18,7 jt         │   │
│ │ +18 bulan ini      │  │ Sukarela Rp 4,6 M  │  │ NPL 2,8%           │  │ Anggota 71%        │   │
│ └────────────────────┘  └────────────────────┘  └────────────────────┘  └────────────────────┘   │
│                                                                                                  │
│ ┌ Penjualan 30 hari (anggota vs non-anggota) ┐  ┌ Perlu Tindakan ────────────────────────────┐   │
│ │  |        __      _/\_                     │  │ (!) 4 persetujuan pinjaman menunggu        │   │
│ │  |  _/\__/  \____/    \__/\                │  │ (!) 2 pengecualian sinkron POS             │   │
│ │  |_/                     \__               │  │ (!) 37 anggota tunggak wajib >= 3 bln      │   │
│ │  +-----------------------------            │  │ (i) Rekonsiliasi bank Feb belum selesai    │   │
│ │  ___ anggota    --- non-anggota            │  └────────────────────────────────────────────┘   │
│ └────────────────────────────────────────────┘                                                   │
│ ┌ Posisi Kas & Bank ─────────────────────────┐  ┌ SHU Berjalan (Jan-Mar 2027) ───────────────┐   │
│ │ Kas besar     Rp   45.200.000              │  │ SHU sebelum pajak   Rp 214.800.000         │   │
│ │ Kas gerai     Rp    6.850.000              │  │ vs anggaran (RAPB)  92%                    │   │
│ │ Bank BRI      Rp  812.400.000              │  │ [Lihat Laporan Hasil Usaha >]              │   │
│ │ Kliring QRIS  Rp    3.120.000              │  └────────────────────────────────────────────┘   │
│ └────────────────────────────────────────────┘                                                   │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```


### 3.5 Profil Anggota 360°


```text
┌─ Keanggotaan > Profil Anggota ───────────────────────────────────────────────────────────────────┐
│ [Foto]  BUDI SANTOSO   A-SKM-2019-00123   Status: AKTIF   Tier: GOLD   Kelompok: Dusun Krajan    │
│         NIK 3374********0003 [Tampilkan*]   HP 0812-****-7788   Anggota sejak 12/05/2019         │
│         [Setor] [Tarik] [Ajukan Pinjaman] [Bayar Angsuran] [Cetak Kartu] [Ubah Data] [...]       │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Tab: [Ringkasan] Simpanan  Pinjaman  Belanja  Poin  SHU  Dokumen  Riwayat & Audit                │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ ┌ Simpanan ──────────────────────────────────┐  ┌ Pinjaman ──────────────────────────────────┐   │
│ │ Pokok          Rp    100.000               │  │ PJ-2026-0412  Reguler 12 bln               │   │
│ │ Wajib          Rp  4.200.000               │  │ Sisa pokok     Rp  3.750.000               │   │
│ │ Sukarela       Rp  1.200.000               │  │ Angsuran berikut 25/03  Rp 712.500         │   │
│ │ Tunggakan wajib: 0 bulan                   │  │ Kolektibilitas: LANCAR (DPD 0)             │   │
│ │ [Mutasi >]                                 │  │ [Jadwal >]                                 │   │
│ └────────────────────────────────────────────┘  └────────────────────────────────────────────┘   │
│ ┌ Belanja & Poin ────────────────────────────┐  ┌ SHU ───────────────────────────────────────┐   │
│ │ Belanja 12 bulan   Rp 9.840.000            │  │ SHU 2026 diterima  Rp 517.400              │   │
│ │ Poin 1.250  (exp. 300 poin 30/04)          │  │   jasa modal Rp  86.000                    │   │
│ │ Limit kredit sisa  Rp   750.000            │  │   jasa usaha Rp 431.400                    │   │
│ └────────────────────────────────────────────┘  │ Estimasi 2027 (berjalan) Rp 180.000        │   │
│                                                 └────────────────────────────────────────────┘   │
│ * Menampilkan NIK penuh memerlukan izin "lihat data sensitif" dan tercatat di audit log.         │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```


### 3.6 Teller Simpanan — Setor / Tarik


```text
┌─ Simpanan > Teller ──────────────────────────────────────────────────────────────────────────────┐
│ Kas teller: Rp 12.450.000   Teller: Dodi   Cabang: Pusat                [Tutup Kas Teller]       │
├────────────────────────────────────────────────┬─────────────────────────────────────────────────┤
│ Anggota [Scan QR / no. anggota______] (F2)     │ Rekening anggota                                │
│                                                │   Pokok      Rp    100.000   lunas              │
│ BUDI SANTOSO  A-SKM-2019-00123  AKTIF          │   Wajib      Rp  4.050.000   tunggak 3 bln      │
│                                                │   Sukarela   Rp  1.200.000                      │
│ Transaksi: (o) Setor   ( ) Tarik               │                                                 │
│ Rekening:  [Simpanan Wajib          v]         │ Pemeriksaan aturan                              │
│ Nominal:   [Rp 150.000_______________]         │   [OK] Rekening aktif                           │
│ Alokasi:   Jan, Feb, Mar 2027 (3 x 50.000)     │   [OK] Setoran >= minimum Rp 10.000             │
│ Metode:    (o) Tunai ( ) Transfer ( ) QRIS     │   [!]  Tunggakan wajib 3 bulan akan dilunasi    │
│ Keterangan [_____________________________]     │                                                 │
│                                                │ Pratinjau jurnal (mode akuntan)                 │
│ [Batal (Esc)]          [Simpan & Cetak (F9)]   │   D 110104 Kas Teller        150.000            │
│                                                │   K 310201 Simpanan Wajib    150.000            │
├────────────────────────────────────────────────┴─────────────────────────────────────────────────┤
│ Setelah simpan: slip tercetak, bukti dikirim via WA, saldo & tagihan diperbarui seketika.        │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```


### 3.7 Pengajuan Pinjaman — Simulasi & Kelayakan


```text
┌─ Pinjaman > Pengajuan Baru ──────────────────────────────────────────────────────────────────────┐
│ PJ-2027-000184 (DRAFT)   Anggota: BUDI SANTOSO A-SKM-2019-00123                [Riwayat]         │
├────────────────────────────────────────────────┬─────────────────────────────────────────────────┤
│ Produk   [Pinjaman Reguler          v]         │ Kelayakan (rule pack v7)                        │
│ Nominal  [Rp 10.000.000____________]           │ [LULUS] Masa anggota 94 bln >= 6 bln            │
│ Tenor    [12 v] bulan                          │ [LULUS] Status AKTIF                            │
│ Metode   Anuitas 18%/tahun                     │ [LULUS] Tunggakan wajib 0 bln                   │
│ Tujuan   [Modal usaha warung______]            │ [LULUS] Rasio angsuran 37% <= 40%               │
│ Agunan   [+ Tambah agunan]                     │                                                 │
│ Dokumen  [+ Unggah] KTP.jpg, SlipGaji.pdf      │ Plafon maksimum      Rp 12.600.000              │
│                                                │   = min(3 x simpanan 16,5 jt,                   │
│ Simulasi angsuran                              │         kapasitas bayar 12,6 jt,                │
│ Ke  Jatuh tempo   Pokok    Jasa     Total      │         plafon produk 50 jt)                    │
│  1  25/04/2027   766.800  150.000  916.800     │                                                 │
│  2  25/05/2027   778.300  138.500  916.800     │ Rute persetujuan: Manajer > Bendahara           │
│ ...                                            │ (nominal 5-25 jt, SLA 24 + 48 jam)              │
│ 12  25/03/2028   903.300   13.500  916.800     │                                                 │
│ Biaya provisi 1% Rp 100.000 (dipotong)         │ [Lihat jejak keputusan >]                       │
├────────────────────────────────────────────────┴─────────────────────────────────────────────────┤
│ [Simpan Draft]                   [Cetak Simulasi]                   [Ajukan Persetujuan >]       │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```


### 3.8 Inbox Persetujuan


```text
┌─ Persetujuan ────────────────────────────────────────────────────────────────────────────────────┐
│ Filter: [Semua jenis v]  [Menunggu saya v]                       Lewat SLA: 1   Total: 6         │
├────────────────────────────────────────────┬─────────────────────────────────────────────────────┤
│ > Pinjaman PJ-2027-000184   Rp 10 jt       │ Pinjaman PJ-2027-000184 — Budi Santoso              │
│   Budi Santoso   langkah 2/2  sisa 30 jam  │ Nominal Rp 10.000.000 | 12 bln | anuitas 18%        │
│   Jurnal JU-2027-03-000091  Rp 4,2 jt      │ Plafon Rp 12.600.000 | Kolektibilitas LANCAR        │
│   Penyesuaian akrual         sisa 12 jam   │ Langkah 1 Manajer: DISETUJUI (Andi, 11/03)          │
│   Rule FIN-POS-07 v3 Promo Jumat           │   "Riwayat bayar baik"                              │
│   Diajukan Manajer           sisa 2 hari   │ Langkah 2 Bendahara: MENUNGGU ANDA                  │
│   Opname GDG-01 Maret  selisih -Rp 320rb   │                                                     │
│   (!) lewat SLA 6 jam                      │ Lampiran: KTP.jpg  SlipGaji.pdf  Simulasi.pdf       │
│                                            │ Catatan [________________________________]          │
│                                            │                                                     │
│                                            │ [Tolak]  [Minta Revisi]  [Delegasikan]  [Setujui]   │
└────────────────────────────────────────────┴─────────────────────────────────────────────────────┘
```


### 3.9 Jurnal Umum — Form Entri


```text
┌─ Akuntansi > Jurnal Umum > Baru ─────────────────────────────────────────────────────────────────┐
│ No: (otomatis)   Tanggal [31/03/2027]   Jenis [Penyesuaian v]   Periode: Mar 2027 (SOFT_CLOSED)  │
│ Keterangan [Akrual beban listrik Maret 2027_________________________________________]            │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ No  Akun                                   Dimensi (Cabang/Unit)   Debit         Kredit          │
│ --  -------------------------------------  ----------------------  ------------  ------------    │
│  1  [610201 Listrik, Air & Telepon     v]  [Pusat / Toko       v]  [ 2.450.000]  [          ]    │
│  2  [610201 Listrik, Air & Telepon     v]  [Pusat / Simpan Pjm v]  [ 1.750.000]  [          ]    │
│  3  [210702 Beban Lain YMH Dibayar     v]  [Pusat / -          v]  [          ]  [ 4.200.000]    │
│     [+ Baris]                                               Total     4.200.000     4.200.000    │
│                                                             Selisih   0  (SEIMBANG)              │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Lampiran [+ Unggah] tagihan-pln-mar.pdf      [x] Buat jurnal pembalik otomatis 01/04/2027        │
│ Info: Akun kontrol buku pembantu (mis. 110401, 210101) tidak tersedia di jurnal manual.          │
│ Persetujuan: nominal >= Rp 1.000.000 memerlukan persetujuan Bendahara (rule ACC-APR-01 v2)       │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ [Batal]                                    [Simpan Draft]                 [Ajukan untuk Posting] │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```


### 3.10 Buku Besar


```text
┌─ Akuntansi > Buku Besar ─────────────────────────────────────────────────────────────────────────┐
│ Akun [110103 Kas Gerai / Kasir POS v] (termasuk sub-akun)  Periode [01/03/2027]-[31/03/2027]     │
│ Dimensi: Cabang [Semua v] Unit [Toko v]       [Mode akuntan: ON]       [Ekspor Excel] [PDF]      │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Tanggal     No. Jurnal          Keterangan                       Debit       Kredit      Saldo   │
│ ----------  ------------------  -------------------------  ----------  ----------  ----------    │
│ 01/03/2027                      Saldo awal                                          6.120.000    │
│ 01/03/2027  JO-2027-03-000412 > Penjualan shift G01-0412    4.850.000              10.970.000    │
│ 01/03/2027  JO-2027-03-000415 > Setoran kas gerai ke bank               4.500.000   6.470.000    │
│ 02/03/2027  JO-2027-03-000431 > Penjualan shift G01-0413    5.210.000              11.680.000    │
│ 02/03/2027  JO-2027-03-000433 > Selisih kurang kas shift                    5.000  11.675.000    │
│ ...                                                                                              │
│ 31/03/2027                      Saldo akhir                                         6.850.000    │
│                                 Total mutasi              142.300.000 141.570.000                │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Klik > membuka jurnal: baris D/K, dokumen sumber (laporan shift + struk), lampiran, audit.       │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```


### 3.11 Laporan Keuangan — Laporan Hasil Usaha


```text
┌─ Laporan > Keuangan ─────────────────────────────────────────────────────────────────────────────┐
│ Laporan [Laporan Hasil Usaha v]  Periode [Jan-Mar 2027 v]  Pembanding [Jan-Mar 2026 v]           │
│ Lingkup [Konsolidasi v]  Format [SAK EP - Permenkop 2/2024 v]   [Ekspor PDF] [Excel] [Kirim]     │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Pos                                                     Jan-Mar 2027     Jan-Mar 2026     %      │
│ ------------------------------------------------------  ---------------  ---------------  ----   │
│ A. PARTISIPASI ANGGOTA                                                                           │
│    Partisipasi bruto anggota                         >   1.284.500.000    1.102.300.000   +17    │
│    Beban pokok partisipasi                           >    (982.100.000)    (861.900.000)  +14    │
│    Partisipasi neto anggota                                302.400.000      240.400.000   +26    │
│ B. PENDAPATAN DARI NON-ANGGOTA                                                                   │
│    Penjualan non-anggota                             >     412.700.000      388.000.000    +6    │
│    HPP non-anggota                                   >    (351.800.000)    (332.500.000)   +6    │
│    Laba kotor non-anggota                                   60.900.000       55.500.000   +10    │
│ C. Pendapatan usaha lainnya                          >      12.300.000        9.100.000   +35    │
│ SISA HASIL USAHA KOTOR                                     375.600.000      305.000.000   +23    │
│ E. Beban usaha                                       >    (158.900.000)    (141.200.000)  +13    │
│ F. Pendapatan (beban) lain-lain                      >      (1.900.000)        2.300.000     -   │
│ SHU SEBELUM PAJAK                                          214.800.000      166.100.000   +29    │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Tab lain: Neraca | Perubahan Ekuitas | Arus Kas | CaLK | Neraca Saldo | Neraca Lajur | Rasio     │
│ Klik > untuk rincian akun -> buku besar -> jurnal -> dokumen sumber.                             │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```


### 3.12 Tutup Buku Bulanan — Checklist


```text
┌─ Akuntansi > Tutup Buku ─────────────────────────────────────────────────────────────────────────┐
│ Periode: Maret 2027        Status: OPEN -> [Soft Close]  -> [Close]          Progres: 6/8        │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ [OK]  Akrual jasa simpanan dijalankan (31/03 23:05)                                   [Log]      │
│ [OK]  Akrual jasa pinjaman, denda, kolektibilitas & CKPN dijalankan                   [Log]      │
│ [OK]  Penyusutan aset tetap Maret diposting (JO-2027-03-001902)                     [Jurnal]     │
│ [OK]  Antrean posting gagal = 0                                                                  │
│ [OK]  Semua shift kasir & kas teller ditutup (48 shift)                                          │
│ [OK]  Buku pembantu = akun kontrol (7 akun kontrol, selisih Rp 0)                   [Detail]     │
│ [X]   Perangkat POS belum sinkron: G02-T01 (terakhir 30/03 18:12)               [Hubungi gerai]  │
│ [X]   Rekonsiliasi bank BRI 0123: 3 transaksi belum cocok                       [Rekonsiliasi]   │
│ [ ]   Cek tambahan tenant (rule period.before_close): stok opname bulanan        menunggu        │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Tombol [Soft Close] aktif bila semua item wajib OK. [Close] memerlukan persetujuan Bendahara.    │
│ Setelah Close: snapshot laporan disimpan, posting ke Maret ditolak, laporan dikirim ke pengurus. │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```


### 3.13 Kalkulator & Distribusi SHU


```text
┌─ SHU & RAT > Kalkulator SHU 2026 ────────────────────────────────────────────────────────────────┐
│ Tahun buku 2026 (CLOSED)   SHU bersih: Rp 642.000.000   Status: DISIMULASIKAN                    │
│ Langkah: (1) Alokasi  (2) Dasar Perhitungan  (3) Per Anggota  (4) Penetapan RAT  (5) Distribusi  │
├────────────────────────────────────────────────┬─────────────────────────────────────────────────┤
│ Alokasi (rule shu.allocation v4, AD/ART)       │ Dasar perhitungan                               │
│ Komponen               %        Nominal        │ Jasa modal: simpanan pokok + wajib,             │
│ Cadangan             40,0   256.800.000        │   rata-rata saldo akhir bulan                   │
│ Jasa modal           20,0   128.400.000        │   Total basis Rp 6,42 M -> 2,00%                │
│ Jasa usaha           25,0   160.500.000        │ Jasa usaha: belanja anggota (bobot 1,0)         │
│ Pengurus & pengawas   5,0    32.100.000        │   + jasa pinjaman dibayar (bobot 1,0)           │
│ Karyawan              5,0    32.100.000        │   Total basis Rp 5,35 M -> 3,00%                │
│ Pendidikan            2,5    16.050.000        │                                                 │
│ Sosial                2,5    16.050.000        │ Skenario: [A: AD/ART v] [B: cadangan 35%]       │
│ Total               100,0   642.000.000        │ [Bandingkan skenario]                           │
│ [Ubah alokasi -> versi rule baru]              │                                                 │
├────────────────────────────────────────────────┴─────────────────────────────────────────────────┤
│ Contoh: Budi Santoso  modal 4,30 jt x 2,00% = 86.000 | usaha 14,38 jt x 3,00% = 431.400          │
│ [Unduh daftar SHU per anggota]  [Paket laporan RAT]       [Lanjut ke Penetapan RAT >]            │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```


## 4. Rule Engine

### 4.1 Katalog Aturan


```text
┌─ Aturan > Katalog ───────────────────────────────────────────────────────────────────────────────┐
│ Aturan Koperasi                     [+ Aturan Baru]  [Salin dari Rule Pack]  [Tabel Referensi]   │
│ Domain [Semua v]  Tipe [Semua v]  Kategori [Semua v]  Status [Aktif v]  Cari [______________]    │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Kode          Nama                            Hook                    Tipe      Versi  Status    │
│ ------------  ------------------------------  ----------------------  --------  -----  --------- │
│ ADART-SP-01   Nominal simpanan pokok          saving.required_amount  PARAM     v2     AKTIF     │
│ ADART-SW-01   Simpanan wajib per golongan     saving.required_amount  TABEL     v3     AKTIF     │
│ FIN-LON-02    Plafon pinjaman reguler         loan.plafond            FORMULA   v4     AKTIF     │
│                                                                                 v5     TERJADWAL │
│ FIN-POS-07    Promo Jumat sembako (Gold)      pos.after_checkout      ECA       v3     DITINJAU  │
│ WF-LON-01     Persetujuan pinjaman jenjang    loan.approval_route     WORKFLOW  v2     AKTIF     │
│ ACC-PST-POS   Jurnal penjualan POS            posting.pos.sale_comp.  POSTING   v1     AKTIF     │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Kolom tambahan: berlaku mulai, pembuat, penyetuju, eksekusi 30 hari, error 30 hari.              │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```


### 4.2 Editor Aturan — ECA (Mode Sederhana)


```text
┌─ Aturan > Editor ────────────────────────────────────────────────────────────────────────────────┐
│ FIN-POS-07  Promo Jumat sembako (Gold)          v3 DRAFT        Kategori: FINANSIAL              │
│ Hook: pos.after_checkout (asinkron)    Prioritas [50]    Lingkup: Gerai [Semua v]                │
│ Berlaku [01/04/2027] s/d [__/__/____]  ( ) Retroaktif              [Mode Sederhana | Lanjutan]   │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ KETIKA   Penjualan POS selesai                                                                   │
│ JIKA     [Semua v] kondisi berikut terpenuhi:                                   [+ Kondisi]      │
│          [anggota.tier         v] [sama dengan v] [GOLD                 v]           [x]         │
│          [waktu.hari           v] [sama dengan v] [JUMAT                v]           [x]         │
│          [ada item dengan      v] [kategori    v] [termasuk v] [SEMBAKO, BERAS]      [x]         │
│          [+ Grup kondisi (SALAH SATU)]                                                           │
│ MAKA                                                                                [+ Aksi]     │
│    1.    [Beri poin            v]  jumlah = [ floor(subtotal_sembako / 10000) ]  [fx]   [x]      │
│          alasan [Promo Jumat Sembako]   masa berlaku [12] bulan                                  │
│    2.    [Kirim notifikasi     v]  template [POIN_GANDA v]  kanal [WhatsApp v]          [x]      │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Kalimat: "Ketika penjualan POS selesai, jika tier anggota adalah GOLD dan hari adalah Jumat dan  │
│          ada item berkategori SEMBAKO atau BERAS, maka beri poin sebesar subtotal sembako dibagi │
│          10.000 (dibulatkan ke bawah) yang berlaku 12 bulan, lalu kirim notifikasi WhatsApp."    │
│ Validasi: [OK] skema  [OK] tipe variabel  [OK] aksi diizinkan  [OK] 3/3 test case lulus          │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ [Simpan Draft]  [Uji Cepat]  [Simulasi & Dampak]  [Bandingkan dgn v2]       [Ajukan Persetujuan] │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```


### 4.3 Editor Tabel Keputusan & Formula


```text
┌─ Aturan > Editor (Lanjutan) ─────────────────────────────────────────────────────────────────────┐
│ ┌ Tabel keputusan: Tarif jasa sukarela (saving.interest.rate) ───────────────────────────────┐   │
│ │ Kebijakan hit [FIRST v]   Input: rekening.saldo_rata_rata_bulan                            │   │
│ │ #   Saldo rata-rata (Rp)             Tarif/thn   Catatan                                   │   │
│ │ 1   < 1.000.000                       0,00%                                                │   │
│ │ 2   >= 1.000.000 dan < 10.000.000     2,00%                                                │   │
│ │ 3   >= 10.000.000 dan < 50.000.000    3,00%                                                │   │
│ │ 4   >= 50.000.000                     3,50%                                                │   │
│ │ [+ Baris]   Pemeriksaan: [OK] tidak ada celah  [OK] tidak tumpang tindih                   │   │
│ └────────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                  │
│ ┌ Formula: Plafon pinjaman reguler (loan.plafond) ───────────────────────────────────────────┐   │
│ │ floor_to(                                                                                  │   │
│ │   min( 3 * anggota.total_simpanan,                                                         │   │
│ │        (0.40 * anggota.penghasilan_bulanan - anggota.angsuran_aktif_bulanan)               │   │
│ │          * pinjaman.tenor_bulan,                                                           │   │
│ │        param("pinjaman.reguler.plafon_maks") ),                                            │   │
│ │   100000 )                                                                                 │   │
│ │ Autocomplete: anggota.| -> total_simpanan, penghasilan_bulanan, masa_keanggotaan_bulan     │   │
│ │ Tipe hasil: Rupiah [OK]   Node: 18/500   Kedalaman: 6/20                                   │   │
│ └────────────────────────────────────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```


### 4.4 Uji, Simulasi & Analisis Dampak


```text
┌─ Aturan > Uji & Simulasi ────────────────────────────────────────────────────────────────────────┐
│ FIN-LON-02 Plafon pinjaman reguler   Bandingkan: v4 (aktif)  vs  v5 (draft)                      │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Test case                                             Harapan        Hasil v5       Status       │
│ Gaji 5 jt, simpanan 4 jt, tenor 12, tanpa angsuran    12.000.000     12.000.000     LULUS        │
│ Gaji 3 jt, simpanan 10 jt, tenor 6, angsuran 500rb    4.200.000      4.200.000      LULUS        │
│ Simpanan 0                                            0              0              LULUS        │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Replay historis: [01/09/2026] - [28/02/2027]  Sumber: 412 pengajuan        [Jalankan Replay]     │
│                                                                                                  │
│ Hasil replay                          v4 (aktif)        v5 (draft)        Selisih                │
│ Rata-rata plafon                      Rp 9,8 jt         Rp 11,2 jt        +14,3%                 │
│ Pengajuan melebihi plafon             37                19                -18                    │
│ Total plafon tersedia                 Rp 4,04 M         Rp 4,61 M         +Rp 0,57 M             │
│                                                                                                  │
│ Daftar pengajuan yang hasilnya berubah (18)                                    [Lihat >]         │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ [ ] Jalankan shadow mode 14 hari sebelum aktif        [Kembali ke Editor]  [Ajukan Persetujuan]  │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```


### 4.5 Template Posting & Account Role


```text
┌─ Aturan > Template Posting ──────────────────────────────────────────────────────────────────────┐
│ ACC-PST-POS  Jurnal penjualan POS  Hook: posting.pos.sale_completed  v1 AKTIF  [Versi baru]      │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Baris  Sisi  Account role (akun hasil pemetaan)                    Nominal                       │
│ -----  ----  ----------------------------------------------------  ---------------------------   │
│ 1      D     BAYAR_{metode} untuk setiap pembayaran                 pembayaran.nominal           │
│              CASH -> 110103.001 Kas Gerai Sukamaju 1                                             │
│              QRIS -> 110203.001 Kliring QRIS                                                     │
│              SAVING_BALANCE -> 210101 Simpanan Sukarela (buku pembantu: rekening)                │
│ 2      K     PENJUALAN_ANGGOTA / PENJUALAN_NON_ANGGOTA             total - pajak - konsinyasi    │
│ 3      K     UTANG_KONSINYASI (lewati bila 0)                      porsi penitip                 │
│ 4      K     PPN_KELUARAN (lewati bila 0)                          pajak                         │
│ 5      D     HPP_ANGGOTA / HPP_NON_ANGGOTA                         total HPP                     │
│ 6      K     PERSEDIAAN_BARANG_DAGANG -> 110701.001 Gudang Utama   total HPP                     │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Pratinjau dengan transaksi contoh [G01-270331-0087 v]:  Debit 174.500 = Kredit 174.500  SEIMBANG │
│ Role belum dipetakan: (tidak ada)                    [Kelola pemetaan account role >]            │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```


## 5. POS (Kasir)

### 5.1 Layar Kasir (Tablet/Desktop Landscape)


```text
┌─ POS ────────────────────────────────────────────────────────────────────────────────────────────┐
│ Gerai Sukamaju 1 | Kasir: Rina | Shift G01-0412 | ONLINE | Sinkron 09:41 | Antrean 0 | Bundle v7 │
├──────────────────────────────────────────────────────┬───────────────────────────────────────────┤
│ [Scan barcode / cari produk ..............] (F2)     │ PEMBELI  [Scan kartu anggota (F3)]        │
│                                                      │ BUDI SANTOSO  A-SKM-2019-00123  GOLD      │
│ Kategori: [Semua] [Sembako] [Minuman] [Bumbu] [>]    │ Poin 1.250 | Skr 1,2 jt | Kredit 750rb    │
│                                                      │ ----------------------------------------- │
│ ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │ Beras Premium 5kg   2 x 72.000    144.000 │
│ │ Beras 5kg    │  │ Minyak 2L    │  │ Gula 1kg     │ │   harga anggota (umum 75.000)             │
│ │ Rp 72.000    │  │ Rp 36.500    │  │ Rp 17.000    │ │ Minyak Goreng 2L    1 x 36.500     36.500 │
│ └──────────────┘  └──────────────┘  └──────────────┘ │ Gula Pasir 1kg      1 x 17.000     17.000 │
│ ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │ ----------------------------------------- │
│ │ Telur 1kg    │  │ Kopi 10s     │  │ Mie 5s       │ │ Subtotal                          197.500 │
│ │ Rp 28.000    │  │ Rp 14.500    │  │ Rp 15.500    │ │ Diskon tier Gold 2%                -3.950 │
│ └──────────────┘  └──────────────┘  └──────────────┘ │ TOTAL                         Rp 193.550  │
│                                                      │ Estimasi poin: +19 (+19 promo Jumat)      │
│ [Tahan (F6)] [Daftar Ditahan] [Retur (F7)] [...]     │                                           │
│                                                      │ [Batal (F8)]          [ BAYAR (F12) ]     │
└──────────────────────────────────────────────────────┴───────────────────────────────────────────┘
```


### 5.2 Pembayaran (Split)


```text
┌─ POS > Pembayaran ───────────────────────────────────────────────────────────────────────────────┐
│ Total tagihan: Rp 193.550                                  Sisa: Rp 0   Kembalian: Rp 6.450      │
├──────────────────────────────────────────────┬───────────────────────────────────────────────────┤
│ Metode pembayaran                            │ Rincian pembayaran                                │
│ [Tunai (1)]        [QRIS (2)]                │ 1. Poin 1.000                 Rp  10.000  [x]     │
│ [EDC (3)]          [Transfer (4)]            │ 2. Saldo sukarela             Rp  83.550  [x]     │
│ [Saldo Sukarela (5)]                         │ 3. Tunai (diterima 106.450)   Rp 100.000  [x]     │
│ [Kredit Anggota (6)]  sisa limit 750.000     │ -----------------------------------------------   │
│ [Poin (7)]  1.250 poin = Rp 12.500           │ Terbayar                      Rp 193.550          │
│ [Voucher (8)]                                │                                                   │
│                                              │ Offline? saldo sukarela & poin dibatasi           │
│ Nominal [Rp 100.000_____]  [Uang pas]        │ oleh rule pos.offline_limit (Rp 200.000)          │
│ Pecahan cepat: [50rb] [100rb] [200rb]        │                                                   │
│                                              │ [Kembali]     [Selesai & Cetak Struk (Enter)]     │
└──────────────────────────────────────────────┴───────────────────────────────────────────────────┘
```


### 5.3 Tutup Shift


```text
┌─ POS > Tutup Shift ──────────────────────────────────────────────────────────────────────────────┐
│ Shift G01-0412 | Rina | Buka 07:00 (modal Rp 500.000) | Transaksi 143 | Semua tersinkron [OK]    │
├──────────────────────────────────────────────┬───────────────────────────────────────────────────┤
│ Hitung uang fisik                            │ Ringkasan shift                                   │
│ Pecahan        Jumlah        Nilai           │ Penjualan tunai          4.420.000                │
│ 100.000        [ 32 ]    3.200.000           │ Retur tunai                (60.000)               │
│  50.000        [ 21 ]    1.050.000           │ Modal awal                 500.000                │
│  20.000        [ 18 ]      360.000           │ Kas seharusnya           4.860.000                │
│  10.000        [ 15 ]      150.000           │ Kas fisik                4.855.000                │
│   5.000        [ 12 ]       60.000           │ SELISIH                     (5.000)               │
│   2.000        [ 14 ]       28.000           │ Kebijakan: dibebankan (<= Rp 10.000)              │
│   1.000 & koin [    ]        7.000           │                                                   │
│ Total fisik              4.855.000           │ Non-tunai: QRIS 2,1 jt | Sukarela 0,8 jt          │
│                                              │ Kredit anggota 1,2 jt | Poin 45.000               │
├──────────────────────────────────────────────┴───────────────────────────────────────────────────┤
│ [Cetak Laporan Shift]                  [Batal]                  [Tutup Shift & Buat Setoran]     │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```


## 6. Aplikasi Anggota (Mobile)


```text
┌ Beranda ───────────────────────┐  ┌ Kartu Anggota ─────────────────┐  ┌ Ajukan Pinjaman ───────────────┐
│ Halo, Budi           [Notif]   │  │                                │  │ Produk [Reguler        v]      │
│ Kopdes Sukamaju     [Ganti v]  │  │       ##########  ######       │  │ Nominal [Rp 10.000.000  ]      │
├────────────────────────────────┤  │       ##  QR  ##  ##  ##       │  │ Tenor   [12 bulan       v]     │
│ Total simpanan                 │  │       ##########  ######       │  ├────────────────────────────────┤
│ Rp 5.500.000         [Lihat]   │  │                                │  │ Plafon Anda Rp 12.600.000      │
│ Pokok 0,1 Wajib 4,2 Skr 1,2 jt │  │   BUDI SANTOSO                 │  │ Angsuran +/- Rp 916.800/bln    │
├────────────────────────────────┤  │   A-SKM-2019-00123             │  │ Provisi Rp 100.000             │
│ Pinjaman PJ-2026-0412          │  │   Tier GOLD                    │  ├────────────────────────────────┤
│ Sisa Rp 3.750.000              │  │                                │  │ Dokumen: [+ Slip gaji]         │
│ Angsuran 25/03  Rp 712.500     │  │   Kode berganti tiap 60 dtk    │  │ [x] Saya menyetujui S&K        │
│ [Bayar]                        │  │   Tunjukkan ke kasir/teller    │  │                                │
├────────────────────────────────┤  │                                │  │ [ Ajukan ]                     │
│ Poin 1.250 | GOLD              │  │ [Unduh PDF]  [Laporkan Hilang] │  │                                │
│ Estimasi SHU 2027 Rp 180.000   │  └────────────────────────────────┘  │ Status pengajuan:              │
├────────────────────────────────┤                                      │ Diajukan > Manajer [OK] >      │
│ [Kartu] [Simpanan] [Pinjam]    │                                      │ Bendahara (menunggu)           │
│ [Belanja] [Akun]               │                                      └────────────────────────────────┘
└────────────────────────────────┘
```


Catatan aplikasi anggota:

- Login: nomor HP + OTP (WA/SMS), lalu PIN 6 digit/biometrik. Bila nomor HP terdaftar di beberapa koperasi, pengguna memilih koperasi.
- QR kartu bersifat dinamis (token bertanda tangan, berganti berkala) agar tidak dapat dipakai ulang dari tangkapan layar lama.
- Jika pengajuan ditolak, alasan ditampilkan dari jejak keputusan rule dalam bahasa sederhana.
- Mode hemat data: gambar dikompresi, data saldo di-cache untuk dibuka saat offline (ditandai "data per 09:41").

[← 06 ERD](./06-erd.md) · [Lanjut ke 08 — Task Todo →](./08-task-todo.md)
