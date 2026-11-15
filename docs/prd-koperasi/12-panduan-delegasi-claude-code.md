# 12 — Panduan Delegasi ke Claude Code

[← Kembali ke daftar isi](./README.md)

---

## 1. Status Kesiapan Dokumen

| Area | Dokumen | Siap didelegasikan? | Catatan |
|---|---|---|---|
| Konteks, tujuan, arsitektur, tech stack | 01 | Ya | |
| Cakupan fitur & acceptance criteria | 02 | Ya | User story rinci tersedia untuk 10 alur kunci; fitur lain mengikuti deskripsi tabel + dok. 09/10 |
| Rule engine | 03 | Ya | Skema JSON, hook, operator, aksi, guardrail lengkap |
| COA & jurnal otomatis | 04 | **Ya, setelah validasi akuntan** | Gate `F0-DISC-02` |
| Alur proses | 05 | Ya | |
| Model data | 06 | Ya | Kolom kunci + kamus data tabel inti; kolom tambahan diturunkan dari 10 §2–§6 |
| Wireframe | 07 | Ya untuk struktur | Desain visual final dari Figma (Fase 0) |
| Rencana & tugas | 08 | Ya | ID tugas dipakai sebagai unit delegasi |
| Error handling & edge case | 09 | Ya | Kontrak wajib semua modul |
| API, event, state, rumus, validasi, izin | 10 | Ya | |
| Design system & animasi | 11 | Ya untuk token & perilaku | Butuh aset Figma untuk detail visual |

**Keputusan yang harus diambil manusia sebelum fase terkait dimulai** (dari dok. 08 §13):

| Pertanyaan | Memblokir |
|---|---|
| Q1–Q5 (format LHU, akrual jasa, CKPN, poin, granularitas posting POS) | Template posting & laporan di F1–F2 — **Claude Code dapat memulai dengan nilai default dokumen ini**, karena semuanya dibuat dapat dikonfigurasi |
| Q6 (model harga SaaS) | Billing F4 |
| Q7 (vendor payment gateway & WA) | Integrasi F3 — buat adapter + mock dulu |
| Q8 (integrasi sistem pemerintah) | Backlog |
| Q10 (batas offline) | Nilai default rule pack F2 |

## 2. Yang Didelegasikan vs Tetap di Tangan Manusia

| Didelegasikan ke Claude Code | Tetap manusia |
|---|---|
| Menulis kode, migrasi, tes, dokumentasi OpenAPI, Storybook | Menjawab pertanyaan terbuka & keputusan produk |
| Paket inti (`money`, `rule-engine`, `pricing`, `contracts`, `motion`) | Validasi akuntansi (skenario jurnal, laporan, SHU) |
| CRUD back-office, laporan, POS PWA, aplikasi Expo | Uji perangkat keras (printer, scanner, tablet) & uji lapangan gerai |
| Terraform & pipeline CI (dengan review) | Kredensial, akun vendor, rilis store, pendaftaran PSE, legal |
| Memperbaiki temuan review & CI | Review PR kode sensitif, pentest, keputusan go-live |

## 3. Persiapan Repositori

1. Buat repositori baru untuk platform (repo ini berisi situs Natarakku Coffee).
2. Salin folder `docs/prd-koperasi/` ke repo baru sebagai `docs/prd/`.
3. Salin [`CLAUDE.template.md`](./CLAUDE.template.md) menjadi `CLAUDE.md` di root repo baru, lalu sesuaikan.
4. Aktifkan proteksi branch `main`: wajib PR, CI hijau, minimal 1 review.
5. Siapkan environment cloud dengan akses ke registry paket; PostgreSQL & Redis dijalankan via Docker/Testcontainers untuk tes.

## 4. Aturan Main Delegasi

1. **Satu sesi = satu epic kecil atau 3–6 ID tugas yang saling terkait.** Jangan "kerjakan Fase 1".
2. **Selalu sebut dokumen & bagian sumber** (mis. "dok. 03 §8, dok. 10 §5.2").
3. **Minta rencana dulu** untuk epic besar; setujui rencana, baru eksekusi.
4. **Tes sebagai kontrak**: untuk kalkulasi & akuntansi, berikan skenario (input → hasil yang diharapkan) dari akuntan.
5. **Setiap sesi berakhir dengan PR** berisi: ringkasan, ID tugas, tes yang ditambahkan, kode error baru, keputusan yang diambil, dan hal yang belum selesai.
6. **Review manusia wajib** untuk modul: auth, RLS/tenant, akuntansi/posting, rule engine, sinkron POS, pembayaran.
7. **Perbarui dokumen** bila implementasi menemukan celah: PR yang sama mengubah `docs/prd/` (mis. menambah kode error ke dok. 09).

## 5. Urutan Sesi yang Disarankan (Fase 0–1)

| Sesi | ID tugas | Fokus | Gate manusia |
|---|---|---|---|
| S01 | F0-ARCH-01, -02, -04, -05 | Monorepo, CI, skeleton NestJS, konvensi DB | Review struktur |
| S02 | F0-ARCH-06 | Tenant context + RLS + tes negatif lintas tenant | **Review keamanan wajib** |
| S03 | F0-ARCH-07, -08, -09, -13 | Outbox, `@kop/money`, `@kop/contracts`, kerangka error RFC 9457 | Review |
| S04 | F0-IAM-01 … -04 | Auth, 2FA, RBAC, audit log | **Review keamanan wajib** |
| S05 | F0-ARCH-11, -12, -14, F0-PLT-04 | Shell Next.js, `@kop/ui`, `@kop/motion`, penomoran dokumen | Review desainer |
| S06 | F0-PLT-01 … -03, -05 | Konsol super admin, onboarding, template, pairing perangkat | Demo |
| S07 | F1-RUL-01, -02 | Paket rule engine (evaluator, operator, guardrail) | Review TL |
| S08 | F1-RUL-03 … -06, -13 | Penyimpanan rule, tipe rule, context builder, tabel referensi | Review TL |
| S09 | F1-ACC-01 … -05 | COA, account role, periode, layanan jurnal | **Review akuntan** |
| S10 | F1-ACC-07, -08, F1-RUL-05 | Posting Engine + saldo akun | **Review akuntan + TL** |
| S11 | F1-MBR-01 … -06, -09, -10 | Keanggotaan inti | Demo |
| S12 | F1-SAV-01 … -05, -10 | Simpanan & teller | **Review akuntan** |
| S13 | F1-SAV-06 … -09, -11, F1-MBR-07 | Tagihan wajib, potong gaji, jasa, keluar anggota | **Review akuntan** |
| S14 | F1-ACC-09 … -13, -15 | Buku besar, buku pembantu, kas & bank, saldo awal, laporan | **Review akuntan** |
| S15 | F1-ACC-14, -16, F1-QA-03 | Tutup buku, tes properti, tes edge case | Review |
| S16 | F1-RUL-07 … -12 | UI rule builder v1, uji, persetujuan, rule pack | Demo ke pengurus pilot |
| S17 | F1-MBR-08, -11, F1-RPT-01, -02, F1-ORG-01 | Profil 360, impor, dashboard, laporan simpanan | Demo |
| S18 | F1-QA-01, -02 | E2E & pilot internal | UAT |

Fase 2–5 mengikuti pola yang sama per epic di dokumen 08.

## 6. Template Prompt per Sesi

```text
Konteks: baca CLAUDE.md dan dokumen PRD yang disebut di bawah sebelum mulai.

Tugas: kerjakan F1-SAV-03 dan F1-SAV-04 (dok. 08 §4).
Sumber spesifikasi:
- Fitur & acceptance criteria: dok. 02 SAV-03
- Alur: dok. 05 §5.3
- Data: dok. 06 §4 + kamus data §11.5
- API: dok. 10 §2.3 (deposits, withdrawals) — wajib Idempotency-Key
- Validasi: dok. 10 §6; error: dok. 09 §4.3 (SAV_*), edge case EC-06..EC-08
- Jurnal: dok. 04 §5.1 S3–S4 (via Posting Engine, sinkron)
- Rule hook: saving.before_deposit / saving.before_withdraw (dok. 03 §4)
- UI: wireframe dok. 07 §3.6; status & animasi dok. 11 §5.1 dan §6

Batasan:
- Ikuti semua aturan di CLAUDE.md (uang desimal, RLS, idempotensi, tidak ada panggilan eksternal dalam transaksi DB).
- Jangan mengubah modul lain di luar kebutuhan; bila perlu, jelaskan di PR.

Selesai bila:
- Tes unit, integrasi (Testcontainers), dan E2E untuk alur setor/tarik lulus
- Tes untuk setiap kode error yang mungkin & edge case EC-06..EC-08
- Tes idempotensi (request ganda → satu transaksi & satu jurnal)
- Lint & typecheck bersih; PR dibuat dengan ringkasan dan daftar keputusan

Mulai dengan rencana singkat (file yang diubah, urutan langkah), lalu kerjakan.
```

## 7. Checklist Review PR (untuk manusia)

- [ ] Sesuai ID tugas & acceptance criteria; tidak ada perubahan di luar lingkup tanpa penjelasan
- [ ] Setiap query tenant melalui konteks RLS; tes negatif lintas tenant ada
- [ ] Uang memakai `@kop/money`/`NUMERIC`; tidak ada `number` untuk nominal
- [ ] Transaksi keuangan: atomik, idempoten, menghasilkan jurnal via Posting Engine
- [ ] Kode error dari katalog dok. 09; pesan Bahasa Indonesia jelas
- [ ] Tidak ada PII di log; data sensitif terenkripsi/termasking
- [ ] Tes mencakup jalur sukses, error, dan edge case relevan
- [ ] UI punya status memuat/kosong/error/berhasil; animasi dari token dan menghormati reduced motion
- [ ] Migrasi DB dapat di-rollback; OpenAPI diperbarui
- [ ] Dokumen PRD diperbarui bila ada keputusan/celah baru

## 8. Risiko Delegasi & Mitigasi

| Risiko | Mitigasi |
|---|---|
| Kode tampak benar tetapi logika akuntansi keliru | Skenario jurnal dari akuntan sebagai tes; review akuntan di sesi bertanda |
| Asumsi diam-diam saat spesifikasi ambigu | Minta Claude Code mencatat setiap asumsi di PR & mengusulkan pembaruan dokumen |
| Penyimpangan arsitektur antar sesi | `CLAUDE.md` + aturan batas modul di CI (dependency-cruiser) |
| Celah keamanan | Review keamanan wajib di S02/S04, pentest di F5 |
| Konteks sesi terlalu besar | Sesi kecil per epic; rujuk bagian dokumen spesifik, bukan seluruh PRD |

[← 11 Design System & Animasi](./11-design-system-dan-animasi.md) · [Kembali ke daftar isi](./README.md)
