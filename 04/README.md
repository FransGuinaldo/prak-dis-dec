# Laporan Praktikum Modul 4
## Konsistensi dan Replikasi pada Sistem Terdistribusi

| | |
|---|---|
| **Nama** | Francisco G I dc Corbafo |
| **NIM** | 255410003|
| **Mata kuliah** | Sistem Terdistribusi dan Terdesentralisasi |
| **Tanggal praktikum** | 30 Oktober 2026 |


---

## 1. Tujuan

1. Mengonfigurasi *streaming replication* PostgreSQL 18 (primary-replica) dengan Docker Compose.
2. Menguji replikasi data dan simulasi *High Availability* (failover) dengan `pg_promote()`.
3. Mengonfigurasi cluster Apache Ignite 3.1.0 dengan 3 node dan menguji replikasi antar node (Master-Master).

## 2. Lingkungan

| Komponen | Keterangan |
|---|---|
| Sistem operasi | Windows dengan WSL 2 (Ubuntu) |
| Terminal | Terminal VS Code (Bash, WSL Ubuntu) |
| Docker | Docker 29.1.3, Docker Compose v5.6.1 (Docker Desktop) |
| Image PostgreSQL | `postgres:18.3-alpine3.23` |
| Image Ignite | `apacheignite/ignite:3.1.0` |

<img width="878" height="838" alt="image" src="https://github.com/user-attachments/assets/953d7b82-b599-4abc-a60a-a8c220a516ef" />


---

## 3. Bagian 4.1: Streaming Replication PostgreSQL

### 3.1 Konsep

*Streaming replication* mengirim catatan perubahan (WAL, *Write-Ahead Log*) dari **primary** ke **replica (standby)** secara kontinu. Primary melayani baca-tulis, replica hanya baca (*hot standby*). Mode yang dipakai adalah **asinkron**: primary tidak menunggu konfirmasi replica, sehingga cepat, tetapi replica dapat tertinggal sesaat.

### 3.2 File yang dipakai

```
streaming-replication/
├── 00_init.sql
├── docker-compose.yaml
└── env.sh
```

| File | Fungsi |
|---|---|
| `00_init.sql` | Dijalankan saat primary pertama kali diinisialisasi: membuat user `replicator` dan *replication slot* `replication_slot`. |
| `docker-compose.yaml` | Mendefinisikan `postgres_primary` dan `postgres_replica`. |
| `env.sh` | Alias Bash (`dcu`, `dcd`, `dps`, `der`, `dep`, `dlr`, `dlp`). `sudo` dihapus dan `docker-compose` diganti `docker compose` karena memakai Docker Desktop. |

**Parameter penting primary:**

| Parameter | Fungsi |
|---|---|
| `wal_level=replica` | Menyimpan informasi WAL yang cukup untuk replikasi. |
| `hot_standby=on` | Replica dapat dibaca saat standby. |
| `max_wal_senders=10` | Jumlah maksimum proses pengirim WAL. |
| `max_replication_slots=10` | Jumlah maksimum slot replikasi. |
| `hot_standby_feedback=on` | Replica memberi tahu primary baris yang masih dipakai agar tidak dibersihkan dini. |
| `POSTGRES_HOST_AUTH_METHOD` | Autentikasi `scram-sha-256` dan aturan koneksi replikasi untuk `replicator`. |

**Cara kerja replica:** saat start, replica menjalankan `pg_basebackup` dari primary (`-R` membuat konfigurasi standby, `--slot` memakai slot replikasi, `-X stream` ikut mengalirkan WAL), diulang tiap 5 detik sampai primary siap, lalu menjalankan `postgres` dalam mode standby.

<img width="768" height="812" alt="image" src="https://github.com/user-attachments/assets/adbc5ebf-aba0-4daf-9cf4-ce79b5de9376" />

<img width="472" height="73" alt="img01b-struktur file" src="https://github.com/user-attachments/assets/4f815e02-f7da-471c-b532-9f3824ae6631" />


```
-rw-r--r-- 1 frans frans  146 Oct 30 08:31 00_init.sql
-rw-r--r-- 1 frans frans 1417 Oct 30 08:31 docker-compose.yaml
-rw-r--r-- 1 frans frans  365 Oct 30 08:31 env.sh
```

### 3.3 Menjalankan docker compose

```bash
source env.sh
dcu
dps
```

Percobaan pertama gagal untuk replica:

```
Error response from daemon: failed to set up container networking: ... failed to bind host port 0.0.0.0:5433/tcp: address already in use
```

Port 5433 di komputer host sudah dipakai proses lain. Port host replica diubah menjadi **5434**:

```bash
sed -i 's/- 5433:5432/- 5434:5432/' docker-compose.yaml
dcd
dcu
```

Perubahan ini hanya memengaruhi port yang dipetakan ke host. Replica tetap terhubung ke primary lewat jaringan internal Docker (`postgres_primary:5432`), sehingga replikasi tidak terpengaruh.

<img width="1496" height="217" alt="Screenshot 2026-10-31 050419" src="https://github.com/user-attachments/assets/fb51d173-a800-45eb-83c7-60956bc92a63" />


Setelah itu kedua container berstatus `healthy`:

```
NAMES                                      STATUS                        PORTS
streaming-replication-postgres_replica-1   Up About a minute (healthy)   0.0.0.0:5434->5432/tcp
streaming-replication-postgres_primary-1   Up About a minute (healthy)   0.0.0.0:5432->5432/tcp
```

![dcu dan dps healthy](img/03-dcu-dps-healthy.png)

### 3.4 Log replica

```bash
dlr
```

Cuplikan penting:

```
pg_basebackup: error: connection to server at "postgres_primary" (172.20.0.2), port 5432 failed: Connection refused
pg_basebackup failed. Retrying in 5 seconds ...
Backup done, starting replica...
... LOG:  entering standby mode
... LOG:  database system is ready to accept read-only connections
... LOG:  started streaming WAL from primary at 0/3000000 on timeline 1
```

Galat `Connection refused` di awal wajar: replica sempat mencoba terhubung sebelum primary siap, lalu mengulang sampai berhasil.

![Log replica](img/04-log-replica.png)

### 3.5 Status replikasi di primary

```bash
dep
psql -U zuser -d zdb
\x
SELECT * FROM pg_stat_replication;
```

```
-[ RECORD 1 ]----+------------------------------
pid              | 61
usename          | replicator
application_name | walreceiver
client_addr      | 172.20.0.3
state            | streaming
sent_lsn         | 0/304F638
write_lsn        | 0/304F638
flush_lsn        | 0/304F638
replay_lsn       | 0/304F638
sync_priority    | 0
sync_state       | async
```

![pg_stat_replication](img/05-pg-stat-replication.png)

```sql
SELECT * FROM pg_replication_slots;
```

```
slot_name           | replication_slot
slot_type           | physical
active              | t
active_pid          | 61
restart_lsn         | 0/304F638
wal_status          | reserved
```

`\dt` pada primary menampilkan `Did not find any tables.`

![pg_replication_slots](img/06-pg-replication-slots.png)

| Pengamatan | Arti |
|---|---|
| `state = streaming` | WAL dialirkan secara kontinu ke replica. |
| `sent_lsn = write_lsn = flush_lsn = replay_lsn = 0/304F638` | Replica sudah menerima, menulis, dan menerapkan WAL sampai posisi yang sama dengan primary (tidak ada *lag*). |
| `sync_state = async` | Replikasi asinkron. |
| `active = t`, `active_pid = 61` | Slot dipakai koneksi replica yang sama (pid 61 sama dengan di `pg_stat_replication`). |
| `wal_status = reserved` | Primary menahan WAL yang dibutuhkan replica. |

### 3.6 Status di replica

```bash
der
psql -U zuser -d zdb
SELECT pg_is_in_recovery();
\dt
```

```
 pg_is_in_recovery
-------------------
 t
Did not find any tables.
```

Nilai `t` menunjukkan server berada dalam mode standby/recovery: hanya menerima WAL dan melayani baca.

![pg_is_in_recovery di replica](img/07-replica-in-recovery.png)

### 3.7 Pengujian replikasi data

Di **primary**:

```sql
CREATE TABLE testrep (first INT, second VARCHAR(15));
SELECT * FROM testrep;
INSERT INTO testrep (first, second) VALUES (21, 'Test pertama');
INSERT INTO testrep (first, second) VALUES (23, 'Test ke dua');
SELECT * FROM testrep;
```

```
CREATE TABLE
(0 rows)
INSERT 0 1
INSERT 0 1
 first |    second
-------+--------------
    21 | Test pertama
    23 | Test ke dua
(2 rows)
```

![Create dan insert di primary](img/08-primary-create-insert.png)

Di **replica**, tanpa mengisi apa pun:

```sql
\dt
SELECT * FROM testrep;
```

```
 Schema |  Name   | Type  | Owner
--------+---------+-------+-------
 public | testrep | table | zuser

 first |    second
-------+--------------
    21 | Test pertama
    23 | Test ke dua
(2 rows)
```

Tabel dan data muncul otomatis di replica. Ini membuktikan DDL dan DML di primary direplikasi melalui WAL.

![Data testrep di replica](img/09-replica-testrep.png)

### 3.8 High Availability (failover)

```bash
docker compose stop postgres_primary
dps
```

```
✔ Container streaming-replication-postgres_primary-1 Stopped
```

`dps` kini hanya menampilkan container replica.

![Primary dihentikan](img/10-stop-primary.png)

Di replica:

```sql
SELECT pg_is_in_recovery();   -- t
SELECT pg_promote();          -- t
SELECT pg_is_in_recovery();   -- f
```

Nilai `f` setelah `pg_promote()` menunjukkan replica telah keluar dari mode recovery dan menjadi primary baru yang dapat menerima baca-tulis.

![pg_promote](img/11-pg-promote.png)

### 3.9 Mematikan semua

```bash
dcd
dps
```

```
✔ Container streaming-replication-postgres_replica-1 Removed
✔ Container streaming-replication-postgres_primary-1 Removed
✔ Network streaming-replication_default              Removed
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

![dcd dan dps kosong](img/12-dcd-postgres.png)

### 3.10 Pembahasan 4.1

1. **Pola primary-replica.** Hanya primary yang menerima tulis, sehingga urutan perubahan jelas dan konflik tulis tidak terjadi. Konsekuensinya, primary menjadi titik tunggal kegagalan untuk tulis sampai replica dipromosikan.
2. **Asinkron vs sinkron.** Dengan `sync_state = async`, latensi tulis rendah, tetapi bila primary mati mendadak, transaksi terakhir yang belum sampai ke replica dapat hilang. Mode sinkron (`synchronous_standby_names`) menghilangkan risiko itu dengan latensi lebih tinggi. Pada pengujian ini semua nilai LSN identik sehingga tidak ada *lag* yang terlihat.
3. **Replication slot.** Slot membuat primary menahan WAL sampai dikonsumsi replica, sehingga replica yang sempat terputus tidak kehilangan perubahan. Risikonya, bila replica mati lama, WAL menumpuk dan memakan disk primary.
4. **Failover manual.** `pg_promote()` dijalankan manual. Pada produksi, deteksi kegagalan dan promosi biasanya diotomatisasi (misalnya Patroni). Primary lama tidak otomatis menjadi replica dan perlu dikonfigurasi ulang.
5. **Konsistensi.** Replica melayani baca, tetapi pada mode asinkron pembacaan dapat sedikit tertinggal dari primary (*eventual consistency*). Aplikasi yang butuh data terbaru sebaiknya membaca dari primary.

---

## 4. Bagian 4.2: Replikasi Master-Master dengan Apache Ignite 3.1.0

### 4.1 Konsep

Pada Apache Ignite 3, setiap node dapat menerima koneksi klien untuk baca maupun tulis, sehingga disebut Master-Master. Data dibagi ke partisi dan disalin ke beberapa node, dan konsistensi replika dijaga oleh protokol konsensus (Raft). Berbeda dengan primary-replica, klien bebas terhubung ke node mana saja.

### 4.2 File yang dipakai

| File | Fungsi |
|---|---|
| `docker-compose.yml` | Mendefinisikan 3 node Ignite (`node1`, `node2`, `node3`), diunduh dari dokumentasi Apache Ignite. |
| `sql.zip` | Berisi 6 file SQL contoh (basis data Chinook). |
| `env.sh` | Alias `dcu`, `dcd`, `dps`, `dclm` (CLI dengan folder `sql` di-mount), `dcli`. |

```
sql/
├── current_catalog.sql
├── ee_and_cust.sql
├── invoices.sql
├── media_and_genre.sql
├── schema.sql
└── tracks.sql

1 directory, 6 files
```

![unzip dan tree sql](img/13-tree-sql.png)

### 4.3 Menjalankan 3 node

```bash
source env.sh
dcu
dps
```

| Container | Port REST API (host) | Port aplikasi (host) | IP (docker inspect) |
|---|---|---|---|
| `ignite3-node1-1` | 10300 | 10800 | 172.20.0.3 |
| `ignite3-node2-1` | 10301 | 10801 | 172.20.0.4 |
| `ignite3-node3-1` | 10302 | 10802 | 172.20.0.2 |

![dcu dan dps Ignite](img/14-dcu-dps-ignite.png)

![docker inspect IP](img/15-docker-inspect-ip.png)

> **Catatan:** pada Docker Desktop, jaringan bridge container umumnya tidak dapat dijangkau langsung dari WSL, sehingga `ping` ke IP container dapat gagal (`Destination Host Unreachable`). Hal ini tidak memengaruhi praktikum karena CLI terhubung lewat `localhost` dengan port yang dipetakan.

### 4.4 Inisialisasi cluster

```bash
dclm
```

```
Connected to http://localhost:10300
The cluster is not initialized. Run cluster init command to initialize it.
[node1]> cluster init --name=ignite3 --metastorage-group=node1,node2,node3
Cluster was initialized successfully
[node1]> cluster status
[name: ignite3, nodes: 3, status: active, cmgNodes: [node1, node2, node3], msNodes: [node1, node2, node3]]
```

`--metastorage-group` menentukan node yang menyimpan metadata cluster. Karena `--cluster-management-group` tidak diisi, node yang sama juga menjadi Cluster Management Group, sehingga metadata direplikasi di ketiga node.

![cluster init dan status](img/16-cluster-init-status.png)

### 4.5 Menjalankan file SQL

Dijalankan dari prompt `[node1]>` (terhubung ke `http://localhost:10300`):

```
sql --file=/opt/ignite/sql/schema.sql
sql --file=/opt/ignite/sql/current_catalog.sql
sql --file=/opt/ignite/sql/media_and_genre.sql
sql --file=/opt/ignite/sql/tracks.sql
sql --file=/opt/ignite/sql/ee_and_cust.sql
sql --file=/opt/ignite/sql/invoices.sql
```

| File | Hasil |
|---|---|
| `schema.sql` | 20 kali `Updated 0 rows.` |
| `current_catalog.sql` | 275 dan 347 baris |
| `media_and_genre.sql` | 25 dan 5 baris |
| `tracks.sql` | 1000, 1000, 1000, 503 baris |
| `ee_and_cust.sql` | 8 dan 59 baris |
| `invoices.sql` | 412, 1000, 1000, 240, 18, 1000 (8 kali), 715 baris |

Sebelum `schema.sql` dijalankan, `system.tables` mengembalikan `(empty)`.

![schema.sql](img/17-sql-schema.png)

![File SQL data](img/18-sql-data.png)

### 4.6 Tabel di node 1

```sql
SELECT * FROM system.tables WHERE schema = 'PUBLIC';
```

| TABLE_NAME | ZONE_NAME | TABLE_COLOCATION_COLUMNS |
|---|---|---|
| ALBUM | CHINOOK | ARTISTID |
| PLAYLISTTRACK | CHINOOK | PLAYLISTID, TRACKID |
| TRACK | CHINOOK | ALBUMID |
| CUSTOMER | CHINOOK | CUSTOMERID |
| MEDIATYPE | CHINOOKREPLICATED | MEDIATYPEID |
| PLAYLIST | CHINOOK | PLAYLISTID |
| ARTIST | CHINOOK | ARTISTID |
| INVOICELINE | CHINOOK | INVOICEID |
| EMPLOYEE | CHINOOK | EMPLOYEEID |
| GENRE | CHINOOKREPLICATED | GENREID |
| INVOICE | CHINOOK | CUSTOMERID |

![system.tables node 1](img/19-system-tables-node1.png)

### 4.7 Verifikasi di node 2 (port 10301)

```
[node1]> connect http://localhost:10301
Connected to http://localhost:10301
[node2]> sql
```

Tabel `PUBLIC` yang sama (11 tabel) muncul di node 2.

![system.tables node 2](img/20-system-tables-node2.png)

Query analytics:

```sql
-- Monthly sales summary for the last 12 months
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
FROM
    Invoice i
GROUP BY
    EXTRACT(YEAR FROM i.InvoiceDate), EXTRACT(MONTH FROM i.InvoiceDate)
ORDER BY
    YearMonth DESC;
```

Hasil di node 2 (cuplikan):

| YEARMONTH | INVOICECOUNT | CUSTOMERCOUNT | MONTHLYREVENUE | AVERAGEORDERVALUE |
|---|---|---|---|---|
| 2024-12 | 7 | 7 | 38.62 | 5.5171428571428571 |
| 2024-11 | 7 | 7 | 49.62 | 7.0885714285714285 |
| 2024-10 | 7 | 7 | 37.62 | 5.3742857142857142 |
| 2024-04 | 5 | 5 | 33.66 | 6.7320000000000000 |
| 2024-02 | 5 | 5 | 27.72 | 5.5440000000000000 |
| ... | ... | ... | ... | ... |
| 2020-01 | 6 | 6 | 35.64 | 5.9400000000000000 |

![Analytics node 2](img/21-analytics-node2.png)

### 4.8 Verifikasi di node 3 (port 10302)

```
[node2]> connect http://localhost:10302
Connected to http://localhost:10302
[node3]> sql
```

Query analytics yang sama menghasilkan data **identik** dengan node 2 (2024-12: 7 invoice, 7 pelanggan, 38.62, 5.5171428571428571; 2024-11: 49.62; ... ; 2020-01: 35.64).

![Analytics node 3](img/22-analytics-node3.png)

### 4.9 Mematikan semua node

```bash
dcd
dps
```

Hasil yang diharapkan: ketiga container dan network `ignite3_default` berstatus `Removed`, dan `dps` kosong.

![dcd dan dps Ignite](img/23-dcd-ignite.png)

### 4.10 Pembahasan 4.2

1. **Data masuk lewat satu node, terbaca di semua node.** Semua file SQL dijalankan hanya melalui node 1, tetapi node 2 dan node 3 mengembalikan tabel dan hasil agregasi yang sama persis. Ini menunjukkan data dan skema tersedia di seluruh cluster, dan klien dapat memakai node mana pun sebagai titik akses.
2. **Zona dan kolokasi.** Sebagian besar tabel berada di zona `CHINOOK` (data dipartisi, kolom kolokasi seperti `ARTISTID`, `ALBUMID`, `CUSTOMERID` menempatkan data terkait berdekatan), sedangkan `MEDIATYPE` dan `GENRE` berada di zona `CHINOOKREPLICATED` (tabel kecil yang disalin penuh ke banyak node untuk mempercepat join).
3. **Konsistensi.** Replika dijaga konsensus Raft, sehingga hasil query di node 1, 2, dan 3 konsisten. Ini berbeda dengan replikasi asinkron PostgreSQL yang dapat menghasilkan pembacaan sedikit tertinggal di replica.
4. **Perbandingan dengan PostgreSQL.**

| Aspek | PostgreSQL streaming replication | Apache Ignite 3 cluster |
|---|---|---|
| Topologi | Satu primary, satu atau lebih replica | Semua node setara sebagai titik akses |
| Tulis | Hanya di primary | Dapat melalui node mana pun |
| Baca dari replika | Replica hanya baca, bisa tertinggal (async) | Semua node dapat dibaca |
| Failover | Manual (`pg_promote()`) atau alat tambahan | Ditangani cluster (konsensus) selama mayoritas node hidup |
| Kelebihan | Sederhana, matang, dukungan luas | Skalabilitas dan ketersediaan lebih baik |
| Kekurangan | Primary titik tunggal untuk tulis | Konfigurasi dan operasional lebih kompleks |

---

## 5. Kesesuaian dengan Modul

| Langkah di modul | Hasil praktikum | Sesuai |
|---|---|---|
| Dua container PostgreSQL `healthy` (hal. 4) | Primary dan replica `(healthy)` | Ya |
| Log replica (hal. 5) | `Connection refused` di awal, lalu `started streaming WAL from primary` | Ya |
| `pg_stat_replication` (hal. 6) | `state = streaming`, `sync_state = async` | Ya |
| `pg_replication_slots` (hal. 7) | `replication_slot`, `physical`, `active = t` | Ya |
| `pg_is_in_recovery()` di replica (hal. 8) | `t` | Ya |
| `testrep` dibuat di primary (hal. 9) | 2 baris | Ya |
| Data muncul di replica (hal. 10) | Tabel dan 2 baris muncul | Ya |
| Failover (hal. 11) | `t`, `t`, `f` | Ya |
| `dcd` PostgreSQL (hal. 12) | Semua `Removed`, `dps` kosong | Ya |
| 3 node Ignite, port 10300-10302 dan 10800-10802 (hal. 13) | Sama | Ya |
| `cluster init` dan `cluster status` (hal. 15) | `nodes: 3, status: active` | Ya |
| `schema.sql` dan file SQL lain (hal. 17, 19) | Jumlah baris sama persis | Ya |
| `system.tables` (hal. 18) | 11 tabel, dua zona | Ya |
| Query analytics node 2 dan node 3 (hal. 21, 22) | Hasil identik, 2024-12 = 38.62 | Ya |
| `dcd` Ignite (hal. 22) | Dijalankan di akhir praktikum | Lihat bagian 4.9 |

**Perbedaan wajar:** port replica 5434 (bukan 5433), nama container `streaming-replication-...` (bukan `src-...`), tanpa `sudo`, serta ID container, IP, dan waktu yang berbeda.

---

## 6. Kendala dan Solusi

| No | Kendala | Penyebab | Solusi |
|---|---|---|---|
| 1 | `source` tidak dikenali di terminal VS Code | Terminal bawaan adalah PowerShell | Memakai WSL Ubuntu (Bash) |
| 2 | Container lama `Restarting` dan bentrok nama | Sisa percobaan sebelumnya | `docker rm -f` dan `docker container prune` |
| 3 | Replica gagal start: port 5433 `address already in use` | Port 5433 dipakai proses lain di host | Mengubah port host replica ke 5434 |
| 4 | `bash: SELECT: command not found` | `psql` ikut tertempel dan dijalankan setelah `exit`, sehingga SQL diketik di shell | Mengetik satu perintah per baris dan menunggu prompt berubah (`zdb=#`) |
| 5 | `docker-compose.yml` dan `sql.zip` Ignite gagal diunduh dari WSL (timeout) | `ignite.apache.org` tidak terjangkau dari WSL | Mengunduh lewat browser Windows lalu menyalin dengan `cp` |
| 6 | `cluster init` galat `Unmatched arguments` | Dua perintah tertempel sekaligus | Menjalankan perintah satu per satu |
| 7 | `system.tables` kosong | File SQL belum dijalankan; `sql --file=...` sempat diketik di prompt `sql-cli>` | Menjalankan `sql --file=...` dari prompt `[node1]>` |
| 8 | `ping` ke IP container gagal | Jaringan container Docker Desktop tidak terjangkau dari WSL (dan IP yang di-ping sempat salah) | Tidak berpengaruh; CLI terhubung lewat `localhost` |

## 7. Kesimpulan

1. Streaming replication PostgreSQL 18 berhasil dikonfigurasi: status `streaming`, slot replikasi aktif, dan data `testrep` di primary otomatis muncul di replica.
2. Simulasi failover berhasil: setelah primary dihentikan, `pg_promote()` mengubah `pg_is_in_recovery()` dari `t` menjadi `f`, yaitu replica menjadi primary baru.
3. Cluster Apache Ignite 3.1.0 dengan 3 node berhasil diinisialisasi (`nodes: 3, status: active`). Data yang dimuat lewat node 1 terbaca identik dari node 2 dan node 3, sehingga replikasi antar node terbukti.
4. Pola primary-replica (PostgreSQL) cocok untuk skenario sederhana dengan satu titik tulis, sedangkan cluster Ignite memberi akses lebih setara antar node dengan biaya konfigurasi yang lebih tinggi.

---

## Lampiran: Daftar Screenshot

| No | File | Isi |
|---|---|---|
| 1 | `img/00-versi-docker.png` | Versi Docker dan Compose |
| 2 | `img/01-struktur-file.png` | `ls -la` tiga file PostgreSQL |
| 3 | `img/02-error-port-5433.png` | Galat port 5433 bentrok |
| 4 | `img/03-dcu-dps-healthy.png` | `dcu` dan `dps` (healthy) |
| 5 | `img/04-log-replica.png` | Log replica (`dlr`) |
| 6 | `img/05-pg-stat-replication.png` | `pg_stat_replication` |
| 7 | `img/06-pg-replication-slots.png` | `pg_replication_slots` dan `\dt` kosong |
| 8 | `img/07-replica-in-recovery.png` | `pg_is_in_recovery()` = t di replica |
| 9 | `img/08-primary-create-insert.png` | Membuat dan mengisi `testrep` di primary |
| 10 | `img/09-replica-testrep.png` | Data `testrep` di replica |
| 11 | `img/10-stop-primary.png` | Primary dihentikan |
| 12 | `img/11-pg-promote.png` | `pg_promote()` dan status recovery |
| 13 | `img/12-dcd-postgres.png` | `dcd` dan `dps` kosong (PostgreSQL) |
| 14 | `img/13-tree-sql.png` | `unzip` dan `tree sql/` |
| 15 | `img/14-dcu-dps-ignite.png` | `dcu` dan `dps` Ignite |
| 16 | `img/15-docker-inspect-ip.png` | IP tiap node Ignite |
| 17 | `img/16-cluster-init-status.png` | `cluster init` dan `cluster status` |
| 18 | `img/17-sql-schema.png` | Hasil `schema.sql` |
| 19 | `img/18-sql-data.png` | Hasil file SQL data |
| 20 | `img/19-system-tables-node1.png` | `system.tables` di node 1 |
| 21 | `img/20-system-tables-node2.png` | `system.tables` di node 2 |
| 22 | `img/21-analytics-node2.png` | Query analytics di node 2 |
| 23 | `img/22-analytics-node3.png` | Query analytics di node 3 |
| 24 | `img/23-dcd-ignite.png` | `dcd` dan `dps` kosong (Ignite) |

## Referensi

- Modul 4, Praktikum Sistem Terdistribusi dan Terdesentralisasi, Universitas Teknologi Digital Indonesia.
- Dokumentasi Apache Ignite 3: https://ignite.apache.org/docs/ignite3/latest/index
- Eremeykin, *How to set up single primary PostgreSQL replication with Docker Compose*: https://medium.com/@eremeykin/how-to-setup-single-primary-postgresql-replication-with-docker-compose-98c48f233bbf
