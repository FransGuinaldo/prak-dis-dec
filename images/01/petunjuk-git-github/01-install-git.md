# Instalasi Git

[ [Kembali](README.md) ]

Bagian ini merupakan seri tulisan tentang [Git](https://git-scm.com/). Silahkan ke [README.md](README.md) untuk memahami gambaran garis besar materi-materi yang dituliskan.

Git tersedia untuk berbagai sistem operasi. *Precompiled binaries* bisa diperoleh di [halaman dowbload Git](https://git-scm.com/downloads) untuk 3 sistem operasi utama: Linux, Mac OS X, dan Windows. Git bisa menggunakan antarmuka grafis (GUI) maupun CLI (*command line interface*). Pada materi ini, kita akan banyak menggunakan antarmuka CLI melalui shell (Linux / Mac OS X) atau *command prompt* / PowerShell di Windows. Setelah instalasi, periksa keberhasilan instalasi dengan menggunakan:

```
$ git --version
git version 2.30.1
```

Jika muncul versi (tergantung versi yang terinstall), maka kita bisa mulai menggunakan Git.

## Linux

Git untuk Linux biasanya sudah ada pada masing-masing distro dan bisa diinstall dengan *package manager* dari distro yang bersangkutan. Sebagai contoh:

1. OpenSuSE dan turunannya: sudo zypper in git
2. Arch Linux dan turunannya: sudo pacman -S git
3. Debian dan turunannya: sudo apt install git
4. RedHat dan turunannya: sudo dnf install git

Silahkan melihat pada manual dari *package manager* di distro yang bersangkutan. 

## Mac OS X

Git untuk Mac OS X juga sudah tersedia dan bisa diinstall menggunakan [Homebrew](http://brew.sh) (*package manager* di Mac OS X):

```
brew install git
```

## Windows

Sebelum install Git di Windows, anda harus sudah mempunyai editor teks yang didukung oleh Windws. Editor yang bisa dipilih banyak, tetapi disarankan menggunakan [Notepad++](https://notepad-plus-plus.org/) atau [Visual Studion Code](https://code.visualstudio.com/) atau [Vim](https://www.vim.org/). Keberadaan editor teks ini akan menentukan keberhasilan instalasi (lihat langkah 5).

1. Setelah download Git, double click pada file yang di-download. Akan dimunculkan lisensi. Klik **Next** untuk lanjut.

<img width="595" height="458" alt="image" src="https://github.com/user-attachments/assets/ff29d809-2e9e-4a19-9bed-55d261555eea" />


2. Setelah itu, pilih lokasi instalasi. Secara default akan terisi *C:\Program Files\Git*. Ganti lokasi jika memang anda menginginkan lokasi lain, klik **Next**

<img width="593" height="459" alt="image" src="https://github.com/user-attachments/assets/7854e75d-1f26-4c80-a512-5908478aad2f" />


3. Pilih komponen. Tidak perlu diubah-ubah, sesuai dengan default saja. Klik pada **Next**.

<img width="596" height="463" alt="image" src="https://github.com/user-attachments/assets/b5f41077-dc31-47a2-8dd8-1f20919c6f32" />


4. Mengisi shortcut untuk menu Start. Gunakan default (Git), ganti jika ingin mengganti - misalnya Git VCS.

<img width="593" height="461" alt="image" src="https://github.com/user-attachments/assets/5e3be426-587d-43f2-8821-219e50a59142" />


5. Pilih editor yang akan digunakan bersama dengan Git. Pada dasarnya anda bebas menggunakan editor teks apapun. 

<img width="596" height="460" alt="image" src="https://github.com/user-attachments/assets/28faba38-4395-4cab-aa1f-d6124d213482" />


Beberapa editor teks yang bisa anda gunakan adalah:

<img width="410" height="144" alt="image" src="https://github.com/user-attachments/assets/81a6cd62-66d1-49c3-a471-83278263eda3" />


Anda juga bisa menggunakan editor pilihan anda sendiri selain di daftar dengan memilih pilihan terakhir dan kemudian mengisikan *executable file* dari editor teks yang akan digunakan.

<img width="414" height="63" alt="image" src="https://github.com/user-attachments/assets/7418baf2-667c-4e28-bb12-6955c1939e30" />


6. Setiap melakukan inisialisasi repo Git, suatu nama branch akan diberikan. Default nama adalah **master** tetapi umumnya sekarang diganti dengan **main**. Ubahlah konfigurasi tersebut:

<img width="594" height="457" alt="image" src="https://github.com/user-attachments/assets/460fa109-a035-4a19-b52b-bb5ca14b3f96" />


7. Pada saat instalasi, Git menyediakan akses git melalui Bash maupun command prompt. Pilih pilihan kedua supaya bisa menggunakan dari dua antarmuka tersebut. Bash adalah shell di Linux. Dengan menggunakan bash di Windows, pekerjaan di command line Windows bisa dilakukan menggunakan bash - termasuk ekskusi dari Git.

<img width="594" height="459" alt="image" src="https://github.com/user-attachments/assets/0e993b5b-06ce-431a-9f29-b22a4bd54d23" />


8. Pilih **native Windows Secure Channel library** HTTPS. Git menggunakan https untuk akes ke repo GitHub atau repo-repo lain (GitLab, Assembla).

<img width="596" height="462" alt="image" src="https://github.com/user-attachments/assets/cabc98c7-256a-49e1-a58b-cef51000a0e1" />


9. Pilih pilihan pertama untuk konversi akhir baris (CR-LF).

<img width="591" height="461" alt="image" src="https://github.com/user-attachments/assets/8de9e36c-66c1-4675-afee-43dae94083fa" />


10. Pilih MinTTY untuk terminal yang digunakan untuk mengakses Git Bash.

<img width="594" height="460" alt="image" src="https://github.com/user-attachments/assets/06a91612-a0d4-44b7-bd7f-cbbd74cd8ce0" />


11. Tetapkan perilaku standar dari **git pull**. Pilih default saja yaitu **Fast-forward or merge**. Arti dari hal ini akan dipelajari pada proses pembelajaran lanjutan. 

![11](images/01/install-11.jpg)<img width="593" height="459" alt="image" src="https://github.com/user-attachments/assets/1af9325e-620e-4382-94b8-58f3bc127173" />


12. Memilih **credential helper**.

![12](images/01/install-12.jpg)<img width="593" height="461" alt="image" src="https://github.com/user-attachments/assets/27532f2f-7e58-4e71-a724-80d0c7a1529d" />


13. Untuk opsi ekstra, pilih serta aktifkan *file system caching*.

![13](images/01/install-13.jpg)<img width="593" height="460" alt="image" src="https://github.com/user-attachments/assets/97b8bb2a-9f9f-4e8d-a6f7-fa1a6e63a3c1" />


14. Setelah itu proses instalasi akan dilakukan.

![014](images/01/install-14.jpg)<img width="595" height="457" alt="image" src="https://github.com/user-attachments/assets/8e2a544d-9560-46b5-a9d1-ad7e20de32b8" />


15. Jika selesai akan muncul dialog pemberitahuan. Klik pada **Finish**.

![015](images/01/install-15.jpg)<img width="595" height="458" alt="image" src="https://github.com/user-attachments/assets/d058a67a-729f-4908-954b-0c9038f48c93" />


16. Untuk mencoba dari command prompt, masuk ke command prompt, setelah itu jalankan "git" untuk melihat apakah sudah terinstall atau belum. Jika sudah terinstall dengan benar, makan akan muncul hasil berikut:

```bash
C:\Program Files\Git\bin>git
usage: git [-v | --version] [-h | --help] [-C <path>] [-c <name>=<value>]
           [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
           [-p | --paginate | -P | --no-pager] [--no-replace-objects] [--no-lazy-fetch]
           [--no-optional-locks] [--no-advice] [--bare] [--git-dir=<path>]
           [--work-tree=<path>] [--namespace=<name>] [--config-env=<name>=<envvar>]
           <command> [<args>]

These are common Git commands used in various situations:

start a working area (see also: git help tutorial)
   clone      Clone a repository into a new directory
   init       Create an empty Git repository or reinitialize an existing one

work on the current change (see also: git help everyday)
   add        Add file contents to the index
   mv         Move or rename a file, a directory, or a symlink
   restore    Restore working tree files
   rm         Remove files from the working tree and from the index

examine the history and state (see also: git help revisions)
   bisect     Use binary search to find the commit that introduced a bug
   diff       Show changes between commits, commit and working tree, etc
   grep       Print lines matching a pattern
   log        Show commit logs
   show       Show various types of objects
   status     Show the working tree status

grow, mark and tweak your common history
   backfill   Download missing objects in a partial clone
   branch     List, create, or delete branches
   commit     Record changes to the repository
   merge      Join two or more development histories together
   rebase     Reapply commits on top of another base tip
   reset      Set `HEAD` or the index to a known state
   switch     Switch branches
   tag        Create, list, delete or verify tags

collaborate (see also: git help workflows)
   fetch      Download objects and refs from another repository
   pull       Fetch from and integrate with another repository or a local branch
   push       Update remote refs along with associated objects

'git help -a' and 'git help -g' list available subcommands and some
concept guides. See 'git help <command>' or 'git help <concept>'
to read about a specific subcommand or concept.
See 'git help git' for an overview of the system.

C:\Program Files\Git\bin>
```

Lihat versi dari Git:

```bash
C:\Program Files\Git\bin>git --version
git version 2.53.0.windows.1

C:\Program Files\Git\bin>
```

Jika anda sudah terbiasa menggunakan **winget**, instalasi bisa dilakukan secara cepat dari Powershell / command prompt dengan perintah:

```bash
winget install --id Git.Git -e --source winget
```

Hasilnya akan sama dengan langkah-langkah di atas jika memilih kondisi default.
