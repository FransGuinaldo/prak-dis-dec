# Laporan Praktikum Modul 7: Cloud Computing

| | |
|---|---|
| **Nama** | Francisco G I dc Corbafo |
| **NIM** | 255410003 |
| **Mata kuliah** | Sistem Terdistribusi dan Terdesentralisasi |
| **Modul** | Modul 7: Cloud Computing |
| **Dosen penyusun modul** | Bambang Purnomosidi D.P., Dr., S.E., Ak., S.Kom., MMSI.|
| **Prodi** | Informatika |
| **Tanggal praktikum** | 15 November 2026 |

---

## 0. Pengantar dan Tugas 0

Cloud Computing menggunakan pendekatan XaaS (*Everything as a Service*). Penyedia cloud menyediakan sumber daya komputasi, dan konsumen menggunakannya dalam bentuk layanan. Secara umum XaaS dibagi menjadi SaaS, PaaS, dan IaaS.

### 0.1 Contoh vendor

| Kategori | Vendor | Keterangan |
|---|---|---|
| SaaS | Google Workspace | Aplikasi siap pakai lewat browser |
| PaaS | Heroku | Platform menjalankan aplikasi tanpa mengurus server |
| IaaS | Amazon Web Services (EC2) | Infrastruktur virtual berupa VM, storage, dan jaringan |

### 0.2 Uraian layanan

| Kategori | Layanan yang disediakan vendor | Yang masih dikelola pengguna |
|---|---|---|
| SaaS (Google Workspace) | Gmail, Docs/Sheets/Slides, Drive, Meet. Vendor mengurus server, update, keamanan, dan backup. | Akun, isi dokumen, pengaturan akses |
| PaaS (Heroku) | Runtime bahasa pemrograman, deployment lewat `git push`, add-on database, scaling. Vendor mengurus OS, server, dan jaringan. | Kode aplikasi, konfigurasi, data |
| IaaS (AWS EC2) | Mesin virtual dengan CPU/RAM pilihan, penyimpanan (EBS, S3), jaringan virtual (VPC). Vendor mengurus pusat data, hardware, dan virtualisasi. | OS, patch keamanan, runtime, aplikasi, data |

### 0.3 Arsitektur XaaS

![Arsitektur XaaS: pembagian tanggung jawab IaaS, PaaS, SaaS]
<img width="1266" height="658" alt="image" src="https://github.com/user-attachments/assets/6d5972cd-c0f0-4fb2-807a-32ecb7a7771d" />


| Lapisan | IaaS | PaaS | SaaS |
|---|---|---|---|
| Aplikasi | Pengguna | Pengguna | Vendor |
| Data | Pengguna | Pengguna | Vendor |
| Runtime | Pengguna | Vendor | Vendor |
| Sistem operasi | Pengguna | Vendor | Vendor |
| Server dan jaringan | Vendor | Vendor | Vendor |

1. IaaS memberi kontrol tertinggi tetapi menuntut pengelolaan paling banyak. Cocok untuk administrator sistem.
2. PaaS menyembunyikan OS dan runtime sehingga pengembang fokus pada kode.
3. SaaS memberi kontrol paling sedikit tetapi paling mudah dipakai pengguna akhir.

### 0.4 Kaitan dengan Containerized App

Salah satu hal yang membentuk layanan-layanan tersebut adalah *Containerized App* (CA), yaitu aplikasi yang dijalankan oleh container (Docker, Podman, dan sejenisnya). Penyedia PaaS mengemas kode pengguna menjadi container lalu menjalankannya di infrastruktur mereka. Pada praktikum ini, aplikasi blog Flaskr dari tutorial Flask dijadikan CA.

---

## 1. Persiapan Lingkungan

Modul memakai Linux dengan sistem init dinit. Praktikum ini dikerjakan di **Windows dengan WSL2 (Ubuntu)** dan **Docker Desktop**.

| Hal | Di modul | Di praktikum ini |
|---|---|---|
| Sistem operasi | Linux (dinit) | Ubuntu di WSL2 |
| Mengaktifkan Docker | `sudo dinitctl start dockerd` | Docker Desktop dengan integrasi WSL, tanpa start manual |
| Mematikan dockerd sebelum Podman | `sudo dinitctl stop dockerd` | Tidak diperlukan, engine ada di Docker Desktop |
| Akses ke Docker | `sudo docker ...` | `sudo docker ...` |

### 1.1 Memeriksa WSL

```powershell
PS C:\Users\kikuc> wsl -l -v
  NAME              STATE           VERSION
* Ubuntu            Stopped         2
  docker-desktop    Stopped         2
PS C:\Users\kikuc> wsl -d ubuntu
```
<img width="607" height="157" alt="image" src="https://github.com/user-attachments/assets/84f0824d-a37c-4704-a4c1-e53dff1d8a21" />
<img width="1239" height="694" alt="image" src="https://github.com/user-attachments/assets/a2a7a505-ae5d-4525-89df-232526441b31" />



### 1.2 Memasang alat

```bash
cd ~
sudo apt update
sudo apt install -y git curl podman
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc
```
<img width="1527" height="689" alt="image" src="https://github.com/user-attachments/assets/4674c081-47b6-46c3-a0eb-32202254899e" />
<img width="1415" height="82" alt="image" src="https://github.com/user-attachments/assets/4683dcdc-88d3-42c5-8fdb-73dd546988fb" />



Pemasangan `podman` menarik 36 paket baru (termasuk `buildah`, `crun`, `netavark`, dan `conmon`) serta memperbarui `git` dan `curl`. Pengecekan versi:

```text
$ git --version
git version 2.53.0
$ uv --version
uv 0.13.8 (x86_64-unknown-linux-gnu)
$ podman --version
podman version 5.7.0
$ docker --version
Docker version 29.9.2, build a82e528
```
<img width="568" height="187" alt="image" src="https://github.com/user-attachments/assets/fca88eb8-5a89-4b7a-ab90-dd569213c14a" />


### 1.3 Kendala Docker dan penyelesaiannya

Pada percobaan pertama, perintah `docker` belum ditemukan di Ubuntu:

```text
$ docker --version
The command 'docker' could not be found in this WSL 2 distro.
We recommend to activate the WSL integration in Docker Desktop settings.
```

Setelah integrasi WSL diaktifkan di Docker Desktop (*Settings > Resources > WSL Integration > Ubuntu*), `docker --version` berhasil. Namun `docker ps` menampilkan error izin:

```text
permission denied while trying to connect to the docker API at unix:///var/run/docker.sock
```

<img width="1191" height="673" alt="image" src="https://github.com/user-attachments/assets/eaac9474-b519-4edc-80ed-b29035a4113f" />


Penyebabnya adalah user belum menjadi anggota grup `docker` (keluaran `groups` tidak memuat `docker`). Seluruh perintah Docker pada praktikum ini dijalankan dengan `sudo`, sama seperti pada modul:

```text
$ sudo docker ps
CONTAINER ID   IMAGE                      ...   STATUS                      NAMES
9e7da48684d6   postgres:18.3-alpine3.23   ...   Up 2 minutes (unhealthy)    prak-dis-dec-postgres_replica-1
```
<img width="1876" height="157" alt="image" src="https://github.com/user-attachments/assets/2588b52b-f784-49dd-b941-29229172554d" />


Pada saat memasang paket, sempat muncul pesan `Waiting for cache lock ... held by process (unattended-upgr)`. Pesan ini wajar, karena proses pembaruan otomatis Ubuntu sedang memakai `apt`, dan perintah selesai sendiri setelah proses tersebut berakhir.

---

## 2. Mengambil Source Code

Direktori kerja berada di `$HOME/src` pada filesystem Linux (bukan `/mnt/c`).

```bash
mkdir -p ~/src && cd ~/src
git clone https://github.com/pallets/flask
cp -R ~/src/flask/examples/tutorial flask-app
cd flask-app
ls -la
```

```text
Cloning into 'flask'...
Receiving objects: 100% (26408/26408), 12.08 MiB | 1.22 MiB/s, done.
Resolving deltas: 100% (17652/17652), done.

total 32
-rw-r--r-- 1 guinaldoinacio guinaldoinacio  119 Nov 15 10:46 .gitignore
-rw-r--r-- 1 guinaldoinacio guinaldoinacio 1475 Nov 15 10:46 LICENSE.txt
-rw-r--r-- 1 guinaldoinacio guinaldoinacio 1273 Nov 15 10:46 README.rst
drwxr-xr-x 4 guinaldoinacio guinaldoinacio 4096 Nov 15 10:46 flaskr
-rw-r--r-- 1 guinaldoinacio guinaldoinacio  873 Nov 15 10:46 pyproject.toml
drwxr-xr-x 2 guinaldoinacio guinaldoinacio 4096 Nov 15 10:46 tests
```

<img width="1048" height="624" alt="Screenshot 2026-11-15 115302" src="https://github.com/user-attachments/assets/2648d04a-bd0e-4b2b-9364-bdc64e14e147" />




---

## 3. Membuat Env Python 3.14.4

```bash
uv python pin 3.14.4
uv venv
source .venv/bin/activate
python --version
```

```text
Pinned `.python-version` to `3.14.4`
Using CPython 3.14.4 interpreter at: /usr/bin/python3.14
Creating virtual environment at: .venv
(flask-app) guinaldoinacio@corbafo:~/src/flask-app$ python --version
Python 3.14.4
```

`uv` memakai interpreter sistem `/usr/bin/python3.14`, sehingga tidak perlu mengunduh Python lagi. Prompt berubah menjadi `(flask-app)` yang menandakan environment aktif.

<img width="1027" height="271" alt="image" src="https://github.com/user-attachments/assets/e9349535-cc1a-41c9-be93-0420bfab3b8a" />


---

## 4. Install Paket dan Uji Lokal

```bash
uv pip install -e .
```

```text
Resolved 8 packages in 577ms
      Built flaskr @ file:///home/guinaldoinacio/src/flask-app
Prepared 8 packages in 687ms
Installed 8 packages in 5ms
 + blinker==1.9.0
 + click==8.5.1
 + flask==3.1.3
 + flaskr==1.0.0 (from file:///home/guinaldoinacio/src/flask-app)
 + itsdangerous==2.2.0
 + jinja2==3.1.6
 + markupsafe==3.0.4
 + werkzeug==3.1.9
```

<img width="1027" height="417" alt="image" src="https://github.com/user-attachments/assets/3b19fa98-da14-4927-9542-e24c958f8b52" />


Inisialisasi database dan jalankan aplikasi:

```bash
flask --app flaskr init-db
flask --app flaskr run
```

```text
Initialized the database.
 * Serving Flask app 'flaskr'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://127.0.0.1:5000
Press CTRL+C to quit
127.0.0.1 - - [15/Nov/2026 10:49:10] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [15/Nov/2026 10:49:10] "GET /static/style.css HTTP/1.1" 200 -
127.0.0.1 - - [15/Nov/2026 10:49:10] "GET /favicon.ico HTTP/1.1" 404 -
```

<img width="1531" height="203" alt="image" src="https://github.com/user-attachments/assets/56c96df0-5361-4d8f-b3d7-474cc837e4c0" />
<img width="1048" height="134" alt="image" src="https://github.com/user-attachments/assets/ed036cfe-8222-4042-8a89-553bfcb79e9f" />



Aplikasi diakses melalui `http://localhost:5000`.

<img width="1179" height="199" alt="Screenshot 2026-11-15 110848" src="https://github.com/user-attachments/assets/9ac3ae47-8c61-4632-94c5-c1e29ee84512" />


---

## 5. Membuat Aplikasi Menjadi CA dengan Docker

### 5.1 Dockerfile

```dockerfile
FROM python:3.14

RUN mkdir /app
WORKDIR /app
ADD . /app/
RUN pip install -e .
RUN flask --app flaskr init-db

EXPOSE 5000
CMD ["flask", "--app", "flaskr", "run", "--host=0.0.0.0", "--port=5000"]
```

<img width="939" height="332" alt="image" src="https://github.com/user-attachments/assets/bb0c1e6f-4d57-4c9e-94f2-dafbb0817807" />


| Baris | Fungsi |
|---|---|
| `FROM python:3.14` | Base image Python 3.14 |
| `RUN mkdir /app` | Membuat direktori aplikasi di dalam image |
| `WORKDIR /app` | Menjadikan `/app` direktori kerja |
| `ADD . /app/` | Menyalin source code ke image |
| `RUN pip install -e .` | Memasang aplikasi dan dependensinya |
| `RUN flask --app flaskr init-db` | Menginisialisasi database saat build |
| `EXPOSE 5000` | Mendokumentasikan port aplikasi |
| `CMD [...]` | Perintah saat container dijalankan. `--host=0.0.0.0` wajib agar bisa diakses dari luar container |

### 5.2 Build image

Karena engine Docker sudah berjalan di Docker Desktop, tidak ada langkah `start` daemon.

```bash
sudo docker build -f Dockerfile -t flaskr:1.0.0 .
```

```text
[+] Building 232.4s (11/11) FINISHED                                  docker:default
 => [internal] load metadata for docker.io/library/python:3.14                    4.9s
 => [1/6] FROM docker.io/library/python:3.14@sha256:a7097ae6...                 193.1s
 => [2/6] RUN mkdir /app                                                          0.4s
 => [3/6] WORKDIR /app                                                            0.1s
 => [4/6] ADD . /app/                                                             0.2s
 => [5/6] RUN pip install -e .                                                    7.1s
 => [6/6] RUN flask --app flaskr init-db                                          0.6s
 => exporting to image                                                           25.6s
 => => naming to docker.io/library/flaskr:1.0.0                                   0.0s
```

Build pertama memakan waktu 232,4 detik, sebagian besar (193,1 detik) untuk mengunduh base image `python:3.14`.

<img width="1867" height="687" alt="image" src="https://github.com/user-attachments/assets/22a01c6d-27d2-4e20-a64d-0682d70f2036" />
<img width="1856" height="626" alt="image" src="https://github.com/user-attachments/assets/ce656bd2-a68c-46d9-9707-d4bfa04fe0e9" />



```bash
sudo docker image ls
```

```text
IMAGE                       ID             DISK USAGE   CONTENT SIZE
apacheignite/ignite:3.1.0   c0f43bde84be        939MB          389MB
flaskr:1.0.0                90262b6e0e86       1.65GB          424MB
hello-world:latest          c567b231f656       25.9kB         9.49kB
postgres:18.3-alpine3.23    54451ecb8ab3        409MB          115MB
```

Image `flaskr:1.0.0` berhasil dibuat.

<img width="1879" height="159" alt="image" src="https://github.com/user-attachments/assets/09eca1fd-8e70-4333-b6bb-99fe4d1694cf" />


### 5.3 Menjalankan image

Port 5000 di container dipetakan ke port 5001 di localhost:

```bash
sudo docker run -p 5001:5000 flaskr:1.0.0
```

```text
 * Serving Flask app 'flaskr'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://172.17.0.2:5000
Press CTRL+C to quit
172.17.0.1 - - [15/Nov/2026 03:56:17] "GET / HTTP/1.1" 200 -
172.17.0.1 - - [15/Nov/2026 03:56:17] "GET /static/style.css HTTP/1.1" 200 -
172.17.0.1 - - [15/Nov/2026 03:56:17] "GET /favicon.ico HTTP/1.1" 404 -
172.17.0.1 - - [15/Nov/2026 03:56:29] "GET /auth/register HTTP/1.1" 200 -
172.17.0.1 - - [15/Nov/2026 03:56:44] "GET /auth/login HTTP/1.1" 200 -
```

<img width="1527" height="430" alt="image" src="https://github.com/user-attachments/assets/98a0909b-8c50-49d8-8e8a-af9bdd12e542" />
<img width="1179" height="199" alt="Screenshot 2026-11-15 110848" src="https://github.com/user-attachments/assets/27017569-c1eb-42c6-9a32-2110e9f4b659" />




Keterangan log:

- Baris `Running on all addresses (0.0.0.0)` membuktikan opsi `--host=0.0.0.0` bekerja.
- `172.17.0.2` adalah IP container pada jaringan bridge Docker, dan `172.17.0.1` adalah gateway bridge yang meneruskan permintaan dari browser. Ini menunjukkan port mapping `-p 5001:5000` bekerja.
- Kode `404` pada `favicon.ico` wajar karena aplikasi tidak menyediakan ikon.
- Jam pada log container (03:56) memakai UTC, sedangkan jam sistem WSL memakai waktu lokal (UTC+7). Karena itu terlihat berbeda dengan log di bagian 4 (10:49).

Setelah pengujian, container dihentikan dengan `Ctrl+C`:

```bash
sudo docker ps -a
```

```text
CONTAINER ID   IMAGE          COMMAND                  STATUS                      NAMES
4ed9803f292a   flaskr:1.0.0   "flask --app flaskr …"   Exited (0) 11 seconds ago   competent_engelbart
```

<img width="1872" height="355" alt="image" src="https://github.com/user-attachments/assets/8d1a632a-c204-494f-adae-2d07073a8553" />


Pada posisi ini, CA berhasil dijalankan dengan Docker.

---

## 6. Membuat Aplikasi Menjadi CA dengan Podman

Podman tidak memerlukan daemon, jadi tidak ada service yang perlu diaktifkan. Karena engine Docker berada di Docker Desktop (bukan di dalam Ubuntu), langkah mematikan `dockerd` pada modul tidak diperlukan. Container Docker sudah dihentikan agar port 5001 bebas.

Podman membaca `Dockerfile` yang sama dan dijalankan tanpa `sudo`:

```bash
podman build -f Dockerfile -t flaskr:1.0.0 .
```

```text
WARN[0000] "/" is not a shared mount, this could cause issues or missing mounts with rootless containers
STEP 1/8: FROM python:3.14
Resolved "python" as an alias (/etc/containers/registries.conf.d/shortnames.conf)
Trying to pull docker.io/library/python:3.14...
...
STEP 5/8: RUN pip install -e .
...
Successfully installed blinker-1.9.0 click-8.5.1 flask-3.1.3 flaskr-1.0.0 itsdangerous-2.2.0 jinja2-3.1.6 markupsafe-3.0.4 werkzeug-3.1.9
STEP 6/8: RUN flask --app flaskr init-db
Initialized the database.
STEP 7/8: EXPOSE 5000
STEP 8/8: CMD ["flask", "--app", "flaskr", "run", "--host=0.0.0.0", "--port=5000"]
COMMIT flaskr:1.0.0
Successfully tagged localhost/flaskr:1.0.0
```

Peringatan `"/" is not a shared mount` muncul karena Podman rootless di WSL2, dan tidak menghalangi proses build. Peringatan `Running pip as the 'root' user` di dalam build juga wajar karena pip berjalan sebagai root di dalam container.

![podman build](img/15-podman-build.png)

```bash
podman image ls
```

```text
REPOSITORY                TAG         IMAGE ID      CREATED       SIZE
localhost/flaskr          1.0.0       0d116eb4e384  1 second ago  1.17 GB
docker.io/library/python  3.14        78689ccd8331  9 days ago    1.14 GB
```

![podman image ls](img/16-podman-image-ls.png)

Menjalankan container:

```bash
podman run -p 5001:5000 flaskr:1.0.0
```

```text
 * Serving Flask app 'flaskr'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://172.22.185.180:5000
Press CTRL+C to quit
```

![podman run](img/17-podman-run.png)

![Flaskr di localhost:5001 (Podman)](img/18-browser-5001-podman.png)

Log akses pengujian register, login, dan logout:

```text
172.22.185.180 - - [15/Nov/2026 04:15:35] "GET / HTTP/1.1" 200 -
172.22.185.180 - - [15/Nov/2026 04:18:16] "GET /auth/register HTTP/1.1" 200 -
172.22.185.180 - - [15/Nov/2026 04:18:32] "POST /auth/register HTTP/1.1" 302 -
172.22.185.180 - - [15/Nov/2026 04:18:32] "GET /auth/login HTTP/1.1" 200 -
172.22.185.180 - - [15/Nov/2026 04:18:41] "POST /auth/login HTTP/1.1" 302 -
172.22.185.180 - - [15/Nov/2026 04:18:41] "GET / HTTP/1.1" 200 -
172.22.185.180 - - [15/Nov/2026 04:18:58] "GET /auth/logout HTTP/1.1" 302 -
172.22.185.180 - - [15/Nov/2026 04:18:58] "GET / HTTP/1.1" 200 -
```

![Log register, login, logout](img/19-podman-log-register-login.png)

`POST /auth/register` dan `POST /auth/login` dengan kode `302` menunjukkan akun berhasil dibuat dan login berhasil, sehingga database SQLite di dalam container dapat ditulis. Alamat sumber `172.22.185.180` adalah IP jaringan WSL2, berbeda dengan Docker karena Podman rootless memakai jaringan sendiri.

---

## 7. Analisis

### 7.1 Perbandingan Docker dan Podman

| Aspek | Docker | Podman |
|---|---|---|
| Arsitektur | Client-server dengan daemon (`dockerd`) | *Daemonless*, tiap perintah berjalan sebagai proses biasa |
| Hak akses | Pada praktikum ini memakai `sudo` | *Rootless*, tanpa `sudo` |
| Format build | `Dockerfile` | `Dockerfile` yang sama |
| Jaringan container | Bridge Docker (`172.17.0.x`) | Jaringan rootless (`172.22.185.180` pada WSL2) |
| Nama image | `flaskr:1.0.0` | `localhost/flaskr:1.0.0` |
| Ukuran image | 1,65 GB (disk usage), 424 MB (content size) | 1,17 GB |
| Image ID | `90262b6e0e86` | `0d116eb4e384` |

Perbedaan ukuran terjadi karena cara masing-masing alat menghitung dan menyimpan layer, bukan karena isi aplikasinya berbeda.

### 7.2 Catatan teknis

1. `--host=0.0.0.0` pada `CMD` wajib agar aplikasi dapat diakses dari luar container.
2. `init-db` dijalankan saat *build*, sehingga database tertanam di image. Data akan hilang jika container dibuat ulang karena tidak memakai *volume*.
3. Ukuran image besar karena memakai `python:3.14` penuh. Varian `-slim` dapat memperkecilnya.
4. Peringatan *development server* menandakan bahwa untuk produksi sebaiknya memakai server WSGI seperti Gunicorn.
5. Versi paket pada praktikum ini (misalnya `click 8.5.1` dan `werkzeug 3.1.9`) lebih baru daripada yang tertera di modul, karena `pyproject.toml` hanya menetapkan batas minimum versi.

### 7.3 Kaitan dengan XaaS

Ubuntu pada WSL2 berperan seperti lapisan infrastruktur (IaaS). Docker dan Podman menyediakan isolasi dan runtime, yang merupakan inti dari PaaS. Flaskr yang diakses lewat browser merupakan contoh aplikasi layanan (SaaS).

---

## 8. Kesimpulan

1. XaaS terdiri atas SaaS, PaaS, dan IaaS yang dibedakan oleh pembagian tanggung jawab antara vendor dan pengguna.
2. Aplikasi Flaskr berhasil dikemas menjadi *Containerized App* menggunakan satu `Dockerfile`.
3. Image berhasil dibangun dan dijalankan dengan Docker maupun Podman, dengan port mapping `5001:5000`.
4. Docker dan Podman sama-sama membaca `Dockerfile`, tetapi berbeda pada arsitektur (daemon vs daemonless) dan model hak akses.
5. Praktikum di Windows dapat dikerjakan dengan WSL2 dan Docker Desktop dengan penyesuaian terhadap perintah pada modul.

---

## Lampiran A. Daftar Screenshot

| No | Nama file | Isi yang diambil |
|---|---|---|
| 00 | `00-arsitektur-xaas.png` | Diagram arsitektur XaaS |
| 01 | `01-wsl-list.png` | `wsl -l -v` di PowerShell |
| 02 | `02-versi-alat.png` | `git`, `uv`, `podman`, `docker --version` |
| 03 | `03-docker-permission.png` | Error awal Docker dan `sudo docker ps` |
| 04 | `04-clone-dan-ls.png` | `git clone`, `cp -R`, `ls -la` |
| 05 | `05-env-python.png` | `uv venv`, aktivasi, `python --version` |
| 06 | `06-uv-pip-install.png` | `uv pip install -e .` |
| 07 | `07-init-db-run.png` | `flask init-db` dan `flask run` |
| 08 | `08-browser-5000.png` | Browser di `localhost:5000` |
| 09 | `09-dockerfile.png` | `cat Dockerfile` |
| 10 | `10-docker-build.png` | `sudo docker build` selesai |
| 11 | `11-docker-image-ls.png` | `sudo docker image ls` |
| 12 | `12-docker-run.png` | `sudo docker run -p 5001:5000 flaskr:1.0.0` |
| 13 | `13-browser-5001-docker.png` | Browser di `localhost:5001` (Docker) |
| 14 | `14-docker-ps-a.png` | `sudo docker ps -a` |
| 15 | `15-podman-build.png` | `podman build` selesai |
| 16 | `16-podman-image-ls.png` | `podman image ls` |
| 17 | `17-podman-run.png` | `podman run -p 5001:5000 flaskr:1.0.0` |
| 18 | `18-browser-5001-podman.png` | Browser di `localhost:5001` (Podman) |
| 19 | `19-podman-log-register-login.png` | Log `POST /auth/register` dan `POST /auth/login` |
