**Laporan Praktikum Modul 4**
**Konsistensi dan Replikasi pada Sistem Terdistribusi**
	
**Mata Kuliah	Sistem Terdistribusi dan Terdesentralisasi**

Nama : Francisco G I dc Corbafo

NIM : 255410003

Kelas : IF-1



Catatan: Tempat tangkapan layar ditandai dengan ![...](img/xxx.png). Ganti dengan screenshot hasil pengerjaan Anda sendiri, simpan di folder img/ pada repo.

Daftar Isi
1. Tujuan
2. Landasan Teori Singkat
3. Alat dan Prasyarat
4. Bagian A: Streaming Replication PostgreSQL 18
5. Bagian B: Replikasi Master-Master Apache Ignite 3
6. Perbandingan dan Analisis
7. Kendala dan Solusi
8. Kesimpulan
9. Referensi
1. Tujuan
1. Memahami konsep replikasi dan konsistensi data pada sistem terdistribusi.
2. Mengonfigurasi streaming replication model single-primary (primary–replica) pada PostgreSQL 18 menggunakan Docker Compose.
3. Menguji propagasi data dari primary ke replica, serta simulasi High Availability dengan promosi replica (pg_promote()).
4. Mengonfigurasi cluster Apache Ignite 3.1.0 tiga node dan menguji replikasi data antar-node (model "master-master").
5. Membandingkan kedua pendekatan replikasi.

   
2. Landasan Teori Singkat
Replikasi adalah penyimpanan salinan data yang sama pada beberapa node. Manfaatnya: ketersediaan tinggi (high availability), toleransi kegagalan, dan peningkatan kinerja baca. Konsekuensinya: harus ada mekanisme menjaga konsistensi antar salinan.


2.1 Streaming Replication PostgreSQL
- Model single-primary: hanya satu server (primary) menerima operasi tulis; replica (standby) menerima dan memutar ulang (replay) WAL (Write-Ahead Log) dari primary secara kontinu, dan hanya melayani baca (hot standby).
- Mode yang dipakai pada modul ini asinkron (sync_state = async): primary tidak menunggu konfirmasi replica sebelum commit. Artinya replica bisa tertinggal sesaat (replication lag) → konsistensi bersifat eventual pada sisi replica.
- Replication slot memastikan primary menyimpan WAL sampai dikonsumsi replica, sehingga replica tidak kehilangan data walau sempat terputus.


2.2 Apache Ignite 3
- Basis data terdistribusi berbentuk cluster beberapa node. Data dibagi menjadi partisi dan direplikasi antar node menggunakan konsensus Raft.
- Klien dapat terhubung ke node mana pun dalam cluster untuk menjalankan operasi baca/tulis. Istilah "master-master" pada laporan ini digunakan secara sederhana; secara teknis, tiap partisi tetap memiliki satu leader Raft dan node lain dapat meneruskan operasi ke leader tersebut.
- Zone menentukan strategi distribusi data (jumlah partisi dan replika). Pada dataset contoh terdapat zone CHINOOK dan CHINOOKREPLICATED.

  
3. Alat dan Prasyarat
Komponen	Keterangan
Sistem operasi	Linux (jika OS lain, sesuaikan perintah)
Docker Engine	Terpasang dan daemon dockerd aktif
Docker Compose	Terpasang
PostgreSQL	Image postgres:18.3-alpine3.23
Apache Ignite	Image apacheignite/ignite:3.1.0


Memastikan Docker berjalan
docker --version
docker compose version     # atau: docker-compose --version
Mengaktifkan daemon (sesuaikan dengan init system):
# init system dinit (contoh pada modul)
sudo dinitctl start dockerd

# init system systemd
sudo systemctl start docker

4. Bagian A: Streaming Replication PostgreSQL 18
4.1 Langkah-langkah Pengerjaan
Langkah 1 – Buat struktur direktori
mkdir streaming-replication && cd streaming-replication
touch 00_init.sql docker-compose.yaml env.sh pg_hba.conf
Struktur akhir:
streaming-replication/
├── 00_init.sql
├── docker-compose.yaml
└── env.sh
Penting: nama direktori menjadi prefix nama container. Pada modul, direktori bernama src sehingga container bernama src-postgres_primary-1 dan src-postgres_replica-1. Jika direktori Anda bernama streaming-replication, maka nama container menjadi streaming-replication-postgres_primary-1, dst.

Langkah 2 – Isi 00_init.sql
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'replicator_password';
SELECT pg_create_physical_replication_slot('replication_slot');
Langkah 3 – Isi docker-compose.yaml
x-postgres-common:
  &postgres-common
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
      POSTGRES_INITDB_ARGS: "--auth-host=scram-sha-256"
    command: |
      postgres
      -c wal_level=replica
      -c hot_standby=on
      -c max_wal_senders=10
      -c max_replication_slots=10
      -c hot_standby_feedback=on
      -c hba_file=/etc/postgresql/pg_hba.conf
    volumes:
      - ./00_init.sql:/docker-entrypoint-initdb.d/00_init.sql:ro
      - ./pg_hba.conf:/etc/postgresql/pg_hba.conf:ro

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
Buat juga file pg_hba.conf dengan isi berikut. File ini secara eksplisit mengizinkan autentikasi password untuk koneksi biasa dan koneksi replikasi.
# Koneksi lokal di dalam container
local   all             all                                     trust

# Koneksi database biasa
host    all             all             0.0.0.0/0               scram-sha-256
host    all             all             ::/0                    scram-sha-256

# Koneksi replikasi fisik
host    replication     replicator      0.0.0.0/0               scram-sha-256
host    replication     replicator      ::/0                    scram-sha-256
Pastikan indentasi YAML menggunakan spasi, bukan tab. Simpan pg_hba.conf di direktori yang sama dengan docker-compose.yaml.
Langkah 4 – Isi env.sh (alias, opsional)
Ganti <nama-project> dengan nama project Compose yang tampil pada docker ps (biasanya nama direktori kerja). Hapus tanda < > saat menggantinya.
# Gunakan plugin Docker Compose v2
alias dcu="sudo docker compose up -d"
alias dcd="sudo docker compose down"
alias dps="sudo docker ps"
# Ganti <nama-project> sesuai nama project/container yang tampil di docker ps
alias der="sudo docker exec -it <nama-project>-postgres_replica-1 bash"
alias dep="sudo docker exec -it <nama-project>-postgres_primary-1 bash"
alias dlr="sudo docker logs <nama-project>-postgres_replica-1"
alias dlp="sudo docker logs <nama-project>-postgres_primary-1"
Muat alias ke shell:
source env.sh
Langkah 5 – Jalankan cluster
dcu

Image akan di-pull otomatis jika belum ada.
Langkah 6 – Periksa status container
dps
Pastikan kolom STATUS kedua container menunjukkan (healthy). Jika tidak, lihat log:
dlr   # log replica
dlp   # log primary

Pada log replica yang normal akan terlihat urutan: pg_basebackup sempat gagal (primary belum siap) → retry → Backup done, starting replica... → entering standby mode → started streaming WAL from primary.

Langkah 7 – Verifikasi status replikasi di primary
dep
psql -U zuser -d zdb
\x
SELECT * FROM pg_stat_replication;
SELECT * FROM pg_replication_slots;


Langkah 8 – Verifikasi status di replica
der
psql -U zuser -d zdb
SELECT pg_is_in_recovery();

Langkah 9 – Uji replikasi data
a. Sebelum membuat tabel (di primary dan replica):
\dt
Hasil: Did not find any tables. pada keduanya.

b. Manipulasi data di primary:
CREATE TABLE testrep (first INT, second VARCHAR(15));
INSERT INTO testrep (first, second) VALUES (21, 'Test pertama');
INSERT INTO testrep (first, second) VALUES (23, 'Test ke dua');
SELECT * FROM testrep;

c. Cek di replica:
\dt
SELECT * FROM testrep;

d. (Tambahan, disarankan) Buktikan replica bersifat read-only:
-- di replica
INSERT INTO testrep (first, second) VALUES (99, 'coba tulis');
-- Hasil yang diharapkan: ERROR: cannot execute INSERT in a read-only transaction
Langkah 10 – Simulasi High Availability (failover)
a. Matikan primary:
sudo docker compose stop postgres_primary

b. Promosikan replica menjadi primary baru:
der
psql -U zuser -d zdb
SELECT pg_is_in_recovery();   -- t
SELECT pg_promote();          -- t
SELECT pg_is_in_recovery();   -- f (sekarang bisa read-write)

c. (Tambahan) Buktikan node hasil promosi bisa menulis:
INSERT INTO testrep (first, second) VALUES (25, 'Setelah promote');
SELECT * FROM testrep;
Langkah 11 – Bersihkan lingkungan
dcd
dps

Catatan: docker compose down menghapus container; karena tidak ada volume data, data ikut hilang. Ini wajar untuk praktikum.

4.2 Penjelasan Konfigurasi
00_init.sql
Perintah	Fungsi
CREATE USER replicator WITH REPLICATION ...	Membuat user khusus yang memiliki atribut REPLICATION agar boleh menarik WAL.
pg_create_physical_replication_slot('replication_slot')	Membuat slot fisik sehingga primary menahan WAL sampai dikonsumsi replica.


File ini diletakkan di /docker-entrypoint-initdb.d/ sehingga dijalankan sekali saat primary diinisialisasi.
Parameter server primary
Parameter	Fungsi
wal_level=replica	Level detail WAL yang cukup untuk replikasi fisik.
hot_standby=on	Mengizinkan standby menerima query baca saat recovery.
max_wal_senders=10	Jumlah maksimum proses pengirim WAL (koneksi replikasi).
max_replication_slots=10	Jumlah maksimum slot replikasi.
hot_standby_feedback=on	Replica memberi tahu primary tentang query yang sedang berjalan agar vacuum tidak menghapus baris yang masih dibutuhkan (mengurangi konflik query).
pg_hba.conf	Menentukan aturan autentikasi koneksi database dan koneksi replikasi; file dipasang melalui volume dan dipilih dengan hba_file.


Konfigurasi replica
Elemen	Fungsi
PGUSER, PGPASSWORD	Kredensial yang dipakai pg_basebackup untuk terhubung ke primary.
PGDATA=/var/lib/postgresql/18/docker	Direktori data PostgreSQL 18 pada image Docker (berbeda dari versi lama: /var/lib/postgresql/data).
pg_basebackup -R	Menyalin seluruh data primary, dan -R otomatis membuat standby.signal serta pengaturan primary_conninfo agar node berjalan sebagai standby.
--slot=replication_slot	Memakai slot yang dibuat di 00_init.sql.
-X stream	Mengalirkan WAL selama proses backup berlangsung.
Loop until ... done	Mengulang percobaan tiap 5 detik jika primary belum siap.
chmod 0700	PostgreSQL menolak start jika izin direktori data terlalu longgar.
depends_on	Replica dijalankan setelah container primary dibuat (belum menjamin primary siap, karena itu ada retry).


4.3 Hasil dan Analisis
Bagian ini harus disesuaikan dengan hasil eksekusi dan screenshot. Jika praktikum belum dijalankan, tulis sebagai hasil yang diharapkan, bukan hasil yang sudah terbukti.

- pg_stat_replication di primary menampilkan satu record dengan usename = replicator, state = streaming, sync_state = async. Nilai sent_lsn, write_lsn, flush_lsn, dan replay_lsn identik, berarti replica sudah mengejar primary (lag ≈ 0).
- pg_replication_slots menunjukkan replication_slot bertipe physical, active = t, dan wal_status = reserved.
- pg_is_in_recovery() = t pada replica membuktikan node tersebut berstatus standby.
- Tabel testrep dan datanya yang dibuat di primary otomatis muncul di replica tanpa tindakan tambahan.
- Setelah pg_promote(), pg_is_in_recovery() menjadi f, artinya replica sudah menjadi primary yang menerima tulis.
Catatan konsistensi dan keterbatasan:
1. Karena replikasi asinkron, transaksi yang sudah di-commit di primary tetapi belum sampai di replica dapat hilang jika primary mati mendadak sebelum failover.
2. Failover pada modul ini manual. Untuk otomatis diperlukan alat seperti Patroni atau repmgr.
3. Primary lama tidak bisa langsung kembali sebagai replica; perlu dibangun ulang (pg_basebackup atau pg_rewind) agar tidak terjadi split-brain.
4. Aplikasi harus diarahkan ke primary baru (port 5433 pada setup ini).
5. Bagian B: Replikasi Master-Master Apache Ignite 3
5.1 Langkah-langkah Pengerjaan
Langkah 1 – Siapkan direktori dan unduh berkas
mkdir ignite3-cluster && cd ignite3-cluster
curl -O https://ignite.apache.org/docs/ignite3/latest/quick-start/sql-files/docker-compose.yml
curl -O https://ignite.apache.org/docs/ignite3/latest/quick-start/sql-files/sql.zip
unzip sql.zip
Pastikan docker-compose.yml memakai image apacheignite/ignite:3.1.0.

Struktur direktori sql/:
sql/
├── current_catalog.sql
├── ee_and_cust.sql
├── invoices.sql
├── media_and_genre.sql
├── schema.sql
└── tracks.sql
Langkah 2 – Buat env.sh
alias dcu="sudo docker compose up -d"
alias dcd="sudo docker compose down"
alias dps="sudo docker ps"
alias dclm="sudo docker run --rm -it --network=host -e LANG=C.UTF-8 -e LC_ALL=C.UTF-8 -v ./sql:/opt/ignite/sql apacheignite/ignite:3.1.0 cli"
alias dcli="sudo docker run --rm -it --network=host -e LANG=C.UTF-8 -e LC_ALL=C.UTF-8 apacheignite/ignite:3.1.0 cli"
source env.sh
dclm = CLI dengan folder ./sql ter-mount ke /opt/ignite/sql; dcli = CLI biasa.

Langkah 3 – Jalankan cluster
dcu
Hasil yang diharapkan: network ignite3_default dibuat dan tiga container (ignite3-node1-1, ignite3-node2-1, ignite3-node3-1) berstatus Started.

Langkah 4 – Periksa container dan port
dps
Port host	Fungsi
10300–`10302`	REST API administratif (node1–node3); dipakai juga oleh Ignite CLI.
10800–`10802`	Port koneksi klien/aplikasi.



Langkah 5 – Cek alamat IP tiap node dan uji konektivitas
sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <CONTAINER_ID>
ping -c 2 <IP_NODE>
Lakukan untuk ketiga node (pada modul: 172.18.0.2, 172.18.0.3, 172.18.0.4).

Langkah 6 – Inisialisasi cluster
dcli
Jawab Y saat ditanya terhubung ke http://localhost:10300, lalu:
cluster init --name=ignite3 --metastorage-group=node1,node2,node3
cluster status
Hasil yang diharapkan: name: ignite3, nodes: 3, status: active.

Inisialisasi hanya dilakukan sekali. Jika keluar CLI (Ctrl-D) lalu masuk lagi, cluster tetap aktif dan tidak perlu di-init ulang.

Catatan: teks modul menyebut image 3.0.0 pada beberapa perintah, sedangkan versi yang dipakai adalah 3.1.0. Gunakan 3.1.0 secara konsisten.

Langkah 7 – Eksekusi berkas SQL di node1
Keluar CLI (Ctrl-D), lalu masuk dengan dclm agar folder sql ter-mount:
dclm
Jalankan berurutan (urutan penting karena ada ketergantungan antar tabel):
sql --file=/opt/ignite/sql/schema.sql
sql --file=/opt/ignite/sql/current_catalog.sql
sql --file=/opt/ignite/sql/media_and_genre.sql
sql --file=/opt/ignite/sql/tracks.sql
sql --file=/opt/ignite/sql/ee_and_cust.sql
sql --file=/opt/ignite/sql/invoices.sql


Keluaran Updated N rows. menandakan perintah berhasil.
Langkah 8 – Periksa tabel di node1
sql
SELECT * FROM system.tables WHERE schema = 'PUBLIC';

Langkah 9 – Verifikasi propagasi data di node2 (port 10301)
Keluar mode SQL (Ctrl-D), kemudian:
connect http://localhost:10301
sql
SELECT * FROM system.tables WHERE schema = 'PUBLIC';
Jawab Y jika diminta konfirmasi pindah koneksi. Prompt akan berubah menjadi [node2].

Jalankan query analitik:
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

Langkah 10 – Verifikasi propagasi data di node3 (port 10302)
connect http://localhost:10302
sql
Ulangi query analitik di atas.

Langkah 11 – (Tambahan, disarankan) Uji tulis dari node lain
Untuk membuktikan sifat master-master, tulis data dari node3, lalu baca dari node1:
-- di node3 (port 10302)
INSERT INTO Genre (GenreId, Name) VALUES (9999, 'Genre Uji Replikasi');
-- di node1 (port 10300)
SELECT * FROM Genre WHERE GenreId = 9999;
Sesuaikan nama kolom/tabel dengan skema pada schema.sql.

Langkah 12 – Matikan semua node
dcd

5.2 Penjelasan
- cluster init --metastorage-group=node1,node2,node3: menentukan node-node yang menyimpan metadata cluster (katalog, skema, topologi) dan ikut dalam konsensus. Pemakaian ketiga node membuat metadata tetap tersedia walau satu node gagal (mayoritas 2 dari 3).
- Port 10300–10302 adalah REST/CLI per node; ketiganya berada pada satu cluster yang sama, sehingga terhubung ke node mana pun menampilkan data yang sama.
- Zone CHINOOK dan CHINOOKREPLICATED (terlihat pada kolom ZONE_NAME hasil system.tables): tabel besar seperti TRACK, INVOICE berada di zone CHINOOK, sedangkan tabel kecil/referensi seperti GENRE dan MEDIATYPE berada di zone CHINOOKREPLICATED yang direplikasi ke lebih banyak node.
- Kolom TABLE_COLOCATION_COLUMNS: data terkait (misal INVOICE dengan CUSTOMERID) ditempatkan pada partisi yang sama (colocation) sehingga join lebih efisien.
5.3 Hasil dan Analisis
Bagian ini harus disesuaikan dengan hasil eksekusi dan screenshot. Jika praktikum belum dijalankan, tulis sebagai hasil yang diharapkan, bukan hasil yang sudah terbukti.

- Perubahan (pembuatan skema dan pengisian data) yang dilakukan melalui node1 langsung terlihat di node2 dan node3 dengan hasil query identik (misal 2024-12 → InvoiceCount 7, MonthlyRevenue 38.62; 2024-11 → 49.62).
- Hal ini membuktikan data tersebar/tereplikasi pada seluruh node dan klien dapat mengakses data melalui node mana pun.
- Berbeda dengan PostgreSQL (satu primary tetap), pada Ignite tidak ada langkah promosi manual; ketersediaan dijaga oleh protokol konsensus Raft.
6. Perbandingan dan Analisis
Aspek	PostgreSQL Streaming Replication	Apache Ignite 3 Cluster
Model	Single-primary (primary–replica)	Multi-node, klien bisa akses node mana pun ("master-master")
Operasi tulis	Hanya di primary	Dapat melalui node mana pun
Operasi baca	Primary dan replica	Semua node
Mekanisme sinkronisasi	Pengiriman WAL (asinkron pada setup ini)	Konsensus Raft per partisi
Konsistensi	Eventual pada replica (ada lag)	Konsisten kuat berbasis konsensus
Failover	Manual (pg_promote()) atau dengan alat tambahan	Otomatis oleh protokol cluster
Kompleksitas setup	Relatif sederhana, banyak alat pendukung	Perlu inisialisasi cluster dan memahami zone/partisi
Cocok untuk	Skala baca, backup, HA sederhana	Data terdistribusi skala besar, komputasi in-memory


Hubungan dengan teori CAP: pendekatan PostgreSQL asinkron cenderung mengutamakan availability dan latensi tulis rendah dengan mengorbankan konsistensi pada replica; sedangkan Ignite dengan Raft mengutamakan konsistensi dan membutuhkan mayoritas node hidup agar operasi tetap berjalan.
7. Kendala dan Solusi
Kendala	Penyebab	Solusi
bash: dcu: command not found	Alias belum dimuat	Jalankan source env.sh
Log replica: pg_basebackup: error: connection ... failed	Primary belum siap saat replica start	Normal; loop retry akan mencoba lagi hingga berhasil
Container tidak healthy	Konfigurasi YAML/kredensial keliru	Periksa dlp dan dlr, perhatikan indentasi YAML
Nama container tidak sesuai alias	Nama project Compose berbeda dari alias	Sesuaikan nama project berdasarkan hasil docker ps
docker-compose: command not found	Docker versi baru memakai plugin	Gunakan docker compose (tanpa tanda hubung)
Port 5432/5433 sudah dipakai	PostgreSQL lokal berjalan	Hentikan layanan lokal atau ubah port di compose
File SQL tidak ditemukan di CLI Ignite	Masuk CLI dengan dcli, bukan dclm	Gunakan dclm agar folder sql ter-mount
Error saat memuat SQL Ignite	Urutan file salah	Jalankan schema.sql terlebih dahulu sesuai urutan
Tautan Docker pada modul tidak terbuka	Salah ketik https://.docs.docker.com	Gunakan https://docs.docker.com/engine/install/


8. Kesimpulan
1. Streaming replication PostgreSQL 18 berhasil dikonfigurasi menggunakan Docker Compose; data pada primary terbukti tereplikasi otomatis ke replica, yang bersifat read-only selama berada dalam mode recovery.
2. Replica dapat dipromosikan menjadi primary dengan pg_promote() sebagai mekanisme high availability manual saat primary gagal. Replikasi asinkron memiliki risiko kehilangan data yang belum tereplikasi.
3. Cluster Apache Ignite 3.1.0 tiga node berhasil diinisialisasi; data yang dimasukkan melalui satu node dapat diakses melalui node lain dengan hasil yang sama, menunjukkan replikasi antar-node.
4. Pemilihan model replikasi bergantung pada kebutuhan: PostgreSQL cocok untuk skenario primary–replica yang sederhana, sedangkan Ignite cocok untuk basis data terdistribusi dengan akses dari banyak node.
9. Referensi
1. Purnomosidi D. P., B. Modul 4: Konsistensi dan Replikasi pada Sistem Terdistribusi. Praktikum Sistem Terdistribusi dan Terdesentralisasi, Universitas Teknologi Digital Indonesia.
2. Eremeykin. How to Setup Single Primary PostgreSQL Replication with Docker Compose. https://medium.com/@eremeykin/how-to-setup-single-primary-postgresql-replication-with-docker-compose-98c48f233bbf
3. PostgreSQL Documentation – High Availability, Load Balancing, and Replication. https://www.postgresql.org/docs/current/high-availability.html
4. Apache Ignite 3 Documentation. https://ignite.apache.org/docs/ignite3/latest/index
5. Docker Documentation. https://docs.docker.com/
