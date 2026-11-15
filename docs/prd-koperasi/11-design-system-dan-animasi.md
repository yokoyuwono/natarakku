# 11 — Design System & Animasi Interaktif

[← Kembali ke daftar isi](./README.md)

---

Tujuan: antarmuka yang **terasa hidup dan responsif** lewat animasi ringan (*micro-interaction*), tanpa
mengorbankan kecepatan kerja kasir/teller, kinerja di perangkat Android murah, dan aksesibilitas.

## 1. Prinsip Animasi

| # | Prinsip | Artinya |
|---|---|---|
| M1 | **Bermakna** | Animasi menjelaskan sesuatu: asal/tujuan elemen, perubahan status, umpan balik aksi. Tidak ada animasi dekoratif yang berulang terus |
| M2 | **Cepat** | 100–300 ms untuk interaksi; maks 400 ms untuk transisi besar; maks 800 ms untuk animasi sekali-tampil (mis. hitung naik KPI) |
| M3 | **Tidak menghalangi** | Pengguna dapat langsung mengetik/klik saat animasi berjalan; animasi dapat diinterupsi. Fokus input POS tidak pernah tertunda animasi |
| M4 | **Murah** | Hanya menganimasikan `transform` dan `opacity`; hindari `height`/`width`/`top` pada daftar panjang; target ≥ 50 fps di perangkat referensi |
| M5 | **Menghormati preferensi** | `prefers-reduced-motion: reduce` → animasi gerak diganti fade singkat atau dihilangkan; ada sakelar "Kurangi animasi" di pengaturan pengguna & perangkat POS |
| M6 | **Angka keuangan tetap terbaca** | Nominal tidak boleh "berputar" saat pengguna perlu membacanya untuk keputusan; hitung naik hanya di dashboard & sekali saat dimuat |
| M7 | **Konsisten** | Semua durasi & easing dari token (§3); tidak ada nilai ad-hoc di komponen |

## 2. Token Desain Dasar

| Kategori | Token |
|---|---|
| Warna peran | `primary` (aksi utama), `success` (berhasil, saldo bertambah), `warning` (perlu perhatian), `danger` (error, blokir), `info`, `neutral-0…900`; mode gelap opsional (fase lanjut). Kontras teks ≥ 4,5:1 |
| Warna semantik keuangan | Debit/kredit **tidak** diwarnai merah/hijau (netral); hanya status (lunas, menunggak, macet) yang berwarna, selalu disertai ikon/teks |
| Tipografi | Inter (UI), angka memakai `font-variant-numeric: tabular-nums` agar kolom nominal rata; skala 12/14/16/20/24/32 |
| Spasi | Kelipatan 4 px (4, 8, 12, 16, 24, 32, 48) |
| Radius | 6 px (input, tombol), 10 px (kartu), 16 px (modal, kartu anggota) |
| Elevasi | 0 (datar), 1 (kartu), 2 (dropdown/drawer), 3 (modal), 4 (toast) |
| Ikon | Set tunggal (mis. Lucide), 20/24 px |
| Ukuran sentuh | ≥ 44 px (POS & mobile ≥ 48 px) |

## 3. Token Gerak (Motion Tokens)

| Token | Nilai | Pemakaian |
|---|---|---|
| `duration.instant` | 80 ms | Umpan balik tekan tombol, toggle |
| `duration.fast` | 150 ms | Hover, tooltip, fade, highlight baris |
| `duration.base` | 200 ms | Dropdown, accordion, item masuk daftar |
| `duration.slow` | 300 ms | Drawer, modal, transisi halaman/langkah wizard |
| `duration.emphasis` | 600–800 ms | Hitung naik KPI, centang sukses, gambar grafik (sekali) |
| `easing.standard` | `cubic-bezier(0.2, 0, 0, 1)` | Sebagian besar transisi |
| `easing.decelerate` | `cubic-bezier(0, 0, 0, 1)` | Elemen masuk layar |
| `easing.accelerate` | `cubic-bezier(0.3, 0, 1, 1)` | Elemen keluar layar |
| `spring.gentle` | stiffness 300, damping 30 | Mobile: kartu, bottom sheet, tarik-untuk-segarkan |
| `stagger.list` | 30 ms per item, maks 8 item | Kemunculan daftar/kartu (item ke-9 dst. tanpa jeda) |
| `distance.small` | 4–8 px | Geser saat masuk (slide-in kecil) |

Implementasi: paket `@kop/motion` mengekspor token sebagai CSS variables, preset Tailwind, dan preset
Motion/Reanimated, plus hook `useReducedMotion()` yang menggabungkan preferensi OS dan pengaturan pengguna.

```ts
// packages/motion/src/presets.ts (ilustrasi)
export const fadeSlideIn = {
  initial: { opacity: 0, y: 6 },
  animate: { opacity: 1, y: 0, transition: { duration: 0.2, ease: [0, 0, 0, 1] } },
  exit:    { opacity: 0, y: -4, transition: { duration: 0.15, ease: [0.3, 0, 1, 1] } },
};
export const reduced = { initial: { opacity: 0 }, animate: { opacity: 1, transition: { duration: 0.1 } } };
```

## 4. Teknologi Animasi

| Permukaan | Teknologi | Catatan |
|---|---|---|
| Back-office (Next.js) | **Motion** (framer-motion) untuk transisi komponen & layout; CSS transitions/Tailwind untuk hover/fokus | `AnimatePresence` untuk modal/drawer/toast; `layout` animation untuk reorder di rule builder |
| POS (PWA + Capacitor) | **CSS transitions & keyframes saja** (tanpa pustaka animasi JS) | Menekan ukuran bundle & beban CPU di Android murah |
| Aplikasi anggota (Expo) | **React Native Reanimated 3** (+ Moti untuk preset), haptics `expo-haptics` | Animasi berjalan di UI thread |
| Ilustrasi status | **Lottie** hanya untuk ≤ 5 animasi (sukses, kosong, offline, error, selamat datang); masing-masing ≤ 30 KB | Selalu ada fallback gambar statis |
| Grafik | Animasi bawaan pustaka chart (gambar sekali saat muncul, ≤ 600 ms) | Tidak beranimasi ulang saat filter berubah > 1×/detik |

## 5. Katalog Animasi per Permukaan

### 5.1 Back-office (Web)

| ID | Elemen / interaksi | Animasi | Durasi / easing | Mode kurangi animasi |
|---|---|---|---|---|
| BO-01 | Pindah halaman | Konten fade + geser 4 px ke atas; header & sidebar diam | 200 ms decelerate | Fade 100 ms |
| BO-02 | Skeleton memuat | Shimmer lembut kiri→kanan | Loop 1,2 s | Blok statis abu-abu |
| BO-03 | Daftar/tabel pertama kali tampil | Baris muncul berurutan (stagger) | 200 ms + 30 ms/baris, maks 8 | Tanpa stagger |
| BO-04 | Data baru masuk (mis. transaksi realtime) | Baris baru slide-in + latar sorot memudar | 150 ms + sorot 1,5 s | Sorot saja |
| BO-05 | Drawer detail | Geser dari kanan + backdrop fade | 300 ms standard | Fade |
| BO-06 | Modal / dialog konfirmasi | Scale 0,96→1 + fade | 200 ms | Fade |
| BO-07 | Toast notifikasi | Slide dari kanan atas, hilang otomatis 5 s (error tidak hilang otomatis) | 200 ms in / 150 ms out | Fade |
| BO-08 | Tombol tekan | Scale 0,98 | 80 ms | Tanpa scale |
| BO-09 | Tombol memproses | Label → spinner + teks "Memproses…", lebar tombol tetap | 150 ms | Spinner statis |
| BO-10 | Sukses posting/simpan | Ikon centang tergambar (stroke) di tombol/toast | 400 ms | Ikon langsung tampil |
| BO-11 | Error validasi form | Field bergetar horizontal kecil (±4 px, 2×) + pesan fade-in | 250 ms | Pesan saja |
| BO-12 | Accordion / grup filter | Chevron berputar + konten fade (tinggi dengan `grid-template-rows`) | 200 ms | Instan |
| BO-13 | KPI dashboard | Hitung naik dari 0 ke nilai, **sekali** saat halaman dimuat | 700 ms decelerate | Nilai langsung |
| BO-14 | Grafik | Garis/bar tergambar sekali | 600 ms | Langsung |
| BO-15 | Wizard (onboarding, SHU) | Indikator langkah mengisi progres; konten geser horizontal sesuai arah | 300 ms | Fade |
| BO-16 | Checklist tutup buku | Item lulus berubah menjadi centang berurutan saat pemeriksaan berjalan; item gagal berdenyut lembut sekali | 150 ms per item | Instan |
| BO-17 | Drill-down laporan → buku besar | Baris yang diklik "terangkat" lalu panel buku besar meluncur | 300 ms | Fade |
| BO-18 | Inbox persetujuan | Item yang diputuskan geser keluar, daftar merapat | 200 ms | Hilang instan |
| BO-19 | Hover kartu/baris | Elevasi naik 1 tingkat | 150 ms | Warna latar saja |

### 5.2 Rule Builder

| ID | Interaksi | Animasi |
|---|---|---|
| RB-01 | Tambah/hapus kondisi atau aksi | Baris baru expand + fade; baris dihapus collapse (200 ms) |
| RB-02 | Seret untuk mengurutkan aksi/baris tabel keputusan | Layout animation; elemen lain bergeser halus (200 ms) |
| RB-03 | Kalimat bahasa Indonesia | Bagian kalimat yang berubah disorot sebentar saat rule diedit (1 s memudar) |
| RB-04 | Validasi | Lencana status (skema, tipe, aksi, test case) berubah berurutan dari "memeriksa" ke OK/gagal (stagger 80 ms) |
| RB-05 | Jejak keputusan | Baris LULUS/GAGAL muncul berurutan seperti "dievaluasi satu per satu" (stagger 60 ms) — membantu pengurus memahami logika |
| RB-06 | Hasil replay/simulasi | Bilah perbandingan v-aktif vs draft tumbuh dari 0 (600 ms, sekali) |
| RB-07 | Diff versi | Teks yang ditambah/dihapus diberi latar hijau/merah yang muncul fade |

### 5.3 POS (Kasir)

Kendala: perangkat Android 2 GB RAM, kasir mengetik/scan cepat. Semua animasi ≤ 150 ms kecuali umpan balik sukses.

| ID | Interaksi | Animasi | Suara / getar |
|---|---|---|---|
| POS-A1 | Scan berhasil / item ditambah | Baris baru slide-in dari atas + kilatan latar hijau muda 300 ms; jika item sama ditambah, angka qty "melompat" (scale 1→1,15→1) | Bip pendek |
| POS-A2 | Scan gagal (barcode tidak dikenal) | Kolom scan bergetar ±4 px + border merah 1 s; teks error tetap sampai scan berikutnya | Bip rendah ganda |
| POS-A3 | Total berubah | Nilai total kilatan warna lembut 200 ms (**tanpa** hitung naik agar langsung terbaca) | — |
| POS-A4 | Anggota teridentifikasi | Kartu pembeli flip/fade masuk menampilkan nama & tier; label "harga anggota" muncul pada item yang berubah harganya | Bip pendek |
| POS-A5 | Hapus item | Baris geser ke kiri lalu collapse (150 ms); tombol "Urungkan" 5 s | — |
| POS-A6 | Buka panel pembayaran | Panel geser dari kanan (200 ms) | — |
| POS-A7 | Pembayaran ditambahkan | Baris rincian pembayaran slide-in; bilah "terbayar" mengisi sesuai porsi | — |
| POS-A8 | Transaksi selesai | Centang besar tergambar (400 ms) + kembalian ditampilkan besar; layar kembali ke keranjang kosong otomatis setelah 2 s atau saat scan berikutnya | Nada sukses |
| POS-A9 | Menunggu QRIS | QR dengan cincin hitung mundur (progress ring) | — |
| POS-A10 | Offline/online | Banner status geser turun & berubah warna; ikon sinkron berputar saat mengunggah, berhenti saat selesai | — |
| POS-A11 | Antrean sinkron | Lencana angka berkurang dengan animasi angka saat terunggah | — |
| POS-A12 | Butuh PIN supervisor | Dialog scale-in; PIN salah → titik PIN bergetar | Getar (Android) |

**Mode performa POS:** pengaturan perangkat "Animasi: Penuh / Minimal / Mati". Default "Minimal" untuk perangkat
yang terdeteksi lambat (benchmark singkat saat pairing: rata-rata frame > 20 ms → Minimal).

### 5.4 Aplikasi Anggota (Mobile)

| ID | Interaksi | Animasi | Haptics |
|---|---|---|---|
| APP-A1 | Splash → beranda | Logo koperasi fade + kartu saldo naik dengan spring | — |
| APP-A2 | Saldo tersembunyi/tampil | Ikon mata; angka blur → jelas (200 ms) | Ringan |
| APP-A3 | Tarik untuk menyegarkan | Indikator spring + kartu saldo memantul halus setelah data baru | Ringan saat lepas |
| APP-A4 | Kartu anggota | Kartu membalik (flip 3D 400 ms) dari tampilan depan ke QR; QR dinamis menampilkan cincin waktu 60 s | Ringan |
| APP-A5 | Daftar mutasi | Item muncul stagger (maks 8); grup tanggal sticky | — |
| APP-A6 | Simulasi pinjaman | Slider nominal/tenor → angka angsuran berubah halus (interpolasi 150 ms, nilai akhir selalu tepat) | Ketukan per kelipatan Rp 500 rb |
| APP-A7 | Status pengajuan | Stepper progres; langkah yang selesai mengisi dengan animasi garis | — |
| APP-A8 | Pembayaran berhasil | Centang tergambar + kartu bukti naik | Sukses |
| APP-A9 | Slip SHU diterima | Kartu SHU muncul dengan efek kilau (shine) **sekali**; konfeti ringan opsional (≤ 1,2 s, dapat dimatikan) | Sukses |
| APP-A10 | Bottom sheet (filter, konfirmasi) | Spring gentle; dapat diseret untuk tutup | — |
| APP-A11 | Status kosong / offline | Ilustrasi Lottie ringan sekali putar | — |

## 6. Pola Status Komponen (wajib di semua layar)

| Status | Tampilan | Animasi |
|---|---|---|
| Memuat | Skeleton mengikuti bentuk konten (bukan spinner layar penuh) | BO-02 |
| Kosong | Ilustrasi + kalimat penjelas + 1 tindakan utama ("Daftarkan anggota pertama") | Fade-in |
| Error | Pesan pola dok. 09 §9 + tombol coba lagi + kode ref. | Fade-in; tanpa getar untuk error halaman |
| Sebagian | Data lama tetap tampil dengan label "Data per 09:41" saat gagal memuat ulang | — |
| Berhasil | Toast/centang sesuai konteks | BO-10 / POS-A8 / APP-A8 |
| Offline | Banner tetap, aksi yang butuh online dinonaktifkan dengan tooltip penjelasan | POS-A10 |
| Nonaktif | Kontras dikurangi + tooltip alasan ("Butuh izin Bendahara") | — |

## 7. Aksesibilitas Gerak

1. Hormati `prefers-reduced-motion` (web) dan "Kurangi gerakan" (Android/iOS) secara otomatis; sakelar manual tersedia.
2. Tidak ada elemen yang berkedip > 3 kali per detik.
3. Animasi tidak pernah menjadi satu-satunya penanda status (selalu ada teks/ikon).
4. Setelah modal/drawer terbuka, fokus dipindah ke elemen pertama; setelah tertutup, fokus kembali ke pemicu.
5. Pengumuman perubahan penting via `aria-live` (mis. "Item ditambahkan, total Rp 193.550").
6. Suara POS dapat dimatikan; volume mengikuti sistem.

## 8. Anggaran Kinerja

| Permukaan | Target |
|---|---|
| Back-office | LCP < 2,5 s, INP < 200 ms, CLS < 0,1 pada desktop kelas menengah; animasi tidak menyebabkan layout shift |
| POS | Tambah item → tampil < 100 ms; tidak ada frame > 50 ms saat scan beruntun 3 item/detik di perangkat referensi (Android Go, 2 GB) |
| Aplikasi anggota | Cold start < 3 s di Android entry-level; animasi 60 fps (≥ 50 fps di perangkat referensi) |
| Ukuran tambahan | Pustaka animasi web ≤ 40 KB gzip (dimuat terpisah per rute); POS 0 KB pustaka animasi; total aset Lottie ≤ 150 KB |

## 9. Kriteria Penerimaan Animasi

- [ ] Durasi & easing hanya dari token `@kop/motion`
- [ ] Lulus uji `prefers-reduced-motion` (Playwright `reducedMotion: 'reduce'`) — tidak ada animasi gerak
- [ ] Interaksi input tidak tertunda: E2E POS scan 20 item beruntun lulus tanpa item hilang/terlambat
- [ ] Profil kinerja (Chrome DevTools / Android GPU profiler) di perangkat referensi memenuhi §8
- [ ] Animasi dapat diinterupsi (klik saat drawer membuka tidak menyebabkan status kacau)
- [ ] Ditinjau desainer sebelum merge (rekaman video pendek dilampirkan di PR)

## 10. Deliverable Desain (Fase 0)

| Deliverable | Pemilik | Kapan |
|---|---|---|
| Figma: token warna, tipografi, spasi, radius, elevasi | UX | Minggu 2 |
| Figma: komponen inti (tombol, input, tabel, kartu, modal, drawer, toast, stepper, skeleton, status kosong) | UX | Minggu 3 |
| Spesifikasi gerak: prototipe interaktif BO-05/06/07/10, POS-A1/A2/A8, APP-A4 | UX | Minggu 4 |
| Paket `@kop/ui` & `@kop/motion` (token + preset + `useReducedMotion`) | FE | Minggu 4 |
| Storybook dengan semua status komponen & mode kurangi animasi | FE | Minggu 6 |

[← 10 Spesifikasi Teknis](./10-spesifikasi-teknis-detail.md) · [Lanjut ke 12 — Panduan Delegasi →](./12-panduan-delegasi-claude-code.md)
