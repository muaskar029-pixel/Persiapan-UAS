# 03 - Network, SSH, dan Firewall Hardening

**Sumber utama:** CIS Ubuntu Linux 24.04 LTS STIG Benchmark v1.0.0, 06-12-2025.  
**Target dokumen:** modul belajar tematik untuk memahami hardening Ubuntu 24.04 LTS berbasis STIG.  
**Catatan penting:** file ini adalah dokumentasi pembelajaran dan panduan konseptual. Untuk audit resmi, tetap gunakan dokumen benchmark asli, hasil scanning, dan kebijakan organisasi.

---


## 1. Ruang lingkup modul

Modul ini membahas kontrol yang berkaitan dengan **UFW**, **remote access**, **SSH**, **kriptografi koneksi**, **timeout jaringan**, **X11 forwarding/proxy display**, **TCP syncookies**, **wireless adapter**, dan pembatasan port/protocol/service. Dalam STIG, network hardening bukan hanya soal firewall, tetapi juga memastikan jalur administrasi aman, sesi tidak dibiarkan terbuka, dan protokol yang tidak perlu tidak mengekspos sistem.

## 2. Peta aturan dalam modul ini

| Tema | Aturan terkait | Makna dalam hardening |
|---|---|---|
| Host-based firewall dan remote access control | `UBTU-24-100300` (Automated, CAT II), `UBTU-24-100310` (Automated, CAT II), `UBTU-24-600200` (Manual, CAT II) | UFW harus tersedia, aktif, dan mampu membatasi akses jaringan termasuk rate limit pada interface terdampak. |
| Monitoring remote access dan port/protocol/service | `UBTU-24-200090` (Automated, CAT II), `UBTU-24-300041` (Manual, CAT II) | Remote access dan layanan jaringan harus dipantau serta disesuaikan dengan daftar port/protocol/service yang disetujui. |
| SSH service dan kriptografi | `UBTU-24-100800` (Automated, CAT I), `UBTU-24-100810` (Automated, CAT I), `UBTU-24-100820` (Automated, CAT II), `UBTU-24-100830` (Automated, CAT II), `UBTU-24-100840` (Automated, CAT II), `UBTU-24-100850` (Automated, CAT II), `UBTU-24-100860` (Automated, CAT II) | SSH harus tersedia, aktif, dan menggunakan cipher, MAC, serta key exchange yang sesuai baseline STIG. |
| SSH session control | `UBTU-24-600000` (Automated, CAT II), `UBTU-24-600010` (Automated, CAT II), `UBTU-24-300031` (Automated, CAT I), `UBTU-24-200680` (Manual, CAT II) | Sesi SSH harus memiliki timeout, tidak boleh unattended login, dan harus menampilkan consent banner sesuai kebutuhan STIG. |
| X11/proxy display dan exposure GUI | `UBTU-24-300022` (Automated, CAT I), `UBTU-24-300023` (Automated, CAT II) | Remote X dan proxy display dibatasi karena dapat memperluas jalur akses antaraplikasi dan host. |
| Network kernel dan wireless | `UBTU-24-600190` (Automated, CAT II), `UBTU-24-600230` (Automated, CAT II) | TCP syncookies membantu mitigasi SYN flood, sedangkan wireless adapter dibatasi pada sistem yang tidak membutuhkan koneksi wireless. |

## 3. Host-based firewall

Host-based firewall adalah firewall yang berjalan langsung pada host. Pada Ubuntu, STIG menggunakan UFW sebagai pendekatan firewall lokal. Firewall jaringan di router atau perimeter tidak cukup, karena serangan bisa datang dari jaringan internal, lateral movement, VPN, host yang sudah terinfeksi, atau konfigurasi segmentasi yang tidak sempurna.

Aturan terkait: `UBTU-24-100300` (Automated, CAT II), `UBTU-24-100310` (Automated, CAT II).

Prinsip host firewall:

- deny traffic yang tidak dibutuhkan;
- allow hanya service yang disetujui;
- batasi koneksi masuk ke port administratif;
- dokumentasikan pengecualian;
- uji sebelum mengaktifkan agar tidak memutus akses admin.

UFW juga berkaitan dengan rate limiting pada interface terdampak. Rate limit bukan pengganti anti-DDoS, tetapi membantu mengurangi penyalahgunaan koneksi berulang, brute force, atau traffic yang terlalu agresif pada service tertentu.

## 4. Remote access dan port/protocol/service management

Remote access adalah jalur paling sensitif karena memberi admin kemampuan mengelola sistem dari luar konsol lokal. STIG menuntut remote access dimonitor dan fungsi/port/protocol/service dibatasi sesuai kebutuhan organisasi.

Aturan terkait: `UBTU-24-200090` (Automated, CAT II), `UBTU-24-300041` (Manual, CAT II).

Secara konseptual, daftar port terbuka harus bisa dijelaskan:

- port apa yang terbuka;
- service apa yang mendengarkan;
- siapa yang membutuhkan akses;
- dari jaringan mana akses diizinkan;
- apakah service itu server, client, atau komponen internal;
- apakah ada risiko protokol tidak terenkripsi;
- apakah ada pengecualian yang terdokumentasi.

Kontrol ini berhubungan dengan prinsip **only approved ports, protocols, and services**. Bukan berarti semua port harus ditutup, tetapi semua port terbuka harus punya alasan bisnis dan kontrol keamanan.

## 5. SSH sebagai jalur administrasi utama

SSH adalah mekanisme standar untuk remote administration pada Linux. Karena SSH memberi akses shell, kesalahan konfigurasi SSH dapat berdampak besar. STIG memperlakukan SSH sebagai kontrol kritis, termasuk memastikan SSH terpasang, service aktif, dan algoritma kriptografi dibatasi.

Aturan terkait service SSH: `UBTU-24-100800` (Automated, CAT I), `UBTU-24-100810` (Automated, CAT I).

SSH digunakan karena menyediakan confidentiality dan integrity untuk komunikasi administratif. Tanpa SSH atau mekanisme aman setara, kredensial dan data sesi bisa dibaca atau dimodifikasi saat transit.

## 6. SSH cipher, MAC, dan key exchange

STIG membatasi cipher, MAC, dan key exchange agar koneksi SSH menggunakan algoritma yang sesuai dengan ekspektasi kriptografi baseline. Tiga komponen ini memiliki fungsi berbeda:

- **Cipher** melindungi kerahasiaan data sesi.
- **MAC** membantu memastikan integritas pesan.
- **Key exchange** digunakan untuk membuat secret/session key secara aman antara client dan server.

Aturan terkait: `UBTU-24-100820` (Automated, CAT II), `UBTU-24-100830` (Automated, CAT II), `UBTU-24-100840` (Automated, CAT II), `UBTU-24-100850` (Automated, CAT II), `UBTU-24-100860` (Automated, CAT II).

Kesalahan umum dalam hardening SSH adalah hanya mengatur server tetapi melupakan client. STIG membahas keduanya. Server harus membatasi algoritma yang diterima dari client, sementara client juga harus dikonfigurasi agar tidak menggunakan algoritma yang tidak sesuai baseline saat terhubung ke host lain.

Konsep auditnya sederhana: cari konfigurasi SSH yang aktif, pastikan tidak ada baris konflik, pastikan baris tidak dikomentari, dan restart/reload service dengan hati-hati. Sebelum menerapkan perubahan SSH, selalu siapkan sesi cadangan atau akses konsol agar tidak terkunci keluar.

## 7. SSH timeout, unattended login, dan banner

Sesi administratif yang dibiarkan terbuka merupakan risiko. Jika admin meninggalkan terminal atau koneksi SSH tidak diputus saat idle, orang lain dapat menggunakan sesi tersebut tanpa autentikasi ulang. Karena itu, STIG mengatur timeout untuk koneksi SSH.

Aturan terkait: `UBTU-24-600000` (Automated, CAT II), `UBTU-24-600010` (Automated, CAT II), `UBTU-24-300031` (Automated, CAT I), `UBTU-24-200680` (Manual, CAT II).

Selain timeout, STIG juga menuntut consent banner untuk SSH. Banner bukan kontrol teknis untuk menghentikan attacker, tetapi berfungsi sebagai pemberitahuan legal dan kebijakan akses. Dalam audit compliance, banner membantu membuktikan bahwa user telah diberi notice sebelum menggunakan sistem.

Banner harus hati-hati. Banner yang menampilkan versi OS, nama host internal, detail aplikasi, atau informasi sensitif justru dapat membantu attacker melakukan reconnaissance.

## 8. X11 forwarding dan proxy display

Remote X connection dan proxy display dapat membuka jalur risiko tambahan. X11 forwarding memungkinkan aplikasi GUI berjalan di remote host tetapi tampil di client. Jika tidak dibutuhkan, fitur ini harus dinonaktifkan karena memperluas permukaan serangan dan dapat membuka peluang pencurian input/output GUI.

Aturan terkait: `UBTU-24-300022` (Automated, CAT I), `UBTU-24-300023` (Automated, CAT II).

Pada server, kebutuhan X11 biasanya kecil. Jika organisasi memang membutuhkan X11 forwarding untuk misi tertentu, pengecualian harus divalidasi dan didokumentasikan.

## 9. TCP syncookies

TCP syncookies adalah mitigasi kernel untuk membantu menghadapi SYN flood. SYN flood mencoba memenuhi antrean koneksi setengah terbuka sehingga service tidak dapat menerima koneksi sah. Syncookies membantu kernel menangani kondisi tersebut tanpa menyimpan semua state awal koneksi secara normal.

Aturan terkait: `UBTU-24-600190` (Automated, CAT II).

Ini bukan solusi lengkap untuk DDoS besar, tetapi merupakan hardening kernel dasar yang berguna pada host.

## 10. Wireless adapter

STIG meminta wireless network adapter dinonaktifkan jika tidak diperlukan. Pada server atau workstation sensitif, wireless dapat menjadi jalur akses tambahan yang sulit dipantau dibandingkan koneksi kabel tersegmentasi.

Aturan terkait: `UBTU-24-600230` (Automated, CAT II).

Risiko wireless meliputi:

- koneksi ke jaringan tidak tepercaya;
- rogue access point;
- bypass segmentasi kabel;
- konfigurasi WPA/enterprise yang salah;
- exposure perangkat pada area fisik yang lebih luas.

Jika wireless dibutuhkan, kebijakan harus menjelaskan alasan, metode autentikasi, enkripsi, segmentasi, dan monitoring.

## 11. Checklist pemahaman modul

- UFW tetap penting meskipun ada firewall jaringan.
- Semua port/protocol/service terbuka harus memiliki alasan dan dokumentasi.
- SSH harus aman dari sisi server dan client.
- Cipher, MAC, dan key exchange mengamankan aspek berbeda dari SSH.
- Timeout dan larangan unattended login mengurangi risiko sesi dicuri.
- Banner adalah notice legal/kebijakan, bukan tempat menampilkan informasi sistem.
- X11/proxy display sebaiknya dimatikan jika tidak ada kebutuhan misi.
- TCP syncookies membantu mitigasi SYN flood.
- Wireless adapter harus dinonaktifkan pada sistem yang tidak memerlukannya.

## 12. Kesimpulan

Network hardening dalam STIG berpusat pada pengurangan jalur akses yang tidak perlu dan penguatan jalur akses yang memang diperlukan. SSH harus tetap tersedia sebagai kanal aman, tetapi dikunci dengan algoritma, timeout, banner, dan pembatasan fitur. Firewall lokal, TCP syncookies, pembatasan port, dan kontrol wireless melengkapi lapisan pertahanan host.
