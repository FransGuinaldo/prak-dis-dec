# LAPORAN PRAKTIKUM
## SISTEM TERDISTRIBUSI DAN TERDESENTRALISASI

### Identitas Praktikum

| Keterangan | Identitas |
|---|---|
| Nama | Francisco G. I. Corbafo |
| NIM | 255410003 |
| Program Studi | Informatika |
| Mata Kuliah | Sistem Terdistribusi dan Terdesentralisasi |
| Modul | [07 Cloud Computing] |
| Dosen Penyusun | [] |

---

## A. Judul Praktikum

**Instalasi Lingkungan Pengembangan Linux, Konfigurasi WSL 2, dan Deployment Aplikasi Flask Menggunakan Docker.**

## B. Tujuan Praktikum

1. Memahami penggunaan WSL 2 sebagai lingkungan Linux pada Windows.
2. Melakukan instalasi dan konfigurasi Git, curl, Podman, dan uv.
3. Memahami penggunaan Python dan virtual environment untuk menjalankan aplikasi Flask.
4. Mempelajari proses pengambilan source code menggunakan Git Clone.
5. Membuat Dockerfile untuk mengemas aplikasi Flask ke dalam Docker Image.
6. Menjalankan aplikasi Flask menggunakan container dan mengaksesnya melalui browser.

## C. Alat dan Bahan

1. Laptop atau komputer dengan sistem operasi Windows.
2. Windows Subsystem for Linux (WSL 2).
3. Ubuntu Linux.
4. Git.
5. curl.
6. Podman.
7. uv.
8. Python 3.14.4.
9. Docker Desktop.
10. Browser untuk mengakses aplikasi Flask.

## D. Langkah-Langkah Praktikum

### 1. Memeriksa WSL 2

Langkah pertama dilakukan dengan membuka PowerShell dan memeriksa distribusi Linux yang tersedia menggunakan perintah:

```powershell
wsl -l -v
```

Hasil menunjukkan bahwa Ubuntu dan docker-desktop sudah terpasang dengan WSL versi 2.

Selanjutnya masuk ke Ubuntu menggunakan:

```powershell
wsl -d Ubuntu
```

Setelah masuk, direktori kerja diarahkan ke home menggunakan:

```bash
cd ~
pwd
```

Hasil menunjukkan direktori kerja berada di `/home/guinaldoinacio`.

### 2. Memperbarui dan Menginstal Paket

Melakukan pembaruan daftar paket Ubuntu menggunakan:

```bash
sudo apt update
```

Kemudian menginstal Git, curl, dan Podman:

```bash
sudo apt install -y git curl podman
```

Proses instalasi berhasil dilakukan dan seluruh paket yang dibutuhkan sudah terpasang.

### 3. Menginstal uv

Menginstal uv menggunakan perintah:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Kemudian memperbarui konfigurasi terminal:

```bash
source ~/.bashrc
export PATH="$HOME/.local/bin:$PATH"
```

Memeriksa versi aplikasi:

```bash
git --version
uv --version
podman --version
```

Hasil pemeriksaan menunjukkan Git versi 2.53.0, uv versi 0.13.8, dan Podman versi 5.7.0.

### 4. Memeriksa Docker

Melakukan pemeriksaan Docker menggunakan:

```bash
docker --version
docker ps
```

Pada awalnya Docker belum dapat diakses melalui Ubuntu. Setelah integrasi Docker Desktop dengan WSL tersedia, Docker berhasil dijalankan.

Namun, terdapat kendala permission denied saat mengakses Docker. Untuk sementara, perintah dijalankan menggunakan:

```bash
sudo docker ps
```

Hasil menunjukkan container PostgreSQL sudah berjalan.

### 5. Mengunduh Source Code Flask

Membuat direktori untuk menyimpan source code:

```bash
mkdir -p ~/src && cd ~/src
```

Mengunduh repository Flask dari GitHub:

```bash
git clone https://github.com/pallets/flask
```

Kemudian menyalin folder tutorial:

```bash
cp -R ~/src/flask/examples/tutorial flask-app
```

Masuk ke direktori aplikasi:

```bash
cd flask-app
```

Memeriksa isi folder menggunakan:

```bash
ls -la
```

Hasil menunjukkan terdapat folder `flaskr`, `tests`, serta file `README.rst`, `LICENSE.txt`, dan `pyproject.toml`.

### 6. Mengatur Versi Python dan Virtual Environment

Menentukan versi Python menggunakan uv:

```bash
uv python pin 3.14.4
```

Membuat virtual environment:

```bash
uv venv
```

Mengaktifkan virtual environment:

```bash
source .venv/bin/activate
```

Memeriksa versi Python:

```bash
python --version
```

Hasil menunjukkan Python 3.14.4 berhasil digunakan.

### 7. Menginstal Aplikasi Flask

Menginstal aplikasi Flask dalam mode editable menggunakan:

```bash
uv pip install -e .
```

Setelah instalasi selesai, database aplikasi diinisialisasi dengan perintah:

```bash
flask --app flaskr init-db
```

Hasilnya menunjukkan pesan `Initialized the database.` yang berarti database berhasil dibuat.

### 8. Menjalankan Aplikasi Flask

Menjalankan aplikasi menggunakan:

```bash
flask --app flaskr run
```

Aplikasi berhasil dijalankan pada alamat:

```text
http://127.0.0.1:5000
```

Selanjutnya alamat tersebut dibuka melalui browser. Hasil menunjukkan halaman aplikasi Flask berhasil ditampilkan.

### 9. Membuat Dockerfile

Membuat file Dockerfile pada direktori aplikasi menggunakan:

```bash
cat > Dockerfile << 'EOF'
FROM python:3.14

RUN mkdir /app
WORKDIR /app
ADD . /app/
RUN pip install -e .
RUN flask --app flaskr init-db

EXPOSE 5000
CMD ["flask", "--app", "flaskr", "run", "--host=0.0.0.0", "--port=5000"]
EOF
```

Kemudian memeriksa isi Dockerfile:

```bash
cat Dockerfile
```

Dockerfile tersebut berisi instruksi untuk menggunakan image Python 3.14, menyalin aplikasi ke dalam container, menginstal dependensi, menginisialisasi database, membuka port 5000, dan menjalankan aplikasi Flask.

### 10. Membuat Docker Image dan Menjalankan Container

Membuat Docker Image menggunakan:

```bash
sudo docker build -f Dockerfile -t flaskr:1.0.0 .
```

Setelah proses build selesai, memeriksa daftar image:

```bash
sudo docker image ls
```

Hasil menunjukkan image `flaskr:1.0.0` berhasil dibuat.

Selanjutnya menjalankan container:

```bash
sudo docker run -p 5001:5000 flaskr:1.0.0
```

Aplikasi kemudian dapat diakses melalui browser menggunakan alamat:

```text
http://localhost:5001
```

Hasil menunjukkan aplikasi Flask berhasil berjalan di dalam container Docker.

---

## E. Hasil Praktikum

Berdasarkan kegiatan praktikum, beberapa hasil yang diperoleh adalah:

1. WSL 2 dan Ubuntu berhasil digunakan sebagai lingkungan Linux pada Windows.
2. Git, curl, Podman, dan uv berhasil diinstal dan dapat digunakan.
3. Repository Flask berhasil diunduh dari GitHub.
4. Virtual environment menggunakan Python 3.14.4 berhasil dibuat.
5. Aplikasi Flask berhasil diinstal dan dijalankan secara lokal.
6. Database aplikasi berhasil diinisialisasi.
7. Dockerfile berhasil dibuat sesuai kebutuhan aplikasi.
8. Docker Image `flaskr:1.0.0` berhasil dibangun.
9. Container berhasil dijalankan menggunakan port 5001.
10. Aplikasi Flask berhasil diakses melalui browser menggunakan Docker.

## F. Kendala dan Penyelesaian

Selama praktikum terdapat beberapa kendala yang ditemukan.

1. Perintah `git -version` tidak dapat digunakan karena penulisan perintah kurang tepat. Penyelesaiannya menggunakan `git --version`.
2. Perintah Docker pada awalnya tidak ditemukan di Ubuntu. Hal ini berkaitan dengan integrasi Docker Desktop dan WSL 2.
3. Terjadi masalah permission denied ketika menjalankan Docker. Untuk mengatasinya, perintah Docker sementara dijalankan menggunakan `sudo`.
4. Saat menjalankan `apt install`, terdapat pesan bahwa proses instalasi sedang menggunakan package lock. Hal ini disebabkan proses `unattended-upgrades` yang sedang berjalan sehingga perlu menunggu proses tersebut selesai.

## G. Kesimpulan

Praktikum ini memberikan pemahaman mengenai penggunaan WSL 2 sebagai lingkungan Linux pada Windows serta proses pengembangan aplikasi menggunakan Python dan Flask. Git digunakan untuk mengambil source code dari GitHub, sedangkan uv membantu mengatur versi Python dan virtual environment. Selain itu, Docker digunakan untuk mengemas aplikasi Flask ke dalam Docker Image dan menjalankannya melalui container. Dari kegiatan ini, dapat dipahami bahwa containerisasi membantu aplikasi berjalan dalam lingkungan yang lebih terstruktur dan mudah dipindahkan ke sistem lain.
