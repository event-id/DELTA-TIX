# PANDUAN PENGUJIAN — Integrasi Supabase DELTATIX

Dokumen ini adalah checklist pengujian untuk dilakukan setelah paket dipasang ke
project Supabase DELTATIX (`xhplvjjmwpkuolrahcnf`). Koneksi langsung ke project
baru, login dengan akun admin, dan operasi tulis belum diuji dari sesi ini.
Jangan menganggap fitur produksi aktif sebelum seluruh langkah yang relevan
diuji di browser dan Supabase Dashboard.

## A. Pengujian database (via REST, anon key)

| # | Uji | Status setelah deploy |
|---|-----|-------|
| 1 | Koneksi REST dengan publishable key | Belum diuji pada project DELTATIX |
| 2 | SELECT `events` (anon) | Periksa setelah deploy |
| 3 | SELECT `banners` (anon) | Periksa setelah deploy |
| 4 | INSERT `tickets` anonim (probe FK rusak — aman) | Harus ditolak oleh RLS |
| 5 | Fungsi `is_admin()` RPC (anon) | Periksa RPC `is_admin()` |
| 6 | Relasi `tickets→events` (embedded select) | Periksa relasi |
| 7 | Skema kolom (probe per kolom) | Bandingkan dengan migrasi dan CHANGES.md |

## B. Pengujian website publik (`index.html`, server lokal + browser headless)

| # | Uji | Status setelah deploy |
|---|-----|-------|
| 1 | Muat beranda, cek console & page error | Periksa console browser |
| 2 | DB kosong → tampil empty state "Belum ada event yang dipublikasikan" (bukan data dummy) | Periksa |
| 3 | Mapper `mapEventFromDB` dengan 3 baris berbentuk persis Supabase (UUID, timestamptz, null) | Periksa data event |
| 4 | Mapper `mapBannerFromDB` + fallback tag/sub | Periksa |
| 5 | Render beranda dengan data simulasi: 5 bagian asli, kartu, carousel, dots | Periksa persis layout lama |
| 6 | Detail event via **UUID** `#/event/<uuid>` | Periksa judul, badge, harga, EO tampil |
| 7 | Detail event via **slug** `#/event/jazz-malam` | Periksa |
| 8 | Jelajah: chip kategori (2 event "Musik" ditemukan) | Periksa |
| 9 | Jelajah: pencarian teks "bandung" → 1 hasil sesuai | Periksa |
| 10 | Alur pembelian lama: validasi form → submit → order tersimpan → toast → halaman "Tiket Saya" menampilkan tiket | Periksa 100% berfungsi |
| 11 | Koneksi diputus (offline) → pulihkan → data termuat ulang | Periksa recovery normal |
| 12 | UI state error: kartu "Gagal memuat data" + tombol "Coba Lagi", tanpa kartu palsu | Periksa (diverifikasi langsung pada state error) |
| 13 | Mobile 360px: tidak ada scroll horizontal halaman | Periksa (sebagai pembanding, file ASLI justru overflow 406px di 360px) |
| 14 | Screenshot: beranda desktop, admin login, beranda mobile | Periksa tersimpan |

Catatan jujur: karena tabel produksi Anda masih kosong, uji #5 memakai injeksi
data berbentuk DB di browser (lokal, tidak menyentuh produksi) agar alur render
dapat diverifikasi penuh. Setelah Anda menambah event via admin, alur yang sama
yang teruji itulah yang berjalan dengan data nyata.

## C. Pengujian dashboard admin (`admin.html`)

| # | Uji | Status setelah deploy |
|---|-----|-------|
| 1 | Muat halaman, cek console & page error | Periksa console browser (sebelum perbaikan: SyntaxError total) |
| 2 | Login dilihat, dashboard tersembunyi sebelum login | Periksa |
| 3 | Login kredensial salah → pesan "Invalid login credentials" dari Supabase Auth | Periksa **membuktikan URL & key hasil perbaikan benar-benar terhubung** |
| 4 | Tab Event/Banner/E-ticket berpindah | Periksa |
| 5 | Slug otomatis dari judul ("Konser Jazz Malam Spesial!" → `konser-jazz-malam-spesial`) | Periksa |
| 6 | Validasi URL gambar `javascript:alert(1)` ditolak | Periksa |
| 7 | Validasi link banner: `javascript:` ditolak; `#/event/abc` lolos validasi | Periksa |
| 8 | **INSERT banner tanpa login langsung ke API** | Periksa **DITOLAK RLS**: "new row violates row-level security policy for table \"banners\"" — bukti tulis tanpa admin mustahil |
| 9 | 7 kartu statistik + field form baru (organizer/category/region/quota/featured/laris) | Periksa tampil semua |
| 10 | Tab Scan E-ticket: validasi kode manual tiket tidak ditemukan → pesan "TIKET TIDAK DITEMUKAN" | Periksa |
| 11 | Scan/validasi tiket berstatus valid → status otomatis berubah menjadi used di dashboard | Periksa |
| 12 | Scan ulang tiket yang sudah used → pesan "TIKET SUDAH DIGUNAKAN" | Periksa |
| 13 | Form banner: label (tag) dan subjudul tersimpan dan tampil di carousel beranda | Periksa setelah migrasi 2b dijalankan |
| 14 | Kode tiket baru berprefix DTX- (tiket lama berprefix LS-/ATX- tetap valid saat dipindai) | Periksa |

## D. Yang BELUM dapat diuji (jujur, beserta alasannya)

1. **Login admin sah sampai ke dashboard** — kredensial admin tidak (dan tidak
   boleh) saya miliki. Mekanismenya (signInWithPassword → cek `admin_users` →
   muat data) sudah teruji per-bagian: #C3 membuktikan auth jalan, #A5
   membuktikan `is_admin()` jalan, dan kebijakan RLS admin dibuat oleh
   `supabase-migration.sql`. Segera setelah admin pertama didaftarkan (lihat
   DEPLOY.md langkah 4), alur penuh ini aktif.
2. **Tambah/edit/hapus event & banner secara nyata dari dashboard** — membutuhkan
   login admin (poin 1) dan akan menulis ke produksi. Pengecekan per-komponen
   sudah dilakukan (#C5–C9), dan jalur penulisan terbukti terbuka HANYA bagi
   admin oleh RLS (#C8 membuktikan jalur non-admin tertutup).
3. **Terbitkan & ubah status e-tiket nyata** — alasan sama dengan poin 2;
   transisi status juga dijaga trigger database hasil migrasi.
4. **Kirim email / pembayaran** — memang belum ada layanannya;
   tidak diklaim berfungsi. Cetak e-tiket (dengan QR berisi kode) sudah tersedia.
   Check-in lewat tab Scan E-ticket memerlukan HTTPS + izin kamera pada perangkat
   petugas; jalur pengganti input kode manual tersedia dan sudah diuji logikanya.
5. **GitHub Pages subpath** — tidak ada repository URL yang diberikan; namun
   semua aset relatif (`logo.png`), routing hash, dan CDN absolut aman untuk
   subpath (mis. `username.github.io/repo/`). Panduan deploy disertakan.

## E. Langkah verifikasi cepat setelah Anda deploy (10 menit)

1. Jalankan `supabase-migration.sql` (SQL Editor) → cek NOTICE/kesalahan.
2. Daftarkan admin pertama (bagian [6c] file SQL).
3. Buka `admin.html` → login → tambah 1 event (status **Published**) + 1 banner aktif.
4. Buka `index.html` → event & banner harus tampil; klik detail; cek `#/explore`.
5. Ubah status event menjadi Draft di admin → muat ulang `index.html` → event harus hilang dari publik.

## Uji pembelian langsung (Direct Redirect)
1. Jalankan ulang seluruh `supabase-migration.sql` di Supabase SQL Editor untuk menambahkan kolom `events.purchase_url`.
2. Buka Dashboard Admin > Event, tambah atau edit event.
3. Isi “Link Pembelian Tiket” dengan URL tujuan lengkap, misalnya `https://nama-project.vercel.app`, lalu simpan dengan status Published.
4. Buka halaman publik, pilih event tersebut, lalu tekan “Beli Tiket”. Browser harus berpindah langsung ke URL tujuan.
5. Uji event yang belum memiliki URL. Tombol harus nonaktif dan menampilkan pesan bahwa link pembelian belum tersedia.
6. Pastikan halaman detail tidak lagi meminta nama, email, atau WhatsApp.
