# 02 — Daftar Fitur

[← Kembali ke daftar isi](./README.md)

---

## Legenda

**Prioritas**

| Kode | Arti |
|---|---|
| **P0** | Wajib ada untuk rilis fase terkait; tanpa ini produk tidak layak dipakai |
| **P1** | Penting; masuk fase terkait bila kapasitas memungkinkan, paling lambat fase berikutnya |
| **P2** | Nilai tambah; dijadwalkan setelah v1 atau bila ada permintaan pasar |

**Fase** (detail jadwal di [08 — Task Todo](./08-task-todo.md))

| Kode | Fase | Minggu |
|---|---|---|
| F0 | Fondasi platform | 1–4 |
| F1 | MVP Inti: keanggotaan, simpanan, akuntansi inti, posting engine, rule engine v1 | 5–16 |
| F2 | POS, inventori, loyalty, rule engine v2 (ECA) | 17–26 |
| F3 | Pinjaman, workflow persetujuan, aplikasi anggota, pembayaran digital | 27–36 |
| F4 | Tutup buku tahunan, SHU & RAT, pelaporan lanjut, aset tetap, pajak | 37–44 |
| F5 | Hardening, pilot & go-live | 45–52 |
| L | Lanjutan (setelah v1) | — |

**Ringkasan modul**

| Kode | Modul | Jumlah fitur |
|---|---|---|
| PLT | Platform SaaS & Multi-Tenant | 11 |
| IAM | Identitas, Akses & Audit | 10 |
| ORG | Organisasi Koperasi | 6 |
| MBR | Keanggotaan | 12 |
| LOY | Loyalty & Poin | 7 |
| SAV | Simpanan | 12 |
| LON | Pinjaman | 14 |
| POS | Point of Sale | 14 |
| INV | Inventori & Pembelian | 10 |
| ACC | Akuntansi | 17 |
| SHU | SHU & RAT | 7 |
| RUL | Rule Engine | 15 |
| RPT | Laporan & Dashboard | 7 |
| NTF | Notifikasi | 4 |
| INT | Integrasi | 7 |
| APP | Aplikasi Anggota | 9 |

---

## 1. PLT — Platform SaaS & Multi-Tenant

| ID | Fitur | Deskripsi & kriteria utama | Prio | Fase |
|---|---|---|---|---|
| PLT-01 | Onboarding tenant (wizard) | Data badan hukum (no. & tgl akta/SK, NIK koperasi, NPWP), jenis koperasi, pola (konvensional), unit usaha, cabang, tahun buku; pilih **template COA** & **rule pack** sesuai jenis; undang admin pertama | P0 | F0 |
| PLT-02 | Paket langganan & feature flag | Paket (mis. Dasar: anggota+simpanan+akuntansi; Usaha: +POS+pinjaman; Enterprise: +multi cabang, API, silo DB); fitur & kuota dikendalikan *feature flag* per tenant | P0 | F0 |
| PLT-03 | Billing langganan | Tagihan otomatis, tautan pembayaran, masa tenggang, suspend/aktif ulang | P1 | F4 |
| PLT-04 | Subdomain & white-label | `namakoperasi.domain`, logo, warna, nama di struk, aplikasi anggota bertema koperasi | P1 | F2 |
| PLT-05 | Isolasi data tenant | Tenant context middleware + RLS; tes negatif lintas tenant di CI | P0 | F0 |
| PLT-06 | Monitoring penggunaan & kuota | Jumlah anggota, transaksi, storage, pengguna per tenant; peringatan mendekati kuota | P1 | F4 |
| PLT-07 | Impersonasi dukungan | Support masuk sebagai tenant hanya dengan persetujuan admin tenant, berbatas waktu, semua aksi diaudit & diberi banner | P1 | F3 |
| PLT-08 | Ekspor & penghapusan data tenant | Ekspor penuh (CSV/JSON + PDF laporan) untuk portabilitas; penghapusan dengan masa retensi dokumen keuangan 10 tahun (arsip terenkripsi) | P1 | F4 |
| PLT-09 | Pustaka template global | Super admin mengelola versi template COA, template laporan, rule pack, template dokumen (akad, struk); tenant menerima notifikasi pembaruan & memilih adopsi | P0 | F0 |
| PLT-10 | Pengumuman & changelog | Broadcast ke admin tenant, catatan rilis di dalam aplikasi | P2 | L |
| PLT-11 | Backup & restore logis per tenant | Snapshot logis per tenant untuk pemulihan kesalahan pengguna (di samping PITR global) | P1 | F5 |

## 2. IAM — Identitas, Akses & Audit

| ID | Fitur | Deskripsi & kriteria utama | Prio | Fase |
|---|---|---|---|---|
| IAM-01 | Login & 2FA | Email/no. HP + kata sandi; TOTP 2FA wajib untuk peran keuangan & admin; pemilihan tenant bila pengguna terdaftar di >1 tenant | P0 | F0 |
| IAM-02 | Manajemen pengguna | Undang via email/WA, aktif/nonaktif, reset kata sandi, satu pengguna dapat bergabung di banyak tenant | P0 | F0 |
| IAM-03 | Peran & izin (RBAC) | Peran bawaan (Admin, Ketua, Sekretaris, Bendahara, Pengawas, Akuntan, Teller, Kasir, Gudang, Kolektor, Auditor) + peran kustom; matriks izin per modul × aksi (lihat, buat, ubah, setujui, posting, ekspor) | P0 | F0 |
| IAM-04 | Lingkup data | Izin dibatasi per cabang / unit usaha / gerai / gudang | P0 | F0 |
| IAM-05 | Maker-checker | Pembuat ≠ penyetuju untuk: jurnal manual, pencairan pinjaman, penyesuaian stok, publikasi rule, perubahan template posting; ambang nominal diatur rule | P0 | F1 |
| IAM-06 | Audit trail | Log *append-only* dengan rantai hash: pengguna, aksi, entitas, nilai sebelum/sesudah, IP, perangkat; dapat difilter & diekspor oleh pengawas/auditor | P0 | F0 |
| IAM-07 | Sesi & perangkat | Daftar sesi aktif, cabut sesi; registrasi & pairing perangkat POS dengan kode sekali pakai | P0 | F0 |
| IAM-08 | Login anggota | OTP WhatsApp/SMS + PIN 6 digit/biometrik di aplikasi anggota | P1 | F3 |
| IAM-09 | SSO perusahaan | OIDC/SAML untuk koperasi karyawan korporasi | P2 | L |
| IAM-10 | Kebijakan keamanan | Kebijakan kata sandi, lockout, batas waktu sesi, IP allowlist opsional untuk back-office | P1 | F1 |

## 3. ORG — Organisasi Koperasi

| ID | Fitur | Deskripsi & kriteria utama | Prio | Fase |
|---|---|---|---|---|
| ORG-01 | Profil koperasi | Nama, badan hukum, NIK koperasi, NPWP, alamat, jenis, kerangka akuntansi (SAK EP/EMKM), status PKP | P0 | F0 |
| ORG-02 | Pengurus & pengawas | Daftar jabatan, periode, nomor SK/keputusan RAT; dipakai sebagai penyetuju di workflow | P1 | F1 |
| ORG-03 | Cabang & unit usaha | Kantor/cabang, unit usaha (toko, simpan pinjam, apotek, jasa, logistik); menjadi **dimensi akuntansi** | P0 | F0 |
| ORG-04 | Tahun buku & periode | Tahun buku, 12 periode bulanan, status `OPEN`/`SOFT_CLOSED`/`CLOSED` | P0 | F1 |
| ORG-05 | Dokumen AD/ART & peraturan khusus | Unggah dokumen, tautkan pasal ke rule terkait (rule kategori AD/ART) | P1 | F1 |
| ORG-06 | Pengaturan umum | Zona waktu, format nomor dokumen, kebijakan pembulatan, hari kerja/libur (untuk jatuh tempo) | P0 | F0 |

## 4. MBR — Keanggotaan

| ID | Fitur | Deskripsi & kriteria utama | Prio | Fase |
|---|---|---|---|---|
| MBR-01 | Registrasi calon anggota | NIK (validasi 16 digit & cek duplikat dalam tenant via hash), No. KK, nama, TTL, alamat, pekerjaan, instansi/kelompok, penghasilan, no. HP, foto, foto KTP, tanda tangan, ahli waris | P0 | F1 |
| MBR-02 | Persetujuan & aktivasi | Calon → aktif setelah syarat rule terpenuhi (mis. simpanan pokok lunas) dan disetujui pengurus; alasan penolakan tercatat | P0 | F1 |
| MBR-03 | Nomor anggota otomatis | Format dapat diatur (mis. `A-{CABANG}-{YYYY}-{SEQ:5}`) | P0 | F1 |
| MBR-04 | Kartu anggota digital | QR berisi token (bukan NIK), dapat dicetak (PDF ukuran kartu) & tampil di aplikasi anggota; token dapat dirotasi bila kartu hilang | P0 | F1 |
| MBR-05 | Siklus status anggota | Calon, Aktif, Pasif (rule: mis. 6 bulan tidak bayar simpanan wajib), Diberhentikan, Keluar, Meninggal; riwayat status | P0 | F1 |
| MBR-06 | Keluar anggota & penyelesaian | Hitung hak (simpanan pokok, wajib, sukarela, SHU terutang) dikurangi kewajiban (sisa pinjaman, piutang belanja); jurnal reklasifikasi & pembayaran; ahli waris untuk anggota meninggal | P0 | F1 |
| MBR-07 | Profil 360° | Satu layar: data diri, simpanan, pinjaman, belanja, poin, SHU, dokumen, tunggakan, riwayat aktivitas | P0 | F1 |
| MBR-08 | Kelompok anggota | Instansi/departemen (untuk potong gaji), dusun/RW (Kopdes), kelompok tani; penanggung jawab kelompok | P1 | F1 |
| MBR-09 | Import massal | Template Excel, validasi per baris, mode *dry run*, laporan error | P0 | F1 |
| MBR-10 | Consent & privasi | Persetujuan pemrosesan data (versi kebijakan privasi), masking NIK/KK di UI sesuai izin, permintaan akses/koreksi/hapus data | P0 | F1 |
| MBR-11 | Tier keanggotaan | Reguler/Silver/Gold (nama bebas) berdasarkan rule (mis. total belanja 12 bulan, masa keanggotaan) | P1 | F2 |
| MBR-12 | Pelanggan non-anggota terdaftar | Data minimal (nama, no. HP) untuk poin; dapat dikonversi menjadi calon anggota | P1 | F2 |

## 5. LOY — Loyalty & Poin

| ID | Fitur | Deskripsi & kriteria utama | Prio | Fase |
|---|---|---|---|---|
| LOY-01 | Perolehan poin (earn) | Dihitung oleh rule hook `loyalty.points.earn` (mis. 1 poin per Rp10.000, multiplier per tier/kategori/hari) | P1 | F2 |
| LOY-02 | Penukaran poin (redeem) | Poin sebagai metode bayar di POS dengan nilai tukar per rule; minimal poin; validasi saldo | P1 | F2 |
| LOY-03 | Kedaluwarsa poin | Bucket FIFO, job harian, notifikasi H-30 sebelum kedaluwarsa | P1 | F2 |
| LOY-04 | Tier & benefit | Diskon tier, multiplier poin, evaluasi tier bulanan | P1 | F2 |
| LOY-05 | Voucher & promo kode | Kode voucher, kuota, periode, segmen | P2 | L |
| LOY-06 | Ledger poin & akuntansi | Ledger imutabel; penerbitan poin membentuk liabilitas program loyalitas, penukaran mengakui pendapatan, kedaluwarsa diakui sebagai pendapatan | P1 | F2 |
| LOY-07 | Poin non-belanja | Poin via rule ECA, mis. bayar simpanan wajib tepat waktu, hadir RAT | P2 | L |

## 6. SAV — Simpanan

| ID | Fitur | Deskripsi & kriteria utama | Prio | Fase |
|---|---|---|---|---|
| SAV-01 | Produk simpanan | Jenis (Pokok, Wajib, Sukarela, Berjangka, Khusus), klasifikasi akuntansi (Ekuitas/Liabilitas), setoran minimal, saldo minimal, biaya admin, rule jasa | P0 | F1 |
| SAV-02 | Rekening simpanan | Rekening pokok & wajib dibuka otomatis saat calon anggota disetujui; sukarela dibuka sesuai permintaan | P0 | F1 |
| SAV-03 | Teller setor/tarik | Tunai/transfer; slip/bukti cetak & WA; validasi rule (saldo minimal, limit harian, rekening dibekukan); jurnal otomatis | P0 | F1 |
| SAV-04 | Tagihan simpanan wajib | Generate tagihan periodik; status lunas/sebagian/tunggak; denda via rule; pembayaran rapel | P0 | F1 |
| SAV-05 | Setoran kolektif | Potong gaji per instansi (ekspor tagihan → impor realisasi), setoran kolektor lapangan, alokasi otomatis ke tagihan | P0 | F1 |
| SAV-06 | Jasa simpanan | Rule metode (saldo harian/terendah/rata-rata), tarif bertingkat (tabel keputusan), akrual harian, kapitalisasi bulanan, potong pajak final sesuai rule | P1 | F1 |
| SAV-07 | Simpanan berjangka | Tenor, tarif, ARO, jasa dibayar bulanan/saat jatuh tempo, pencairan dini dengan penalti (rule), bilyet PDF | P1 | F3 |
| SAV-08 | Mutasi & buku simpanan | Rekening koran per periode, cetak buku tabungan (printer passbook opsional) | P0 | F1 |
| SAV-09 | Transfer & pembayaran dari saldo | Transfer antar rekening anggota; bayar belanja POS & angsuran dari saldo sukarela | P1 | F2 |
| SAV-10 | Penutupan rekening | Tutup rekening sukarela/berjangka; pokok & wajib hanya saat keluar anggota | P0 | F1 |
| SAV-11 | Autodebet | Simpanan wajib & angsuran didebet otomatis dari saldo sukarela pada tanggal tertentu (rule) | P1 | F3 |
| SAV-12 | Setoran digital | VA/QRIS per anggota; webhook → posting otomatis & notifikasi | P1 | F3 |

## 7. LON — Pinjaman

| ID | Fitur | Deskripsi & kriteria utama | Prio | Fase |
|---|---|---|---|---|
| LON-01 | Produk pinjaman | Metode bunga (flat, efektif/menurun, anuitas), tarif, tenor min/maks, biaya provisi/admin, denda, urutan alokasi pembayaran, rule plafon & kelayakan, workflow persetujuan | P0 | F3 |
| LON-02 | Pengajuan & simulasi | Pengajuan via teller atau aplikasi anggota; simulasi jadwal angsuran; unggah dokumen | P0 | F3 |
| LON-03 | Kelayakan & plafon | Hook `loan.eligibility` & `loan.plafond` (mis. ≤ 3× total simpanan, angsuran ≤ 40% penghasilan, keanggotaan ≥ 6 bulan, tanpa tunggakan); hasil disertai **penjelasan** | P0 | F3 |
| LON-04 | Persetujuan berjenjang | Rule workflow berdasarkan nominal/produk/risiko; SLA & eskalasi; catatan komite | P0 | F3 |
| LON-05 | Akad/perjanjian | Template dokumen dengan variabel; PDF; unggah dokumen bertanda tangan; e-sign/e-meterai (P2) | P0 | F3 |
| LON-06 | Pencairan | Tunai/transfer/ke rekening sukarela; potongan biaya di muka; maker-checker | P0 | F3 |
| LON-07 | Pembayaran angsuran | Teller, potong gaji, autodebet, VA/QRIS, kolektor; alokasi denda → bunga → pokok (dapat diubah); kelebihan bayar ke titipan | P0 | F3 |
| LON-08 | Denda & pelunasan | Denda harian/bulanan via rule; pelunasan dipercepat dengan diskon bunga sesuai rule; pembayaran sebagian | P0 | F3 |
| LON-09 | Kolektibilitas & CKPN | Klasifikasi harian berdasarkan DPD (kategori & ambang dapat diatur), CKPN bulanan per kategori atau individual; jurnal otomatis | P0 | F3 |
| LON-10 | Restrukturisasi & hapus buku | Penjadwalan ulang dengan persetujuan; hapus buku dari CKPN; pencatatan pemulihan | P1 | F3 |
| LON-11 | Agunan | Registrasi jaminan (BPKB, sertifikat, dsb.), nilai taksasi, dokumen, status pengembalian | P1 | F3 |
| LON-12 | Kredit barang | Belanja di POS dikonversi menjadi pinjaman cicilan (produk khusus) | P1 | F3 |
| LON-13 | Kolektor lapangan | Daftar tagih harian, rute, setoran kolektor, rekonsiliasi dengan kas | P2 | L |
| LON-14 | Laporan portofolio | Outstanding, *aging*, NPL, kolektibilitas, proyeksi arus kas masuk, pinjaman jatuh tempo | P0 | F3 |

## 8. POS — Point of Sale

| ID | Fitur | Deskripsi & kriteria utama | Prio | Fase |
|---|---|---|---|---|
| POS-01 | Kasir | Scan barcode/cari nama/SKU, keranjang, ubah qty/satuan, diskon item & transaksi (di atas batas → PIN supervisor) | P0 | F2 |
| POS-02 | Identifikasi anggota | Scan QR kartu / no. anggota / no. HP → tampil nama, tier, poin, saldo sukarela, sisa limit kredit; harga anggota berlaku | P0 | F2 |
| POS-03 | Harga & promo | Daftar harga anggota/umum/grosir, harga bertingkat per qty, promo via rule `pos.price.resolve` & `pos.discount.apply` | P0 | F2 |
| POS-04 | Multi pembayaran (split) | Tunai, QRIS, EDC debit/kredit (input ref), transfer, **saldo simpanan sukarela**, **kredit anggota/potong gaji**, **poin**, voucher | P0 | F2 |
| POS-05 | Shift kasir | Buka shift (modal awal), tutup shift (hitung per pecahan), selisih kas → jurnal (rule: dibebankan atau piutang kasir), setoran ke kas besar | P0 | F2 |
| POS-06 | Struk | Printer thermal 58/80 mm (ESC/POS via Bluetooth/USB/LAN), cetak ulang, struk digital via WA | P0 | F2 |
| POS-07 | Retur & void | Retur dengan referensi struk, void sebelum tutup shift, keduanya butuh otorisasi; stok & jurnal dibalik | P0 | F2 |
| POS-08 | Mode offline | Transaksi tetap jalan tanpa internet; cache produk, harga, rule, anggota; batas transaksi offline; sinkron otomatis idempoten | P0 | F2 |
| POS-09 | Tahan transaksi & multi terminal | *Park/hold* keranjang; beberapa terminal per gerai | P1 | F2 |
| POS-10 | Konsinyasi | Barang titipan (sering milik anggota produsen); penjualan membentuk utang ke penitip & pendapatan komisi; laporan & pembayaran ke penitip | P1 | F2 |
| POS-11 | Limit kredit anggota | Rule `pos.credit_limit` (mis. maks 30% gaji, tidak ada tunggakan); penagihan via potong gaji | P0 | F2 |
| POS-12 | Perangkat keras | Scanner HID, printer thermal, cash drawer (via printer), customer display (P2), timbangan (P2) | P0 | F2 |
| POS-13 | Penjualan jasa & PPOB | Item non-stok (fotokopi, jasa); PPOB pulsa/listrik via agregator | P2 | L |
| POS-14 | Laporan kasir | Penjualan per shift/kasir/metode bayar, anggota vs non-anggota, produk terlaris | P0 | F2 |

## 9. INV — Inventori & Pembelian

| ID | Fitur | Deskripsi & kriteria utama | Prio | Fase |
|---|---|---|---|---|
| INV-01 | Master produk | SKU, banyak barcode, kategori, merek, satuan & konversi (pcs/pak/dus), kode pajak, flag konsinyasi & non-stok, custom field | P0 | F2 |
| INV-02 | Gudang & lokasi | Multi gudang/gerai; stok per lokasi | P0 | F2 |
| INV-03 | Valuasi persediaan | Rata-rata tertimbang bergerak (default); FIFO opsional (P2) | P0 | F2 |
| INV-04 | Pembelian | PO, penerimaan barang (GRN), faktur supplier, retur pembelian, utang dagang & pembayaran | P0 | F2 |
| INV-05 | Supplier | Data supplier termasuk **anggota sebagai pemasok** (hasil tani/produksi anggota → dasar jasa usaha SHU bila diatur) | P0 | F2 |
| INV-06 | Stok opname | Hitung fisik per lokasi (bisa via HP), selisih → penyesuaian dengan persetujuan & jurnal | P0 | F2 |
| INV-07 | Transfer stok | Antar gudang/gerai dengan status kirim/terima | P1 | F2 |
| INV-08 | Stok minimum & saran pemesanan | Peringatan stok minimum; saran qty berdasar rata-rata penjualan | P1 | F2 |
| INV-09 | Batch & kedaluwarsa | Untuk apotek/sembako; FEFO saran pengambilan; peringatan mendekati kedaluwarsa | P1 | F2 |
| INV-10 | Kartu stok | Mutasi per produk per lokasi dengan saldo qty & nilai | P0 | F2 |

## 10. ACC — Akuntansi

| ID | Fitur | Deskripsi & kriteria utama | Prio | Fase |
|---|---|---|---|---|
| ACC-01 | Bagan Akun Standar | Template COA per jenis koperasi; hierarki Kelompok → Golongan → Jenis → Akun (standar, terkunci) → Sub-akun (tenant); pemetaan ke pos laporan & klasifikasi arus kas | P0 | F1 |
| ACC-02 | Saldo awal & migrasi | Wizard saldo awal per akun + rincian buku pembantu (simpanan per anggota, pinjaman berjalan, persediaan, piutang/utang); validasi seimbang & cocok | P0 | F1 |
| ACC-03 | Jurnal umum | Multi baris, dimensi (cabang/unit/proyek), lampiran, status Draft → Diajukan → Posted; maker-checker sesuai ambang; jurnal berulang | P0 | F1 |
| ACC-04 | Posting Engine | Jurnal otomatis dari semua transaksi operasional berdasarkan **template posting** (rule) & **account role**; idempoten; antrean posting gagal | P0 | F1 |
| ACC-05 | Buku besar | Per akun & periode, filter dimensi, saldo awal–mutasi–saldo akhir, *drill-down* ke jurnal & dokumen sumber | P0 | F1 |
| ACC-06 | Buku pembantu | Simpanan per anggota, piutang pinjaman, piutang usaha/kredit anggota, utang supplier, utang konsinyasi, persediaan; rekonsiliasi otomatis vs akun kontrol | P0 | F1 |
| ACC-07 | Kas & bank | Kas masuk/keluar, kas kecil (imprest), transfer antar kas/bank, buku kas harian, cash opname | P0 | F1 |
| ACC-08 | Jurnal penyesuaian & pembalik | Akrual, penyesuaian akhir periode, jurnal pembalik otomatis di awal periode berikut (opsional) | P0 | F1 |
| ACC-09 | Neraca saldo & neraca lajur | Neraca saldo per tanggal; neraca lajur (saldo awal, penyesuaian, disesuaikan, LHU, neraca) | P0 | F1 |
| ACC-10 | Laporan keuangan | **Laporan Posisi Keuangan (Neraca)**, **Laporan Hasil Usaha (Laba Rugi)** dengan partisipasi anggota vs non-anggota, **Perubahan Ekuitas**, **Arus Kas** (langsung & tidak langsung), **CaLK** (template); komparatif; per unit/cabang/konsolidasi | P0 | F1 (Neraca, LHU), F4 (lainnya) |
| ACC-11 | Tutup buku | Bulanan (checklist, kunci periode) dan tahunan (jurnal penutup ke Ikhtisar Hasil Usaha → SHU tahun berjalan, buka tahun baru) | P0 | F1 (bulanan), F4 (tahunan) |
| ACC-12 | Aset tetap | Registrasi, kategori, metode (garis lurus/saldo menurun), penyusutan bulanan otomatis, mutasi, pelepasan (laba/rugi) | P1 | F4 |
| ACC-13 | Dimensi akuntansi | Cabang, unit usaha, proyek/program (mis. dana hibah), status partisipan (anggota/non-anggota) | P0 | F1 |
| ACC-14 | Konsolidasi | Laporan gabungan multi unit/cabang dengan eliminasi transaksi antar unit | P1 | F4 |
| ACC-15 | Anggaran (RAPB) | Rencana Anggaran Pendapatan & Belanja per akun/bulan/unit; realisasi vs anggaran | P1 | F4 |
| ACC-16 | Pajak | Kode & tarif pajak berversi (PPN, PPh final UMKM, PPh final jasa simpanan, PPh 21/23); rekap & ekspor; format Coretax (P2) | P1 | F4 |
| ACC-17 | Analisis rasio | Likuiditas, solvabilitas, rentabilitas, NPL, BOPO, perputaran persediaan; tren | P1 | F4 |

## 11. SHU — Sisa Hasil Usaha & RAT

| ID | Fitur | Deskripsi & kriteria utama | Prio | Fase |
|---|---|---|---|---|
| SHU-01 | Konfigurasi alokasi SHU | Rule `shu.allocation`: persentase cadangan, jasa modal, jasa usaha, pengurus & pengawas, karyawan, pendidikan, sosial, pembangunan daerah kerja; validasi total 100% | P0 | F4 |
| SHU-02 | Perhitungan per anggota | Jasa modal ∝ simpanan (pokok+wajib, opsional sukarela; metode saldo akhir/rata-rata bulanan); jasa usaha ∝ transaksi (belanja POS anggota, jasa pinjaman dibayar, penjualan ke koperasi/konsinyasi) — sumber & bobot diatur rule | P0 | F4 |
| SHU-03 | Simulasi & estimasi | Simulasi beberapa skenario alokasi sebelum RAT; estimasi SHU berjalan di aplikasi anggota | P1 | F4 |
| SHU-04 | Penetapan & distribusi | Setelah RAT (unggah notulen) → jurnal pembagian; pembayaran tunai/transfer/ke simpanan sukarela; potongan pajak sesuai rule; slip SHU per anggota | P0 | F4 |
| SHU-05 | Paket laporan RAT | Laporan pertanggungjawaban: keuangan, keanggotaan, usaha, SHU, rencana kerja & RAPB; ekspor PDF/Word | P1 | F4 |
| SHU-06 | Dana-dana | Saldo & pemakaian dana pendidikan, sosial, pengurus, karyawan, PDK; jurnal pemakaian | P1 | F4 |
| SHU-07 | e-RAT | Undangan, daftar hadir, kuorum, voting digital | P2 | L |

## 12. RUL — Rule Engine (Custom Rule Koperasi)

Spesifikasi lengkap di [03 — Rule Engine](./03-rule-engine.md).

| ID | Fitur | Deskripsi & kriteria utama | Prio | Fase |
|---|---|---|---|---|
| RUL-01 | Katalog hook per domain | Titik evaluasi rule terdokumentasi (keanggotaan, simpanan, pinjaman, POS, loyalty, SHU, akuntansi, notifikasi) beserta variabel & aksi yang diizinkan | P0 | F1 |
| RUL-02 | Rule parameter | Nilai bertipe (angka, persen, rupiah, boolean, pilihan, tanggal) dengan tanggal berlaku | P0 | F1 |
| RUL-03 | Rule formula | Ekspresi aman, mis. `min(3 * anggota.total_simpanan, 0.4 * anggota.penghasilan * pinjaman.tenor)`; editor dengan autocomplete & validasi tipe | P0 | F1 |
| RUL-04 | Tabel keputusan | Baris kondisi → keluaran, kebijakan hit (FIRST/UNIQUE/COLLECT_SUM), deteksi celah & tumpang tindih | P0 | F1 |
| RUL-05 | Rule ECA (Event–Condition–Action) | Builder visual WHEN–IF–THEN; mode sinkron (validasi/ubah nilai, dapat memblokir) & asinkron (poin, notifikasi, tag) | P0 | F2 |
| RUL-06 | Workflow persetujuan | Langkah, penyetuju per peran/jabatan, kondisi nominal, paralel/serial, SLA & eskalasi, delegasi | P0 | F3 |
| RUL-07 | Template posting jurnal | Pemetaan event → baris debit/kredit menggunakan account role & ekspresi nominal; validasi wajib seimbang | P0 | F1 |
| RUL-08 | Versi & tanggal berlaku | Setiap perubahan = versi baru; `effective_from/to`; tidak retroaktif kecuali ditandai dan disetujui | P0 | F1 |
| RUL-09 | Uji & simulasi | Test case (input → harapan), *replay* terhadap data historis (read-only), laporan dampak & perbandingan versi | P0 | F1 (test case), F2 (replay) |
| RUL-10 | Persetujuan publikasi | Maker-checker; rule kategori AD/ART butuh persetujuan Ketua + 1 pengurus; rule akuntansi butuh akuntan + bendahara | P0 | F1 |
| RUL-11 | Log eksekusi & penjelasan | Setiap evaluasi tercatat (versi, input ringkas, hasil, durasi); UI "Jejak Keputusan" untuk menjelaskan hasil ke anggota/auditor | P0 | F1 |
| RUL-12 | Rule pack per jenis koperasi | Paket default dari platform; tenant dapat menyalin & memodifikasi; notifikasi bila pack diperbarui | P0 | F1 |
| RUL-13 | Guardrail | Batas kompleksitas & waktu, aksi yang diizinkan per hook, larangan rantai rekursif, prioritas & strategi konflik | P0 | F1 |
| RUL-14 | Custom field | Atribut tambahan anggota/produk/pinjaman yang dapat dipakai di rule (mis. `anggota.custom.golongan_pegawai`) | P1 | F2 |
| RUL-15 | Rule terjadwal | Pemicu waktu (harian, akhir bulan, tanggal tertentu), mis. denda simpanan wajib tiap tanggal 11 | P1 | F2 |

## 13. RPT — Laporan & Dashboard

| ID | Fitur | Deskripsi & kriteria utama | Prio | Fase |
|---|---|---|---|---|
| RPT-01 | Dashboard eksekutif | Anggota aktif, total simpanan, outstanding pinjaman, NPL, penjualan hari ini, posisi kas & bank, SHU berjalan, antrean posting gagal | P0 | F1 |
| RPT-02 | Laporan operasional | Penjualan (produk, kategori, kasir, anggota vs non-anggota), stok, pembelian, simpanan, tunggakan simpanan wajib, pinjaman | P0 | F1–F3 |
| RPT-03 | Laporan keuangan | Lihat ACC-09/ACC-10 | P0 | F1/F4 |
| RPT-04 | Ekspor & jadwal | PDF, Excel, CSV; kirim terjadwal via email ke pengurus | P0 | F1 (ekspor), F3 (jadwal) |
| RPT-05 | Report builder | Pilih sumber, kolom, filter, grup; simpan sebagai laporan kustom | P2 | L |
| RPT-06 | Laporan untuk pembina | Ekspor data koperasi terstruktur (keanggotaan, usaha, keuangan) untuk Dinas/Kementerian; integrasi API bila tersedia | P2 | L |
| RPT-07 | Dashboard platform | Metrik agregat lintas tenant (anonim) untuk super admin | P1 | F4 |

## 14. NTF — Notifikasi

| ID | Fitur | Deskripsi & kriteria utama | Prio | Fase |
|---|---|---|---|---|
| NTF-01 | In-app & push | Pusat notifikasi back-office & aplikasi anggota | P1 | F3 |
| NTF-02 | WhatsApp | Template pesan: bukti setor, struk digital, pengingat angsuran H-3, tagihan simpanan wajib, OTP; opt-in tercatat | P1 | F3 |
| NTF-03 | Email & SMS | Fallback dan laporan terjadwal | P1 | F3 |
| NTF-04 | Template & pemicu | Template dapat diedit tenant; pemicu via rule ECA atau jadwal | P1 | F3 |

## 15. INT — Integrasi

| ID | Fitur | Deskripsi & kriteria utama | Prio | Fase |
|---|---|---|---|---|
| INT-01 | Payment gateway | QRIS dinamis, VA per anggota; verifikasi signature webhook; rekonsiliasi settlement & MDR | P1 | F3 |
| INT-02 | Potong gaji (payroll) | Ekspor tagihan per instansi (simpanan wajib, angsuran, kredit belanja) → impor realisasi potongan → alokasi otomatis | P0 | F1 |
| INT-03 | Mutasi bank | Impor CSV/MT940 untuk rekonsiliasi bank dengan aturan pencocokan | P1 | F4 |
| INT-04 | API publik & webhook | REST API berkunci per tenant, webhook event (mis. `sale.completed`) | P2 | L |
| INT-05 | Migrasi data | Template impor anggota, saldo simpanan, pinjaman berjalan + jadwal sisa, saldo awal akun, produk & stok; *dry run* | P0 | F1–F2 |
| INT-06 | PPOB | Pulsa, token listrik, tagihan via agregator | P2 | L |
| INT-07 | e-Sign & e-Meterai | Tanda tangan elektronik akad pinjaman | P2 | L |

## 16. APP — Aplikasi Anggota

| ID | Fitur | Deskripsi & kriteria utama | Prio | Fase |
|---|---|---|---|---|
| APP-01 | Login aman | OTP WA/SMS, PIN, biometrik; pilih koperasi bila anggota di >1 koperasi | P1 | F3 |
| APP-02 | Beranda | Saldo per jenis simpanan, pinjaman aktif & angsuran berikutnya, poin, tier, pengumuman | P1 | F3 |
| APP-03 | Kartu anggota QR | QR dinamis (token berganti berkala) untuk dipindai kasir | P1 | F3 |
| APP-04 | Mutasi & riwayat | Mutasi simpanan, riwayat belanja (struk digital), riwayat poin | P1 | F3 |
| APP-05 | Pinjaman | Simulasi, pengajuan, status persetujuan, jadwal angsuran | P1 | F3 |
| APP-06 | Pembayaran | Bayar angsuran/simpanan via VA/QRIS | P1 | F3 |
| APP-07 | SHU | Estimasi SHU berjalan & riwayat SHU per tahun + rincian jasa modal/usaha | P1 | F4 |
| APP-08 | Notifikasi | Push untuk transaksi, pengingat, pengumuman | P1 | F3 |
| APP-09 | Profil & permintaan | Ubah data (dengan persetujuan), unduh kartu, permintaan data pribadi (UU PDP) | P1 | F3 |

---

## 17. User Story Kunci & Acceptance Criteria

> Semua fitur juga wajib memenuhi kontrak lintas modul: error & edge case ([09](./09-error-handling-dan-edge-case.md)),
> API/event/rumus/validasi/izin ([10](./10-spesifikasi-teknis-detail.md)), serta status UI & animasi ([11](./11-design-system-dan-animasi.md)).

### US-01 — Kasir melayani anggota dengan harga anggota dan poin (POS-02, POS-03, LOY-01)

> Sebagai **kasir**, saya ingin memindai kartu anggota agar harga anggota dan poin otomatis diterapkan.

- **Diberikan** anggota berstatus Aktif dengan tier Gold, **ketika** kasir memindai QR kartunya, **maka** layar menampilkan nama,
  nomor anggota, tier, saldo poin, saldo sukarela, dan sisa limit kredit dalam ≤ 1 detik (online) atau dari cache (offline).
- **Dan** setiap item yang memiliki harga anggota langsung berubah ke harga anggota dengan label "harga anggota".
- **Dan** estimasi poin yang akan diperoleh tampil sebelum pembayaran, dihitung oleh versi rule yang sama dengan server.
- **Diberikan** anggota berstatus Pasif/Keluar, **maka** harga umum berlaku dan kasir melihat peringatan status.

### US-02 — Kasir tetap berjualan saat internet putus (POS-08)

- **Diberikan** perangkat POS terdaftar dan terakhir sinkron < 72 jam, **ketika** koneksi terputus, **maka** transaksi tunai tetap dapat diselesaikan dan struk tercetak dengan nomor struk unik per perangkat.
- **Dan** pembayaran dengan saldo sukarela/kredit anggota/poin hanya diizinkan sampai **batas offline** per anggota (rule `pos.offline_limit`).
- **Ketika** koneksi kembali, **maka** transaksi terunggah otomatis; pengunggahan ulang transaksi yang sama tidak membuat duplikat.
- **Diberikan** konflik (mis. saldo anggota tidak cukup di server), **maka** transaksi tetap diterima, selisih dicatat sebagai piutang anggota, dan muncul di **antrean pengecualian** untuk ditindaklanjuti.
- **Diberikan** perangkat belum sinkron > 72 jam (dapat diatur), **maka** POS terkunci untuk transaksi baru sampai sinkron.

### US-03 — Bendahara memproses potong gaji bulanan (SAV-05, INT-02)

- **Ketika** bendahara memilih instansi & periode lalu menekan "Buat Tagihan", **maka** sistem membuat file tagihan per anggota (simpanan wajib, angsuran, kredit belanja) sesuai rule.
- **Ketika** file realisasi potongan diunggah, **maka** sistem mencocokkan per anggota, mengalokasikan ke tagihan sesuai prioritas, membentuk jurnal, dan menampilkan daftar selisih (potongan kurang/lebih).
- **Dan** kelebihan potongan masuk ke titipan anggota (bukan hilang), kekurangan menjadi tunggakan.

### US-04 — Akuntan menutup buku bulanan (ACC-11)

- **Ketika** akuntan membuka "Tutup Buku Oktober", **maka** checklist otomatis menampilkan: antrean posting gagal = 0, jurnal draft = 0, rekonsiliasi buku pembantu = cocok, rekonsiliasi bank = selesai, akrual jasa & penyusutan sudah dijalankan, kas kasir semua shift ditutup.
- **Diberikan** salah satu item gagal, **maka** tombol tutup buku nonaktif dan item menautkan ke layar perbaikan.
- **Ketika** semua lulus dan bendahara menyetujui, **maka** periode berstatus `CLOSED`, laporan bulan itu dibekukan (snapshot), dan posting ke periode tersebut ditolak.

### US-05 — Pengurus mengubah aturan plafon pinjaman (RUL-03, RUL-08, RUL-09, RUL-10)

- **Ketika** pengurus mengubah formula plafon dari `3 × simpanan` menjadi `4 × simpanan` dan menyimpan, **maka** tersimpan sebagai versi Draft baru tanpa memengaruhi versi aktif.
- **Ketika** pengurus menjalankan simulasi terhadap 100 pengajuan 6 bulan terakhir, **maka** sistem menampilkan jumlah pengajuan yang hasilnya berubah dan total kenaikan plafon.
- **Ketika** diajukan, **maka** rule kategori AD/ART meminta persetujuan Ketua dan satu pengurus lain; setelah disetujui, berlaku mulai `effective_from` yang ditentukan.
- **Dan** pengajuan pinjaman yang dibuat sebelum tanggal berlaku tetap dinilai dengan versi lama (snapshot versi tersimpan di pengajuan).

### US-06 — Akuntan menyesuaikan jurnal otomatis (RUL-07, ACC-04)

- **Ketika** akuntan menambahkan sub-akun `110201.002 Bank BSI` dan memetakan account role `BANK_SETORAN_QRIS` ke akun tersebut, **maka** settlement QRIS berikutnya terjurnal ke akun baru.
- **Ketika** akuntan mengubah template posting sehingga total debit ≠ total kredit untuk salah satu test case, **maka** sistem menolak publikasi dengan pesan yang menunjukkan baris penyebabnya.

### US-07 — Pengurus menghitung dan membagikan SHU (SHU-01 s.d. SHU-04)

- **Diberikan** tahun buku 2027 sudah ditutup, **ketika** pengurus membuka kalkulator SHU, **maka** SHU bersih diambil dari saldo akun SHU tahun berjalan dan alokasi persentase dari rule `shu.allocation` versi aktif.
- **Dan** sistem menghitung jasa modal & jasa usaha per anggota dari data simpanan, belanja POS anggota, dan jasa pinjaman yang dibayar, dengan total pembagian = alokasi (selisih pembulatan ke akun pembulatan).
- **Ketika** notulen RAT diunggah dan penetapan disetujui, **maka** jurnal pembagian SHU terbentuk dan setiap anggota menerima slip SHU (PDF/WA/aplikasi).

### US-08 — Anggota memeriksa saldo dan mengajukan pinjaman dari aplikasi (APP-02, APP-05)

- **Ketika** anggota login dengan OTP + PIN, **maka** beranda menampilkan saldo per jenis simpanan, pinjaman aktif, angsuran berikutnya, dan poin.
- **Ketika** anggota mengajukan pinjaman, **maka** plafon maksimum & simulasi angsuran tampil sebelum kirim, dan bila tidak layak, alasan ditampilkan dalam bahasa yang mudah dipahami (dari log penjelasan rule).

### US-09 — Super admin mengaktifkan tenant baru (PLT-01, PLT-09)

- **Ketika** super admin/calon tenant menyelesaikan wizard dengan jenis "Koperasi Konsumen", **maka** tenant dibuat dengan COA standar konsumen, rule pack konsumen, peran bawaan, periode tahun berjalan, dan admin pertama menerima undangan.
- **Dan** seluruh proses ≤ 10 menit tanpa intervensi engineer.

### US-10 — Pengawas menelusuri angka laporan ke bukti (ACC-05, IAM-06)

- **Ketika** pengawas mengklik angka "Kas" di neraca, **maka** muncul buku besar akun kas; klik baris jurnal membuka dokumen sumber (struk POS/slip setoran/bukti kas) beserta lampiran dan riwayat audit (siapa membuat & menyetujui).

[← 01 Ringkasan](./01-ringkasan-analisa-dan-tech-stack.md) · [Lanjut ke 03 — Rule Engine →](./03-rule-engine.md)
