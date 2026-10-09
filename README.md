# LAPORAN PRAKTIKUM
## Pengenalan Git dan GitHub

**Nama:Francisco G I dc Corbafo**  
**NIM:255410003** 
**Kelas:IF-1** 
**Mata Kuliah:Praktikum Sistem Terdistribusi Dan Terdesentralisasi** 
**Tanggal Praktikum:09 October 2026** 

---

## Ilustrasi Praktikum

![Infografik laporan praktikum Git dan GitHub](git-github-infografik.png)

*Gambar 1. Ringkasan alur kerja Git dan GitHub. Gambar ini merupakan ilustrasi pendukung; sesuaikan laporan dengan langkah dan hasil praktik yang benar-benar dilakukan.*

## 1. Pendahuluan

Dalam pengembangan perangkat lunak, perubahan kode perlu dikelola dengan rapi agar riwayat pekerjaan dapat ditelusuri dan kolaborasi berjalan lebih mudah. Git adalah sistem pengendalian versi yang digunakan untuk mencatat perubahan pada berkas. GitHub adalah layanan daring untuk menyimpan repository Git dan mendukung kerja sama antaranggota tim.

## 2. Tujuan Praktikum

1. Memahami fungsi dasar Git dan GitHub.
2. Mengetahui cara memasang dan mengonfigurasi Git.
3. Membuat repository dan menyalinnya ke komputer lokal.
4. Mempraktikkan perintah `status`, `add`, `commit`, `push`, dan `pull`.
5. Memahami branch, merge, fork, dan pull request.
6. Mengetahui cara menangani perubahan yang tidak diinginkan dan konflik.

## 3. Dasar Teori

### 3.1 Git

Git merupakan sistem pengendalian versi terdistribusi. Git menyimpan riwayat perubahan sehingga pengguna dapat meninjau atau membandingkan versi berkas.

### 3.2 GitHub

GitHub merupakan platform daring untuk meng-host repository Git. GitHub menyediakan fitur kolaborasi seperti issues, pull request, dan code review.

### 3.3 Istilah penting

- **Repository:** tempat penyimpanan proyek beserta riwayat versinya.
- **Working directory:** folder tempat pengguna mengubah berkas.
- **Staging area:** daftar perubahan yang disiapkan untuk commit.
- **Commit:** rekaman perubahan pada repository.
- **Branch:** jalur pengembangan terpisah dari branch lain.
- **Merge:** menggabungkan perubahan dari satu branch ke branch lain.
- **Remote:** repository yang berada di lokasi lain, misalnya GitHub.
- **Fork:** salinan repository ke akun GitHub sendiri.
- **Pull request:** usulan untuk menggabungkan perubahan ke repository atau branch tujuan.

## 4. Alat dan Bahan

1. Komputer atau laptop.
2. Koneksi internet.
3. Git yang dapat diunduh dari [git-scm.com](https://git-scm.com/downloads).
4. Akun GitHub di [github.com](https://github.com/).
5. Terminal, Git Bash, atau terminal pada Visual Studio Code.

## 5. Langkah Kerja

### 5.1 Instalasi dan pemeriksaan Git

1. Unduh Git dari [halaman resmi Git](https://git-scm.com/downloads).
2. Jalankan installer dan ikuti petunjuk instalasi.
3. Buka Git Bash atau terminal.
4. Periksa versi Git:

```bash
git --version
```

Jika versi Git ditampilkan, instalasi dapat dikenali oleh terminal.

### 5.2 Konfigurasi identitas

Atur nama dan email yang akan dicatat pada commit. Ganti contoh berikut dengan identitas yang sesuai.

```bash
git config --global user.name "Nama Anda"
git config --global user.email "email@example.com"
git config --global --list
```

### 5.3 Membuat repository di GitHub

1. Masuk ke akun GitHub.
2. Pilih **New repository**.
3. Isi nama repository, misalnya `latihan-git`.
4. Tentukan akses repository (Public atau Private).
5. Klik **Create repository**.

### 5.4 Clone repository ke komputer

Salin URL HTTPS repository dari GitHub, lalu jalankan perintah berikut dengan URL repository milik sendiri.

```bash
git clone https://github.com/USERNAME/latihan-git.git
cd latihan-git
```

Perintah `clone` mengunduh repository ke komputer, sedangkan `cd` masuk ke folder proyek.

### 5.5 Memeriksa perubahan

```bash
git status
```

Perintah ini menunjukkan branch aktif dan status berkas, termasuk perubahan yang belum disiapkan atau belum di-commit.

### 5.6 Menambahkan dan melakukan commit

Buat atau ubah berkas, misalnya `README.md`, lalu jalankan:

```bash
git add README.md
git commit -m "Menambahkan README"
```

Gunakan `git add .` untuk menyiapkan semua perubahan di folder saat ini, setelah memastikan berkas yang tidak diperlukan atau rahasia tidak ikut ditambahkan.

### 5.7 Mengirim perubahan ke GitHub

```bash
git push -u origin main
```

Perintah ini mengirim commit pada branch `main` ke remote bernama `origin`. Jika branch utama repository menggunakan nama lain, sesuaikan nama branchnya.

### 5.8 Mengambil perubahan terbaru

```bash
git pull
```

`git pull` mengambil perubahan dari remote dan mengintegrasikannya ke branch aktif. Pastikan perubahan lokal sudah tersimpan atau diamankan sebelum menarik perubahan.

### 5.9 Membuat branch dan berpindah branch

```bash
git switch -c fitur-readme
```

Buat perubahan pada branch tersebut, lalu simpan perubahan:

```bash
git add README.md
git commit -m "Memperbarui README pada branch fitur"
git push -u origin fitur-readme
```

Setelah branch dikirim ke GitHub, perubahan dapat diajukan melalui pull request.

### 5.10 Pull request dan merge

1. Buka repository di GitHub.
2. Pilih menu **Pull requests** dan buat pull request baru.
3. Periksa perubahan yang diajukan.
4. Jika perubahan sudah benar dan disetujui, lakukan merge sesuai aturan proyek.

Pull request membantu anggota tim meninjau perubahan sebelum digabungkan.

### 5.11 Fork dan sinkronisasi repository

Untuk berkontribusi ke repository yang bukan milik sendiri, fork repository ke akun GitHub, kemudian clone fork tersebut ke komputer. Jika diperlukan, tambahkan repository asli sebagai remote `upstream`:

```bash
git remote add upstream https://github.com/PEMILIK/REPOSITORY.git
git fetch upstream
git switch main
git merge upstream/main
```

Ganti URL dan nama branch sesuai repository sumber. Kirim perubahan ke fork menggunakan `git push`, lalu ajukan pull request ke repository asli.

### 5.12 Membatalkan perubahan dan menangani konflik

Untuk membuang perubahan pada satu berkas yang belum di-commit, gunakan dengan hati-hati:

```bash
git restore README.md
```

Perintah tersebut membuang perubahan lokal yang belum di-stage pada berkas itu. Pastikan tidak ada perubahan yang masih dibutuhkan.

Untuk membatalkan commit yang sudah dibagikan, umumnya lebih aman membuat commit pembalik:

```bash
git revert HEAD
```

Jika terjadi konflik saat merge atau pull, buka berkas yang konflik, pilih dan rapikan isi yang benar, hapus penanda konflik seperti `<<<<<<<`, `=======`, dan `>>>>>>>`, lalu simpan dan lanjutkan:

```bash
git add README.md
git commit -m "Menyelesaikan konflik"
```

Perintah lanjutan dapat berbeda tergantung apakah konflik terjadi pada merge, rebase, atau cherry-pick. Jangan gunakan `git reset --hard` kecuali benar-benar memahami bahwa perubahan lokal dapat hilang.

### 5.13 Melihat riwayat commit

```bash
git log --oneline
```

Perintah ini menampilkan ringkasan riwayat commit sehingga perubahan dapat ditelusuri.

## 6. Hasil dan Pembahasan

Praktik Git dan GitHub bertujuan menghasilkan alur kerja yang dapat diulang: membuat atau mengkloning repository, mengubah berkas, memeriksa status, menambahkan perubahan ke staging area, membuat commit, dan mengirim commit ke remote. Branch dan pull request mendukung pengembangan fitur secara terpisah serta peninjauan sebelum perubahan digabungkan.

**Catatan hasil praktik:** Lengkapi bagian ini berdasarkan hasil yang benar-benar muncul di terminal dan GitHub. Tambahkan screenshot asli, misalnya:

- Gambar 2. Hasil `git --version`.
- Gambar 3. Repository di GitHub.
- Gambar 4. Hasil `git status`, `git add`, dan `git commit`.
- Gambar 5. Hasil `git push` dan berkas yang tampil di GitHub.
- Gambar 6. Branch atau pull request jika praktik tersebut dilakukan.

Jangan menyatakan suatu langkah berhasil apabila belum dilakukan atau belum diperiksa.

## 7. Kesimpulan

Git membantu mengelola riwayat perubahan berkas dan memungkinkan pengguna kembali meninjau versi sebelumnya. GitHub menyediakan tempat daring untuk menyimpan repository serta mendukung kolaborasi melalui branch, fork, dan pull request. Penggunaan perintah Git secara teliti membantu menjaga perubahan tetap terorganisasi. Pemahaman tentang konflik dan pembatalan perubahan juga penting agar pekerjaan tidak hilang.

## 8. Daftar Pustaka

1. Git. **Git Documentation**. https://git-scm.com/doc
2. GitHub Docs. **GitHub Documentation**. https://docs.github.com/
3. NEO-X-School. **Petunjuk Git dan GitHub**. https://github.com/NEO-X-School/notes/tree/main/petunjuk-git-github

---

*Laporan ini merupakan templat praktikum. Lengkapi identitas, tanggal, dan hasil pengamatan sesuai kegiatan yang benar-benar dilakukan. Simpan berkas gambar `git-github-infografik.png` dalam folder yang sama dengan README.md agar gambar tampil saat README dibuka di GitHub.*
