# Penjelasan Konseptual CIS Debian Linux 12 Benchmark  
## E. Bab 4 — Host Based Firewall

**Sumber utama:** *CIS Debian Linux 12 Benchmark v2.0.0 — 05-28-2026*  
**Fokus pembahasan:** konseptual, bukan praktik command.  
**Ruang lingkup:** Bab 4 — Host Based Firewall, khususnya konfigurasi **Uncomplicated Firewall (UFW)** pada Debian Linux 12.

---

## Orientasi Bab 4 — Host Based Firewall

Bab **Host Based Firewall** membahas perlindungan jaringan pada tingkat host, yaitu perlindungan yang dipasang langsung pada sistem Debian itu sendiri. Jika Bab 2 membahas layanan apa saja yang sebaiknya tidak berjalan, dan Bab 3 membahas perilaku jaringan pada level perangkat, modul kernel, serta parameter IPv4/IPv6, maka Bab 4 membahas lapisan kontrol yang menyaring lalu lintas jaringan yang masuk, keluar, atau melewati host.

Secara sederhana, firewall adalah kumpulan aturan yang menentukan apakah suatu paket jaringan boleh lewat atau tidak. Paket jaringan biasanya diperiksa berdasarkan informasi seperti sumber, tujuan, port, protokol, arah koneksi, dan status koneksinya. Dalam konteks CIS Debian Linux 12 Benchmark, firewall lokal diposisikan sebagai kontrol penting untuk membatasi komunikasi jaringan hanya pada kebutuhan yang sah.

CIS menjelaskan bahwa Linux menyediakan dukungan firewall melalui **NFTables**, yaitu subsistem kernel Linux untuk filtering dan klasifikasi paket jaringan. NFTables menggantikan sebagian komponen Netfilter lama, tetapi tetap memanfaatkan banyak infrastruktur Netfilter seperti hook ke network stack, connection tracking, queueing, dan logging. Di atas mekanisme kernel tersebut, CIS memberikan panduan menggunakan **Uncomplicated Firewall (UFW)** sebagai antarmuka yang lebih sederhana untuk mengelola aturan firewall.

Bab ini tidak terlalu panjang dibanding Bab 1, 2, atau 3, tetapi sangat penting karena firewall sering menjadi batas terakhir ketika ada layanan yang tidak sengaja terbuka, konfigurasi jaringan keliru, atau perangkat berada di jaringan yang tidak sepenuhnya tepercaya.

---

# 4.1 Configure Uncomplicated Firewall

## 4.1.0 Apa itu Host-Based Firewall?

**Host-based firewall** adalah firewall yang berjalan langsung di dalam sistem operasi host. Jika host tersebut adalah server Debian, maka aturan firewall diterapkan langsung pada server Debian itu. Jika host tersebut adalah workstation, maka aturan firewall diterapkan langsung pada workstation.

Firewall jenis ini bekerja pada titik akhir komunikasi. Artinya, sebelum trafik diterima oleh aplikasi di host, trafik tersebut bisa disaring terlebih dahulu oleh sistem operasi. Begitu juga ketika aplikasi di host mencoba membuat koneksi keluar, firewall lokal dapat menentukan apakah koneksi itu boleh dilakukan.

Dalam konteks Debian Linux, host-based firewall biasanya mengatur:

1. koneksi masuk menuju host;
2. koneksi keluar dari host;
3. trafik yang diteruskan atau dirutekan melalui host;
4. aturan berdasarkan port dan protokol;
5. aturan berdasarkan alamat sumber atau tujuan;
6. aturan berdasarkan aplikasi atau layanan tertentu;
7. logging terhadap trafik yang diizinkan atau ditolak.

Konsep utamanya adalah: **setiap host harus memiliki perlindungan sendiri**, bukan hanya bergantung pada firewall jaringan di luar dirinya.

---

## 4.1.1 Perbedaan Firewall Host dan Firewall Jaringan

Firewall jaringan biasanya ditempatkan di antara beberapa segmen jaringan. Contohnya firewall di router, firewall antar-VLAN, firewall di gateway internet, atau security group di cloud. Firewall ini mengontrol trafik yang melewati titik jaringan tertentu.

Firewall host berbeda. Ia berada langsung di endpoint atau server. Karena itu, firewall host tetap bekerja meskipun trafik tidak melewati firewall jaringan tertentu.

Perbandingan konseptualnya:

| Aspek | Firewall Jaringan | Host-Based Firewall |
|---|---|---|
| Lokasi | Router, gateway, perimeter, cloud security group, firewall appliance | Langsung di sistem operasi host |
| Cakupan | Melindungi segmen jaringan atau banyak host | Melindungi satu host tertentu |
| Fokus | Mengontrol trafik antar-jaringan | Mengontrol trafik yang masuk, keluar, atau diteruskan oleh host |
| Risiko jika salah konfigurasi | Banyak host bisa terdampak | Host tersebut bisa terekspos |
| Kelebihan | Terpusat dan mudah mengatur segmentasi besar | Spesifik, dekat dengan aplikasi, tetap aktif meski host berpindah jaringan |
| Keterbatasan | Tidak selalu melihat trafik internal dalam segmen yang sama | Perlu dikelola di setiap host |

Firewall jaringan penting, tetapi tidak cukup. Misalnya, sebuah firewall router mungkin memblokir akses dari internet ke server. Namun jika attacker sudah berada di jaringan internal, firewall router mungkin tidak lagi menjadi penghalang. Pada kondisi seperti ini, host-based firewall dapat membatasi akses langsung ke host.

Contoh lain: jika sebuah server hanya boleh menerima SSH dari laptop admin tertentu, pembatasan ini bisa diterapkan di firewall host. Jadi meskipun ada banyak perangkat di jaringan LAN, hanya sumber tertentu yang boleh mengakses port administrasi.

---

## 4.1.2 Kenapa Firewall Lokal Tetap Penting Walaupun Ada Firewall Router?

Firewall router atau firewall perimeter biasanya melindungi dari trafik antar-segmen atau dari luar jaringan. Namun banyak risiko modern terjadi di dalam jaringan itu sendiri. Karena itu, firewall lokal tetap penting.

Beberapa alasan utamanya:

### 1. Melindungi dari lateral movement

Jika satu perangkat dalam jaringan berhasil dikompromikan, attacker dapat mencoba bergerak ke perangkat lain. Ini disebut **lateral movement**. Firewall host membantu membatasi koneksi antar-perangkat internal.

Misalnya, server database tidak seharusnya menerima koneksi dari semua perangkat di LAN. Ia mungkin hanya perlu menerima koneksi dari application server. Dengan host-based firewall, aturan tersebut bisa ditegakkan langsung di server database.

### 2. Mengurangi dampak layanan yang tidak sengaja terbuka

Kadang ada service yang aktif tanpa disadari. Misalnya aplikasi development membuka port tertentu, daemon lama masih berjalan, atau software baru membuka listener. Firewall host dapat menjadi lapisan tambahan agar service tersebut tidak otomatis bisa diakses dari jaringan.

### 3. Melindungi host saat berpindah jaringan

Workstation atau laptop bisa berpindah dari jaringan kampus, rumah, hotspot, laboratorium, hingga jaringan publik. Firewall router di tiap lokasi berbeda-beda. Host-based firewall membuat perangkat tetap memiliki baseline perlindungan sendiri.

### 4. Melindungi dari salah konfigurasi firewall jaringan

Firewall jaringan bisa salah konfigurasi. Rule bisa terlalu longgar, VLAN bisa salah, security group cloud bisa terbuka, atau router bisa berubah konfigurasi. Firewall host menjadi kontrol pertahanan berlapis.

### 5. Mengontrol trafik keluar

Firewall router sering fokus pada trafik masuk. Namun pada sistem yang lebih ketat, trafik keluar juga harus dikontrol. Misalnya server produksi tidak boleh sembarang menghubungi internet, kecuali ke repository update, DNS, NTP, atau endpoint tertentu.

### 6. Mengamankan IPv4 dan IPv6

Lingkungan yang sudah mengamankan IPv4 belum tentu mengamankan IPv6. Karena UFW mendukung IPv4 dan IPv6, firewall host membantu memastikan dua jenis trafik tersebut ikut dikendalikan.

### 7. Melindungi host multi-interface

Beberapa host memiliki lebih dari satu interface, misalnya Ethernet, Wi-Fi, bridge, VPN, container network, atau virtual interface. Router mungkin tidak selalu mengontrol seluruh jalur itu. Firewall host dapat mengatur trafik berdasarkan perspektif sistem itu sendiri.

Intinya, firewall lokal bukan pengganti firewall jaringan, melainkan bagian dari **defense in depth**. Semakin penting suatu host, semakin penting pula ia memiliki kontrol lokal yang tidak sepenuhnya bergantung pada perangkat jaringan lain.

---

## 4.1.3 UFW sebagai Pendekatan Sederhana pada Debian

**UFW** adalah singkatan dari **Uncomplicated Firewall**. Sesuai namanya, UFW dirancang agar pengelolaan firewall lebih mudah dipahami dibanding mengelola aturan firewall langsung dengan sintaks yang lebih kompleks.

Pada Debian Linux 12 Benchmark, CIS menggunakan UFW sebagai pendekatan untuk mengonfigurasi host-based firewall. Secara konseptual, UFW berperan sebagai frontend, sedangkan filtering sebenarnya tetap dilakukan oleh mekanisme firewall kernel Linux seperti Netfilter/NFTables.

UFW memberikan abstraksi yang lebih ramah bagi administrator. Administrator dapat berpikir dalam bentuk kebijakan sederhana seperti:

- izinkan SSH dari sumber tertentu;
- tolak koneksi masuk secara default;
- izinkan HTTP dan HTTPS pada web server;
- batasi koneksi keluar hanya pada protokol tertentu;
- matikan routing antar-interface secara default.

Dengan pendekatan ini, UFW cocok untuk banyak sistem Debian karena administrator tidak harus langsung berurusan dengan detail aturan low-level.

Namun kemudahan UFW bukan berarti firewall boleh dikonfigurasi sembarangan. CIS tetap menekankan bahwa firewall harus diuji, terutama jika sistem dikonfigurasi melalui koneksi jarak jauh. Salah konfigurasi firewall dapat membuat administrator terkunci dari server, khususnya jika SSH belum diizinkan sebelum kebijakan default deny diterapkan.

---

## 4.1.4 Catatan Penting: Gunakan Satu Metode Firewall

Salah satu prinsip penting dalam Bab 4 adalah jangan mencampur terlalu banyak metode konfigurasi firewall pada satu sistem.

Secara konseptual, Linux dapat dikonfigurasi menggunakan beberapa alat, seperti:

- UFW;
- nft command;
- iptables compatibility layer;
- firewalld;
- script manual;
- tool otomatis dari manajemen konfigurasi.

Masalah muncul jika beberapa alat mengubah aturan firewall yang sama tanpa koordinasi. Akibatnya:

1. aturan bisa saling menimpa;
2. hasil akhir sulit diprediksi;
3. audit menjadi membingungkan;
4. administrator tidak tahu rule mana yang sebenarnya aktif;
5. perubahan dari satu tool dapat merusak aturan dari tool lain;
6. dokumentasi tidak lagi sesuai dengan keadaan sistem.

Karena itu, jika organisasi memilih UFW, maka UFW sebaiknya menjadi metode utama untuk mengelola firewall pada host tersebut. Jika organisasi menggunakan tool lain, prinsip CIS tetap dapat diterapkan, tetapi organisasi perlu memastikan hasil akhirnya sesuai rekomendasi.

---

# Konsep Default Deny

## Apa itu default deny?

**Default deny** adalah prinsip keamanan yang menyatakan bahwa semua koneksi ditolak secara default, kecuali ada aturan eksplisit yang mengizinkannya.

Prinsip ini berlawanan dengan **default allow**, yaitu semua koneksi diizinkan kecuali ada aturan yang melarangnya.

Dalam konteks hardening, default deny lebih aman karena sistem tidak otomatis membuka akses hanya karena administrator lupa menulis aturan penolakan.

Perbandingan konseptualnya:

| Model | Cara kerja | Risiko utama |
|---|---|---|
| Default allow | Semua trafik boleh lewat kecuali dilarang | Administrator harus tahu semua trafik buruk yang perlu diblokir |
| Default deny | Semua trafik ditolak kecuali diizinkan | Administrator harus mendefinisikan trafik sah yang memang dibutuhkan |

CIS lebih mendukung pendekatan default deny karena lebih mudah mendefinisikan kebutuhan yang sah daripada mencoba menebak seluruh kemungkinan trafik yang tidak sah.

Contohnya, pada web server internal, kebutuhan masuk mungkin hanya:

- HTTP dari jaringan tertentu;
- HTTPS dari jaringan tertentu;
- SSH dari host admin tertentu;
- monitoring dari server monitoring.

Selain itu, koneksi lain tidak perlu diterima.

---

## Allowlist lebih aman daripada denylist

Konsep default deny berkaitan erat dengan **allowlist**. Dalam allowlist, hanya hal yang sudah diketahui aman dan diperlukan yang diizinkan.

Sebaliknya, denylist mencoba melarang hal-hal yang dianggap berbahaya. Masalahnya, daftar hal berbahaya selalu berubah dan tidak pernah lengkap. Attacker bisa menggunakan port, protokol, atau teknik yang tidak ada dalam daftar larangan.

Karena itu, dalam firewall host, pendekatan yang lebih aman adalah:

> Tolak semuanya terlebih dahulu, lalu izinkan hanya trafik yang benar-benar diperlukan.

Namun pendekatan ini harus hati-hati. Jika aturan yang sah belum dibuat, sistem bisa kehilangan konektivitas penting. Misalnya SSH, DNS, NTP, update package, backup, monitoring, atau koneksi aplikasi tertentu bisa terblokir.

---

# Incoming, Outgoing, dan Routed Traffic

## 1. Incoming Traffic

**Incoming traffic** adalah trafik yang masuk menuju host. Contohnya:

- client mengakses web server;
- admin melakukan SSH ke server;
- monitoring server memeriksa agent di host;
- aplikasi lain terhubung ke database;
- pengguna mengakses layanan tertentu di host.

Dalam CIS Debian Linux 12 Benchmark, default policy untuk incoming traffic diarahkan agar berada pada kondisi **deny** atau **reject**, bukan accept.

Secara konseptual, ini berarti sistem tidak menerima koneksi masuk secara otomatis. Koneksi masuk hanya diterima jika ada aturan eksplisit yang mengizinkannya.

### Kenapa incoming harus ketat?

Koneksi masuk adalah jalur utama attacker untuk mencoba mengakses service yang berjalan di host. Jika default incoming adalah allow, maka setiap service yang mendengarkan pada jaringan berpotensi langsung bisa diakses.

Risikonya antara lain:

1. service yang lupa dimatikan menjadi terekspos;
2. port administrasi seperti SSH bisa diakses terlalu luas;
3. aplikasi development yang tidak aman bisa terbuka;
4. layanan internal bisa diakses dari jaringan yang tidak seharusnya;
5. serangan brute force, scanning, dan exploit lebih mudah dilakukan.

Dengan default deny incoming, host hanya membuka layanan yang memang sudah disetujui.

### Deny dan reject

Dalam konteks firewall, **deny/drop** dan **reject** sama-sama menolak trafik, tetapi perilakunya berbeda:

- **deny/drop** menolak secara diam-diam; pengirim tidak mendapat respons jelas dan biasanya menunggu timeout.
- **reject** menolak dengan respons eksplisit seperti connection refused atau destination unreachable.

Secara keamanan, drop sering dipakai untuk mengurangi informasi yang diberikan kepada pihak luar. Namun reject kadang berguna untuk troubleshooting karena responsnya lebih jelas. Pilihan akhirnya bergantung pada kebijakan organisasi.

### Contoh konseptual

Jika server adalah web server, maka incoming yang sah mungkin hanya:

- TCP 80 untuk HTTP;
- TCP 443 untuk HTTPS;
- SSH hanya dari alamat admin tertentu;
- monitoring hanya dari server monitoring.

Selain itu, koneksi masuk sebaiknya ditolak.

---

## 2. Outgoing Traffic

**Outgoing traffic** adalah trafik yang dibuat oleh host menuju jaringan luar. Contohnya:

- server melakukan update package ke repository;
- server melakukan query DNS;
- server sinkronisasi waktu ke NTP server;
- aplikasi mengakses API eksternal;
- malware mencoba menghubungi command and control server;
- script mencuri data ke server luar.

Banyak sistem secara default mengizinkan seluruh koneksi keluar. Dalam profil keamanan yang lebih tinggi, CIS membahas pembatasan outgoing traffic sebagai bagian dari Level 2.

### Kenapa outgoing traffic perlu dipikirkan?

Sering kali firewall hanya dipahami sebagai alat untuk memblokir koneksi masuk. Padahal koneksi keluar juga penting. Jika suatu aplikasi atau malware di dalam host dapat bebas menghubungi internet, maka host bisa menjadi sumber kebocoran data atau bagian dari serangan lebih lanjut.

Risiko outgoing yang terlalu bebas antara lain:

1. malware dapat menghubungi server kendali;
2. data dapat keluar tanpa kontrol;
3. aplikasi dapat mengakses layanan eksternal yang tidak disetujui;
4. attacker dapat membuat reverse shell;
5. sistem dapat menjadi proxy tidak sah;
6. audit sulit menentukan koneksi keluar mana yang legitimate.

### Kenapa outgoing default deny termasuk Level 2?

Menerapkan default deny untuk trafik keluar lebih sulit daripada incoming deny. Banyak sistem membutuhkan koneksi keluar untuk fungsi normal, misalnya:

- DNS;
- NTP;
- HTTP/HTTPS untuk update;
- repository package;
- backup remote;
- logging remote;
- monitoring;
- integrasi API;
- license server;
- endpoint keamanan.

Jika default outgoing langsung dibuat deny tanpa perencanaan, aplikasi dapat berhenti bekerja. Karena itu, rekomendasi ini lebih cocok untuk lingkungan dengan kebutuhan keamanan tinggi, dokumentasi jaringan jelas, dan daftar allowlist yang matang.

### Pendekatan konseptual yang aman

Sebelum menerapkan outgoing deny, organisasi harus tahu:

1. host perlu mengakses apa saja;
2. protokol dan port apa yang dibutuhkan;
3. tujuan koneksi mana yang sah;
4. apakah koneksi keluar perlu dibatasi berdasarkan IP, domain, atau service;
5. bagaimana update dan sinkronisasi waktu tetap berjalan;
6. bagaimana logging dan monitoring tidak terputus.

Jadi outgoing firewall bukan sekadar “blokir semua”, tetapi membangun kontrol egress yang sadar kebutuhan sistem.

---

## 3. Routed Traffic

**Routed traffic** adalah trafik yang tidak ditujukan untuk host itu sendiri, tetapi melewati host untuk diteruskan dari satu interface ke interface lain.

Misalnya sebuah host memiliki dua interface:

- satu interface ke jaringan internal;
- satu interface ke jaringan lain atau internet.

Jika host meneruskan trafik antar-interface, maka host berperan seperti router. Dalam kebanyakan server biasa, fungsi ini tidak diperlukan.

CIS mendorong agar default routed traffic berada dalam kondisi disabled atau deny. Artinya, host tidak boleh meneruskan paket jaringan antar-interface kecuali memang dirancang sebagai router, gateway, firewall, VPN concentrator, atau perangkat sejenis.

### Kenapa routed traffic berisiko?

Jika host tidak sengaja menjadi router, maka ia dapat membuka jalur komunikasi yang tidak direncanakan. Risiko ini sangat penting pada sistem multi-interface, sistem virtualisasi, container host, lab jaringan, atau server yang memiliki VPN.

Risiko routed traffic yang tidak dikendalikan:

1. host menjadi jalur bypass segmentasi jaringan;
2. jaringan internal terekspos ke jaringan lain;
3. attacker dapat memanfaatkan host sebagai pivot;
4. aturan firewall jaringan bisa dilewati;
5. trafik antar-interface tidak tercatat sesuai desain;
6. arsitektur keamanan menjadi tidak valid.

### Kapan routed traffic boleh diaktifkan?

Routed traffic boleh dipertimbangkan jika host memang berfungsi sebagai:

- router;
- gateway;
- VPN server;
- firewall host khusus;
- NAT gateway;
- container host dengan kebutuhan forwarding tertentu;
- lab jaringan yang memang dirancang untuk routing.

Namun jika fungsi itu tidak dibutuhkan, default yang aman adalah tidak meneruskan trafik.

---

# Poin-Poin Rekomendasi CIS pada Bab 4

## 4.1.1 Ensure UFW is Installed

### Makna konseptual

Rekomendasi ini memastikan host memiliki utilitas firewall yang dapat digunakan untuk membangun aturan yang tahan lama dan mudah dikelola. Tanpa utilitas firewall, administrator tidak memiliki cara yang konsisten untuk mengatur kebijakan filtering pada host.

Dalam CIS Debian Linux 12 Benchmark, UFW dipilih karena sederhana, familiar, dan cocok untuk host-based firewall. UFW menyediakan antarmuka command-line untuk mengelola aturan firewall tanpa harus selalu menulis aturan low-level secara langsung.

### Kenapa ini penting?

Firewall bukan hanya paket tambahan. Ia adalah kontrol dasar untuk membatasi komunikasi jaringan. Jika UFW tidak tersedia, maka host mungkin tetap bergantung pada konfigurasi jaringan eksternal atau aturan firewall lain yang belum tentu ada.

Secara konseptual, instalasi UFW berarti host disiapkan agar bisa menerapkan prinsip:

- hanya layanan yang diperlukan yang boleh menerima koneksi;
- koneksi masuk tidak otomatis diterima;
- koneksi keluar dapat dikontrol pada profil yang lebih ketat;
- routing tidak dibuka tanpa kebutuhan;
- aturan firewall bisa diaudit.

### Risiko jika tidak ada host firewall

Jika tidak ada host firewall:

1. service yang aktif bisa langsung terekspos;
2. proteksi terlalu bergantung pada router atau firewall jaringan;
3. host sulit menerapkan allowlist lokal;
4. host lebih rentan terhadap lateral movement;
5. kebijakan keamanan berbeda-beda antar-host;
6. audit jaringan menjadi kurang lengkap.

### Catatan kehati-hatian

Firewall perlu dikonfigurasi dengan benar. Memasang UFW saja tidak cukup. UFW harus aktif, memiliki default policy yang sesuai, dan memiliki allow rule yang mendukung fungsi sistem.

---

## 4.1.2 Ensure UFW Service is Configured

### Makna konseptual

Rekomendasi ini memastikan layanan UFW aktif dan berjalan sehingga aturan firewall benar-benar diterapkan. Sebuah sistem bisa saja memiliki paket UFW terpasang, tetapi jika service-nya tidak aktif, maka aturan firewall tidak melindungi host.

Jadi ada perbedaan antara:

- UFW tersedia sebagai software;
- UFW aktif sebagai service;
- UFW memiliki aturan yang benar;
- UFW benar-benar menerapkan filtering.

CIS menekankan bahwa firewall harus aktif, bukan hanya terinstal.

### Kenapa service firewall harus berjalan?

Firewall host adalah kontrol runtime. Ia harus bekerja ketika sistem hidup dan berkomunikasi dengan jaringan. Jika service firewall tidak berjalan, maka host dapat menerima atau mengirim trafik sesuai default sistem tanpa pembatasan yang diinginkan.

Risiko jika UFW tidak aktif:

1. aturan firewall tidak diterapkan;
2. port yang seharusnya dibatasi menjadi terbuka;
3. admin mengira host sudah terlindungi padahal belum;
4. hasil audit bisa gagal;
5. perubahan firewall tidak bertahan setelah reboot;
6. kontrol jaringan tidak konsisten.

### Risiko lockout saat remote administration

CIS memberikan perhatian khusus terhadap risiko lockout. Jika administrator mengaktifkan firewall dari koneksi remote tanpa mengizinkan akses administrasi terlebih dahulu, koneksi bisa terputus.

Contoh paling umum adalah SSH. Jika server dikelola melalui SSH, maka aturan untuk mengizinkan SSH dari sumber yang sah harus dipikirkan sebelum default deny diterapkan.

Secara konseptual, urutannya adalah:

1. pahami jalur administrasi;
2. tentukan siapa yang boleh mengakses;
3. buat aturan izin untuk akses sah;
4. aktifkan firewall;
5. uji koneksi;
6. simpan konfigurasi agar berlaku setelah reboot.

Kegagalan memahami urutan ini dapat menyebabkan administrator kehilangan akses ke server.

---

## 4.1.3 Ensure UFW Incoming Default is Configured

### Makna konseptual

Rekomendasi ini mengatur kebijakan default untuk trafik masuk. CIS mendorong agar incoming default berada pada kondisi **deny** atau **reject**, bukan allow.

Jika default incoming adalah allow, maka koneksi yang tidak memiliki aturan penolakan eksplisit tetap bisa masuk. Ini berbahaya karena administrator harus mengetahui semua hal yang perlu diblokir.

Jika default incoming adalah deny, maka koneksi masuk tidak akan diterima kecuali ada aturan yang mengizinkannya.

### Hubungan dengan prinsip least privilege

Default deny incoming adalah bentuk penerapan **least privilege** pada jaringan. Host hanya menerima koneksi yang diperlukan untuk menjalankan fungsi sahnya.

Contoh:

- server web menerima HTTP/HTTPS;
- server database hanya menerima koneksi dari application server;
- server SSH hanya menerima akses dari IP admin;
- workstation tidak menerima koneksi masuk kecuali ada kebutuhan khusus.

### Risiko jika incoming terlalu terbuka

Jika incoming default dibiarkan allow:

1. semua service listening lebih mudah diakses;
2. port yang tidak sengaja terbuka menjadi risiko;
3. scanning jaringan lebih efektif bagi attacker;
4. brute force terhadap service tertentu lebih mudah dilakukan;
5. lateral movement lebih mudah;
6. kesalahan konfigurasi aplikasi lebih berbahaya.

### Kesimpulan konseptual

Incoming default deny adalah salah satu kontrol paling dasar dalam firewall host. Ia membuat host bersikap tertutup secara default, lalu hanya membuka layanan yang memang diperlukan.

---

## 4.1.4 Ensure UFW Outgoing Default is Configured

### Makna konseptual

Rekomendasi ini mengatur kebijakan default untuk trafik keluar. Pada profil Level 2, CIS mengarahkan agar outgoing default dapat dibuat deny atau reject.

Berbeda dari incoming deny yang umumnya cocok untuk banyak sistem, outgoing deny lebih ketat dan dapat berdampak besar. Karena itu, ia ditempatkan pada Level 2.

### Kenapa outgoing default deny lebih sulit?

Sistem modern sering membutuhkan koneksi keluar. Bahkan server yang tampaknya sederhana tetap bisa membutuhkan:

- DNS untuk resolusi nama;
- NTP untuk sinkronisasi waktu;
- HTTP/HTTPS untuk update;
- koneksi ke repository;
- koneksi ke backup server;
- koneksi ke monitoring atau logging server;
- koneksi ke API internal atau eksternal.

Jika semua outgoing langsung ditolak, banyak fungsi bisa gagal. Karena itu, outgoing deny harus diterapkan dengan perencanaan allowlist.

### Manfaat keamanan outgoing deny

Jika dirancang dengan baik, outgoing deny memberi manfaat besar:

1. membatasi exfiltration data;
2. menghambat malware menghubungi command and control;
3. mengurangi risiko reverse shell;
4. mencegah aplikasi menghubungi endpoint tidak sah;
5. membuat pola komunikasi server lebih dapat diprediksi;
6. membantu audit egress traffic.

### Kapan cocok diterapkan?

Outgoing deny cocok untuk:

- server produksi dengan fungsi terbatas;
- sistem yang sangat sensitif;
- database server;
- server internal yang tidak perlu internet bebas;
- lingkungan dengan dokumentasi dependency jaringan yang matang;
- organisasi dengan kontrol egress ketat.

Namun pada workstation umum, server development, atau sistem yang sering berubah, kebijakan ini perlu diuji lebih hati-hati agar tidak mengganggu pekerjaan.

---

## 4.1.5 Ensure UFW Routed Default is Configured

### Makna konseptual

Rekomendasi ini memastikan host tidak meneruskan trafik antar-interface secara default. CIS mengarahkan agar routed traffic berada pada kondisi disabled atau deny.

Ini penting karena tidak semua host boleh berperan sebagai router. Server aplikasi, database server, workstation, atau server layanan biasa umumnya tidak perlu meneruskan paket dari satu jaringan ke jaringan lain.

### Hubungan dengan network segmentation

Network segmentation dibuat untuk membatasi komunikasi antar-segmen. Jika sebuah host secara tidak sengaja meneruskan trafik antar-interface, ia dapat merusak segmentasi tersebut.

Misalnya:

- interface pertama terhubung ke jaringan internal;
- interface kedua terhubung ke jaringan tamu;
- host meneruskan trafik antar-keduanya.

Jika ini terjadi tanpa desain yang benar, jaringan tamu bisa memiliki jalur tidak sah ke jaringan internal.

### Risiko host menjadi pivot

Dalam serangan jaringan, attacker sering mencari host yang dapat digunakan sebagai pivot. Host pivot adalah host yang dipakai sebagai batu loncatan untuk menjangkau jaringan lain.

Routed traffic yang tidak dikendalikan dapat membantu attacker melakukan pivot. Karena itu, host biasa sebaiknya tidak meneruskan trafik.

### Kapan pengecualian diperlukan?

Ada host yang memang perlu routing, seperti:

- router Linux;
- firewall gateway;
- VPN server;
- NAT gateway;
- container host dengan desain jaringan tertentu;
- lab jaringan;
- host virtualisasi tertentu.

Jika pengecualian diperlukan, aturan dan alasannya harus didokumentasikan. Dalam pendekatan CIS, pengecualian bukan berarti gagal, asalkan ada alasan bisnis/teknis, mitigasi, dan dokumentasi risiko.

---

# Ringkasan Hubungan Incoming, Outgoing, dan Routed

| Jenis trafik | Arah trafik | Pertanyaan keamanan utama | Sikap CIS secara konseptual |
|---|---|---|---|
| Incoming | Dari luar menuju host | Siapa yang boleh mengakses layanan di host? | Tolak secara default, izinkan hanya layanan sah |
| Outgoing | Dari host menuju luar | Ke mana host boleh membuat koneksi? | Pada profil lebih ketat, batasi dengan allowlist |
| Routed | Melewati host antar-interface | Apakah host boleh menjadi router? | Nonaktifkan/tolak secara default kecuali memang perlu |

Ketiganya membentuk kontrol firewall yang lengkap. Incoming melindungi host dari akses luar, outgoing membatasi perilaku host ke luar, dan routed mencegah host menjadi jalur lintas jaringan yang tidak dirancang.

---

# Alur Berpikir Konseptual Saat Mendesain Firewall Host

Untuk memahami Bab 4 secara utuh, administrator tidak boleh hanya menghafal “aktifkan UFW”. Yang lebih penting adalah memahami alur berpikir berikut:

## 1. Identifikasi fungsi host

Pertanyaan pertama:

> Host ini sebenarnya berfungsi sebagai apa?

Contoh jawaban:

- web server;
- database server;
- workstation;
- file server;
- DNS server;
- router;
- server aplikasi;
- server monitoring.

Fungsi host menentukan trafik apa yang sah.

## 2. Identifikasi layanan yang perlu menerima koneksi

Setelah fungsi host jelas, tentukan layanan yang perlu menerima koneksi masuk.

Misalnya web server mungkin membutuhkan HTTP dan HTTPS. Database server mungkin hanya membutuhkan port database dari application server tertentu. Workstation mungkin tidak perlu menerima koneksi masuk sama sekali.

## 3. Identifikasi koneksi keluar yang dibutuhkan

Koneksi keluar perlu dianalisis terutama jika profil keamanan tinggi diterapkan. Host mungkin membutuhkan DNS, NTP, update repository, backup, monitoring, atau API tertentu.

## 4. Identifikasi apakah host meneruskan trafik

Jika host bukan router, VPN gateway, NAT gateway, atau firewall khusus, maka routed traffic sebaiknya tidak aktif.

## 5. Terapkan default deny

Setelah kebutuhan sah diketahui, kebijakan default deny dapat diterapkan secara aman. Default deny tanpa inventaris kebutuhan dapat menyebabkan gangguan layanan.

## 6. Dokumentasikan allow rule

Setiap rule yang mengizinkan trafik harus memiliki alasan. Dokumentasi yang baik menjawab:

- rule ini untuk layanan apa;
- siapa sumbernya;
- apa tujuannya;
- port/protokol apa yang dipakai;
- siapa pemilik layanan;
- apakah rule masih diperlukan;
- kapan terakhir ditinjau.

## 7. Uji sebelum produksi penuh

Perubahan firewall harus diuji. Ini penting terutama untuk server remote, karena salah aturan dapat memutus akses administrasi atau menghentikan layanan penting.

---

# Kesalahan Umum dalam Konfigurasi Host-Based Firewall

## 1. Mengaktifkan firewall tanpa mengizinkan akses administrasi

Ini dapat menyebabkan administrator terkunci dari server. Dalam lingkungan remote, SSH harus dipikirkan sebelum firewall diaktifkan.

## 2. Membuka SSH dari mana saja

Mengizinkan SSH dari semua alamat memang mudah, tetapi berisiko. Akses administrasi sebaiknya dibatasi berdasarkan sumber yang sah.

## 3. Menganggap firewall router sudah cukup

Firewall router tidak selalu melindungi dari serangan internal, lateral movement, atau kesalahan konfigurasi host.

## 4. Tidak mengamankan IPv6

Jika IPv6 aktif tetapi rule firewall hanya memikirkan IPv4, host bisa tetap terekspos melalui IPv6.

## 5. Membiarkan outgoing bebas pada server sensitif

Server sensitif sebaiknya tidak memiliki akses keluar yang tidak terbatas. Koneksi keluar perlu diketahui dan dibatasi sesuai kebutuhan.

## 6. Mengaktifkan routing tanpa sadar

Host multi-interface dapat menjadi jalur antar-jaringan tanpa disadari. Ini dapat merusak segmentasi jaringan.

## 7. Menggunakan banyak tool firewall sekaligus

Menggabungkan UFW, nft manual, iptables script, dan tool lain tanpa koordinasi dapat menghasilkan aturan yang sulit diprediksi.

## 8. Tidak meninjau rule lama

Rule firewall yang dulu dibutuhkan belum tentu masih relevan. Rule lama yang dibiarkan dapat menjadi celah akses.

---

# Contoh Konseptual Penerapan pada Berbagai Jenis Host

## 1. Web Server

Web server biasanya perlu menerima HTTP dan HTTPS. Jika dikelola lewat SSH, SSH sebaiknya hanya boleh dari alamat admin atau jaringan manajemen.

Pola konseptual:

- incoming default deny;
- allow HTTP/HTTPS dari sumber yang sesuai;
- allow SSH terbatas;
- outgoing hanya untuk DNS, NTP, update, dan kebutuhan aplikasi;
- routed disabled.

## 2. Database Server

Database server seharusnya tidak terbuka untuk semua perangkat. Biasanya hanya application server yang boleh mengaksesnya.

Pola konseptual:

- incoming default deny;
- allow port database hanya dari application server;
- SSH hanya dari admin tertentu;
- outgoing sangat terbatas;
- routed disabled.

## 3. Workstation

Workstation biasanya lebih banyak membuat koneksi keluar daripada menerima koneksi masuk.

Pola konseptual:

- incoming default deny;
- outgoing bisa lebih longgar tergantung kebijakan;
- layanan sharing atau remote access hanya jika perlu;
- routed disabled;
- firewall tetap aktif walaupun perangkat berpindah jaringan.

## 4. Server Router atau VPN Gateway

Host ini memang perlu routed traffic, tetapi justru karena itu aturannya harus lebih ketat dan terdokumentasi.

Pola konseptual:

- incoming dibatasi untuk layanan manajemen dan VPN;
- routed traffic hanya antar-interface yang sah;
- rule forwarding harus spesifik;
- logging lebih penting;
- dokumentasi pengecualian diperlukan karena host memang berbeda dari server biasa.

---

# Hubungan Bab 4 dengan Bab Sebelumnya

## Hubungan dengan Bab 2 — Services

Bab 2 mengurangi layanan yang tidak perlu. Bab 4 membatasi akses jaringan terhadap layanan yang tetap berjalan.

Keduanya saling melengkapi. Service yang tidak perlu sebaiknya dimatikan. Service yang memang perlu tetap harus dibatasi aksesnya oleh firewall.

Contoh:

- Jika web server tidak dibutuhkan, service web server dimatikan.
- Jika web server dibutuhkan, service tetap berjalan tetapi firewall hanya membuka port yang diperlukan.

## Hubungan dengan Bab 3 — Network

Bab 3 mengatur perangkat jaringan, kernel module, dan parameter jaringan. Bab 4 mengatur kebijakan filtering trafik.

Contoh hubungan:

- Bab 3 memastikan IPv6 tidak menjadi jalur tidak terkontrol.
- Bab 4 memastikan firewall mengontrol trafik IPv4 dan IPv6.
- Bab 3 membatasi forwarding dan routing pada level kernel.
- Bab 4 membatasi routed traffic pada level firewall.

## Hubungan dengan Bab 5 — Access Control

Bab 5 membahas akses seperti SSH, sudo, PAM, dan akun pengguna. Firewall pada Bab 4 dapat membatasi siapa yang bisa mencapai layanan akses tersebut.

Contoh:

- SSH dikonfigurasi aman di Bab 5.
- Firewall membatasi sumber yang boleh mengakses SSH di Bab 4.

Jadi hardening SSH tidak cukup hanya pada konfigurasi SSH. Jalur jaringan menuju SSH juga harus dibatasi.

---

# Kesimpulan Bab 4

Bab 4 CIS Debian Linux 12 Benchmark mengajarkan bahwa setiap host perlu memiliki kontrol firewall lokal. Firewall lokal bukan pengganti firewall router, tetapi lapisan pertahanan tambahan yang bekerja langsung pada sistem Debian.

Konsep utama Bab 4 adalah:

1. firewall adalah kumpulan aturan untuk mengizinkan atau menolak paket jaringan;
2. Linux menggunakan mekanisme kernel seperti NFTables/Netfilter untuk filtering paket;
3. UFW digunakan sebagai frontend sederhana untuk mengelola firewall;
4. hanya satu metode firewall sebaiknya digunakan agar aturan tidak saling bertabrakan;
5. incoming traffic sebaiknya ditolak secara default;
6. outgoing traffic dapat dibatasi pada profil keamanan lebih tinggi;
7. routed traffic sebaiknya disabled atau deny kecuali host memang berfungsi sebagai router;
8. firewall lokal tetap penting walaupun sudah ada firewall jaringan;
9. konfigurasi firewall harus diuji agar tidak menyebabkan lockout atau gangguan layanan;
10. setiap rule yang mengizinkan trafik harus memiliki alasan dan dokumentasi.

Secara konseptual, Bab 4 bukan hanya tentang “mengaktifkan UFW”, tetapi tentang membangun pola pikir bahwa host tidak boleh menerima, mengirim, atau meneruskan trafik tanpa kebutuhan yang jelas.

---

# Ringkasan Cepat untuk Hafalan

- **Host-based firewall**: firewall yang berjalan langsung di host/server/workstation.
- **Firewall jaringan**: firewall di router, gateway, atau perimeter.
- **UFW**: frontend sederhana untuk mengelola firewall Linux.
- **NFTables/Netfilter**: mekanisme kernel Linux yang melakukan filtering paket.
- **Default deny**: semua trafik ditolak kecuali diizinkan secara eksplisit.
- **Incoming**: trafik dari luar menuju host.
- **Outgoing**: trafik dari host menuju luar.
- **Routed**: trafik yang melewati host untuk diteruskan antar-interface.
- **Incoming deny**: cocok sebagai baseline umum.
- **Outgoing deny**: lebih ketat, cocok untuk Level 2 atau sistem sensitif.
- **Routed disabled**: mencegah host menjadi router tanpa sengaja.
- **Risiko utama firewall remote**: administrator bisa terkunci jika akses SSH belum diizinkan.
- **Prinsip utama**: izinkan hanya komunikasi yang benar-benar dibutuhkan.
