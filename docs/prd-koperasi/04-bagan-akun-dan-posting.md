# 04 — Bagan Akun Standar (COA) & Matriks Posting Jurnal

[← Kembali ke daftar isi](./README.md)

---

## 1. Prinsip

1. **Standar platform, ekstensi tenant.** Level 1–4 adalah akun standar yang **terkunci** (kode & nama baku,
   tidak dapat dihapus, nama tampilan boleh diberi alias). Tenant hanya menambah **sub-akun level 5**
   (mis. per rekening bank, per gerai) di bawah akun standar yang ditandai *dapat diperluas*.
2. **Setiap akun dipetakan ke pos laporan.** Atribut `report_line` menentukan baris di Neraca/LHU, dan `cashflow_class`
   menentukan klasifikasi arus kas. Laporan selalu berformat standar walaupun tenant menambah sub-akun.
3. **Pemisahan anggota vs non-anggota** dilakukan dengan akun pendapatan & HPP terpisah (bukan sekadar dimensi),
   karena Laporan Hasil Usaha koperasi menyajikannya terpisah dan SHU jasa usaha bergantung padanya.
4. **Jurnal otomatis tidak merujuk kode akun langsung** melainkan **account role** (mis. `KAS_GERAI`) yang dipetakan
   ke akun per tenant/gerai/produk. Perubahan pemetaan tidak memerlukan perubahan template.
5. Template COA mengacu konvensi umum di Indonesia dan lampiran kebijakan akuntansi Permenkop UKM No. 2/2024
   (berbasis SAK EP). **Susunan final wajib divalidasi akuntan/KAP mitra di Fase 0** (`F0-DISC-02`).

## 2. Struktur Kode Akun

| Level | Nama | Digit | Contoh | Keterangan |
|---|---|---|---|---|
| 1 | Kelompok | 1 | `1` Aset | Terkunci |
| 2 | Golongan | 2 | `11` Aset Lancar | Terkunci |
| 3 | Jenis | 4 | `1101` Kas | Terkunci, header |
| 4 | Akun | 6 | `110103` Kas Gerai / Kasir POS | Terkunci, **akun posting** standar |
| 5 | Sub-akun tenant | 6 + `.` + 3 | `110103.001` Kas Gerai Sukamaju 1 | Dibuat tenant; akun posting; mewarisi atribut induk |

**Kelompok akun:**

| Kode | Kelompok | Saldo normal | Laporan |
|---|---|---|---|
| 1 | Aset | Debit | Neraca |
| 2 | Liabilitas | Kredit | Neraca |
| 3 | Ekuitas | Kredit | Neraca |
| 4 | Pendapatan (Partisipasi Anggota & Non-Anggota) | Kredit | Laporan Hasil Usaha |
| 5 | Beban Pokok (HPP & Beban Pokok Simpan Pinjam) | Debit | Laporan Hasil Usaha |
| 6 | Beban Usaha | Debit | Laporan Hasil Usaha |
| 7 | Pendapatan Lain-lain | Kredit | Laporan Hasil Usaha |
| 8 | Beban Lain-lain | Debit | Laporan Hasil Usaha |
| 9 | Beban Pajak Penghasilan | Debit | Laporan Hasil Usaha |

Akun *contra* (mis. akumulasi penyusutan, CKPN) memiliki saldo normal berlawanan dengan kelompoknya dan diberi
akhiran `09` pada level akun.

## 3. Bagan Akun Standar Koperasi (Template Default)

Kolom **Ext** = dapat diperluas dengan sub-akun tenant. **Unit** = unit usaha tempat akun relevan
(SP = simpan pinjam, TK = toko/sektor riil, U = umum). Akun yang tidak relevan dengan jenis koperasi disembunyikan
(bukan dihapus) oleh template.

### 1 — ASET

| Kode | Nama Akun | Saldo | Ext | Unit | Buku pembantu |
|---|---|---|---|---|---|
| **11** | **Aset Lancar** | | | | |
| 1101 | Kas | | | | |
| 110101 | Kas Besar (Kantor) | D | Ya | U | — |
| 110102 | Kas Kecil | D | Ya | U | — |
| 110103 | Kas Gerai / Kasir POS | D | Ya | TK | per shift |
| 110104 | Kas Teller Simpan Pinjam | D | Ya | SP | per teller |
| 110105 | Kas dalam Perjalanan (setoran kasir/teller belum diterima) | D | — | U | — |
| 1102 | Bank & Setara Kas | | | | |
| 110201 | Bank — Rekening Operasional | D | Ya (per rekening) | U | rekonsiliasi bank |
| 110202 | Deposito Berjangka ≤ 3 Bulan | D | Ya | U | — |
| 110203 | Dana di Payment Gateway (Kliring QRIS/VA) | D | Ya | U | per settlement |
| 1103 | Investasi Jangka Pendek | | | | |
| 110301 | Deposito Berjangka > 3 Bulan | D | Ya | U | — |
| 110302 | Simpanan pada Koperasi Lain / Sekunder | D | Ya | U | — |
| 1104 | Piutang Pinjaman Anggota | | | | |
| 110401 | Piutang Pinjaman Anggota — Pokok | D | Ya (per produk) | SP | per pinjaman |
| 110402 | Piutang Jasa/Bunga Pinjaman (Akrual) | D | — | SP | per pinjaman |
| 110403 | Piutang Pinjaman Calon Anggota / Koperasi Lain | D | Ya | SP | per pinjaman |
| 110409 | Cadangan Kerugian Penurunan Nilai Piutang Pinjaman (CKPN) | **K** | — | SP | per pinjaman (individual) |
| 1105 | Piutang Usaha | | | | |
| 110501 | Piutang Dagang Anggota (Kredit Belanja) | D | — | TK | per anggota |
| 110502 | Piutang Dagang Non-Anggota | D | — | TK | per pelanggan |
| 110503 | Piutang Potongan Gaji Instansi | D | Ya (per instansi) | U | per batch |
| 110509 | Cadangan Kerugian Piutang Usaha | **K** | — | TK | — |
| 1106 | Piutang Lain-lain | | | | |
| 110601 | Piutang Karyawan / Kasir | D | — | U | per karyawan |
| 110602 | Piutang Lain-lain | D | Ya | U | — |
| 1107 | Persediaan | | | | |
| 110701 | Persediaan Barang Dagang | D | Ya (per gudang) | TK | kartu stok |
| 110702 | Persediaan Barang dalam Perjalanan | D | — | TK | per transfer/PO |
| 110703 | Persediaan Perlengkapan / ATK | D | — | U | — |
| 1108 | Uang Muka & Beban Dibayar di Muka | | | | |
| 110801 | Uang Muka Pembelian | D | — | TK | per supplier |
| 110802 | Sewa Dibayar di Muka | D | Ya | U | — |
| 110803 | Asuransi Dibayar di Muka | D | Ya | U | — |
| 1109 | Pajak Dibayar di Muka | | | | |
| 110901 | PPN Masukan | D | — | U | — |
| 110902 | PPh Dibayar di Muka (Pasal 22/23/25) | D | Ya | U | — |
| **12** | **Investasi Jangka Panjang** | | | | |
| 1201 | Penyertaan | | | | |
| 120101 | Penyertaan pada Koperasi Sekunder | D | Ya | U | — |
| 120102 | Penyertaan pada Badan Usaha Lain | D | Ya | U | — |
| 1202 | Investasi Jangka Panjang Lainnya | | | | |
| 120201 | Deposito Jangka Panjang | D | Ya | U | — |
| **13** | **Aset Tetap** | | | | |
| 1301 | Tanah — 130101 Tanah | D | Ya | U | register aset |
| 1302 | Bangunan — 130201 Bangunan · 130209 Akumulasi Penyusutan Bangunan (K) | D/K | Ya | U | register aset |
| 1303 | Kendaraan — 130301 Kendaraan · 130309 Akumulasi Penyusutan Kendaraan (K) | D/K | Ya | U | register aset |
| 1304 | Peralatan & Mesin — 130401 Peralatan & Mesin · 130409 Akumulasi Penyusutan (K) | D/K | Ya | U | register aset |
| 1305 | Inventaris Kantor — 130501 Inventaris · 130509 Akumulasi Penyusutan (K) | D/K | Ya | U | register aset |
| 1306 | Aset Tetap dalam Penyelesaian — 130601 | D | Ya | U | — |
| **14** | **Aset Tak Berwujud & Aset Lain** | | | | |
| 1401 | Aset Tak Berwujud — 140101 Perangkat Lunak · 140109 Akumulasi Amortisasi (K) | D/K | Ya | U | register aset |
| 1402 | Aset Lain-lain — 140201 Uang Jaminan · 140202 Agunan yang Diambil Alih | D | Ya | U | — |

### 2 — LIABILITAS

| Kode | Nama Akun | Saldo | Ext | Unit | Buku pembantu |
|---|---|---|---|---|---|
| **21** | **Liabilitas Jangka Pendek** | | | | |
| 2101 | Simpanan Anggota | | | | |
| 210101 | Simpanan Sukarela | K | Ya (per produk) | SP | per rekening |
| 210102 | Simpanan Berjangka | K | Ya (per produk) | SP | per rekening |
| 210103 | Simpanan Khusus (Hari Raya, Pendidikan, dll.) | K | Ya | SP | per rekening |
| 210104 | Jasa Simpanan yang Masih Harus Dibayar | K | — | SP | per rekening |
| 2102 | Simpanan Calon Anggota & Koperasi Lain | | | | |
| 210201 | Simpanan Calon Anggota | K | — | SP | per rekening |
| 210202 | Simpanan Koperasi Lain | K | Ya | SP | per rekening |
| 2103 | Utang Usaha | | | | |
| 210301 | Utang Dagang (Supplier) | K | — | TK | per supplier |
| 210302 | Utang Konsinyasi (kepada Penitip) | K | — | TK | per penitip |
| 210303 | Utang Penerimaan Barang Belum Difaktur | K | — | TK | per GRN |
| 2104 | Utang Pajak | | | | |
| 210401 | PPN Keluaran | K | — | U | — |
| 210402 | Utang PPh Pasal 21 | K | — | U | — |
| 210403 | Utang PPh Pasal 23 | K | — | U | — |
| 210404 | Utang PPh Final atas Jasa Simpanan | K | — | SP | — |
| 210405 | Utang PPh Final UMKM | K | — | U | — |
| 210406 | Utang PPh Badan | K | — | U | — |
| 210407 | Utang PPh atas SHU Anggota | K | — | U | — |
| 2105 | Utang SHU & Dana-dana | | | | |
| 210501 | Utang SHU Bagian Anggota | K | — | U | per anggota |
| 210502 | Dana Pengurus & Pengawas | K | — | U | — |
| 210503 | Dana Karyawan | K | — | U | — |
| 210504 | Dana Pendidikan | K | — | U | — |
| 210505 | Dana Sosial | K | — | U | — |
| 210506 | Dana Pembangunan Daerah Kerja | K | — | U | — |
| 2106 | Pendapatan Diterima di Muka & Liabilitas Program | | | | |
| 210601 | Liabilitas Program Loyalitas (Poin) | K | — | TK | ledger poin |
| 210602 | Pendapatan Diterima di Muka | K | Ya | U | — |
| 210603 | Titipan Dana Anggota (kelebihan bayar/belum teralokasi) | K | — | U | per anggota |
| 2107 | Beban yang Masih Harus Dibayar | | | | |
| 210701 | Gaji & Tunjangan yang Masih Harus Dibayar | K | — | U | — |
| 210702 | Beban Lain yang Masih Harus Dibayar | K | Ya | U | — |
| 2108 | Utang Jangka Pendek kepada Lembaga Keuangan | | | | |
| 210801 | Utang Bank Jangka Pendek | K | Ya | U | — |
| 210802 | Bagian Lancar Utang Jangka Panjang | K | Ya | U | — |
| 2109 | Utang Lain-lain | | | | |
| 210901 | Titipan Potongan Gaji (belum dialokasikan) | K | Ya (per instansi) | U | per batch |
| 210902 | Utang Lain-lain | K | Ya | U | — |
| 210903 | Utang Pengembalian Simpanan Anggota Keluar | K | — | U | per anggota |
| **22** | **Liabilitas Jangka Panjang** | | | | |
| 2201 | Utang Bank Jangka Panjang — 220101 | K | Ya | U | — |
| 2202 | Pinjaman dari Lembaga Pemerintah (mis. LPDB) — 220201 | K | Ya | U | — |
| 2203 | Pinjaman dari Koperasi Sekunder — 220301 | K | Ya | U | — |
| 2204 | Liabilitas Imbalan Kerja — 220401 | K | — | U | — |

### 3 — EKUITAS

| Kode | Nama Akun | Saldo | Ext | Buku pembantu |
|---|---|---|---|---|
| **31** | **Modal Anggota** | | | |
| 310101 | Simpanan Pokok | K | — | per anggota |
| 310201 | Simpanan Wajib | K | — | per anggota |
| **32** | **Modal Penyertaan** — 320101 Modal Penyertaan | K | Ya | per pemodal |
| **33** | **Hibah / Donasi** — 330101 Hibah | K | Ya | per pemberi |
| **34** | **Cadangan** | | | |
| 340101 | Cadangan Umum | K | — | — |
| 340201 | Cadangan Tujuan Risiko | K | — | — |
| **35** | **Sisa Hasil Usaha** | | | |
| 350101 | SHU Tahun Berjalan | K | — | — |
| 350201 | SHU Belum Dibagi (Tahun Lalu) | K | — | — |
| 350301 | Ikhtisar Hasil Usaha (akun penutup sementara) | K/D | — | — |

> Tunggakan simpanan wajib **tidak** dicatat sebagai piutang di neraca; dipantau di modul simpanan (tagihan).
> Kebijakan ini dapat ditinjau akuntan bila koperasi memakai perlakuan berbeda.

### 4 — PENDAPATAN

| Kode | Nama Akun | Saldo | Ext | Pos LHU |
|---|---|---|---|---|
| **41** | **Pendapatan Usaha Simpan Pinjam** | | | |
| 410101 | Jasa Pinjaman Anggota | K | Ya (per produk) | Partisipasi Bruto Anggota |
| 410102 | Provisi & Administrasi Pinjaman Anggota | K | — | Partisipasi Bruto Anggota |
| 410103 | Denda Keterlambatan Anggota | K | — | Partisipasi Bruto Anggota |
| 410201 | Jasa Pinjaman Calon Anggota / Koperasi Lain | K | — | Pendapatan dari Non-Anggota |
| **42** | **Pendapatan Usaha Sektor Riil (Toko/Gerai/Jasa)** | | | |
| 420101 | Penjualan Barang kepada Anggota | K | Ya (per unit) | Partisipasi Bruto Anggota |
| 420201 | Penjualan Barang kepada Non-Anggota | K | Ya (per unit) | Penjualan Non-Anggota |
| 420301 | Pendapatan Komisi Konsinyasi | K | — | Pendapatan Usaha Lainnya |
| 420302 | Pendapatan Jasa (fotokopi, sewa, fee PPOB) | K | Ya | Pendapatan Usaha Lainnya |
| 420303 | Pendapatan Poin Kedaluwarsa | K | — | Pendapatan Usaha Lainnya |
| 4209 | Pengurang Pendapatan (*contra*) | | | |
| 420901 | Retur & Potongan Penjualan — Anggota | **D** | — | Pengurang Partisipasi Bruto |
| 420902 | Retur & Potongan Penjualan — Non-Anggota | **D** | — | Pengurang Penjualan Non-Anggota |
| 420903 | Alokasi Pendapatan ke Program Poin | **D** | — | Pengurang Partisipasi/Penjualan |

### 5 — BEBAN POKOK

| Kode | Nama Akun | Saldo | Pos LHU |
|---|---|---|---|
| **51** | **Harga Pokok Penjualan** | | |
| 510101 | HPP — Penjualan kepada Anggota | D | Beban Pokok Partisipasi |
| 510201 | HPP — Penjualan kepada Non-Anggota | D | HPP Non-Anggota |
| 510301 | Selisih & Penyesuaian Persediaan | D | Beban Pokok (gabungan) |
| **52** | **Beban Pokok Usaha Simpan Pinjam** | | |
| 520101 | Beban Jasa Simpanan Sukarela | D | Beban Pokok Partisipasi |
| 520102 | Beban Jasa Simpanan Berjangka | D | Beban Pokok Partisipasi |
| 520201 | Beban Bunga Pinjaman yang Diterima (Bank/LPDB/Sekunder) | D | Beban Pokok Partisipasi |

### 6 — BEBAN USAHA

| Kode | Nama Akun | Kode | Nama Akun |
|---|---|---|---|
| **6101** | **Beban Pegawai** | **6104** | **Beban Penyusutan & Amortisasi** |
| 610101 | Gaji & Upah | 610401 | Penyusutan Bangunan |
| 610102 | Tunjangan & THR | 610402 | Penyusutan Kendaraan |
| 610103 | BPJS Kesehatan & Ketenagakerjaan | 610403 | Penyusutan Peralatan & Mesin |
| 610104 | Honorarium Pengurus & Pengawas | 610404 | Penyusutan Inventaris |
| **6102** | **Beban Operasional Kantor** | 610405 | Amortisasi Aset Tak Berwujud |
| 610201 | Listrik, Air & Telepon | **6105** | **Beban Penyisihan Kerugian** |
| 610202 | Internet & Langganan Perangkat Lunak | 610501 | Beban CKPN Piutang Pinjaman |
| 610203 | ATK & Cetakan | 610502 | Beban Penyisihan Piutang Usaha |
| 610204 | Sewa | **6201** | **Beban Pemasaran & Penjualan** |
| 610205 | Pemeliharaan & Perbaikan | 620101 | Promosi |
| 610206 | Transportasi | 620102 | Pengemasan & Pengiriman |
| 610207 | Konsumsi | 620103 | Biaya MDR / Payment Gateway |
| **6103** | **Beban Organisasi** | 620104 | Selisih Kurang Kas Kasir (bila dibebankan) |
| 610301 | Beban Rapat Anggota (RAT) | | |
| 610302 | Beban Rapat Pengurus & Pengawas | | |
| 610303 | Pendidikan & Pelatihan (di luar Dana Pendidikan) | | |
| 610304 | Jasa Audit & Profesional | | |

### 7, 8, 9 — PENDAPATAN & BEBAN LAIN-LAIN, PAJAK

| Kode | Nama Akun | Saldo | Kode | Nama Akun | Saldo |
|---|---|---|---|---|---|
| 710101 | Jasa Giro / Bunga Bank | K | 810101 | Beban Administrasi Bank | D |
| 710102 | Pendapatan Sewa Aset | K | 810102 | Pajak atas Jasa Giro / Bunga Bank | D |
| 710103 | Laba Pelepasan Aset Tetap | K | 810103 | Rugi Pelepasan Aset Tetap | D |
| 710104 | Pemulihan Piutang yang Telah Dihapus | K | 810104 | Selisih Kurang Kas (non-kasir) | D |
| 710105 | Selisih Lebih Kas | K | 810105 | Selisih Pembulatan | D/K |
| 710106 | Pendapatan Lain-lain | K | 810106 | Beban Lain-lain | D |
| | | | 910101 | Beban Pajak Penghasilan | D |

## 4. Account Role (Pemetaan untuk Jurnal Otomatis)

Template posting merujuk **role**, bukan kode. Satu role dapat dipetakan berbeda per lingkup
(gerai, cabang, produk simpanan/pinjaman, metode bayar). Resolusi: lingkup paling spesifik → default tenant → default template.

| Account role | Default akun | Lingkup pemetaan |
|---|---|---|
| `KAS_GERAI` / `BAYAR_CASH` | 110103 | per gerai (sub-akun per gerai) |
| `KAS_TELLER` | 110104 | per teller/cabang |
| `KAS_BESAR` | 110101 | per cabang |
| `KAS_DALAM_PERJALANAN` | 110105 | — |
| `BANK_OPERASIONAL` / `BAYAR_TRANSFER` | 110201 | per rekening |
| `BAYAR_QRIS` / `BAYAR_VA` / `KLIRING_PG` | 110203 | per penyedia |
| `BAYAR_EDC` | 110203 | per bank EDC |
| `BAYAR_SAVING_BALANCE` | 210101 | per produk simpanan |
| `BAYAR_MEMBER_CREDIT` / `PIUTANG_DAGANG_ANGGOTA` | 110501 | — |
| `BAYAR_POINTS` / `LIABILITAS_POIN` | 210601 | — |
| `PIUTANG_POTONG_GAJI` | 110503 | per instansi |
| `TITIPAN_POTONG_GAJI` | 210901 | per instansi |
| `PIUTANG_PINJAMAN_POKOK` | 110401 | per produk pinjaman |
| `PIUTANG_JASA_PINJAMAN` | 110402 | per produk pinjaman |
| `CKPN_PINJAMAN` / `BEBAN_CKPN` | 110409 / 610501 | per produk pinjaman |
| `SIMPANAN_POKOK` / `SIMPANAN_WAJIB` | 310101 / 310201 | — |
| `SIMPANAN_SUKARELA` / `SIMPANAN_BERJANGKA` / `SIMPANAN_KHUSUS` | 210101 / 210102 / 210103 | per produk simpanan |
| `JASA_SIMPANAN_YMH_DIBAYAR` / `BEBAN_JASA_SIMPANAN` | 210104 / 520101 | per produk simpanan |
| `PENDAPATAN_JASA_PINJAMAN` / `PENDAPATAN_PROVISI` / `PENDAPATAN_DENDA` | 410101 / 410102 / 410103 | per produk pinjaman |
| `PENJUALAN_ANGGOTA` / `PENJUALAN_NON_ANGGOTA` | 420101 / 420201 | per unit usaha / kategori |
| `RETUR_ANGGOTA` / `RETUR_NON_ANGGOTA` | 420901 / 420902 | per unit usaha |
| `ALOKASI_PROGRAM_POIN` / `PENDAPATAN_POIN_KEDALUWARSA` | 420903 / 420303 | — |
| `HPP_ANGGOTA` / `HPP_NON_ANGGOTA` | 510101 / 510201 | per unit usaha / kategori |
| `PERSEDIAAN_BARANG_DAGANG` | 110701 | per gudang |
| `SELISIH_PERSEDIAAN` | 510301 | — |
| `UTANG_DAGANG` / `UTANG_KONSINYASI` / `PENDAPATAN_KOMISI_KONSINYASI` | 210301 / 210302 / 420301 | — |
| `UTANG_GRN_BELUM_DIFAKTUR` | 210303 | — |
| `PPN_KELUARAN` / `PPN_MASUKAN` | 210401 / 110901 | — |
| `UTANG_PPH_JASA_SIMPANAN` / `UTANG_PPH_SHU` | 210404 / 210407 | — |
| `SELISIH_KAS_LEBIH` / `SELISIH_KAS_KURANG` / `PIUTANG_KASIR` | 710105 / 620104 / 110601 | — |
| `BIAYA_MDR` | 620103 | per penyedia |
| `UTANG_SHU_ANGGOTA`, `DANA_PENGURUS`, `DANA_KARYAWAN`, `DANA_PENDIDIKAN`, `DANA_SOSIAL`, `DANA_PDK`, `CADANGAN_UMUM` | 210501–210506, 340101 | — |
| `SHU_TAHUN_BERJALAN` / `SHU_BELUM_DIBAGI` / `IKHTISAR_HASIL_USAHA` | 350101 / 350201 / 350301 | — |
| `UTANG_PENGEMBALIAN_SIMPANAN` | 210903 | — |
| `TITIPAN_ANGGOTA` | 210603 | — |
| `SELISIH_PEMBULATAN` | 810105 | — |

## 5. Matriks Posting (Jurnal Otomatis Default)

Notasi: **D** = debit, **K** = kredit. Nama dalam kurung adalah account role. Semua jurnal membawa dimensi
cabang, unit usaha, `participant_type`, dan referensi buku pembantu (anggota/rekening/pinjaman/supplier).

### 5.1 Keanggotaan & Simpanan

| # | Event | Debit | Kredit | Catatan |
|---|---|---|---|---|
| S1 | Setoran simpanan pokok (tunai) | Kas Teller (`KAS_TELLER`) | Simpanan Pokok 310101 | Saat calon menjadi anggota |
| S2 | Setoran simpanan wajib | Kas/Bank/Kliring PG | Simpanan Wajib 310201 | Alokasi ke tagihan bulan terkait |
| S3 | Setoran simpanan sukarela/khusus | Kas/Bank/Kliring PG | Simpanan Sukarela/Khusus 2101xx | — |
| S4 | Penarikan simpanan sukarela | Simpanan Sukarela 210101 | Kas Teller / Bank | Biaya admin (rule) → K Pendapatan Jasa 420302 |
| S5 | Akrual jasa simpanan (bulanan) | Beban Jasa Simpanan 5201xx | Jasa Simpanan YMH Dibayar 210104 | Job akhir bulan |
| S6 | Kapitalisasi jasa ke rekening | Jasa Simpanan YMH Dibayar 210104 | Simpanan Sukarela 210101 (neto) + Utang PPh Final Jasa Simpanan 210404 | Pajak sesuai rule `saving.tax.calculate` |
| S7 | Potong gaji — tagihan diterbitkan | Piutang Potongan Gaji Instansi 110503 | Titipan Potongan Gaji 210901 | Opsional (bila koperasi mengakui saat tagihan) |
| S8 | Potong gaji — dana diterima & dialokasikan | Bank 110201 | Simpanan Wajib / Piutang Pinjaman / Piutang Dagang Anggota sesuai alokasi; kelebihan → Titipan Anggota 210603 | Bila S7 dipakai: D Titipan 210901 / K Piutang 110503 |
| S9 | Anggota keluar — reklasifikasi | Simpanan Pokok 310101 + Simpanan Wajib 310201 | Utang Pengembalian Simpanan 210903 | Ekuitas → liabilitas saat keluar disetujui |
| S10 | Anggota keluar — kompensasi kewajiban | Utang Pengembalian Simpanan 210903 | Piutang Pinjaman 110401 / Piutang Dagang Anggota 110501 | — |
| S11 | Anggota keluar — pembayaran | Utang Pengembalian Simpanan 210903 + Simpanan Sukarela 210101 | Kas/Bank | — |

### 5.2 Pinjaman

| # | Event | Debit | Kredit | Catatan |
|---|---|---|---|---|
| L1 | Pencairan pinjaman | Piutang Pinjaman Pokok 110401 (nominal penuh) | Kas/Bank/Simpanan Sukarela (neto) + Pendapatan Provisi & Admin 410102 (potongan di muka) | Kebijakan amortisasi provisi dapat diatur akuntan |
| L2 | Akrual jasa pinjaman (bila kebijakan akrual) | Piutang Jasa Pinjaman 110402 | Jasa Pinjaman Anggota 410101 | Job akhir bulan untuk angsuran jatuh tempo |
| L3 | Pembayaran angsuran | Kas/Bank/Kliring PG/Simpanan Sukarela | Piutang Pinjaman Pokok 110401 + Jasa Pinjaman 410101 (atau Piutang Jasa 110402 bila sudah diakrual) + Denda 410103 | Alokasi sesuai `loan.payment.allocation_order` |
| L4 | Kelebihan bayar | Kas/Bank | Titipan Dana Anggota 210603 | Dipakai untuk angsuran berikutnya |
| L5 | Pembentukan/penambahan CKPN | Beban CKPN 610501 | CKPN Piutang Pinjaman 110409 | Selisih kebutuhan CKPN vs saldo |
| L6 | Pemulihan (penurunan) CKPN | CKPN 110409 | Beban CKPN 610501 | — |
| L7 | Hapus buku pinjaman | CKPN 110409 | Piutang Pinjaman Pokok 110401 | Butuh persetujuan pengurus |
| L8 | Penerimaan atas pinjaman yang dihapus buku | Kas/Bank | Pemulihan Piutang Dihapus 710104 | — |
| L9 | Pelunasan dipercepat dengan diskon jasa | Kas/Bank | Piutang Pinjaman 110401 + Jasa Pinjaman 410101 (setelah diskon) | Diskon tidak dijurnal terpisah (jasa diakui neto) |

### 5.3 POS, Penjualan & Loyalty

| # | Event | Debit | Kredit | Catatan |
|---|---|---|---|---|
| P1 | Penjualan (per metode bayar) | `BAYAR_CASH` / `BAYAR_QRIS` / `BAYAR_EDC` / `BAYAR_SAVING_BALANCE` (210101) / `BAYAR_MEMBER_CREDIT` (110501) / `BAYAR_POINTS` (210601) | Penjualan Anggota 420101 **atau** Non-Anggota 420201 (neto setelah diskon) + PPN Keluaran 210401 (bila PKP) | Lihat template §8.8 dokumen 03 |
| P2 | HPP penjualan | HPP Anggota 510101 / Non-Anggota 510201 | Persediaan 110701 | Rata-rata bergerak; barang konsinyasi tidak ber-HPP |
| P3 | Penjualan barang konsinyasi | (termasuk dalam P1) | Utang Konsinyasi 210302 (porsi penitip) + Komisi Konsinyasi 420301 | — |
| P4 | Poin diterbitkan | Alokasi Pendapatan ke Program Poin 420903 | Liabilitas Program Loyalitas 210601 | Nilai = poin × nilai tukar Rp per poin (rule `loyalty.points.redeem_value`); estimasi tingkat penukaran dapat diterapkan akuntan sebagai kebijakan lanjutan |
| P5 | Poin ditukar | Liabilitas Program Loyalitas 210601 | (sebagai metode bayar di P1) | — |
| P6 | Poin kedaluwarsa | Liabilitas Program Loyalitas 210601 | Pendapatan Poin Kedaluwarsa 420303 | Job harian |
| P7 | Retur penjualan | Retur & Potongan Anggota 420901 / Non-Anggota 420902 + PPN Keluaran | Kas / Simpanan Sukarela / Piutang Dagang Anggota | Plus pembalikan HPP: D Persediaan / K HPP |
| P8 | Settlement QRIS/EDC dari payment gateway | Bank 110201 + Biaya MDR 620103 | Kliring PG 110203 | Dari webhook/laporan settlement |
| P9 | Tutup shift — selisih lebih | Kas Gerai 110103 | Selisih Lebih Kas 710105 | — |
| P10 | Tutup shift — selisih kurang | Selisih Kurang Kas Kasir 620104 **atau** Piutang Kasir 110601 | Kas Gerai 110103 | Sesuai rule `pos.cash_difference.policy` |
| P11 | Setoran kas gerai ke kas besar/bank | Kas Dalam Perjalanan 110105 → Kas Besar/Bank | Kas Gerai 110103 → Kas Dalam Perjalanan 110105 | Dua langkah: kirim & terima |
| P12 | Tagihan kredit belanja dikirim ke instansi | Piutang Potongan Gaji Instansi 110503 | Piutang Dagang Anggota 110501 | Bagian dari batch potong gaji |
| P13 | Kredit belanja dikonversi menjadi pinjaman cicilan | Piutang Pinjaman Pokok 110401 | Piutang Dagang Anggota 110501 | LON-12 |

> **Granularitas posting POS** diatur rule `pos.posting_granularity`: `PER_TRANSAKSI` (satu jurnal per struk) atau
> `RINGKAS_PER_SHIFT` (satu jurnal per shift dikelompokkan per akun & dimensi, rincian tetap di buku pembantu dan
> dapat di-*drill-down* ke struk). Default: `RINGKAS_PER_SHIFT` untuk efisiensi.

### 5.4 Pembelian & Persediaan

| # | Event | Debit | Kredit | Catatan |
|---|---|---|---|---|
| B1 | Penerimaan barang (GRN) dengan faktur | Persediaan 110701 + PPN Masukan 110901 | Utang Dagang 210301 | Bila faktur menyusul: K Utang Penerimaan Barang Belum Difaktur 210303, lalu direklasifikasi ke 210301 saat faktur diterima |
| B2 | Pembayaran supplier | Utang Dagang 210301 | Kas/Bank | — |
| B3 | Uang muka pembelian | Uang Muka Pembelian 110801 | Kas/Bank | Dikompensasi saat faktur |
| B4 | Retur pembelian | Utang Dagang 210301 | Persediaan 110701 + PPN Masukan | — |
| B5 | Stok opname — kurang | Selisih Persediaan 510301 | Persediaan 110701 | Butuh persetujuan |
| B6 | Stok opname — lebih | Persediaan 110701 | Selisih Persediaan 510301 | — |
| B7 | Pembayaran ke penitip konsinyasi | Utang Konsinyasi 210302 | Kas/Bank | Penitip anggota dapat dibayar ke simpanan sukarela |
| B8 | Pembelian hasil produksi anggota | Persediaan 110701 | Kas/Bank/Simpanan Sukarela/Utang Dagang | Nilai transaksi dicatat sebagai dasar jasa usaha SHU (bila diatur) |

### 5.5 Kas, Bank, Aset Tetap & Lainnya

| # | Event | Debit | Kredit |
|---|---|---|---|
| K1 | Pengeluaran kas operasional | Beban 6xxxxx | Kas/Bank |
| K2 | Pengisian kas kecil (imprest) | Kas Kecil 110102 | Kas Besar/Bank |
| K3 | Transfer antar kas/bank | Kas/Bank tujuan | Kas/Bank asal |
| K4 | Pembelian aset tetap | Aset Tetap 13xx01 | Kas/Bank/Utang |
| K5 | Penyusutan bulanan | Beban Penyusutan 6104xx | Akumulasi Penyusutan 13xx09 |
| K6 | Pelepasan aset | Kas + Akumulasi Penyusutan (+ Rugi 810103) | Aset Tetap (+ Laba 710103) |
| K7 | Jasa giro & pajaknya | Bank + Pajak Jasa Giro 810102 | Jasa Giro 710101 |
| K8 | Pembayaran pajak | Utang Pajak 2104xx | Bank |
| K9 | Gaji & honor | Beban Pegawai 6101xx | Kas/Bank + Utang PPh 21 210402 |

### 5.6 Tutup Buku Tahunan & SHU

| # | Event | Debit | Kredit | Catatan |
|---|---|---|---|---|
| Y1 | Menutup akun pendapatan | Semua akun 4xxxxx & 7xxxxx (saldo kredit) | Ikhtisar Hasil Usaha 350301 | Akun *contra* pendapatan ditutup sebaliknya |
| Y2 | Menutup akun beban | Ikhtisar Hasil Usaha 350301 | Semua akun 5xxxxx, 6xxxxx, 8xxxxx, 9xxxxx | — |
| Y3 | Memindahkan hasil usaha | Ikhtisar Hasil Usaha 350301 | SHU Tahun Berjalan 350101 | Bila rugi: sebaliknya |
| Y4 | Awal tahun berikut | SHU Tahun Berjalan 350101 | SHU Belum Dibagi 350201 | Menunggu keputusan RAT |
| Y5 | Penetapan pembagian SHU (setelah RAT) | SHU Belum Dibagi 350201 | Cadangan Umum 340101 + Utang SHU Anggota 210501 (jasa modal + jasa usaha) + Dana Pengurus 210502 + Dana Karyawan 210503 + Dana Pendidikan 210504 + Dana Sosial 210505 + Dana PDK 210506 | Alokasi sesuai rule `shu.allocation` versi yang ditetapkan RAT |
| Y6 | Pembayaran SHU anggota | Utang SHU Anggota 210501 | Kas/Bank atau Simpanan Sukarela 210101 + Utang PPh atas SHU 210407 | Sesuai `shu.tax` |
| Y7 | Pemakaian dana (mis. pendidikan) | Dana Pendidikan 210504 | Kas/Bank | — |

## 6. Struktur Laporan Keuangan (Template Default)

Template laporan berupa daftar **pos laporan** (`report_line`) dengan urutan, induk, rumus subtotal, dan tanda.
Setiap akun level 4/5 menunjuk satu pos. Template berversi dan dapat diperbarui platform bila regulasi berubah.

### 6.1 Laporan Posisi Keuangan (Neraca) — ringkas

```text
ASET
  Aset Lancar
    Kas dan Setara Kas ............................ 1101, 1102
    Investasi Jangka Pendek ....................... 1103
    Piutang Pinjaman Anggota (neto CKPN) .......... 1104
    Piutang Usaha (neto) .......................... 1105
    Piutang Lain-lain ............................. 1106
    Persediaan .................................... 1107
    Uang Muka & Beban Dibayar di Muka ............. 1108
    Pajak Dibayar di Muka ......................... 1109
  Aset Tidak Lancar
    Investasi Jangka Panjang / Penyertaan ......... 12
    Aset Tetap (neto akumulasi penyusutan) ........ 13
    Aset Tak Berwujud & Aset Lain ................. 14
JUMLAH ASET

LIABILITAS
  Liabilitas Jangka Pendek
    Simpanan Anggota (sukarela, berjangka, khusus)  2101
    Simpanan Calon Anggota & Koperasi Lain ........ 2102
    Utang Usaha ................................... 2103
    Utang Pajak ................................... 2104
    Utang SHU & Dana-dana ......................... 2105
    Pendapatan Diterima di Muka & Liabilitas Poin . 2106
    Beban YMH Dibayar ............................. 2107
    Utang Jangka Pendek Lembaga Keuangan .......... 2108
    Utang Lain-lain ............................... 2109
  Liabilitas Jangka Panjang ....................... 22
EKUITAS
    Simpanan Pokok ................................ 310101
    Simpanan Wajib ................................ 310201
    Modal Penyertaan .............................. 32
    Hibah ......................................... 33
    Cadangan ...................................... 34
    SHU Belum Dibagi .............................. 350201
    SHU Tahun Berjalan ............................ 350101 (+ laba/rugi berjalan sebelum tutup buku)
JUMLAH LIABILITAS DAN EKUITAS
```

### 6.2 Laporan Hasil Usaha (Laba Rugi Koperasi) — ringkas

```text
A. PARTISIPASI ANGGOTA
   Partisipasi Bruto Anggota
     Penjualan kepada anggota (neto retur/potongan/alokasi poin)  4201 - 420901 - porsi 420903
     Jasa pinjaman, provisi, denda anggota ........................ 4101
   Beban Pokok Partisipasi
     HPP penjualan kepada anggota ................................. 510101
     Beban jasa simpanan & bunga pinjaman yang diterima ........... 52
   PARTISIPASI NETO ANGGOTA (A)

B. PENDAPATAN DARI NON-ANGGOTA
   Penjualan kepada non-anggota (neto) ............................ 4202 - 420902 - porsi 420903
   Jasa pinjaman calon anggota/koperasi lain ...................... 4102
   HPP non-anggota ................................................ 510201
   LABA KOTOR NON-ANGGOTA (B)

C. PENDAPATAN USAHA LAINNYA (komisi konsinyasi, jasa, poin kedaluwarsa)  4203
D. SELISIH PERSEDIAAN ............................................. 510301

SISA HASIL USAHA KOTOR (A + B + C - D)
E. BEBAN USAHA (pegawai, operasional, organisasi, penyusutan, penyisihan, pemasaran)  6
SISA HASIL USAHA DARI OPERASI
F. PENDAPATAN (BEBAN) LAIN-LAIN ................................... 7 - 8
SHU SEBELUM PAJAK
G. BEBAN PAJAK PENGHASILAN ........................................ 9
SHU SETELAH PAJAK (SHU BERSIH)
```

> Label dan susunan akhir (mis. "Laporan Hasil Usaha" vs "Laporan Laba Rugi dan Penghasilan Komprehensif Lain")
> mengikuti lampiran Permenkop UKM No. 2/2024 dan hasil validasi akuntan di Fase 0. Template mendukung keduanya.

### 6.3 Laporan Lainnya

| Laporan | Sumber | Catatan |
|---|---|---|
| Laporan Perubahan Ekuitas | Mutasi akun kelompok 3 per komponen | Kolom: simpanan pokok, wajib, penyertaan, hibah, cadangan, SHU |
| Laporan Arus Kas — metode langsung | Baris jurnal akun kas/bank diklasifikasikan via `cashflow_class` akun lawan | Operasi (penerimaan dari anggota/non-anggota, pembayaran ke pemasok/karyawan, penyaluran & penerimaan pinjaman), investasi, pendanaan (simpanan pokok/wajib, pinjaman diterima, pembagian SHU) |
| Laporan Arus Kas — metode tidak langsung | SHU + penyesuaian non-kas + perubahan modal kerja | Untuk pemeriksaan silang |
| Catatan atas Laporan Keuangan (CaLK) | Template naratif + tabel rincian otomatis (rincian kas, piutang & CKPN per kolektibilitas, persediaan, aset tetap, simpanan, SHU) | Diedit akuntan sebelum finalisasi |
| Neraca Saldo & Neraca Lajur | Saldo per akun | Neraca lajur 10 kolom |
| Laporan per Unit Usaha & Konsolidasi | Dimensi `business_unit_id` | Dengan eliminasi antar unit |

## 7. Aturan Validasi COA & Posting

1. Hanya akun **level 4 atau 5** yang dapat menerima posting; akun header ditolak.
2. Akun yang memiliki sub-akun tidak dapat menerima posting langsung (posting harus ke sub-akun).
3. Akun kontrol buku pembantu (mis. 110401, 210101, 310101, 310201, 110501, 210301, 110701) **tidak dapat dipakai di jurnal manual**
   kecuali melalui dokumen modul terkait atau jurnal penyesuaian khusus yang menyebut referensi buku pembantu.
4. Akun tidak dapat dihapus bila pernah memiliki mutasi; hanya dapat dinonaktifkan untuk posting baru.
5. Perubahan pemetaan account role berlaku untuk jurnal **sejak tanggal berlaku**; jurnal lama tidak berubah.
6. Setiap template posting wajib memiliki minimal satu test case seimbang dan seluruh role yang dipakai sudah terpetakan
   sebelum dapat dipublikasikan.
7. Posting ke periode `CLOSED` ditolak; perilaku transaksi offline yang terlambat mengikuti rule `period.late_transaction.policy`.

[← 03 Rule Engine](./03-rule-engine.md) · [Lanjut ke 05 — Flowchart →](./05-flowchart.md)
