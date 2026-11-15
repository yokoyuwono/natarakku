# PRD — Platform Koperasi Multi-Tenant Terintegrasi

> **Keanggotaan · Simpan Pinjam · Point of Sale · Loyalty · Akuntansi Formal (SAK EP) · Rule Engine Kustom**

| Atribut | Keterangan |
|---|---|
| Nama kerja produk | **KoperasiOS** (nama kerja, dapat diganti) |
| Jenis dokumen | Product Requirements Document (PRD) |
| Versi | 1.1 — Draft untuk review |
| Tanggal | 15 November 2026 |
| Status | Draft, menunggu review stakeholder (pengurus koperasi, akuntan, tech lead) |
| Pemilik dokumen | Product Owner (TBD) |
| Reviewer wajib | Tech Lead, Akuntan/KAP mitra, perwakilan 3–5 koperasi pilot |

---

## Daftar Isi

| No | Dokumen | Isi |
|---|---|---|
| 01 | [Ringkasan Analisa Kebutuhan & Tech Stack](./01-ringkasan-analisa-dan-tech-stack.md) | Latar belakang, masalah, tujuan & KPI, persona, regulasi, analisa multi-tenant, NFR, risiko, arsitektur, tech stack, ADR |
| 02 | [Daftar Fitur](./02-daftar-fitur.md) | Fitur per modul (prioritas & fase), user story kunci & acceptance criteria |
| 03 | [Spesifikasi Rule Engine (Custom Rule Koperasi)](./03-rule-engine.md) | Tipe rule, hook, variabel, aksi, format JSON, siklus hidup, simulasi, guardrail, rule pack default |
| 04 | [Bagan Akun Standar (COA) & Matriks Posting](./04-bagan-akun-dan-posting.md) | Struktur kode akun, daftar akun standar koperasi, pemetaan pos laporan, jurnal otomatis per transaksi, struktur laporan keuangan |
| 05 | [Flowchart](./05-flowchart.md) | Alur onboarding, anggota, simpanan, pinjaman, POS, sinkron offline, posting jurnal, tutup buku, SHU, rule |
| 06 | [ERD](./06-erd.md) | Entity Relationship Diagram per domain + kamus data tabel inti + konvensi database |
| 07 | [Konsep Wireframe](./07-wireframe.md) | Sitemap, prinsip UI, wireframe layar utama (back-office, POS, rule builder, aplikasi anggota) |
| 08 | [Task Todo & Roadmap](./08-task-todo.md) | Fase, milestone, Gantt, daftar tugas per epic, DoD, checklist go-live, pertanyaan terbuka |
| 09 | [Error Handling & Edge Case](./09-error-handling-dan-edge-case.md) | Format error API, katalog kode error, idempotensi, konkurensi, retry, penanganan offline/webhook, pola error UI, logging, 42 edge case |
| 10 | [Spesifikasi Teknis Detail](./10-spesifikasi-teknis-detail.md) | Konvensi & katalog API, katalog event, state machine, rumus kalkulasi, validasi field, penomoran, matriks izin, jadwal job, keamanan, notifikasi, format impor |
| 11 | [Design System & Animasi](./11-design-system-dan-animasi.md) | Token desain & gerak, katalog animasi ringan per layar (back-office, rule builder, POS, aplikasi anggota), aksesibilitas, anggaran kinerja |
| 12 | [Panduan Delegasi ke Claude Code](./12-panduan-delegasi-claude-code.md) | Status kesiapan, pembagian kerja manusia vs AI, urutan sesi, template prompt, checklist review + [`CLAUDE.template.md`](./CLAUDE.template.md) |

**Cara membaca:** mulai dari dokumen 01 untuk konteks, lalu 02 untuk cakupan. Dokumen 03 dan 04 adalah
spesifikasi inti yang membedakan produk ini (rule engine kustom dan akuntansi standar). Dokumen 05–07
menerjemahkan kebutuhan ke alur, data, dan tampilan. Dokumen 08 adalah rencana eksekusi.

Dokumen 09–11 adalah **kontrak teknis lintas modul** (error, API/event/rumus, UI & animasi) yang wajib diikuti
setiap fitur. Dokumen 12 menjelaskan cara mendelegasikan implementasi ke Claude Code.

Semua diagram ditulis dalam **Mermaid** sehingga tampil langsung di GitHub/GitLab dan dapat diedit sebagai teks.

---

## Ringkasan Eksekutif (1 halaman)

**Masalah.** Banyak koperasi di Indonesia masih mengelola anggota, simpanan, pinjaman, toko, dan pembukuan
di aplikasi terpisah atau di Excel. Akibatnya terjadi input ganda, saldo simpanan tidak cocok dengan buku besar,
laporan untuk RAT disusun manual berminggu-minggu, dan perhitungan SHU sulit ditelusuri. Sejak Permenkop UKM
No. 2 Tahun 2024, koperasi simpan pinjam wajib menyusun laporan keuangan berbasis SAK EP paling lambat tahun buku 2025,
sementara aturan internal tiap koperasi (AD/ART, besaran simpanan, plafon pinjaman, pembagian SHU) berbeda-beda
dan sering berubah. Undang-undang perkoperasian pun sedang direvisi.

**Solusi.** Platform SaaS multi-tenant tempat setiap koperasi (tenant) memiliki:

1. **Keanggotaan & loyalty** — data anggota lengkap, kartu anggota QR, tier, poin.
2. **Simpanan & pinjaman** — produk dapat dikonfigurasi, teller, potong gaji, kolektibilitas, CKPN.
3. **Point of Sale & inventori** — kasir offline-first, harga anggota, bayar pakai saldo simpanan/kredit anggota/poin.
4. **Akuntansi formal** — Bagan Akun Standar, jurnal otomatis dari semua transaksi, buku besar, buku pembantu,
   kas & bank, neraca, laporan hasil usaha (laba rugi), arus kas, perubahan ekuitas, tutup buku, dan SHU.
5. **Rule Engine kustom** — pengurus/akuntan dapat membuat aturan sendiri (parameter, formula, tabel keputusan,
   *event–condition–action*, alur persetujuan, template jurnal) tanpa koding, terversi, dapat disimulasikan,
   dan dapat diaudit.

**Prinsip pembeda.** Tidak ada transaksi operasional tanpa jurnal. Setiap transaksi kasir, teller, dan pinjaman
otomatis membentuk jurnal berpasangan (double-entry) melalui *Posting Engine* yang aturannya juga bisa dikustom.
Karena data anggota, transaksi, dan akuntansi ada di satu tempat, SHU dapat dihitung otomatis dari
jasa modal (simpanan) dan jasa usaha (belanja di toko, jasa pinjaman).

**Tech stack ringkas.** TypeScript end-to-end: NestJS (modular monolith) + PostgreSQL 16 (Row Level Security
untuk isolasi tenant) + Redis/BullMQ, Next.js untuk back-office, PWA offline-first (+ Capacitor) untuk POS,
React Native (Expo) untuk aplikasi anggota. Infrastruktur di region Jakarta.

**Rencana.** ±52 minggu dalam 6 fase. MVP akuntansi + keanggotaan + simpanan siap pilot internal di minggu ke-16,
POS di minggu ke-26, pinjaman & aplikasi anggota di minggu ke-36, SHU & tutup buku tahunan di minggu ke-44,
pilot dan go-live di minggu ke-52.

---

## Glosarium

| Istilah | Arti |
|---|---|
| **Tenant** | Satu koperasi pelanggan platform; datanya terisolasi dari koperasi lain |
| **Anggota** | Pemilik sekaligus pengguna jasa koperasi; memiliki simpanan pokok & wajib |
| **Calon anggota** | Pendaftar yang belum memenuhi syarat/persetujuan menjadi anggota |
| **Non-anggota / pelanggan** | Pembeli di unit usaha koperasi yang bukan anggota |
| **Simpanan Pokok** | Setoran sekali saat masuk; bagian modal sendiri (ekuitas); tidak dapat ditarik selama menjadi anggota |
| **Simpanan Wajib** | Setoran berkala (umumnya bulanan); ekuitas; tidak dapat ditarik selama menjadi anggota |
| **Simpanan Sukarela** | Simpanan yang dapat disetor/ditarik sewaktu-waktu; liabilitas koperasi |
| **Simpanan Berjangka** | Simpanan dengan jangka waktu tertentu (mirip deposito); liabilitas |
| **SHU** | Sisa Hasil Usaha — selisih pendapatan dan beban dalam satu tahun buku |
| **Jasa Modal / Jasa Usaha** | Bagian SHU anggota berdasarkan simpanan (modal) dan berdasarkan transaksi (usaha) |
| **RAT** | Rapat Anggota Tahunan — forum tertinggi; mengesahkan laporan & pembagian SHU |
| **AD/ART** | Anggaran Dasar / Anggaran Rumah Tangga koperasi |
| **KSP / USP** | Koperasi Simpan Pinjam / Unit Simpan Pinjam |
| **KSPPS** | Koperasi Simpan Pinjam dan Pembiayaan Syariah |
| **KSU** | Koperasi Serba Usaha |
| **KDKMP / Kopdes Merah Putih** | Koperasi Desa/Kelurahan Merah Putih (program nasional sejak 2025) |
| **Pengurus / Pengawas** | Pengelola koperasi (Ketua, Sekretaris, Bendahara) / pengawas kinerja pengurus |
| **SAK EP** | Standar Akuntansi Keuangan Entitas Privat (berlaku mulai 1 Januari 2025) |
| **COA** | *Chart of Accounts* — Bagan Akun |
| **Buku Besar (GL)** | Kumpulan mutasi dan saldo per akun |
| **Buku Pembantu** | Rincian per pihak (anggota, supplier, barang) yang total saldonya = akun kontrol di buku besar |
| **Neraca Saldo** | Daftar saldo semua akun pada tanggal tertentu; total debit = total kredit |
| **Laporan Hasil Usaha (LHU)** | Laporan laba rugi versi koperasi; menyajikan partisipasi anggota & pendapatan non-anggota |
| **Tutup Buku** | Mengunci periode akuntansi; tutup tahunan memindahkan saldo pendapatan/beban ke SHU |
| **Kolektibilitas** | Klasifikasi kualitas pinjaman berdasarkan lama tunggakan |
| **CKPN** | Cadangan Kerugian Penurunan Nilai piutang pinjaman |
| **DPD** | *Days Past Due* — jumlah hari keterlambatan |
| **NPL** | *Non Performing Loan* — rasio pinjaman bermasalah |
| **HPP** | Harga Pokok Penjualan |
| **Konsinyasi** | Titip jual — barang milik penitip (sering anggota produsen) dijual koperasi |
| **Rule** | Aturan bisnis yang dapat dikonfigurasi tenant di Rule Engine |
| **Hook** | Titik di proses bisnis tempat rule dievaluasi (mis. `loan.plafond`) |
| **Rule Pack** | Paket rule default per jenis koperasi yang disediakan platform |
| **Posting Engine** | Komponen yang mengubah transaksi operasional menjadi jurnal akuntansi |
| **Account Role** | Peran akun (mis. `KAS_GERAI`) yang dipetakan ke akun nyata per tenant/gerai |
| **RLS** | *Row Level Security* PostgreSQL — isolasi baris data per tenant |
| **Outbox** | Pola menyimpan event di tabel DB pada transaksi yang sama agar tidak hilang |

---

## Riwayat Revisi

| Versi | Tanggal | Perubahan | Oleh |
|---|---|---|---|
| 1.0 | 2026-11-14 | Draft awal lengkap (analisa, fitur, rule engine, COA, flowchart, ERD, wireframe, todo) | Tim Produk |
| 1.1 | 2026-11-15 | Tambah dokumen 09 (error & edge case), 10 (spesifikasi teknis detail), 11 (design system & animasi), 12 (panduan delegasi) + template CLAUDE.md; tugas & DoD di 08 diperbarui | Tim Produk |

## Referensi

- [Permenkop UKM No. 2 Tahun 2024 tentang Kebijakan Akuntansi Koperasi — JDIH BPK](https://peraturan.bpk.go.id/Details/308465/permenkop-ukm-no-2-tahun-2024)
- [DDTCNews — Kapan Koperasi Simpan Pinjam Wajib Terapkan SAK EP](https://news.ddtc.co.id/berita/nasional/1801892/kapan-koperasi-simpan-pinjam-wajib-terapkan-sak-ep-simak-di-sini)
- [Bisnis.com — RI Punya 224.256 Koperasi, KopDes Merah Putih Mendominasi (Juli 2026)](https://ekonomi.bisnis.com/read/20260708/12/1986551/ri-punya-224256-koperasi-kopdes-merah-putih-mendominasi-juli-2026)
- [Metro TV — 10.070 Koperasi Merah Putih Sudah Beroperasi (Oktober 2026)](https://www.metrotvnews.com/read/NP6CPnxq-zulkifli-hasan-10-070-koperasi-merah-putih-sudah-beroperasi)
- [CNN Indonesia — Menkop Beberkan 5 Poin Usulan Baru RUU Perkoperasian (Juni 2026)](https://www.cnnindonesia.com/ekonomi/20260617142413-92-1370023/sudah-34-tahun-menkop-beberkan-5-poin-usulan-baru-ruu-perkoperasian)
- UU No. 25 Tahun 1992 tentang Perkoperasian; UU No. 4 Tahun 2023 (P2SK); UU No. 27 Tahun 2022 (Pelindungan Data Pribadi); PP No. 71 Tahun 2019 (PSTE); UU No. 8 Tahun 1997 (Dokumen Perusahaan).

> **Catatan verifikasi.** Rujukan regulasi dan angka pasar di dokumen ini dihimpun per November 2026.
> Sebelum implementasi, pasal, lampiran format laporan, tarif pajak, dan status RUU Perkoperasian wajib
> diverifikasi ulang bersama konsultan hukum dan akuntan/KAP mitra (lihat tugas `F0-DISC-02` di dokumen 08).
