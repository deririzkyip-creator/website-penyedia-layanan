# Website Profil CI4 (XAMPP + CodeIgniter 4 + Bootstrap)

Aplikasi ini menyediakan:
- Beranda
- Tentang
- Kontak
- Login admin
- Panel admin untuk edit konten Beranda (berita), Tentang, dan Kontak

Struktur folder:
- public_html/ -> folder public (diakses browser)
- diluar/ci4/ -> folder aplikasi CodeIgniter 4 (app, database, vendor, writable, dll)

## Cara Menjalankan (XAMPP)

1. Aktifkan Apache dan MySQL dari XAMPP.
2. Buka phpMyAdmin, lalu import file diluar/ci4/database/web.sql.
3. Pastikan konfigurasi DB di diluar/ci4/app/Config/Database.php sesuai.
4. Akses website di browser:
   - http://localhost/my%20web/public_html/

## Login Admin Default
- Username: admin
- Password: admin123

## Catatan
- Front controller public sudah diarahkan dari public_html/index.php ke aplikasi CI4 di diluar/ci4.
- Struktur lama di diluar/app, diluar/config, diluar/database, dan diluar/bootstrap.php sudah dihapus agar tidak duplikat.
- Jika route tidak berjalan, pastikan modul rewrite Apache aktif.

## Struktur Final Untuk Hosting

Gunakan pola berikut agar source aplikasi tidak bisa diakses langsung dari web:

```
public_html/
   index.php
   .htaccess
   assets/
diluar/
   ci4/
      app/
      vendor/
      writable/
      database/
```

Langkah deploy ringkas:
1. Upload isi folder public_html lokal ke public_html hosting.
2. Upload folder diluar/ci4 lokal ke folder non-public hosting (tetap di luar public_html).
3. Pastikan path di public_html/index.php menunjuk ke ../diluar/ci4.
4. Import database dari diluar/ci4/database/web.sql.
5. Set konfigurasi DB di diluar/ci4/app/Config/Database.php.
6. Pastikan folder diluar/ci4/writable dapat ditulis server.
