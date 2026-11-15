# CLAUDE.md — KoperasiOS

> Template. Salin ke root repositori platform sebagai `CLAUDE.md` dan sesuaikan.

## Proyek

Platform SaaS koperasi multi-tenant: keanggotaan, simpanan, pinjaman, POS offline-first, loyalty,
akuntansi formal (SAK EP), SHU, dan rule engine kustom. Spesifikasi lengkap di `docs/prd/` (mulai dari `README.md`).

## Sumber Kebenaran

| Topik | Dokumen |
|---|---|
| Fitur & acceptance criteria | `docs/prd/02-daftar-fitur.md` |
| Rule engine | `docs/prd/03-rule-engine.md` |
| COA, account role, jurnal otomatis | `docs/prd/04-bagan-akun-dan-posting.md` |
| Alur | `docs/prd/05-flowchart.md` |
| Data | `docs/prd/06-erd.md` |
| UI | `docs/prd/07-wireframe.md`, `docs/prd/11-design-system-dan-animasi.md` |
| Tugas | `docs/prd/08-task-todo.md` (sebut ID tugas di commit & PR) |
| Error & edge case | `docs/prd/09-error-handling-dan-edge-case.md` |
| API, event, state, rumus, validasi, izin | `docs/prd/10-spesifikasi-teknis-detail.md` |

Bila kode dan dokumen bertentangan, **dokumen menang** kecuali PR yang sama memperbarui dokumen beserta alasannya.

## Stack

TypeScript · Turborepo + pnpm · NestJS (`apps/api`, `apps/worker`) · PostgreSQL 16 + Drizzle · Redis + BullMQ ·
Next.js 15 (`apps/web`) · Vite React PWA + Capacitor (`apps/pos`) · Expo (`apps/member-app`) ·
Zod (`packages/contracts`) · Vitest, Testcontainers, Playwright, fast-check.

## Perintah

```bash
pnpm install
pnpm dev                 # semua app
pnpm test                # unit
pnpm test:int            # integrasi (butuh Docker)
pnpm test:e2e            # Playwright (--trace off)
pnpm lint && pnpm typecheck
pnpm db:migrate          # migrasi lokal
```

## Aturan Wajib (jangan dilanggar)

1. **Uang**: selalu `@kop/money` (desimal) di kode dan `NUMERIC(20,2)` di DB. Dilarang `number`/float untuk nominal. Nilai uang di JSON berupa string.
2. **Tenant**: setiap akses data tenant melalui transaksi dengan `SET LOCAL app.tenant_id`. Jangan pernah mengambil `tenant_id` dari body request. Endpoint baru wajib punya tes negatif lintas tenant.
3. **Akuntansi**: transaksi yang berdampak keuangan menghasilkan jurnal **hanya** lewat Posting Engine + template posting. Jurnal `POSTED` imutabel; koreksi = jurnal pembalik. Total debit = kredit.
4. **Atomik & idempoten**: satu perintah = satu transaksi DB (transaksi + jurnal + outbox). Endpoint yang mengubah uang/stok/poin wajib `Idempotency-Key`.
5. **Tidak ada panggilan eksternal (HTTP, WA, payment gateway, storage) di dalam transaksi DB** — gunakan outbox.
6. **Rule**: logika yang dapat berbeda antar koperasi harus lewat hook rule engine, bukan `if` khusus tenant di kode.
7. **Error**: format RFC 9457, kode dari katalog dok. 09. Kode baru → tambahkan ke katalog di PR yang sama.
8. **PII**: NIK, No. KK, rekening bank terenkripsi; jangan pernah di-log.
9. **Batas modul**: modul hanya saling memanggil lewat service publik atau event; dicek dependency-cruiser di CI.
10. **UI**: setiap layar punya status memuat/kosong/error/berhasil; animasi hanya dari token `@kop/motion` dan wajib menghormati reduced motion.

## Konvensi

- Bahasa kode & identifier: Inggris. Teks UI & pesan error untuk pengguna: Bahasa Indonesia.
- Nama event `domain.past_tense`; nama hook sesuai dok. 03 §4.
- Commit: `feat(savings): ... [F1-SAV-03]`.
- Setiap PR: ringkasan, ID tugas, tes yang ditambahkan, kode error baru, asumsi yang diambil, sisa pekerjaan.

## Definisi Selesai

Lihat `docs/prd/08-task-todo.md` §10 dan `docs/prd/09-error-handling-dan-edge-case.md` §12.

## Saat Ragu

Jangan menebak aturan akuntansi atau kebijakan koperasi. Pilih default yang ada di dokumen, buat dapat dikonfigurasi
lewat rule, dan catat asumsinya di PR.
