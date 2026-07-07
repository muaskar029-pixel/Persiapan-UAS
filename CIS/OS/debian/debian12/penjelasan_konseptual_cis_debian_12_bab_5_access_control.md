# Penjelasan Konseptual CIS Debian Linux 12 Benchmark
# F. Bab 5 — Access Control

**Sumber utama:** CIS Debian Linux 12 Benchmark v2.0.0, Bab 5 — *Access Control*  
**Fokus pembahasan:** konseptual, bukan panduan praktik command-by-command.  
**Tujuan:** memahami mengapa rekomendasi access control dalam CIS penting untuk hardening Debian Linux 12.

---

## Gambaran Umum Bab 5 — Access Control

Bab **Access Control** dalam CIS Debian Linux 12 Benchmark membahas pengamanan akses ke sistem. Jika Bab 1 sampai Bab 4 banyak berbicara tentang fondasi sistem, layanan, jaringan, dan firewall, maka Bab 5 masuk ke wilayah yang sangat dekat dengan identitas pengguna: siapa yang boleh masuk, bagaimana mereka masuk, seberapa jauh hak aksesnya, bagaimana autentikasi dikontrol, dan bagaimana akun lokal dikelola.

Dalam konteks keamanan Linux, access control adalah salah satu lapisan paling penting karena banyak serangan tidak langsung menghancurkan sistem dari luar, tetapi mencoba memperoleh akses sah atau semi-sah ke dalam sistem. Contohnya adalah brute force SSH, kredensial lemah, akun lama yang masih aktif, izin file konfigurasi yang terlalu longgar, penyalahgunaan sudo, login root langsung, password kosong, shell interaktif pada akun sistem, atau konfigurasi PAM yang terlalu permisif.

Secara konseptual, Bab 5 dapat dipahami sebagai usaha menjawab pertanyaan berikut:

1. Bagaimana akses administrasi jarak jauh melalui SSH harus dikontrol?
2. Bagaimana eskalasi hak akses dari user biasa ke administrator harus dibatasi dan diaudit?
3. Bagaimana proses autentikasi lokal dikendalikan melalui PAM?
4. Bagaimana akun user, akun root, akun sistem, password, shell, PATH, dan umask diamankan?

Access control bukan sekadar soal “bisa login atau tidak”. Access control mencakup identitas, autentikasi, otorisasi, audit trail, pembatasan privilege, dan perilaku default lingkungan user. Dalam sistem Linux yang aman, user tidak boleh langsung diberi hak penuh tanpa alasan, akses root harus dikendalikan, password harus memiliki kebijakan yang baik, akun sistem tidak boleh diperlakukan seperti akun manusia, dan mekanisme login harus menghasilkan jejak audit yang jelas.

---

# 5.1 Configure SSH Server

## SSH sebagai Pintu Akses Administrasi

SSH atau *Secure Shell* adalah salah satu layanan paling penting pada server Linux. Melalui SSH, administrator dapat mengakses sistem dari jarak jauh, menjalankan perintah, mengubah konfigurasi, melakukan pemeliharaan, memeriksa log, mengelola service, dan melakukan troubleshooting. Karena fungsinya sangat kuat, SSH dapat dianggap sebagai pintu utama administrasi server.

Dalam lingkungan server, SSH sering kali lebih sensitif dibandingkan layanan lain karena siapa pun yang berhasil masuk melalui SSH akan mendapatkan akses shell. Jika akun yang dipakai memiliki hak sudo atau akses root, kompromi SSH dapat berubah menjadi kompromi penuh terhadap sistem. Oleh karena itu, CIS menempatkan konfigurasi SSH sebagai bagian besar dari Bab 5.

Hardening SSH bertujuan untuk memastikan bahwa akses jarak jauh:

- hanya tersedia bagi pihak yang berwenang;
- menggunakan algoritma kriptografi yang kuat;
- tidak memberi ruang untuk login lemah;
- menghasilkan audit trail yang jelas;
- membatasi kesempatan brute force;
- mencegah penyalahgunaan fitur forwarding;
- dan mengurangi kemungkinan bypass kontrol keamanan.

## Risiko Salah Konfigurasi SSH

Kesalahan konfigurasi SSH dapat membuka jalur serangan yang serius. Beberapa contoh risiko konseptualnya adalah:

- **Login root langsung** membuat penyerang cukup menebak satu akun bernama `root`.
- **Password kosong** memungkinkan login tanpa rahasia autentikasi yang layak.
- **Cipher atau MAC lemah** dapat melemahkan keamanan komunikasi terenkripsi.
- **MaxAuthTries terlalu besar** memberi lebih banyak kesempatan brute force dalam satu koneksi.
- **Forwarding aktif tanpa kebutuhan** dapat dipakai untuk tunneling yang tidak diawasi.
- **PermitUserEnvironment aktif** dapat membuka peluang manipulasi environment variable.
- **ListenAddress terlalu luas** membuat SSH terbuka pada interface yang tidak seharusnya.
- **LogLevel terlalu rendah** mengurangi bukti saat investigasi insiden.

SSH bukan hanya layanan jaringan; ia adalah mekanisme akses administratif. Karena itu, setiap opsi SSH harus dipandang dari sisi risiko akses, autentikasi, privilege, dan audit.

---

## Permission File Konfigurasi SSH

File konfigurasi SSH, seperti konfigurasi utama `sshd_config` dan file konfigurasi tambahan di direktori terkait, menentukan perilaku layanan SSH. Jika file ini dapat diubah oleh user yang tidak berwenang, maka kebijakan akses SSH dapat dimanipulasi.

Secara konseptual, permission file konfigurasi SSH harus dibatasi agar:

- hanya root atau administrator yang berwenang dapat mengubahnya;
- user biasa tidak dapat menurunkan keamanan SSH;
- konfigurasi tidak dapat disisipkan opsi berbahaya;
- audit terhadap perubahan konfigurasi menjadi lebih jelas.

Misalnya, jika user biasa dapat mengubah konfigurasi SSH, ia dapat mencoba mengaktifkan login root, mengubah port/listen address, melemahkan cipher, mengaktifkan forwarding, atau mengubah parameter autentikasi. Karena itu, kontrol permission file konfigurasi adalah bagian dari prinsip *configuration integrity*.

---

## Host Key Private dan Public

SSH menggunakan **host key** untuk membuktikan identitas server kepada client. Host key berbeda dari password user. Host key adalah identitas kriptografis milik server. Ketika client melakukan koneksi SSH, client dapat memverifikasi apakah server yang dihubungi adalah server yang sama seperti sebelumnya.

Ada dua jenis host key yang perlu dipahami:

### Private Host Key

Private host key adalah bagian rahasia dari identitas server. File ini harus sangat dilindungi. Jika private host key bocor, penyerang dapat menyamar sebagai server tersebut dalam skenario tertentu, terutama jika client tidak memverifikasi perubahan host key dengan benar.

Risiko jika private host key tidak dilindungi:

- penyerang dapat melakukan impersonasi server;
- kepercayaan client terhadap identitas server dapat rusak;
- serangan man-in-the-middle menjadi lebih mungkin;
- integritas administrasi jarak jauh terganggu.

### Public Host Key

Public host key tidak bersifat rahasia seperti private key. Namun, akses dan integritasnya tetap perlu dijaga agar identitas server tidak membingungkan dan tidak mudah dimanipulasi. Public key dipakai client untuk mengenali server.

Secara prinsip, private key harus sangat terbatas aksesnya, sedangkan public key boleh lebih terbuka tetapi tetap harus dimiliki dan dikelola dengan benar.

---

## Banner SSH

Banner SSH adalah pesan yang ditampilkan sebelum atau saat proses login SSH. Dalam konteks CIS, banner bukan sekadar hiasan, tetapi dapat berfungsi sebagai **legal notice** dan pengingat bahwa akses hanya boleh dilakukan oleh pihak yang berwenang.

Fungsi konseptual banner SSH:

- memberi peringatan bahwa sistem hanya untuk pengguna yang diotorisasi;
- mendukung kebutuhan kebijakan organisasi;
- membantu aspek legal dan audit;
- mengurangi klaim bahwa pengguna tidak mengetahui batasan akses.

Namun, banner tidak boleh membocorkan informasi sensitif. Banner yang menampilkan versi OS, nama aplikasi internal, struktur jaringan, atau informasi teknologi terlalu detail dapat membantu penyerang melakukan fingerprinting.

Contoh prinsip banner yang baik:

- menyatakan bahwa akses terbatas untuk pihak berwenang;
- menyatakan bahwa aktivitas dapat dimonitor;
- tidak menyebut versi kernel, versi Debian, hostname internal sensitif, atau informasi teknis lain yang tidak perlu.

---

## Cipher, MAC, dan KexAlgorithms

SSH mengandalkan kriptografi. Tiga komponen penting dalam konfigurasi kriptografi SSH adalah **Cipher**, **MAC**, dan **KexAlgorithms**.

### Cipher

Cipher adalah algoritma enkripsi yang digunakan untuk menjaga kerahasiaan data selama sesi SSH. Jika cipher lemah digunakan, komunikasi dapat lebih rentan terhadap serangan kriptografi.

Tujuan hardening cipher adalah memastikan SSH hanya menggunakan algoritma enkripsi yang masih dianggap kuat dan sesuai kebijakan keamanan modern.

### MAC

MAC atau *Message Authentication Code* digunakan untuk memastikan integritas pesan. Dengan MAC, penerima dapat mendeteksi apakah data berubah selama transmisi. Jika MAC lemah digunakan, integritas komunikasi dapat lebih sulit dijamin.

Secara sederhana:

- cipher menjaga kerahasiaan;
- MAC menjaga integritas;
- keduanya saling melengkapi dalam sesi SSH.

### KexAlgorithms

KexAlgorithms atau *key exchange algorithms* adalah algoritma yang digunakan untuk menyepakati kunci sesi antara client dan server. Proses key exchange sangat penting karena dari proses inilah kunci enkripsi sesi dibentuk.

Jika algoritma key exchange lemah, maka fondasi sesi terenkripsi dapat terpengaruh. Karena itu, CIS mendorong pemilihan algoritma key exchange yang lebih aman dan menghindari algoritma lama yang sudah tidak sesuai standar keamanan modern.

---

## ClientAliveInterval

`ClientAliveInterval` berkaitan dengan mekanisme SSH untuk memeriksa apakah client masih aktif. Secara konseptual, parameter ini membantu menghindari sesi SSH yang dibiarkan menggantung terlalu lama.

Risiko sesi SSH yang idle terlalu lama:

- terminal administrator dapat ditinggalkan dalam keadaan login;
- orang lain yang memiliki akses fisik ke perangkat admin dapat mengambil alih sesi;
- sesi lama yang tidak dipantau dapat menjadi celah operasional;
- jejak aktivitas menjadi kurang jelas.

Dengan pembatasan idle session, sistem dapat memutus koneksi yang tidak aktif setelah periode tertentu. Ini membantu mengurangi risiko sesi terbuka yang lupa ditutup.

---

## DisableForwarding

SSH forwarding memungkinkan SSH dipakai untuk meneruskan koneksi jaringan. Fitur ini berguna dalam administrasi tertentu, tetapi juga dapat disalahgunakan.

Jenis forwarding yang sering dikaitkan dengan SSH antara lain:

- local port forwarding;
- remote port forwarding;
- dynamic forwarding;
- agent forwarding;
- X11 forwarding.

Secara konseptual, forwarding dapat mengubah SSH menjadi jalur tunnel. Jika tidak dibutuhkan, fitur ini dapat memperbesar risiko karena user dapat melewati kontrol jaringan, membuat jalur akses tersembunyi, atau mengakses layanan internal melalui tunnel SSH.

CIS mendorong pembatasan forwarding sebagai bagian dari prinsip **least functionality**: fitur yang tidak dibutuhkan sebaiknya tidak diaktifkan.

---

## GSSAPIAuthentication

GSSAPIAuthentication berkaitan dengan mekanisme autentikasi berbasis GSSAPI, yang biasanya relevan dalam lingkungan tertentu seperti Kerberos atau integrasi enterprise authentication.

Jika organisasi tidak menggunakan GSSAPI, mengaktifkannya dapat menambah kompleksitas dan memperbesar permukaan konfigurasi. Dalam hardening, fitur autentikasi tambahan yang tidak digunakan sebaiknya dinonaktifkan.

Konsep utamanya:

- autentikasi harus sesederhana mungkin sesuai kebutuhan;
- mekanisme yang tidak digunakan tidak perlu aktif;
- semakin banyak metode autentikasi aktif, semakin banyak jalur yang harus diamankan.

---

## HostbasedAuthentication

HostbasedAuthentication memungkinkan autentikasi berdasarkan kepercayaan terhadap host. Model ini berisiko karena kepercayaan tidak hanya diberikan kepada user, tetapi juga kepada mesin asal.

Risiko konseptual HostbasedAuthentication:

- jika host asal dikompromikan, akses ke server target dapat ikut terpengaruh;
- model kepercayaan antar host dapat sulit diaudit;
- dapat menghidupkan kembali pola autentikasi lama yang kurang sesuai dengan prinsip modern;
- membuka peluang penyalahgunaan konfigurasi trust.

Dalam sistem modern, autentikasi sebaiknya tetap berpusat pada identitas user, metode autentikasi kuat, dan kontrol akses eksplisit.

---

## IgnoreRhosts

File seperti `.rhosts` dan `.shosts` berasal dari model kepercayaan lama. Jika mekanisme ini dipakai, user atau host tertentu dapat dipercaya tanpa proses autentikasi yang kuat seperti password atau key modern.

Mengaktifkan `IgnoreRhosts` berarti SSH tidak menggunakan file-file trust lama tersebut. Secara konseptual, ini penting karena:

- mekanisme trust berbasis `.rhosts` sulit dikontrol;
- file tersebut dapat dimanipulasi;
- autentikasi berbasis trust lama tidak sekuat kebijakan modern;
- audit akses menjadi lebih lemah.

Dengan mengabaikan rhosts, sistem mengurangi ketergantungan pada metode autentikasi historis yang tidak lagi ideal.

---

## LoginGraceTime

`LoginGraceTime` menentukan berapa lama koneksi SSH boleh berada dalam keadaan belum berhasil login. Jika waktu ini terlalu lama, penyerang dapat mempertahankan koneksi setengah terbuka lebih lama.

Konsep keamanan di balik pengaturan ini adalah membatasi jendela waktu autentikasi. Koneksi yang tidak segera menyelesaikan login sebaiknya diputus.

Manfaatnya:

- mengurangi potensi penyalahgunaan koneksi menggantung;
- membatasi waktu brute force dalam satu sesi;
- mengurangi konsumsi resource akibat koneksi tidak selesai;
- mempercepat pembersihan koneksi mencurigakan.

---

## LogLevel

`LogLevel` menentukan detail logging SSH. Logging sangat penting untuk audit dan investigasi insiden. Jika logging terlalu minim, administrator mungkin tidak memiliki informasi cukup untuk mengetahui siapa mencoba login, kapan percobaan terjadi, dan apakah ada pola serangan.

Konsep pentingnya:

- log SSH membantu mendeteksi brute force;
- log membantu melacak akses administratif;
- log menjadi bukti dalam investigasi;
- log mendukung kepatuhan audit.

Namun logging juga harus seimbang. Logging yang terlalu verbose dapat menghasilkan noise, sedangkan logging yang terlalu rendah dapat menghilangkan bukti penting.

---

## MaxAuthTries

`MaxAuthTries` membatasi jumlah percobaan autentikasi dalam satu koneksi SSH. Jika nilainya terlalu besar, penyerang memiliki lebih banyak kesempatan mencoba password atau key dalam satu sesi.

Pembatasan ini membantu mengurangi efektivitas brute force dan password guessing. Prinsipnya sederhana: semakin sedikit kesempatan gagal yang diberikan, semakin kecil ruang penyerang untuk mencoba kombinasi kredensial.

Namun nilai ini juga harus mempertimbangkan usability. Terlalu ketat dapat mengganggu admin yang sah, tetapi terlalu longgar dapat memperbesar risiko.

---

## MaxStartups

`MaxStartups` membatasi jumlah koneksi SSH yang belum selesai autentikasi. Ini penting untuk mencegah kondisi ketika banyak koneksi belum login membanjiri SSH daemon.

Risiko jika tidak dibatasi:

- penyerang dapat membuat banyak koneksi setengah terbuka;
- SSH daemon dapat kehabisan resource;
- login admin sah dapat terganggu;
- server dapat mengalami denial-of-service kecil pada layanan administrasi.

Pengaturan ini tidak hanya melindungi autentikasi, tetapi juga ketersediaan layanan SSH.

---

## MaxSessions

`MaxSessions` membatasi jumlah sesi yang dapat dibuka melalui satu koneksi SSH. Satu koneksi SSH dapat membuka lebih dari satu sesi, misalnya shell, command, atau subsistem tertentu.

Pembatasan ini penting agar satu koneksi tidak digunakan secara berlebihan. Secara konseptual, ini membantu mengendalikan resource dan mengurangi ruang penyalahgunaan oleh user yang sudah berhasil login.

---

## PermitEmptyPasswords

`PermitEmptyPasswords` menentukan apakah akun dengan password kosong boleh login melalui SSH. Secara keamanan, password kosong adalah kondisi yang sangat berbahaya.

Risiko password kosong:

- autentikasi menjadi hampir tidak berarti;
- akun lokal yang salah konfigurasi dapat langsung dieksploitasi;
- brute force tidak lagi diperlukan;
- audit keamanan akan menilai sistem sangat lemah.

Menolak password kosong adalah kontrol dasar yang wajib dipahami. Sistem seharusnya tidak pernah mengizinkan login remote ke akun tanpa password.

---

## PermitRootLogin

`PermitRootLogin` menentukan apakah user root dapat login langsung melalui SSH. CIS mendorong agar root login langsung dinonaktifkan.

Secara konseptual, larangan login root langsung penting karena:

- root adalah akun superuser yang namanya pasti diketahui;
- login root langsung melemahkan prinsip non-repudiation;
- sulit membedakan administrator mana yang melakukan aksi jika semua memakai root langsung;
- lebih aman jika admin login dengan akun personal lalu melakukan eskalasi privilege yang tercatat.

Dengan menonaktifkan login root langsung, sistem mendorong pola:

1. admin login sebagai user individual;
2. admin melakukan eskalasi dengan sudo jika diperlukan;
3. aktivitas lebih mudah ditelusuri ke identitas personal.

Ini meningkatkan audit trail dan akuntabilitas.

---

## PermitUserEnvironment

`PermitUserEnvironment` memungkinkan user mengatur environment variable melalui SSH. Environment variable dapat memengaruhi perilaku program, path eksekusi, library, locale, atau parameter lain.

Risiko jika diaktifkan:

- user dapat mencoba memanipulasi `PATH`;
- program dapat diarahkan ke binary palsu;
- kontrol keamanan tertentu dapat dibypass;
- perilaku sesi SSH menjadi sulit diprediksi.

Dalam hardening, environment user harus dikendalikan. User tidak boleh memiliki kebebasan yang dapat memengaruhi proses autentikasi atau eksekusi program secara tidak aman.

---

## UsePAM

`UsePAM` menghubungkan SSH dengan PAM atau *Pluggable Authentication Modules*. Jika UsePAM aktif, SSH dapat memanfaatkan kebijakan PAM untuk autentikasi, account checks, session management, dan pembatasan lain.

Secara konseptual, UsePAM penting karena:

- kebijakan login dapat dikonsolidasikan melalui PAM;
- pembatasan akun, waktu, lockout, dan session dapat berlaku untuk SSH;
- sistem dapat menerapkan kontrol autentikasi yang konsisten;
- audit dan kebijakan akses menjadi lebih terpusat.

Jika UsePAM tidak digunakan, beberapa kontrol berbasis PAM mungkin tidak diterapkan pada login SSH. Karena itu, pada banyak sistem Linux modern, integrasi SSH dengan PAM penting untuk menjaga konsistensi access control.

---

## Post-Quantum Key Exchange

Post-quantum key exchange mengacu pada algoritma pertukaran kunci yang dirancang untuk menghadapi risiko kriptografi di era komputasi kuantum. Walaupun komputer kuantum skala besar yang mampu memecahkan kriptografi umum belum menjadi ancaman praktis untuk semua organisasi, standar keamanan modern mulai memperhatikan konsep **crypto agility** dan kesiapan terhadap ancaman masa depan.

Dalam konteks SSH, key exchange adalah fase awal pembentukan sesi aman. Jika algoritma key exchange rentan, sesi terenkripsi dapat terancam. Rekomendasi terkait post-quantum key exchange menunjukkan bahwa CIS mulai memperhatikan kebutuhan algoritma yang lebih tahan terhadap risiko jangka panjang.

Konsep yang perlu dipahami:

- keamanan kriptografi tidak statis;
- algoritma yang aman hari ini bisa menjadi lemah di masa depan;
- organisasi perlu menyesuaikan konfigurasi kriptografi seiring perkembangan standar;
- post-quantum readiness adalah bagian dari strategi keamanan jangka panjang.

---

## ListenAddress

`ListenAddress` menentukan alamat IP mana yang digunakan SSH daemon untuk menerima koneksi. Jika SSH mendengarkan di semua interface, maka layanan SSH bisa terbuka pada jaringan yang tidak seharusnya.

Contoh risiko konseptual:

- server memiliki interface internal dan eksternal, tetapi SSH terbuka di keduanya;
- SSH tidak sengaja dapat diakses dari jaringan tamu;
- layanan administrasi terekspos ke segmen jaringan yang tidak perlu;
- firewall mungkin tidak menjadi satu-satunya pengendali akses.

Mengatur ListenAddress membantu membatasi permukaan akses SSH pada alamat jaringan yang memang dikhususkan untuk administrasi. Ini sesuai prinsip *least exposure*.

---

# 5.2 Configure Privilege Escalation

## Konsep Privilege Escalation

Privilege escalation adalah proses memperoleh hak akses yang lebih tinggi dari hak akses awal. Dalam Linux, user biasa biasanya memiliki hak terbatas, sedangkan root memiliki hak penuh. Eskalasi privilege diperlukan agar administrator dapat menjalankan tugas tertentu tanpa login langsung sebagai root.

Secara keamanan, privilege escalation bukan sesuatu yang harus dihilangkan sepenuhnya. Sistem tetap membutuhkan mekanisme administrasi. Yang harus dilakukan adalah mengontrolnya.

Kontrol privilege escalation bertujuan agar:

- hanya user tertentu yang dapat melakukan eskalasi;
- eskalasi membutuhkan autentikasi ulang;
- aktivitas eskalasi tercatat;
- sesi privileged tidak berlangsung terlalu lama;
- akses root langsung dapat dikurangi;
- akuntabilitas tetap terjaga.

---

## sudo

`sudo` adalah mekanisme umum di Linux untuk menjalankan perintah dengan hak akses user lain, biasanya root. Dibandingkan login langsung sebagai root, sudo lebih baik untuk audit karena dapat mencatat user asal yang menjalankan perintah.

Keunggulan konseptual sudo:

- mendukung prinsip least privilege;
- admin dapat diberi hak hanya untuk perintah tertentu jika diperlukan;
- aktivitas dapat dicatat atas nama user personal;
- tidak perlu membagikan password root ke banyak orang;
- akses administratif lebih mudah dikendalikan.

Namun sudo juga harus dikonfigurasi dengan hati-hati. Jika terlalu longgar, sudo bisa sama berbahayanya dengan memberikan akses root penuh tanpa kontrol.

---

## sudo dengan pty

PTY atau *pseudo-terminal* adalah terminal semu yang dapat digunakan saat menjalankan perintah melalui sudo. Mengharuskan sudo menggunakan pty membantu meningkatkan keamanan dan auditabilitas.

Secara konseptual, penggunaan pty membantu:

- mengurangi risiko penyalahgunaan input/output command;
- mendukung logging yang lebih baik;
- membuat sesi privileged lebih terkontrol;
- mengurangi kemungkinan serangan tertentu terhadap mekanisme terminal.

Dalam hardening, sudo tidak hanya soal siapa yang boleh menjalankan perintah, tetapi juga bagaimana perintah itu dijalankan dan dicatat.

---

## sudo Log File

Log sudo berfungsi mencatat aktivitas eskalasi privilege. Tanpa log, administrator sulit mengetahui siapa menjalankan perintah apa dengan hak tinggi.

Manfaat sudo log:

- mendukung audit trail;
- membantu investigasi insiden;
- mendeteksi penyalahgunaan privilege;
- memberi bukti aktivitas administratif;
- meningkatkan akuntabilitas.

Log sudo harus disimpan dengan permission yang tepat agar tidak bisa diubah oleh user biasa. Jika log dapat dimodifikasi, bukti aktivitas privileged dapat dihapus atau dimanipulasi.

---

## Password untuk Eskalasi

CIS menekankan bahwa eskalasi privilege sebaiknya membutuhkan password. Jika user dapat menjalankan sudo tanpa password, maka kompromi akun user biasa langsung dapat berubah menjadi akses privileged.

Risiko sudo tanpa password:

- malware yang berjalan sebagai user dapat langsung mengeksekusi perintah root;
- akun yang ditinggalkan login dapat disalahgunakan;
- batas antara user biasa dan privileged menjadi terlalu tipis;
- audit tetap ada, tetapi kontrol autentikasi melemah.

Meminta password saat eskalasi memberikan lapisan verifikasi tambahan bahwa pengguna yang menjalankan perintah benar-benar pemilik akun tersebut.

---

## Re-authentication

Re-authentication berarti user harus melakukan autentikasi ulang ketika melakukan eskalasi privilege setelah periode tertentu atau dalam kondisi tertentu. Ini penting agar akses privileged tidak terus aktif tanpa batas.

Konsepnya mirip dengan pintu aman: meskipun seseorang pernah masuk, ia tidak boleh diberi akses selamanya tanpa verifikasi ulang.

Manfaat re-authentication:

- mengurangi risiko sesi yang ditinggalkan;
- mencegah privilege terus aktif terlalu lama;
- meningkatkan kepastian identitas saat perintah penting dijalankan;
- mendukung prinsip least privilege secara temporal.

---

## timestamp_timeout

`timestamp_timeout` menentukan berapa lama sudo mengingat autentikasi user sebelum meminta password lagi. Jika timeout terlalu panjang, user dapat terus menjalankan sudo tanpa autentikasi ulang dalam waktu lama.

Pengaturan timeout harus seimbang:

- terlalu panjang: meningkatkan risiko penyalahgunaan sesi;
- terlalu pendek: mengganggu administrasi normal;
- nilai wajar: menjaga keamanan sekaligus usability.

Secara konseptual, timestamp_timeout adalah kontrol waktu terhadap privilege escalation.

---

## Pembatasan su

`su` memungkinkan user berpindah menjadi user lain, termasuk root, jika mengetahui password target. Jika `su` terbuka untuk semua user, maka siapa pun dapat mencoba berpindah ke root.

Pembatasan `su` biasanya dilakukan agar hanya kelompok tertentu yang boleh mencoba menggunakan su. Ini penting karena:

- mengurangi kesempatan brute force password root;
- membatasi jalur eskalasi privilege;
- mendorong penggunaan sudo yang lebih mudah diaudit;
- mengurangi penyalahgunaan akun root.

Dalam hardening modern, sudo sering lebih disukai daripada su karena sudo lebih granular dan lebih baik untuk logging.

---

# 5.3 Pluggable Authentication Modules

## Gambaran Umum PAM

PAM atau *Pluggable Authentication Modules* adalah kerangka kerja autentikasi modular di Linux. Dengan PAM, berbagai layanan seperti login lokal, sudo, SSH, dan aplikasi lain dapat memakai modul autentikasi yang konsisten.

PAM memisahkan aplikasi dari mekanisme autentikasi. Artinya, aplikasi tidak perlu mengimplementasikan seluruh logika password, lockout, password quality, atau session policy sendiri. Aplikasi cukup memanggil PAM, dan PAM menjalankan modul sesuai konfigurasi.

Manfaat PAM:

- kebijakan autentikasi lebih terpusat;
- modul dapat ditambah atau diganti;
- layanan berbeda dapat memakai aturan yang konsisten;
- kontrol password, lockout, dan session lebih mudah dikelola;
- hardening autentikasi dapat dilakukan secara sistematis.

Namun PAM juga sensitif. Salah konfigurasi PAM dapat menyebabkan sistem terlalu permisif, menolak login sah, atau bahkan mengunci administrator dari sistem. Karena itu, PAM harus dipahami secara hati-hati.

---

## 5.3.1 PAM Software Packages

### pam

`pam` adalah fondasi utama untuk mekanisme Pluggable Authentication Modules. Paket ini menyediakan infrastruktur yang memungkinkan sistem memakai modul PAM dalam proses autentikasi.

Secara konseptual, keberadaan PAM penting karena banyak kontrol access control bergantung pada PAM. Tanpa PAM yang tepat dan mutakhir, sistem mungkin tidak dapat menerapkan kebijakan autentikasi modern secara konsisten.

### libpam-modules

`libpam-modules` menyediakan modul-modul PAM umum yang digunakan oleh sistem. Modul-modul ini menjalankan fungsi autentikasi, account management, password management, dan session management.

Paket modul PAM harus tersedia dan mutakhir karena modul inilah yang benar-benar menerapkan aturan, bukan hanya framework PAM-nya.

### libpam-pwquality

`libpam-pwquality` berkaitan dengan pemeriksaan kualitas password. Modul ini membantu memastikan password memenuhi syarat tertentu, seperti panjang minimum, kompleksitas, variasi karakter, dan larangan pola yang terlalu mudah.

Tujuan konseptualnya bukan memaksa password rumit tanpa alasan, tetapi mengurangi kemungkinan password mudah ditebak.

### cracklib-runtime

`cracklib-runtime` mendukung pemeriksaan password terhadap pola atau kamus yang lemah. Password yang terdapat dalam kamus, terlalu umum, atau mudah ditebak sebaiknya ditolak.

Dalam keamanan autentikasi, password lemah adalah salah satu akar masalah paling sering. Pemeriksaan berbasis kamus membantu mencegah penggunaan password yang tampak memenuhi panjang minimum tetapi tetap mudah ditebak.

---

## 5.3.2 pam-auth-update Profiles

`pam-auth-update` adalah mekanisme Debian untuk mengelola profile PAM. Profile ini membantu mengaktifkan modul tertentu secara lebih terstruktur daripada mengedit semua file PAM secara manual.

Konsep profile penting karena konfigurasi PAM harus konsisten. Jika setiap file PAM diubah secara manual tanpa pola, risiko salah konfigurasi meningkat.

### pam_unix

`pam_unix` adalah modul PAM tradisional untuk autentikasi menggunakan akun lokal Unix/Linux. Modul ini berinteraksi dengan database akun lokal seperti `/etc/passwd` dan `/etc/shadow`.

Secara konseptual, pam_unix adalah dasar autentikasi lokal. Ia menangani password lokal, validasi akun, dan integrasi dengan mekanisme user Linux.

### pam_faillock

`pam_faillock` digunakan untuk mencatat dan mengunci akun setelah sejumlah percobaan login gagal. Ini merupakan kontrol terhadap brute force.

Manfaat pam_faillock:

- mengurangi risiko password guessing;
- memperlambat serangan brute force;
- memberi sinyal adanya percobaan login mencurigakan;
- dapat dikombinasikan dengan logging untuk deteksi insiden.

Namun lockout juga perlu hati-hati karena dapat disalahgunakan untuk denial-of-service terhadap akun tertentu jika threshold terlalu agresif.

### pam_pwquality

`pam_pwquality` memeriksa kualitas password saat user membuat atau mengganti password. Modul ini membantu memastikan password tidak terlalu pendek, tidak terlalu sederhana, dan tidak mudah ditebak.

Konsep utamanya adalah mencegah password lemah sebelum password tersebut digunakan.

### pam_pwhistory

`pam_pwhistory` mencegah user menggunakan ulang password lama. Tanpa password history, user dapat mengganti password baru lalu segera kembali ke password lama.

Manfaatnya:

- mendorong rotasi password yang nyata;
- mencegah reuse password lama yang mungkin sudah bocor;
- memperkuat kebijakan password expiration;
- mengurangi pola user yang hanya mengganti password secara formalitas.

---

## 5.3.3 PAM Arguments

PAM arguments adalah parameter yang diberikan kepada modul PAM untuk mengatur perilakunya. Modul yang sama dapat bertindak berbeda tergantung argumen yang dipakai.

Contoh konsep:

- pam_faillock dapat diberi argumen jumlah gagal login;
- pam_pwquality dapat diberi argumen panjang minimum password;
- pam_pwhistory dapat diberi argumen jumlah password lama yang diingat;
- pam_unix dapat diberi argumen algoritma hashing atau larangan password kosong.

Argumen PAM penting karena detail kecil dapat mengubah kekuatan kontrol autentikasi. Kesalahan argumen bisa menyebabkan kebijakan tidak berjalan sesuai harapan.

---

## Lockout Login Gagal

Lockout login gagal berarti akun dikunci setelah terlalu banyak percobaan autentikasi gagal. Tujuannya adalah mengurangi efektivitas brute force.

Jika tidak ada lockout, penyerang dapat terus mencoba password dalam jumlah besar. Dengan lockout, percobaan gagal memiliki konsekuensi.

Namun kebijakan lockout harus seimbang:

- terlalu longgar: brute force tetap efektif;
- terlalu ketat: user sah mudah terkunci;
- perlu memperhatikan akun lokal dan akun terpusat seperti LDAP/AD agar tidak terjadi double lockout.

---

## Unlock Time

Unlock time menentukan berapa lama akun tetap terkunci setelah mencapai batas gagal login. Jika unlock time terlalu pendek, brute force dapat berlanjut dengan jeda kecil. Jika terlalu panjang, user sah dapat terganggu.

Konsep unlock time adalah memberikan penalti waktu pada percobaan gagal tanpa membuat pemulihan akun menjadi terlalu sulit.

---

## Root Lockout

Root lockout berarti mekanisme gagal login juga berlaku terhadap akun root. Ini sensitif karena root adalah akun paling kuat, tetapi juga berisiko jika root terkunci dan tidak ada jalur administrasi alternatif.

Secara keamanan, root tidak boleh kebal terhadap kontrol brute force. Namun organisasi harus memastikan ada mekanisme pemulihan yang aman jika akun administratif terkunci.

---

## Password Length

Panjang password adalah salah satu faktor paling penting dalam ketahanan password terhadap brute force. Password yang lebih panjang biasanya jauh lebih sulit ditebak dibanding password pendek, bahkan jika password pendek menggunakan variasi karakter.

Konsep pentingnya:

- panjang meningkatkan ruang kemungkinan password;
- password pendek mudah ditebak atau dipecahkan;
- panjang minimum harus disesuaikan dengan kebijakan organisasi;
- penggunaan passphrase panjang sering lebih baik daripada password pendek yang terlalu rumit.

---

## Password Complexity

Password complexity mengatur variasi karakter atau syarat tertentu dalam password. Misalnya huruf besar, huruf kecil, angka, simbol, atau jumlah karakter berbeda.

Tujuan kompleksitas adalah mencegah password terlalu sederhana. Namun kompleksitas tidak boleh dipahami secara sempit. Password seperti `Password123!` mungkin memenuhi kompleksitas, tetapi tetap lemah karena mudah ditebak.

Kebijakan yang baik menyeimbangkan:

- panjang password;
- larangan password umum;
- pemeriksaan kamus;
- variasi karakter;
- edukasi user.

---

## Dictionary Check

Dictionary check menolak password yang terdapat dalam kamus atau daftar kata umum. Ini penting karena banyak serangan password menggunakan wordlist.

Contoh password yang harus dihindari:

- kata umum;
- nama organisasi;
- nama user;
- pola keyboard;
- password yang sering bocor;
- kata yang hanya ditambah angka sederhana.

Dictionary check membantu mencegah password yang tampak valid tetapi mudah ditebak.

---

## Password History

Password history mencegah user memakai ulang password lama. Ini penting terutama jika organisasi menerapkan password expiration.

Tanpa history, user bisa mengganti password menjadi password baru lalu kembali ke password lama. Dengan history, sistem mengingat beberapa password sebelumnya dan menolaknya jika digunakan ulang.

Password history membantu mencegah:

- reuse password bocor;
- rotasi formalitas;
- pola password berulang;
- penurunan efektivitas kebijakan expiration.

---

## Password Hashing

Password di Linux tidak seharusnya disimpan dalam bentuk plaintext. Sistem menyimpan hash password. Hashing adalah proses satu arah untuk merepresentasikan password tanpa menyimpan password aslinya.

Algoritma hashing yang kuat penting karena jika file hash password bocor, penyerang akan mencoba melakukan cracking offline. Algoritma lemah membuat cracking lebih mudah.

Konsep penting:

- password hash harus memakai algoritma modern;
- hash berbeda dari enkripsi biasa karena tidak dimaksudkan untuk didekripsi;
- salt dan parameter hashing membantu melawan precomputed attacks;
- algoritma lama harus dihindari.

---

## use_authtok

`use_authtok` adalah argumen PAM yang memastikan modul menggunakan token autentikasi/password yang sudah diperoleh sebelumnya dalam stack PAM, bukan meminta atau membuat token baru secara terpisah.

Secara konseptual, ini membantu menjaga konsistensi alur perubahan password. Dalam stack PAM yang melibatkan beberapa modul seperti pwquality, pwhistory, dan pam_unix, penggunaan token yang konsisten penting agar password yang diperiksa adalah password yang sama dengan yang akhirnya disimpan.

---

## nullok

`nullok` adalah argumen yang dapat mengizinkan password kosong. Dalam hardening, keberadaan `nullok` sangat berisiko karena membuka kemungkinan akun tanpa password dianggap sah.

Secara konseptual, password kosong berarti autentikasi kehilangan maknanya. Karena itu, konfigurasi PAM sebaiknya tidak mengizinkan `nullok` untuk akun yang dapat login.

Risiko nullok:

- akun tanpa password dapat login;
- kesalahan provisioning akun menjadi fatal;
- serangan lokal atau remote menjadi lebih mudah jika layanan menerima autentikasi tersebut.

---

# 5.4 User Accounts and Environment

## Gambaran Umum

Bagian ini membahas pengamanan akun lokal dan lingkungan default user. Banyak kompromi sistem terjadi bukan karena layanan jaringan langsung, tetapi karena akun lokal yang salah konfigurasi, password policy yang buruk, akun sistem yang bisa login, UID/GID duplikat, root PATH yang tidak aman, atau umask yang terlalu permisif.

User account hardening bertujuan memastikan:

- akun manusia dan akun sistem dibedakan;
- root tetap unik dan terkendali;
- password aging diterapkan dengan wajar;
- akun tidak aktif dikunci;
- shell login hanya diberikan kepada akun yang membutuhkan;
- default permission file tidak terlalu terbuka;
- lingkungan root tidak mudah dimanipulasi.

---

## 5.4.1 Shadow Password Suite Parameters

Shadow password suite mengacu pada mekanisme pengelolaan password dan account aging di Linux, terutama melalui file seperti `/etc/shadow` dan konfigurasi terkait. File shadow menyimpan informasi password hash dan kebijakan umur password.

### Password Expiration

Password expiration menentukan masa berlaku password. Setelah waktu tertentu, user harus mengganti password.

Tujuannya:

- mengurangi risiko password lama yang sudah bocor tetap digunakan;
- memaksa rotasi kredensial;
- mendukung kebijakan keamanan organisasi;
- membatasi umur kredensial.

Namun password expiration harus diterapkan secara bijak. Jika terlalu sering, user mungkin membuat pola password lemah seperti mengganti angka di akhir. Karena itu expiration harus didukung oleh password history dan quality checks.

### Minimum Password Age

Minimum password age menentukan waktu minimum sebelum user boleh mengganti password lagi. Kontrol ini mencegah user mengganti password berkali-kali untuk melewati password history lalu kembali ke password lama.

Tanpa minimum age, user dapat melakukan rotasi cepat hingga password lama keluar dari daftar history.

### Warning Days

Warning days adalah periode peringatan sebelum password kedaluwarsa. User diberi tahu bahwa password akan segera habis masa berlakunya.

Manfaatnya:

- mengurangi gangguan operasional;
- memberi waktu user mengganti password;
- mencegah akun tiba-tiba tidak dapat digunakan;
- mendukung pengalaman user yang lebih baik.

### Password Hash

Parameter password hash menentukan algoritma hashing yang digunakan untuk password baru. Algoritma yang kuat penting untuk melindungi password jika database hash bocor.

Secara konseptual, password hash adalah pertahanan terakhir saat file shadow terekspos. Algoritma modern membuat cracking lebih mahal secara komputasi.

### Inactive Password Lock

Inactive password lock mengunci akun setelah password kedaluwarsa dan tidak diganti dalam periode tertentu. Ini penting karena akun yang tidak digunakan tetapi tetap aktif dapat menjadi celah.

Risiko akun tidak aktif:

- sering luput dari pemantauan;
- pemiliknya mungkin sudah pindah peran atau keluar organisasi;
- password mungkin tidak pernah diganti;
- dapat menjadi target penyerang karena aktivitasnya jarang diperhatikan.

### Last Password Change Date

Tanggal perubahan password terakhir harus masuk akal dan tidak berada di masa depan. Jika tanggal berada di masa depan, mekanisme password aging dapat terganggu.

Konsepnya adalah integritas metadata akun. Sistem keamanan bergantung pada data waktu yang benar. Jika informasi tanggal salah, kebijakan expiration dan aging bisa tidak berjalan sebagaimana mestinya.

---

## 5.4.2 Root and System Accounts

### UID 0

Dalam Linux, UID 0 adalah identitas superuser. Secara tradisional, akun `root` memiliki UID 0. Jika ada akun lain dengan UID 0, akun tersebut memiliki kekuatan setara root.

Risiko akun tambahan dengan UID 0:

- backdoor administratif tersembunyi;
- audit menjadi membingungkan;
- kontrol root dapat dilewati;
- akuntabilitas hilang karena banyak nama akun memiliki hak root penuh.

Prinsipnya: root harus menjadi satu-satunya akun dengan UID 0.

### GID 0

GID 0 biasanya terkait dengan group root. Kontrol terhadap GID 0 penting karena group root memiliki kedudukan sensitif dalam sistem.

Jika user atau group lain diberi GID 0 tanpa alasan, batas privilege dapat kabur. Ini dapat membuka akses tidak semestinya ke file atau mekanisme yang bergantung pada group root.

### Kontrol Akses Root

Akun root harus dikontrol secara ketat karena memiliki hak penuh terhadap sistem. Kontrol root tidak hanya berarti password kuat, tetapi juga mencakup:

- pembatasan login langsung;
- penggunaan sudo untuk audit;
- perlindungan shell root;
- integritas PATH root;
- pembatasan akses fisik dan remote;
- pemantauan aktivitas root.

Root bukan akun biasa. Setiap penggunaan root harus dianggap sebagai aktivitas administratif sensitif.

### Root PATH Integrity

`PATH` menentukan lokasi direktori tempat shell mencari program yang akan dieksekusi. Untuk root, PATH sangat sensitif. Jika PATH root berisi direktori yang dapat ditulis user biasa, penyerang dapat menaruh program palsu dengan nama yang sama seperti command umum.

Contoh konsep serangan:

1. PATH root mengarah ke direktori tidak aman.
2. Penyerang menaruh binary palsu bernama seperti command sistem.
3. Root menjalankan command tersebut tanpa path absolut.
4. Program palsu berjalan dengan privilege root.

Karena itu, integritas PATH root sangat penting.

### Root umask

`umask` menentukan permission default saat file atau direktori baru dibuat. Untuk root, umask yang terlalu permisif dapat membuat file administratif dapat dibaca atau dimodifikasi pihak lain.

Root sering membuat file sensitif, sehingga default permission harus konservatif. Umask yang kuat membantu mencegah file baru dibuat dengan akses terlalu luas.

### System Account Shell

Akun sistem digunakan untuk menjalankan service, bukan untuk login interaktif manusia. Contohnya akun untuk daemon tertentu. Akun seperti ini seharusnya tidak memiliki shell login valid kecuali benar-benar diperlukan.

Risiko jika akun sistem memiliki shell valid:

- jika service dikompromikan, penyerang dapat memperoleh shell interaktif;
- akun non-manusia dapat digunakan untuk login;
- batas antara akun service dan user manusia menjadi kabur;
- audit menjadi lebih sulit.

Akun sistem sebaiknya memakai shell non-login seperti konsep `nologin` atau `false`.

### Locked Non-login Accounts

Akun yang tidak memiliki shell login valid juga sebaiknya dikunci agar tidak dapat digunakan untuk autentikasi. Shell non-login saja tidak selalu cukup jika mekanisme lain masih memungkinkan autentikasi.

Mengunci akun non-login membantu memastikan akun tersebut benar-benar tidak dapat dipakai sebagai jalur login.

---

## 5.4.3 User Default Environment

### /etc/shells

`/etc/shells` berisi daftar shell yang dianggap valid oleh sistem. File ini digunakan oleh beberapa program untuk menentukan apakah shell user sah.

Jika shell non-login seperti `nologin` tercantum sebagai shell valid, ini dapat menimbulkan kebingungan atau potensi perilaku yang tidak diinginkan. Secara konseptual, hanya shell interaktif yang memang boleh dipakai user manusia yang seharusnya dianggap valid.

### Shell Timeout

Shell timeout membatasi berapa lama sesi shell boleh idle sebelum otomatis keluar. Ini penting untuk mengurangi risiko terminal yang ditinggalkan dalam keadaan login.

Risiko tanpa shell timeout:

- sesi admin tetap terbuka lama;
- akses fisik ke workstation admin dapat disalahgunakan;
- shell user dapat diambil alih;
- sesi idle menjadi celah operasional.

Shell timeout adalah kontrol sederhana tetapi berguna untuk mengurangi risiko sesi tidak aktif.

### Default umask

Default umask menentukan permission file/direktori yang dibuat oleh user. Jika umask terlalu longgar, file baru dapat dibaca atau dimodifikasi user lain.

Dalam sistem multi-user, default umask penting karena:

- melindungi file pribadi user;
- mencegah kebocoran data antar user;
- mengurangi risiko file konfigurasi dibuat terlalu terbuka;
- menjaga prinsip least privilege pada level filesystem.

Umask adalah contoh bahwa access control tidak hanya terjadi saat login, tetapi juga saat user membuat file sehari-hari.

---

# Rangkuman Konseptual Bab 5

Bab 5 — Access Control adalah bagian yang sangat penting dalam CIS Debian Linux 12 Benchmark karena mengatur pintu masuk, hak akses, autentikasi, dan identitas user. Fokus utamanya bukan hanya membuat sistem “bisa login”, tetapi membuat proses login dan eskalasi privilege terjadi secara aman, terbatas, dan dapat diaudit.

Inti dari Bab 5 dapat diringkas sebagai berikut:

1. **SSH harus diperlakukan sebagai pintu administrasi utama.** Karena itu, konfigurasi SSH harus membatasi login lemah, algoritma lemah, forwarding tidak perlu, root login langsung, dan eksposur interface yang tidak perlu.

2. **Privilege escalation harus dikontrol.** Administrator tetap membutuhkan hak root, tetapi akses tersebut harus dilakukan melalui mekanisme yang tercatat, membutuhkan autentikasi, memiliki batas waktu, dan tidak terlalu longgar.

3. **PAM adalah pusat kebijakan autentikasi.** PAM mengatur modul login, lockout, kualitas password, history password, dan hashing. Salah konfigurasi PAM dapat berdampak langsung pada keamanan login.

4. **Akun user dan akun sistem harus dibedakan.** Akun manusia boleh login sesuai kebutuhan. Akun sistem sebaiknya tidak memiliki shell interaktif dan harus dikunci jika tidak dipakai untuk login.

5. **Root harus unik dan sangat dibatasi.** UID 0 dan GID 0 harus dikontrol, root PATH harus aman, root umask harus konservatif, dan aktivitas root harus dapat ditelusuri.

6. **Lingkungan default user juga bagian dari keamanan.** Shell timeout, umask, dan daftar shell valid membantu mencegah risiko yang muncul setelah user berhasil login.

Dengan memahami Bab 5 secara konseptual, kita dapat melihat bahwa access control bukan hanya tentang password. Ia adalah gabungan dari identitas, autentikasi, privilege, logging, environment, dan akuntabilitas. Dalam hardening Debian 12, Bab ini menjadi penghubung antara keamanan teknis sistem dan perilaku manusia yang mengakses sistem tersebut.

---

# Catatan untuk Pembelajaran

Untuk pembahasan konseptual, yang paling penting adalah memahami alasan di balik setiap rekomendasi. Command dan konfigurasi teknis bisa dipelajari pada tahap praktik. Namun sebelum menjalankan hardening, administrator harus memahami dampaknya terlebih dahulu.

Beberapa kontrol dalam Bab 5 dapat menyebabkan user terkunci, akses SSH gagal, sudo tidak berjalan, atau autentikasi terganggu jika diterapkan tanpa pengujian. Karena itu, perubahan access control harus diuji di lingkungan lab atau sistem non-produksi sebelum diterapkan pada server utama.

