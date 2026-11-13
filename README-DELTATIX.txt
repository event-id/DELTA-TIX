DELTATIX — WEBSITE + SUPABASE
================================

Isi paket:
- index.html: halaman publik dan katalog event dari Supabase.
- admin.html: login admin, pengelolaan event/banner/tiket, statistik, dan cetak e-ticket dengan QR.
- supabase-migration.sql: migrasi kolom, fungsi, indeks, RLS, trigger, dan kebijakan Storage.
- DEPLOY.md: panduan konfigurasi dan deployment.
- TESTING.md: checklist pengujian.
- CHANGES.md: catatan audit dan perubahan.

KONFIGURASI YANG SUDAH DIISI
Project URL: https://xhplvjjmwpkuolrahcnf.supabase.co
Publishable key: sb_publishable_... (sudah diisi pada index.html dan admin.html)

LANGKAH SETELAH EKSTRAK
1. Baca DEPLOY.md dari awal sampai akhir.
2. Pastikan tabel inti events, banners, tickets, admin_users sudah ada.
3. Jalankan supabase-migration.sql di SQL Editor pada project DELTATIX.
   Migrasi ini dirancang idempoten, tetapi baca isi file dan pastikan project benar.
4. Pastikan akun admin sudah terdaftar di Authentication dan public.admin_users.
5. Upload index.html dan admin.html beserta aset pendukung ke hosting yang sama.
6. Uji login admin, tambah/edit event, banner, terbitkan tiket, dan cetak QR.

BATASAN FITUR SAAT INI
- Check-in e-ticket tersedia di dashboard admin (tab "Scan E-ticket"): scan QR
  lewat kamera atau input kode manual. Tiket valid yang berhasil divalidasi
  otomatis berubah status menjadi "used" (update kondisional di database).
  QR pada e-ticket tetap hanya pembawa kode tiket.
- Integrasi pembayaran otomatis (payment gateway/webhook) belum disertakan.
  Alur checkout yang ada tidak boleh dianggap sebagai konfirmasi pembayaran aman.
- Publishable key memang digunakan di browser. Jangan pernah memasukkan service_role
  atau secret key ke file HTML. Keamanan tetap bergantung pada RLS dan policy.
- File logo.png tidak disertakan dalam paket ini. Bila file tersebut ada di
  hosting Anda, ikon logo akan tampil; jika tidak, ikon disembunyikan otomatis
  dan teks brand tetap muncul (fallback yang memang dirancang).

Branding halaman disesuaikan menjadi DELTATIX.


PEMBELIAN TIKET LANGSUNG (DIRECT REDIRECT)
- Buka Dashboard Admin > Event.
- Saat menambah atau mengedit event, isi kolom “Link Pembelian Tiket” dengan URL lengkap, misalnya https://nama-project.vercel.app.
- Simpan event dan pastikan statusnya Published.
- Pengunjung membuka detail event lalu menekan “Beli Tiket” untuk langsung menuju URL tersebut.
- Jika kolom link kosong, tombol pembelian akan dinonaktifkan dan halaman memberi tahu bahwa link belum tersedia.
- Setelah pembaruan kode ini, jalankan ulang supabase-migration.sql di Supabase SQL Editor agar kolom purchase_url tersedia.
