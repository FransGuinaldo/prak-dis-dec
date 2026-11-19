# Laporan Praktikum Modul 8
## Arsitektur Microservices untuk Sistem Terdistribusi

| | |
|---|---|
| **Mata Kuliah** | Sistem Terdistribusi dan Terdesentralisasi |
| **Program Studi** | Informatika, Universitas Teknologi Digital Indonesia |
| **Nama** | Francisco G I dc Corbafo |
| **NIM** | 255410003 |

---

## 1. Tujuan

1. Memahami arsitektur microservices, yaitu pemisahan frontend dan backend berupa service.
2. Membuat service berbasis REST API menggunakan FastAPI dan SQLModel (ORM Python).
3. Menyimpan data pada SQLite dan menampilkannya melalui endpoint REST API.
4. Menguji endpoint menggunakan `curl` dan browser.

## 2. Lingkungan Praktikum

| Komponen | Keterangan |
|---|---|
| Sistem operasi | Ubuntu (WSL2) di atas Windows, laptop Lenovo ThinkPad Intel Core i7 (x86-64) |
| Package manager Python | uv 0.13.9 |
| Python | 3.14.4 |
| Framework | FastAPI 0.165.0, SQLModel 0.0.50, SQLAlchemy 2.1.9 |
| Server ASGI | Uvicorn 0.54.0 |
| Database | SQLite 3.46.1 (dari repositori apt Ubuntu) |

> Catatan: Versi SQLite (3.46.1) lebih lama daripada contoh modul (3.53.1) dan jumlah paket Python berbeda (61 paket, di modul 45) karena versi lebih baru. Hal ini tidak memengaruhi hasil praktikum.

## 3. Persiapan Lingkungan

### 3.1 Update sistem dan install uv

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget unzip
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc
uv --version
```

### 3.2 Folder kerja, Python, dan virtual environment

```bash
mkdir -p ~/sistem-terdistribusi/08/src
cd ~/sistem-terdistribusi/08/src
uv python list
uv python install 3.14
uv python pin 3.14
uv venv
source .venv/bin/activate
```

Setelah environment aktif, prompt berubah menjadi `(src)`.

### 3.3 Install FastAPI dan SQLModel

```bash
uv pip install "fastapi[standard]" sqlmodel
```

Hasil: 61 paket terpasang, termasuk `fastapi`, `sqlmodel`, `sqlalchemy`, `pydantic`, dan `uvicorn`.

### 3.4 Install SQLite

```bash
sudo apt install -y sqlite3
sqlite3 --version
```

Hasil: `3.46.1 2024-08-13 ... (64-bit)`.

*(Sisipkan screenshot: `uv python list`, prompt `(src)`, hasil `uv pip install`, dan `sqlite3 --version`.)*

## 4. Bagian A: Contoh Modul (Service `sdm`)

### 4.1 Membuat database dan tabel

```bash
sqlite3 departemen-sdm.db
```

```sql
.mode box
CREATE TABLE sdm (id INTEGER PRIMARY KEY, npp CHAR(6), nama VARCHAR(50));
INSERT INTO sdm (npp, nama) VALUES('112233', 'Karyawan 1');
INSERT INTO sdm (npp, nama) VALUES('223344', 'Karyawan 2');
INSERT INTO sdm (npp, nama) VALUES('334455', 'Karyawan 3');
SELECT * FROM sdm;
```

Hasil:

```
┌────┬────────┬────────────┐
│ id │  npp   │    nama    │
├────┼────────┼────────────┤
│ 1  │ 112233 │ Karyawan 1 │
│ 2  │ 223344 │ Karyawan 2 │
│ 3  │ 334455 │ Karyawan 3 │
└────┴────────┴────────────┘
```

Setelah keluar (`.quit`) dan membuka database kembali, data tetap ada. Ini membuktikan data tersimpan permanen pada file `departemen-sdm.db`.

> Kendala: Pada percobaan awal, seluruh perintah ditempel sekaligus sehingga SQLite membaca `.mode box CREATE TABLE ...` sebagai satu perintah. Selain itu, `INSERT` sempat dijalankan dua kali sehingga data menjadi 6 baris. Data dobel dibersihkan dengan:
> ```bash
> sqlite3 departemen-sdm.db "DELETE FROM sdm WHERE id > 3;"
> ```

### 4.2 Source code `service.py`

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
    # SQLModel select statement
    statement = select(Sdm)
    results = session.exec(statement).all()
    return results
```

**Penjelasan:**

| Bagian | Fungsi |
|---|---|
| `class Sdm(SQLModel, table=True)` | Model ORM yang dipetakan ke tabel `sdm` |
| `Field(default=None, primary_key=True)` | `id` sebagai primary key yang terisi otomatis |
| `create_engine("sqlite:///departemen-sdm.db")` | Koneksi ke file SQLite |
| `get_session()` | Generator yang membuka dan menutup sesi database per request |
| `Depends(get_session)` | Dependency injection FastAPI |
| `@app.get("/sdm/")` | Endpoint `GET /sdm/` |
| `select(Sdm)` dan `.all()` | Mengambil seluruh baris tabel |

### 4.3 Menjalankan service dan pengujian

```bash
uvicorn service:app --reload &
curl http://localhost:8000/sdm/
```

Log server:

```
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Application startup complete.
INFO:     127.0.0.1:38266 - "GET /sdm/ HTTP/1.1" 200 OK
```

Hasil `curl`:

```json
[{"npp":"112233","nama":"Karyawan 1","id":1},{"npp":"223344","nama":"Karyawan 2","id":2},{"npp":"334455","nama":"Karyawan 3","id":3}]
```

Hasil ini sama dengan contoh pada modul.

*(Sisipkan screenshot: tabel `sdm`, `cat service.py`, uvicorn berjalan, hasil `curl`, dan tampilan browser `http://localhost:8000/sdm/`.)*

## 5. Bagian B: Tugas

### 5.1 Tugas 1: Membuat tabel baru di SQLite

Tabel `produk` pada database `toko.db`, terdiri atas satu primary key dan kolom bertipe INT, CHAR, VARCHAR, BOOLEAN, dan FLOAT.

```bash
sqlite3 toko.db
```

```sql
CREATE TABLE produk (id INTEGER PRIMARY KEY, stok INT, kode CHAR(6), nama VARCHAR(50), tersedia BOOLEAN, harga FLOAT);
.schema produk
```

| Kolom | Tipe | Keterangan |
|---|---|---|
| `id` | INTEGER PRIMARY KEY | Primary key |
| `stok` | INT | Jumlah stok |
| `kode` | CHAR(6) | Kode produk |
| `nama` | VARCHAR(50) | Nama produk |
| `tersedia` | BOOLEAN | Status ketersediaan |
| `harga` | FLOAT | Harga produk |

### 5.2 Tugas 2: Mengisi 5 data dengan script Python

File `seed.py`:

```python
from sqlmodel import Field, Session, SQLModel, create_engine


class Produk(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    stok: int
    kode: str
    nama: str
    tersedia: bool
    harga: float


engine = create_engine("sqlite:///toko.db")

data = [
    Produk(stok=10, kode="P00001", nama="Keyboard", tersedia=True, harga=150000.5),
    Produk(stok=0, kode="P00002", nama="Mouse", tersedia=False, harga=75000.0),
    Produk(stok=25, kode="P00003", nama="Monitor", tersedia=True, harga=1250000.0),
    Produk(stok=5, kode="P00004", nama="Webcam", tersedia=True, harga=320000.75),
    Produk(stok=8, kode="P00005", nama="Headset", tersedia=True, harga=210000.0),
]

with Session(engine) as session:
    session.add_all(data)
    session.commit()

print("5 data berhasil dimasukkan")
```

Eksekusi dan verifikasi:

```bash
python seed.py
sqlite3 toko.db ".mode box" "SELECT * FROM produk;"
```

```
5 data berhasil dimasukkan
┌────┬──────┬────────┬──────────┬──────────┬───────────┐
│ id │ stok │  kode  │   nama   │ tersedia │   harga   │
├────┼──────┼────────┼──────────┼──────────┼───────────┤
│ 1  │ 10   │ P00001 │ Keyboard │ 1        │ 150000.5  │
│ 2  │ 0    │ P00002 │ Mouse    │ 0        │ 75000.0   │
│ 3  │ 25   │ P00003 │ Monitor  │ 1        │ 1250000.0 │
│ 4  │ 5    │ P00004 │ Webcam   │ 1        │ 320000.75 │
│ 5  │ 8    │ P00005 │ Headset  │ 1        │ 210000.0  │
└────┴──────┴────────┴──────────┴──────────┴───────────┘
```

Di SQLite, `BOOLEAN` disimpan sebagai `1` (true) dan `0` (false).

### 5.3 Tugas 3: Membuat RESTful API endpoint

File `produk_service.py`:

```python
from typing import List
from fastapi import Depends, FastAPI
from sqlmodel import Field, Session, SQLModel, create_engine, select


class Produk(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    stok: int
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

Menjalankan service:

```bash
uvicorn produk_service:app --reload &
```

### 5.4 Tugas 4: Menampilkan hasil dengan curl

```bash
curl -s http://localhost:8000/produk/ | python -m json.tool
```

Hasil (log server: `GET /produk/ HTTP/1.1" 200 OK`):

```json
[
    {"tersedia": true,  "stok": 10, "kode": "P00001", "nama": "Keyboard", "id": 1, "harga": 150000.5},
    {"tersedia": false, "stok": 0,  "kode": "P00002", "nama": "Mouse",    "id": 2, "harga": 75000.0},
    {"tersedia": true,  "stok": 25, "kode": "P00003", "nama": "Monitor",  "id": 3, "harga": 1250000.0},
    {"tersedia": true,  "stok": 5,  "kode": "P00004", "nama": "Webcam",   "id": 4, "harga": 320000.75},
    {"tersedia": true,  "stok": 8,  "kode": "P00005", "nama": "Headset",  "id": 5, "harga": 210000.0}
]
```

Pada JSON, nilai `BOOLEAN` dikonversi SQLModel menjadi `true`/`false`. Seluruh 5 data tampil dengan tipe INT, CHAR, VARCHAR, BOOLEAN, dan FLOAT sesuai ketentuan tugas.

*(Sisipkan screenshot: `.schema produk`, `python seed.py` dan `SELECT`, `cat produk_service.py`, hasil `curl`, dan tampilan browser `http://localhost:8000/produk/`.)*

## 6. Tambahan (di luar tugas)

### 6.1 Dokumentasi otomatis Swagger UI

FastAPI menyediakan dokumentasi interaktif pada `http://localhost:8000/docs`. Halaman ini berhasil diakses (`GET /docs 200 OK`).

### 6.2 Halaman tabel HTML

File `produk_web.py` menambahkan endpoint `/tampil` yang menampilkan data produk sebagai tabel HTML, sedangkan endpoint `/produk/` tetap tersedia. Akses `GET /tampil` berhasil (`200 OK`).

### 6.3 Menjalankan dua service bersamaan

Dua service dapat aktif bersamaan dengan port berbeda:

```bash
uvicorn produk_web:app --reload --port 8000 &
uvicorn service:app --reload --port 8001 &
```

| Layanan | Alamat |
|---|---|
| produk (JSON) | `http://localhost:8000/produk/` |
| produk (tabel HTML) | `http://localhost:8000/tampil` |
| sdm (JSON) | `http://localhost:8001/sdm/` |

Menghentikan semua server: `pkill -f uvicorn`.

## 7. Kendala dan Solusi

| Kendala | Penyebab | Solusi |
|---|---|---|
| `extra argument: "TABLE"` di sqlite3 | Beberapa baris ditempel sekaligus, dan perintah titik (`.mode`) tidak diakhiri Enter | Ketik perintah satu per satu, atau gunakan heredoc dari terminal |
| Data `sdm` menjadi 6 baris | `INSERT` terjalankan dua kali | `DELETE FROM sdm WHERE id > 3;` |
| `SELECT: command not found` | Perintah SQL diketik di terminal biasa | Jalankan di dalam prompt `sqlite>` |
| `Address already in use` | Port 8000 masih dipakai server lama | `pkill -f uvicorn`, lalu jalankan ulang, atau gunakan `--port` lain |
| `Expecting value: line 1 column 1` pada `json.tool` | Server sudah berhenti sehingga `curl -s` tidak mengembalikan apa pun | Jalankan ulang server sebelum `curl` |
| `No such file or directory` saat mengetik URL | URL diketik di terminal | Buka URL di browser |

## 8. Kesimpulan

1. Lingkungan Ubuntu (WSL2) dengan uv, Python 3.14.4, FastAPI, SQLModel, dan SQLite berhasil disiapkan.
2. Service `sdm` pada contoh modul berjalan dan menghasilkan keluaran yang sama dengan modul.
3. Tugas berhasil diselesaikan: tabel `produk` (1 primary key, INT, CHAR, VARCHAR, BOOLEAN, FLOAT), 5 data diisi lewat script Python, endpoint `GET /produk/`, dan hasil ditampilkan menggunakan `curl`.
4. Arsitektur microservices memisahkan layanan data (backend) dari penyaji tampilan sehingga setiap service dapat dijalankan dan dikembangkan secara independen, seperti terlihat pada dua service yang berjalan bersamaan di port 8000 dan 8001.

## 9. Struktur Berkas

```
~/sistem-terdistribusi/08/src/
├── .python-version
├── .venv/
├── departemen-sdm.db
├── toko.db
├── service.py
├── seed.py
├── produk_service.py
└── produk_web.py
```
