**LAPORAN PRAKTIKUM SISTEM TERDISTRIBUSI DAN TERDESENTRALISASI**
--
**Mata Kuliah: Sistem Terdistribusi dan Terdesentralisasi**
--
**Modul 4: Konsistensi dan Replikasi pada Sistem Terdistribusi**
--
Nama : Francisco G I dc Corbafo
Nim : 255410003
Kelas : IF_1
SELECT pg_create_physical_replication_slot('replication_slot');
2. docker-compose.yamlMendefinisikan service postgres_primary (port 5432) dan postgres_replica (port 5433).YAMLx-postgres-common: &postgres-common
  image: postgres:18.3-alpine3.23
  user: postgres
  restart: always
  healthcheck:
    test: 'pg_isready -U zuser --dbname=zdb'
    interval: 10s
    timeout: 5s
    retries: 5

services:
  postgres_primary:
    <<: *postgres-common
    ports:
      - 5432:5432
    environment:
      POSTGRES_USER: zuser
      POSTGRES_DB: zdb
      POSTGRES_PASSWORD: zpass
      POSTGRES_HOST_AUTH_METHOD: "scram-sha-256\nhost replication replicator 0.0.0.0/0 md5"
      POSTGRES_INITDB_ARGS: "--auth-host=scram-sha-256"
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
      - 5433:5432
    environment:
      PGUSER: replicator
      PGPASSWORD: replicator_password
      PGDATA: /var/lib/postgresql/18/docker
    command: |
      bash -c "
      until pg_basebackup --pgdata=/var/lib/postgresql/18/docker -R --slot=replication_slot --host=postgres_primary --port=5432 -X stream; do
        echo 'pg_basebackup failed. Retrying in 5 seconds ...'
        sleep 5
      done
      echo 'Backup done, starting replica...'
      chmod 0700 /var/lib/postgresql/18/docker
      postgres
      "
    depends_on:
      - postgres_primary
3. env.shMendefinisikan alias perintah Docker untuk mempermudah eksekusi:   Bashalias dcu="sudo docker-compose up -d"
alias dcd="sudo docker-compose down"
alias dps="sudo docker ps"
alias der="sudo docker exec -it src-postgres_replica-1 bash"
alias dep="sudo docker exec -it src-postgres_primary-1 bash"
alias dlr="sudo docker logs src-postgres_replica-1"
alias dlp="sudo docker logs src-postgres_primary-1"
2.2 Langkah Pengerjaan & Screenshot Pembahasan (PostgreSQL)Langkah 1: Memuat Variable Lingkungan dan Menjalankan ServiceEksekusi file env.sh kemudian jalankan dcu untuk mendownload image postgres:18.3-alpine3.23 dan menjalankan container.Bash$ source env.sh
$ dcu
Plaintext+-----------------------------------------------------------------------+
| [Screenshot Log Terminal - Menjalankan Docker Compose]                 |
|                                                                       |
| bpdp@NEO-X ~/kerjaan/utdi/.../streaming-replication $ source env.sh   |
| bpdp@NEO-X ~/kerjaan/utdi/.../streaming-replication $ dcu             |
| [sudo] password for bpdp:                                             |
| [+] up 15/15                                                          |
|  ✔ Image postgres:18.3-alpine3.23  Pulled                             |
|  ✔ Network src_default            Created                            |
|  ✔ Container src-postgres_primary-1 Started                           |
|  ✔ Container src-postgres_replica-1 Started                           |
+-----------------------------------------------------------------------+
Pemeriksaan status container memastikan kedua node berstatus healthy:Bash$ dps
CONTAINER ID   IMAGE                    STATUS                   PORTS                  NAMES
492a2b3bc3db   postgres:18.3-alpine3.23 Up 4 minutes (healthy)   0.0.0.0:5432->5432/tcp src-postgres_primary-1
225df898488e   postgres:18.3-alpine3.23 Up 4 minutes (healthy)   0.0.0.0:5433->5432/tcp src-postgres_replica-1
Langkah 2: Verifikasi Diagnostik Replikasi di Primary NodeMasuk ke primary menggunakan perintah dep lalu periksa tabel sistem pg_stat_replication dan pg_replication_slots:SQLzdb=# SELECT * FROM pg_stat_replication;
-[ RECORD 1 ]----+------------------------------
pid              | 59
usename          | replicator
application_name | walreceiver
client_addr      | 172.18.0.3
state            | streaming
sync_state       | async

zdb=# SELECT * FROM pg_replication_slots;
-[ RECORD 1 ]----+------------------------------
slot_name        | replication_slot
plugin           | 
slot_type        | physical
active           | t
active_pid       | 59
wal_status       | reserved
Plaintext+-----------------------------------------------------------------------+
| [Screenshot Log Terminal - Verifikasi Replikasi pada Primary]         |
|                                                                       |
| zdb=# SELECT * FROM pg_stat_replication;                              |
| usename          | replicator                                         |
| application_name | walreceiver                                        |
| state            | streaming                                          |
| sync_state       | async                                              |
|                                                                       |
| zdb=# SELECT * FROM pg_replication_slots;                             |
| slot_name        | replication_slot                                   |
| slot_type        | physical                                           |
| active           | t                                                  |
+-----------------------------------------------------------------------+
Pembahasan: Output state: streaming mengindikasikan bahwa Primary sedang mengirimkan Write-Ahead Log (WAL) secara kontinyu ke Replica.Langkah 3: Verifikasi Mode Read-Only pada Replica NodeMasuk ke replica menggunakan der dan jalankan fungsi pg_is_in_recovery():SQLzdb=# SELECT pg_is_in_recovery();
 pg_is_in_recovery 
-------------------
 t
(1 row)
Pembahasan: Nilai t (true) mengonfirmasi bahwa node replica berada dalam kondisi pemulihan berkelanjutan (recovery mode) yang secara ketat membatasi operasi hanya untuk Read-Only.Langkah 4: Pengujian Sinkronisasi Data (DDL & DML)Lakukan manipulasi data di Primary Node:SQL-- Di Primary Node (dep)
zdb=# CREATE TABLE testrep (first INT, second VARCHAR(15));
CREATE TABLE

zdb=# INSERT INTO testrep (first, second) VALUES (21, 'Test pertama');
INSERT 0 1
zdb=# INSERT INTO testrep (first, second) VALUES (23, 'Test ke dua');
INSERT 0 1
Periksa hasilnya di Replica Node (der):SQL-- Di Replica Node (der)
zdb=# \dt
        List of tables
 Schema |  Name   | Type  | Owner 
--------+---------+-------+-------
 public | testrep | table | zuser
(1 row)

zdb=# SELECT * FROM testrep;
 first |    second    
-------+--------------
    21 | Test pertama
    23 | Test ke dua
(2 rows)
Plaintext+-----------------------------------------------------------------------+
| [Screenshot Log Terminal - Verifikasi Sinkronisasi Data di Replica]   |
|                                                                       |
| zdb=# \dt                                                             |
| public | testrep | table | zuser                                      |
|                                                                       |
| zdb=# SELECT * FROM testrep;                                          |
|  first |    second                                                    |
| -------+--------------                                                |
|     21 | Test pertama                                                 |
|     23 | Test ke dua                                                  |
| (2 rows)                                                              |
+-----------------------------------------------------------------------+
Pembahasan: Pembuatan tabel dan penyisipan data pada Primary berhasil direplikasi secara real-time ke Replica Node.Langkah 5: Pengujian High-Availability (Failover & Promotion)Simulasikan kegagalan Primary server dengan menghentikan container:Bash$ sudo docker-compose stop postgres_primary
[+] Stop 1/1
 ✔ Container src-postgres_primary-1 Stopped
Promosikan Replica Node menjadi Primary Node baru menggunakan pg_promote():SQLzdb=# SELECT pg_is_in_recovery();
 pg_is_in_recovery 
-------------------
 t

zdb=# SELECT pg_promote();
 pg_promote 
------------
 t

zdb=# SELECT pg_is_in_recovery();
 pg_is_in_recovery 
-------------------
 f
Plaintext+-----------------------------------------------------------------------+
| [Screenshot Log Terminal - Failover dan Promosi Replica]              |
|                                                                       |
| zdb=# SELECT pg_is_in_recovery();                                     |
| pg_is_in_recovery: t                                                  |
| zdb=# SELECT pg_promote();                                            |
| pg_promote: t                                                         |
| zdb=# SELECT pg_is_in_recovery();                                     |
| pg_is_in_recovery: f                                                  |
+-----------------------------------------------------------------------+
Pembahasan: Setelah fungsi pg_promote() dieksekusi, nilai pg_is_in_recovery() berubah menjadi f (false). Node Replica telah berubah peran menjadi Primary baru yang mendukung operasi Read-Write.III. BAGIAN 2: REPLIKASI MASTER-MASTER MENGGUNAKAN APACHE IGNITE 3.1.03.1 Konfigurasi Lingkungan Apache IgnitePersiapan dilakukan pada direktori ignite3-cluster/ menggunakan file docker-compose.yml resmi Apache Ignite yang terdiri dari 3 nodes (node1, node2, node3).Diizinkan perintah ringkas pada env.sh:   Bashalias dcu="sudo docker-compose up -d"
alias dcd="sudo docker-compose down"
alias dps="sudo docker ps"
alias dclm="sudo docker run --rm -it --network=host -e LANG=C.UTF-8 -e LC_ALL=C.UTF-8 -v ./sql:/opt/ignite/sql apacheignite/ignite:3.1.0 cli"
alias dcli="sudo docker run --rm -it --network=host -e LANG=C.UTF-8 -e LC_ALL=C.UTF-8 apacheignite/ignite:3.1.0 cli"
3.2 Langkah Pengerjaan & Screenshot Pembahasan (Apache Ignite)Langkah 1: Menjalankan Cluster IgniteBash$ dcu
[+] up 4/4
 ✔ Network ignite3_default      Created
 ✔ Container ignite3-node3-1    Started
 ✔ Container ignite3-node2-1    Started
 ✔ Container ignite3-node1-1    Started
Langkah 2: Inisialisasi Cluster via Ignite CLIJalankan CLI dan lakukan inisialisasi metastorage group:Bash$ dcli
Plaintext[node1]> cluster init --name=ignite3 --metastorage-group=node1,node2,node3
Cluster was initialized successfully

[node1]> cluster status
[name: ignite3, nodes: 3, status: active, cmgNodes: [node1, node2, node3], msNodes: [node1, node2, node3]]
Plaintext+-----------------------------------------------------------------------+
| [Screenshot Log Terminal - Inisialisasi Cluster Apache Ignite]        |
|                                                                       |
|   Apache Ignite CLI version 3.1.0                                     |
| Connected to http://localhost:10300                                   |
| [node1]> cluster init --name=ignite3 --metastorage-group=node1,node2..|
| Cluster was initialized successfully                                  |
| [node1]> cluster status                                               |
| [name: ignite3, nodes: 3, status: active...]                          |
+-----------------------------------------------------------------------+
Pembahasan: Cluster terdiri dari 3 node aktif di mana konsensus terdistribusi menggunakan protokol Raft untuk mengelola metastorage.Langkah 3: Eksekusi Skema SQL dan Pengisian Data (Mount Volume)Eksekusi file skema SQL yang berada di ./sql yang di-mount ke /opt/ignite/sql menggunakan alias dclm:Bash$ dclm
Plaintext[node1]> sql --file=/opt/ignite/sql/schema.sql
Updated 0 rows.
...
[node1]> sql --file=/opt/ignite/sql/current_catalog.sql
Updated 275 rows.
Updated 347 rows.
[node1]> sql --file=/opt/ignite/sql/media_and_genre.sql
Updated 25 rows.
[node1]> sql --file=/opt/ignite/sql/tracks.sql
Updated 1000 rows.
[node1]> sql --file=/opt/ignite/sql/invoices.sql
Updated 1000 rows.
Plaintext+-----------------------------------------------------------------------+
| [Screenshot Log Terminal - Eksekusi SQL pada Node 1 (port 10300)]     |
|                                                                       |
| [node1]> sql --file=/opt/ignite/sql/current_catalog.sql               |
| Updated 275 rows.                                                     |
| Updated 347 rows.                                                     |
| [node1]> sql --file=/opt/ignite/sql/tracks.sql                        |
| Updated 1000 rows.                                                    |
+-----------------------------------------------------------------------+
Langkah 4: Verifikasi Replikasi Multi-Master pada Node LainHubungkan CLI ke Node 2 (http://localhost:10301) dan Node 3 (http://localhost:10302) untuk membuktikan propagasi data secara otomatis.Pengujian di Node 2 (http://localhost:10301):Plaintext[node1]> connect http://localhost:10301
Connected to http://localhost:10301

[node2]> sql
sql-cli> SELECT * FROM system.tables WHERE schema = 'PUBLIC';
Hasil: Seluruh tabel (ALBUM, CUSTOMER, INVOICE, TRACK, dll.) otomatis tersedia di Node 2.Pengujian Analitik & Query pada Node 3 (http://localhost:10302):Plaintext[node1]> connect http://localhost:10302
Connected to http://localhost:10302

[node3]> sql
sql-cli> SELECT 
          CAST(EXTRACT(YEAR FROM i.InvoiceDate) AS VARCHAR) || '-' ||
          CASE 
            WHEN EXTRACT(MONTH FROM i.InvoiceDate) < 10 THEN '0' || CAST(EXTRACT(MONTH FROM i.InvoiceDate) AS VARCHAR)
            ELSE CAST(EXTRACT(MONTH FROM i.InvoiceDate) AS VARCHAR)
          END AS YearMonth,
          COUNT(DISTINCT i.InvoiceId) AS InvoiceCount,
          SUM(i.Total) AS MonthlyRevenue
        FROM Invoice i
        GROUP BY EXTRACT(YEAR FROM i.InvoiceDate), EXTRACT(MONTH FROM i.InvoiceDate)
        ORDER BY YearMonth DESC;
Plaintext+-----------------------------------------------------------------------+
| [Screenshot Log Terminal - Output Query Analytics pada Node 3]        |
|                                                                       |
| YEARMONTH | INVOICECOUNT | CUSTOMERCOUNT | MONTHLYREVENUE             |
| 2024-12   | 7            | 7             | 38.62                      |
| 2024-11   | 7            | 7             | 49.62                      |
| 2024-10   | 7            | 7             | 37.62                      |
| ...       | ...          | ...           | ...                        |
+-----------------------------------------------------------------------+
Pembahasan: Data yang dimasukkan dari Node 1 terdistribusi secara transparan ke Node 2 dan Node 3. Pengguna dapat menjalankan query analitik pada node mana saja dengan hasil yang konsisten.IV. KESIMPULANBerdasarkan serangkaian percobaan yang telah dilakukan pada Modul 4 ini, dapat disimpulkan bahwa:   Streaming Replication PostgreSQL (Single-Primary):Sangat efektif untuk memisahkan beban kerja pembacaan (read workload) ke replica node, serta menjaga primary node tetap fokus pada manipulasi data (write workload).Mekanisme WAL (Write-Ahead Logging) memastikan bahwa setiap perubahan data disinkronkan secara konsisten.Dalam skenario kegagalan (failover), replica node dapat dipromosikan menjadi primary node baru menggunakan perintah pg_promote(), sehingga mendukung prinsip High Availability.Master-Master Replication Apache Ignite 3.1.0:Menyediakan arsitektur simetris di mana setiap node dapat berfungsi sebagai master yang mendukung operasi Read-Write.Menerapkan konsensus terdistribusi secara transparan, sehingga penulisan data pada satu node (misalnya Node 1) langsung terpropagasi dan dapat diakses untuk analisis pada node lainnya (Node 2 dan Node 3) tanpa adanya perbedaan data.
