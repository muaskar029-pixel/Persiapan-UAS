# 05 — CIS Apache HTTP Server 2.4 Benchmark: Penjelasan Konseptual Bagian 1

**Sumber utama:** CIS Apache HTTP Server 2.4 Benchmark v2.3.0 — 09-23-2025  
**Ruang lingkup file ini:** Overview, Recommendation Definitions, Profile Definitions, Bab 1 sampai Bab 5.  
**Fokus pembahasan:** konseptual, bukan panduan praktik perintah tahap demi tahap.

---

## Daftar Isi

1. [Overview](#1-overview)
2. [Recommendation Definitions](#2-recommendation-definitions)
3. [Profile Definitions](#3-profile-definitions)
4. [Bab 1 — Planning and Installation](#4-bab-1--planning-and-installation)
5. [Bab 2 — Minimize Apache Modules](#5-bab-2--minimize-apache-modules)
6. [Bab 3 — Principles, Permissions, and Ownership](#6-bab-3--principles-permissions-and-ownership)
7. [Bab 4 — Apache Access Control](#7-bab-4--apache-access-control)
8. [Bab 5 — Minimize Features, Content and Options](#8-bab-5--minimize-features-content-and-options)
9. [Ringkasan Konseptual Bagian 1](#9-ringkasan-konseptual-bagian-1)

---

# 1. Overview

## 1.1 Apa itu CIS Apache HTTP Server 2.4 Benchmark

**CIS Apache HTTP Server 2.4 Benchmark** adalah panduan konfigurasi keamanan yang disusun oleh Center for Internet Security untuk membantu organisasi mengamankan Apache HTTP Server versi 2.4 yang berjalan di Linux.

Benchmark ini tidak membahas keamanan web secara umum saja, tetapi berfokus pada **technical configuration settings**. Artinya, CIS memberikan rekomendasi tentang bagaimana Apache sebaiknya dikonfigurasi agar memiliki postur keamanan yang lebih kuat.

Dalam konteks Apache, pengamanan tidak cukup hanya dengan memasang web server lalu memastikan halaman web dapat dibuka. Apache adalah komponen yang langsung berhadapan dengan jaringan, client, browser, scanner, bot, dan potensi attacker. Karena itu, konfigurasi Apache harus dibuat secara hati-hati agar hanya fitur yang diperlukan saja yang aktif, file penting tidak mudah diubah, informasi sensitif tidak bocor, dan akses ke konten web dikendalikan secara tepat.

Secara sederhana:

> CIS Apache Benchmark adalah **baseline hardening** untuk Apache HTTP Server 2.4.

Baseline berarti titik awal keamanan yang disarankan, bukan satu-satunya kontrol keamanan yang harus dimiliki organisasi.

## 1.2 Fungsi Benchmark dalam Program Cybersecurity

CIS Benchmark dirancang sebagai bagian dari program cybersecurity yang lebih besar. Benchmark ini membantu organisasi membuat konfigurasi yang konsisten, terukur, dan dapat diaudit.

Namun, benchmark tidak menggantikan aktivitas cyber hygiene lain seperti:

- monitoring sistem operasi dan aplikasi,
- patching keamanan,
- endpoint protection,
- antivirus atau EDR,
- logging dan monitoring aktivitas,
- vulnerability management,
- secure development lifecycle,
- network security control,
- incident response.

Apache yang sudah mengikuti CIS Benchmark tetap perlu dipantau, diperbarui, dan diuji secara berkala. CIS bukan “alat ajaib” yang langsung membuat sistem kebal serangan. CIS lebih tepat dipahami sebagai **kerangka konfigurasi aman**.

## 1.3 Kenapa Apache Perlu Dihardening

Apache HTTP Server biasanya berjalan sebagai layanan jaringan yang menerima request dari client. Bila Apache terbuka ke internet atau jaringan internal, maka setiap kesalahan konfigurasi dapat menjadi celah.

Contoh risiko yang sering muncul pada Apache:

- modul yang tidak diperlukan ikut aktif,
- directory listing terbuka,
- file `.git`, `.svn`, `.htaccess`, atau backup dapat dibaca dari web,
- default page dan contoh script masih tersedia,
- server mengungkap informasi versi atau struktur direktori,
- akses ke direktori sistem tidak dibatasi dengan benar,
- HTTP method berbahaya masih aktif,
- Apache process berjalan dengan privilege terlalu tinggi,
- file konfigurasi dapat dimodifikasi user yang tidak berwenang,
- web root dapat ditulis oleh akun Apache sehingga vulnerability upload menjadi lebih berbahaya.

Hardening bertujuan mengurangi peluang serangan dengan cara memperkecil permukaan serangan, memperkuat kontrol akses, dan membatasi dampak jika terjadi kompromi.

## 1.4 Baseline, Bukan Hukum Mutlak

CIS menyebut item di dalam benchmark sebagai **recommendations**, bukan **requirements**. Ini penting. Rekomendasi CIS adalah praktik aman yang umumnya berlaku luas, tetapi setiap organisasi tetap memiliki konteks berbeda.

Contohnya:

- CIS menyarankan modul proxy dinonaktifkan jika tidak digunakan.
- Namun jika Apache memang dipakai sebagai reverse proxy, maka modul proxy boleh aktif.
- CIS menyarankan default CGI content dihapus.
- Namun jika ada CGI tertentu yang benar-benar dibutuhkan aplikasi, maka penggunaannya harus didokumentasikan dan dibatasi.

Dengan demikian, CIS bukan berarti “semua harus dimatikan”, tetapi “semua harus **disadari, dibutuhkan, dibatasi, dan terdokumentasi**”.

## 1.5 Konteks Teknologi Target

Benchmark ini ditujukan untuk **Apache Web Server versi 2.4 yang berjalan di Linux**. Dokumen CIS menyebutkan bahwa panduan ini diuji terhadap Apache Web Server 2.4.65 yang dibangun dari source `httpd-2.4.65.tar.gz` dari situs Apache.

Hal ini penting karena perilaku default Apache dapat berbeda antara:

- build dari source,
- paket vendor Debian/Ubuntu,
- paket vendor RHEL/CentOS/Alma/Rocky,
- image container,
- instalasi panel hosting,
- Apache yang dikompilasi custom.

CIS menekankan bahwa nilai default di benchmark tidak selalu sama dengan nilai default pada instalasi kita. Paket vendor sering membawa modul tambahan, patch, struktur direktori, dan konfigurasi default yang berbeda dari source build.

---

# 2. Recommendation Definitions

Bagian **Recommendation Definitions** menjelaskan struktur setiap rekomendasi CIS. Ini penting agar pembaca tidak hanya mengikuti judul rekomendasi, tetapi memahami isi, alasan, dampak, audit, dan remediation-nya.

## 2.1 Title

**Title** adalah judul singkat dari rekomendasi. Judul biasanya diawali dengan kata “Ensure”, misalnya:

- `Ensure the WebDAV Modules Are Disabled`
- `Ensure the Apache Web Server Runs As a Non-Root User`
- `Ensure Access to OS Root Directory Is Denied By Default`

Judul memberi gambaran cepat tentang keadaan konfigurasi yang diharapkan.

## 2.2 Assessment Status

**Assessment Status** menjelaskan apakah rekomendasi dapat dinilai secara otomatis atau membutuhkan validasi manual.

Ada dua status utama:

1. **Automated**
2. **Manual**

Keduanya sama-sama penting. Automated bukan berarti lebih penting, dan Manual bukan berarti opsional.

## 2.3 Automated

Rekomendasi **Automated** adalah rekomendasi yang status patuh atau tidak patuhnya dapat diperiksa secara teknis dengan hasil pass/fail yang jelas.

Contohnya:

- apakah modul `mod_status` aktif,
- apakah `TraceEnable` disetel `off`,
- apakah file Apache dimiliki oleh root,
- apakah `AllowOverride` bernilai `None`.

Automated mudah dimasukkan ke dalam assessment tool karena kondisi teknisnya dapat diuji langsung.

## 2.4 Manual

Rekomendasi **Manual** membutuhkan penilaian manusia atau analisis konteks. Hal ini biasanya terjadi ketika “benar atau salah”-nya suatu konfigurasi bergantung pada kebutuhan bisnis.

Contohnya:

- apakah server memang tidak multi-use,
- apakah akses ke web content sudah tepat,
- apakah modul authentication yang aktif memang dibutuhkan,
- apakah direktori writable aplikasi sudah dipisahkan sesuai fungsi,
- apakah header Referrer-Policy sudah sesuai kebutuhan aplikasi.

Manual tetap penting karena banyak risiko keamanan tidak bisa dipahami hanya dari hasil skrip. Misalnya, skrip dapat melihat bahwa suatu direktori dapat diakses, tetapi hanya manusia yang bisa memutuskan apakah akses itu sesuai kebutuhan aplikasi.

## 2.5 Profile

**Profile** menunjukkan kelompok rekomendasi berdasarkan tingkat keamanan dan dampak operasional. Pada benchmark Apache ini terdapat Level 1 dan Level 2.

Profile membantu organisasi memilih baseline yang sesuai dengan kebutuhan, risiko, dan toleransi gangguan operasional.

## 2.6 Description

**Description** menjelaskan setting atau area konfigurasi yang sedang dibahas. Pada bagian ini CIS biasanya menjelaskan fungsi modul, directive, file, atau mekanisme Apache yang menjadi objek rekomendasi.

Misalnya, pada rekomendasi WebDAV, Description menjelaskan bahwa WebDAV memungkinkan client membuat, memindahkan, dan menghapus file/resource di web server.

## 2.7 Rationale Statement

**Rationale** adalah alasan keamanan mengapa rekomendasi tersebut penting.

Bagian ini sangat penting untuk pembahasan konseptual. Dari rationale kita memahami prinsip keamanan di balik kontrol tersebut, misalnya:

- mengurangi attack surface,
- mencegah information disclosure,
- membatasi unauthorized modification,
- mencegah privilege abuse,
- mengurangi risiko denial of service,
- membatasi akses ke file sensitif.

## 2.8 Impact Statement

**Impact** menjelaskan konsekuensi keamanan, fungsional, atau operasional yang mungkin muncul jika rekomendasi diterapkan.

Tidak semua rekomendasi memiliki impact yang besar, tetapi beberapa dapat memengaruhi aplikasi. Contohnya:

- menonaktifkan modul proxy dapat merusak fungsi reverse proxy,
- membatasi HTTP method dapat mengganggu API tertentu,
- membatasi file extension dapat membuat file sah tidak dapat diakses,
- membatasi Referrer-Policy terlalu ketat dapat mengganggu integrasi aplikasi.

Karena itu, CIS selalu perlu diterapkan dengan testing.

## 2.9 Audit Procedure

**Audit Procedure** menjelaskan cara memeriksa apakah sistem sudah sesuai rekomendasi.

Dalam file konseptual ini, audit tidak dibahas sebagai langkah praktik, tetapi konsepnya adalah:

- mengidentifikasi konfigurasi yang aktif,
- memeriksa modul yang dimuat,
- memeriksa directive Apache,
- memeriksa permission dan ownership file,
- memeriksa apakah akses web sesuai kebijakan.

## 2.10 Remediation Procedure

**Remediation Procedure** menjelaskan cara memperbaiki konfigurasi agar sesuai rekomendasi CIS.

Secara konsep, remediation pada Apache biasanya berupa:

- menonaktifkan modul yang tidak diperlukan,
- menghapus konten default,
- mengubah directive konfigurasi,
- memperbaiki ownership dan permission,
- menambahkan header keamanan,
- membatasi HTTP method,
- memperjelas IP listen.

Remediation harus diuji sebelum diterapkan ke server produksi.

## 2.11 Default Value

**Default Value** menunjukkan kondisi default jika diketahui. Pada Apache, default value bisa berbeda tergantung cara instalasi.

CIS menekankan bahwa source build default dan vendor package default dapat berbeda. Karena itu, administrator tidak boleh berasumsi bahwa Apache yang terpasang pasti sama dengan nilai default dalam benchmark.

## 2.12 References

**References** berisi tautan atau sumber dokumentasi terkait, biasanya dokumentasi Apache resmi atau sumber keamanan lain.

References membantu administrator memahami konteks teknis lebih dalam sebelum membuat perubahan konfigurasi.

## 2.13 CIS Controls Mapping

CIS Benchmark juga memetakan rekomendasi ke CIS Critical Security Controls. Mapping ini berguna untuk compliance, audit, dan pelaporan keamanan.

Contoh tema kontrol yang sering muncul pada bagian ini:

- secure configuration,
- access control,
- unnecessary services/modules,
- logging,
- application layer filtering,
- authorized software,
- least privilege.

## 2.14 Additional Information

**Additional Information** berisi informasi tambahan yang tidak masuk ke bagian lain tetapi berguna untuk memahami rekomendasi.

---

# 3. Profile Definitions

Benchmark Apache ini menggunakan dua profile utama: **Level 1** dan **Level 2**.

## 3.1 Level 1

**Level 1** adalah baseline keamanan yang dimaksudkan agar:

- praktis,
- masuk akal untuk diterapkan,
- memberikan manfaat keamanan yang jelas,
- tidak menghambat fungsi teknologi secara berlebihan.

Level 1 cocok sebagai baseline awal untuk sebagian besar server Apache produksi.

Contoh rekomendasi Level 1:

- Apache tidak menjadi multi-use server,
- modul WebDAV dinonaktifkan jika tidak diperlukan,
- Apache berjalan sebagai non-root user,
- akses ke OS root directory ditolak secara default,
- TRACE method dinonaktifkan,
- akses ke `.ht*`, `.git`, dan `.svn` dibatasi.

## 3.2 Level 2

**Level 2** memperluas Level 1. Level 2 bukan profile yang berdiri sendiri; Level 2 mengasumsikan Level 1 juga diterapkan.

Level 2 cocok untuk lingkungan dengan kebutuhan keamanan lebih tinggi, misalnya:

- sistem publik yang kritikal,
- aplikasi yang menangani data sensitif,
- lingkungan compliance ketat,
- server dengan exposure tinggi,
- organisasi yang menerapkan defense in depth lebih agresif.

Level 2 dapat memiliki dampak lebih besar terhadap fungsi atau performa. Karena itu, Level 2 perlu diuji lebih hati-hati.

Contoh rekomendasi Level 2:

- membatasi Options pada web root,
- menghapus default CGI script,
- membatasi HTTP request methods,
- menolak request berbasis IP address,
- menentukan IP listen secara eksplisit,
- membatasi browser framing,
- menetapkan header Referrer-Policy dan Permissions-Policy.

## 3.3 Cara Memilih Profile

Pemilihan profile sebaiknya mempertimbangkan:

- fungsi server,
- exposure jaringan,
- sensitivitas data,
- kebutuhan compliance,
- tingkat toleransi gangguan aplikasi,
- kemampuan tim melakukan testing dan monitoring.

Untuk pembelajaran dan baseline awal, Level 1 biasanya menjadi dasar. Untuk server produksi penting, Level 2 dapat diterapkan bertahap setelah diuji.

---

# 4. Bab 1 — Planning and Installation

Bab 1 membahas perencanaan dan instalasi Apache. Secara konseptual, CIS ingin menekankan bahwa keamanan Apache tidak dimulai setelah web server berjalan, tetapi sejak tahap desain, pemilihan paket, perencanaan jaringan, dan penentuan fungsi server.

## 4.1 Tujuan Bab 1

Tujuan utama Bab 1 adalah memastikan Apache dibangun di atas fondasi yang aman:

- kebijakan keamanan web sudah dipahami,
- infrastruktur jaringan sudah dikendalikan,
- sistem operasi sudah dihardening,
- logging dan monitoring sudah disiapkan,
- server tidak dijadikan multi-use system,
- Apache dipasang dari sumber binary yang tepat dan dapat dipercaya.

Bab ini dapat dipahami sebagai prinsip:

> Jangan mengamankan Apache secara terpisah dari lingkungan tempat Apache berjalan.

Apache yang dikonfigurasi aman tetap berisiko jika OS, jaringan, DNS, firewall, patching, atau proses development tidak aman.

## 4.2 1.1 — Pre-Installation Planning Checklist

Rekomendasi ini bersifat **Manual** dan masuk **Level 1**.

### Konsep Utama

Sebelum Apache dipasang, organisasi harus melakukan perencanaan. Apache tidak boleh langsung diinstal tanpa mempertimbangkan desain keamanan.

Checklist perencanaan mencakup beberapa hal penting:

- kebijakan keamanan web organisasi,
- kontrol akses jaringan melalui firewall, router, dan switch,
- hardening sistem operasi,
- pengurangan listening services pada OS,
- penerapan patch keamanan,
- monitoring log terpusat,
- monitoring kapasitas disk,
- mekanisme log rotation,
- edukasi developer, architect, dan tester tentang secure development,
- pengamanan DNS,
- penggunaan Network Intrusion Detection System bila diperlukan,
- pengelolaan WHOIS agar tidak membocorkan informasi personel sensitif.

### Penjelasan Konseptual

Apache hanyalah satu lapisan dalam arsitektur web. Jika layer lain lemah, Apache yang sudah dihardening tetap dapat dikompromikan.

Contoh:

- Jika DNS tidak aman, pengguna bisa diarahkan ke server palsu.
- Jika firewall tidak membatasi akses, port internal bisa terekspos.
- Jika developer menulis aplikasi rentan upload shell, hardening Apache hanya mengurangi dampaknya, bukan menghilangkan risiko sepenuhnya.
- Jika log tidak dimonitor, serangan bisa terjadi tanpa diketahui.

Jadi, rekomendasi ini menekankan bahwa hardening Apache harus menjadi bagian dari **secure deployment lifecycle**.

### Inti Keamanan

Perencanaan pra-instalasi bertujuan memastikan:

- Apache dipasang karena memang dibutuhkan,
- lingkungan server mendukung keamanan Apache,
- tim memahami risiko web server,
- kontrol pendukung tersedia sebelum server diproduksikan.

## 4.3 1.2 — Server Is Not a Multi-Use System

Rekomendasi ini bersifat **Manual** dan masuk **Level 1**.

### Konsep Multi-Use System

Multi-use system adalah server yang menjalankan banyak fungsi utama sekaligus, misalnya:

- web server,
- database server,
- mail server,
- DNS server,
- file server,
- middleware,
- proxy,
- development tools.

CIS menyarankan agar Apache web server sebaiknya memiliki fungsi utama sebagai web server saja.

### Kenapa Multi-Use Server Berisiko

Semakin banyak layanan aktif, semakin banyak attack surface.

Jika Apache server juga menjalankan DNS, mail, database, dan file sharing, maka attacker memiliki lebih banyak pintu masuk. Kompromi pada satu layanan dapat berdampak ke layanan lain.

Contoh:

- Celah pada mail server dapat memberi akses ke server yang juga menjalankan Apache.
- Database lokal di server web meningkatkan dampak jika web server dikompromikan.
- File sharing yang salah konfigurasi dapat membocorkan source code aplikasi.

### Prinsip Keamanan

Rekomendasi ini berhubungan dengan prinsip **single purpose server**.

Server yang memiliki satu fungsi utama lebih mudah:

- dipantau,
- diaudit,
- dipatch,
- dikonfigurasi,
- dibatasi aksesnya,
- dipulihkan saat insiden.

## 4.4 1.3 — Apache Installed From Appropriate Binaries

Rekomendasi ini bersifat **Manual** dan masuk **Level 1**.

### Konsep Binary yang Tepat

Apache dapat dipasang dari beberapa sumber:

- paket vendor OS,
- repository resmi distribusi,
- build dari source,
- package pihak ketiga,
- image container,
- bundle panel hosting.

CIS menyarankan penggunaan binary dari vendor untuk sebagian besar situasi karena lebih mudah dipelihara, diuji, dan diperbarui.

### Kenapa Sumber Instalasi Penting

Sumber instalasi menentukan:

- siapa yang menyediakan patch keamanan,
- bagaimana update diterapkan,
- modul apa saja yang aktif secara default,
- struktur direktori konfigurasi,
- integrasi dengan service manager OS,
- keandalan package signature dan supply chain.

Binary dari vendor biasanya sudah disesuaikan dengan OS dan dapat diperbarui melalui mekanisme package management resmi.

### Source Build vs Vendor Package

CIS mencatat perbedaan besar antara source build dan vendor package.

Source build default Apache cenderung lebih minimal. Vendor package sering menyertakan lebih banyak modul dan integrasi karena dirancang agar “siap pakai”.

Implikasinya:

- jangan menganggap default di benchmark sama dengan default pada server;
- selalu cek modul dan konfigurasi aktual;
- paket vendor bisa aman, tetapi perlu ditinjau karena mungkin lebih banyak fitur aktif.

---

# 5. Bab 2 — Minimize Apache Modules

Bab 2 membahas pengurangan modul Apache. Ini adalah salah satu bagian paling penting dalam hardening Apache.

## 5.1 Konsep Modul Apache

Apache bersifat modular. Banyak fitur Apache tersedia dalam bentuk modul, misalnya:

- modul logging,
- modul authentication,
- modul proxy,
- modul WebDAV,
- modul status,
- modul autoindex,
- modul userdir,
- modul info,
- modul basic/digest authentication.

Modul memberi fleksibilitas, tetapi juga menambah attack surface. Modul yang aktif berarti ada kode tambahan yang dimuat dan bisa dipicu oleh request atau konfigurasi tertentu.

Prinsip CIS:

> Aktifkan hanya modul yang benar-benar dibutuhkan oleh fungsi bisnis.

## 5.2 Kenapa Modul yang Tidak Dibutuhkan Harus Dinonaktifkan

Modul yang tidak digunakan tetap dapat menimbulkan risiko:

- memiliki vulnerability,
- membuka handler atau URL khusus,
- mengubah perilaku request,
- memperluas kemampuan server tanpa disadari,
- membocorkan informasi,
- memungkinkan fitur yang tidak diinginkan seperti directory listing atau WebDAV.

Menonaktifkan modul tidak perlu bukan sekadar “merapikan konfigurasi”, tetapi bagian dari **attack surface reduction**.

## 5.3 2.1 — Necessary Authentication and Authorization Modules

Rekomendasi ini bersifat **Manual** dan masuk **Level 1**.

### Konsep Authentication dan Authorization

Dalam Apache:

- `authn_*` berkaitan dengan authentication, yaitu pembuktian identitas user.
- `authz_*` berkaitan dengan authorization, yaitu penentuan apakah user boleh mengakses resource.

Authentication menjawab pertanyaan: “Siapa kamu?”  
Authorization menjawab pertanyaan: “Apa yang boleh kamu akses?”

### Risiko Modul Auth yang Berlebihan

Modul authentication dan authorization adalah “pintu depan” ke area terlindungi. Jika terlalu banyak modul aktif, semakin banyak kemungkinan konfigurasi keliru.

Contoh risiko:

- modul LDAP aktif padahal tidak digunakan,
- modul Basic Auth aktif padahal aplikasi memakai SSO,
- modul group file aktif tetapi file group tidak dikelola baik,
- authorization rule bentrok antar konteks.

### Prinsip Konseptual

Hanya modul auth yang benar-benar digunakan yang boleh aktif. Jika aplikasi memakai mekanisme autentikasi internal, reverse proxy SSO, atau identity provider lain, modul Apache auth perlu ditinjau ulang.

## 5.4 2.2 — Log Config Module Enabled

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Fungsi `mod_log_config`

`mod_log_config` memungkinkan Apache mengatur format logging request client. Modul ini penting karena access log adalah sumber utama untuk:

- monitoring trafik,
- investigasi insiden,
- deteksi abuse,
- analisis forensik,
- troubleshooting,
- audit aktivitas web.

### Kenapa Modul Ini Harus Aktif

Tidak semua modul harus dimatikan. `mod_log_config` justru harus aktif karena logging adalah kontrol keamanan dasar.

Tanpa logging yang memadai, organisasi sulit menjawab:

- siapa mengakses URL tertentu,
- kapan request terjadi,
- dari IP mana request berasal,
- response code apa yang diberikan,
- user agent apa yang digunakan,
- apakah ada scanning atau brute force.

### Prinsip Konseptual

Pengurangan modul tidak berarti menonaktifkan semua modul. Modul yang mendukung visibility dan audit justru perlu dipertahankan.

## 5.5 2.3 — WebDAV Modules Disabled

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Apa itu WebDAV

WebDAV adalah ekstensi HTTP yang memungkinkan client membuat, memindahkan, mengubah, dan menghapus file/resource pada web server.

Modul terkait antara lain:

- `mod_dav`,
- `mod_dav_fs`.

### Risiko WebDAV

WebDAV dapat berbahaya jika tidak dibutuhkan karena memberikan kemampuan tulis terhadap resource web.

Risikonya antara lain:

- upload file tidak sah,
- pengubahan konten web,
- penghapusan file,
- bypass proses deployment normal,
- penyalahgunaan kontrol akses yang salah konfigurasi.

### Prinsip Konseptual

Jika server hanya menyajikan konten web dan tidak perlu fitur remote authoring, WebDAV sebaiknya dimatikan.

## 5.6 2.4 — Status Module Disabled

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Fungsi `mod_status`

`mod_status` menyediakan statistik performa server, misalnya status worker, request aktif, dan informasi server runtime.

### Risiko `mod_status`

Informasi performa dapat membantu attacker memahami kondisi server, seperti:

- beban server,
- pola request,
- worker yang sibuk,
- kemungkinan timing exploit,
- endpoint internal tertentu.

Jika endpoint status terbuka atau dapat diaktifkan melalui konfigurasi tidak terduga, ia dapat menjadi sumber information disclosure.

### Prinsip Konseptual

Informasi operasional server tidak boleh tersedia ke pihak yang tidak berwenang. Jika status memang dibutuhkan, aksesnya harus sangat dibatasi dan didokumentasikan. Dalam baseline CIS, modul ini dinonaktifkan.

## 5.7 2.5 — Autoindex Module Disabled

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Fungsi Autoindex

`mod_autoindex` membuat listing otomatis isi direktori jika tidak ada file index seperti `index.html`.

### Risiko Directory Listing

Directory listing dapat membocorkan:

- nama file,
- struktur direktori,
- file backup,
- file konfigurasi,
- file lama,
- naming convention,
- artefak deployment.

Attacker sering memanfaatkan directory listing untuk menemukan file yang tidak dimaksudkan untuk publik.

### Prinsip Konseptual

Web server seharusnya hanya menyajikan file yang memang dimaksudkan untuk diakses. Jika index tidak ada, sebaiknya server tidak otomatis menampilkan daftar isi direktori.

## 5.8 2.6 — Proxy Modules Disabled if Not in Use

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Fungsi Proxy Module

Modul proxy memungkinkan Apache bertindak sebagai:

- forward proxy,
- reverse proxy,
- proxy HTTP,
- proxy FastCGI,
- proxy AJP,
- proxy WebSocket,
- load balancer.

### Risiko Proxy

Proxy adalah fungsi yang kuat tetapi berisiko jika salah konfigurasi.

Risiko utamanya:

- open proxy,
- attacker memakai server untuk menyembunyikan asal serangan,
- proxy ke jaringan internal,
- bypass segmentasi jaringan,
- SSRF-like behavior,
- peningkatan kompleksitas konfigurasi.

### Prinsip Konseptual

Apache sebaiknya menjadi web server atau proxy server sesuai desain, bukan keduanya tanpa alasan jelas. Jika fungsi proxy tidak diperlukan, modul proxy harus dimatikan.

Jika Apache memang digunakan sebagai reverse proxy, maka itu harus menjadi pengecualian yang disadari dan dikonfigurasi aman.

## 5.9 2.7 — User Directories Module Disabled

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Apa itu UserDir

`mod_userdir` memungkinkan direktori home user diakses melalui URL seperti:

`http://example.com/~username/`

Biasanya direktori seperti `public_html` di dalam home user dapat menjadi konten web.

### Risiko UserDir

UserDir berisiko karena setiap user lokal dapat berpotensi menyediakan konten web.

Risikonya:

- konten tidak melalui proses review,
- setiap akun baru dapat menjadi sumber konten publik,
- informasi dalam home directory bisa tidak sengaja terekspos,
- mapping yang salah dapat membuka direktori sensitif.

### Prinsip Konseptual

Konten web produksi harus dikendalikan melalui jalur deployment resmi, bukan melalui direktori home user.

## 5.10 2.8 — Info Module Disabled

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Fungsi `mod_info`

`mod_info` menyediakan informasi konfigurasi server melalui URL seperti `/server-info`.

### Risiko `mod_info`

Informasi konfigurasi dapat membocorkan:

- path sistem,
- modul aktif,
- directive Apache,
- username,
- detail backend,
- informasi database atau integrasi,
- struktur virtual host.

Informasi seperti ini sangat berguna untuk reconnaissance attacker.

### Prinsip Konseptual

Konfigurasi server adalah informasi sensitif. Ia tidak boleh disajikan melalui web kecuali dalam lingkungan sangat terbatas, dan pada baseline CIS modul ini dinonaktifkan.

## 5.11 2.9 — Basic and Digest Authentication Modules Disabled

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Basic dan Digest Authentication

`mod_auth_basic` mendukung HTTP Basic Authentication.  
`mod_auth_digest` mendukung HTTP Digest Authentication.

Keduanya adalah mekanisme lama untuk membatasi akses berdasarkan username dan password.

### Risiko Basic Authentication

Basic Authentication mengirim kredensial yang di-encode Base64. Jika tidak melalui HTTPS, kredensial mudah terbaca. Walaupun melalui HTTPS, ada masalah lain:

- kredensial dikirim berulang pada request,
- browser dapat menyimpan kredensial,
- tidak ada session management server-side yang baik,
- logout tidak selalu bermakna seperti aplikasi modern,
- kredensial dapat muncul pada header request.

### Risiko Digest Authentication

CIS menilai Digest Authentication juga tidak ideal dan dapat lebih bermasalah karena isu penyimpanan dan manajemen password.

### Prinsip Konseptual

Untuk aplikasi modern, autentikasi sebaiknya ditangani oleh mekanisme yang lebih kuat seperti aplikasi web, SSO, federated identity, reverse proxy authentication, atau identity provider yang sesuai.

---

# 6. Bab 3 — Principles, Permissions, and Ownership

Bab 3 membahas identitas, permission, ownership, dan file system resource yang digunakan Apache.

## 6.1 Tujuan Bab 3

Tujuan utama Bab 3 adalah memastikan Apache tidak memiliki privilege yang berlebihan dan file penting tidak dapat diubah oleh pihak yang tidak berwenang.

Prinsip besarnya adalah:

- Apache process tidak berjalan sebagai root untuk melayani request,
- akun Apache adalah service account terbatas,
- file Apache dimiliki root,
- permission tulis dibatasi,
- web root tidak dapat ditulis oleh Apache group secara sembarangan,
- direktori writable aplikasi dipisahkan dan dikendalikan.

## 6.2 3.1 — Apache Runs As a Non-Root User

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Konsep Non-Root Execution

Apache mungkin dimulai oleh root agar dapat bind ke port rendah seperti 80 dan 443. Namun worker process yang menangani request sebaiknya berjalan sebagai user non-root khusus.

### Kenapa Tidak Boleh Melayani Request sebagai Root

Jika Apache berjalan sebagai root dan terjadi vulnerability, attacker bisa memperoleh privilege root. Ini sangat berbahaya karena root dapat mengubah seluruh sistem.

Dengan menjalankan worker sebagai user terbatas, dampak kompromi dapat dibatasi.

### Kenapa User Apache Harus Khusus

Akun seperti `nobody` atau `daemon` sering dipakai oleh banyak service. CIS menyarankan akun unik untuk Apache agar aksesnya tidak bercampur dengan service lain.

Prinsipnya:

> Satu service, satu identitas terbatas.

## 6.3 3.2 — Apache User Account Has an Invalid Shell

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Konsep Service Account

Akun Apache adalah akun layanan, bukan akun manusia. Ia tidak perlu login interaktif.

### Risiko Shell Valid

Jika akun Apache memiliki shell valid, attacker atau user lokal mungkin dapat login atau berpindah menjadi akun Apache.

Ini meningkatkan risiko:

- penyalahgunaan file yang dapat diakses Apache,
- persistence,
- lateral movement,
- eksekusi command dengan identitas Apache.

### Prinsip Konseptual

Service account harus tidak dapat digunakan untuk login normal. Shell seperti `nologin` atau shell invalid membatasi penyalahgunaan akun.

## 6.4 3.3 — Apache User Account Is Locked

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Konsep Account Locking

Akun Apache seharusnya tidak memiliki password aktif. Jika password aktif, user dapat mencoba login atau `su` ke akun tersebut.

### Kenapa Perlu Dikunci

Locking adalah defense in depth. Walaupun shell sudah invalid, akun tetap lebih aman jika password login juga tidak dapat dipakai.

### Prinsip Konseptual

Akun layanan bukan akun interaktif. Administrasi harus dilakukan dengan akun admin dan sudo, bukan dengan login sebagai service account.

## 6.5 3.4 — Apache Directories and Files Owned By Root

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Konsep Ownership

File program dan konfigurasi Apache harus dimiliki root. Ini mencegah user biasa atau akun Apache mengubah binary, modul, atau konfigurasi server.

### Risiko Ownership Lemah

Jika file Apache dimiliki user biasa atau service account, attacker yang menguasai user tersebut dapat:

- mengubah konfigurasi,
- mengganti modul,
- menyisipkan backdoor,
- mengubah script,
- mengubah konten dengan cara tidak sah.

### Prinsip Konseptual

Akun yang menjalankan layanan tidak boleh sekaligus memiliki kontrol penuh terhadap software dan konfigurasi layanan tersebut.

## 6.6 3.5 — Group Set Correctly on Apache Directories and Files

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Konsep Group Ownership

Selain user owner, group owner juga penting. CIS menyarankan file dan direktori Apache menggunakan group root atau group setara root, kecuali area web content tertentu yang memang membutuhkan group khusus untuk proses update konten.

### Risiko Group yang Terlalu Luas

Jika group yang tidak tepat memiliki akses ke file Apache, anggota group tersebut dapat mengubah file penting.

Contoh risiko:

- developer group dapat mengubah konfigurasi produksi tanpa kontrol,
- web group dapat mengubah binary atau modul,
- group umum dapat menulis file server.

### Prinsip Konseptual

Group write access harus diberikan hanya jika ada kebutuhan nyata dan melalui proses change management.

## 6.7 3.6 — Other Write Access Restricted

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Konsep “Other Write”

Dalam permission Unix, “other” berarti semua user yang bukan owner dan bukan group. Jika “other” bisa menulis file Apache, maka hampir semua user lokal dapat memodifikasi file tersebut.

### Risiko Other Write

Other write access sangat berbahaya karena memungkinkan:

- defacement,
- backdoor,
- perubahan konfigurasi,
- penyisipan malware,
- manipulasi konten web.

### Prinsip Konseptual

Tidak boleh ada file atau direktori Apache yang writable oleh “other”. Permission harus mengikuti least privilege.

## 6.8 3.7 — Core Dump Directory Secured

Rekomendasi ini bersifat **Manual** dan masuk **Level 1**.

### Apa itu Core Dump

Core dump adalah snapshot memori proses saat crash. File ini dapat berisi informasi sensitif seperti:

- data request,
- token,
- session,
- credential,
- path internal,
- konfigurasi runtime.

### Risiko Core Dump

Jika core dump tersimpan di lokasi yang dapat dibaca user lain atau berada di web root, informasi sensitif dapat bocor.

### Prinsip Konseptual

Jika core dump diizinkan, direktorinya harus aman, tidak berada di document root, dimiliki pihak berwenang, dan tidak dapat diakses user biasa.

## 6.9 3.8 — Lock File Secured

Rekomendasi ini bersifat **Manual** dan masuk **Level 1**.

### Fungsi Lock File

Apache dapat menggunakan lock file untuk mekanisme mutex atau sinkronisasi akses resource antar proses.

### Risiko Lock File di Direktori Writable

Jika lock file berada di direktori yang dapat ditulis user lain, attacker dapat membuat file dengan nama yang sama sehingga Apache gagal start atau terganggu.

Ini termasuk risiko denial of service.

### Prinsip Konseptual

File koordinasi internal proses harus berada di lokasi lokal yang aman, bukan di direktori web, bukan di NFS, dan bukan di direktori writable oleh user biasa.

## 6.10 3.9 — Pid File Secured

Rekomendasi ini bersifat **Manual** dan masuk **Level 1**.

### Fungsi PID File

PID file menyimpan process ID server. File ini membantu system manager atau administrator mengirim sinyal ke proses Apache.

### Risiko PID File Tidak Aman

Jika PID file dapat dimanipulasi, attacker dapat mengganggu proses manajemen service atau mencegah Apache berjalan normal.

### Prinsip Konseptual

File kontrol proses harus berada di direktori yang hanya dapat ditulis oleh root atau user startup yang sah.

## 6.11 3.10 — ScoreBoard File Secured

Rekomendasi ini bersifat **Manual** dan masuk **Level 1**.

### Fungsi ScoreBoard

ScoreBoard digunakan untuk komunikasi antar proses Apache. Pada banyak Linux modern, shared memory digunakan sehingga file scoreboard mungkin tidak diperlukan.

### Risiko ScoreBoard File

Jika file scoreboard berada di direktori writable, attacker bisa:

- menyebabkan denial of service,
- membaca informasi internal proses,
- mengganggu komunikasi antar proses.

### Prinsip Konseptual

Jika mekanisme IPC menggunakan file, lokasi file harus aman dan tidak bisa dimanipulasi user lain.

## 6.12 3.11 — Group Write Access for Apache Directories and Files Restricted

Rekomendasi ini bersifat **Manual** dan masuk **Level 1**.

### Konsep Group Write

Group write memberikan hak menulis kepada anggota group. Ini berguna untuk kolaborasi, tetapi berbahaya jika diterapkan pada file Apache secara luas.

### Risiko Group Write Berlebihan

Jika group dapat menulis file Apache, maka semua anggota group tersebut dapat mengubah file server.

Risikonya:

- perubahan tidak sah,
- bypass change management,
- backdoor,
- defacement,
- kesalahan konfigurasi oleh user non-admin.

### Prinsip Konseptual

Group write harus dibatasi. Untuk software dan konfigurasi Apache, akses tulis group biasanya tidak diperlukan.

## 6.13 3.12 — Group Write Access for Document Root Restricted

Rekomendasi ini bersifat **Manual** dan masuk **Level 1**.

### Konsep Document Root

Document root adalah direktori utama konten web. File di dalamnya dapat disajikan ke client.

### Kenapa Apache Group Tidak Boleh Menulis Web Root

Jika group Apache dapat menulis ke document root, maka vulnerability pada aplikasi yang berjalan sebagai Apache dapat digunakan untuk menulis file ke area publik.

Contoh dampak:

- upload web shell,
- defacement,
- penyisipan JavaScript berbahaya,
- perubahan file aplikasi,
- persistence attacker.

### Prinsip Konseptual

Apache boleh membaca konten web, tetapi tidak seharusnya dapat menulis ke web root kecuali ada alasan khusus yang dibatasi sangat ketat.

## 6.14 3.13 — Special Purpose Application Writable Directories Restricted

Rekomendasi ini bersifat **Manual** dan masuk **Level 1**.

### Konsep Direktori Writable Aplikasi

Aplikasi web sering membutuhkan direktori writable, misalnya untuk:

- upload file,
- session storage,
- cache,
- temporary files,
- generated reports,
- application data.

### Syarat Direktori Writable yang Aman

CIS menekankan beberapa prinsip:

1. direktori harus single purpose,
2. direktori sebaiknya berada di luar document root,
3. direktori dimiliki root atau administrator,
4. tidak writable oleh “other”,
5. akses write diberikan seminimal mungkin.

### Kenapa Harus Single Purpose

Mencampur fungsi direktori dapat membuka celah. Misalnya, direktori yang dipakai untuk upload file sekaligus session dapat memungkinkan attacker mengunggah file yang memengaruhi session.

### Kenapa di Luar Document Root

Jika direktori writable berada di document root, file yang diunggah bisa langsung diakses melalui URL. Ini sangat berbahaya jika attacker dapat mengunggah script.

### Prinsip Konseptual

Writable directory adalah area risiko tinggi. Ia harus diperlakukan sebagai zona khusus yang terisolasi dari web root dan dikontrol dengan least privilege.

---

# 7. Bab 4 — Apache Access Control

Bab 4 membahas kontrol akses Apache. Fokusnya adalah bagaimana Apache menentukan resource mana yang boleh diakses oleh client.

## 7.1 Tujuan Bab 4

Tujuan utama Bab 4 adalah memastikan Apache menerapkan prinsip:

> deny by default, allow by explicit need.

Artinya, akses ke sistem operasi dan direktori server harus ditolak secara default. Hanya konten web yang memang dimaksudkan untuk disajikan yang boleh dibuka.

## 7.2 Konsep Access Control di Apache

Apache menggunakan directive seperti:

- `<Directory>`,
- `<Location>`,
- `Require`,
- `Allow`,
- `Deny`,
- `AllowOverride`,
- `AllowOverrideList`.

Pada Apache 2.4, pendekatan modern adalah menggunakan `Require`. Pendekatan lama `Order`, `Allow`, dan `Deny` sudah deprecated, walaupun masih dapat muncul pada konfigurasi lama.

## 7.3 4.1 — Access to OS Root Directory Denied By Default

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Konsep OS Root Directory

OS root directory adalah `/`, yaitu akar dari seluruh filesystem Linux. Jika Apache dapat memetakan request ke file di luar web root, risiko kebocoran file sistem meningkat.

### Kenapa Harus Deny by Default

Apache memiliki prinsip default access yang sering disalahpahami. Jika server dapat menemukan file melalui aturan URL mapping normal dan tidak ada larangan, maka file tersebut dapat disajikan.

Karena itu, akses ke `/` harus ditolak secara default.

### Prinsip Konseptual

Kebijakan aman adalah:

- blokir seluruh OS root,
- lalu buka hanya direktori web yang memang dibutuhkan.

Ini mirip firewall: semua ditolak dulu, lalu dibuat allow rule khusus.

## 7.4 4.2 — Appropriate Access to Web Content Is Allowed

Rekomendasi ini bersifat **Manual** dan masuk **Level 1**.

### Konsep Allow yang Tepat

Setelah OS root ditolak, Apache tetap harus mengizinkan akses ke web content yang sah.

Akses dapat diberikan untuk:

- semua client,
- jaringan tertentu,
- host tertentu,
- user tertentu,
- local access,
- lokasi khusus seperti portal atau usage page.

### Kenapa Manual

Rekomendasi ini manual karena “akses yang tepat” tergantung aplikasi.

Contoh:

- website publik boleh `Require all granted` pada document root tertentu,
- admin panel hanya boleh IP internal,
- endpoint monitoring hanya boleh localhost,
- portal tertentu perlu user authentication.

Tool otomatis tidak bisa selalu tahu apakah suatu akses benar secara bisnis.

### Prinsip Konseptual

Akses web content harus spesifik dan sesuai kebutuhan. Jangan memberikan akses luas hanya karena halaman harus “bisa dibuka”.

## 7.5 4.3 — OverRide Disabled for OS Root Directory

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Apa itu AllowOverride

`AllowOverride` menentukan apakah file `.htaccess` boleh mengubah konfigurasi Apache pada direktori tertentu.

Jika `AllowOverride All`, file `.htaccess` dapat mengubah banyak hal, termasuk:

- authentication,
- authorization,
- options,
- handler,
- access control,
- content handling.

### Kenapa Berbahaya di OS Root

Jika override diizinkan dari root directory, konfigurasi Apache dapat menjadi tersebar dan sulit dikontrol. File `.htaccess` yang tidak diinginkan dapat mengubah perilaku akses.

### Prinsip Konseptual

Untuk OS root directory, override harus dimatikan. Konfigurasi keamanan harus dikelola terpusat di file konfigurasi Apache, bukan tersebar di file yang mungkin berada di direktori web.

## 7.6 4.4 — OverRide Disabled for All Directories

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Konsep Desentralisasi Konfigurasi

`.htaccess` memudahkan konfigurasi per direktori, tetapi dari sisi hardening ia membawa risiko:

- konfigurasi tersebar,
- lebih sulit diaudit,
- file dapat dibuat oleh pihak yang tidak berwenang,
- rule access control bisa berubah tanpa review pusat,
- performance overhead karena Apache perlu mencari `.htaccess`.

### Kenapa Dinonaktifkan Semua Direktori

CIS menyarankan agar `AllowOverride` bernilai `None` pada semua direktori dan `AllowOverrideList` tidak digunakan.

### Prinsip Konseptual

Semua konfigurasi Apache sebaiknya berada dalam file konfigurasi yang dikontrol administrator dan change management, bukan dalam file `.htaccess` yang berada di area konten.

---

# 8. Bab 5 — Minimize Features, Content and Options

Bab 5 membahas pengurangan fitur, konten, dan options Apache. Ini berhubungan langsung dengan attack surface.

## 8.1 Tujuan Bab 5

Tujuan Bab 5 adalah memastikan Apache hanya menyediakan fitur dan konten yang diperlukan.

Fokusnya mencakup:

- pembatasan `Options`,
- penghapusan konten default,
- penghapusan CGI contoh,
- pembatasan HTTP method,
- pemblokiran file sensitif,
- pembatasan request berbasis IP,
- pengaturan Listen yang eksplisit,
- header keamanan browser.

## 8.2 Konsep `Options` Apache

Directive `Options` mengontrol fitur per direktori, seperti:

- `Indexes`,
- `FollowSymLinks`,
- `SymLinksIfOwnerMatch`,
- `ExecCGI`,
- `Includes`,
- `IncludesNOEXEC`,
- `MultiViews`.

Setiap option membuka kemampuan tertentu. Jika tidak dibutuhkan, option tersebut sebaiknya tidak aktif.

## 8.3 5.1 — Options for OS Root Directory Restricted

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Konsep

Untuk OS root directory `/`, semua options sebaiknya dimatikan dengan nilai `None`.

### Kenapa Penting

Root directory adalah area paling luas. Jika options seperti `Indexes`, `FollowSymLinks`, atau `ExecCGI` aktif pada level root, efeknya dapat melebar ke banyak area yang tidak dimaksudkan.

### Prinsip Konseptual

Di level paling luas, gunakan kebijakan paling ketat. Fitur hanya boleh diaktifkan pada direktori spesifik yang memang membutuhkan.

## 8.4 5.2 — Options for Web Root Directory Restricted

Rekomendasi ini bersifat **Automated** dan masuk **Level 2**.

### Konsep

Web root atau document root juga harus dibatasi. CIS merekomendasikan `None`, atau `MultiViews` jika benar-benar diperlukan untuk content negotiation.

### Risiko Options Berlebihan di Web Root

Jika options terlalu banyak aktif:

- `Indexes` dapat membuka directory listing,
- `ExecCGI` dapat memungkinkan eksekusi script,
- `Includes` dapat memungkinkan server-side include berbahaya,
- `FollowSymLinks` dapat membuka akses ke luar web root.

### Prinsip Konseptual

Web root harus menyajikan konten, bukan menjadi tempat fitur dinamis yang tidak dikendalikan.

## 8.5 5.3 — Options for Other Directories Minimized

Rekomendasi ini bersifat **Automated** dan masuk **Level 2**.

### Konsep

Direktori lain di luar web root juga harus memiliki options minimal.

CIS membahas beberapa options:

- `MultiViews`: boleh jika content negotiation dibutuhkan.
- `ExecCGI`: hanya cocok untuk direktori khusus seperti `cgi-bin`.
- `FollowSymLinks`: sebaiknya dihindari jika mungkin.
- `SymLinksIfOwnerMatch`: lebih aman daripada `FollowSymLinks`, tetapi ada overhead.
- `Includes`: berisiko karena dapat mengeksekusi command.
- `IncludesNOEXEC`: lebih terbatas, tetapi tetap hanya jika perlu.
- `Indexes`: sebaiknya dimatikan kecuali benar-benar diperlukan.

### Prinsip Konseptual

Setiap direktori harus diberi fitur sesuai fungsi spesifiknya. Jangan menggunakan options luas secara global.

## 8.6 5.4 — Default HTML Content Removed

Rekomendasi ini bersifat **Manual** dan masuk **Level 1**.

### Konsep Default Content

Apache sering menyertakan konten default seperti:

- halaman “It works!”,
- welcome page,
- manual lokal,
- status page,
- info page,
- contoh konfigurasi,
- contoh handler.

### Risiko Default Content

Default content dapat:

- menunjukkan bahwa server memakai Apache,
- membocorkan struktur instalasi,
- menyediakan endpoint contoh yang tidak diproteksi,
- menjadi target exploit lama,
- membantu fingerprinting.

### Prinsip Konseptual

Server produksi hanya boleh berisi konten produksi yang memang diperlukan. Konten demo dan dokumentasi lokal harus dihapus atau tidak disajikan.

## 8.7 5.5 — Default CGI Content `printenv` Removed

Rekomendasi ini bersifat **Manual** dan masuk **Level 2**.

### Apa itu `printenv`

`printenv` adalah script CGI contoh yang menampilkan environment variables CGI.

### Risiko

Script ini dapat membocorkan:

- path server,
- konfigurasi CGI,
- informasi runtime,
- variabel lingkungan,
- detail platform.

CGI juga punya sejarah panjang vulnerability karena menerima input user dan mengeksekusi program.

### Prinsip Konseptual

Script contoh tidak boleh berada di server produksi. Jika tidak dibuat untuk kebutuhan produksi, jangan jalankan.

## 8.8 5.6 — Default CGI Content `test-cgi` Removed

Rekomendasi ini bersifat **Manual** dan masuk **Level 2**.

### Apa itu `test-cgi`

`test-cgi` adalah script CGI contoh untuk menguji kemampuan CGI server.

### Risiko

Seperti `printenv`, script ini tidak dibutuhkan di produksi dan dapat membocorkan detail server atau menjadi target eksploitasi.

### Prinsip Konseptual

Lingkungan produksi bukan tempat menyimpan script demonstrasi atau testing.

## 8.9 5.7 — HTTP Request Methods Restricted

Rekomendasi ini bersifat **Manual** dan masuk **Level 2**.

### Konsep HTTP Methods

HTTP memiliki berbagai method, seperti:

- `GET`,
- `HEAD`,
- `POST`,
- `OPTIONS`,
- `PUT`,
- `DELETE`,
- `TRACE`,
- `CONNECT`.

Untuk web biasa, umumnya hanya `GET`, `HEAD`, `POST`, dan kadang `OPTIONS` yang diperlukan.

### Risiko Method Berbahaya

Method seperti `PUT` dan `DELETE` dapat memodifikasi resource jika didukung aplikasi atau server. Method tidak lazim juga sering dipakai scanner untuk enumerasi.

### Prinsip Konseptual

Apache harus menerima hanya method yang dibutuhkan aplikasi. Semakin banyak method diterima, semakin luas kemungkinan abuse.

Catatan penting: pembatasan method biasa tidak otomatis mematikan `TRACE`; CIS membahas TRACE secara khusus pada rekomendasi berikutnya.

## 8.10 5.8 — HTTP TRACE Method Disabled

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Apa itu TRACE

TRACE adalah method HTTP yang memantulkan request kembali ke client. Awalnya berguna untuk diagnostik.

### Risiko TRACE

TRACE tidak dibutuhkan untuk operasi normal dan dapat disalahgunakan untuk serangan tertentu atau diagnostic abuse.

### Prinsip Konseptual

Fitur diagnostik yang memantulkan request tidak perlu tersedia pada server produksi. TRACE harus dimatikan.

## 8.11 5.9 — Old HTTP Protocol Versions Disallowed

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Konsep Versi HTTP Lama

HTTP/1.1 sudah lama menjadi standar umum, sedangkan HTTP/1.0 dan versi lebih lama dianggap kuno untuk banyak skenario modern.

### Risiko Versi Lama atau Invalid

Scanner dan tool attacker sering mengirim request dengan versi protocol abnormal untuk melihat respons server. Ini bagian dari fingerprinting dan enumeration.

### Prinsip Konseptual

Server sebaiknya menolak request dengan versi HTTP yang lama, tidak valid, atau tidak sesuai kebutuhan. Ini mengurangi respons terhadap trafik abnormal.

## 8.12 5.10 — Access to `.ht*` Files Restricted

Rekomendasi ini bersifat **Automated** dan masuk **Level 1**.

### Apa itu `.ht*`

File `.ht*` mencakup file seperti:

- `.htaccess`,
- `.htpasswd`,
- `.htgroup`.

File ini dapat berisi konfigurasi, password hash, atau data access control.

### Risiko

Jika file `.htaccess` atau `.htpasswd` dapat dibaca melalui web, informasi sensitif bisa bocor.

### Prinsip Konseptual

Walaupun `.htaccess` tidak digunakan, akses ke file `.ht*` tetap harus diblokir sebagai defense in depth.

## 8.13 5.11 — Access to `.git` Files Restricted

Rekomendasi ini bersifat **Manual** dan masuk **Level 1**.

### Apa itu `.git`

`.git` adalah direktori metadata repository Git. Jika berada di document root dan dapat diakses, attacker dapat mengunduh source code atau riwayat perubahan.

### Risiko

Kebocoran `.git` dapat membuka:

- source code aplikasi,
- credential yang pernah commit,
- endpoint internal,
- struktur framework,
- bug lama,
- secret yang sudah dihapus dari file saat ini tetapi masih ada di history.

### Prinsip Konseptual

Repository metadata tidak boleh disajikan oleh web server. Jika tidak sengaja ada di web root, akses harus diblokir.

## 8.14 5.12 — Access to `.svn` Files Restricted

Rekomendasi ini bersifat **Manual** dan masuk **Level 1**.

### Apa itu `.svn`

`.svn` adalah direktori metadata Subversion.

### Risiko

Risikonya mirip `.git`: source code dan metadata development dapat bocor.

### Prinsip Konseptual

Metadata version control bukan konten web. Akses harus dibatasi.

## 8.15 5.13 — Access to Inappropriate File Extensions Restricted

Rekomendasi ini bersifat **Manual** dan masuk **Level 2**.

### Konsep File Extension Allowlist

CIS menyarankan pendekatan allowlist: hanya ekstensi file yang memang sah untuk website yang boleh diakses.

Contoh file yang sering tertinggal dan berisiko:

- `.bak`,
- `.old`,
- `.config`,
- `.sql`,
- `.tar`,
- `.zip`,
- `.swp`,
- file backup editor,
- file konfigurasi sementara.

### Risiko

File seperti ini dapat membocorkan:

- source code,
- kredensial database,
- konfigurasi aplikasi,
- backup data,
- struktur internal.

### Prinsip Konseptual

Web server sebaiknya hanya menyajikan jenis file yang memang diizinkan. Jangan mengandalkan asumsi bahwa attacker tidak tahu nama file.

## 8.16 5.14 — IP Address Based Requests Disallowed

Rekomendasi ini bersifat **Automated** dan masuk **Level 2**.

### Konsep Request Berbasis IP

Request berbasis IP terjadi ketika client mengakses website dengan alamat IP langsung, bukan hostname.

Contoh:

- `http://192.0.2.10/`
- bukan `https://www.example.com/`

### Risiko

Scanner otomatis dan malware sering menggunakan IP address untuk mencari server aktif. Request berbasis IP dapat melewati asumsi virtual host dan hostname.

### Prinsip Konseptual

Server produksi sebaiknya merespons hanya hostname yang memang sah. Ini membantu mengurangi scanning otomatis dan akses tidak sesuai virtual host.

## 8.17 5.15 — IP Addresses for Listening Requests Specified

Rekomendasi ini bersifat **Automated** dan masuk **Level 2**.

### Konsep Listen Directive

`Listen` menentukan IP address dan port tempat Apache menerima request.

Jika Apache listen pada semua interface, server mungkin terbuka pada jaringan yang tidak dimaksudkan.

### Risiko Listen Terlalu Luas

Server dengan banyak interface dapat tidak sengaja mendengarkan pada:

- interface internal,
- interface management,
- interface VPN,
- IPv6 address,
- jaringan yang tidak diproteksi.

### Prinsip Konseptual

Apache harus listen hanya pada IP dan port yang memang direncanakan. Ini memperjelas exposure jaringan.

## 8.18 5.16 — Browser Framing Restricted

Rekomendasi ini bersifat **Automated** dan masuk **Level 2**.

### Konsep Browser Framing

Framing terjadi ketika halaman web dimuat di dalam frame atau iframe milik halaman lain.

### Risiko Clickjacking

Attacker dapat membuat halaman palsu yang memuat website sah dalam iframe, lalu menipu user agar mengklik sesuatu. Ini disebut clickjacking atau UI redressing.

### Header yang Digunakan

CIS menyebut dua pendekatan:

- `Content-Security-Policy` dengan `frame-ancestors`,
- `X-Frame-Options` dengan nilai seperti `DENY` atau `SAMEORIGIN`.

CSP lebih disukai karena lebih modern dan fleksibel.

### Prinsip Konseptual

Website harus menentukan siapa yang boleh menampilkan halamannya dalam frame. Jika tidak ada kebutuhan, framing dari situs lain harus ditolak.

## 8.19 5.17 — HTTP Header Referrer-Policy Set Appropriately

Rekomendasi ini bersifat **Manual** dan masuk **Level 2**.

### Konsep Referrer-Policy

Referrer-Policy mengatur seberapa banyak informasi URL asal yang dikirim browser saat berpindah halaman atau memuat resource.

### Risiko Referrer Berlebihan

Referrer dapat membocorkan:

- path internal,
- parameter query,
- token yang salah diletakkan di URL,
- struktur aplikasi,
- informasi halaman sensitif.

### Kenapa Manual

Nilai Referrer-Policy harus disesuaikan dengan kebutuhan aplikasi. Jika terlalu ketat, integrasi tertentu bisa terganggu. Jika terlalu longgar, informasi bisa bocor.

### Prinsip Konseptual

Kirim hanya informasi referrer yang dibutuhkan. Jangan membocorkan detail URL sensitif ke pihak lain.

## 8.20 5.18 — HTTP Header Permissions-Policy Set Appropriately

Rekomendasi ini bersifat **Manual** dan masuk **Level 2**.

### Konsep Permissions-Policy

Permissions-Policy mengatur fitur browser apa saja yang boleh digunakan oleh halaman, misalnya akses kamera, mikrofon, geolocation, fullscreen, payment, dan fitur lain tergantung browser.

### Risiko Tanpa Permissions-Policy

Jika tidak dibatasi, browser feature tertentu mungkin tersedia lebih luas dari yang dibutuhkan aplikasi. Ini dapat memperbesar dampak XSS, iframe abuse, atau misuse fitur browser.

### Kenapa Manual

Setiap aplikasi memiliki kebutuhan fitur berbeda. Aplikasi video conference mungkin membutuhkan kamera dan mikrofon, tetapi website katalog biasa tidak.

### Prinsip Konseptual

Permissions-Policy menerapkan zero trust pada fitur browser: fitur hanya boleh digunakan jika benar-benar dibutuhkan.

---

# 9. Ringkasan Konseptual Bagian 1

Bagian 1 CIS Apache HTTP Server 2.4 Benchmark dapat diringkas ke dalam beberapa prinsip besar.

## 9.1 Apache Harus Direncanakan, Bukan Sekadar Diinstal

Bab 1 menekankan bahwa keamanan Apache dimulai dari perencanaan:

- OS harus aman,
- jaringan harus dikontrol,
- patching harus berjalan,
- logging harus tersedia,
- server tidak boleh multi-use tanpa alasan,
- sumber instalasi harus terpercaya.

## 9.2 Modul Harus Minimal

Bab 2 menekankan bahwa setiap modul adalah tambahan attack surface.

Modul yang tidak dibutuhkan harus dinonaktifkan, terutama:

- WebDAV,
- status,
- autoindex,
- proxy jika tidak dipakai,
- userdir,
- info,
- Basic/Digest Auth.

Namun modul logging harus aktif karena visibility adalah bagian dari keamanan.

## 9.3 Apache Harus Berjalan dengan Privilege Terbatas

Bab 3 menekankan least privilege:

- Apache worker tidak berjalan sebagai root,
- akun Apache tidak bisa login,
- akun Apache dikunci,
- file Apache dimiliki root,
- permission tulis dibatasi,
- document root tidak writable oleh Apache group,
- direktori writable aplikasi harus dipisahkan dan dikontrol.

## 9.4 Access Control Harus Deny by Default

Bab 4 menekankan:

- OS root directory ditolak secara default,
- hanya web content yang perlu dibuka,
- `.htaccess` tidak dijadikan mekanisme konfigurasi utama,
- override dimatikan agar konfigurasi terpusat dan mudah diaudit.

## 9.5 Fitur dan Konten Harus Diperkecil

Bab 5 menekankan pengurangan fitur:

- options dibatasi,
- default content dihapus,
- CGI contoh dihapus,
- HTTP method dibatasi,
- TRACE dimatikan,
- versi HTTP lama ditolak,
- file sensitif seperti `.ht*`, `.git`, `.svn` diblokir,
- ekstensi file dikendalikan,
- request berbasis IP ditolak,
- IP listen dibuat eksplisit,
- browser security headers diterapkan.

## 9.6 Inti Utama

Inti dari Bagian 1 adalah:

> Apache yang aman adalah Apache yang fungsinya jelas, modulnya minimal, permission-nya ketat, aksesnya eksplisit, kontennya bersih, dan perilaku web-nya dibatasi sesuai kebutuhan.

CIS Benchmark tidak hanya memberi daftar konfigurasi, tetapi mengajarkan pola pikir hardening: **minimize, restrict, separate, document, and test**.

---

## Catatan Pembelajaran

Untuk pembahasan konseptual, yang paling penting bukan menghafal directive satu per satu, tetapi memahami alasan di balik rekomendasi:

- Kenapa fitur tidak perlu harus dimatikan?
- Kenapa file konfigurasi harus dimiliki root?
- Kenapa Apache tidak boleh menulis web root?
- Kenapa `.git` dan `.svn` berbahaya jika tersaji lewat web?
- Kenapa header keamanan browser termasuk bagian dari hardening server?

Jika prinsip-prinsip ini dipahami, maka penerapan praktis CIS Apache akan lebih mudah, karena setiap command atau konfigurasi dapat dikaitkan dengan tujuan keamanan yang jelas.
