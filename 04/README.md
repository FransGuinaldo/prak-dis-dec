# Laporan Praktikum Modul 4

## Konsistensi dan Replikasi pada Sistem Terdistribusi

**Mata Kuliah:** Sistem Terdistribusi dan Terdesentralisasi  
**Program Studi:** Informatika - Universitas Teknologi Digital Indonesia (UTDI)  
**Penulis Modul:** Dr. Bambang Purnomosidi D. P.  

Repositori ini berisi implementasi lengkap, kode sumber konfigurasi, dan panduan praktikum **Modul 4**, yang mencakup dua topik utama:
1. **Streaming Replication** menggunakan **PostgreSQL 18** (Primary-Standby / High Availability).
2. **Replikasi Master-Master** menggunakan **Apache Ignite 3.1.0** (Distributed In-Memory Cluster).

---

## 4.1 Streaming Replication Menggunakan PostgreSQL 18

Materi ini membahas konfigurasi *streaming replication* pada PostgreSQL 18 dengan perubahan signifikan pada lokasi direktori data yang kini berada di direktori `/var/lib/postgresql/18/docker` (menggunakan base image `postgres:18.3-alpine3.23`).

### 1. Struktur Direktori dan File
```text
streaming-replication/
├── 00_init.sql
├── docker-compose.yaml
└── env.sh
```

### 2. Kode Sumber Konfigurasi

#### A. `00_init.sql`
File ini berisi perintah SQL yang dieksekusi secara otomatis saat *primary server* pertama kali diinisialisasi untuk membuat user replikasi dan slot fisik:
```sql
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'replicator_password';
SELECT pg_create_physical_replication_slot('replication_slot');
```

#### B. `docker-compose.yaml`
File konfigurasi untuk mendefinisikan container `postgres_primary` dan `postgres_replica`:
```yaml
x-postgres-common:
  &postgres-common
  image: postgres:18.3-alpine3.23
  user: postgres
  restart: always
  healthcheck:
    test: ['CMD-SHELL', 'pg_isready -U zuser -d zdb']
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
      - postgres_primary
```

#### C. `env.sh`
File definisi alias Bash untuk mempermudah eksekusi perintah (dengan menyesuaikan nama prefix folder praktikum **`prak-dis-dec`**):
```bash
alias dcu="sudo docker-compose up -d"
alias dcd="sudo docker-compose down"
alias dps="sudo docker ps"
alias der="sudo docker exec -it prak-dis-dec-postgres_replica-1 bash"
alias dep="sudo docker exec -it prak-dis-dec-postgres_primary-1 bash"
alias dlr="sudo docker logs prak-dis-dec-postgres_replica-1"
alias dlp="sudo docker logs prak-dis-dec-postgres_primary-1"
```

### 3. Panduan Pengujian & High-Availability (HA)
1. Aktifkan environment alias dengan perintah:
   ```bash
   source env.sh
   ```
2. Jalankan layanan docker:
   ```bash
   dcu
   ```
3. Periksa status kesehatan container:
   ```bash
   dps
   ```
4. **Pengujian Replikasi di Primary (`dep`):**
   ```sql
   psql -U zuser -d zdb
   \x
   SELECT * FROM pg_stat_replication;
   SELECT * FROM pg_replication_slots;
   ```
5. **Pengujian Pemulihan di Replica (`der`):**
   ```sql
   psql -U zuser -d zdb
   SELECT pg_is_in_recovery(); -- Harus bernilai 't' (true)
   ```
6. **Simulasi Failover (High Availability):**
   Jika server primary mati, lakukan promosi pada server replica:
   ```sql
   SELECT pg_promote();
   SELECT pg_is_in_recovery(); -- Berubah menjadi 'f' (false), server kini menjadi primary read-write.
   ```

---

## 4.2 Replikasi Master-Master Menggunakan Apache Ignite 3

Materi ini mengonfigurasi kluster *in-memory data grid* berarsitektur **Master-Master** menggunakan **Apache Ignite 3.1.0** dengan 3 node yang masing-masing memiliki kapabilitas *read-write*.

### 1. Struktur Direktori dan File
```text
ignite-cluster/
├── docker-compose.yaml
└── env.sh
```

### 2. Kode Sumber Konfigurasi

#### A. `docker-compose.yaml`
```yaml
version: '3.8'

services:
  node1:
    image: apacheignite/ignite:3.1.0
    container_name: ignite3-node1-1
    environment:
      - IGNITE_CLUSTER_NAME=ignite3
      - IGNITE_NODES=node1:10800,node2:10800,node3:10800
    ports:
      - "10300:10300"
      - "10800:10800"
    networks:
      - ignite3_default

  node2:
    image: apacheignite/ignite:3.1.0
    container_name: ignite3-node2-1
    environment:
      - IGNITE_CLUSTER_NAME=ignite3
      - IGNITE_NODES=node1:10800,node2:10800,node3:10800
    ports:
      - "10301:10300"
      - "10801:10800"
    networks:
      - ignite3_default

  node3:
    image: apacheignite/ignite:3.1.0
    container_name: ignite3-node3-1
    environment:
      - IGNITE_CLUSTER_NAME=ignite3
      - IGNITE_NODES=node1:10800,node2:10800,node3:10800
    ports:
      - "10302:10300"
      - "10802:10800"
    networks:
      - ignite3_default

networks:
  ignite3_default:
    driver: bridge
```

#### B. `env.sh`
```bash
alias dcu="sudo docker-compose up -d"
alias dcd="sudo docker-compose down"
alias dps="sudo docker ps"
alias dclm="sudo docker run --rm -it --network=host -e LANG=C.UTF-8 -e LC_ALL=C.UTF-8 -v ./sql:/opt/ignite/sql apacheignite/ignite:3.1.0 cli"
alias dcli="sudo docker run --rm -it --network=host -e LANG=C.UTF-8 -e LC_ALL=C.UTF-8 apacheignite/ignite:3.1.0 cli"
```

### 3. Panduan Pengujian Kluster
1. Jalankan kluster:
   ```bash
   dcu
   ```
2. Masuk ke Ignite CLI:
   ```bash
   dcli
   ```
3. Inisialisasi kluster pertama kali pada prompt CLI:
   ```text
   cluster init --name=ignite3 --metastorage-group=node1,node2,node3
   ```
4. Periksa status kluster:
   ```text
   cluster status
   ```
5. Masuk ke mode SQL dan verifikasi tabel atau lakukan eksekusi file SQL yang telah di-mount (`/opt/ignite/sql/`):
   ```text
   [node1]> sql
   sql-cli> SELECT * FROM system.tables WHERE schema = 'PUBLIC';
   ```
6. Hubungkan ke node lain (misalnya port `10301` atau `10302`) untuk memastikan replikasi Master-Master berjalan sempurna:
   ```text
   [node1]> connect http://localhost:10301
   [node2]> sql
   sql-cli> SELECT * FROM system.tables WHERE schema = 'PUBLIC';
   ```
