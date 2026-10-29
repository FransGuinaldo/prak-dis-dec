# LAPORAN PRAKTIKUM SISTEM TERDISTRIBUSI DAN TERDESENTRALISASI

**Mata Kuliah:** Sistem Terdistribusi dan Terdesentralisasi  
**Modul 4:** Konsistensi dan Replikasi pada Sistem Terdistribusi  
**Dosen Pengampu:** Dr. Bambang Purnomosidi D. P.[cite: 1]  
**Program Studi:** Informatika, Universitas Teknologi Digital Indonesia[cite: 1]  

---

## I. PENDAHULUAN

### 1.1 Latar Belakang
Replikasi data merupakan teknik fundamental dalam sistem terdistribusi untuk menjamin ketersediaan tinggi (*high availability*), keandalan data (*fault tolerance*), serta peningkatan kinerja pembacaan data melalui pembagian beban (*load balancing*)[cite: 1]. Praktikum ini berfokus pada dua arsitektur replikasi utama:

1. **Single-Primary Replication (Master-Slave):** Menggunakan PostgreSQL 18 di mana *primary node* menangani operasi *Read-Write* dan direplikasi secara *streaming* ke *replica node* (*Read-Only*)[cite: 1].
2. **Multi-Master Replication:** Menggunakan Apache Ignite 3.1.0 di mana seluruh *node* bersifat setara dan mendukung operasi *Read-Write* yang tersinkronisasi di seluruh *cluster*.

---

## II. STREAMING REPLICATION MENGGUNAKAN POSTGRESQL 18

### 2.1 Konfigurasi File Lingkungan

Untuk menjalankan *streaming replication* pada Docker, disiapkan 3 file utama di dalam direktori `streaming-replication/`[cite: 1]:

#### 1. `00_init.sql`
Digunakan untuk inisialisasi akun pengguna replikasi dan pembuatan *physical replication slot*.

```sql
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'replicator_password';
SELECT pg_create_physical_replication_slot('replication_slot');
