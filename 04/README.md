# Laporan Praktikum Modul 4: Konsistensi dan Replikasi pada Sistem Terdistribusi

| | |
|---|---|
| **Nama** | Francisco G I dc Corbafp |
| **NIM** | _255410003_ |
| **Mata kuliah** | Sistem Terdistribusi dan Terdesentralisasi |
| **Tanggal praktikum** | 30 Oktober 2026 |

> **Cara memakai laporan ini:** setiap `![...](img/xx.png)` adalah tempat screenshot. Simpan screenshot di folder `img/` dengan nama file yang sama. Daftar lengkap nama file ada di bagian **Lampiran: Daftar Screenshot**.

---

## 1. Tujuan

1. Mengonfigurasi *streaming replication* (primary-replica) pada PostgreSQL 18 memakai Docker Compose.
2. Menguji replikasi data dan simulasi *High Availability* (failover) dengan `pg_promote()`.
3. Mengonfigurasi cluster Apache Ignite 3.1.0 dengan 3 node dan menguji replikasi data antar node (Master-Master).

## 2. Lingkungan

| Komponen | Keterangan |
|---|---|
| Sistem operasi | Windows dengan WSL 2 (Ubuntu) |
| Terminal | Terminal VS Code (Bash di WSL Ubuntu) |
| Docker | Docker Engine 29.1.3, Docker Compose v5.6.1 (Docker Desktop) |
| Image PostgreSQL | `postgres:18.3-alpine3.23` |
| Image Ignite | `apacheignite/ignite:3.1.0` |

![Versi Docker dan Docker Compose](img/00-versi-docker.png)
*Screenshot: output `docker --version`, `docker compose version`.*

---

## 3. Bagian 4.1: Streaming Replication PostgreSQL

### 3.1 Konsep singkat

*Streaming replication* mengirim catatan perubahan (WAL, *Write-Ahead Log*) dari server **primary** ke server **replica (standby)** secara terus-menerus. Primary melayani baca-tulis, sedangkan replica hanya melayani baca (*hot standby*). Pada praktikum ini mode replikasi adalah **asynchronous** (`sync_state = async`): primary tidak menunggu konfirmasi replica sebelum menyelesaikan transaksi, sehingga lebih cepat tetapi ada kemungkinan replica tertinggal sesaat.

### 3.2 Struktur direktori dan file

```
streaming-replication/
├── 00_init.sql
├── docker-compose.yaml
└── env.sh
```

| File | Fungsi |
|---|---|
| `00_init.sql` | Dijalankan otomatis saat primary pertama kali diinisialisasi. Membuat user `replicator` dengan hak replikasi dan *replication slot* fisik `replication_slot`. |
| `docker-compose.yaml` | Mendefinisikan dua service: `postgres_primary` dan `postgres_replica`. |
| `env.sh` | Kumpulan alias Bash (`dcu`, `dcd`, `dps`, `der`, `dep`, `dlr`, `dlp`) untuk mempersingkat perintah. Pada praktikum ini `sudo` dihapus dan `docker-compose` diganti `docker compose`, karena memakai Docker Desktop. |

**Parameter penting pada primary:**

| Parameter | Fungsi |
|---|---|
| `wal_level=replica` | Menyimpan informasi WAL yang cukup untuk replikasi. |
| `hot_standby=on` | Mengizinkan replica menerima koneksi baca saat berstatus standby. |
| `max_wal_senders=10` | Jumlah maksimum proses pengirim WAL. |
| `max_replication_slots=10` | Jumlah maksimum slot replikasi. |
| `hot_standby_feedback=on` | Replica memberi tahu primary baris mana yang masih dipakai, agar tidak dibersihkan terlalu dini. |
| `POSTGRES_HOST_AUTH_METHOD` | Mengatur `pg_hba.conf`: autentikasi `scram-sha-256`, ditambah aturan koneksi replikasi untuk user `replicator`. |

**Cara kerja replica:** saat start, replica menjalankan `pg_basebackup` dari primary (`-R` membuat file konfigurasi standby, `--slot` memakai slot replikasi, `-X stream` ikut mengalirkan WAL). Perintah diulang tiap 5 detik sampai primary siap, baru kemudian `postgres` dijalankan dalam mode standby.

![Isi tiga file dan ls -la](img/01-struktur-file.png)
*Screenshot: output `ls -la` yang menampilkan `00_init.sql`, `docker-compose.yaml`, `env.sh`.*

### 3.3 Menjalankan docker compose

```bash
source env.sh
dcu
dps
```

Pada percobaan pertama, replica gagal start dengan galat `failed to bind host port 0.0.0.0:5433/tcp: address already in use`. Port 5433 di komputer sudah dipakai proses lain, sehingga port host replica diubah ke **5434**:

```bash
sed -i 's/- 5433:5432/- 5434:5432/' docker-compose.yaml
dcd
dcu
```

Perubahan ini hanya memengaruhi port yang dipetakan ke komputer host. Replica tetap terhubung ke primary lewat jaringan internal Docker (`postgres_primary:5432`), sehingga replikasi tidak terpengaruh.

![Galat port 5433 bentrok](img/02-error-port-5433.png)
*Screenshot: pesan galat `address already in use` saat `dcu` pertama.*

![dcu berhasil dan dps healthy](img/03-dcu-dps-healthy.png)
*Screenshot: output `dcu` dan `dps` dengan kedua container berstatus `(healthy)`, port 5432 dan 5434.*

### 3.4 Log replica

```bash
dlr
```

Log menampilkan galat `Connection refused` di awal. Ini normal karena replica mencoba terhubung sebelum primary siap, lalu mengulang. Setelah primary siap muncul `Backup done, starting replica...`, `entering standby mode`, `database system is ready to accept read-only connections`, dan `started streaming WAL from primary at 0/3000000 on timeline 1`.

![Log replica](img/04-log-replica.png)
*Screenshot: output `dlr`.*

### 3.5 Pengujian status replikasi

**Pada primary** (`dep`, lalu `psql -U zuser -d zdb`):

```sql
\x
SELECT * FROM pg_stat_replication;
SELECT * FROM pg_replication_slots;
\x
\dt
```

Hasil penting:

| Kolom | Nilai | Arti |
|---|---|---|
| `usename` | `replicator` | Replica terhubung memakai user replikasi. |
| `application_name` | `walreceiver` | Proses penerima WAL di replica. |
| `client_addr` | `172.20.0.3` | Alamat IP container replica di jaringan Docker. |
| `state` | `streaming` | WAL sedang dialirkan secara kontinu. |
| `sent_lsn` = `write_lsn` = `flush_lsn` = `replay_lsn` | `0/304F638` | Replica sudah menerima, menulis, dan menerapkan WAL sampai posisi yang sama dengan primary (tidak ada *lag*). |
| `sync_state` | `async` | Replikasi asinkron. |

Pada `pg_replication_slots`: slot `replication_slot` bertipe `physical`, `active = t`, dan `active_pid = 61` sama dengan `pid` pada `pg_stat_replication`, yaitu slot dipakai oleh koneksi replica tersebut. `wal_status = reserved` berarti WAL yang dibutuhkan replica ditahan oleh primary. Sebelum tabel dibuat, `\dt` menampilkan `Did not find any tables.`

![pg_stat_replication di primary](img/05-pg-stat-replication.png)
*Screenshot: hasil `SELECT * FROM pg_stat_replication;` (tampilan `\x`).*

![pg_replication_slots di primary](img/06-pg-replication-slots.png)
*Screenshot: hasil `SELECT * FROM pg_replication_slots;` dan `\dt` kosong di primary.*

**Pada replica** (`der`, lalu `psql -U zuser -d zdb`):

```sql
SELECT pg_is_in_recovery();
\dt
```

`pg_is_in_recovery()` bernilai `t` (true), artinya server berada dalam mode standby (recovery), hanya menerima WAL dan melayani baca. Belum ada tabel.

![pg_is_in_recovery di replica](img/07-replica-in-recovery.png)
*Screenshot: `SELECT pg_is_in_recovery();` = `t` dan `\dt` kosong di replica.*

### 3.6 Pengujian replikasi data

**Di primary** dibuat tabel dan diisi dua baris:

```sql
CREATE TABLE testrep (first INT, second VARCHAR(15));
SELECT * FROM testrep;
INSERT INTO testrep (first, second) VALUES (21, 'Test pertama');
INSERT INTO testrep (first, second) VALUES (23, 'Test ke dua');
SELECT * FROM testrep;
```

![Membuat tabel dan insert di primary](img/08-primary-create-insert.png)
*Screenshot: seluruh proses `CREATE TABLE`, `INSERT`, dan `SELECT` di primary (2 rows).*

**Di replica**, tanpa mengisi apa pun, tabel dan data sudah muncul:

```sql
\dt
SELECT * FROM testrep;
```

![Data testrep muncul di replica](img/09-replica-testrep.png)
*Screenshot: `\dt` menampilkan `testrep` dan `SELECT * FROM testrep;` menampilkan 2 baris di replica.*

Ini membuktikan bahwa perubahan di primary (DDL dan DML) direplikasi otomatis ke replica melalui WAL.

### 3.7 High Availability: failover

Primary dimatikan, lalu replica dipromosikan:

```bash
docker compose stop postgres_primary
dps
```

```sql
SELECT pg_is_in_recovery();   -- t
SELECT pg_promote();          -- t
SELECT pg_is_in_recovery();   -- f
```

![Primary dihentikan](img/10-stop-primary.png)
*Screenshot: `docker compose stop postgres_primary` dan `dps` yang hanya menampilkan replica.*

![pg_promote](img/11-pg-promote.png)
*Screenshot: urutan `pg_is_in_recovery()` (t), `pg_promote()` (t), `pg_is_in_recovery()` (f).*

Nilai `f` setelah `pg_promote()` menunjukkan replica telah keluar dari mode recovery dan menjadi primary baru yang dapat menerima operasi baca-tulis.

### 3.8 Mematikan semua container

```bash
dcd
dps
```

![dcd dan dps kosong](img/12-dcd-postgres.png)
*Screenshot: `dcd` (container dan network `Removed`) dan `dps` kosong.*

### 3.9 Pembahasan 4.1

1. **Pola primary-replica.** Hanya satu server (primary) yang menerima tulis, sehingga urutan perubahan jelas dan konflik tulis tidak terjadi. Konsekuensinya, primary menjadi titik tunggal kegagalan untuk operasi tulis sampai replica dipromosikan.
2. **Asinkron vs sinkron.** `sync_state = async` berarti latensi tulis rendah, tetapi bila primary mati mendadak, transaksi terakhir yang belum sampai ke replica bisa hilang. Mode sinkron (`synchronous_standby_names`) menghilangkan risiko itu dengan harga latensi lebih tinggi. Pada pengujian ini nilai LSN primary dan replica identik sehingga tidak ada *lag* yang terlihat.
3. **Peran replication slot.** Slot membuat primary menahan WAL sampai dikonsumsi replica, sehingga replica yang sempat terputus tidak kehilangan data perubahan. Risikonya, jika replica mati lama, WAL menumpuk dan memakan disk di primary.
4. **Failover manual.** `pg_promote()` dijalankan manual. Pada sistem produksi, deteksi kegagalan dan promosi biasanya diotomatisasi (misalnya dengan Patroni). Setelah promosi, primary lama tidak otomatis menjadi replica dan perlu dikonfigurasi ulang.
5. **Konsistensi.** Replica melayani pembacaan, tetapi pada mode asinkron pembacaan di replica dapat sedikit tertinggal dari primary (*eventual consistency*). Aplikasi yang butuh data terbaru harus membaca dari primary.

---

## 4. Bagian 4.2: Replikasi Master-Master dengan Apache Ignite 3.1.0

### 4.1 Konsep singkat

Pada Apache Ignite 3, semua node dalam cluster dapat menerima koneksi klien untuk baca maupun tulis, sehingga sering disebut Master-Master. Data dibagi ke dalam partisi dan disalin ke beberapa node, dan konsistensi replika dijaga oleh protokol konsensus (Raft). Berbeda dengan primary-replica pada PostgreSQL, klien bebas terhubung ke node mana saja.

### 4.2 File yang dipakai

| File | Fungsi |
|---|---|
| `docker-compose.yml` | Mendefinisikan 3 node Ignite (`node1`, `node2`, `node3`). Diunduh dari dokumentasi Apache Ignite. |
| `sql.zip` (`sql/`) | Berisi 6 file SQL contoh (basis data Chinook): `schema.sql`, `current_catalog.sql`, `media_and_genre.sql`, `tracks.sql`, `ee_and_cust.sql`, `invoices.sql`. |
| `env.sh` | Alias `dcu`, `dcd`, `dps`, `dclm` (CLI dengan folder `sql` di-mount), `dcli` (CLI tanpa mount). |

![Isi sql.zip dengan tree](img/13-tree-sql.png)
*Screenshot: output `unzip sql.zip` dan `tree sql/` (6 file `.sql`).*

### 4.3 Menjalankan 3 node

```bash
source env.sh
dcu
dps
```

| Container | Port REST API (host) | Port aplikasi (host) |
|---|---|---|
| `ignite3-node1-1` | 10300 | 10800 |
| `ignite3-node2-1` | 10301 | 10801 |
| `ignite3-node3-1` | 10302 | 10802 |

Port 10300-10302 dipakai untuk REST API administratif dan koneksi CLI, sedangkan 10800-10802 untuk koneksi aplikasi.

![dcu dan dps Ignite](img/14-dcu-dps-ignite.png)
*Screenshot: output `dcu` dan `dps` yang menampilkan tiga container Ignite dan port-nya.*

**Alamat IP node** (dari `docker inspect`):

| Node | IP |
|---|---|
| `ignite3-node1-1` | 172.20.0.3 |
| `ignite3-node2-1` | 172.20.0.4 |
| `ignite3-node3-1` | 172.20.0.2 |

![docker inspect IP node](img/15-docker-inspect-ip.png)
*Screenshot: tiga perintah `docker inspect` beserta IP-nya.*

> **Catatan:** pada Docker Desktop, jaringan bridge container umumnya tidak dapat dijangkau langsung dari WSL, sehingga `ping` ke IP container dapat gagal (`Destination Host Unreachable`). Hal ini tidak memengaruhi praktikum karena CLI terhubung lewat `localhost` dengan port yang dipetakan.

### 4.4 Inisialisasi cluster

```bash
dclm
```

```
cluster init --name=ignite3 --metastorage-group=node1,node2,node3
cluster status
```

Hasil: `Cluster was initialized successfully` dan `[name: ignite3, nodes: 3, status: active, cmgNodes: [node1, node2, node3], msNodes: [node1, node2, node3]]`.

`--metastorage-group` menentukan node yang menyimpan metadata cluster (Meta Storage). Karena `--cluster-management-group` tidak diisi, node yang sama juga menjadi Cluster Management Group. Ketiga node ikut serta sehingga metadata direplikasi di ketiganya.

> Pada percobaan pertama, perintah `cluster init` dan `cluster status` tertempel sekaligus dan menghasilkan galat `Unmatched arguments`. Perintah diulang satu per satu dan berhasil.

![cluster init dan status](img/16-cluster-init-status.png)
*Screenshot: `cluster init ...` (berhasil) dan `cluster status` (`nodes: 3, status: active`).*

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
| `schema.sql` | 20 pernyataan DDL (`Updated 0 rows.`) |
| `current_catalog.sql` | 275 dan 347 baris |
| `media_and_genre.sql` | 25 dan 5 baris |
| `tracks.sql` | 1000, 1000, 1000, 503 baris |
| `ee_and_cust.sql` | 8 dan 59 baris |
| `invoices.sql` | 412, 1000, 1000, 240, 18, 1000 (8 kali), 715 baris |

![schema.sql](img/17-sql-schema.png)
*Screenshot: hasil `schema.sql`.*

![File SQL lainnya](img/18-sql-data.png)
*Screenshot: hasil `current_catalog.sql`, `media_and_genre.sql`, `tracks.sql`, `ee_and_cust.sql`, `invoices.sql`.*

> Catatan: sebelum file SQL dijalankan, `SELECT * FROM system.tables WHERE schema = 'PUBLIC';` menghasilkan `(empty)`. Setelah `schema.sql` dijalankan, tabel muncul.

### 4.6 Memeriksa tabel pada node 1

```
sql
SELECT * FROM system.tables WHERE schema = 'PUBLIC';
```

Muncul 11 tabel: `ALBUM`, `PLAYLISTTRACK`, `TRACK`, `CUSTOMER`, `MEDIATYPE`, `PLAYLIST`, `ARTIST`, `INVOICELINE`, `EMPLOYEE`, `GENRE`, `INVOICE`.

![system.tables di node 1](img/19-system-tables-node1.png)
*Screenshot: hasil `system.tables` di node 1 (11 tabel).*

### 4.7 Verifikasi di node 2 (port 10301)

```
connect http://localhost:10301
sql
```

Tabel `PUBLIC` yang sama muncul di node 2, dan query analytics berikut menghasilkan data:

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

Baris teratas hasil di node 2:

| YEARMONTH | INVOICECOUNT | CUSTOMERCOUNT | MONTHLYREVENUE | AVERAGEORDERVALUE |
|---|---|---|---|---|
| 2024-12 | 7 | 7 | 38.62 | 5.5171428571428571 |
| 2024-11 | 7 | 7 | 49.62 | 7.0885714285714285 |
| 2024-10 | 7 | 7 | 37.62 | 5.3742857142857142 |

![system.tables di node 2](img/20-system-tables-node2.png)
*Screenshot: `connect http://localhost:10301`, prompt `[node2]>`, dan hasil `system.tables`.*

![Query analytics di node 2](img/21-analytics-node2.png)
*Screenshot: query analytics dan bagian atas tabel hasil di node 2.*

### 4.8 Verifikasi di node 3 (port 10302)

```
connect http://localhost:10302
sql
```

Query analytics yang sama dijalankan dan menghasilkan data yang **identik** dengan node 2 (2024-12: 7 invoice, 7 pelanggan, 38.62, 5.5171428571428571; 2024-11: 49.62; dan seterusnya sampai 2020-01: 35.64).

![Query analytics di node 3](img/22-analytics-node3.png)
*Screenshot: `connect http://localhost:10302`, prompt `[node3]>`, query analytics, dan bagian atas tabel hasil di node 3.*

### 4.9 Mematikan semua node

```bash
dcd
dps
```

![dcd dan dps Ignite](img/23-dcd-ignite.png)
*Screenshot: `dcd` (3 container dan network `Removed`) dan `dps` kosong.*

### 4.10 Pembahasan 4.2

1. **Data masuk lewat satu node, terbaca di semua node.** Seluruh file SQL dijalankan hanya melalui node 1, tetapi node 2 dan node 3 mengembalikan tabel dan hasil agregasi yang sama persis. Ini menunjukkan data dan skema tersedia di seluruh cluster, dan klien dapat memakai node mana pun sebagai titik akses.
2. **Kolokasi dan zona distribusi.** Pada `system.tables`, sebagian besar tabel berada di zona `CHINOOK` (data dipartisi, dengan kolom kolokasi seperti `ARTISTID`, `ALBUMID`, `CUSTOMERID` sehingga data yang saling terkait ditempatkan berdekatan), sedangkan `MEDIATYPE` dan `GENRE` berada di zona `CHINOOKREPLICATED` (tabel kecil yang disalin penuh ke banyak node untuk mempercepat join). Jadi replikasi dipengaruhi konfigurasi zona.
3. **Konsistensi.** Karena replika dijaga konsensus Raft, hasil query di node 1, 2, dan 3 konsisten. Ini berbeda dengan pola asinkron PostgreSQL di bagian 4.1 yang dapat menghasilkan pembacaan sedikit tertinggal di replica.
4. **Perbandingan dengan PostgreSQL.**

| Aspek | PostgreSQL streaming replication | Apache Ignite 3 cluster |
|---|---|---|
| Topologi | Satu primary, satu atau lebih replica | Semua node setara sebagai titik akses |
| Tulis | Hanya di primary | Dapat melalui node mana pun |
| Baca dari replika | Replica hanya baca, bisa tertinggal (async) | Semua node dapat dibaca |
| Failover | Manual (`pg_promote()`) atau dengan alat tambahan | Ditangani cluster (konsensus) selama mayoritas node hidup |
| Kelebihan | Sederhana, matang, dukungan luas | Skalabilitas dan ketersediaan lebih baik |
| Kekurangan | Primary titik tunggal untuk tulis | Konfigurasi dan operasional lebih kompleks |

---

## 5. Kendala dan Solusi

| No | Kendala | Penyebab | Solusi |
|---|---|---|---|
| 1 | `source` tidak dikenali di terminal VS Code | Terminal bawaan adalah PowerShell | Memakai WSL Ubuntu (Bash) |
| 2 | Container lama `Restarting` dan bentrok nama | Sisa percobaan sebelumnya | `docker rm -f` dan `docker container prune` |
| 3 | Replica gagal start: port 5433 `address already in use` | Port 5433 dipakai proses lain di host | Mengubah port host replica menjadi 5434 |
| 4 | Perintah SQL tidak jalan (`bash: SELECT: command not found`) | `psql` ikut tertempel dan dijalankan setelah `exit`, sehingga SQL diketik di shell | Mengetik satu perintah per baris dan menunggu prompt berubah (`zdb=#`) |
| 5 | `docker-compose.yml` dan `sql.zip` Ignite gagal diunduh dari WSL (timeout) | `ignite.apache.org` tidak terjangkau dari WSL | Mengunduh lewat browser Windows lalu menyalin ke WSL dengan `cp` |
| 6 | `cluster init` galat `Unmatched arguments` | Dua perintah tertempel sekaligus | Menjalankan perintah satu per satu |
| 7 | Tabel `PUBLIC` kosong di `system.tables` | File SQL belum dijalankan; `sql --file=...` sempat diketik di prompt `sql-cli>` | Menjalankan `sql --file=...` dari prompt `[node1]>` |
| 8 | `ping` ke IP container gagal | Jaringan container Docker Desktop tidak terjangkau dari WSL | Tidak berpengaruh; CLI terhubung lewat `localhost` |

## 6. Kesimpulan

1. Streaming replication PostgreSQL 18 berhasil dikonfigurasi. Status `streaming`, slot replikasi aktif, dan data pada tabel `testrep` di primary otomatis muncul di replica.
2. Simulasi failover berhasil: setelah primary dihentikan, `pg_promote()` mengubah `pg_is_in_recovery()` dari `t` menjadi `f`, yaitu replica menjadi primary baru.
3. Cluster Apache Ignite 3.1.0 dengan 3 node berhasil diinisialisasi (`nodes: 3, status: active`). Data yang dimuat melalui node 1 dapat dibaca dengan hasil identik dari node 2 dan node 3, sehingga replikasi antar node terbukti.
4. Pola primary-replica (PostgreSQL) cocok untuk skenario sederhana dengan satu titik tulis, sedangkan cluster Ignite memberi akses yang lebih setara antar node, dengan biaya kompleksitas konfigurasi yang lebih tinggi.

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
| 10 | `img/09-replica-testrep.png` | Data `testrep` muncul di replica |
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
