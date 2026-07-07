# Penjelasan Konseptual CIS Debian Linux 12 Benchmark
## C. Bab 2 — Services

**Sumber utama:** *CIS Debian Linux 12 Benchmark v2.0.0 — 05-28-2026*, khususnya Bab 2 **Services**.

**Catatan penggunaan:** dokumen ini berisi pembahasan konseptual. Fokusnya adalah memahami alasan keamanan di balik rekomendasi CIS, bukan menjalankan perintah teknis. Perintah audit dan remediation dibahas terpisah pada pembahasan praktik.

---

## Gambaran Umum Bab 2 — Services

Bab **Services** dalam CIS Debian Linux 12 Benchmark membahas layanan-layanan sistem yang berjalan di Debian 12. Dalam konteks Linux, *service* atau *daemon* adalah proses latar belakang yang menyediakan fungsi tertentu, misalnya layanan web, DNS, DHCP, file sharing, sinkronisasi waktu, printer, atau scheduler.

Secara konseptual, bab ini berdiri di atas prinsip dasar hardening: **sistem hanya boleh menjalankan layanan yang benar-benar dibutuhkan untuk fungsi bisnis atau fungsi teknis yang sah**. Semakin banyak layanan berjalan, semakin besar pula peluang sistem memiliki celah konfigurasi, celah perangkat lunak, port terbuka, proses yang bisa disalahgunakan, atau jalur masuk tambahan bagi penyerang.

CIS tidak mengatakan bahwa semua layanan dalam bab ini pasti buruk. Banyak layanan di bawah ini sah dan sangat penting pada server tertentu. Contohnya, web server wajib aktif pada server yang memang berfungsi sebagai web server; DNS server wajib aktif pada server DNS; DHCP server diperlukan pada infrastruktur tertentu; NFS atau Samba bisa diperlukan pada file server. Namun, CIS menekankan bahwa layanan-layanan tersebut **tidak boleh ada secara tidak sengaja** pada sistem yang tidak memerlukannya.

Dengan kata lain, inti Bab 2 bukan sekadar “matikan layanan”, melainkan:

1. mengenali layanan yang terpasang,
2. memeriksa apakah layanan tersebut punya kebutuhan yang sah,
3. memastikan layanan yang tidak diperlukan tidak berjalan,
4. memastikan layanan yang diperlukan disetujui, terdokumentasi, dan dikonfigurasi aman,
5. mengurangi port terbuka dan daemon yang tidak perlu.

---

# 2.1 Configure Server Services

## Konsep Service Minimization

**Service minimization** adalah prinsip mengurangi layanan aktif hanya pada layanan yang dibutuhkan. Pada server, prinsip ini sangat penting karena server biasanya terhubung ke jaringan, menerima koneksi, dan menjalankan proses jangka panjang.

Sebuah service yang berjalan dapat menambah risiko melalui beberapa cara:

- membuka port jaringan,
- menerima input dari jaringan,
- memiliki konfigurasi yang salah,
- memiliki bug keamanan,
- berjalan dengan hak akses tinggi,
- menjadi target brute force,
- menjadi media lateral movement,
- menjadi jalur eksfiltrasi data,
- menjadi proses persistence bagi attacker.

Dalam praktik hardening, pertanyaan utama terhadap setiap service bukan hanya “apakah service ini aman?”, tetapi juga “apakah service ini memang perlu ada?”. Service yang tidak dibutuhkan sebaiknya tidak terpasang, tidak aktif, atau setidaknya tidak bisa berjalan.

## Prinsip Hanya Menjalankan Layanan yang Dibutuhkan

CIS menggunakan pendekatan konservatif: jika sebuah sistem tidak secara khusus ditugaskan untuk menjalankan layanan tertentu, maka layanan tersebut sebaiknya tidak digunakan.

Contohnya:

- server aplikasi biasa tidak otomatis perlu menjadi DHCP server,
- workstation biasa tidak perlu menjalankan DNS server,
- server database tidak perlu menyediakan FTP server,
- server produksi tidak perlu menjalankan Avahi untuk discovery otomatis,
- server headless tidak perlu menjalankan X Window server.

Prinsip ini mencegah situasi ketika sebuah layanan aktif hanya karena ikut terpasang sebagai dependency, bawaan paket, sisa eksperimen, atau lupa dimatikan setelah troubleshooting.

## Risiko Daemon yang Tidak Digunakan

Daemon yang tidak digunakan tetap berisiko walaupun administrator tidak pernah memakainya secara sadar. Risiko utamanya adalah **unmanaged exposure**: ada komponen yang hidup di sistem, tetapi tidak diawasi, tidak dipatch secara serius, tidak dikonfigurasi dengan benar, dan tidak dianggap bagian dari arsitektur resmi.

Risiko daemon tidak digunakan antara lain:

- menjadi target eksploitasi dari jaringan,
- mengekspos informasi sistem,
- membuat port listening yang tidak diketahui,
- menyebabkan konflik konfigurasi,
- menghasilkan log yang tidak dipahami,
- membuka jalur akses tidak resmi,
- mengacaukan audit keamanan karena sistem terlihat menjalankan layanan yang tidak sesuai fungsi.

Dalam audit, layanan tidak dikenal sering menjadi temuan penting karena menunjukkan bahwa konfigurasi sistem tidak terkendali sepenuhnya.

## Perbedaan Service Server dan Client

**Server service** adalah layanan yang menyediakan fungsi kepada sistem lain. Biasanya service ini mendengarkan koneksi masuk, membuka port, dan menunggu permintaan dari klien. Contohnya web server, DNS server, DHCP server, FTP server, NFS server, Samba server, dan SNMP server.

**Client service/tool** adalah komponen yang digunakan untuk mengakses layanan di sistem lain. Client tidak selalu membuka port, tetapi tetap bisa berbahaya jika menggunakan protokol lama, tidak terenkripsi, atau dapat mendorong user memakai jalur komunikasi tidak aman. Contohnya telnet client, FTP client, rsh client, talk client, dan NIS client.

Perbedaannya penting: server service memperbesar risiko **incoming attack**, sedangkan client tool sering memperbesar risiko **unsafe usage** atau **credential exposure**.

## Hubungan Service dengan Port Terbuka

Banyak service server bekerja dengan cara membuka port TCP atau UDP. Port terbuka adalah titik masuk jaringan. Firewall dapat membatasi akses, tetapi firewall bukan alasan untuk membiarkan layanan tidak perlu tetap berjalan. Layanan yang berjalan tetap menambah kompleksitas dan tetap berisiko, terutama jika firewall berubah, salah konfigurasi, atau ada akses dari jaringan internal.

Karena itu, CIS memasukkan rekomendasi manual untuk memastikan hanya layanan yang disetujui yang listening pada interface jaringan. Ini penting karena tidak semua konteks bisa dinilai otomatis. Sebuah port bisa sah pada satu server, tetapi menjadi temuan serius pada server lain.

---

## 2.1.1 autofs

**autofs** adalah layanan untuk melakukan *automatic mounting*, misalnya pada CD/DVD, USB drive, atau media lain. Secara fungsi, autofs memudahkan penggunaan media removable karena sistem bisa memasang media secara otomatis tanpa tindakan manual administrator.

Dari sisi keamanan, automount dapat menjadi risiko karena media fisik yang ditancapkan ke sistem bisa langsung tersedia dalam filesystem. Pada server, ini berbahaya karena media eksternal dapat menjadi sumber malware, file tidak sah, atau jalur eksfiltrasi data. Jika seseorang memiliki akses fisik ke mesin dan automount aktif, kontrol manual terhadap media menjadi lebih lemah.

CIS mendorong agar autofs tidak digunakan kecuali memang dibutuhkan. Pada workstation, kebutuhan portable drive mungkin lebih umum. Namun pada server, penggunaan automount biasanya tidak diperlukan. Jika organisasi memang mengizinkan media removable, keputusan itu harus didukung kebijakan, kontrol fisik, dan prosedur penggunaan yang jelas.

**Inti konseptual:** autofs bukan selalu buruk, tetapi automount mengurangi kontrol manual terhadap media eksternal. Dalam hardening server, mounting media sebaiknya menjadi tindakan sadar dan terkontrol.

---

## 2.1.2 Mail Transfer Agent Local-Only

**Mail Transfer Agent (MTA)** seperti Postfix, Sendmail, atau Exim digunakan untuk mengirim dan menerima email. Pada sistem Linux, MTA kadang diperlukan untuk email lokal, misalnya notifikasi sistem atau laporan job dari cron.

Masalah keamanan muncul ketika MTA mendengarkan koneksi dari jaringan padahal sistem tersebut bukan mail server. MTA adalah perangkat lunak kompleks dan secara historis banyak menjadi target serangan. Jika server tidak ditujukan untuk menerima email dari sistem lain, maka MTA cukup dipakai untuk kebutuhan lokal saja.

Konsep **local-only** berarti MTA tidak perlu membuka layanan publik untuk menerima koneksi jaringan. Sistem tetap bisa memproses email lokal, tetapi tidak memperluas permukaan serangan sebagai mail server.

**Inti konseptual:** kebutuhan email lokal tidak sama dengan kebutuhan menjadi mail server. CIS membedakan keduanya agar sistem tetap bisa menjalankan fungsi internal tanpa membuka layanan jaringan yang tidak perlu.

---

## 2.1.3 avahi

**Avahi** adalah implementasi *zeroconf* dan multicast DNS/DNS-SD. Fungsinya memungkinkan perangkat menemukan layanan di jaringan lokal secara otomatis, misalnya printer, file sharing, atau layanan lain tanpa konfigurasi manual.

Pada jaringan desktop atau lingkungan rumah, discovery otomatis bisa memudahkan pengguna. Namun pada server, fitur ini sering tidak diperlukan. Discovery otomatis dapat mengumumkan keberadaan sistem atau layanan ke jaringan lokal, sehingga memperbesar risiko pengungkapan informasi dan menambah permukaan serangan.

Dalam lingkungan server yang dikontrol, layanan apa pun yang tersedia seharusnya diketahui melalui dokumentasi infrastruktur, bukan ditemukan secara otomatis melalui broadcast/multicast.

**Inti konseptual:** Avahi berguna untuk kemudahan discovery, tetapi pada server hardening, kemudahan ini sering tidak sebanding dengan risiko eksposur layanan.

---

## 2.1.4 Approved Listening Services

Rekomendasi ini adalah salah satu poin paling penting dalam Bab 2. CIS menekankan bahwa hanya layanan yang disetujui yang boleh mendengarkan koneksi pada interface jaringan.

**Listening service** adalah proses yang menunggu koneksi masuk pada port tertentu. Misalnya web server mendengarkan port HTTP/HTTPS, SSH mendengarkan port remote login, DNS mendengarkan port DNS, dan seterusnya.

Tidak semua listening service dapat dinilai otomatis karena konteksnya berbeda-beda. Port 80 sah pada web server, tetapi mencurigakan pada server database. Port DNS sah pada DNS server, tetapi tidak sah pada workstation. Karena itu, poin ini bersifat manual: administrator, security specialist, dan auditor harus menilai apakah setiap port dan proses memang sesuai fungsi sistem.

Penilaian konseptualnya mencakup:

- layanan apa yang listening,
- port apa yang digunakan,
- interface mana yang digunakan,
- siapa yang boleh mengakses,
- apakah layanan disetujui oleh kebijakan lokal,
- apakah layanan terdokumentasi,
- apakah layanan dilindungi firewall,
- apakah service owner jelas,
- apakah logging tersedia.

**Inti konseptual:** port terbuka harus punya alasan bisnis atau alasan teknis yang sah. Jika tidak bisa dijelaskan, service tersebut sebaiknya dianggap tidak sesuai baseline.

---

## 2.1.5 DHCP Server

**DHCP server** memberikan konfigurasi jaringan secara otomatis kepada client, seperti alamat IP, gateway, DNS, dan parameter jaringan lain. Pada Debian 12, layanan DHCP server dapat disediakan oleh paket seperti Kea.

DHCP server sangat penting pada jaringan tertentu, tetapi sangat berbahaya jika berjalan pada sistem yang tidak ditugaskan untuk itu. DHCP server yang salah konfigurasi atau tidak sah dapat menyebabkan client menerima gateway atau DNS yang salah. Dampaknya bisa berupa gangguan jaringan, serangan man-in-the-middle, pengalihan trafik, atau kegagalan akses layanan.

Dalam hardening, hanya server jaringan yang secara resmi bertugas sebagai DHCP server yang boleh menjalankan layanan ini.

**Inti konseptual:** DHCP server mengontrol identitas jaringan client. Karena itu, DHCP server tidak boleh aktif secara tidak sengaja.

---

## 2.1.6 Web Server

**Web server** menyediakan konten web atau aplikasi web. Contohnya Apache, Nginx, atau layanan HTTP lain. Pada server yang memang bertugas sebagai web server, layanan ini sah dan diperlukan. Namun pada sistem yang bukan web server, layanan web tidak seharusnya berjalan.

Web server memproses input dari jaringan dan sering terhubung dengan modul, bahasa pemrograman, framework, database, file upload, dan autentikasi. Semakin banyak komponen, semakin besar potensi salah konfigurasi atau celah keamanan.

CIS tidak melarang web server secara mutlak. Yang dilarang adalah web server yang tidak diperlukan. Jika sistem memang digunakan sebagai web server, maka seharusnya ada pengecualian terdokumentasi dan hardening khusus untuk web stack tersebut.

**Inti konseptual:** web server adalah layanan publik yang kuat, tetapi berisiko tinggi. Jalankan hanya pada sistem yang memang dirancang untuk melayani web.

---

## 2.1.7 DNS Server

**DNS server** menerjemahkan nama domain atau hostname menjadi alamat IP dan sebaliknya. Pada Debian, salah satu implementasi umum adalah BIND.

DNS server memiliki peran penting dalam infrastruktur. Namun jika sebuah sistem bukan DNS server resmi, layanan DNS tidak perlu berjalan. DNS server yang tidak sah dapat menyebabkan kebocoran informasi zona, cache poisoning, rekursi terbuka, atau menjadi bagian dari serangan amplifikasi.

Dalam organisasi, DNS harus menjadi layanan yang terkelola, terdokumentasi, dan dipantau. Menjalankan DNS server sembarangan dapat mengacaukan resolusi nama dan memperluas risiko jaringan.

**Inti konseptual:** DNS adalah layanan inti jaringan. Karena dampaknya luas, hanya server DNS resmi yang boleh menjalankannya.

---

## 2.1.8 FTP Server

**FTP server** memungkinkan transfer file melalui protokol FTP. Walaupun historically banyak digunakan, FTP memiliki masalah mendasar: protokol ini tidak melindungi kredensial dan data dengan enkripsi bawaan.

Jika user login melalui FTP biasa, username, password, dan data dapat terlihat oleh pihak yang mampu memantau jaringan. Karena itu, untuk transfer file modern, protokol yang lebih aman seperti SFTP atau mekanisme terenkripsi lain lebih disarankan.

FTP server mungkin masih digunakan untuk kebutuhan khusus, misalnya anonymous download pada lingkungan tertentu. Namun untuk baseline server umum, FTP server sebaiknya tidak berjalan kecuali benar-benar diperlukan dan dikontrol.

**Inti konseptual:** FTP server memperbesar risiko karena transfer dan autentikasinya tidak aman secara default. Jika transfer file diperlukan, gunakan mekanisme yang lebih aman.

---

## 2.1.9 dnsmasq

**dnsmasq** adalah layanan ringan yang dapat menyediakan DNS caching, DNS forwarding, dan DHCP. Karena fungsinya gabungan, dnsmasq sering dipakai pada router kecil, lab, VM, container environment, atau jaringan lokal sederhana.

Risikonya sama dengan DNS dan DHCP: jika aktif pada sistem yang tidak dimaksudkan untuk itu, dnsmasq dapat memengaruhi resolusi nama atau konfigurasi jaringan. Service kecil tetap bisa berdampak besar jika ia menjawab query DNS atau memberikan konfigurasi DHCP.

**Inti konseptual:** dnsmasq berguna pada konteks tertentu, tetapi karena menyentuh DNS dan DHCP, ia tidak boleh berjalan tanpa tujuan yang jelas.

---

## 2.1.10 LDAP Server

**LDAP server** menyediakan layanan direktori terpusat. LDAP dapat digunakan untuk menyimpan informasi user, grup, organisasi, autentikasi, atau objek direktori lainnya. Pada Linux, LDAP server umum dapat berupa slapd/OpenLDAP.

LDAP server berisi informasi sensitif dan sering menjadi bagian dari sistem identitas. Jika tidak diperlukan, menjalankan LDAP server memperluas risiko akses tidak sah terhadap informasi direktori atau autentikasi.

Jika sebuah server memang bukan identity/directory server, layanan LDAP server sebaiknya tidak ada. Jika LDAP diperlukan, maka konfigurasi keamanan, enkripsi, kontrol akses, dan logging harus dirancang secara khusus.

**Inti konseptual:** LDAP server adalah komponen identitas. Karena itu, ia harus diperlakukan sebagai layanan kritis, bukan service tambahan biasa.

---

## 2.1.11 Message Access Server

**Message access server** seperti IMAP dan POP3 digunakan agar pengguna dapat mengambil email dari mailbox. Contoh implementasi yang disebut dalam benchmark adalah Dovecot.

Layanan IMAP/POP3 hanya diperlukan jika sistem bertugas menyediakan akses mailbox. Jika bukan mail server, layanan ini tidak diperlukan. Message access server dapat menjadi target brute force password, eksploitasi parser email, atau penyalahgunaan mailbox.

Perlu dibedakan antara MTA dan message access server. MTA menangani pengiriman/penerimaan email antarserver, sedangkan IMAP/POP3 menyediakan akses mailbox bagi user. Sistem yang hanya perlu email lokal tidak otomatis memerlukan IMAP atau POP3.

**Inti konseptual:** jangan menjalankan layanan akses email user jika sistem tidak berfungsi sebagai mail server atau mailbox server.

---

## 2.1.12 Network File System Services (NFS)

**NFS** memungkinkan sistem memasang filesystem dari server lain melalui jaringan. Ini umum pada lingkungan Unix/Linux untuk file sharing antarserver.

Sebagai server service, NFS mengekspos filesystem ke jaringan. Jika tidak dikonfigurasi dengan hati-hati, NFS dapat membuka akses file yang tidak seharusnya, memperlemah isolasi, atau menjadi jalur pencurian data.

NFS bisa sah pada file server, cluster, atau lingkungan tertentu. Tetapi pada server umum yang tidak bertugas membagikan filesystem, NFS server sebaiknya tidak berjalan.

**Inti konseptual:** NFS memberikan akses file melalui jaringan. Karena menyentuh data langsung, layanan ini hanya boleh aktif pada server yang memang ditugaskan sebagai file server.

---

## 2.1.13 NIS Server

**NIS** atau Network Information Service adalah mekanisme lama untuk mendistribusikan informasi konfigurasi seperti user, group, dan host di jaringan Unix. Teknologi ini sudah banyak digantikan oleh LDAP atau sistem identitas modern.

NIS memiliki kelemahan historis, termasuk autentikasi yang lemah dan risiko pengungkapan informasi. Karena itu, menjalankan NIS server pada sistem modern umumnya tidak disarankan kecuali ada kebutuhan legacy yang sangat spesifik.

**Inti konseptual:** NIS adalah teknologi legacy untuk informasi identitas dan konfigurasi. Dalam baseline modern, NIS server sebaiknya tidak digunakan.

---

## 2.1.14 Print Server

**Print server** menyediakan layanan pencetakan ke jaringan. Pada Linux, layanan seperti CUPS dapat digunakan untuk mengelola printer dan job pencetakan.

Pada workstation atau server print khusus, layanan ini bisa sah. Namun pada server umum, print server tidak diperlukan. Print service dapat membuka port, menerima job dari jaringan, memproses file dokumen, dan memiliki risiko parser atau konfigurasi akses.

**Inti konseptual:** layanan print hanya perlu aktif pada sistem yang memang bertugas melayani pencetakan. Pada server lain, layanan ini hanya menambah permukaan serangan.

---

## 2.1.15 rpcbind

**rpcbind** digunakan untuk memetakan layanan RPC ke port jaringan. Layanan ini sering terkait dengan NFS dan layanan RPC lain.

Risiko rpcbind adalah ia membantu mengungkap layanan RPC yang tersedia dan dapat menjadi bagian dari permukaan serangan jaringan. Jika sistem tidak membutuhkan layanan RPC seperti NFS, rpcbind biasanya tidak diperlukan.

**Inti konseptual:** rpcbind bukan sekadar service kecil; ia menjadi komponen pendukung layanan RPC. Jika tidak ada kebutuhan RPC, rpcbind sebaiknya tidak berjalan.

---

## 2.1.16 rsync

**rsync** digunakan untuk sinkronisasi file. Sebagai client tool, rsync sering bermanfaat. Namun sebagai daemon/server service, rsync dapat menerima koneksi dari jaringan untuk melakukan sinkronisasi.

Jika rsync daemon berjalan tanpa kebutuhan yang jelas, ia dapat membuka akses file, menjadi jalur transfer data tidak sah, atau menjadi mekanisme eksfiltrasi. Penggunaan rsync melalui SSH berbeda konteksnya dari menjalankan rsync daemon yang listening sendiri.

**Inti konseptual:** rsync sebagai alat administrasi bisa berguna, tetapi rsync service yang mendengarkan jaringan harus dibatasi pada kebutuhan resmi.

---

## 2.1.17 Samba File Server

**Samba** menyediakan file sharing dan printer sharing yang kompatibel dengan protokol SMB/CIFS, umum dipakai untuk integrasi dengan sistem Windows.

Samba sangat berguna pada file server lintas platform, tetapi juga membuka permukaan serangan yang besar: autentikasi, permission, share configuration, guest access, dan akses file jaringan. Salah konfigurasi Samba dapat menyebabkan data internal terbuka ke user yang tidak berhak.

Jika server tidak bertugas sebagai file server SMB, layanan Samba sebaiknya tidak berjalan.

**Inti konseptual:** Samba memberi akses file lintas jaringan. Karena itu, ia harus aktif hanya pada file server yang memang dirancang, dipantau, dan dikonfigurasi untuk fungsi tersebut.

---

## 2.1.18 SNMP

**SNMP** digunakan untuk monitoring dan manajemen perangkat jaringan atau server. SNMP dapat mengambil informasi sistem seperti status interface, penggunaan resource, dan metrik lainnya.

Masalah keamanan SNMP terletak pada versi lama dan konfigurasi lemah. SNMPv1 dan SNMPv2 sering menggunakan community string yang dapat dianggap seperti password sederhana dan dalam banyak konteks tidak memberikan perlindungan kuat. Jika SNMP diperlukan, versi yang lebih aman seperti SNMPv3 dengan autentikasi dan enkripsi harus dipertimbangkan.

Jika sistem tidak dimonitor melalui SNMP, service SNMP server tidak perlu berjalan.

**Inti konseptual:** SNMP dapat membantu monitoring, tetapi juga dapat membocorkan informasi sistem. Aktifkan hanya jika dibutuhkan dan gunakan konfigurasi aman.

---

## 2.1.19 Telnet Server

**Telnet server** menerima koneksi remote login melalui protokol Telnet. Telnet adalah protokol lama dan tidak terenkripsi. Kredensial dan isi sesi dapat dikirim dalam bentuk yang dapat disadap di jaringan.

Dalam sistem modern, Telnet server hampir selalu dianggap tidak layak untuk remote administration. SSH menggantikan fungsi Telnet dengan enkripsi dan mekanisme autentikasi yang lebih kuat.

**Inti konseptual:** Telnet server membuka remote login tanpa perlindungan enkripsi yang memadai. Untuk administrasi modern, gunakan SSH, bukan Telnet.

---

## 2.1.20 TFTP Server

**TFTP** adalah protokol transfer file sederhana. Berbeda dengan FTP, TFTP lebih minimal dan sering digunakan untuk kebutuhan boot jaringan seperti PXE.

Kelemahan TFTP adalah tidak memiliki autentikasi, kontrol akses, dan enkripsi bawaan yang kuat. Karena itu, TFTP server dapat menjadi jalur akses file yang berisiko jika tidak dibatasi sangat ketat.

TFTP mungkin sah di server provisioning atau PXE boot. Namun di luar konteks itu, TFTP server tidak perlu berjalan.

**Inti konseptual:** TFTP cocok untuk kebutuhan infrastruktur tertentu, tetapi sangat berisiko jika terbuka tanpa kontrol jaringan yang ketat.

---

## 2.1.21 Web Proxy

**Web proxy** seperti Squid menjadi perantara akses web. Proxy dapat digunakan untuk caching, filtering, kontrol akses, atau audit trafik.

Proxy server yang tidak diperlukan dapat menjadi jalur penyalahgunaan jaringan. Jika terbuka atau salah konfigurasi, proxy dapat dipakai untuk menyembunyikan asal trafik, melewati kontrol, atau menjadi open proxy.

Jika sistem bukan proxy resmi, layanan web proxy sebaiknya tidak berjalan.

**Inti konseptual:** proxy mengontrol dan meneruskan trafik web. Karena dampaknya terhadap jaringan luas, hanya proxy resmi yang boleh aktif.

---

## 2.1.22 xinetd

**xinetd** adalah *super daemon* yang dapat mendengarkan permintaan untuk berbagai layanan dan menjalankan daemon yang sesuai ketika ada koneksi. Ia menggantikan konsep inetd lama.

Risikonya adalah xinetd dapat menjadi pintu untuk mengaktifkan banyak layanan lama atau kecil yang mungkin tidak terlihat jelas sebagai daemon mandiri. Jika tidak ada layanan yang membutuhkan xinetd, keberadaannya hanya menambah kompleksitas dan potensi eksposur.

**Inti konseptual:** xinetd dapat menjadi pengelola banyak layanan jaringan. Jika tidak diperlukan, lebih baik tidak digunakan agar tidak ada layanan tersembunyi yang aktif melalui super daemon.

---

## 2.1.23 X Window Server

**X Window System** menyediakan GUI pada Linux. Pada workstation, GUI bisa dibutuhkan. Namun pada server, terutama server headless, X Window biasanya tidak diperlukan.

GUI stack menambah banyak paket, library, proses, dan potensi celah. Selain itu, graphical login pada server dapat memperluas risiko akses lokal maupun remote jika tidak dikontrol.

Jika server hanya digunakan melalui CLI atau SSH, X Window server biasanya tidak perlu terpasang atau aktif. Namun jika ada kebutuhan aplikasi tertentu, misalnya aplikasi yang bergantung pada library grafis, keputusan harus disesuaikan dengan fungsi sistem.

**Inti konseptual:** server tidak otomatis membutuhkan GUI. Menghilangkan komponen grafis yang tidak perlu mengurangi kompleksitas dan attack surface.

---

# 2.2 Configure Client Services

## Risiko Client Tools Lama

Client tool tidak selalu membuka port, tetapi tetap dapat menjadi risiko. Banyak client lama menggunakan protokol tidak terenkripsi, autentikasi lemah, atau pola penggunaan yang tidak sesuai keamanan modern.

Risiko client tool lama antara lain:

- user dapat tidak sengaja memakai protokol tidak aman,
- kredensial dapat dikirim tanpa enkripsi,
- attacker dapat memancing user menghubungi server palsu,
- tool dapat digunakan untuk lateral movement,
- tool dapat menjadi bagian dari skrip lama yang tidak diaudit,
- keberadaannya menunjukkan toleransi terhadap protokol usang.

CIS mendorong agar client yang tidak diperlukan dihapus atau tidak tersedia, terutama jika protokolnya sudah digantikan oleh opsi yang lebih aman.

## Kenapa Client Tertentu Tetap Berbahaya Walaupun Bukan Server

Banyak orang mengira risiko hanya datang dari layanan server yang membuka port. Padahal client juga dapat membocorkan data. Misalnya, Telnet client dan FTP client dapat mengirim username/password melalui jaringan tanpa perlindungan memadai. Rsh/rlogin dapat mendorong penggunaan autentikasi lama. Talk menggunakan komunikasi lama yang tidak aman.

Client lama juga sering dipakai saat troubleshooting. Ini memang bisa membantu, tetapi setelah troubleshooting selesai, tool tersebut sebaiknya tidak dibiarkan tersedia jika tidak dibutuhkan. Keberadaannya dapat mendorong penggunaan yang salah di kemudian hari.

---

## 2.2.1 NIS Client

**NIS client** digunakan untuk menghubungkan sistem ke NIS server dan menerima informasi konfigurasi seperti user atau group.

NIS adalah teknologi lama yang memiliki kelemahan keamanan historis. Karena sudah banyak digantikan oleh LDAP atau sistem identitas modern, NIS client sebaiknya tidak tersedia jika tidak diperlukan.

**Inti konseptual:** jika organisasi tidak memakai NIS, client NIS tidak perlu ada. Client identitas lama dapat memperluas risiko autentikasi dan konfigurasi.

---

## 2.2.2 rsh Client

**rsh client** mencakup tool lama seperti rsh, rcp, dan rlogin. Tool ini dulu digunakan untuk remote shell, remote copy, dan login jarak jauh.

Masalah utamanya adalah desain protokol lama yang tidak memenuhi standar keamanan modern. Fungsi rsh family sudah digantikan oleh SSH dan SCP/SFTP yang menyediakan enkripsi dan autentikasi lebih kuat.

**Inti konseptual:** rsh client mendorong penggunaan remote access legacy. Dalam baseline modern, remote administration sebaiknya memakai SSH.

---

## 2.2.3 talk Client

**talk** memungkinkan pengguna mengirim dan menerima pesan melalui terminal antar sistem. Ini adalah mekanisme komunikasi lama.

Risikonya adalah protokol ini tidak dirancang untuk kebutuhan keamanan modern dan dapat menggunakan komunikasi tidak terenkripsi. Pada sistem modern, kebutuhan komunikasi user biasanya dipenuhi oleh alat lain yang lebih aman dan terkontrol.

**Inti konseptual:** talk adalah client komunikasi lama yang jarang diperlukan dan tidak sesuai baseline hardening modern.

---

## 2.2.4 Telnet Client

**Telnet client** digunakan untuk terhubung ke Telnet server. Walaupun tidak membuat sistem menjadi server, keberadaan client ini tetap berisiko karena user dapat memakainya untuk login ke sistem lain melalui protokol tidak terenkripsi.

Ketika kredensial dikirim melalui Telnet, pihak yang dapat memantau jaringan berpotensi membaca username, password, dan isi sesi. SSH adalah pengganti yang lebih aman.

**Inti konseptual:** Telnet client memungkinkan kebiasaan administrasi tidak aman. Menghapusnya membantu mencegah penggunaan protokol legacy.

---

## 2.2.5 LDAP Client

**LDAP client** digunakan agar sistem dapat melakukan lookup atau autentikasi melalui LDAP. Berbeda dengan NIS, LDAP masih banyak digunakan. Namun tidak semua sistem membutuhkan LDAP client.

Jika organisasi menggunakan LDAP untuk autentikasi atau direktori, LDAP client mungkin sah dan diperlukan. Namun jika tidak digunakan, keberadaannya tidak perlu. Poin ini membutuhkan pemahaman konteks organisasi karena menghapus LDAP client pada sistem yang bergantung pada LDAP dapat mengganggu login atau integrasi identitas.

**Inti konseptual:** LDAP client bukan selalu buruk. Ia harus ada hanya jika sistem memang menggunakan LDAP sebagai bagian dari arsitektur identitas.

---

## 2.2.6 FTP Client

**FTP client** digunakan untuk mengirim atau mengambil file dari server FTP. Sama seperti FTP server, FTP client bermasalah karena FTP tidak menyediakan perlindungan kuat terhadap data dan kredensial secara default.

Jika transfer file diperlukan, pendekatan yang lebih aman seperti SFTP, SCP, HTTPS, atau mekanisme terenkripsi lain lebih sesuai. FTP client yang dibiarkan tersedia dapat membuat user tanpa sengaja memakai jalur transfer tidak aman.

**Inti konseptual:** FTP client memperbesar kemungkinan penggunaan transfer file tidak terenkripsi. Hilangkan jika tidak ada kebutuhan yang sah.

---

# 2.3 Configure Time Synchronization

## Pentingnya Sinkronisasi Waktu

Sinkronisasi waktu adalah aspek keamanan yang sering terlihat sederhana, tetapi sangat penting. Sistem yang waktunya tidak akurat dapat menyebabkan masalah pada log, audit, autentikasi, sertifikat, token, troubleshooting, dan forensik.

Dalam keamanan sistem, waktu bukan sekadar tampilan jam. Waktu adalah dasar untuk menjawab pertanyaan:

- kapan user login,
- kapan file berubah,
- kapan service restart,
- kapan serangan terjadi,
- urutan kejadian apa yang benar,
- apakah sertifikat masih valid,
- apakah token autentikasi masih berlaku,
- apakah log dari beberapa server bisa dikorelasikan.

Jika waktu antar server berbeda jauh, analisis insiden bisa menjadi kacau. Log dari satu server mungkin terlihat terjadi sebelum atau sesudah kejadian sebenarnya. Ini menyulitkan audit dan investigasi.

## Hubungan Waktu dengan Log, Audit, Forensik, dan Sertifikat

Waktu yang konsisten membantu organisasi melakukan korelasi log. Misalnya, satu serangan dapat meninggalkan jejak di firewall, reverse proxy, web server, database, dan sistem autentikasi. Jika semua sistem memakai waktu yang sinkron, investigator dapat menyusun timeline kejadian secara akurat.

Dalam audit, timestamp yang akurat menunjukkan bahwa catatan aktivitas dapat dipercaya. Dalam forensik, timestamp membantu membedakan sebab dan akibat. Dalam sertifikat TLS, waktu sistem menentukan apakah sertifikat dianggap valid, belum berlaku, atau sudah kedaluwarsa.

Tanpa sinkronisasi waktu, sistem keamanan bisa salah menilai kejadian.

## Kenapa Hanya Satu Daemon Sinkronisasi Waktu yang Dipakai

CIS menekankan agar hanya satu daemon sinkronisasi waktu yang digunakan. Alasannya adalah menghindari konflik. Jika dua daemon mencoba mengatur jam sistem secara bersamaan, hasilnya bisa tidak konsisten, sulit diprediksi, atau membingungkan saat troubleshooting.

Contoh daemon yang umum adalah **systemd-timesyncd** dan **chrony**. Keduanya sama-sama bisa dipakai untuk sinkronisasi waktu, tetapi tidak sebaiknya berjalan bersamaan untuk fungsi yang sama.

**Inti konseptual:** pilih satu mekanisme sinkronisasi waktu yang sesuai lingkungan, konfigurasikan ke server waktu yang sah, lalu pastikan hanya mekanisme itu yang aktif.

---

## 2.3.1 Time Synchronization is in Use

Rekomendasi ini menekankan bahwa sistem harus menggunakan sinkronisasi waktu. Tujuannya bukan memilih satu tool tertentu, melainkan memastikan waktu sistem tidak berjalan sendiri tanpa referensi yang dipercaya.

Sumber waktu harus sesuai kebijakan organisasi. Pada lingkungan enterprise, biasanya digunakan NTP internal atau server waktu resmi organisasi agar semua sistem mengacu pada sumber yang sama.

**Inti konseptual:** semua sistem harus punya sumber waktu yang konsisten dan tepercaya.

---

## 2.3.1.1 Single Time Synchronization Daemon

Poin ini memastikan tidak ada lebih dari satu daemon sinkronisasi waktu yang aktif. Daemon yang saling bersaing dapat menyebabkan drift, koreksi waktu tidak stabil, dan kesulitan audit.

Jika organisasi memilih systemd-timesyncd, maka chrony tidak perlu aktif. Jika memilih chrony, maka systemd-timesyncd tidak perlu mengambil peran sinkronisasi yang sama.

**Inti konseptual:** satu sistem, satu otoritas sinkronisasi waktu.

---

## 2.3.2 systemd-timesyncd

**systemd-timesyncd** adalah daemon sinkronisasi waktu ringan yang terintegrasi dengan systemd. Untuk banyak sistem standar, ia cukup sederhana dan memadai.

Kelebihan konseptualnya:

- ringan,
- terintegrasi dengan systemd,
- cocok untuk kebutuhan sinkronisasi dasar,
- mudah dikendalikan sebagai bagian dari service systemd.

Namun dalam lingkungan yang membutuhkan kontrol waktu lebih kompleks, presisi lebih tinggi, atau skenario jaringan tertentu, organisasi bisa memilih chrony.

### 2.3.2.1 Authorized Timeserver

Sistem harus dikonfigurasi menggunakan timeserver yang sah. Ini penting karena sumber waktu yang tidak tepercaya dapat memengaruhi log, sertifikat, dan mekanisme keamanan berbasis waktu.

Menggunakan timeserver acak dari internet tanpa kebijakan dapat menimbulkan risiko operasional dan keamanan. Dalam organisasi, lebih baik menggunakan timeserver internal atau sumber eksternal yang disetujui.

### 2.3.2.2 Enabled and Running

Konfigurasi saja tidak cukup. Daemon sinkronisasi waktu harus benar-benar aktif dan berjalan agar jam sistem terus dikoreksi.

**Inti konseptual:** systemd-timesyncd harus dikonfigurasi ke sumber waktu resmi dan aktif jika dipilih sebagai mekanisme sinkronisasi.

---

## 2.3.3 chrony

**chrony** adalah daemon sinkronisasi waktu yang banyak digunakan di Linux. Chrony sering dipilih untuk lingkungan yang membutuhkan sinkronisasi lebih kuat, kemampuan menangani perubahan jaringan, atau kontrol NTP yang lebih fleksibel.

Kelebihan konseptualnya:

- cocok untuk server,
- fleksibel dalam konfigurasi sumber waktu,
- dapat bekerja baik pada kondisi jaringan yang tidak selalu stabil,
- umum digunakan dalam hardening enterprise.

### 2.3.3.1 Chrony Configured

Chrony harus diarahkan ke server waktu yang sah. Seperti pada systemd-timesyncd, sumber waktu harus disetujui oleh organisasi.

Konfigurasi waktu yang baik bukan hanya “bisa sync”, tetapi juga “sync ke sumber yang benar”.

### 2.3.3.2 Chrony Running as User `_chrony`

CIS memperhatikan prinsip **least privilege**. Chrony sebaiknya berjalan dengan user khusus, bukan dengan hak akses yang lebih luas dari yang diperlukan.

Jika daemon berjalan dengan user khusus yang terbatas, dampak kompromi dapat dikurangi. Ini adalah contoh bahwa hardening tidak hanya bicara tentang apakah service aktif, tetapi juga bagaimana service itu dijalankan.

### 2.3.3.3 Chrony Enabled and Running

Jika chrony dipilih sebagai mekanisme sinkronisasi waktu, maka service chrony harus aktif dan berjalan. Tanpa service aktif, konfigurasi tidak memberikan manfaat.

**Inti konseptual:** chrony adalah opsi sinkronisasi waktu yang kuat, tetapi harus dikonfigurasi ke sumber resmi, berjalan dengan privilege terbatas, dan aktif secara konsisten.

---

# 2.4 Job Schedulers

## Konsep cron dan at

Linux memiliki mekanisme untuk menjalankan tugas terjadwal. Dua mekanisme klasik adalah **cron** dan **at**.

**cron** digunakan untuk menjalankan tugas berulang, misalnya setiap jam, setiap hari, setiap minggu, atau pada jadwal tertentu. Cron sering dipakai untuk rotasi log, backup, update tertentu, monitoring, pembersihan temporary file, dan pekerjaan pemeliharaan sistem.

**at** digunakan untuk menjalankan tugas satu kali pada waktu tertentu. Berbeda dari cron yang berulang, at cocok untuk eksekusi terjadwal sekali jalan.

Dari sisi operasional, scheduler sangat berguna. Dari sisi keamanan, scheduler harus dikontrol karena ia dapat menjalankan perintah secara otomatis, termasuk dengan hak akses tinggi.

## Risiko Scheduled Task

Scheduled task berisiko karena perintah yang dijadwalkan dapat berjalan tanpa interaksi manusia. Jika attacker dapat menulis ke file cron atau membuat job at, attacker dapat menanam persistence, menjalankan payload berkala, mengubah konfigurasi, atau menjalankan perintah sebagai user tertentu.

Risiko scheduler antara lain:

- privilege escalation,
- persistence setelah reboot atau pada jadwal tertentu,
- eksekusi skrip berbahaya,
- penghapusan log otomatis,
- eksfiltrasi data berkala,
- bypass monitoring jika job disamarkan sebagai maintenance.

Karena itu, akses terhadap file dan direktori scheduler harus dibatasi.

## Kenapa File Cron Harus Dibatasi Aksesnya

File dan direktori cron seperti `/etc/crontab`, `/etc/cron.hourly`, `/etc/cron.daily`, `/etc/cron.weekly`, `/etc/cron.monthly`, `/etc/cron.yearly`, dan `/etc/cron.d` menentukan perintah apa yang dijalankan sistem secara otomatis.

Jika user biasa dapat menulis ke lokasi tersebut, ia dapat memasukkan perintah yang berjalan dengan hak akses lebih tinggi. Jika user biasa dapat membaca isi job tertentu, ia mungkin mendapatkan informasi tentang struktur sistem, lokasi script, credential yang salah disimpan, atau celah timing untuk menyerang.

Karena itu, CIS menekankan pembatasan owner, group, dan permission agar hanya root atau administrator yang berwenang dapat mengatur scheduled job sistem.

## Hubungan Scheduler dengan Persistence Attacker

Persistence adalah kemampuan attacker mempertahankan akses setelah berhasil masuk. Cron dan at adalah mekanisme persistence yang umum karena mudah digunakan dan berjalan otomatis.

Contoh konseptual persistence melalui scheduler:

- menjalankan reverse shell setiap beberapa menit,
- mengunduh ulang malware jika dihapus,
- membuat user baru pada waktu tertentu,
- menghapus log secara berkala,
- menjalankan script backdoor setelah reboot,
- menjalankan task pada jam sepi agar tidak terdeteksi.

Hardening scheduler bertujuan membuat mekanisme ini tidak mudah disalahgunakan.

---

## 2.4.1 Configure cron

### 2.4.1.1 Cron Daemon Enabled and Active

Menariknya, CIS tidak selalu menyuruh mematikan cron. Cron justru perlu aktif karena banyak tugas pemeliharaan sistem dan keamanan bergantung pada cron. Contohnya log rotation, maintenance job, atau monitoring tertentu.

Artinya, prinsip CIS bukan “semua daemon dimatikan”, tetapi “daemon yang diperlukan harus aktif dan dikontrol dengan benar”. Cron diperlukan, tetapi aksesnya harus aman.

**Inti konseptual:** cron adalah komponen penting untuk maintenance. Ia harus berjalan jika dibutuhkan sistem, tetapi file dan hak aksesnya harus dikendalikan ketat.

### 2.4.1.2 `/etc/crontab`

`/etc/crontab` adalah file utama yang mengatur job cron sistem. Karena file ini dapat menentukan perintah yang dijalankan otomatis, hak aksesnya harus sangat terbatas.

Jika user tidak berwenang dapat mengubah file ini, ia dapat menjalankan perintah sebagai bagian dari jadwal sistem. Jika dapat membaca file ini, ia dapat mengetahui pola maintenance dan mencari celah.

### 2.4.1.3 `/etc/cron.hourly`

Direktori ini berisi job yang berjalan setiap jam. Karena frekuensinya tinggi, penyalahgunaan direktori ini dapat berdampak cepat. File berbahaya yang diletakkan di sini dapat dieksekusi berulang kali.

### 2.4.1.4 `/etc/cron.daily`

Direktori ini berisi job harian. Banyak tugas maintenance ditempatkan di sini. Akses tulis oleh user biasa dapat menjadi jalur persistence yang relatif stabil.

### 2.4.1.5 `/etc/cron.weekly`

Direktori ini berisi job mingguan. Walaupun frekuensinya lebih rendah, tetap berisiko karena task dapat berjalan otomatis dengan hak akses sistem.

### 2.4.1.6 `/etc/cron.monthly`

Direktori ini berisi job bulanan. Attacker dapat memakai job jarang berjalan untuk persistence yang sulit terdeteksi karena aktivitasnya tidak sering muncul.

### 2.4.1.7 `/etc/cron.yearly`

Direktori ini berisi job tahunan. Walaupun jarang, prinsipnya sama: hanya administrator yang boleh mengontrolnya. Keamanan tidak hanya bergantung pada frekuensi eksekusi, tetapi juga pada hak akses yang dimiliki job tersebut.

### 2.4.1.8 `/etc/cron.d`

`/etc/cron.d` memberikan kontrol jadwal yang lebih granular. Banyak paket menempatkan file cron di sini. Karena sifatnya fleksibel, direktori ini harus dibatasi ketat agar tidak menjadi tempat attacker menyisipkan jadwal berbahaya.

### 2.4.1.9 Access to crontab

`crontab` adalah program yang memungkinkan user mengatur jadwal cron masing-masing. Dalam banyak sistem server, tidak semua user perlu diberi kemampuan membuat cron job.

Menggunakan konsep allow list lebih aman daripada deny list. Dengan allow list, hanya user yang secara eksplisit diizinkan yang dapat menggunakan crontab. Dengan deny list, user baru bisa saja lupa dimasukkan ke daftar larangan.

**Inti konseptual:** kontrol akses crontab mencegah user biasa membuat persistence atau scheduled command tanpa izin.

---

## 2.4.2 Configure at

### 2.4.2.1 Access to at

`at` memungkinkan user menjadwalkan perintah satu kali pada waktu tertentu. Walaupun tidak berulang seperti cron, at tetap bisa disalahgunakan untuk menjalankan perintah berbahaya di masa depan.

Contoh risikonya:

- attacker menjadwalkan payload untuk berjalan setelah admin selesai memeriksa sistem,
- user biasa menjadwalkan perintah yang mengganggu sistem,
- job dijalankan pada waktu sepi agar tidak terlihat,
- task satu kali dipakai untuk menghapus jejak.

Karena itu, akses ke at perlu dibatasi. Seperti crontab, pendekatan allow list lebih mudah diaudit karena hanya user yang disetujui yang boleh menjadwalkan job.

**Inti konseptual:** at adalah scheduler satu kali, tetapi tetap bisa menjadi alat persistence atau delayed execution. Aksesnya harus dikontrol.

---

# Ringkasan Konseptual Bab 2

Bab 2 CIS Debian Linux 12 Benchmark menekankan bahwa keamanan service bukan hanya soal patch atau firewall, tetapi juga soal **keputusan arsitektural: layanan apa yang pantas hidup di sistem ini?**

Prinsip besarnya adalah:

1. **Minimalkan layanan.** Jangan menjalankan daemon yang tidak diperlukan.
2. **Validasi port terbuka.** Semua listening service harus punya alasan dan persetujuan.
3. **Bedakan server dan client.** Client lama juga dapat membahayakan keamanan walaupun tidak membuka port.
4. **Gunakan waktu yang sinkron.** Log, audit, forensik, sertifikat, dan autentikasi bergantung pada waktu yang benar.
5. **Kontrol scheduler.** Cron dan at penting untuk operasi, tetapi juga sering disalahgunakan untuk persistence.
6. **Dokumentasikan pengecualian.** Jika sebuah layanan memang dibutuhkan, ia harus disetujui, dikonfigurasi aman, dipantau, dan dicatat sebagai bagian dari fungsi sistem.

Dengan memahami Bab 2, administrator tidak hanya tahu “service apa yang harus dimatikan”, tetapi juga memahami cara berpikir CIS: **setiap service harus punya tujuan, setiap port harus punya pemilik, setiap pengecualian harus punya alasan, dan setiap mekanisme otomatis harus dikontrol.**

---

# Peta Belajar Singkat

| Area | Pertanyaan Konseptual | Tujuan Hardening |
|---|---|---|
| Server services | Apakah sistem ini memang harus menyediakan layanan tersebut ke jaringan? | Mengurangi port terbuka dan daemon tidak perlu |
| Client services | Apakah user masih perlu tool legacy ini? | Mencegah penggunaan protokol tidak aman |
| Time synchronization | Apakah waktu sistem sinkron ke sumber resmi? | Menjaga integritas log, audit, dan forensik |
| Job schedulers | Siapa yang boleh membuat scheduled task? | Mencegah privilege escalation dan persistence |

