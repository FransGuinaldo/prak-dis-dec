# Laporan Praktikum Modul 8
## Arsitektur Microservices untuk Sistem Terdistribusi

| | |
|---|---|
| **Mata Kuliah** | Sistem Terdistribusi dan Terdesentralisasi |
| **Program Studi** | Informatika, Universitas Teknologi Digital Indonesia |
| **Nama** | Francisco G I dc Corbafo |
| **NIM** | 255410003 |

---

## 1. Tujuan Praktikum
1. Menyiapkan lingkungan pengembangan Python modern menggunakan `uv` serta basis data SQLite3 pada sistem operasi Linux.
2. Membangun layanan web (REST API) menggunakan **FastAPI** dan **SQLModel** sebagai ORM (*Object-Relational Mapping*).
3. Melakukan operasi *seeding* data, pengujian *endpoint* API, serta manajemen proses *server* Uvicorn.

---

## 2. Persiapan Lingkungan Pengembangan

### 2.1. Instalasi Paket Sistem (`curl` & `sqlite3`)
Memastikan utilitas `curl` dan `sqlite3` terpasang pada sistem Ubuntu:
```bash
sudo apt update && sudo apt install -y curl sqlite3
sqlite3 --version
```
* **Hasil:** `sqlite3` versi `3.46.1` dan `curl` versi `8.18.0` telah terpasang dan siap digunakan.

### 2.2. Instalasi `uv` dan Konfigurasi Virtual Environment
Menginstal manajer paket `uv`, menyiapkan direktori kerja `~/modul08/src`, serta mengaktifkan *virtual environment* dengan Python `3.14.4`:
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env
uv --version

mkdir -p ~/modul08/src && cd ~/modul08/src
uv python install 3.14.4
uv python pin 3.14.4
uv venv
source .venv/bin/activate
python --version
```
* **Hasil:** `uv 0.13.9` dan `Python 3.14.4` berhasil dikonfigurasi di dalam `.venv`.

### 2.3. Instalasi Dependensi Proyek
Menginstal pustaka `fastapi[standard]` dan `sqlmodel` ke dalam lingkungan virtual:
```bash
uv pip install "fastapi[standard]" sqlmodel
```
* **Hasil:** Sebanyak 61 paket berhasil diinstal (termasuk `fastapi==0.165.0`, `sqlmodel==0.0.50`, `sqlalchemy==2.1.9`, dan `uvicorn==0.54.0`).

---

## 3. Studi Kasus 1: Layanan Data Produk (`toko.db`)

### 3.1. Pembuatan Skema Basis Data
Membuat berkas basis data `toko.db` dengan tabel `produk`:
```bash
sqlite3 toko.db "CREATE TABLE produk (
  id INTEGER PRIMARY KEY,
  kode CHAR(6),
  nama VARCHAR(50),
  tersedia BOOLEAN,
  harga FLOAT
);"
sqlite3 toko.db ".schema produk"
```

### 3.2. Implementasi API (`service.py`)
Membuat layanan API untuk membaca daftar produk dari `toko.db` melalui *endpoint* `GET /produk/`:
```python
from typing import List
from fastapi import Depends, FastAPI
from sqlmodel import Field, Session, SQLModel, create_engine, select


class Produk(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    kode: str
    nama: str
    tersedia: bool
    harga: float


engine = create_engine("sqlite:///toko.db")


def get_session():
    with Session(engine) as session:
        yield session


app = FastAPI()


@app.get("/produk/", response_model=List[Produk])
def read_produk(session: Session = Depends(get_session)):
    statement = select(Produk)
    results = session.exec(statement).all()
    return results
```

### 3.3. Pengisian Data Awal / Seeding (`seed.py`)
Membuat skrip `seed.py` untuk memasukkan 5 data produk awal ke dalam basis data:
```python
from sqlmodel import Session
from service import Produk, engine

data = [
    Produk(kode="P00001", nama="Pensil 2B",  tersedia=True,  harga=3500.0),
    Produk(kode="P00002", nama="Buku Tulis", tersedia=True,  harga=8000.5),
    Produk(kode="P00003", nama="Penggaris",  tersedia=False, harga=5000.0),
    Produk(kode="P00004", nama="Spidol",     tersedia=True,  harga=12500.75),
    Produk(kode="P00005", nama="Penghapus",  tersedia=False, harga=2000.0),
]

with Session(engine) as session:
    session.add_all(data)
    session.commit()
    print(f"Berhasil menambahkan {len(data)} data")
```

Verifikasi isi tabel menggunakan perintah `sqlite3`:
```text
┌────┬────────┬────────────┬──────────┬──────────┐
│ id │  kode  │    nama    │ tersedia │  harga   │
├────┼────────┼────────────┼──────────┼──────────┤
│ 1  │ P00001 │ Pensil 2B  │ 1        │ 3500.0   │
│ 2  │ P00002 │ Buku Tulis │ 1        │ 8000.5   │
│ 3  │ P00003 │ Penggaris  │ 0        │ 5000.0   │
│ 4  │ P00004 │ Spidol     │ 1        │ 12500.75 │
│ 5  │ P00005 │ Penghapus  │ 0        │ 2000.0   │
└────┴────────┴────────────┴──────────┴──────────┘
```

### 3.4. Pengujian Endpoint Produk
Menjalankan server di latar belakang dan mengakses *endpoint* menggunakan `curl`:
```bash
uvicorn service:app --port 8000 &
sleep 2
curl http://localhost:8000/produk/
```
* **Respons HTTP (`200 OK`):**
```json
[
  {"harga":3500.0,"kode":"P00001","nama":"Pensil 2B","tersedia":true,"id":1},
  {"harga":8000.5,"kode":"P00002","nama":"Buku Tulis","tersedia":true,"id":2},
  {"harga":5000.0,"kode":"P00003","nama":"Penggaris","tersedia":false,"id":3},
  {"harga":12500.75,"kode":"P00004","nama":"Spidol","tersedia":true,"id":4},
  {"harga":2000.0,"kode":"P00005","nama":"Penghapus","tersedia":false,"id":5}
]
```

---

## 4. Studi Kasus 2: Layanan Departemen SDM (`departemen-sdm.db`)

### 4.1. Pembuatan Basis Data dan Data Karyawan
Membuat basis data baru bernama `departemen-sdm.db` beserta tabel `sdm` dan 3 baris data karyawan:
```bash
sqlite3 departemen-sdm.db "
CREATE TABLE sdm (id INTEGER PRIMARY KEY, npp CHAR(6), nama VARCHAR(50));
INSERT INTO sdm (npp, nama) VALUES('112233', 'Karyawan 1');
INSERT INTO sdm (npp, nama) VALUES('223344', 'Karyawan 2');
INSERT INTO sdm (npp, nama) VALUES('334455', 'Karyawan 3');
"
sqlite3 departemen-sdm.db ".mode box" "SELECT * FROM sdm;"
```
* **Hasil Tabel `sdm`:**
```text
┌────┬────────┬────────────┐
│ id │  npp   │    nama    │
├────┼────────┼────────────┤
│ 1  │ 112233 │ Karyawan 1 │
│ 2  │ 223344 │ Karyawan 2 │
│ 3  │ 334455 │ Karyawan 3 │
└────┴────────┴────────────┘
```

### 4.2. Implementasi API SDM (`sdm_service.py`)
```python
from typing import List, Optional
from fastapi import Depends, FastAPI, HTTPException
from sqlmodel import Field, Session, SQLModel, create_engine, select


class Sdm(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    npp: str
    nama: str


engine = create_engine("sqlite:///departemen-sdm.db")


def get_session():
    with Session(engine) as session:
        yield session


app = FastAPI()


@app.get("/sdm/", response_model=List[Sdm])
def read_sdm(session: Session = Depends(get_session)):
    statement = select(Sdm)
    results = session.exec(statement).all()
    return results
```

### 4.3. Penanganan Konflik Port & Pengujian Endpoint SDM
1. **Kendala (`Errno 98: Address already in use`):** Saat menjalankan `sdm_service:app`, terjadi bentrok port `8000` karena proses Uvicorn sebelumnya (`service:app`, PID `1100`) masih berjalan di latar belakang, sehingga permintaan ke `/sdm/` menghasilkan `404 Not Found`.
2. **Penyelesaian:** Menghentikan proses PID `1100` dengan `kill 1100` dan memverifikasi ketersediaan port menggunakan `ss -ltn | grep 8000`.
3. **Pengujian Ulang:**
```bash
uvicorn sdm_service:app --reload &
sleep 2
curl http://localhost:8000/sdm/
```
* **Respons HTTP (`200 OK`):**
```json
[
  {"id":1,"npp":"112233","nama":"Karyawan 1"},
  {"id":2,"npp":"223344","nama":"Karyawan 2"},
  {"id":3,"npp":"334455","nama":"Karyawan 3"}
]
```

---

## 5. Pembersihan Lingkungan Kerja
Setelah seluruh pengujian selesai, layanan Uvicorn dihentikan secara menyeluruh dan berkas basis data sementara `toko.db` dihapus:
```bash
pkill -f uvicorn
rm -f toko.db
```

---

## 6. Kesimpulan
Praktikum berhasil mendemonstrasikan integrasi antara **FastAPI**, **SQLModel**, dan **SQLite3** dalam menyediakan REST API berbasis data relasional. Selain itu, pengelolaan proses latar belakang pada Linux (*background jobs*) terbukti krusial untuk mencegah konflik *port* antarlayanan yang berjalan pada jaringan lokal.
