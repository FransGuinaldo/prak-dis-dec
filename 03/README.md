# LAPORAN PRAKTIKUM MODUL 3
## Sinkronisasi pada Sistem Terdistribusi

**Nama:** Francisco G I dc Corbafo

**NIM:** 255410003  
**Kelas:** IF-1  
**Mata Kuliah:** Praktikum Sistem Terdistribusi dan Terdesentralisasi  
**Tanggal Praktikum:** 21 Oktober 2026  

---

## 3.1 Sinkronisasi Waktu

### Tujuan
Memahami proses sinkronisasi waktu komputer menggunakan protokol NTP (*Network Time Protocol*).

### Langkah Praktikum
1. Mengunduh dan memasang NetTime dari [https://www.timesynctool.com/](https://www.timesynctool.com/).
2. Membuka NetTime.
3. Memastikan server NTP tersedia pada pengaturan aplikasi.
4. Menjalankan proses sinkronisasi waktu.
5. Mengamati hasil sinkronisasi dan mencatat pesan yang ditampilkan aplikasi.

### Hasil dan Pembahasan
<img width="878" height="633" alt="nettime png" src="https://github.com/user-attachments/assets/80655a6c-fc73-402f-b7b2-b34b05efcd0a" />


NetTime menyelaraskan waktu komputer dengan waktu dari server NTP melalui jaringan. Komputer meminta informasi waktu dari server, kemudian aplikasi menyesuaikan waktu komputer berdasarkan hasil tersebut. Sinkronisasi waktu penting agar pencatatan waktu pada beberapa komputer lebih konsisten.

**Catatan:** Sesuaikan pembahasan ini dengan hasil yang benar-benar ditampilkan NetTime. Jangan menyatakan sinkronisasi berhasil sebelum prosesnya dijalankan.

---

## 3.2 Vector Clock

### Tujuan
Memahami penggunaan *vector clock* untuk membandingkan urutan kejadian (*event*) dalam sistem terdistribusi.

### Langkah Praktikum
1. Buka terminal di folder source code.
2. Jalankan perintah:

   ```powershell
   python vclocks.py
   ```

3. Amati dan simpan keluaran program.
4. Bandingkan nilai vector clock pada keluaran program dengan perhitungan manual.
5. Buat modul Python yang berisi kelas `VectorClock`, lalu buat program contoh yang menggunakan modul tersebut.

### Hasil Program
<img width="563" height="852" alt="image" src="https://github.com/user-attachments/assets/9fa2c8b6-c1dd-4e2a-a42e-da2f7b458058" /><img width="448" height="535" alt="image" src="https://github.com/user-attachments/assets/7ffebc52-3d48-4465-8688-7f8e28c904b3" /><img width="458" height="204" alt="image" src="https://github.com/user-attachments/assets/f399e4d3-48d3-4655-91de-2727e4c5e43b" />
### Pembahasan
Vector clock menyimpan penghitung untuk setiap proses. Secara umum, ketika suatu proses melakukan kejadian lokal, nilai penghitung milik proses tersebut bertambah. Saat mengirim pesan, proses menyertakan vector clock-nya. Ketika menerima pesan, proses menggabungkan informasi clock yang diterima dengan clock miliknya menggunakan nilai maksimum pada setiap posisi, lalu menambah penghitung proses penerima sesuai aturan algoritmanya.

Untuk membandingkan hasil program dengan cara manual:
1. Catat nilai awal vector clock setiap proses.
2. Ikuti urutan kejadian pada source code.
3. Perbarui nilai clock pada setiap kejadian lokal, pengiriman pesan, dan penerimaan pesan.
4. Bandingkan nilai akhir per kejadian dengan keluaran program.

Dua kejadian dapat dibandingkan berdasarkan nilai vector clock-nya. Jika semua komponen clock A kurang dari atau sama dengan clock B dan setidaknya satu komponen lebih kecil, A terjadi sebelum B. Jika tidak ada hubungan seperti itu, kedua kejadian bersifat konkuren.

### Modul `VectorClock` dan Contoh Penggunaan
<img width="645" height="476" alt="image" src="https://github.com/user-attachments/assets/62527f7e-07a6-42a6-a8e0-803098acb393" /> <img width="635" height="610" alt="image" src="https://github.com/user-attachments/assets/babb6d9d-cf86-4fd9-a98c-5f8c97f3d4e0" />
<img width="570" height="287" alt="modikasi python vclocks3 py" src="https://github.com/user-attachments/assets/c23e99bf-2a76-4ae4-8887-0939eafecf94" />



---

## 3.3 Problem Tanpa Sinkronisasi

### Tujuan
Mengamati bahwa urutan pelaksanaan beberapa thread tidak selalu dapat diprediksi.

### Langkah Praktikum
1. Jalankan program multithread dari modul.
2. Ulangi beberapa kali sampai mendapatkan keluaran yang berbeda.
3. Ambil tangkapan layar dari keluaran yang berbeda tersebut.

Pada folder praktikum, nama file yang tersedia mungkin tertulis `multithreads-example.py`, sedangkan modul menyebut `multithreaded-example.py`. Gunakan nama file yang benar-benar ada di folder.

Contoh perintah:

```powershell
python multithreads-example.py
```

### Hasil Program
<img width="537" height="827" alt="image" src="https://github.com/user-attachments/assets/676568b6-0426-4fbe-927c-2257adf2d4fb" />


### Pembahasan
Thread dapat berjalan secara berselang-seling. Urutan keluaran dipengaruhi oleh penjadwalan sistem operasi dan waktu pelaksanaan setiap thread. Karena itu, urutan pesan yang tampil tidak selalu sama pada setiap percobaan. Jika keluaran program ternyata selalu sama, ulangi percobaan dan jelaskan hasil yang benar-benar diperoleh.

---

## 3.3.1 Data Race / Race Conditions

### A. Program `race-conditions-01.py`

#### Langkah Praktikum
Jalankan:

```powershell
python race-conditions-01.py
```

Amati keluaran program dan buat visualisasi manual menggunakan pensil atau ballpoint, kemudian foto hasil gambar tersebut.

#### Hasil Program
<img width="612" height="593" alt="image" src="https://github.com/user-attachments/assets/2b87528c-0fb8-4c42-9b77-0f50a2cc2c21" />

#### Pembahasan
Data race dapat terjadi ketika beberapa thread mengakses data yang sama secara bersamaan, setidaknya salah satunya mengubah data, dan akses tersebut tidak dilindungi sinkronisasi yang sesuai. Urutan operasi dapat saling menyela sehingga hasil akhirnya tidak sesuai dengan yang diharapkan. Pada visualisasi, gambarkan urutan operasi thread yang mengakses atau mengubah data bersama berdasarkan source code yang digunakan.

### B. Program `race-conditions-02.py`

#### Langkah Praktikum
Jalankan:

```powershell
python race-conditions-02.py
```

Amati keluaran program.

#### Hasil Program
<img width="906" height="710" alt="image" src="https://github.com/user-attachments/assets/f409c47b-df33-4e70-aed7-216c1857b057" />

#### Pembahasan
Jelaskan mekanisme pada source code yang mencegah akses bermasalah terhadap data bersama. Jika program menggunakan lock atau mekanisme sinkronisasi lain, jelaskan bagaimana mekanisme tersebut memastikan bagian kode yang mengakses data bersama dijalankan secara terkendali. Cocokkan penjelasan dengan kode yang benar-benar digunakan.

---

## 3.3.2 Deadlock

### A. Program `deadlock-01.py`

#### Langkah Praktikum
Jalankan:

```powershell
python deadlock-01.py
```

Amati perilaku program. Jika program berhenti menunggu dan tidak selesai, catat kondisi tersebut. Jangan menunggu tanpa batas; hentikan proses dari terminal jika diperlukan.

#### Hasil Program
<img width="659" height="701" alt="deadlock-01 py" src="https://github.com/user-attachments/assets/9b857dff-b517-43e6-a33b-a3a1b1f06fc1" />


#### Pembahasan
Deadlock adalah keadaan ketika beberapa thread atau proses saling menunggu sumber daya yang dipegang pihak lain sehingga tidak ada yang dapat melanjutkan. Pada visualisasi manual, gambarkan thread/proses, sumber daya atau lock yang dipegang, dan sumber daya yang sedang ditunggu. Gunakan urutan sesuai source code.

### B. Program `deadlock-02.py`

#### Langkah Praktikum
Jalankan:

```powershell
python deadlock-02.py
```

Amati keluaran program dan bandingkan dengan `deadlock-01.py`.

#### Hasil Program
<img width="435" height="847" alt="deadlock-02 py" src="https://github.com/user-attachments/assets/647e4fd2-3bd6-494c-b9a2-3ff2c0c34da3" />

#### Pembahasan
Jelaskan perubahan pada program kedua yang mencegah deadlock. Penyebab pastinya harus merujuk pada source code—misalnya, urutan pengambilan lock yang konsisten atau mekanisme penguncian yang berbeda, jika memang digunakan oleh program tersebut.

---

## 3.4 Algoritma Raft

### Tujuan
Memahami secara sederhana proses pemilihan koordinator (*leader*) pada algoritma Raft.

### Langkah Praktikum
1. Jalankan program:

   ```powershell
   python raft.py
   ```

2. Amati keluaran program dan catat urutan proses pemilihan leader.
3. Buat visualisasi manual dengan pensil atau ballpoint, lalu foto hasilnya.
4. Pisahkan implementasi Raft menjadi modul Python dan buat program contoh yang mengimpor serta menggunakan modul tersebut.

### Hasil Program

import random
import time
from enum import Enum
from typing import List, Dict, Optional

class State(Enum):
    FOLLOWER = 1
    CANDIDATE = 2
    LEADER = 3

class RaftNode:
    def __init__(self, node_id: int, all_nodes: List[int]):
        self.id = node_id
        self.nodes = all_nodes
        self.state = State.FOLLOWER
        self.current_term = 0
        self.voted_for: Optional[int] = None
        self.log: List[str] = []
        self.commit_index = 0
        self.last_heartbeat = time.time()
        self.election_timeout = self._random_timeout()

    def _random_timeout(self) -> float:
        return time.time() + random.uniform(1.0, 2.0)

    def on_heartbeat(self, term: int):
        if term >= self.current_term:
            self.current_term = term
            self.state = State.FOLLOWER
            self.voted_for = None
            self.last_heartbeat = time.time()
            self.election_timeout = self._random_timeout()

    def start_election(self):
        self.current_term += 1
        self.state = State.CANDIDATE
        self.voted_for = self.id
        votes = 1  # vote for self

        # Simulate requesting votes from others
        for node_id in self.nodes:
            if node_id == self.id:
                continue
            # In real Raft, we'd send RequestVote RPC
            # Here, we simulate: grant vote if term is higher and log is not behind
            votes += 1  # Simplified: assume all grant vote

        if votes > len(self.nodes) // 2:
            self.state = State.LEADER
            print(f"Node {self.id} elected leader in term {self.current_term}")

    def append_entry(self, entry: str):
        if self.state == State.LEADER:
            self.log.append(entry)
            print(f"Leader {self.id} appended: {entry}")
            # In real system, replicate to followers
            self.commit_index = len(self.log) - 1

    def tick(self):
        now = time.time()
        if self.state == State.LEADER:
            # Send heartbeat (simplified)
            pass
        elif now > self.election_timeout:
            self.start_election()
        elif self.state == State.FOLLOWER and now - self.last_heartbeat > 2.0:
            # Missed heartbeats → start election
            self.election_timeout = self._random_timeout()
            self.start_election()

def simulate_raft():
    node_ids = [1, 2, 3]
    nodes = [RaftNode(i, node_ids) for i in node_ids]

    # Simulate time steps
    for step in range(20):
        time.sleep(0.5)
        print(f"\n--- Step {step + 1} ---")

        # Randomly trigger heartbeat from current leader (if any)
        leaders = [n for n in nodes if n.state == State.LEADER]
        if leaders:
            leader = random.choice(leaders)
            for node in nodes:
                if node.id != leader.id:
                    node.on_heartbeat(leader.current_term)
            # Leader appends a command every few steps
            if step % 5 == 0:
                leader.append_entry(f"command-{step}")

        # Each node processes its state
        for node in nodes:
            node.tick()

        # Print status
        for node in nodes:
            print(f"Node {node.id}: {node.state.name} | Term {node.current_term} | Log len {len(node.log)}")

if __name__ == "__main__":
    print(" Simulating Raft Consensus for Drone Swarm Coordination\n")
    simulate_raft()

<img width="614" height="752" alt="image" src="https://github.com/user-attachments/assets/3232c285-f52e-4073-b4e3-45ce9e8445e2" /> <img width="376" height="733" alt="raft 2py" src="https://github.com/user-attachments/assets/1ae145e2-c3ff-45c3-9abd-14edb6f0fe97" /> <img width="530" height="477" alt="raft 3py" src="https://github.com/user-attachments/assets/ff63930e-a880-4d6e-8d07-6b901960fc75" />



### Pembahasan
Secara sederhana, dalam Raft setiap node dapat berada dalam keadaan follower, candidate, atau leader. Jika follower tidak menerima tanda bahwa leader masih aktif selama batas waktu tertentu, node tersebut dapat menjadi candidate dan memulai pemilihan dengan meminta suara dari node lain. Candidate yang memperoleh suara mayoritas menjadi leader untuk term tersebut. Leader kemudian mengoordinasikan operasi sesuai protokol.

---

## Kesimpulan

Praktikum ini membahas sinkronisasi waktu, penggunaan vector clock untuk memahami urutan kejadian, serta dampak kurangnya sinkronisasi pada program multithread. Data race dapat menghasilkan perubahan data yang tidak terduga, sedangkan deadlock dapat membuat thread saling menunggu. Praktikum juga memperkenalkan algoritma Raft untuk memilih leader dalam sistem terdistribusi. Kesimpulan akhir perlu disesuaikan dengan hasil eksekusi dan bukti praktikum yang benar-benar diperoleh.

---
