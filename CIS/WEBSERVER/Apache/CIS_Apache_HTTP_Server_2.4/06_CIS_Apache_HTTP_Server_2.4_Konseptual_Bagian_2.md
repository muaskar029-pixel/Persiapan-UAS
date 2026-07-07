# 06 — CIS Apache HTTP Server 2.4 Benchmark: Penjelasan Konseptual Bagian 2

**Sumber utama:** CIS Apache HTTP Server 2.4 Benchmark v2.3.0 — 09-23-2025  
**Ruang lingkup file ini:** Bab 6 sampai Bab 12 dan appendix ringkas.  
**Fokus pembahasan:** konseptual, bukan panduan praktik perintah tahap demi tahap.

---

## Daftar Isi

1. [Bab 6 — Logging, Monitoring and Maintenance](#1-bab-6--logging-monitoring-and-maintenance)
2. [Bab 7 — SSL/TLS Configuration](#2-bab-7--ssltls-configuration)
3. [Bab 8 — Information Leakage](#3-bab-8--information-leakage)
4. [Bab 9 — Denial of Service Mitigations](#4-bab-9--denial-of-service-mitigations)
5. [Bab 10 — Request Limits](#5-bab-10--request-limits)
6. [Bab 11 — SELinux](#6-bab-11--selinux)
7. [Bab 12 — AppArmor](#7-bab-12--apparmor)
8. [Appendix Ringkas](#8-appendix-ringkas)
9. [Ringkasan Besar Bagian 2](#9-ringkasan-besar-bagian-2)

---

# 1. Bab 6 — Logging, Monitoring and Maintenance

Bab 6 membahas operasi harian Apache dari sisi **logging, monitoring, patching, dan perlindungan aplikasi web**. Setelah Apache dipasang dan dikonfigurasi dengan benar, pekerjaan keamanan belum selesai. Server harus tetap dipantau, log harus dapat dianalisis, patch harus diterapkan, dan serangan aplikasi web perlu dideteksi.

Secara konseptual, Bab 6 menekankan bahwa Apache bukan hanya layanan yang “menjawab request HTTP”, tetapi juga sumber bukti keamanan. Tanpa log yang baik, administrator sulit mengetahui apakah serangan terjadi, apakah percobaan scanning sedang berlangsung, apakah aplikasi menghasilkan error, atau apakah attacker berhasil mengakses resource tertentu.

## 1.1 Peran Logging dalam Keamanan Apache

Logging pada Apache memiliki beberapa fungsi utama:

- mencatat request yang masuk,
- mencatat error dan kondisi abnormal,
- membantu investigasi insiden,
- membantu monitoring performa dan kapasitas,
- membantu membedakan error biasa dengan pola serangan,
- menyediakan bukti untuk audit dan forensik.

Log tidak boleh dipahami hanya sebagai “catatan teknis”. Dalam konteks keamanan, log adalah **jejak kejadian**. Ketika terjadi insiden, log sering menjadi sumber utama untuk menjawab pertanyaan:

- kapan serangan terjadi,
- dari mana request berasal,
- URL apa yang diakses,
- status response apa yang diberikan,
- apakah request gagal atau berhasil,
- apakah ada pola scanning,
- apakah aplikasi mengembalikan error tertentu,
- apakah ada eksploitasi terhadap endpoint tertentu.

## 1.2 Error Log dan Severity Level

Rekomendasi 6.1 membahas konfigurasi **ErrorLog** dan **LogLevel**.

**ErrorLog** menentukan lokasi atau tujuan pencatatan error Apache. Error dapat disimpan ke file lokal, dikirim ke syslog, atau dipisah per virtual host sesuai kebutuhan operasional.

**LogLevel** menentukan tingkat detail error yang dicatat. Level log Apache mengikuti gaya syslog, mulai dari level paling kritis sampai paling verbose, seperti `emerg`, `alert`, `crit`, `error`, `warn`, `notice`, `info`, dan `debug`.

CIS merekomendasikan agar logging cukup detail untuk mendeteksi anomali, tetapi tetap harus disesuaikan dengan kemampuan storage dan monitoring. Nilai konseptual yang ditekankan adalah:

- log terlalu minim membuat insiden sulit dianalisis,
- log terlalu ramai dapat membebani storage dan membuat noise,
- log harus cukup informatif untuk mendeteksi percobaan akses tidak sah,
- kesalahan seperti banyak request `not found` atau `unauthorized` bisa menjadi tanda scanning atau probing.

Pada Apache 2.4, request yang tidak ditemukan dapat membutuhkan level tertentu agar masuk ke error log. Karena itu, CIS menekankan pentingnya konfigurasi yang memungkinkan error relevan tetap terekam.

## 1.3 Syslog Facility untuk Error Logging

Rekomendasi 6.2 membahas pengiriman error log ke **syslog facility**.

Secara konseptual, syslog membantu menyatukan log Apache dengan log sistem lain. Jika Apache hanya menulis log di file lokal yang jarang diperiksa, maka serangan aplikasi web dapat terlewat. Dengan mengirim error log ke syslog, organisasi lebih mudah mengintegrasikannya ke proses monitoring yang sudah ada, seperti SIEM atau log analytics.

Tujuannya bukan sekadar memindahkan lokasi log, tetapi memastikan log Apache masuk ke alur deteksi yang sama dengan log sistem, firewall, autentikasi, dan aplikasi lain.

Hal penting:

- error log aplikasi web sering berisi tanda awal serangan,
- broken link, internal error, request aneh, dan probing dapat terlihat di log,
- log yang terpusat lebih sulit hilang jika server lokal rusak atau dikompromikan,
- syslog membantu korelasi antar-sumber log.

Rekomendasi ini termasuk Level 2 karena integrasi syslog/SIEM biasanya memerlukan proses operasional yang lebih matang.

## 1.4 Access Log Apache

Rekomendasi 6.3 membahas **access log**.

Error log menunjukkan kesalahan, tetapi access log menunjukkan aktivitas request. Jika hanya error yang dicatat, administrator dapat melihat request gagal, tetapi tidak dapat memastikan apakah request berikutnya berhasil. Karena itu, access log penting untuk investigasi.

Access log biasanya berisi:

- alamat host atau IP client,
- identitas remote jika tersedia,
- user terautentikasi jika ada,
- waktu request,
- baris request pertama,
- status response,
- ukuran response,
- referer,
- user-agent.

Dalam praktik modern, jika Apache berada di belakang load balancer atau reverse proxy, IP client yang terlihat di `%h` bisa jadi hanya IP load balancer. Secara konseptual, ini berarti administrator perlu memastikan log menangkap identitas client yang benar, misalnya melalui header seperti `X-Forwarded-For` jika memang arsitekturnya memerlukan itu.

Tanpa access log yang benar, investigasi menjadi lemah karena organisasi tidak tahu secara lengkap request apa yang masuk dan response apa yang diberikan.

## 1.5 Log Storage dan Log Rotation

Rekomendasi 6.4 membahas penyimpanan dan rotasi log.

CIS menekankan bahwa log harus memiliki ruang penyimpanan yang cukup dan harus dirotasi dengan benar. Penyimpanan log tidak boleh sembarangan karena attacker dapat memicu banyak request untuk membanjiri log. Jika log Apache disimpan di partisi root dan partisi tersebut penuh, server bisa mengalami gangguan layanan.

Konsep utamanya:

- log harus memiliki kapasitas penyimpanan memadai,
- log idealnya tidak memenuhi root filesystem,
- rotasi log mencegah file log tumbuh tanpa batas,
- retensi log harus cukup untuk investigasi,
- jika tidak ada central logging, CIS menekankan retensi minimal sekitar 3 bulan atau 13 minggu,
- virtual host yang memiliki log sendiri juga harus masuk dalam kebijakan rotasi.

Log rotation bukan hanya administrasi file. Dalam keamanan, log rotation adalah keseimbangan antara **ketersediaan bukti** dan **ketersediaan sistem**. Log yang terlalu cepat dihapus menghambat forensik. Log yang tidak dirotasi dapat menyebabkan disk penuh.

## 1.6 Patching Apache

Rekomendasi 6.5 membahas penerapan patch.

Patching adalah bagian utama dari vulnerability management. Konfigurasi aman tidak cukup jika Apache atau modulnya memiliki vulnerability yang sudah diketahui. CIS menekankan bahwa patch Apache yang tersedia harus diterapkan dalam jangka waktu yang wajar setelah tersedia.

Secara konseptual:

- vulnerability baru dapat muncul setelah sistem dianggap aman,
- patch harus diuji sebelum produksi,
- update dari vendor distribusi biasanya lebih mudah dikelola daripada build source manual,
- Apache yang dibangun dari source membutuhkan proses pemantauan patch yang lebih disiplin,
- modul tambahan seperti ModSecurity atau modul pihak ketiga juga perlu diperbarui.

Patching bukan hanya “upgrade versi”. Patching adalah proses yang mencakup pemantauan advisory, pengujian, deployment, rollback plan, dan verifikasi.

## 1.7 ModSecurity sebagai Web Application Firewall

Rekomendasi 6.6 membahas **ModSecurity**.

ModSecurity adalah web application firewall atau WAF yang bekerja di level aplikasi web. Ia dapat membantu memonitor, mencatat, dan memblokir pola request yang berbahaya. Namun, CIS menekankan bahwa ModSecurity tanpa rule set tidak memberikan perlindungan berarti.

Konsep penting:

- ModSecurity adalah framework WAF,
- perlindungan efektif membutuhkan rule set,
- rule harus dituning agar tidak terlalu banyak false positive,
- WAF bukan pengganti secure coding,
- WAF membantu defense-in-depth.

ModSecurity dapat membantu mendeteksi serangan umum seperti SQL injection, cross-site scripting, request abnormal, eksploitasi parameter, dan pola serangan aplikasi web lainnya. Namun, jika tidak dikelola dengan baik, WAF dapat menghasilkan banyak alert tidak relevan atau bahkan memblokir fungsi aplikasi yang sah.

## 1.8 OWASP ModSecurity Core Rule Set

Rekomendasi 6.7 membahas **OWASP ModSecurity Core Rule Set** atau CRS.

CRS adalah kumpulan aturan terbuka untuk ModSecurity. CRS menyediakan baseline proteksi terhadap berbagai kategori ancaman, seperti:

- pelanggaran protokol HTTP,
- request yang mencurigakan,
- HTTP flood dan slow HTTP DoS,
- serangan web umum,
- bot, crawler, dan scanner berbahaya,
- file upload berbahaya,
- potensi kebocoran data sensitif,
- akses terhadap trojan atau backdoor,
- error aplikasi yang bisa membocorkan informasi.

CIS juga menekankan bahwa CRS perlu tuning. Aturan WAF yang terlalu agresif dapat mengganggu aplikasi. Aturan yang terlalu longgar tidak cukup melindungi. Karena itu, implementasi CRS harus melibatkan administrator, security specialist, dan pengembang aplikasi.

## 1.9 Ringkasan Bab 6

Bab 6 menempatkan Apache dalam siklus operasi keamanan:

- log harus lengkap,
- error harus terlihat,
- access log harus dapat diinvestigasi,
- log harus disimpan dan dirotasi dengan baik,
- patch harus diterapkan,
- WAF dapat digunakan sebagai lapisan tambahan,
- ModSecurity dan CRS perlu tuning berkelanjutan.

Intinya, Apache yang aman bukan hanya Apache yang dikonfigurasi sekali, tetapi Apache yang **dipantau, diperbarui, dan dianalisis secara terus-menerus**.

---

# 2. Bab 7 — SSL/TLS Configuration

Bab 7 membahas konfigurasi SSL/TLS pada Apache. SSL/TLS berfungsi melindungi komunikasi antara client dan server melalui enkripsi, autentikasi server, dan perlindungan terhadap manipulasi data selama transit.

Namun CIS menekankan poin penting: menggunakan SSL/TLS tidak otomatis membuat web server aman. TLS melindungi data dalam perjalanan, tetapi tidak memperbaiki aplikasi yang rentan, tidak mencegah SQL injection, dan tidak melindungi data setelah sudah sampai di server.

Dengan kata lain:

> TLS melindungi jalur komunikasi, bukan seluruh sistem.

## 2.1 mod_ssl dan mod_nss

Rekomendasi 7.1 membahas keberadaan modul SSL/TLS.

Apache biasanya menggunakan **mod_ssl** untuk menyediakan kemampuan SSL/TLS. Pada beberapa lingkungan tertentu, ada juga modul **mod_nss** yang menggunakan Network Security Services dari Mozilla.

Konsep penting:

- Apache membutuhkan modul TLS agar bisa melayani HTTPS,
- HTTPS diperlukan untuk melindungi kredensial, session cookie, dan data sensitif,
- server juga dapat diautentikasi melalui sertifikat,
- attacker tetap dapat menyerang aplikasi melalui HTTPS, hanya saja traffic-nya terenkripsi.

Karena itu, TLS harus dipahami sebagai bagian dari keamanan, bukan satu-satunya keamanan.

## 2.2 Sertifikat Valid dan Tepercaya

Rekomendasi 7.2 membahas sertifikat digital.

Sertifikat yang aman harus:

- ditandatangani oleh certificate authority yang dipercaya,
- belum kedaluwarsa,
- memiliki nama host yang sesuai dengan domain yang diakses,
- tidak bergantung pada algoritma tanda tangan yang lemah,
- memiliki chain certificate yang valid.

Sertifikat self-signed mungkin berguna untuk lab, tetapi tidak cocok untuk layanan publik karena browser tidak dapat mempercayainya secara otomatis. Jika user terbiasa menerima warning sertifikat, maka risiko phishing dan man-in-the-middle meningkat.

Secara konseptual, sertifikat berfungsi untuk menjawab pertanyaan:

> “Apakah client benar-benar sedang terhubung ke server yang dimaksud?”

Tanpa sertifikat yang valid, enkripsi tetap mungkin terjadi, tetapi identitas server tidak dapat dipercaya.

## 2.3 Perlindungan Private Key

Rekomendasi 7.3 membahas private key server.

Private key adalah aset kriptografi yang sangat sensitif. Jika private key bocor, attacker dapat menyamar sebagai server atau mendekripsi komunikasi tertentu dalam kondisi tertentu, tergantung cipher dan protokol yang digunakan.

Karena itu private key harus:

- disimpan dengan permission ketat,
- hanya dapat dibaca oleh user yang benar-benar membutuhkan,
- tidak berada di lokasi yang dapat diakses web,
- tidak ikut tersalin ke backup atau repository yang tidak aman,
- diperlakukan seperti rahasia tingkat tinggi.

Public certificate boleh dibagikan, tetapi private key tidak boleh bocor.

## 2.4 Menonaktifkan TLS Lama

Rekomendasi 7.4 membahas TLSv1.0 dan TLSv1.1.

Versi TLS lama memiliki kelemahan desain dan tidak lagi cocok untuk standar keamanan modern. CIS menekankan bahwa TLSv1.0 dan TLSv1.1 harus dinonaktifkan agar server tidak dapat dinegosiasikan turun ke protokol lama.

Konsep penting:

- attacker dapat mencoba downgrade protocol,
- client lama mungkin hanya mendukung TLS usang,
- kompatibilitas lama harus ditimbang dengan risiko keamanan,
- layanan modern sebaiknya menggunakan TLS yang lebih baru dan kuat.

Dalam hardening, dukungan terhadap client sangat lama sering menjadi trade-off. Jika organisasi masih mendukung client lama, pengecualian harus didokumentasikan.

## 2.5 Menonaktifkan Cipher Lemah

Rekomendasi 7.5 membahas cipher SSL/TLS yang lemah.

Cipher suite menentukan algoritma yang digunakan untuk key exchange, autentikasi, enkripsi, dan integrity. Cipher yang lemah dapat membuat komunikasi lebih mudah diserang.

Contoh kategori yang biasanya dihindari:

- cipher null,
- export-grade cipher,
- RC4,
- DES/3DES dalam banyak kebijakan modern,
- algoritma dengan kekuatan kriptografi rendah,
- kombinasi yang tidak lagi direkomendasikan.

Tujuannya adalah memastikan server tidak menawarkan pilihan cipher yang dapat dieksploitasi attacker atau dipilih oleh client lama secara tidak aman.

## 2.6 Insecure SSL Renegotiation

Rekomendasi 7.6 membahas renegotiation yang tidak aman.

SSL/TLS renegotiation adalah proses negosiasi ulang parameter koneksi. Fitur ini pernah memiliki masalah keamanan serius karena attacker dapat menyisipkan data tertentu ke dalam sesi TLS pada implementasi yang rentan.

Secara konseptual:

- renegotiation yang tidak aman dapat membuka peluang manipulasi sesi,
- konfigurasi harus memastikan renegotiation yang rentan tidak aktif,
- server harus memakai library TLS yang sudah diperbaiki,
- kompatibilitas dengan client lama tidak boleh mengorbankan keamanan.

## 2.7 SSL Compression

Rekomendasi 7.7 membahas SSL compression.

Kompresi pada lapisan TLS pernah dikaitkan dengan serangan yang dapat membocorkan informasi melalui analisis ukuran traffic, seperti keluarga serangan CRIME. Karena itu, SSL compression tidak boleh diaktifkan.

Konsepnya sederhana:

> Kompresi dapat membuat pola data sensitif terlihat dari perubahan ukuran response.

Walaupun data terenkripsi, informasi sampingan seperti ukuran dan pola dapat memberi petunjuk kepada attacker.

## 2.8 Medium Strength Cipher

Rekomendasi 7.8 membahas cipher dengan kekuatan menengah.

CIS memisahkan cipher lemah dan medium-strength cipher untuk memperketat konfigurasi TLS. Cipher yang dahulu dianggap cukup dapat menjadi tidak layak seiring meningkatnya kemampuan komputasi dan berkembangnya teknik kriptanalisis.

Konsep penting:

- standar kriptografi berubah seiring waktu,
- konfigurasi TLS harus ditinjau berkala,
- cipher yang “tidak lemah sekali” belum tentu sesuai standar keamanan tinggi,
- Level 2 biasanya lebih ketat terhadap kompatibilitas.

## 2.9 Semua Konten Melalui HTTPS

Rekomendasi 7.9 membahas kewajiban mengakses konten melalui HTTPS.

Jika sebagian halaman menggunakan HTTP dan sebagian HTTPS, maka pengguna masih dapat terkena risiko:

- cookie dicuri melalui koneksi HTTP,
- session downgrade,
- mixed content,
- manipulasi konten oleh pihak di jaringan,
- user diarahkan ke versi tidak aman.

Konsep utama:

> Situs yang menangani data sensitif sebaiknya konsisten menggunakan HTTPS untuk seluruh konten, bukan hanya halaman login.

HTTP dapat digunakan untuk redirect ke HTTPS, tetapi konten utama sebaiknya disajikan melalui HTTPS.

## 2.10 OCSP Stapling

Rekomendasi 7.10 membahas OCSP Stapling.

OCSP digunakan untuk memeriksa apakah sertifikat telah dicabut. Tanpa OCSP Stapling, browser/client perlu menghubungi pihak CA untuk memeriksa status sertifikat. Dengan OCSP Stapling, server menyertakan bukti status sertifikat yang masih valid dalam proses TLS handshake.

Manfaat konseptual:

- mempercepat validasi sertifikat,
- mengurangi ketergantungan client pada koneksi ke CA,
- meningkatkan privasi karena client tidak selalu perlu menghubungi CA,
- membantu memastikan sertifikat tidak dicabut.

## 2.11 HTTP Strict Transport Security

Rekomendasi 7.11 membahas **HSTS**.

HSTS adalah header keamanan yang memberi tahu browser agar selalu menggunakan HTTPS untuk domain tertentu selama periode tertentu. Ini mencegah user atau attacker memaksa browser kembali ke HTTP.

Konsep penting:

- HSTS membantu mencegah SSL stripping,
- browser mengingat bahwa domain harus diakses melalui HTTPS,
- `includeSubDomains` memperluas kebijakan ke subdomain,
- `preload` dapat digunakan untuk daftar preload browser, tetapi perlu kehati-hatian.

HSTS kuat, tetapi harus diterapkan dengan matang. Jika konfigurasi HTTPS belum stabil, HSTS dapat membuat user tidak bisa mengakses layanan.

## 2.12 Forward Secrecy

Rekomendasi 7.12 membahas cipher suite dengan **Forward Secrecy**.

Forward Secrecy memastikan bahwa kompromi private key server di masa depan tidak otomatis membuka semua rekaman komunikasi lama. Cipher suite yang menggunakan ephemeral Diffie-Hellman seperti ECDHE atau DHE mendukung konsep ini.

Tanpa forward secrecy, jika private key bocor, rekaman traffic lama yang pernah disadap dapat berisiko lebih besar. Dengan forward secrecy, setiap sesi memiliki material kunci yang unik dan tidak bergantung sepenuhnya pada private key jangka panjang.

Konsep penting:

- ECDHE umumnya lebih disukai,
- DHE dapat dipakai untuk kompatibilitas tertentu,
- cipher suite tanpa FS sebaiknya dihindari untuk keamanan tinggi,
- FS memperkuat perlindungan jangka panjang terhadap data in transit.

## 2.13 Ringkasan Bab 7

Bab 7 menekankan bahwa TLS harus dikonfigurasi secara aman, bukan sekadar diaktifkan. Komponen pentingnya adalah:

- modul TLS tersedia,
- sertifikat valid,
- private key terlindungi,
- protokol lama dinonaktifkan,
- cipher lemah dan menengah dihindari,
- compression dan renegotiation tidak aman dimatikan,
- semua konten memakai HTTPS,
- OCSP Stapling dan HSTS dipertimbangkan,
- forward secrecy digunakan.

---

# 3. Bab 8 — Information Leakage

Bab 8 membahas pencegahan kebocoran informasi dari Apache. Kebocoran informasi tidak selalu berarti data rahasia seperti password bocor. Informasi teknis seperti versi Apache, modul, path, inode, default content, atau struktur server juga bisa membantu attacker.

Dalam keamanan, informasi kecil dapat menjadi bagian dari reconnaissance. Semakin banyak detail yang diketahui attacker, semakin mudah mereka memilih exploit yang cocok.

## 3.1 ServerTokens

Rekomendasi 8.1 membahas direktif **ServerTokens**.

ServerTokens mengatur seberapa banyak informasi server yang muncul pada HTTP response header. Jika terlalu detail, Apache dapat menampilkan informasi seperti versi Apache, modul, sistem operasi, atau komponen tambahan.

CIS merekomendasikan nilai minimal seperti `Prod` atau `ProductOnly`, sehingga header tidak memberikan informasi versi yang tidak perlu.

Konsep penting:

- informasi versi membantu attacker memilih exploit,
- scanner internet sering mengumpulkan banner server,
- semakin detail fingerprint server, semakin mudah targeting,
- pengurangan informasi bukan pengganti patching, tetapi mengurangi efisiensi reconnaissance.

## 3.2 ServerSignature

Rekomendasi 8.2 membahas **ServerSignature**.

ServerSignature dapat menambahkan informasi server pada halaman error atau dokumen yang dihasilkan Apache. Walaupun terlihat kecil, signature ini dapat mengungkap detail server kepada pengguna atau attacker.

CIS mendorong agar signature tidak diaktifkan. Tujuannya sama dengan ServerTokens: mengurangi informasi yang tidak perlu.

Perbedaannya:

- ServerTokens berhubungan dengan header response,
- ServerSignature berhubungan dengan footer pada halaman yang dihasilkan server.

## 3.3 Default Apache Content

Rekomendasi 8.3 membahas penghapusan konten default Apache.

Default content dapat berupa halaman contoh, ikon, manual, CGI default, atau file yang menunjukkan struktur dan jenis server. Walaupun tidak selalu berbahaya langsung, konten default membantu attacker mengenali server.

Contoh risiko:

- ikon default menunjukkan server adalah Apache,
- file contoh dapat menunjukkan versi atau path,
- default directory dapat mengaktifkan fitur yang tidak diperlukan,
- file demo kadang memiliki kelemahan atau informasi sensitif.

Secara konseptual, server produksi seharusnya hanya berisi konten yang memang dibutuhkan aplikasi.

## 3.4 ETag dan Inode

Rekomendasi 8.4 membahas **ETag**.

ETag digunakan dalam mekanisme cache HTTP. Pada Apache, ETag dapat dibentuk dari metadata file seperti waktu modifikasi, ukuran file, dan inode. Jika inode ikut masuk dalam ETag, server dapat membocorkan informasi internal filesystem.

Walaupun inode tampak kecil, informasi ini dapat membantu attacker dalam teknik lanjutan atau fingerprinting.

Konsep penting:

- ETag berguna untuk caching,
- ETag tidak boleh membocorkan detail filesystem,
- nilai seperti inode sebaiknya tidak disertakan,
- konfigurasi aman dapat menggunakan `MTime Size` atau menghapus FileETag sesuai kebutuhan.

## 3.5 Ringkasan Bab 8

Bab 8 mengajarkan prinsip:

> Jangan berikan informasi teknis yang tidak dibutuhkan client.

Apache harus menjawab request, bukan memberi peta internal kepada attacker. Mengurangi informasi versi, signature, konten default, dan metadata filesystem membantu memperkecil risiko reconnaissance.

---

# 4. Bab 9 — Denial of Service Mitigations

Bab 9 membahas mitigasi **Denial of Service** atau DoS. DoS bertujuan membuat layanan tidak tersedia dengan menghabiskan resource seperti koneksi, CPU, memori, bandwidth, atau thread worker.

CIS tidak mengklaim konfigurasi Apache dapat mencegah semua DoS. Serangan besar tetap membutuhkan perlindungan jaringan, CDN, load balancer, rate limiting, atau mitigasi upstream. Namun konfigurasi Apache dapat membuat server lebih tahan terhadap serangan aplikasi sederhana dan penyalahgunaan koneksi.

## 4.1 TimeOut

Rekomendasi 9.1 membahas direktif **TimeOut**.

TimeOut menentukan berapa lama Apache menunggu operasi tertentu sebelum menutup koneksi. Jika nilai terlalu tinggi, koneksi lambat atau menggantung dapat menahan resource terlalu lama.

CIS merekomendasikan nilai yang lebih rendah, misalnya 10 detik atau kurang, untuk membantu server membebaskan resource lebih cepat.

Konsep penting:

- koneksi yang menggantung dapat menghabiskan worker,
- timeout yang terlalu panjang membantu slow connection attack,
- timeout yang terlalu pendek dapat mengganggu client lambat atau aplikasi tertentu,
- pengaturan harus diuji sesuai karakter aplikasi.

## 4.2 KeepAlive

Rekomendasi 9.2 membahas **KeepAlive**.

KeepAlive memungkinkan beberapa request menggunakan koneksi TCP yang sama. Ini mengurangi overhead pembuatan koneksi baru. Dalam konteks DoS, efisiensi koneksi dapat membantu server melayani request normal dengan resource lebih sedikit.

Konsep penting:

- KeepAlive mengurangi biaya setup TCP,
- koneksi yang dipakai ulang dapat meningkatkan performa,
- KeepAlive harus dikombinasikan dengan timeout yang masuk akal,
- tanpa batasan, koneksi tetap dapat disalahgunakan.

## 4.3 MaxKeepAliveRequests

Rekomendasi 9.3 membahas **MaxKeepAliveRequests**.

Direktif ini membatasi jumlah request yang boleh dikirim melalui satu koneksi KeepAlive. Jika nilainya 0, request tidak dibatasi. CIS menyarankan nilai 100 atau lebih agar reuse koneksi tetap efisien tetapi tidak tanpa kendali.

Konsep penting:

- terlalu rendah mengurangi manfaat KeepAlive,
- terlalu tinggi atau unlimited dapat dipertimbangkan sesuai workload,
- nilai harus mendukung efisiensi tanpa membuka peluang abuse.

## 4.4 KeepAliveTimeout

Rekomendasi 9.4 membahas **KeepAliveTimeout**.

Ini menentukan berapa lama Apache menunggu request berikutnya pada koneksi KeepAlive sebelum koneksi ditutup. Nilai yang terlalu lama membuat banyak koneksi idle menahan resource.

CIS merekomendasikan 15 detik atau kurang. Default Apache umumnya lebih rendah, tetapi perlu tetap diverifikasi.

Konsep penting:

- koneksi idle harus cepat dilepas,
- timeout terlalu panjang membuka peluang resource exhaustion,
- timeout terlalu pendek bisa mengurangi performa untuk client tertentu.

## 4.5 Request Header Timeout

Rekomendasi 9.5 membahas timeout untuk header request melalui **RequestReadTimeout**.

Serangan seperti Slowloris mengirim header HTTP sangat lambat agar koneksi tetap terbuka dan worker server tertahan. Dengan request header timeout, Apache dapat menutup koneksi yang tidak mengirim header secara wajar.

CIS merekomendasikan maximum header timeout 40 detik atau kurang.

Konsep penting:

- slow request attack membutuhkan bandwidth rendah,
- timeout header mencegah koneksi setengah jadi bertahan terlalu lama,
- TLS handshake perlu dipertimbangkan pada virtual host HTTPS,
- konfigurasi harus diuji agar tidak memutus client sah.

## 4.6 Request Body Timeout

Rekomendasi 9.6 membahas timeout untuk body request.

Timeout header saja tidak cukup. Attacker dapat mengirim header normal, lalu mengirim body request sangat lambat, misalnya pada serangan slow POST. Karena itu body request juga harus memiliki batas waktu.

CIS merekomendasikan maximum body timeout 20 detik atau kurang.

Konsep penting:

- upload dan POST request harus dibatasi secara wajar,
- aplikasi yang menerima upload besar perlu pengecualian atau tuning,
- body timeout membantu mencegah worker tertahan terlalu lama.

## 4.7 Ringkasan Bab 9

Bab 9 berfokus pada efisiensi resource dan pembatasan koneksi:

- koneksi lama harus ditutup,
- KeepAlive harus efisien,
- request lambat harus dibatasi,
- header dan body request harus punya timeout,
- semua pengaturan harus diuji terhadap aplikasi nyata.

Tujuannya bukan membuat Apache kebal DoS, tetapi meningkatkan ketahanan terhadap serangan yang menyalahgunakan koneksi HTTP.

---

# 5. Bab 10 — Request Limits

Bab 10 membahas pembatasan ukuran request. Request yang terlalu besar dapat membebani server atau mengeksploitasi aplikasi yang tidak memproses input dengan aman.

Request limits berhubungan dengan prinsip:

> Server hanya boleh menerima input sebesar yang benar-benar diperlukan.

Namun CIS juga menekankan bahwa pembatasan ini berisiko mengganggu aplikasi. Karena itu Bab 10 harus diuji di lingkungan test sebelum diterapkan ke produksi.

## 5.1 LimitRequestLine

Rekomendasi 10.1 membahas **LimitRequestLine**.

Request line terdiri dari HTTP method, URI, dan versi protokol. Jika request line terlalu panjang, request dapat membebani server atau diteruskan ke aplikasi yang rentan terhadap input berlebihan.

CIS merekomendasikan nilai 8190 atau kurang, tetapi tidak 0.

Konsep penting:

- URL terlalu panjang dapat menjadi indikator abuse,
- pembatasan membantu mencegah input abnormal,
- aplikasi yang memakai URL sangat panjang perlu diuji,
- limit tidak boleh 0 karena 0 berarti tidak terbatas atau perilaku tidak aman sesuai konteks direktif.

## 5.2 LimitRequestFields

Rekomendasi 10.2 membahas **LimitRequestFields**.

Direktif ini membatasi jumlah header field dalam satu request. Header terlalu banyak dapat membebani parsing atau mengeksploitasi aplikasi/modul yang tidak siap.

CIS merekomendasikan 100 atau kurang, tetapi tidak 0.

Konsep penting:

- banyak header dapat menjadi teknik abuse,
- batas harus cukup untuk browser dan aplikasi normal,
- server-level directive membuat tuning per bagian aplikasi tidak selalu fleksibel.

## 5.3 LimitRequestFieldSize

Rekomendasi 10.3 membahas **LimitRequestFieldSize**.

Direktif ini membatasi ukuran masing-masing header field. Header yang terlalu besar dapat dipakai untuk menyerang modul, CGI, reverse proxy, atau aplikasi backend.

CIS merekomendasikan 8190 byte atau kurang.

Konsep penting:

- header seperti cookie dapat besar,
- aplikasi dengan cookie atau token besar perlu pengujian,
- limit terlalu longgar mengurangi manfaat keamanan,
- limit terlalu ketat dapat memutus request sah.

## 5.4 LimitRequestBody

Rekomendasi 10.4 membahas **LimitRequestBody**.

Direktif ini membatasi ukuran body request. Body request muncul pada POST, PUT, upload file, form submission, dan API request.

CIS merekomendasikan 102400 byte atau kurang tetapi tidak 0. Nilai 0 berarti tidak terbatas.

Konsep penting:

- body terlalu besar dapat memakan resource,
- upload file perlu kebijakan khusus,
- API tertentu mungkin membutuhkan batas berbeda,
- pembatasan bisa diterapkan per direktori atau lokasi sesuai kebutuhan.

## 5.5 Ringkasan Bab 10

Bab 10 mengatur batas ukuran input:

- request line,
- jumlah header,
- ukuran header,
- ukuran body.

Pembatasan ini membantu mencegah input abnormal dan mengurangi peluang eksploitasi, tetapi harus diuji karena sangat mungkin berdampak ke fungsi aplikasi.

---

# 6. Bab 11 — SELinux

Bab 11 membahas penggunaan **SELinux** untuk membatasi proses Apache. SELinux adalah mekanisme Mandatory Access Control atau MAC yang memberikan kontrol akses tambahan di luar permission Linux biasa.

CIS menekankan bahwa SELinux dan AppArmor memiliki fungsi serupa dan tidak disarankan digunakan bersamaan pada sistem yang sama. Pilihan biasanya bergantung pada distribusi Linux. RHEL dan turunannya umum memakai SELinux, sedangkan Debian/Ubuntu lebih umum memakai AppArmor.

## 6.1 Konsep SELinux untuk Apache

Permission Linux biasa menggunakan model Discretionary Access Control atau DAC. Dalam DAC, owner file dapat menentukan permission. Jika proses Apache berhasil dikompromikan dan memiliki akses DAC ke file tertentu, attacker dapat memanfaatkan akses tersebut.

SELinux menambahkan lapisan MAC. Walaupun permission Linux mengizinkan, SELinux masih dapat menolak akses jika policy tidak mengizinkan.

Konsep penting:

- SELinux menerapkan deny-by-default berbasis policy,
- proses memiliki domain/type,
- file dan resource memiliki label/type,
- Apache biasanya dibatasi dalam domain seperti `httpd_t`,
- file web harus memiliki label yang sesuai agar dapat dibaca,
- direktori log, script, port, dan konten memiliki type berbeda.

## 6.2 SELinux Enforcing Mode

Rekomendasi 11.1 membahas SELinux dalam mode enforcing.

Mode utama SELinux:

- **disabled**: SELinux tidak aktif,
- **permissive**: pelanggaran hanya dicatat, tidak diblokir,
- **enforcing**: pelanggaran diblokir sesuai policy.

CIS menekankan mode enforcing karena hanya mode ini yang benar-benar membatasi proses. Permissive berguna untuk testing, tetapi tidak memberikan perlindungan nyata di produksi.

## 6.3 Apache dalam Context httpd_t

Rekomendasi 11.2 membahas proses Apache agar berjalan dalam context terbatas seperti `httpd_t`.

Jika Apache berjalan dalam context unconfined, maka SELinux tidak memberikan pembatasan yang diharapkan. Dengan context `httpd_t`, Apache hanya boleh melakukan tindakan yang diizinkan oleh policy untuk web server.

Contoh konsep label:

- `http_port_t` untuk port yang boleh dipakai Apache,
- `httpd_sys_content_t` untuk konten web yang boleh dibaca,
- `httpd_log_t` untuk log Apache,
- `httpd_sys_script_exec_t` untuk script yang boleh dieksekusi.

Dengan label yang benar, exploit aplikasi yang mencoba membaca file sistem sembarangan dapat diblokir oleh SELinux.

## 6.4 httpd_t Tidak dalam Permissive Mode

Rekomendasi 11.3 membahas permissive mode khusus untuk domain `httpd_t`.

SELinux dapat aktif secara global, tetapi domain tertentu masih bisa dibuat permissive. Jika `httpd_t` permissive, maka pelanggaran Apache hanya dicatat dan tidak diblokir. Ini berbahaya jika tertinggal di produksi.

Konsep penting:

- permissive domain berguna untuk debugging,
- produksi harus enforcing,
- jangan hanya cek status global SELinux,
- pastikan domain Apache tidak dikecualikan dari enforcement.

## 6.5 SELinux Booleans

Rekomendasi 11.4 membahas **SELinux booleans**.

Booleans adalah sakelar kebijakan SELinux yang mengizinkan atau melarang perilaku tertentu. Untuk Apache, boolean dapat mengatur apakah CGI boleh berjalan, apakah Apache boleh melakukan koneksi jaringan tertentu, apakah Apache boleh mengakses direktori user, dan sebagainya.

Konsep penting:

- hanya boolean yang diperlukan yang boleh aktif,
- setiap boolean yang aktif memperluas kemampuan Apache,
- prinsip least privilege tetap berlaku,
- kebutuhan aplikasi harus diuji dan didokumentasikan.

Contoh: jika aplikasi tidak memakai CGI, maka boolean yang mengizinkan CGI tidak perlu aktif. Jika Apache tidak perlu koneksi keluar ke database remote, boolean terkait tidak perlu diaktifkan.

## 6.6 Dampak Operasional SELinux

SELinux sangat kuat, tetapi juga kompleks. Kesalahan label atau policy dapat membuat aplikasi gagal membaca file, menulis log, mengakses socket, atau menjalankan script.

Karena itu CIS menekankan bahwa kontrol SELinux harus diuji di test server sebelum produksi. Administrator perlu memahami:

- kebutuhan aplikasi,
- label file,
- policy SELinux,
- audit log SELinux,
- proses tuning dan exception.

## 6.7 Ringkasan Bab 11

Bab 11 menekankan defense-in-depth dengan SELinux:

- aktifkan enforcing mode,
- pastikan Apache berjalan dalam context terbatas,
- jangan biarkan `httpd_t` permissive,
- aktifkan hanya boolean yang dibutuhkan,
- uji dengan matang sebelum produksi.

SELinux membatasi dampak jika Apache atau aplikasi web berhasil dieksploitasi.

---

# 7. Bab 12 — AppArmor

Bab 12 membahas **AppArmor** sebagai alternatif Mandatory Access Control untuk membatasi Apache. AppArmor umum digunakan pada Debian dan Ubuntu.

Jika SELinux menggunakan label dan type enforcement, AppArmor menggunakan profil berbasis path dan program. Artinya, profil AppArmor mendefinisikan file, direktori, capability, dan network access apa saja yang boleh dipakai oleh program tertentu.

## 7.1 Konsep AppArmor untuk Apache

AppArmor membatasi proses berdasarkan profile. Untuk Apache, profile biasanya terkait executable `apache2`. Jika profile diterapkan dengan benar, Apache hanya dapat mengakses resource yang diizinkan.

Konsep penting:

- AppArmor adalah MAC berbasis nama/path,
- profile mengikat aturan ke program,
- mode enforce benar-benar memblokir pelanggaran,
- mode complain hanya mencatat pelanggaran,
- profile harus disesuaikan dengan kebutuhan aplikasi.

AppArmor berguna untuk membatasi dampak exploit. Misalnya, jika vulnerability aplikasi membuat attacker mencoba membaca file di luar web root, AppArmor dapat memblokir akses tersebut walaupun permission Linux biasa mengizinkan.

## 7.2 AppArmor Framework Enabled

Rekomendasi 12.1 membahas framework AppArmor agar aktif.

Jika AppArmor tidak aktif, profile tidak akan membatasi proses. Karena itu, langkah konseptual pertama adalah memastikan framework berjalan.

Hal penting:

- AppArmor harus tersedia dan aktif,
- paket pendukung Apache mungkin diperlukan,
- status AppArmor perlu diverifikasi,
- default distribusi dapat berbeda.

Pada Debian/Ubuntu, AppArmor sering tersedia secara default, tetapi tetap perlu dipastikan.

## 7.3 Apache AppArmor Profile

Rekomendasi 12.2 membahas profile Apache agar dikonfigurasi dengan benar.

CIS menekankan bahwa default profile bisa terlalu permisif. Profile yang terlalu luas, misalnya memberi akses `/**`, tidak memberikan manfaat least privilege yang kuat.

Profile yang baik harus:

- memberi akses hanya ke file yang diperlukan,
- membatasi direktori konfigurasi,
- membatasi web root,
- membatasi log directory,
- membatasi runtime directory,
- memberi capability hanya jika diperlukan,
- menghindari wildcard terlalu luas,
- diuji terhadap seluruh fungsi aplikasi.

Tools seperti `aa-autodep`, `aa-complain`, dan `aa-logprof` dapat membantu membangun profile berdasarkan perilaku aplikasi, tetapi hasilnya tetap harus direview manusia.

## 7.4 AppArmor Enforce Mode

Rekomendasi 12.3 membahas mode enforce.

Profile AppArmor dapat berada dalam beberapa mode:

- **disabled**: tidak digunakan,
- **complain**: pelanggaran dicatat tetapi tidak diblokir,
- **enforce**: pelanggaran diblokir.

Mode complain berguna saat testing karena administrator dapat melihat akses apa saja yang dibutuhkan aplikasi tanpa langsung memutus fungsi. Namun untuk produksi, mode enforce diperlukan agar AppArmor benar-benar memberikan perlindungan.

Hal penting:

- setelah mengubah mode profile, service Apache sebaiknya direstart,
- proses yang sudah berjalan mungkin belum terikat profile baru,
- hasil audit harus memastikan proses Apache benar-benar confined dan enforce,
- complain mode tidak cukup untuk produksi.

## 7.5 SELinux vs AppArmor

CIS menjelaskan bahwa SELinux dan AppArmor memberikan kontrol serupa, tetapi pendekatannya berbeda.

| Aspek | SELinux | AppArmor |
|---|---|---|
| Model | Type enforcement dan label | Profile berbasis path/program |
| Umum pada | RHEL, CentOS, Alma, Rocky | Debian, Ubuntu |
| Kontrol | Domain, type, label, boolean | Profile, path rule, capability |
| Kompleksitas | Lebih kompleks, sangat granular | Lebih mudah dipahami untuk banyak admin |
| Tujuan | Membatasi proses dengan MAC | Membatasi program dengan MAC |

Tidak disarankan menjalankan keduanya bersamaan untuk tujuan yang sama karena dapat menambah kompleksitas troubleshooting.

## 7.6 Ringkasan Bab 12

Bab 12 menekankan bahwa AppArmor dapat menjadi lapisan tambahan yang kuat untuk Apache:

- framework harus aktif,
- profile Apache harus disesuaikan,
- wildcard terlalu luas harus dihindari,
- mode complain hanya untuk testing,
- produksi membutuhkan enforce mode,
- semua perubahan harus diuji agar aplikasi tidak rusak.

---

# 8. Appendix Ringkas

Appendix pada CIS Apache Benchmark berfungsi sebagai alat bantu audit dan pemetaan kontrol. Appendix bukan bagian konfigurasi teknis utama, tetapi sangat berguna untuk dokumentasi, assessment, dan pelaporan.

## 8.1 Summary Table

Summary Table berisi daftar seluruh rekomendasi CIS Apache HTTP Server 2.4 dalam format checklist. Biasanya tersedia kolom seperti:

- nomor rekomendasi,
- judul rekomendasi,
- status apakah sudah diset dengan benar,
- kolom Yes/No.

Fungsi Summary Table:

- membantu auditor melakukan checklist cepat,
- membantu administrator melihat cakupan hardening,
- membantu dokumentasi status penerapan,
- membantu gap analysis sebelum remediation.

Untuk bagian 2, item yang masuk dalam cakupan summary antara lain:

- Bab 6: logging, patching, ModSecurity, CRS,
- Bab 7: SSL/TLS, sertifikat, private key, protokol, cipher, HTTPS, HSTS, OCSP, PFS,
- Bab 8: pengurangan information leakage,
- Bab 9: mitigasi DoS,
- Bab 10: request limits,
- Bab 11: SELinux,
- Bab 12: AppArmor.

## 8.2 CIS Controls Mapping

Appendix juga memetakan rekomendasi benchmark ke CIS Controls. Biasanya pemetaan dipisahkan berdasarkan:

- CIS Controls v7 IG 1,
- CIS Controls v7 IG 2,
- CIS Controls v7 IG 3,
- CIS Controls v7 Unmapped Recommendations,
- CIS Controls v8 IG 1,
- CIS Controls v8 IG 2,
- CIS Controls v8 IG 3,
- CIS Controls v8 Unmapped Recommendations.

IG berarti **Implementation Group**. Semakin tinggi IG, semakin kompleks dan matang kontrol keamanan yang diharapkan.

Secara konseptual:

- IG 1 cocok untuk baseline dasar,
- IG 2 cocok untuk organisasi dengan kebutuhan keamanan lebih tinggi,
- IG 3 cocok untuk organisasi dengan risiko tinggi, regulasi kuat, atau aset sangat kritis.

Pemetaan ini membantu organisasi menjelaskan bahwa hardening Apache bukan aktivitas terpisah, tetapi terkait langsung dengan program keamanan yang lebih luas.

## 8.3 Change History

Change History mencatat perubahan antar versi benchmark. Ini penting karena standar keamanan berubah. Rekomendasi yang relevan hari ini dapat berubah di versi berikutnya karena:

- vulnerability baru,
- protokol lama dianggap tidak aman,
- algoritma kriptografi berubah status,
- praktik terbaik industri berubah,
- Apache atau modulnya berubah perilaku.

Karena itu benchmark harus diperlakukan sebagai dokumen hidup. Administrator perlu memastikan versi benchmark yang digunakan sesuai dengan teknologi dan tanggal assessment.

## 8.4 Cara Membaca Appendix Secara Konseptual

Appendix sebaiknya dipahami sebagai alat bantu untuk tiga kegiatan:

1. **Assessment**  
   Melihat apakah konfigurasi sudah sesuai rekomendasi.

2. **Prioritization**  
   Menentukan rekomendasi mana yang harus dikerjakan lebih dulu berdasarkan level, risiko, dan dampak operasional.

3. **Audit Evidence**  
   Menyediakan bukti bahwa organisasi memiliki proses hardening yang dapat dilacak.

Appendix bukan pengganti pemahaman rekomendasi. Checklist hanya mengatakan “sudah/belum”, sedangkan bagian utama benchmark menjelaskan alasan, dampak, dan prosedur audit/remediation.

---

# 9. Ringkasan Besar Bagian 2

Bagian 2 dari CIS Apache HTTP Server 2.4 Benchmark memperluas hardening dari konfigurasi dasar menuju operasi keamanan yang lebih matang.

Secara garis besar:

- **Bab 6** memastikan Apache menghasilkan log yang berguna, disimpan dengan baik, dipatch, dan dapat dilindungi dengan WAF.
- **Bab 7** memastikan TLS dikonfigurasi secara aman, bukan hanya aktif.
- **Bab 8** mengurangi kebocoran informasi teknis yang dapat membantu attacker.
- **Bab 9** meningkatkan ketahanan terhadap DoS berbasis koneksi lambat dan resource exhaustion.
- **Bab 10** membatasi ukuran request agar input abnormal tidak mudah membebani server atau aplikasi.
- **Bab 11** menggunakan SELinux sebagai lapisan MAC untuk membatasi proses Apache.
- **Bab 12** menggunakan AppArmor sebagai alternatif MAC, terutama relevan pada Debian/Ubuntu.
- **Appendix** membantu audit, mapping kontrol, dan dokumentasi.

Inti konseptual dari bagian ini adalah:

> Apache yang aman bukan hanya Apache yang bisa berjalan, tetapi Apache yang dapat dipantau, diperbarui, dibatasi, dienkripsi dengan benar, tidak membocorkan informasi, tahan terhadap penyalahgunaan resource, dan tetap dibatasi walaupun aplikasi web mengalami kompromi.

---

## Catatan Akhir

File ini sengaja disusun sebagai penjelasan konseptual. Perintah teknis, audit command, konfigurasi aktual, dan remediation step-by-step sebaiknya dibahas terpisah agar tidak mencampur pemahaman konsep dengan praktik implementasi.
