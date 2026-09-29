# web_presensi_QR
DOKUMENTASI WEBSITE PRESENSI QR
================================

dosen-presensi/
├── login.php
├── dashboard.php
├── jadwal.php
├── generate_qr.php
├── absensi.php
├── laporan.php
├── export_excel.php
├── export_pdf.php
├── export_word.php
├── profil.php
├── logout.php
├── config/database.php
├── includes/header.php
├── includes/navbar.php
├── includes/footer.php
├── assets/css/style.css
├── assets/js/script.js
└── assets/images/

mhs-presensi/
├── login.php
├── dashboard.php
├── scan_qr.php
├── proses_scan.php
├── riwayat.php
├── jadwal.php
├── profil.php
├── logout.php
├── config/database.php
├── includes/header.php
├── includes/navbar.php
├── includes/footer.php
├── assets/css/style.css
├── assets/js/script.js
└── assets/images/

1. GAMBARAN UMUM
-----------------
Website Presensi QR adalah aplikasi presensi perkuliahan berbasis PHP native dan MySQL.
Aplikasi ini terdiri dari dua portal yang menggunakan database yang sama:

1) Portal Dosen
   Folder: dosen-presensi/
   Digunakan untuk mengelola jadwal, membuat QR Code presensi, memantau kehadiran,
   mengelola data mahasiswa, dan membuat laporan.

2) Portal Mahasiswa
   Folder: mhs-presensi/
   Digunakan untuk login mahasiswa, melakukan scan QR Code, melihat jadwal,
   melihat riwayat presensi, dan mengelola profil.

Kedua portal saling terhubung melalui database presensi_qr.


2. TEKNOLOGI YANG DIGUNAKAN
---------------------------
- PHP native untuk logika aplikasi dan halaman web.
- MySQL untuk menyimpan akun, jadwal, sesi QR, dan data presensi.
- HTML, CSS, Bootstrap, dan Bootstrap Icons untuk tampilan.
- JavaScript serta library html5-qrcode untuk membaca QR Code dari kamera.
- Composer dan Dompdf untuk beberapa kebutuhan ekspor laporan.
- XAMPP sebagai server lokal Apache dan MySQL.


3. CARA KERJA WEBSITE SECARA UMUM
----------------------------------
Alur utama website adalah sebagai berikut:

1) Dosen membuat jadwal kuliah.
2) Pada waktu kuliah berlangsung, dosen memilih jadwal dan membuat QR Code.
3) Sistem membuat token acak untuk sesi presensi.
4) Token tersebut ditampilkan dalam bentuk QR Code.
5) Mahasiswa login dan membuka halaman Scan Presensi.
6) Kamera mahasiswa membaca QR Code.
7) Sistem memeriksa token, waktu, status sesi, dan kelas mahasiswa.
8) Jika semua pemeriksaan berhasil, data presensi disimpan sebagai Hadir.
9) Mahasiswa dapat melihat hasilnya pada halaman Riwayat.
10) Dosen dapat melihat rekap kehadiran pada halaman Absensi atau Laporan.


4. LOGIKA LOGIN DOSEN
----------------------
File utama: dosen-presensi/login.php

1) Dosen mengisi email dan password.
2) Sistem mencari email tersebut pada tabel dosen.
3) Password diperiksa menggunakan password_verify().
4) Jika benar, sistem menyimpan data dosen ke dalam session:
   - dosen_id
   - dosen_nama
   - dosen_foto
5) Dosen diarahkan ke dashboard.php.
6) Jika salah, sistem menampilkan pesan bahwa email atau password salah.
7) Halaman internal memanggil fungsi cekLogin() agar hanya dapat diakses setelah login.


5. LOGIKA LOGIN MAHASISWA
--------------------------
File utama: mhs-presensi/login.php

1) Mahasiswa mengisi NIM dan password.
2) Sistem mencari NIM tersebut pada tabel mahasiswa.
3) Password diperiksa menggunakan password_verify().
4) Jika benar, sistem menyimpan data mahasiswa ke dalam session:
   - mhs_id
   - mhs_nama
   - mhs_nim
   - mhs_kelas
5) Mahasiswa diarahkan ke dashboard.php.
6) Jika salah, sistem menampilkan pesan bahwa NIM atau password salah.
7) Halaman internal menggunakan cekLogin() agar hanya dapat dibuka oleh mahasiswa
   yang sudah login.


6. LOGIKA PEMBUATAN QR CODE
---------------------------
File utama: dosen-presensi/generate_qr.php

1) Dosen memilih jadwal kuliah yang tersedia.
2) Sistem mengambil data jadwal dari tabel jadwal.
3) Sistem membandingkan waktu sekarang dengan tanggal, jam mulai, dan jam selesai.
4) QR hanya dapat dibuat ketika jadwal sudah dimulai dan belum melewati batas waktu.
5) Batas presensi adalah sampai 15 menit setelah jam selesai kuliah.
6) Sistem membuat token acak menggunakan random_bytes(16), kemudian mengubahnya
   menjadi format hexadecimal.
7) Sesi QR lama yang masih aktif diubah menjadi expired.
8) Sesi baru disimpan ke tabel qr_session dengan data:
   - jadwal_id
   - token
   - tanggal
   - waktu_buka
   - waktu_tutup
   - status aktif
9) Token dimasukkan ke URL scan mahasiswa.
10) URL tersebut diubah menjadi gambar QR Code dan ditampilkan kepada dosen.

Contoh bentuk URL:
https://mhs-presensi.rf.gd/scan_qr.php?token=TOKEN_RAHASIA


7. LOGIKA PEMINDAIAN QR OLEH MAHASISWA
--------------------------------------
File utama:
- mhs-presensi/scan_qr.php
- mhs-presensi/proses_scan.php

A. Pada halaman scan_qr.php

1) Sistem memuat library html5-qrcode.
2) Browser meminta izin penggunaan kamera.
3) Kamera membaca isi QR Code.
4) Jika QR berisi URL, sistem mengambil nilai token dari parameter URL.
5) Jika QR bukan URL, isi QR dianggap langsung sebagai token.
6) Token dikirim ke proses_scan.php menggunakan request.

B. Pada proses_scan.php

Sistem melakukan pemeriksaan berikut secara berurutan:

1) Memastikan mahasiswa sudah login.
2) Memastikan token tidak kosong.
3) Mencari token pada tabel qr_session.
4) Memastikan QR Code ditemukan.
5) Memastikan status sesi masih aktif.
6) Memastikan tanggal scan sama dengan tanggal sesi.
7) Memastikan waktu sekarang berada di antara waktu_buka dan waktu_tutup.
8) Jika waktu sudah habis, status sesi diubah menjadi expired.
9) Mengambil kelas mahasiswa yang sedang login.
10) Membandingkan kelas mahasiswa dengan kelas pada jadwal QR.
11) Jika kelas berbeda, presensi ditolak.
12) Memeriksa apakah mahasiswa sudah melakukan presensi pada jadwal dan tanggal
    yang sama.
13) Jika sudah ada, presensi ditolak agar tidak terjadi presensi ganda.
14) Jika semua pemeriksaan berhasil, data disimpan ke tabel presensi dengan status
    Hadir.
15) Sistem mengirim hasil dalam format JSON.
16) Jika berhasil, mahasiswa diarahkan ke halaman riwayat setelah beberapa detik.


8. DATA YANG DISIMPAN SAAT PRESENSI BERHASIL
---------------------------------------------
Tabel: presensi

Data yang disimpan:
- mahasiswa_id   : identitas mahasiswa.
- jadwal_id      : jadwal kuliah yang diikuti.
- qr_session_id  : sesi QR yang digunakan.
- tanggal        : tanggal presensi.
- jam_presensi   : waktu mahasiswa melakukan scan.
- status         : biasanya Hadir.
- created_at     : waktu data dibuat.

Database juga memiliki aturan unik pada kombinasi mahasiswa_id, jadwal_id, dan
tanggal. Aturan ini mencegah satu mahasiswa melakukan presensi dua kali pada
jadwal yang sama di tanggal yang sama.


9. TABEL DATABASE UTAMA
-----------------------
1) dosen
   Menyimpan nama, email, password, dan foto dosen.

2) mahasiswa
   Menyimpan NIM, nama, email, password, kelas, jurusan, dan foto mahasiswa.

3) jadwal
   Menyimpan mata kuliah, kelas, tanggal, hari, jam mulai, dan jam selesai.

4) qr_session
   Menyimpan token QR, jadwal terkait, waktu buka, waktu tutup, dan status sesi.

5) presensi
   Menyimpan catatan kehadiran mahasiswa.

Relasi penting:
- qr_session.jadwal_id terhubung ke jadwal.id.
- presensi.mahasiswa_id terhubung ke mahasiswa.id.
- presensi.jadwal_id terhubung ke jadwal.id.


10. FITUR PORTAL DOSEN
----------------------
- Dashboard: menampilkan ringkasan total mahasiswa dan statistik presensi.
- Jadwal: menambah, mengubah, dan menghapus jadwal kuliah.
- Generate QR: membuat QR Code untuk sesi kuliah aktif.
- Absensi: melihat data presensi mahasiswa.
- Laporan: melihat rekap presensi berdasarkan kebutuhan.
- Export Excel, PDF, dan Word: mengunduh laporan.
- Mahasiswa: mengelola data mahasiswa.
- Profil: mengelola data profil dosen.


11. FITUR PORTAL MAHASISWA
--------------------------
- Halaman index: menjelaskan fungsi website dan cara penggunaannya.
- Login: autentikasi menggunakan NIM dan password.
- Dashboard: menampilkan informasi akun mahasiswa.
- Scan QR: melakukan presensi menggunakan kamera.
- Riwayat: melihat daftar presensi yang pernah dilakukan.
- Jadwal: melihat jadwal perkuliahan.
- Profil: melihat atau mengubah informasi profil.
- Logout: menghapus session dan keluar dari sistem.


12. KEAMANAN DAN VALIDASI
-------------------------
- Password tidak dibandingkan sebagai teks biasa, tetapi diperiksa dengan
  password_verify().
- Token QR dibuat secara acak dan disimpan pada tabel qr_session.
- QR memiliki waktu berlaku sehingga tidak dapat digunakan selamanya.
- QR hanya berlaku untuk kelas yang sesuai dengan kelas mahasiswa.
- Presensi ganda pada jadwal dan tanggal yang sama ditolak.
- Halaman internal dilindungi oleh session login.
- Query dengan input penting menggunakan prepared statement.

Catatan: penggunaan website melalui localhost atau HTTPS diperlukan agar browser
mengizinkan akses kamera pada fitur scan QR.


13. CARA MENJALANKAN DI XAMPP
-----------------------------
1) Simpan folder proyek di:
   C:\xampp\htdocs\presensi_QR

2) Jalankan Apache dan MySQL dari XAMPP Control Panel.

3) Buka phpMyAdmin:
   http://localhost/phpmyadmin/

4) Buat database bernama:
   presensi_qr

5) Import file presensi_qr.sql.

6) Pastikan konfigurasi pada config/database.php sesuai:
   host     = localhost
   user     = root
   password = kosong secara bawaan XAMPP
   database = presensi_qr

7) Buka portal mahasiswa:
   http://localhost/presensi_QR/mhs-presensi/

8) Buka portal dosen:
   http://localhost/presensi_QR/dosen-presensi/


14. RINGKASAN SINGKAT LOGIKA SISTEM
------------------------------------
Dosen membuat jadwal dan QR Code. QR Code berisi token sesi yang memiliki batas
waktu. Mahasiswa melakukan scan menggunakan kamera. Server memvalidasi login,
token, status sesi, tanggal, waktu, kelas, dan duplikasi presensi. Jika valid,
data kehadiran disimpan sebagai Hadir. Data tersebut kemudian dapat dilihat
mahasiswa melalui riwayat dan dipantau dosen melalui absensi atau laporan.

