# Laporan Praktikum Modul 4
## Konsistensi dan Replikasi pada Sistem Terdistribusi

* **Mata Kuliah:** Sistem Terdistribusi dan Terdesentralisasi
* **Nama Mahasiswa:** Francisco G. I. dc Corbafo
* **NIM:** 255410003
* **Kelas:** IF-1
* **Program Studi:** Informatika


---

## 4.1 Streaming Replication Menggunakan PostgreSQL

Materi ini membahas konfigurasi *streaming replication* pada **PostgreSQL 18** (menggunakan image `postgres:18.3-alpine3.23`). Secara prinsip, konfigurasinya serupa dengan versi sebelumnya, namun direktori data berada di `/var/lib/postgresql/18/docker`.

### 4.1.1 Prasyarat
1. Instalasi **Docker** dan **Docker Compose** telah dilakukan.
2. Daemon `dockerd` diaktifkan (pada sistem dengan `dinit` sebagai init system):
   ```bash
   sudo dinitctl start dockerd
   ```
3. Petunjuk praktikum menggunakan sistem operasi Linux.

### 4.1.2 Struktur Direktori dan File
Direktori kerja `streaming-replication/` berisi tiga file utama:
```text
streaming-replication/
├── 00_init.sql
├── docker-compose.yaml
└── env.sh
```

#### 1. `00_init.sql`
File perintah SQL yang dijalankan saat *primary server* diinisialisasi untuk membuat user replikasi dan slot replikasi:
```sql
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'replicator_password';
SELECT pg_create_physical_replication_slot('replication_slot');
```

#### 2. `docker-compose.yaml`
Konfigurasi multi-container untuk `postgres_primary` dan `postgres_replica`:
```yaml
x-postgres-common:
  &postgres-common
  image: postgres:18.3-alpine3.23
  user: postgres
  restart: always
  healthcheck:
    test: 'pg_isready -U zuser -d zdb'
    interval: 10s
    timeout: 5s
    retries: 5

services:
  postgres_primary:
    <<: *postgres-common
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: zuser
      POSTGRES_DB: zdb
      POSTGRES_PASSWORD: zpass
      POSTGRES_HOST_AUTH_METHOD: "scram-sha-256\nhost replication replicator 0.0.0.0/0 md5"
      POSTGRES_INITDB_ARGS: "--auth-host-scram-sha-256"
    command: |
      postgres
      -c wal_level=replica
      -c hot_standby=on
      -c max_wal_senders=10
      -c max_replication_slots=10
      -c hot_standby_feedback=on
    volumes:
      - ./00_init.sql:/docker-entrypoint-initdb.d/00_init.sql

  postgres_replica:
    <<: *postgres-common
    ports:
      - "5433:5432"
    environment:
      PGUSER: replicator
      PGPASSWORD: replicator_password
      PGDATA: /var/lib/postgresql/18/docker
    command: |
      bash -c "
      until pg_basebackup --pgdata=/var/lib/postgresql/18/docker -R --slot=replication_slot --host=postgres_primary --port=5432 -X stream; do
        echo 'pg_basebackup failed. Retrying in 5 seconds...'
        sleep 5
      done
      echo 'Backup done, starting replica...'
      chmod 0700 /var/lib/postgresql/18/docker
      postgres
      "
    depends_on:
      postgres_primary
```

#### 3. `env.sh` (Shortcut Aliases)
File definisi fungsi bantu/alias bash:
```bash
alias dcu="sudo docker-compose up -d"
alias dcd="sudo docker-compose down"
alias dps="sudo docker ps"
alias der="sudo docker exec -it <nama-direktori>-postgres_replica-1 bash"
alias dep="sudo docker exec -it <nama-direktori>-postgres_primary-1 bash"
alias dlr="sudo docker logs <nama-direktori>-postgres_replica-1"
alias dlp="sudo docker logs <nama-direktori>-postgres_primary-1"
```

---

### 4.1.3 Menjalankan Docker Compose & Verifikasi Container

Menjalankan environment menggunakan perintah yang di-source dari `env.sh`:
```bash
$ source env.sh
$ dcu
```

> **Visualisasi Log Proses Container (Replica & Primary Startup):**
> ![Log Proses PostgreSQL Replication](https://via.placeholder.com/800x350.png?text=Log+pg_basebackup+and+PostgreSQL+Recovery+State)
> *Keterangan Gambar:* Log menunjukkan proses `pg_basebackup` berhasil melakukan sinkronisasi dari `postgres_primary`, diikuti oleh status `database system is ready to accept read-only connections` pada standby server.

Pemeriksaan status container (`dps` / `dep`):
```bash
CONTAINER ID   IMAGE                         COMMAND                  CREATED         STATUS                   PORTS                                                                  NAMES
225df898488e   postgres:18.3-alpine3.23      "docker-entrypoint.c…"   4 minutes ago   Up 4 minutes (healthy)   0.0.0.0:5433->5432/tcp, :::5433->5432/tcp                              src-postgres_replica-1
492a2b3bc3db   postgres:18.3-alpine3.23      "docker-entrypoint.c…"   4 minutes ago   Up 4 minutes (healthy)   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp                              src-postgres_primary-1
```

---

### 4.1.4 Pengujian Replikasi Data

#### 1. Diagnostik pada Server Utama (Primary)
Masuk ke terminal primary (`dep`) lalu jalankan `psql`:
```sql
zdb=# \x
zdb=# SELECT * FROM pg_stat_replication;
```
> ![Statistik Replikasi Primary](https://via.placeholder.com/750x300.png?text=pg_stat_replication+Output)
> *Keterangan Gambar:* Menunjukkan status replikator aktif dengan protokol `streaming` dan posisi LSN yang tersinkronisasi.

Pemeriksaan slot replikasi:
```sql
zdb=# SELECT * FROM pg_replication_slots;
```
> ![Replication Slots](https://via.placeholder.com/750x300.png?text=pg_replication_slots+Output)

#### 2. Diagnostik pada Server Replica
Masuk ke container replica (`der`) dan periksa status recovery:
```sql
zdb=# SELECT pg_is_in_recovery();
```
> ![Status Recovery Replica](https://via.placeholder.com/600x250.png?text=pg_is_in_recovery+Returns+True)
> *Keterangan Gambar:* Hasil `t` (*true*) mengkonfirmasi bahwa server bertindak sebagai standby/replica server.

#### 3. Uji Manipulasi Tabel
* **Pada Primary:**
  ```sql
  CREATE TABLE testrep (first INT, second VARCHAR(15));
  INSERT INTO testrep (first, second) VALUES (21, 'Test pertama');
  INSERT INTO testrep (first, second) VALUES (23, 'Test ke dua');
  SELECT * FROM testrep;
  ```
* **Pada Replica (Otomatis Terekam):**
  ```sql
  \dt
  SELECT * FROM testrep;
  ```
> ![Uji Data Replikasi](https://via.placeholder.com/750x320.png?text=Table+testrep+Replicated+Successfully)

---

### 4.1.5 High-Availability dan Failover (`pg_promote()`)

Jika *primary server* mengalami gangguan atau dihentikan:
```bash
sudo docker-compose stop postgres_primary
```
Kita dapat mempromosikan *replica server* agar bertindak penuh sebagai primary (read-write):
```sql
zdb=# SELECT pg_is_in_recovery(); -- Menghasilkan t (True)
zdb=# SELECT pg_promote();         -- Mempromosikan node
zdb=# SELECT pg_is_in_recovery(); -- Menghasilkan f (False)
```
> ![Promosi Replica ke Primary](https://via.placeholder.com/600x280.png?text=pg_promote+Output)

Membersihkan environment praktikum PostgreSQL:
```bash
$ dcd
```

---

## 4.2 Replikasi Master-Master Menggunakan Apache Ignite 3

Materi kedua membahas konfigurasi kluster berbasis **Apache Ignite 3.1.0** (*Master-Master Replication*), di mana setiap node mempunyai kemampuan setara untuk melakukan operasi *read-write*.

### 4.2.1 Prasyarat & Struktur File
* File konfigurasi `docker-compose.yml` dan kumpulan arsip skrip SQL (`sql.zip`) dari dokumentasi resmi Apache Ignite.
* Konfigurasi alias (`env.sh`) untuk shortcut manajemen container dan Ignite CLI (`dcli`, `dclm`).

### 4.2.2 Menjalankan Kluster Docker Compose
```bash
$ dcu
```
Menghasilkan 3 node kluster aktif (`ignite3-node1-1`, `ignite3-node2-1`, `ignite3-node3-1`).
> ![Container Apache Ignite Aktif](https://via.placeholder.com/750x250.png?text=Docker+Compose+Ignite+3+Nodes+Running)

Pemetaan port penting:
* **10300 - 10302**: REST API untuk operasi administratif & CLI koneksi.
* **10800 - 10802**: Port koneksi aplikasi klien.

---

### 4.2.3 Inisialisasi Kluster Melalui CLI
Masuk ke Ignite CLI (`dcli`), lalu lakukan inisialisasi kluster:
```bash
[node1]> cluster init --name ignite3 --metastorage-group=node1,node2,node3
[node1]> cluster status
```
> ![Inisialisasi Kluster Ignite](https://via.placeholder.com/800x300.png?text=Apache+Ignite+CLI+Cluster+Init+Success)

---

### 4.2.4 Eksekusi Skrip SQL Berbasis Sampel Data (Chinook)
Setelah file SQL di-mount ke kontainer (`/opt/ignite/sql`), jalankan skrip skema dan data:
```sql
[node1]> sql --file=/opt/ignite/sql/schema.sql
[node1]> sql --file=/opt/ignite/sql/current_catalog.sql
[node1]> sql --file=/opt/ignite/sql/media_and_genre.sql
[node1]> sql --file=/opt/ignite/sql/tracks.sql
[node1]> sql --file=/opt/ignite/sql/ee_and_cust.sql
[node1]> sql --file=/opt/ignite/sql/invoices.sql
```
> ![Eksekusi File SQL di Ignite](https://via.placeholder.com/800x350.png?text=Ignite+SQL+Schema+and+Data+Loading)

Verifikasi tabel terbuat pada `node1`:
```sql
[node1]> sql
sql-cli> SELECT * FROM system.tables WHERE schema = 'PUBLIC';
```
> ![Tabel Sistem Terdistribusi Ignite](https://via.placeholder.com/800x320.png?text=System+Tables+Query+Result)

---

### 4.2.5 Verifikasi Replikasi pada Node Lain (Node 2 & Node 3)

Melakukan koneksi ke node berikutnya (`http://localhost:10301` dan `http://localhost:10302`) untuk memastikan konsistensi data replikasi Master-Master:
```sql
[node2]> connect http://localhost:10301
[node2]> sql
sql-cli> SELECT * FROM system.tables WHERE schema = 'PUBLIC';
```
> ![Verifikasi Data di Node 2 dan 3](https://via.placeholder.com/800x350.png?text=Node+2+and+3+Replication+Consistency)

Pengujian query analitik kompleks (*Monthly Sales Summary*):
```sql
SELECT 
    CAST(EXTRACT(YEAR FROM i.InvoiceDate) AS VARCHAR) || '-' || 
    CASE 
        WHEN EXTRACT(MONTH FROM i.InvoiceDate) < 10 
        THEN '0' || CAST(EXTRACT(MONTH FROM i.InvoiceDate) AS VARCHAR) 
        ELSE CAST(EXTRACT(MONTH FROM i.InvoiceDate) AS VARCHAR) 
    END AS YearMonth,
    COUNT(DISTINCT i.InvoiceId) AS InvoiceCount,
    COUNT(DISTINCT i.CustomerId) AS CustomerCount,
    SUM(i.Total) AS MonthlyRevenue,
    AVG(i.Total) AS AverageOrderValue
FROM Invoice i
GROUP BY EXTRACT(YEAR FROM i.InvoiceDate), EXTRACT(MONTH FROM i.InvoiceDate)
ORDER BY YearMonth DESC;
```
> ![Hasil Query Analitik Bulanan](https://via.placeholder.com/800x300.png?text=Analytics+Query+Result+Master-Master)

Membersihkan seluruh kluster Apache Ignite:
```bash
$ dcd
```

---
