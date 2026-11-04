# LAPORAN PRAKTIKUM SISTEM TERDISTRIBUSI DAN TERDESENTRALISASI
## MODUL 05: FAULT TOLERANCE

---

**Nama Mahasiswa** : [Francisco G I dc Corbafo]  
**NIM**            : [255410003]  
**Program Studi**   : [Informatika]  
**Dosen Pengampu**  : [Bambang Purnomosidi D.P., Dr., S.E., Ak., S.Kom., MMSI.]  
**Tanggal**         : 4 November 2026  

---

### I. TUJUAN PRAKTIKUM
1. Mengonfigurasi lingkungan pengembangan Python modern menggunakan pengelola paket `uv`.
2. Mengimplementasikan eksekusi *multi-instance scaling* untuk aplikasi web berbasis *framework* BlackSheep.
3. Membangun script pemantauan kesehatan server (*Health Check / Heartbeat*) untuk mendeteksi kegagalan instance (*Failure Detection*).
4. Membangun dan menguji *Load Balancer* mandiri berbasis algoritma *Round-Robin* beserta mekanisme *Fault Tolerance* sederhana.

---

### II. PERANGKAT & LINGKUNGAN KERJA
- **Sistem Operasi**: Windows 11 / Windows PowerShell
- **Bahasa Pemrograman**: Python 3.11.18
- **Package Manager**: `uv` (Astral)
- **Framework Web**: BlackSheep (`blacksheep==2.6.4`)
- **Server ASGI**: Uvicorn (`uvicorn==0.34.3`)

---



### III. LANGKAH KERJA & DOKUMENTASI EKSEKUSI

#### 1. Inisialisasi Environment & Instalasi Dependensi
Langkah awal dimulai dengan menginstal `uv`, mematok versi Python ke 3.11, serta mengonfigurasi *virtual environment*.

**Log Eksekusi Terminal:**
```powershell
PS C:\Users\kikuc> powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
downloading uv 0.13.7 (x86_64-pc-windows-msvc)
installing to C:\Users\kikuc\.local\bin                                                                                         
  uv.exe                                                                                                                        
  uvx.exe                                                                                                                       
  uvw.exe                                                                                                                       
everything's installed!                                                                                                         

PS C:\Users\kikuc> uv python pin 3.11
Pinned `.python-version` to `3.11`

PS C:\Users\kikuc> uv venv
Using CPython 3.11.18
Creating virtual environment at: .venv
Activate with: .venv\Scripts\activate

PS C:\Users\kikuc> .venv\Scripts\Activate
(kikuc) PS C:\Users\kikuc> uv pip install blacksheep uvicorn
Installed 16 packages in 85ms
 + blacksheep==2.6.4
 + uvicorn==0.54.0
```


<img width="1097" height="428" alt="image" src="https://github.com/user-attachments/assets/4fb24925-295c-4d4c-bfac-3735a48ddae1" />

<img width="1097" height="428" alt="image" src="https://github.com/user-attachments/assets/7a537ce8-e490-41b1-ad79-6b6451caa236" />




#### 2. Pembuatan Project dan Dependensi Template
Project baru `blacksheep_lb` dibuat menggunakan alat bantu `blacksheep-cli` dengan struktur *Model-View-Controller* (MVC).

**Log Eksekusi Terminal:**
```powershell
(kikuc) PS C:\Users\kikuc\workspace-05> uv pip install blacksheep-cli
(kikuc) PS C:\Users\kikuc\workspace-05> blacksheep create
✨ Project name: blacksheep_lb
🚀 Project template: mvc
📜 Use OpenAPI Documentation? Yes
🔧 Library to read settings essentials-configuration
🔩 App settings format TOML

🏗️  Project created in blacksheep_lb

(kikuc) PS C:\Users\kikuc\workspace-05\blacksheep_lb> uv pip install -r requirements.txt
```
<img width="780" height="619" alt="Screenshot 2026-11-04 092610" src="https://github.com/user-attachments/assets/f0a99b84-4e2e-4ca6-b19a-ce01c32a3898" />

<img width="659" height="525" alt="Screenshot 2026-11-04 092726" src="https://github.com/user-attachments/assets/2bcf3d77-27dc-47c5-a3af-ec2e2479f01b" />



#### 3. Pengujian Multi-Instance Scaling
Aplikasi dijalankan dalam 2 instance terpisah untuk mensimulasikan lingkungan terdistribusi (*cluster*):
- **Instance 1**: Berjalan pada `http://localhost:44777`
  <img width="1179" height="789" alt="Screenshot 2026-11-04 093342" src="https://github.com/user-attachments/assets/30dff545-8065-445f-ba26-c959f356d61e" />


- **Instance 2**: Berjalan pada `http://localhost:44778`
<img width="1062" height="698" alt="Screenshot 2026-11-04 093413" src="https://github.com/user-attachments/assets/b970598e-79f9-4a42-ae95-8b87f0a01650" />




**Log Eksekusi Server (Instance 1):**
```powershell
(kikuc) PS C:\Users\kikuc\workspace-05\blacksheep_lb> python dev.py
Running for local development ─────────────────────────────────────────────────────────────────────────────────────────────────────────────────
Visit http://localhost:44777/

INFO:     Started server process [4560]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://localhost:44777 (Press CTRL+C to quit)
INFO:     ::1:51642 - "GET / HTTP/1.1" 200 OK
INFO:     ::1:53491 - "GET /styles/public.css HTTP/1.1" 200 OK
```




---

### IV. IMPLEMENTASI SKRIP & HASIL PENGUJIANKU

#### 1. Script Failure Detection (`health_check.py`)
Script ini bertugas melakukan *HTTP ping* berkala ke instance backend untuk memastikan ketersediaan server.

**Kode `health_check.py`:**
```python
import urllib.request
import urllib.error

INSTANCES = [
    "http://localhost:44777",
    "http://localhost:44778"
]

def check_servers():
    print("Memulai pemeriksaan kesehatan server (Health Check)...\n")
    for url in INSTANCES:
        try:
            response = urllib.request.urlopen(url, timeout=2)
            if response.status == 200:
                print(f"[ONLINE] Server {url} berjalan normal (Status: 200 OK)")
            else:
                print(f"[WARNING] Server {url} merespons dengan status {response.status}")
        except urllib.error.URLError as e:
            print(f"[DOWN] Server {url} gagal diakses! Alasan: {e.reason}")
        except Exception as e:
            print(f"[ERROR] Terjadi kesalahan pada {url}: {e}")

if __name__ == "__main__":
    check_servers()
```

**Hasil Pengujian Terminal:**
```powershell
PS C:\Users\kikuc> python health_check.py
Memulai pemeriksaan kesehatan server (Health Check)...

[ONLINE] Server http://localhost:44777 berjalan normal (Status: 200 OK)
[ONLINE] Server http://localhost:44778 berjalan normal (Status: 200 OK)
```
<img width="1048" height="658" alt="Screenshot 2026-11-04 213628" src="https://github.com/user-attachments/assets/4d09dc1e-2fce-4ef4-a70a-975a45d53f20" />
<img width="803" height="159" alt="Screenshot 2026-11-04 213653" src="https://github.com/user-attachments/assets/e914d970-84ee-4a3e-982f-c65dd53591fb" />



---

#### 2. Script Load Balancer & Fault Tolerance (`load_balancer.py`)
Load balancer membagi beban trafik secara bergantian (*Round-Robin*) melalui port `8000` dan mengabaikan backend yang mengalami kemacetan/mati.

**Kode `load_balancer.py`:**
```python
import urllib.request
import urllib.error
from blacksheep import Application, Request, Response
from blacksheep.server.responses import text

app = Application()

BACKENDS = [
    "http://localhost:44777",
    "http://localhost:44778"
]

current_backend_index = 0

def get_next_backend():
    global current_backend_index
    backend = BACKENDS[current_backend_index]
    current_backend_index = (current_backend_index + 1) % len(BACKENDS)
    return backend

@app.router.route("/favicon.ico")
async def favicon(request: Request):
    return Response(204, None, None)

@app.router.route("/{path:path}")
@app.router.route("/")
async def proxy(request: Request, path: str = ""):
    global BACKENDS
    
    attempts = len(BACKENDS)
    target_backend = None
    
    for _ in range(attempts):
        backend = get_next_backend()
        try:
            req = urllib.request.urlopen(backend, timeout=1)
            if req.status == 200:
                target_backend = backend
                break
        except:
            continue
            
    if not target_backend:
        return text("Semua backend server sedang DOWN!", status=503)

    target_url = f"{target_backend}/{path}" if path else f"{target_backend}/"
    
    try:
        req = urllib.request.urlopen(target_url)
        content = req.read().decode('utf-8')
        return text(f"[LOAD BALANCER] Diteruskan ke backend: {target_backend}\n\nRespon:\n{content}")
    except urllib.error.HTTPError as e:
        return text(f"Backend merespons dengan error HTTP: {e.code}", status=e.code)
    except Exception as e:
        return text(f"Gagal menghubungi backend: {str(e)}", status=502)

if __name__ == "__main__":
    import uvicorn
    print("Memulai Load Balancer di http://localhost:8000 ...")
    uvicorn.run(app, host="127.0.0.1", port=8000)
```

**Hasil Pengujian Terminal (`load_balancer.py`):**
```powershell
PS C:\Users\kikuc\workspace-05\blacksheep_lb> ..\..\.venv\Scripts\python.exe load_balancer.py
Memulai Load Balancer di http://localhost:8000 ...
INFO:     Started server process [9760]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     127.0.0.1:55344 - "GET / HTTP/1.1" 200 OK
INFO:     127.0.0.1:55344 - "GET /favicon.ico HTTP/1.1" 204 No Content
INFO:     127.0.0.1:55344 - "GET / HTTP/1.1" 200 OK
INFO:     127.0.0.1:55344 - "GET /favicon.ico HTTP/1.1" 204 No Content
```

---

### V. KESIMPULAN
1. **Multi-Instance Scaling**: Berhasil memisahkan dan menjalankan dua instance independen pada port `44777` dan `44778`.
2. **Failure Detection**: Script `health_check.py` secara akurat mampu mengidentifikasi status aktif (`200 OK`) maupun status mati dari setiap instance backend.
3. **Load Balancing & Fault Tolerance**: Berhasil membagi beban trafik secara seimbang (*Round-Robin*) melalui pintu gerbang port `8000`, serta mampu melakukan pengalihan otomatis jika terjadi kendala pada salah satu node backend.
