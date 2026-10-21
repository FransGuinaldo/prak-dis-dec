**LAPORAN PRAKTIKUM MODUL 3**

**Sinkronisasi pada Sistem Terdistribusi**

**Nama: Francisco G I dc Corbafo**

**NIM: 255410003**

**Kelas: IF-1**

**Mata Kuliah: Praktikum Sistem Terdistribusi dan Terdesentralisasi**

**Tanggal Praktikum: 21 Oktober 2026**

3.1 Sinkronisasi Waktu
Tujuan
Memahami proses sinkronisasi waktu komputer menggunakan protokol NTP (Network Time Protocol).
Langkah Praktikum
1. Mengunduh dan memasang NetTime dari https://www.timesynctool.com/.
2. Membuka NetTime.
3. Memastikan server NTP tersedia pada pengaturan aplikasi.
4. Menjalankan proses sinkronisasi waktu.
5. Mengamati hasil sinkronisasi dan mencatat pesan yang ditampilkan aplikasi.
Hasil dan Pembahasan
Tempelkan tangkapan layar NetTime setelah proses sinkronisasi di sini.

NetTime menyelaraskan waktu komputer dengan waktu dari server NTP melalui jaringan. Komputer meminta informasi waktu dari server, kemudian aplikasi menyesuaikan waktu komputer berdasarkan hasil tersebut. Sinkronisasi waktu penting agar pencatatan waktu pada beberapa komputer lebih konsisten.

Catatan: Sesuaikan pembahasan ini dengan hasil yang benar-benar ditampilkan NetTime. Jangan menyatakan sinkronisasi berhasil sebelum prosesnya dijalankan.

3.2 Vector Clock
Tujuan
Memahami penggunaan vector clock untuk membandingkan urutan kejadian (event) dalam sistem terdistribusi.
Langkah Praktikum
1. Buka terminal di folder source code.
2. Jalankan perintah:
   python vclocks.py
3. Amati dan simpan keluaran program.
4. Bandingkan nilai vector clock pada keluaran program dengan perhitungan manual.
5. Buat modul Python yang berisi kelas VectorClock, lalu buat program contoh yang menggunakan modul tersebut.
Hasil Program
Tempelkan tangkapan layar keluaran vclocks.py di sini.
Pembahasan
Vector clock menyimpan penghitung untuk setiap proses. Secara umum, ketika suatu proses melakukan kejadian lokal, nilai penghitung milik proses tersebut bertambah. Saat mengirim pesan, proses menyertakan vector clock-nya. Ketika menerima pesan, proses menggabungkan informasi clock yang diterima dengan clock miliknya menggunakan nilai maksimum pada setiap posisi, lalu menambah penghitung proses penerima sesuai aturan algoritmanya.

Untuk membandingkan hasil program dengan cara manual:
1. Catat nilai awal vector clock setiap proses.
2. Ikuti urutan kejadian pada source code.
3. Perbarui nilai clock pada setiap kejadian lokal, pengiriman pesan, dan penerimaan pesan.
4. Bandingkan nilai akhir per kejadian dengan keluaran program.
Dua kejadian dapat dibandingkan berdasarkan nilai vector clock-nya. Jika semua komponen clock A kurang dari atau sama dengan clock B dan setidaknya satu komponen lebih kecil, A terjadi sebelum B. Jika tidak ada hubungan seperti itu, kedua kejadian bersifat konkuren.
Modul VectorClock dan Contoh Penggunaan
Tempelkan kode modul yang dibuat dan kode program contoh di sini. Sertakan nama file serta tangkapan layar ketika contoh berhasil dijalankan.

3.3 Problem Tanpa Sinkronisasi
Tujuan
Mengamati bahwa urutan pelaksanaan beberapa thread tidak selalu dapat diprediksi.
Langkah Praktikum
1. Jalankan program multithread dari modul.
2. Ulangi beberapa kali sampai mendapatkan keluaran yang berbeda.
3. Ambil tangkapan layar dari keluaran yang berbeda tersebut.
Pada folder praktikum, nama file yang tersedia mungkin tertulis multithreads-example.py, sedangkan modul menyebut multithreaded-example.py. Gunakan nama file yang benar-benar ada di folder.

Contoh perintah:
python multithreads-example.py
Hasil Program
Tempelkan tangkapan layar dari dua atau lebih percobaan yang hasilnya berbeda di sini.
Pembahasan
Thread dapat berjalan secara berselang-seling. Urutan keluaran dipengaruhi oleh penjadwalan sistem operasi dan waktu pelaksanaan setiap thread. Karena itu, urutan pesan yang tampil tidak selalu sama pada setiap percobaan. Jika keluaran program ternyata selalu sama, ulangi percobaan dan jelaskan hasil yang benar-benar diperoleh.

3.3.1 Data Race / Race Conditions
A. Program race-conditions-01.py
Langkah Praktikum
Jalankan:
python race-conditions-01.py
Amati keluaran program dan buat visualisasi manual menggunakan pensil atau ballpoint, kemudian foto hasil gambar tersebut.
Hasil Program
Tempelkan tangkapan layar keluaran program di sini.
Tempelkan foto visualisasi manual di sini.
Pembahasan
Data race dapat terjadi ketika beberapa thread mengakses data yang sama secara bersamaan, setidaknya salah satunya mengubah data, dan akses tersebut tidak dilindungi sinkronisasi yang sesuai. Urutan operasi dapat saling menyela sehingga hasil akhirnya tidak sesuai dengan yang diharapkan. Pada visualisasi, gambarkan urutan operasi thread yang mengakses atau mengubah data bersama berdasarkan source code yang digunakan.

B. Program race-conditions-02.py
Langkah Praktikum
Jalankan:
python race-conditions-02.py
Amati keluaran program.
Hasil Program
Tempelkan tangkapan layar keluaran program di sini.
Pembahasan
Jelaskan mekanisme pada source code yang mencegah akses bermasalah terhadap data bersama. Jika program menggunakan lock atau mekanisme sinkronisasi lain, jelaskan bagaimana mekanisme tersebut memastikan bagian kode yang mengakses data bersama dijalankan secara terkendali. Cocokkan penjelasan dengan kode yang benar-benar digunakan.

3.3.2 Deadlock
A. Program deadlock-01.py
Langkah Praktikum
Jalankan:
python deadlock-01.py
Amati perilaku program. Jika program berhenti menunggu dan tidak selesai, catat kondisi tersebut. Jangan menunggu tanpa batas; hentikan proses dari terminal jika diperlukan.
Hasil Program
Tempelkan tangkapan layar keluaran atau kondisi program yang macet di sini.
Tempelkan foto visualisasi manual di sini.
Pembahasan
Deadlock adalah keadaan ketika beberapa thread atau proses saling menunggu sumber daya yang dipegang pihak lain sehingga tidak ada yang dapat melanjutkan. Pada visualisasi manual, gambarkan thread/proses, sumber daya atau lock yang dipegang, dan sumber daya yang sedang ditunggu. Gunakan urutan sesuai source code.
B. Program deadlock-02.py
Langkah Praktikum
Jalankan:
python deadlock-02.py
Amati keluaran program dan bandingkan dengan deadlock-01.py.
Hasil Program
Tempelkan tangkapan layar keluaran program di sini.
Pembahasan
Jelaskan perubahan pada program kedua yang mencegah deadlock. Penyebab pastinya harus merujuk pada source code—misalnya, urutan pengambilan lock yang konsisten atau mekanisme penguncian yang berbeda, jika memang digunakan oleh program tersebut.

3.4 Algoritma Raft
Tujuan
Memahami secara sederhana proses pemilihan koordinator (leader) pada algoritma Raft.
Langkah Praktikum
1. Jalankan program:
   python raft.py
2. Amati keluaran program dan catat urutan proses pemilihan leader.
3. Buat visualisasi manual dengan pensil atau ballpoint, lalu foto hasilnya.
4. Pisahkan implementasi Raft menjadi modul Python dan buat program contoh yang mengimpor serta menggunakan modul tersebut.
Hasil Program
Tempelkan tangkapan layar keluaran raft.py di sini.
Tempelkan foto visualisasi manual pemilihan leader di sini.
Pembahasan
Secara sederhana, dalam Raft setiap node dapat berada dalam keadaan follower, candidate, atau leader. Jika follower tidak menerima tanda bahwa leader masih aktif selama batas waktu tertentu, node tersebut dapat menjadi candidate dan memulai pemilihan dengan meminta suara dari node lain. Candidate yang memperoleh suara mayoritas menjadi leader untuk term tersebut. Leader kemudian mengoordinasikan operasi sesuai protokol.

Sesuaikan urutan penjelasan dan gambar dengan keluaran raft.py yang dijalankan, karena detail simulasi bergantung pada implementasi program.
Modul Python dan Contoh Simulasi
Tempelkan struktur file, kode modul, kode program contoh, dan tangkapan layar hasil eksekusi di sini. Pastikan program contoh benar-benar mengimpor modul yang dibuat, bukan hanya menjalankan kembali file raft.py secara langsung.
Kesimpulan
Praktikum ini membahas sinkronisasi waktu, penggunaan vector clock untuk memahami urutan kejadian, serta dampak kurangnya sinkronisasi pada program multithread. Data race dapat menghasilkan perubahan data yang tidak terduga, sedangkan deadlock dapat membuat thread saling menunggu. Praktikum juga memperkenalkan algoritma Raft untuk memilih leader dalam sistem terdistribusi. Kesimpulan akhir perlu disesuaikan dengan hasil eksekusi dan bukti praktikum yang benar-benar diperoleh.
Daftar Bukti Praktikum
- [ ] Tangkapan layar NetTime setelah sinkronisasi NTP.
- [ ] Keluaran vclocks.py dan perbandingan perhitungan manual.
- [ ] Kode modul VectorClock dan contoh penggunaannya.
- [ ] Keluaran multithread yang berbeda pada beberapa percobaan.
- [ ] Keluaran race-conditions-01.py dan foto visualisasi manual.
- [ ] Keluaran race-conditions-02.py.
- [ ] Keluaran deadlock-01.py dan foto visualisasi manual.
- [ ] Keluaran deadlock-02.py.
- [ ] Keluaran raft.py dan foto visualisasi manual pemilihan leader.
- [ ] Modul Python Raft, contoh simulasi, dan bukti eksekusinya.
