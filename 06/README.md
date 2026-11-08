# Laporan Praktikum Modul 6
## Distributed File System – HDFS (Apache Hadoop 3.5.0)

| | |
|---|---|
| **Mata Praktikum** | Sistem Terdistribusi dan Terdesentralisasi |
| **Nama** | Francisco G I dc Corbafo |
| **NIM** | 255410003 |
| **Tanggal Praktikum** | 6 November 2026 |

---

## 1. Tujuan

1. Memasang dan mengonfigurasi Apache Hadoop 3.5.0 dalam mode *pseudo-distributed* (satu node).
2. Menjalankan daemon HDFS (NameNode, DataNode, SecondaryNameNode).
3. Melakukan operasi dasar pada HDFS: membuat direktori dan menyalin file dari sistem lokal.

## 2. Dasar Teori Singkat

**HDFS (Hadoop Distributed File System)** adalah sistem berkas terdistribusi yang dirancang untuk menyimpan file berukuran besar secara andal di banyak mesin. Komponen utamanya:

| Komponen | Peran |
|---|---|
| **NameNode** | Menyimpan metadata (struktur direktori, lokasi block) dan mengatur akses ke file. |
| **DataNode** | Menyimpan block data sebenarnya dan melayani permintaan baca/tulis. |
| **SecondaryNameNode** | Membantu penggabungan (*checkpoint*) metadata NameNode; bukan pengganti NameNode. |

File dipecah menjadi **block** dan direplikasi sesuai `dfs.replication`. Pada praktikum ini nilainya `1` karena hanya ada satu DataNode.

## 3. Lingkungan Praktikum

| Komponen | Keterangan |
|---|---|
| Sistem operasi host | Windows (Java 25 terpasang, tidak dipakai untuk Hadoop) |
| Lingkungan Linux | WSL2, Ubuntu 26.04.1 LTS (kernel 6.18.54.1-microsoft-standard-WSL2) |
| JDK | OpenJDK 17.0.20.1 |
| Apache Hadoop | 3.5.0 (binary x86_64) |
| pdsh | 2.35 |
| OpenSSH | 10.2p1 |
| Username Linux | `guinaldoinacio` |

**Alasan memakai WSL2:** modul mensyaratkan JDK 17 atau 21 dan memakai skrip bash (`start-dfs.sh`), `pdsh`, dan `sshd`. Komputer host memiliki Java 25 dan berjalan di Windows. Dengan WSL2, JDK 17 dipasang terpisah di dalam Ubuntu sehingga Java 25 di Windows tidak terganggu dan seluruh langkah modul dapat diikuti.

## 4. Langkah Pengerjaan

### 4.1 Pemasangan WSL2 dan Ubuntu

Di PowerShell (Administrator):

```powershell
wsl --install -d Ubuntu
```
<img width="1197" height="738" alt="Screenshot 2026-11-08 060549" src="https://github.com/user-attachments/assets/18c2cacb-6c24-40c9-8678-33d9e80a4efa" />


### 4.2 Pemasangan paket, JDK 17, dan layanan SSH

```bash
sudo apt update
sudo apt install -y openjdk-17-jdk openssh-server pdsh wget
java -version
```

```
openjdk version "17.0.20.1" 2026-08-18
OpenJDK Runtime Environment (build 17.0.20.1+1-1-26.04-Ubuntu)
OpenJDK 64-Bit Server VM (build 17.0.20.1+1-1-26.04-Ubuntu, mixed mode, sharing)
```
<img width="1048" height="124" alt="image" src="https://github.com/user-attachments/assets/c6982b0f-2555-44e1-b6b2-764eb8d8b178" />

`pdsh` diarahkan memakai SSH, lalu layanan SSH dijalankan dan diperiksa:

```bash
echo 'export PDSH_RCMD_TYPE=ssh' >> ~/.bashrc
source ~/.bashrc
sudo service ssh start
sudo service ssh status
```

Status layanan: `active (running)`, mendengarkan pada port 22.
<img width="826" height="376" alt="Screenshot 2026-11-08 124608" src="https://github.com/user-attachments/assets/afdc83c2-d882-451a-9a4c-4b414d80e216" />


### 4.3 Konfigurasi SSH tanpa password ke localhost

Skrip `start-dfs.sh` memakai `pdsh` dan SSH untuk memulai daemon, sehingga login ke `localhost` harus tanpa password.

```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 0600 ~/.ssh/authorized_keys
ssh localhost
```
<img width="806" height="673" alt="Screenshot 2026-11-08 064259" src="https://github.com/user-attachments/assets/a6f00627-486f-481f-b963-aa1cc5e3a2e6" />


### 4.4 Unduh dan ekstraksi Hadoop

```bash
mkdir -p ~/software/storage && cd ~/software/storage
wget -4 -c --timeout=30 --tries=10 \
  https://archive.apache.org/dist/hadoop/common/hadoop-3.5.0/hadoop-3.5.0.tar.gz
tar -xzf hadoop-3.5.0.tar.gz
ln -s hadoop-3.5.0 hadoop
ls -l
```

File terunduh lengkap: 581.280.703 byte (554 MB), sama dengan ukuran yang dilaporkan server.

```
lrwxrwxrwx  1 franguinaldo franguinaldo        12 Nov  6 16:46 hadoop -> hadoop-3.5.0
drwxr-xr-x 11 franguinaldo franguinaldo      4096 Mar 25  2026 hadoop-3.5.0
-rw-r--r--  1 franguinaldo franguinaldo 581280703 Apr  2  2026 hadoop-3.5.0.tar.gz
```
<img width="1499" height="472" alt="Screenshot 2026-11-08 064558" src="https://github.com/user-attachments/assets/3ef7906b-686e-4ae4-9a9b-af8c4ac5e5f8" />
<img width="1527" height="497" alt="Screenshot 2026-11-08 064626" src="https://github.com/user-attachments/assets/aa8629f6-9004-4f9f-a154-b33c76c69768" />


### 4.5 Variabel lingkungan

File `~/env/bash/storage/hadoop.sh` berisi:

```bash
export HADOOP_HOME="$HOME/software/storage/hadoop"
export PATH="$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH"
```

File ini dimuat otomatis setiap terminal dibuka dengan menambahkan baris `source ~/env/bash/storage/hadoop.sh` ke `~/.bashrc`.

### 4.6 Konfigurasi Hadoop

**`JAVA_HOME`** pada `$HADOOP_HOME/etc/hadoop/hadoop-env.sh`:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
```

**`hdfs-site.xml`**: replikasi 1 dan lokasi data permanen (bukan `/tmp`):

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///home/franguinaldo/hadoop-data/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///home/franguinaldo/hadoop-data/datanode</value>
    </property>
</configuration>
```

**`core-site.xml`**:

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.proxyuser.franguinaldo.groups</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.franguinaldo.hosts</name>
        <value>*</value>
    </property>
</configuration>
```

> `mapred-site.xml` dan `yarn-site.xml` pada modul tidak dikonfigurasi karena praktikum ini hanya menjalankan HDFS, bukan YARN.

Verifikasi konfigurasi:

```bash
hadoop version
hdfs getconf -confKey fs.defaultFS
hdfs getconf -confKey dfs.namenode.name.dir
```

```
Hadoop 3.5.0
hdfs://localhost:9000
file:///home/franguinaldo/hadoop-data/namenode
```

### 4.7 Format NameNode dan menjalankan HDFS

Format NameNode dilakukan **satu kali saja**, lalu daemon dijalankan dan diperiksa:

```bash
hdfs namenode -format
start-dfs.sh
jps
```

```
7840 SecondaryNameNode
7460 NameNode
7990 Jps
7591 DataNode
```

Ketiga daemon HDFS berjalan. Pemantauan dilakukan melalui web UI di `http://localhost:9870`.

> **[Sisipkan tangkapan layar: halaman Overview web UI dengan status *active*]**

### 4.8 Mengerjakan tugas

**Tugas 1: membuat direktori `/user/<nama-user>/datasets` di HDFS**

```bash
hdfs dfs -mkdir -p /user/franguinaldo/datasets
```

**Tugas 2: mengunduh 3 file CSV dan menyalinnya ke direktori tersebut**

Tiga file CSV diunduh dari `people.sc.fsu.edu`, lalu disalin ke HDFS dan diperiksa:

```bash
mkdir -p ~/csv && cd ~/csv
wget -4 -c --tries=5 https://people.sc.fsu.edu/~jburkardt/data/csv/airtravel.csv
wget -4 -c --tries=5 https://people.sc.fsu.edu/~jburkardt/data/csv/addresses.csv
wget -4 -c --tries=5 https://people.sc.fsu.edu/~jburkardt/data/csv/biostats.csv
hdfs dfs -put ~/csv/*.csv /user/franguinaldo/datasets/
hdfs dfs -ls /user/franguinaldo/datasets
hdfs dfs -cat /user/franguinaldo/datasets/airtravel.csv
```

Isi direktori `datasets` di HDFS:

```
Found 3 items
-rw-r--r--   1 franguinaldo supergroup        328 2026-11-06 16:52 /user/franguinaldo/datasets/addresses.csv
-rw-r--r--   1 franguinaldo supergroup        321 2026-11-06 16:52 /user/franguinaldo/datasets/airtravel.csv
-rw-r--r--   1 franguinaldo supergroup        849 2026-11-06 16:52 /user/franguinaldo/datasets/biostats.csv
```

Isi `airtravel.csv` yang dibaca dari HDFS:

```
"Month", "1958", "1959", "1960"
"JAN",  340,  360,  417
"FEB",  318,  342,  391
"MAR",  362,  406,  419
"APR",  348,  396,  461
"MAY",  363,  420,  472
"JUN",  435,  472,  535
"JUL",  491,  548,  622
"AUG",  505,  559,  606
"SEP",  404,  463,  508
"OCT",  359,  407,  461
"NOV",  310,  362,  390
"DEC",  337,  405,  432
```

> **[Sisipkan tangkapan layar: output daftar file di HDFS]**
> **[Sisipkan tangkapan layar: web UI *Utilities → Browse the file system* pada `/user/franguinaldo/datasets`]**

## 5. Hasil

| Item | Status |
|---|---|
| JDK 17 terpasang | Berhasil |
| SSH tanpa password ke localhost | Berhasil |
| Hadoop 3.5.0 terpasang dan terkonfigurasi | Berhasil |
| NameNode diformat | Berhasil |
| NameNode, DataNode, SecondaryNameNode berjalan | Berhasil |
| Tugas 1: direktori `datasets` | Berhasil |
| Tugas 2: 3 file CSV tersalin ke HDFS (328, 321, dan 849 byte) | Berhasil |

## 6. Pembahasan

**Mode pseudo-distributed.** Seluruh daemon HDFS berjalan di satu mesin. Nilai `dfs.replication=1` dipilih karena hanya ada satu DataNode; pada klaster produksi nilainya umumnya 3 agar data tetap aman bila satu node gagal. Hal ini terlihat pada kolom replikasi bernilai `1` di daftar file HDFS.

**Peran SSH dan pdsh.** Skrip `start-dfs.sh` memulai NameNode, DataNode, dan SecondaryNameNode melalui `pdsh` yang memakai SSH. Karena itu SSH tanpa password ke `localhost` wajib dikonfigurasi. Ketika kunci belum didaftarkan, login masih meminta password; setelah `authorized_keys` terisi, login berjalan tanpa password.

**Lokasi data permanen.** Bila `dfs.namenode.name.dir` tidak diatur, Hadoop menyimpan data di `/tmp/hadoop-<user>`, yang dapat terhapus saat sistem dihidupkan ulang. Pada praktikum ini lokasi diarahkan ke `~/hadoop-data` agar data bertahan. Perintah format NameNode tidak boleh diulang setelah HDFS berisi data, karena seluruh isinya akan terhapus.

**Perbedaan port.** Port `9000` adalah alamat RPC HDFS (`fs.defaultFS`) yang dipakai klien seperti `hdfs dfs`, sedangkan port `9870` adalah web UI NameNode untuk pemantauan.

**Operasi HDFS.** Perintah `-mkdir`, `-put`, `-ls`, dan `-cat` bekerja mirip perintah shell biasa, tetapi beroperasi pada ruang nama HDFS. File kecil (di bawah ukuran block) tetap disimpan sebagai satu block. HDFS dioptimalkan untuk file besar, sehingga banyak file kecil bukan pola penggunaan yang ideal.

**Menjalankan kembali HDFS.** Layanan SSH tidak aktif otomatis setiap WSL dibuka kembali, sehingga harus dinyalakan lebih dulu sebelum daemon HDFS dijalankan. Data tetap tersimpan di `~/hadoop-data`.

## 7. Kendala dan Penyelesaian

| No | Kendala | Penyebab | Penyelesaian |
|---|---|---|---|
| 1 | Java host versi 25, sedangkan modul mensyaratkan 17 atau 21 | Versi JDK Windows tidak sesuai | Memasang JDK 17 di dalam WSL2 Ubuntu secara terpisah dan mengarahkan `JAVA_HOME` ke sana |
| 2 | Perintah `wsl -l -v` gagal di terminal Ubuntu | Perintah `wsl` hanya tersedia di Windows | Perintah dijalankan di PowerShell, bukan di dalam Ubuntu |
| 3 | Login ke localhost meminta password | Kunci publik belum masuk `authorized_keys` | Membuat kunci, mendaftarkannya ke `authorized_keys`, dan mengatur izin file |
| 4 | Unduhan dari `dlcdn.apache.org` timeout (IPv4 timeout, IPv6 tidak terjangkau) | Koneksi ke server Apache tidak stabil dan IPv6 tidak tersedia di WSL | Memakai `archive.apache.org` dengan opsi `-4 -c --timeout --tries` sehingga unduhan berhasil |
| 5 | Muncul `JAVA_HOME is not set` saat menjalankan Hadoop | `JAVA_HOME` belum diatur | Menambahkan `JAVA_HOME` pada `hadoop-env.sh` |

## 8. Kesimpulan

Apache Hadoop 3.5.0 berhasil dipasang dan dikonfigurasi pada Ubuntu 26.04 di atas WSL2 dengan JDK 17. Ketiga daemon HDFS (NameNode, DataNode, SecondaryNameNode) berjalan dengan baik. Direktori `/user/franguinaldo/datasets` berhasil dibuat dan tiga file CSV (`addresses.csv`, `airtravel.csv`, `biostats.csv`) berhasil disalin serta terverifikasi isinya. Seluruh tugas pada Modul 6 telah selesai.

## 9. Referensi

- Modul 6 – Distributed File System HDFS, Praktikum Sistem Terdistribusi dan Terdesentralisasi, Universitas Teknologi Digital Indonesia (Dr. Bambang Purnomosidi D. P.)
- Apache Hadoop Releases: https://hadoop.apache.org/releases.html
- HDFS FileSystem Shell: https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html
