# LAPORAN PRAKTIKUM
## SISTEM TERDISTRIBUSI DAN TERDESENTRALISASI
### MODUL 2 — KOMUNIKASI ANTAR PROSES PADA SISTEM TERDISTRIBUSI

**Nama:** Francisco Guinaldo I dc Corbafo  
**NIM:** 255410003  
**Program Studi:** Informatika  
**Mata Kuliah:** Praktikum Sistem Terdistribusi dan Terdesentralisasi   
**Tanggal Praktikum:** 14 0ctober 2026

---

## 1. TUJUAN PRAKTIKUM

1. Memahami proses yang berjalan pada sistem operasi komputer.
2. Mengamati proses yang muncul ketika suatu aplikasi dijalankan.
3. Mengetahui cara melakukan restart dan menghentikan proses melalui sistem operasi.
4. Memahami komunikasi antarproses pada sistem terdistribusi menggunakan GraphQL.
5. Membuat program client yang dapat mengakses GraphQL server dan menerima data dari server.

## 2. DASAR TEORI

### 2.1 Proses

Proses merupakan hasil eksekusi program atau aplikasi yang bersifat executable. Proses dikelola oleh sistem operasi dan terdiri atas kode program, data, sumber daya, serta informasi keadaan seperti stack dan heap. Setiap aplikasi yang dijalankan akan menjadi proses yang dikelola oleh sistem operasi.

### 2.2 Proses pada Satu Node

Pada satu node, sistem operasi mengatur eksekusi program, alokasi sumber daya, pengelolaan proses, dan komunikasi antarproses. Pengguna dapat melihat proses yang sedang berjalan melalui Task Manager pada Windows atau menggunakan aplikasi seperti `htop` pada Linux.

### 2.3 Komunikasi Antarproses pada Sistem Terdistribusi

Dalam sistem terdistribusi, proses dapat berjalan pada node yang berbeda. Setiap node memiliki memori yang dikelola secara mandiri dan tidak dapat menggunakan shared memory secara langsung seperti proses pada satu komputer. Selain itu, node yang berbeda tidak selalu menggunakan clock yang sama. Oleh karena itu, diperlukan mekanisme komunikasi untuk bertukar data antarproses.

### 2.4 GraphQL

GraphQL merupakan teknologi yang memungkinkan client meminta data melalui query kepada server. Pada praktikum ini, GraphQL server dibuat menggunakan bahasa Python dengan pustaka Strawberry GraphQL. Client mengirimkan permintaan kepada server, kemudian server memproses query dan mengembalikan hasilnya.

## 3. ALAT DAN BAHAN

1. Laptop atau komputer.
2. Sistem operasi Windows atau Linux.
3. Python sesuai kebutuhan praktikum.
4. `uv` untuk pengelolaan lingkungan Python.
5. Pustaka `strawberry-graphql[cli]`.
6. Browser untuk mengakses GraphQL.
7. Text editor atau Visual Studio Code.
8. Terminal atau Command Prompt.

## 4. LANGKAH KERJA DAN HASIL PRAKTIKUM

### 4.1 Mengamati Proses pada Komputer

Langkah-langkah:

1. Membuka Task Manager dengan menekan `Ctrl + Shift + Esc`.
2. Memilih bagian Processes untuk melihat aplikasi dan proses yang sedang berjalan.
3. Mengamati nama proses serta penggunaan sumber daya komputer.
4. Mencatat beberapa proses yang ditemukan.

Contoh tabel pengamatan:

| No. | Nama Proses | Keterangan |
|---|---|---|
| 1 | Google Chrome | Proses aplikasi browser. |
| 2 | File Explorer | Proses pengelolaan file pada Windows. |
| 3 | Visual Studio Code | Proses aplikasi editor kode jika sedang dijalankan. |
| 4 | [Nama proses] | [Keterangan hasil pengamatan]. |

**Dokumentasi:** Tambahkan screenshot Task Manager yang menunjukkan proses pada komputer.

### 4.2 Mengamati Proses dari Aplikasi yang Dijalankan

Langkah-langkah:

1. Menjalankan aplikasi Notepad.
2. Membuka Task Manager.
3. Mencari proses Notepad pada daftar proses.
4. Mengamati perubahan proses setelah aplikasi dijalankan.

**Hasil pengamatan:** Setelah aplikasi Notepad dijalankan, proses yang berkaitan dengan aplikasi tersebut dapat diamati melalui Task Manager. Hal ini menunjukkan bahwa sistem operasi mengelola aplikasi yang sedang dieksekusi sebagai proses.

**Dokumentasi:** Tambahkan screenshot Notepad dan prosesnya pada Task Manager.

### 4.3 Melakukan Restart dan Menghentikan Proses

#### A. Restart Proses

1. Membuka Task Manager.
2. Memilih proses aplikasi yang sedang berjalan.
3. Menggunakan opsi Restart jika tersedia untuk proses tersebut.
4. Jika opsi Restart tidak tersedia, aplikasi dapat ditutup dan dijalankan kembali sebagai alternatif untuk mengamati proses baru.

#### B. Menghentikan Proses

1. Menjalankan aplikasi Notepad.
2. Membuka Task Manager.
3. Memilih proses Notepad.
4. Menekan tombol **End task**.
5. Memastikan aplikasi berhenti berjalan.

**Hasil pengamatan:** Perintah End task digunakan untuk menghentikan proses aplikasi melalui sistem operasi tanpa menggunakan menu keluar aplikasi. Penghentian proses harus dilakukan secara hati-hati agar pekerjaan yang belum disimpan tidak hilang.

**Dokumentasi:** Tambahkan screenshot sebelum dan sesudah proses dihentikan.

### 4.4 Membuat Workspace Python

Workspace dibuat dengan nama `workspace-01` sesuai petunjuk modul.

Perintah pada Command Prompt:

```bash
mkdir workspace-01
cd workspace-01
uv venv
.venv\Scripts\activate
```

Setelah lingkungan Python aktif, instal pustaka Strawberry GraphQL dengan perintah berikut:

```bash
uv pip install "strawberry-graphql[cli]"
```

Periksa versi Python menggunakan perintah:

```bash
python --version
```

Modul mengarahkan penggunaan Python versi 3.14. Penyiapan versi dan lingkungan mengikuti kondisi serta petunjuk praktikum yang digunakan.

### 4.5 Menjalankan GraphQL Server

Modul mengarahkan mahasiswa untuk menjalankan source code `schema.py` yang telah disediakan, kemudian membuka GraphQL melalui browser pada alamat:

```text
http://127.0.0.1:8000/graphql
```

Alamat `127.0.0.1` dapat diganti dengan `localhost`. Server harus berjalan agar halaman GraphQL dapat diakses.

Apabila file `schema.py` dari dosen tersedia, gunakan file tersebut. Jika file tersebut tidak tersedia, contoh server sederhana berikut dapat digunakan untuk menguji komunikasi client-server.

Isi file `schema.py`:

```python
import strawberry
from strawberry.asgi import GraphQL
from starlette.applications import Starlette
from starlette.routing import Route

@strawberry.type
class Book:
    title: str
    author: str

books_data = [
    Book(
        title="Bumi Manusia",
        author="Pramoedya Ananta Toer"
    ),
    Book(
        title="Laskar Pelangi",
        author="Andrea Hirata"
    ),
]

@strawberry.type
class Query:
    @strawberry.field
    def books(self) -> list[Book]:
        return books_data

schema = strawberry.Schema(query=Query)
graphql_app = GraphQL(schema)

app = Starlette(
    routes=[
        Route("/graphql", graphql_app),
    ]
)
```

Jalankan server dengan perintah:

```bash
uv run uvicorn schema:app --host 127.0.0.1 --port 8000
```

Setelah server berjalan, buka browser dan akses alamat GraphQL.

**Hasil praktikum:** Server GraphQL menyediakan endpoint untuk menerima permintaan query dari client.

**Dokumentasi:** Tambahkan screenshot terminal server dan halaman GraphQL di browser.

### 4.6 Mengirim Query GraphQL

Pada halaman GraphQL, masukkan query berikut:

```graphql
{
  books {
    title
    author
  }
}
```

Kemudian klik tombol Run untuk menjalankan query.

Query tersebut meminta server mengembalikan data buku yang berisi judul dan nama penulis.

Contoh hasil jika menggunakan data pada kode server di atas:

```json
{
  "data": {
    "books": [
      {
        "title": "Bumi Manusia",
        "author": "Pramoedya Ananta Toer"
      },
      {
        "title": "Laskar Pelangi",
        "author": "Andrea Hirata"
      }
    ]
  }
}
```

Hasil aktual mengikuti data yang tersedia pada file `schema.py` yang digunakan.

**Dokumentasi:** Tambahkan screenshot query dan hasil yang tampil pada browser.

### 4.7 Membuat Client untuk Mengakses GraphQL Server

Pada bagian tugas, mahasiswa diminta membuat client menggunakan bahasa pemrograman bebas untuk mengakses GraphQL server.

Pada praktikum ini, client dibuat menggunakan Python.

Buat file `client.py`:

```python
import json
from urllib.request import Request, urlopen
from urllib.error import HTTPError, URLError

URL = "http://127.0.0.1:8000/graphql"

query = """
{
    books {
        title
        author
    }
}
"""

payload = json.dumps({
    "query": query
}).encode("utf-8")

request = Request(
    URL,
    data=payload,
    headers={
        "Content-Type": "application/json"
    },
    method="POST"
)

try:
    with urlopen(request, timeout=10) as response:
        result = json.loads(
            response.read().decode("utf-8")
        )

        print(json.dumps(
            result,
            indent=2,
            ensure_ascii=False
        ))

except HTTPError as error:
    print(f"HTTP error: {error.code} - {error.reason}")

except URLError as error:
    print(f"Gagal terhubung ke server: {error.reason}")
```

Langkah menjalankan client:

1. Pastikan GraphQL server masih berjalan.
2. Buka terminal kedua pada folder `workspace-01`.
3. Aktifkan environment Python jika diperlukan.
4. Jalankan perintah:

```bash
python client.py
```

Program client mengirimkan query melalui HTTP POST ke endpoint GraphQL. Server kemudian memproses permintaan tersebut dan mengirimkan respons dalam format JSON.

**Hasil praktikum:** Jika server berjalan dan query berhasil diproses, client akan menampilkan data buku yang dikirimkan oleh server.

**Dokumentasi:** Tambahkan screenshot program `client.py` dan hasil eksekusinya.

### 4.8 Menghentikan GraphQL Server

Setelah praktikum selesai, buka terminal tempat GraphQL server berjalan dan tekan `Ctrl + C`.

Perintah tersebut menghentikan server yang sedang berjalan pada terminal tersebut.

## 5. PEMBAHASAN

Pada praktikum ini, proses aplikasi diamati melalui Task Manager untuk memahami bagaimana sistem operasi mengelola aplikasi yang sedang berjalan. Proses dapat dihentikan melalui perintah End task tanpa menggunakan menu keluar aplikasi. Selanjutnya, komunikasi antarproses dipelajari melalui GraphQL yang menggunakan mekanisme client-server. Client mengirimkan query kepada server melalui HTTP, kemudian server memproses permintaan dan mengembalikan data dalam format JSON. Pembuatan client menggunakan Python menunjukkan bahwa aplikasi dapat memperoleh data dari server melalui antarmuka komunikasi tanpa harus mengakses memori server secara langsung.

## 6. KESIMPULAN

Berdasarkan praktikum yang dilakukan, proses merupakan program yang sedang dieksekusi dan dikelola oleh sistem operasi. Proses dapat diamati melalui Task Manager serta dihentikan menggunakan fitur End task. Pada sistem terdistribusi, komunikasi antarproses dapat dilakukan melalui mekanisme client-server menggunakan GraphQL. Client mengirim query kepada server dan menerima respons berupa data JSON. Praktikum ini memberikan pemahaman mengenai pengelolaan proses dan komunikasi data antara aplikasi yang berbeda.

## 7. DOKUMENTASI

Tambahkan screenshot asli hasil praktikum pada folder `images` di repository GitHub.

Contoh penulisan gambar dalam Markdown:

**Gambar 1. Pengamatan proses komputer**

![Pengamatan proses komputer](images/task-manager.png)

**Gambar 2. Proses aplikasi yang dijalankan**

![Proses aplikasi](images/proses-aplikasi.png)

**Gambar 3. GraphQL Playground**

![GraphQL Playground](images/graphql-playground.png)

**Gambar 4. Hasil client Python**

![Hasil client Python](images/client-result.png)

*Catatan: Nama dan lokasi gambar di atas merupakan contoh. Sesuaikan dengan screenshot asli yang telah diunggah ke repository.*

## 8. REFERENSI

1. Bambang Purnomosidi D. P. *Modul 2 — Praktikum Sistem Terdistribusi dan Terdesentralisasi: Komunikasi Antar Proses pada Sistem Terdistribusi*. Universitas Teknologi Digital Indonesia.
2. Strawberry GraphQL. https://strawberry.rocks/
3. Catatan penggunaan `uv`. https://github.com/NEO-X-School/notes/blob/main/uv/00.md

