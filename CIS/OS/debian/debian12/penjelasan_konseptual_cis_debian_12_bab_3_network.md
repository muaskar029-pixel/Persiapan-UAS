# Penjelasan Konseptual CIS Debian Linux 12 Benchmark  
## D. Bab 3 — Network

**Sumber utama:** *CIS Debian Linux 12 Benchmark v2.0.0 — 05-28-2026*  
**Fokus pembahasan:** konseptual, bukan praktik command.  
**Ruang lingkup:** Bab 3 — Network, meliputi konfigurasi perangkat jaringan, kernel module jaringan, dan parameter kernel IPv4/IPv6.

---

## Orientasi Bab 3 — Network

Bab **Network** dalam CIS Debian Linux 12 Benchmark membahas cara mengurangi risiko dari sisi jaringan sistem operasi. Jika Bab 1 banyak membahas fondasi sistem seperti filesystem, bootloader, proses, dan tampilan login, maka Bab 3 berfokus pada bagaimana sistem berkomunikasi dengan jaringan dan bagaimana kernel Linux menangani paket jaringan.

Secara konseptual, hardening jaringan tidak hanya berarti memasang firewall. Firewall memang penting, tetapi sebelum firewall dibahas, CIS terlebih dahulu mengarahkan agar sistem:

1. mengenali perangkat jaringan yang aktif;
2. menonaktifkan perangkat jaringan yang tidak dibutuhkan;
3. menonaktifkan protokol jaringan kernel yang jarang dipakai;
4. mengatur perilaku kernel terhadap paket IPv4 dan IPv6;
5. mencegah sistem menerima informasi routing berbahaya dari pihak luar;
6. memastikan fitur jaringan hanya aktif sesuai fungsi sistem.

Prinsip besarnya adalah **network attack surface reduction**. Semakin banyak perangkat, protokol, dan fitur jaringan yang aktif, semakin banyak pula kemungkinan jalur serangan. Sebaliknya, semakin sedikit komponen jaringan yang aktif, semakin mudah sistem dikontrol, diaudit, dan diamankan.

Dalam konteks server, prinsip ini menjadi lebih penting karena server biasanya selalu menyala, terhubung ke jaringan, dan memiliki layanan yang bisa diakses oleh pengguna atau sistem lain. Server yang tidak membutuhkan Wi-Fi, Bluetooth, atau protokol jaringan tertentu sebaiknya tidak membiarkan komponen tersebut aktif hanya karena tersedia secara default.

---

# 3.1 Configure Network Devices

## 3.1.0 Konsep Umum Konfigurasi Perangkat Jaringan

Perangkat jaringan adalah komponen yang memungkinkan sistem berkomunikasi dengan dunia luar, baik melalui kabel, Wi-Fi, Bluetooth, atau antarmuka virtual. Pada Linux, perangkat jaringan dapat berupa:

- Ethernet interface;
- wireless interface;
- Bluetooth adapter;
- virtual interface;
- bridge interface;
- tunnel interface;
- container interface;
- VPN interface.

Dalam hardening CIS, tidak semua perangkat jaringan dianggap perlu. Perangkat yang tidak digunakan justru menjadi risiko karena bisa membuka jalur komunikasi tambahan yang tidak dikontrol.

Contoh sederhana: sebuah server perpustakaan yang ditempatkan di ruang server dan hanya perlu diakses melalui kabel LAN tidak membutuhkan Wi-Fi dan Bluetooth. Jika Wi-Fi tetap aktif, maka ada kemungkinan sistem memiliki jalur jaringan kedua di luar arsitektur yang direncanakan. Jika Bluetooth tetap aktif, maka ada kemungkinan perangkat lokal di sekitar server mencoba melakukan pairing atau eksploitasi terhadap layanan Bluetooth.

CIS menekankan prinsip bahwa perangkat yang tidak digunakan sebaiknya dinonaktifkan. Ini bukan karena semua perangkat tersebut pasti berbahaya, tetapi karena perangkat yang tidak dibutuhkan tidak memberikan manfaat keamanan, sedangkan tetap menambah risiko.

---

## 3.1.1 Identifikasi Status IPv6

### Konsep IPv6

IPv6 adalah versi modern dari Internet Protocol yang dirancang untuk menggantikan keterbatasan IPv4, terutama keterbatasan jumlah alamat. IPv6 menggunakan alamat 128-bit, sehingga jumlah alamat yang tersedia jauh lebih besar daripada IPv4.

Namun dalam konteks hardening, pertanyaan utamanya bukan hanya “apakah IPv6 lebih baru?”, tetapi:

> Apakah IPv6 digunakan, dipahami, dikonfigurasi, dipantau, dan diamankan di sistem ini?

Jika organisasi menggunakan IPv6 secara resmi, maka IPv6 harus dikonfigurasi dengan benar. Jika organisasi tidak menggunakan IPv6, maka status IPv6 tetap harus diketahui agar tidak menjadi jalur komunikasi tersembunyi.

### Kenapa status IPv6 perlu diidentifikasi?

Banyak lingkungan masih berfokus pada IPv4. Firewall, monitoring, logging, dokumentasi IP, dan aturan akses sering kali dibuat berdasarkan IPv4. Masalah muncul ketika IPv6 ternyata aktif secara default tetapi tidak ikut dikontrol.

Risikonya antara lain:

- sistem bisa menerima trafik IPv6 tanpa disadari;
- firewall IPv4 sudah ketat, tetapi IPv6 masih longgar;
- monitoring hanya membaca IPv4 sehingga trafik IPv6 tidak terlihat;
- attacker dapat memanfaatkan dual-stack environment;
- sistem menerima router advertisement IPv6 dari jaringan yang tidak tepercaya;
- audit keamanan menjadi tidak lengkap karena hanya memeriksa IPv4.

Karena itu, CIS tidak sekadar mengatakan “matikan IPv6”. Pendekatan yang lebih tepat adalah **identifikasi status IPv6**, lalu putuskan berdasarkan kebutuhan sistem dan kebijakan organisasi.

### IPv6 tidak selalu harus dimatikan

Dalam beberapa sistem modern, IPv6 memang digunakan. Misalnya:

- lingkungan cloud;
- jaringan kampus atau enterprise yang sudah dual-stack;
- layanan publik modern;
- sistem yang membutuhkan integrasi dengan jaringan IPv6;
- aplikasi yang secara eksplisit membutuhkan IPv6.

Jika IPv6 digunakan, maka ia harus diamankan. Jika tidak digunakan, maka organisasi bisa memilih menonaktifkannya atau memastikan seluruh kontrol keamanan IPv6 tetap diterapkan.

### Kesimpulan konseptual

Poin CIS tentang IPv6 mengajarkan bahwa keamanan jaringan dimulai dari **visibility**. Kita tidak bisa mengamankan sesuatu yang tidak kita ketahui statusnya. Maka sebelum berbicara firewall, routing, atau audit, administrator harus tahu apakah IPv6 aktif atau tidak.

---

## 3.1.2 Risiko Wireless Interface pada Server

### Apa itu wireless interface?

Wireless interface adalah antarmuka jaringan nirkabel, misalnya Wi-Fi. Pada laptop atau workstation, Wi-Fi sering dibutuhkan. Namun pada server, terutama server produksi, Wi-Fi biasanya tidak diperlukan karena server umumnya menggunakan koneksi kabel yang lebih stabil dan lebih mudah dikontrol.

### Kenapa Wi-Fi berisiko pada server?

Wi-Fi memperluas permukaan serangan karena komunikasi tidak lagi terbatas pada kabel fisik. Sinyal Wi-Fi dapat dijangkau dari area sekitar, tergantung kekuatan sinyal dan kondisi lingkungan. Ini membuat risiko serangan menjadi lebih luas secara fisik.

Risiko konseptualnya antara lain:

- attacker tidak perlu menyentuh kabel jaringan;
- server dapat terhubung ke jaringan yang salah;
- konfigurasi Wi-Fi bisa lemah atau lupa diamankan;
- kredensial Wi-Fi dapat bocor;
- interface wireless dapat menjadi jalur bypass dari segmentasi jaringan kabel;
- monitoring jaringan kabel tidak akan melihat seluruh aktivitas jika server juga aktif lewat Wi-Fi.

### Wireless sebagai jalur bypass

Misalnya server sudah ditempatkan di VLAN khusus dengan firewall internal yang ketat. Tetapi jika Wi-Fi server aktif dan terhubung ke jaringan lain, maka desain segmentasi itu bisa gagal. Attacker mungkin tidak perlu melewati firewall yang benar karena ada jalur lain melalui Wi-Fi.

Ini disebut **bypass path**, yaitu jalur alternatif yang tidak dirancang dalam arsitektur keamanan.

### Kapan wireless boleh aktif?

Wireless dapat dipertimbangkan jika benar-benar ada kebutuhan operasional. Contohnya:

- perangkat mobile atau workstation;
- sistem lab sementara;
- perangkat edge tertentu;
- kondisi darurat ketika kabel tidak tersedia.

Namun untuk server produksi, wireless harus diperlakukan sebagai pengecualian yang perlu alasan kuat, dokumentasi, dan kontrol tambahan.

---

## 3.1.3 Risiko Bluetooth Service

### Apa itu Bluetooth dalam konteks server?

Bluetooth adalah teknologi komunikasi jarak dekat. Pada desktop atau laptop, Bluetooth digunakan untuk keyboard, mouse, headset, transfer file, atau perangkat lain. Pada server, Bluetooth hampir selalu tidak dibutuhkan.

### Kenapa Bluetooth menjadi risiko?

Bluetooth membuka jalur komunikasi lokal. Artinya, ancaman tidak harus datang dari internet; bisa datang dari pihak yang berada secara fisik di sekitar perangkat.

Risiko Bluetooth antara lain:

- pairing dengan perangkat tidak dikenal;
- eksploitasi stack Bluetooth;
- penyalahgunaan perangkat input seperti keyboard atau mouse;
- transfer data yang tidak diawasi;
- service Bluetooth berjalan tanpa kebutuhan bisnis;
- peningkatan kompleksitas audit karena ada kanal komunikasi tambahan.

### Bluetooth dan physical proximity attack

Serangan Bluetooth biasanya bergantung pada kedekatan fisik. Namun pada server, ini tetap penting. Jika server berada di ruang yang tidak sepenuhnya steril, atau berupa laptop/server kecil yang digunakan dalam lab, Bluetooth aktif dapat menjadi celah.

CIS mendorong agar Bluetooth service tidak digunakan jika tidak dibutuhkan. Ini sesuai prinsip:

> Komponen yang tidak diperlukan sebaiknya tidak aktif.

### Kesimpulan perangkat jaringan

Pada bagian Configure Network Devices, CIS mengajarkan bahwa keamanan jaringan bukan hanya soal port TCP/UDP. Keamanan dimulai dari perangkat apa saja yang memberi sistem kemampuan berkomunikasi. Wi-Fi dan Bluetooth mungkin tampak sepele, tetapi pada server keduanya dapat menjadi jalur tambahan yang sulit diawasi.

---

# 3.2 Configure Network Kernel Modules

## 3.2.0 Konsep Network Kernel Module

Kernel Linux memiliki dukungan terhadap banyak protokol jaringan. Dukungan ini sering disediakan melalui **kernel module**. Kernel module adalah komponen yang dapat dimuat ke kernel untuk menambah kemampuan tertentu tanpa harus membangun ulang kernel.

Dalam konteks jaringan, kernel module dapat menambahkan dukungan terhadap protokol tertentu. Masalahnya, tidak semua protokol dibutuhkan oleh semua sistem. Banyak protokol dibuat untuk kebutuhan khusus seperti telekomunikasi, cluster, embedded system, atau jaringan lama.

Jika modul-modul ini tetap tersedia, sistem memiliki kemampuan untuk menggunakan protokol tersebut meskipun organisasi tidak membutuhkannya. Dari sudut pandang keamanan, kemampuan yang tidak dibutuhkan bisa menjadi risiko.

### Kenapa protokol jaringan yang tidak dipakai perlu dinonaktifkan?

Ada beberapa alasan:

1. **Mengurangi attack surface**  
   Semakin banyak protokol yang didukung, semakin banyak kode kernel yang bisa menjadi target eksploitasi.

2. **Mencegah penyalahgunaan protokol asing**  
   Attacker dapat mencoba menggunakan protokol yang jarang dimonitor oleh administrator.

3. **Menyederhanakan audit**  
   Jika hanya protokol yang dibutuhkan yang aktif, pemeriksaan keamanan menjadi lebih mudah.

4. **Mengurangi risiko dari bug kernel module**  
   Modul yang jarang digunakan tetap bisa memiliki kerentanan.

5. **Menerapkan prinsip least functionality**  
   Sistem hanya memiliki fungsi yang diperlukan untuk menjalankan tugasnya.

### Module availability secara konseptual

CIS tidak hanya peduli apakah modul sedang aktif. CIS juga peduli apakah modul **tersedia untuk dimuat**. Ada perbedaan penting:

- **Loaded** berarti modul sedang aktif di kernel.
- **Loadable** berarti modul belum aktif, tetapi masih bisa dimuat kapan saja.
- **Not available / disabled** berarti modul tidak tersedia atau sudah dicegah agar tidak dapat dimuat.

Dari sisi keamanan, modul yang tidak sedang aktif tetapi masih bisa dimuat tetap memiliki risiko. Karena itu, hardening sering mencakup pencegahan agar modul tertentu tidak dapat dimuat.

---

## 3.2.1 atm

### Apa itu ATM?

ATM adalah singkatan dari **Asynchronous Transfer Mode**. Ini adalah teknologi jaringan yang dahulu digunakan untuk komunikasi berkecepatan tinggi, terutama dalam infrastruktur telekomunikasi dan jaringan tertentu.

Pada server Linux modern biasa, ATM hampir tidak pernah dibutuhkan. Jika sistem tidak secara eksplisit menggunakan jaringan ATM, maka dukungan kernel untuk ATM tidak memberikan manfaat.

### Risiko konseptual

Risiko dari membiarkan modul ATM tersedia adalah:

- menambah kode jaringan yang tidak dibutuhkan;
- membuka kemungkinan penggunaan protokol yang tidak dimonitor;
- memperluas attack surface kernel;
- menambah kompleksitas konfigurasi jaringan.

### Sikap hardening

Jika server tidak memiliki kebutuhan ATM, modul ATM sebaiknya dinonaktifkan. Ini bukan karena ATM selalu berbahaya, tetapi karena sistem yang aman tidak mempertahankan fungsi yang tidak relevan.

---

## 3.2.2 can

### Apa itu CAN?

CAN adalah singkatan dari **Controller Area Network**. Protokol ini banyak digunakan dalam sistem kendaraan, perangkat industri, embedded system, dan perangkat kontrol. CAN bukan protokol umum untuk server enterprise biasa.

### Kenapa CAN tidak relevan untuk kebanyakan server?

Server Debian yang digunakan sebagai web server, database server, file server, atau server aplikasi biasanya tidak berkomunikasi dengan jaringan CAN. Jika modul CAN tetap tersedia, maka kernel memiliki fitur komunikasi yang tidak berkaitan dengan fungsi server.

### Risiko konseptual

- memperluas kemampuan jaringan yang tidak diperlukan;
- dapat dipakai dalam skenario khusus oleh attacker jika ada akses lokal;
- menambah permukaan kode kernel yang dapat dieksploitasi;
- menyulitkan audit karena protokol ini tidak umum diperiksa dalam jaringan IT biasa.

### Sikap hardening

Pada server umum, CAN sebaiknya dinonaktifkan kecuali ada kebutuhan eksplisit, misalnya sistem riset kendaraan, laboratorium embedded, atau gateway industri.

---

## 3.2.3 dccp

### Apa itu DCCP?

DCCP adalah singkatan dari **Datagram Congestion Control Protocol**. Protokol ini dirancang untuk aplikasi yang membutuhkan pengiriman datagram dengan kontrol kongesti, tetapi tidak memerlukan reliabilitas seperti TCP.

DCCP bukan protokol yang umum digunakan pada server Linux biasa.

### Kenapa DCCP dinonaktifkan jika tidak dipakai?

Karena DCCP menambah dukungan protokol jaringan tambahan di kernel. Jika aplikasi di server tidak menggunakan DCCP, maka tidak ada alasan kuat untuk membiarkannya tersedia.

### Risiko konseptual

- trafik DCCP mungkin tidak diawasi oleh firewall atau IDS lama;
- administrator mungkin tidak menyadari protokol ini aktif;
- bug pada implementasi protokol dapat menjadi risiko;
- attacker dapat memanfaatkan protokol jarang dipakai untuk bypass monitoring.

### Sikap hardening

DCCP sebaiknya dinonaktifkan pada sistem yang tidak membutuhkannya. Jika ada aplikasi khusus yang menggunakan DCCP, maka penggunaan tersebut harus didokumentasikan sebagai kebutuhan sistem.

---

## 3.2.4 rds

### Apa itu RDS?

RDS adalah singkatan dari **Reliable Datagram Sockets**. RDS digunakan untuk komunikasi datagram yang andal, terutama dalam skenario tertentu seperti cluster, database, atau lingkungan dengan RDMA.

Pada server umum, RDS biasanya tidak digunakan.

### Risiko konseptual

RDS bekerja pada level kernel. Jika tidak diperlukan, keberadaannya menambah permukaan serangan. Modul jaringan kernel memiliki risiko lebih sensitif dibanding aplikasi biasa karena berjalan sangat dekat dengan inti sistem operasi.

Risiko RDS yang tidak digunakan:

- memperluas attack surface kernel;
- memungkinkan protokol yang tidak diawasi;
- menambah kompleksitas jaringan;
- berpotensi dieksploitasi jika ada kerentanan pada implementasinya.

### Sikap hardening

RDS sebaiknya dinonaktifkan kecuali server memang memiliki kebutuhan cluster atau database tertentu yang membutuhkannya.

---

## 3.2.5 sctp

### Apa itu SCTP?

SCTP adalah singkatan dari **Stream Control Transmission Protocol**. SCTP adalah protokol transport seperti TCP dan UDP, tetapi memiliki fitur seperti multistreaming dan multihoming. SCTP sering ditemukan dalam telekomunikasi, signaling, dan aplikasi khusus.

### Kenapa SCTP perlu diperhatikan?

SCTP bukan protokol umum untuk mayoritas server web, database, atau file server. Jika tidak digunakan, SCTP menjadi fungsi tambahan yang tidak diperlukan.

### Risiko konseptual

- membuka jalur transport selain TCP/UDP;
- dapat lolos dari perhatian administrator yang hanya memonitor TCP/UDP;
- firewall atau kebijakan jaringan mungkin tidak mempertimbangkan SCTP;
- menambah attack surface kernel.

### Sikap hardening

Jika organisasi tidak menggunakan SCTP, modul SCTP sebaiknya dinonaktifkan. Namun jika sistem telekomunikasi atau aplikasi tertentu membutuhkannya, SCTP tidak boleh dimatikan secara sembarangan.

---

## 3.2.6 tipc

### Apa itu TIPC?

TIPC adalah singkatan dari **Transparent Inter-Process Communication**. TIPC digunakan untuk komunikasi antarproses dalam lingkungan cluster atau sistem terdistribusi tertentu.

### Kenapa TIPC jarang dibutuhkan?

Sebagian besar server tidak menggunakan TIPC. Server web, database standar, layanan SSH, DNS, atau aplikasi perpustakaan digital biasanya tidak memerlukan protokol ini.

### Risiko konseptual

- menambah protokol jaringan khusus di kernel;
- dapat menjadi jalur komunikasi yang tidak dikenal administrator;
- dapat memperluas permukaan serangan pada sistem cluster;
- sulit diaudit jika tidak termasuk dalam desain arsitektur jaringan.

### Sikap hardening

Jika tidak ada kebutuhan cluster yang secara eksplisit menggunakan TIPC, modul ini sebaiknya dinonaktifkan.

---

## Ringkasan 3.2

Bagian Network Kernel Modules mengajarkan bahwa hardening jaringan bukan hanya soal menutup port. Protokol yang didukung kernel juga perlu diperhatikan. Protokol seperti ATM, CAN, DCCP, RDS, SCTP, dan TIPC memiliki kegunaan masing-masing, tetapi tidak semua server membutuhkannya.

Prinsip konseptualnya:

> Jika sistem tidak membutuhkan suatu protokol jaringan, jangan biarkan kernel menyediakan protokol itu secara bebas.

---

# 3.3 Configure Network Kernel Parameters

## 3.3.0 Konsep Network Kernel Parameters

Network kernel parameters adalah pengaturan kernel yang mengontrol perilaku stack jaringan. Di Linux, pengaturan ini sering dikelola melalui mekanisme `sysctl`. Secara konseptual, parameter ini menentukan bagaimana kernel merespons paket jaringan, routing, forwarding, ICMP, source routing, dan fitur perlindungan tertentu.

Jika firewall mengatur paket mana yang boleh masuk atau keluar, maka parameter kernel menentukan bagaimana sistem memahami dan memproses paket tersebut.

Contoh perilaku yang dikontrol oleh parameter kernel:

- apakah sistem meneruskan paket seperti router;
- apakah sistem menerima redirect routing dari pihak luar;
- apakah sistem mengabaikan paket ICMP tertentu;
- apakah sistem mencatat paket mencurigakan;
- apakah sistem menerima source-routed packet;
- apakah sistem mengaktifkan perlindungan terhadap SYN flood.

### Kenapa parameter kernel penting?

Karena beberapa risiko jaringan tidak selalu berasal dari service yang listening. Risiko juga bisa muncul dari cara kernel memperlakukan paket jaringan.

Misalnya:

- host biasa tidak seharusnya menjadi router;
- server tidak seharusnya menerima instruksi routing dari pihak luar;
- paket source routing dapat dimanfaatkan untuk manipulasi jalur paket;
- broadcast ICMP dapat dipakai dalam serangan amplifikasi;
- paket dengan alamat sumber tidak masuk akal dapat menjadi indikasi spoofing.

### Hubungan dengan konfigurasi permanen

Secara konseptual, parameter kernel memiliki dua sisi:

1. **Running configuration**  
   Nilai yang sedang aktif di kernel saat ini.

2. **Persistent configuration**  
   Nilai yang akan tetap berlaku setelah reboot.

Hardening yang baik tidak cukup hanya mengubah nilai aktif sementara. Pengaturan harus konsisten setelah sistem restart.

---

# 3.3.1 IPv4 Parameters

## Gambaran Umum IPv4 Parameters

IPv4 parameters mengatur perilaku stack IPv4. Pada Debian Linux, IPv4 masih sangat penting karena banyak jaringan, aplikasi, dan layanan masih memakai IPv4 sebagai protokol utama.

CIS mengatur beberapa aspek penting:

- forwarding;
- ICMP redirects;
- bogus error responses;
- broadcast ICMP;
- source routing;
- reverse path filtering;
- martian packet logging;
- TCP SYN cookies.

Tujuannya adalah memastikan server tidak bertindak sebagai router tanpa sengaja, tidak menerima informasi routing berbahaya, tidak mudah dipakai dalam serangan jaringan, dan mampu mencatat paket mencurigakan.

---

## 3.3.1.1 IP Forwarding

### Konsep IP forwarding

IP forwarding adalah kemampuan sistem untuk meneruskan paket dari satu interface ke interface lain. Jika fitur ini aktif, sistem dapat berperan sebagai router.

Pada router, firewall gateway, VPN gateway, atau server container tertentu, IP forwarding mungkin dibutuhkan. Namun pada server biasa, IP forwarding biasanya tidak diperlukan.

### Risiko jika IP forwarding aktif tanpa kebutuhan

Jika server biasa memiliki IP forwarding aktif, maka server dapat menjadi jalur transit paket. Ini berbahaya karena:

- server bisa menjadi router tidak resmi;
- segmentasi jaringan dapat terlewati;
- attacker dapat menggunakan server sebagai pivot;
- trafik dapat melewati kontrol jaringan yang seharusnya;
- audit jaringan menjadi sulit karena server bertindak di luar perannya.

### Sikap hardening

Untuk host biasa, IP forwarding sebaiknya dimatikan. Jika server memang berfungsi sebagai router, NAT gateway, Kubernetes node, Docker host dengan kebutuhan tertentu, atau VPN gateway, maka pengaturan ini harus disesuaikan dengan dokumentasi kebutuhan.

### Catatan konseptual

Pada lingkungan container atau Kubernetes, beberapa fitur forwarding bisa dibutuhkan. Karena itu, jangan mematikan forwarding secara membabi buta pada sistem yang memang didesain untuk meneruskan trafik.

---

## 3.3.1.2 ICMP Redirects

### Apa itu ICMP redirect?

ICMP redirect adalah pesan yang memberi tahu host bahwa ada jalur routing yang lebih baik untuk mencapai tujuan tertentu. Dalam jaringan yang sangat tepercaya, mekanisme ini bisa membantu efisiensi routing.

Namun dari sudut pandang keamanan, menerima redirect dari luar dapat berbahaya.

### Risiko ICMP redirect

Jika sistem menerima ICMP redirect, pihak luar dapat mencoba memengaruhi tabel routing host. Dampaknya:

- trafik bisa diarahkan melalui gateway berbahaya;
- attacker dapat melakukan man-in-the-middle;
- jalur komunikasi dapat dimanipulasi;
- host dapat mempercayai informasi routing yang tidak valid;
- segmentasi jaringan dapat terganggu.

### Send redirects vs accept redirects

Ada dua sisi ICMP redirect:

1. **Send redirects**  
   Sistem mengirim pesan redirect ke host lain.

2. **Accept redirects**  
   Sistem menerima pesan redirect dari pihak lain.

Untuk server biasa, keduanya umumnya tidak diperlukan. Host non-router tidak seharusnya mengirim redirect, dan host yang aman tidak seharusnya menerima instruksi routing dari pihak luar.

### Secure redirects

Secure redirects adalah bentuk redirect yang hanya diterima dari gateway tertentu. Namun dalam hardening ketat, fitur ini tetap sering dinonaktifkan karena tetap memberi peluang perubahan routing dari luar.

---

## 3.3.1.3 Bogus Error Responses

### Apa itu bogus ICMP error response?

Bogus error response adalah pesan kesalahan ICMP yang tidak valid, tidak sesuai konteks, atau mencurigakan. Paket semacam ini bisa muncul karena salah konfigurasi jaringan, tetapi juga bisa digunakan dalam aktivitas scanning, gangguan, atau serangan.

### Kenapa perlu diabaikan?

Jika sistem memproses error response yang tidak valid, sistem dapat membuang waktu dan sumber daya untuk merespons paket yang tidak berguna atau berbahaya.

Mengabaikan bogus error response membantu:

- mengurangi noise jaringan;
- mencegah reaksi sistem terhadap paket tidak valid;
- mengurangi peluang manipulasi perilaku jaringan;
- meningkatkan ketahanan terhadap paket aneh atau crafted packet.

### Kesimpulan

Parameter ini memperkuat prinsip bahwa sistem tidak harus mempercayai semua pesan jaringan yang diterimanya. Paket yang tidak valid sebaiknya diabaikan.

---

## 3.3.1.4 Broadcast ICMP

### Apa itu broadcast ICMP?

Broadcast ICMP terjadi ketika paket ICMP, misalnya ping, dikirim ke alamat broadcast jaringan. Jika banyak host merespons broadcast tersebut, trafik balasan dapat menjadi besar.

### Risiko broadcast ICMP

Broadcast ICMP dapat dipakai dalam serangan amplifikasi. Dalam skenario tertentu, attacker mengirim paket ke alamat broadcast dengan spoofed source address. Banyak host kemudian merespons ke alamat korban, sehingga korban menerima banjir trafik.

Risikonya:

- sistem ikut menjadi bagian dari serangan amplifikasi;
- jaringan menjadi sibuk karena respons tidak perlu;
- attacker dapat melakukan reconnaissance;
- server merespons trafik yang tidak memiliki kebutuhan operasional.

### Sikap hardening

Server sebaiknya mengabaikan ICMP echo request yang dikirim ke broadcast. Host masih dapat merespons ping langsung jika kebijakan mengizinkan, tetapi tidak perlu merespons broadcast ping.

---

## 3.3.1.5 Source Routing

### Apa itu source routing?

Source routing adalah mekanisme yang memungkinkan pengirim paket menentukan jalur yang harus dilewati paket tersebut. Dalam jaringan modern, routing seharusnya ditentukan oleh perangkat routing tepercaya, bukan oleh pengirim paket.

### Risiko source routing

Source routing berbahaya karena attacker dapat mencoba mengatur jalur paket untuk:

- melewati kontrol keamanan;
- menghindari firewall tertentu;
- memanipulasi jalur komunikasi;
- melakukan spoofing;
- membantu serangan man-in-the-middle.

### Sikap hardening

Host sebaiknya tidak menerima source-routed packet. Server tidak perlu memberi pengirim eksternal kemampuan menentukan jalur routing.

### Kesimpulan

Menonaktifkan source routing adalah bagian dari prinsip bahwa sistem tidak boleh mempercayai instruksi routing dari sumber yang tidak tepercaya.

---

## 3.3.1.6 Reverse Path Filtering

### Apa itu reverse path filtering?

Reverse path filtering adalah mekanisme untuk memeriksa apakah alamat sumber suatu paket masuk akal jika dilihat dari jalur balik routing. Dengan kata lain, sistem memeriksa apakah paket datang dari interface yang sesuai dengan rute balik menuju alamat sumbernya.

### Kenapa penting?

Reverse path filtering membantu mendeteksi dan menolak paket dengan alamat sumber palsu atau tidak masuk akal. Ini berguna untuk mengurangi IP spoofing.

### Risiko jika tidak aktif

Jika reverse path filtering tidak aktif, sistem lebih mudah menerima paket dengan alamat sumber palsu. Ini dapat digunakan untuk:

- spoofing;
- reconnaissance;
- bypass aturan berbasis alamat sumber;
- serangan reflected atau amplified;
- manipulasi trafik.

### Catatan lingkungan khusus

Pada jaringan dengan routing asimetris, reverse path filtering dapat menyebabkan paket valid ditolak karena jalur masuk dan jalur balik berbeda. Karena itu, pada lingkungan routing kompleks, pengaturan ini harus diuji dan disesuaikan.

### Sikap hardening

Untuk host umum, reverse path filtering sebaiknya aktif. Untuk router atau sistem dengan routing kompleks, perlu analisis jaringan lebih detail.

---

## 3.3.1.7 Martian Packet Logging

### Apa itu martian packet?

Martian packet adalah paket yang memiliki alamat sumber atau tujuan yang tidak masuk akal menurut topologi jaringan. Misalnya, paket dari alamat private muncul di tempat yang tidak seharusnya, atau paket memiliki alamat sumber yang tidak valid.

### Kenapa perlu dicatat?

Martian packet dapat menjadi tanda:

- IP spoofing;
- salah konfigurasi routing;
- perangkat jaringan bermasalah;
- percobaan scanning;
- aktivitas attacker;
- kebocoran trafik dari segmentasi yang salah.

Dengan mencatat martian packet, administrator mendapat sinyal bahwa ada sesuatu yang tidak normal di jaringan.

### Hubungan dengan audit dan forensik

Logging martian packet membantu investigasi. Jika terjadi insiden, catatan ini dapat menunjukkan kapan paket aneh mulai muncul, dari interface mana, dan pola seperti apa yang terjadi.

### Sikap hardening

CIS mendorong agar sistem mencatat paket mencurigakan semacam ini, karena visibility sangat penting dalam keamanan jaringan.

---

## 3.3.1.8 TCP SYN Cookies

### Apa itu TCP SYN?

Dalam koneksi TCP, proses awal koneksi disebut three-way handshake. Salah satu tahap awalnya adalah paket SYN. Dalam serangan SYN flood, attacker mengirim banyak SYN untuk membuat server menyimpan banyak koneksi setengah terbuka.

### Apa itu SYN cookies?

SYN cookies adalah mekanisme perlindungan yang membantu sistem menghadapi SYN flood. Dengan SYN cookies, kernel dapat menangani permintaan koneksi tanpa harus langsung menyimpan semua state koneksi setengah terbuka dalam jumlah besar.

### Kenapa penting?

SYN cookies membantu menjaga ketersediaan layanan saat sistem menerima banyak koneksi palsu. Ini berkaitan dengan aspek availability dalam keamanan informasi.

### Risiko jika tidak aktif

Jika SYN cookies tidak aktif, sistem lebih rentan terhadap SYN flood karena resource koneksi dapat habis oleh permintaan palsu.

### Sikap hardening

SYN cookies sebaiknya aktif agar server memiliki perlindungan dasar terhadap jenis serangan denial-of-service tertentu.

---

## Ringkasan IPv4 Parameters

| Area | Tujuan Konseptual |
|---|---|
| IP forwarding | Mencegah host biasa bertindak sebagai router tanpa izin |
| Send redirects | Mencegah host mengirim instruksi routing yang tidak perlu |
| Accept redirects | Mencegah host menerima perubahan routing dari luar |
| Bogus error responses | Mengabaikan pesan error jaringan yang tidak valid |
| Broadcast ICMP | Mencegah sistem ikut serangan amplifikasi |
| Source routing | Mencegah pengirim paket menentukan jalur routing |
| Reverse path filtering | Mengurangi risiko IP spoofing |
| Martian logging | Meningkatkan visibility terhadap paket mencurigakan |
| SYN cookies | Melindungi dari SYN flood |

---

# 3.3.2 IPv6 Parameters

## Gambaran Umum IPv6 Parameters

IPv6 parameters mengatur perilaku stack IPv6. CIS menekankan bahwa jika IPv6 dinonaktifkan, rekomendasi IPv6 tidak berlaku. Tetapi jika IPv6 aktif, maka IPv6 harus diamankan secara serius seperti IPv4.

Kesalahan umum dalam banyak organisasi adalah menganggap IPv6 tidak dipakai, padahal aktif secara default. Akibatnya:

- firewall IPv6 tidak dikonfigurasi;
- monitoring IPv6 tidak aktif;
- host menerima router advertisement;
- sistem membuka jalur komunikasi yang tidak diketahui;
- audit hanya memeriksa IPv4.

Karena itu, Bab 3 membahas IPv6 secara khusus agar keamanan jaringan tidak pincang.

---

## 3.3.2.1 IPv6 Forwarding

### Konsep IPv6 forwarding

IPv6 forwarding adalah kemampuan sistem meneruskan paket IPv6 antarinterface. Sama seperti IPv4 forwarding, fitur ini membuat sistem berperan seperti router.

### Risiko jika aktif tanpa kebutuhan

Jika server biasa meneruskan paket IPv6, maka:

- server dapat menjadi router tidak resmi;
- segmentasi jaringan IPv6 dapat dilewati;
- trafik IPv6 dapat melewati jalur yang tidak diaudit;
- attacker dapat memanfaatkan sistem sebagai pivot;
- kebijakan jaringan menjadi tidak konsisten.

### Sikap hardening

Jika sistem bukan router IPv6, IPv6 forwarding sebaiknya dimatikan. Jika sistem memang router, gateway, atau node jaringan khusus, maka pengecualian harus didokumentasikan.

---

## 3.3.2.2 IPv6 Redirects

### Apa itu IPv6 redirect?

IPv6 redirect adalah pesan ICMPv6 yang memberi tahu host tentang jalur routing yang dianggap lebih baik. Secara fungsi mirip dengan ICMP redirect pada IPv4.

### Risiko IPv6 redirects

Jika sistem menerima redirect IPv6 dari pihak luar, routing host dapat dipengaruhi. Dampaknya:

- trafik IPv6 dapat diarahkan ke gateway yang salah;
- attacker dapat mencoba man-in-the-middle;
- tabel routing dapat dimanipulasi;
- sistem mempercayai informasi routing dari sumber yang tidak seharusnya.

### Sikap hardening

Host biasa sebaiknya tidak menerima IPv6 redirects. Sistem harus bergantung pada konfigurasi routing yang tepercaya, bukan instruksi routing acak dari jaringan.

---

## 3.3.2.3 IPv6 Source Routing

### Apa itu IPv6 source routing?

Source routing pada IPv6 memungkinkan pengirim memengaruhi jalur yang dilalui paket. Dalam praktik modern, fitur semacam ini jarang diperlukan pada server biasa dan memiliki risiko keamanan.

### Risiko source routing di IPv6

- jalur paket dapat dimanipulasi;
- kontrol jaringan dapat dilewati;
- attacker dapat menyusun paket untuk melewati segmentasi;
- audit routing menjadi lebih sulit;
- dapat membantu serangan spoofing atau man-in-the-middle.

### Sikap hardening

Jika tidak ada kebutuhan khusus, sistem tidak seharusnya menerima IPv6 source-routed packet.

---

## 3.3.2.4 Router Advertisement

### Apa itu router advertisement?

Router Advertisement atau RA adalah mekanisme IPv6 yang memungkinkan router mengumumkan informasi jaringan kepada host. RA dapat memberi tahu host tentang prefix jaringan, default gateway, dan konfigurasi auto-configuration.

Dalam jaringan IPv6 normal, RA berguna. Namun pada server yang harus memiliki konfigurasi jaringan stabil dan terkontrol, RA perlu diperhatikan secara serius.

### Risiko router advertisement

Jika server menerima RA dari sumber yang tidak sah, dampaknya bisa besar:

- default gateway IPv6 berubah;
- host mendapatkan alamat IPv6 yang tidak direncanakan;
- trafik keluar diarahkan ke router palsu;
- attacker dapat melakukan man-in-the-middle;
- segmentasi jaringan terganggu;
- sistem terlihat aman di IPv4 tetapi terbuka di IPv6.

### Rogue Router Advertisement

Salah satu risiko terbesar IPv6 adalah **rogue router advertisement**, yaitu ketika perangkat tidak sah mengirim RA dan membuat host mempercayai router palsu.

Dalam lingkungan enterprise, RA harus dikontrol oleh perangkat jaringan yang sah. Server tidak boleh sembarang menerima RA jika alamat dan routing-nya dikelola secara statis atau melalui mekanisme resmi.

### Sikap hardening

Pada server, penerimaan RA umumnya perlu dibatasi atau dimatikan kecuali memang digunakan dalam desain jaringan. Jika server memakai IPv6 statis, menerima RA dari jaringan bisa menjadi risiko.

---

## Risiko Salah Konfigurasi IPv6

IPv6 sering menjadi celah bukan karena protokolnya lemah, tetapi karena administrator lupa mengelolanya. Beberapa risiko salah konfigurasi IPv6 adalah:

### 1. IPv6 aktif tetapi tidak diaudit

Sistem mungkin memiliki alamat IPv6 global atau link-local, tetapi monitoring hanya melihat IPv4. Ini membuat aktivitas IPv6 tidak terlihat.

### 2. Firewall hanya mengatur IPv4

Banyak administrator menulis aturan firewall untuk IPv4 tetapi lupa aturan IPv6. Akibatnya, service yang dibatasi di IPv4 bisa tetap terbuka lewat IPv6.

### 3. Router advertisement tidak dikontrol

Host dapat menerima gateway IPv6 dari perangkat yang tidak sah.

### 4. Dual-stack tidak konsisten

Sistem yang berjalan dual-stack harus memiliki kebijakan keamanan setara untuk IPv4 dan IPv6. Jika tidak, attacker akan memilih jalur yang lebih lemah.

### 5. Logging tidak lengkap

Jika log, SIEM, atau IDS tidak dikonfigurasi untuk IPv6, investigasi insiden menjadi tidak lengkap.

### 6. Kebijakan akses berbasis IP tidak mencakup IPv6

ACL yang hanya menuliskan alamat IPv4 tidak melindungi akses IPv6.

---

## Ringkasan IPv6 Parameters

| Area | Tujuan Konseptual |
|---|---|
| IPv6 forwarding | Mencegah host biasa menjadi router IPv6 |
| IPv6 redirects | Mencegah perubahan routing oleh pihak luar |
| IPv6 source routing | Mencegah pengirim paket menentukan jalur |
| Router advertisement | Mencegah host menerima konfigurasi router tidak sah |
| Identifikasi status IPv6 | Memastikan IPv6 tidak menjadi blind spot keamanan |

---

# Hubungan Bab 3 dengan Prinsip CIS Secara Umum

Bab Network sangat selaras dengan prinsip umum CIS Benchmark:

## 1. Least Functionality

Sistem hanya menjalankan fungsi yang diperlukan. Wireless, Bluetooth, dan protokol kernel yang tidak digunakan tidak perlu aktif.

## 2. Secure Configuration

Parameter kernel harus disetel agar sistem tidak mempercayai paket, redirect, atau routing information secara sembarangan.

## 3. Attack Surface Reduction

Setiap interface, protokol, dan fitur jaringan yang aktif adalah permukaan serangan. Mengurangi komponen aktif berarti mengurangi peluang eksploitasi.

## 4. Defense in Depth

Hardening jaringan tidak menggantikan firewall, EDR, logging, atau patching. Ia menjadi lapisan tambahan yang memperkuat sistem dari level kernel dan perangkat jaringan.

## 5. Auditability

Konfigurasi jaringan yang sederhana dan eksplisit lebih mudah diaudit. Auditor dapat memeriksa apakah perangkat, modul, dan parameter jaringan sesuai fungsi sistem.

---

# Contoh Cara Memahami Bab 3 dalam Skenario Server

Misalkan ada server Debian 12 yang digunakan sebagai server aplikasi perpustakaan. Server ini hanya perlu:

- menerima koneksi SSH dari admin;
- menerima HTTP/HTTPS dari reverse proxy atau client tertentu;
- mengakses repository update;
- mengirim log ke server log;
- menggunakan koneksi LAN kabel.

Maka secara konseptual:

- Wi-Fi tidak dibutuhkan;
- Bluetooth tidak dibutuhkan;
- ATM, CAN, DCCP, RDS, SCTP, dan TIPC tidak dibutuhkan;
- server tidak perlu menjadi router;
- server tidak perlu menerima ICMP redirect;
- server tidak perlu menerima source-routed packet;
- server perlu perlindungan SYN cookies;
- server perlu logging paket mencurigakan;
- IPv6 harus diputuskan: digunakan dan diamankan, atau dinonaktifkan sesuai kebijakan.

Dengan cara ini, Bab 3 tidak dipahami sebagai daftar perintah, melainkan sebagai proses berpikir:

> Apa fungsi jaringan yang benar-benar dibutuhkan server ini, dan fungsi apa yang harus dihilangkan agar risiko berkurang?

---

# Kesimpulan Bab 3 — Network

Bab 3 CIS Debian Linux 12 Benchmark menekankan bahwa keamanan jaringan dimulai dari pengurangan fungsi yang tidak diperlukan dan pengaturan perilaku kernel terhadap trafik jaringan.

Pokok pemahamannya adalah:

1. Perangkat jaringan yang tidak digunakan harus dinonaktifkan.
2. IPv6 harus diidentifikasi statusnya agar tidak menjadi blind spot.
3. Wireless dan Bluetooth pada server dapat menjadi jalur serangan tambahan.
4. Protokol kernel yang jarang dipakai sebaiknya tidak tersedia jika tidak diperlukan.
5. Host biasa tidak seharusnya meneruskan paket seperti router.
6. Sistem tidak boleh sembarang menerima redirect atau source routing.
7. Reverse path filtering membantu mengurangi spoofing.
8. Martian packet logging meningkatkan kemampuan deteksi dan forensik.
9. SYN cookies membantu menjaga availability saat menghadapi SYN flood.
10. IPv6 harus diamankan setara dengan IPv4 jika aktif.

Dengan memahami Bab 3 secara konseptual, administrator tidak hanya mengikuti checklist, tetapi memahami alasan keamanan di balik setiap rekomendasi. Inilah tujuan utama CIS Benchmark sebagai baseline hardening: membantu sistem berada pada konfigurasi awal yang lebih aman, rasional, terukur, dan dapat diaudit.

---

# Peta Pembahasan Singkat

| Subbab CIS | Fokus Konseptual | Inti Keamanan |
|---|---|---|
| 3.1 Configure Network Devices | Perangkat jaringan | Nonaktifkan perangkat yang tidak dibutuhkan |
| 3.1.1 IPv6 status | Visibility IPv6 | Jangan biarkan IPv6 menjadi blind spot |
| 3.1.2 Wireless interfaces | Wi-Fi server | Hindari jalur jaringan tambahan |
| 3.1.3 Bluetooth services | Komunikasi jarak dekat | Kurangi physical proximity attack surface |
| 3.2 Network Kernel Modules | Protokol kernel | Nonaktifkan protokol yang tidak dipakai |
| 3.3 IPv4 Parameters | Perilaku IPv4 | Cegah routing manipulation dan spoofing |
| 3.3 IPv6 Parameters | Perilaku IPv6 | Amankan IPv6 jika aktif |

---

**Catatan akhir:**  
Materi ini disusun untuk memahami konsep Bab 3 secara akademik dan konseptual. Implementasi teknis berupa command, file konfigurasi, audit, remediation, dan testing produksi sebaiknya dibahas terpisah agar tidak mencampur pemahaman konsep dengan praktik operasional.
