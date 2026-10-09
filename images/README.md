# prak-dis-dec
# LAPORAN PRAKTIKUM
## Pengenalan Git dan GitHub

**Nama:Franisco G I dc Corbafo**
**NIM:255410003**
**Kelas:IF-1**
**Mata Kuliah:Praktikum Sistem Terdistribusi dan Terdesentralisasi**
**Tanggal:09 October 2026**

---

## 1. Pendahuluan

Git dan GitHub merupakan alat yang sering digunakan dalam pengembangan perangkat lunak. Git adalah sistem pengontrol versi yang mencatat perubahan pada file proyek, sedangkan GitHub merupakan layanan daring untuk menyimpan repository Git dan mendukung kolaborasi.

Laporan ini menjelaskan konsep dasar Git dan GitHub, pengaturan awal, pengelolaan repository, pencatatan perubahan, penggunaan branch, kolaborasi melalui pull request, serta cara menangani kesalahan umum.

**Sumber panduan:** [NEO-X-School — Petunjuk Git dan GitHub](https://github.com/NEO-X-School/notes/tree/main/petunjuk-git-github)

## 2. Tujuan

1. Memahami perbedaan Git dan GitHub.
2. Mengetahui cara memasang dan mengatur Git.
3. Memahami cara membuat atau menyalin repository.
4. Mampu mencatat dan mengirim perubahan proyek ke GitHub.
5. Memahami penggunaan branch, merge, dan pull request.
6. Mengetahui langkah dasar untuk memeriksa dan memperbaiki kesalahan pada repository.

## 3. Dasar Teori

### 3.1 Git

Git adalah sistem pengontrol versi terdistribusi yang digunakan untuk mencatat riwayat perubahan file. Dengan Git, pengguna dapat melihat perubahan, membuat commit, menggunakan branch, dan mengembalikan perubahan tertentu sesuai kebutuhan.

### 3.2 GitHub

GitHub adalah platform daring untuk menyimpan repository Git. GitHub membantu pengguna berbagi kode, mengelola proyek, meninjau perubahan, dan bekerja bersama anggota tim.

### 3.3 Istilah Penting

- **Repository:** tempat proyek dan riwayat Git dikelola.
- **Commit:** catatan perubahan pada riwayat repository.
- **Branch:** jalur pengembangan yang terpisah dari branch lain.
- **Remote:** repository yang berada di lokasi lain, misalnya di GitHub.
- **Push:** mengirim commit lokal ke remote.
- **Pull:** mengambil dan menggabungkan perubahan dari remote.
- **Clone:** menyalin repository beserta riwayatnya ke komputer.
- **Merge:** menggabungkan perubahan dari satu branch ke branch lain.
- **Pull request:** permintaan untuk meninjau dan menggabungkan perubahan ke branch tujuan.
- **Fork:** membuat salinan repository ke akun GitHub sendiri.

## 4. Alat dan Bahan

1. Komputer atau laptop.
2. Sistem operasi Windows, Linux, atau macOS.
3. Git yang sudah terpasang.
4. Akun GitHub.
5. Terminal, Git Bash, atau terminal Visual Studio Code.
6. Koneksi internet untuk berinteraksi dengan GitHub.

## 5. Langkah Kerja

### 5.1 Memasang dan Memeriksa Git

Git dapat diunduh dari situs resmi [Git](https://git-scm.com/install/). Setelah instalasi selesai, buka Git Bash atau terminal dan jalankan:

```bash
git --version
```

Jika nomor versi ditampilkan, Git dapat dijalankan pada komputer.

### 5.2 Mengatur Identitas Pengguna

Atur nama dan email yang akan dicatat pada commit:

```bash
git config --global user.name "Nama Kamu"
git config --global user.email "emailkamu@example.com"
```

Periksa pengaturan dengan perintah:

```bash
git config --global --list
```

Gunakan nama dan email milik sendiri. Jika ingin commit terhubung dengan akun GitHub, sebaiknya gunakan alamat email yang telah ditambahkan atau diverifikasi pada akun tersebut.

### 5.3 Membuat Repository di GitHub

1. Masuk ke [GitHub](https://github.com/).
2. Pilih menu untuk membuat repository baru.
3. Isi nama repository.
4. Tentukan visibilitas repository, yaitu Public atau Private.
5. Buat repository.
6. Salin URL repository melalui tombol **Code** jika ingin menghubungkannya dengan komputer.

Untuk latihan awal, repository kosong dapat memudahkan proses push pertama dari komputer. Jika repository sudah berisi README atau commit lain, ikuti petunjuk GitHub untuk menyelaraskan riwayat lokal dan remote.

### 5.4 Menyalin Repository dengan Clone

Gunakan URL repository yang benar:

```bash
git clone https://github.com/USERNAME/NAMA-REPOSITORY.git
cd NAMA-REPOSITORY
```

Perintah `git clone` menyalin repository ke komputer, sedangkan `cd` digunakan untuk masuk ke folder proyek. Ganti bagian URL dan nama folder sesuai repository yang digunakan.

### 5.5 Memeriksa Status File

```bash
git status
```

Perintah ini menunjukkan branch aktif serta perubahan file yang belum disiapkan, sudah disiapkan, atau belum dicatat dalam commit.

### 5.6 Menambahkan dan Mencatat Perubahan

Setelah membuat atau mengedit file, jalankan:

```bash
git add -A
git commit -m "Menambahkan atau memperbarui file"
```

`git add -A` menyiapkan perubahan yang terdeteksi untuk commit. `git commit` mencatat perubahan tersebut ke riwayat lokal dengan pesan yang menjelaskan pekerjaan.

### 5.7 Mengirim Perubahan ke GitHub

Jika branch utama bernama `main` dan remote bernama `origin`, push pertama dapat dilakukan dengan:

```bash
git push -u origin main
```

Setelah hubungan branch lokal dan remote diatur, alur rutin biasanya:

```bash
git status
git add -A
git commit -m "Memperbarui proyek"
git push
```

Commit disimpan pada repository lokal. Agar perubahan terkirim ke GitHub, lakukan push.

### 5.8 Menggunakan Branch

Branch memungkinkan pekerjaan dilakukan secara terpisah dari branch utama:

```bash
git switch -c fitur-baru
```

Setelah perubahan dibuat dan di-commit, kirim branch ke GitHub:

```bash
git add -A
git commit -m "Menambahkan fitur baru"
git push -u origin fitur-baru
```

Branch tersebut dapat diajukan melalui pull request untuk diperiksa dan digabungkan ke branch tujuan.

### 5.9 Mengambil Perubahan dari Remote

Untuk mengambil dan menggabungkan perubahan dari branch yang sedang dilacak:

```bash
git pull
```

Untuk mengambil informasi terbaru tanpa langsung menggabungkannya:

```bash
git fetch
```

Sebelum menggabungkan perubahan, periksa status repository dan pastikan perubahan lokal yang belum di-commit sudah diamankan.

### 5.10 Kolaborasi dengan Fork dan Pull Request

Jika ingin berkontribusi ke repository milik pihak lain dan tidak memiliki izin push langsung, alur yang umum adalah:

1. Fork repository melalui GitHub.
2. Clone fork ke komputer.
3. Buat branch baru untuk perubahan.
4. Edit file dan buat commit.
5. Push branch ke fork.
6. Ajukan pull request ke repository asli.
7. Tanggapi hasil pemeriksaan dan selesaikan konflik jika ada.

Fork adalah salinan repository pada akun GitHub, sedangkan clone adalah salinan repository pada komputer lokal.

### 5.11 Memeriksa Riwayat dan Membatalkan Perubahan

Lihat riwayat commit secara ringkas:

```bash
git log --oneline
```

Untuk membuang perubahan pada file yang belum di-commit:

```bash
git restore nama-file
```

Perintah tersebut dapat menghilangkan perubahan lokal yang belum disimpan dalam commit, sehingga harus digunakan dengan hati-hati.

Untuk membalik commit yang sudah dibuat, terutama jika riwayat telah dibagikan:

```bash
git revert HEAD
git push
```

`git revert` membuat commit baru yang membalik perubahan dari commit yang ditentukan.

Perintah berikut dapat menghapus perubahan lokal dan memindahkan branch ke commit sebelumnya:

```bash
git reset --hard HEAD^
```

Perintah ini berisiko menghilangkan pekerjaan yang belum disimpan. Periksa status dan riwayat terlebih dahulu, dan hindari mengubah riwayat bersama tanpa koordinasi.

## 6. Hasil dan Pembahasan

Melalui materi Git dan GitHub, dapat dipahami bahwa Git membantu mengelola perubahan proyek secara terstruktur. Perintah `git status` digunakan untuk memeriksa keadaan repository, `git add` menyiapkan perubahan, `git commit` mencatat perubahan, dan `git push` mengirim commit ke GitHub.

Branch membantu memisahkan pekerjaan baru dari versi utama. Merge dan pull request digunakan untuk menggabungkan perubahan setelah diperiksa. Sementara itu, `git pull` membantu menyelaraskan pekerjaan lokal dengan perubahan dari remote. Penggunaan Git perlu dilakukan dengan hati-hati, khususnya ketika membatalkan perubahan atau menangani konflik.

**Catatan praktik:** bagian ini merupakan pembahasan berdasarkan materi. Lengkapi dengan hasil praktik sebenarnya, misalnya tangkapan layar keluaran `git --version`, `git status`, riwayat commit, dan repository GitHub setelah push berhasil.

## 7. Kesimpulan

Git dan GitHub saling melengkapi dalam pengelolaan proyek perangkat lunak. Git mencatat riwayat perubahan secara lokal, sedangkan GitHub menyediakan tempat penyimpanan daring dan sarana kolaborasi. Penguasaan perintah dasar seperti `status`, `add`, `commit`, `push`, `pull`, serta penggunaan branch dan pull request merupakan dasar penting untuk mengerjakan proyek secara mandiri maupun bersama tim.

## 8. Daftar Pustaka

1. NEO-X-School. *Petunjuk Git dan GitHub*. GitHub. https://github.com/NEO-X-School/notes/tree/main/petunjuk-git-github
2. Git. *Git Documentation*. https://git-scm.com/docs
3. GitHub. *GitHub Documentation*. https://docs.github.com/

---

**Catatan:** Laporan ini merupakan rangkuman pembelajaran umum mengenai Git dan GitHub berdasarkan tautan panduan yang diberikan. Sesuaikan identitas, tanggal, langkah praktik, dan bagian hasil dengan kegiatan yang benar-benar dilakukan.
