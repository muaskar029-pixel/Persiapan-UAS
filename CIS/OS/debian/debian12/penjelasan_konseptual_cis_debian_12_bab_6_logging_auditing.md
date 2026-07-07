# Penjelasan Konseptual CIS Debian Linux 12 Benchmark
# G. Bab 6 — Logging and Auditing

**Sumber utama:** CIS Debian Linux 12 Benchmark v2.0.0, Bab 6 — *Logging and Auditing*  
**Fokus pembahasan:** konseptual, bukan panduan praktik command-by-command.  
**Tujuan:** memahami mengapa logging, auditing, perlindungan log, audit rule, dan integrity checking menjadi bagian penting dalam hardening Debian Linux 12.

---

## Gambaran Umum Bab 6 — Logging and Auditing

Bab **Logging and Auditing** dalam CIS Debian Linux 12 Benchmark membahas kemampuan sistem untuk mencatat aktivitas, menyimpan bukti kejadian, dan menyediakan data yang dapat digunakan untuk pemantauan keamanan, audit kepatuhan, troubleshooting, investigasi insiden, dan forensik digital.

Pada sistem Linux, keamanan tidak cukup hanya dengan mencegah serangan. Sistem juga harus mampu menjawab pertanyaan penting setelah sebuah kejadian terjadi, misalnya:

1. Siapa yang login ke sistem?
2. Kapan login terjadi?
3. Apakah ada percobaan login gagal?
4. Apakah ada perubahan pada file penting seperti `/etc/passwd`, `/etc/shadow`, atau konfigurasi `sudo`?
5. Apakah ada user menjalankan perintah dengan privilege tinggi?
6. Apakah ada perubahan waktu sistem, hostname, konfigurasi jaringan, atau aturan audit?
7. Apakah log masih utuh, atau sudah dimanipulasi?
8. Apakah file sistem penting berubah tanpa sepengetahuan administrator?

Dalam konteks hardening, logging dan auditing adalah lapisan **visibility** dan **accountability**. Tanpa logging yang baik, administrator mungkin tidak tahu bahwa serangan sedang terjadi. Tanpa auditing yang baik, organisasi mungkin tidak bisa membuktikan apa yang berubah, siapa yang melakukannya, dan kapan perubahan terjadi.

Bab ini dapat dipahami dalam tiga kelompok besar:

- **System Logging**, yaitu pencatatan aktivitas umum sistem melalui `journald`, `rsyslog`, dan file log.
- **System Auditing**, yaitu pencatatan aktivitas sensitif dan perubahan penting menggunakan `auditd`.
- **Integrity Checking**, yaitu pemeriksaan integritas file untuk mendeteksi perubahan yang tidak sah.

Perbedaan paling sederhana antara logging biasa dan auditing adalah: logging biasa sering berisi informasi operasional sistem, sedangkan auditing berfokus pada aktivitas yang memiliki nilai keamanan dan kepatuhan. Logging menjawab “apa yang terjadi di sistem”, sedangkan auditing menjawab “aktivitas keamanan apa yang perlu dibuktikan dan dilacak secara formal”.

---

# 6.1 System Logging

## Konsep System Logging

**System logging** adalah proses mencatat kejadian yang terjadi pada sistem operasi, service, aplikasi, kernel, proses login, autentikasi, error, warning, dan aktivitas administratif tertentu. Di Debian modern, dua komponen penting yang sering terlibat adalah **systemd-journald** dan **rsyslog**.

Logging memiliki beberapa fungsi utama:

- membantu troubleshooting ketika service gagal berjalan;
- membantu deteksi aktivitas mencurigakan;
- menyediakan data untuk monitoring dan SIEM;
- mendukung investigasi insiden;
- membantu audit kepatuhan;
- menjadi bukti kronologis saat terjadi perubahan sistem.

Dalam CIS, logging tidak hanya dilihat dari apakah log tersedia, tetapi juga dari bagaimana log disimpan, siapa yang dapat membacanya, apakah log diputar secara teratur, apakah log dikompresi, apakah log dapat diteruskan ke server pusat, dan apakah akses terhadap log dibatasi.

---

# 6.1.1 Configure journald

## Fungsi systemd-journald

`systemd-journald` adalah komponen logging dari systemd. Ia menerima dan menyimpan log dari berbagai sumber, termasuk kernel, service systemd, proses user, standard output dan standard error service, serta pesan syslog tertentu. Pada sistem Debian yang menggunakan systemd, journald menjadi salah satu sumber utama catatan aktivitas sistem.

Secara konseptual, journald berfungsi sebagai pusat awal pengumpulan log lokal. Ia menangkap kejadian dengan struktur metadata yang lebih kaya dibandingkan file log teks tradisional. Setiap entry log dapat memiliki informasi seperti waktu, unit systemd, PID, UID, GID, executable, hostname, dan informasi konteks lain.

Keuntungan journald:

- terintegrasi langsung dengan systemd;
- mampu menyimpan metadata log secara terstruktur;
- dapat mengambil log berdasarkan unit service, waktu, proses, atau prioritas;
- dapat meneruskan log ke mekanisme logging lain seperti rsyslog;
- dapat menyimpan log secara volatile atau persistent.

Dalam konteks CIS, journald perlu dipastikan aktif dan terkonfigurasi dengan aman agar sistem tidak kehilangan informasi operasional dan keamanan yang penting.

## Persistensi Log

Salah satu isu penting dalam journald adalah **persistensi log**. Log dapat disimpan secara sementara di memori atau secara permanen di disk.

Jika log hanya disimpan secara volatile, log dapat hilang setelah reboot. Ini berbahaya untuk investigasi, karena penyerang atau kegagalan sistem dapat menyebabkan bukti kejadian hilang ketika sistem restart.

Persistensi log penting karena:

- kejadian sebelum reboot tetap dapat dianalisis;
- crash atau restart tidak langsung menghapus bukti;
- administrator dapat melihat riwayat aktivitas lintas boot;
- proses audit lebih kuat karena data tidak hanya bergantung pada sesi aktif.

Namun, persistensi juga harus diimbangi dengan kontrol storage dan rotasi. Log yang disimpan terus-menerus tanpa pengelolaan dapat memenuhi disk. Karena itu, CIS tidak hanya membahas penyimpanan log, tetapi juga rotasi dan pengelolaan ukuran log.

## Forwarding ke rsyslog

CIS juga membahas konfigurasi agar journald dapat mengirim log ke `rsyslog`. Secara konseptual, forwarding dari journald ke rsyslog berguna karena rsyslog memiliki kemampuan kuat dalam pengelolaan file log tradisional, filtering, format log, forwarding ke remote log server, dan integrasi dengan infrastruktur logging organisasi.

Alur sederhananya dapat dipahami seperti ini:

```text
Aplikasi / Kernel / Service
        ↓
systemd-journald
        ↓
rsyslog
        ↓
File log lokal atau remote log server
```

Forwarding ke rsyslog membantu organisasi yang masih mengandalkan struktur log tradisional di `/var/log`, sistem SIEM, atau server log terpusat. Dengan demikian, journald tidak berdiri sendiri, tetapi menjadi bagian dari rantai logging yang lebih luas.

## Permission Log journald

Log dapat berisi informasi sensitif. Contohnya adalah username, alamat IP, nama service, error aplikasi, path file, pesan autentikasi, informasi konfigurasi, bahkan indikasi kegagalan keamanan. Karena itu, file log journald tidak boleh dapat dibaca atau diubah oleh sembarang user.

Permission log yang buruk dapat menyebabkan beberapa risiko:

- user biasa dapat membaca informasi sensitif;
- penyerang lokal dapat mempelajari struktur sistem;
- log dapat dimanipulasi untuk menutupi jejak;
- bukti forensik menjadi tidak dapat dipercaya;
- informasi internal dapat bocor.

Secara konseptual, log harus mengikuti prinsip **least privilege**: hanya user atau grup yang memang membutuhkan akses log yang boleh membacanya, dan hanya proses/administrator tertentu yang boleh menulis atau mengelolanya.

## Log Rotation

Log rotation adalah mekanisme untuk mengelola pertumbuhan ukuran log. Tanpa rotasi, file log dapat terus membesar sampai memenuhi disk. Pada server produksi, kondisi disk penuh dapat menyebabkan service gagal berjalan, database bermasalah, autentikasi terganggu, atau sistem menjadi tidak stabil.

Tujuan log rotation:

- mencegah log memenuhi storage;
- menyimpan riwayat log dalam periode tertentu;
- memudahkan pengarsipan;
- memisahkan log lama dan log baru;
- mendukung kebijakan retensi organisasi.

Dalam konteks keamanan, log rotation juga membantu menjaga log tetap dapat dikelola. Log yang terlalu besar sulit dianalisis. Log yang diputar secara teratur lebih mudah dikirim, dikompresi, dicadangkan, dan dianalisis oleh sistem monitoring.

## Compression

Compression atau kompresi log bertujuan mengurangi penggunaan disk untuk log lama. Log biasanya berbasis teks atau memiliki pola berulang, sehingga sering dapat dikompresi dengan efisien.

Manfaat kompresi:

- menghemat ruang penyimpanan;
- memungkinkan retensi log lebih lama;
- mengurangi beban storage;
- memudahkan arsip log historis.

Namun, kompresi bukan pengganti perlindungan log. Log yang dikompresi tetap harus memiliki permission yang tepat dan tetap harus dijaga integritasnya. Kompresi hanya membantu efisiensi penyimpanan, bukan menjamin keamanan isi log.

---

# 6.1.2 Configure rsyslog

## Fungsi rsyslog

`rsyslog` adalah sistem logging yang umum digunakan pada distribusi Linux untuk menerima, memproses, menyaring, menyimpan, dan meneruskan log. Jika journald sangat terintegrasi dengan systemd, rsyslog sering dipakai sebagai mekanisme logging tradisional yang fleksibel dan kuat.

Fungsi utama rsyslog:

- menyimpan log ke file di `/var/log`;
- memisahkan log berdasarkan fasilitas dan prioritas;
- meneruskan log ke server log pusat;
- mendukung format dan filtering yang kompleks;
- mendukung transport aman seperti TLS;
- berperan sebagai penghubung antara host lokal dan sistem monitoring/SIEM.

Dalam desain keamanan organisasi, rsyslog sering menjadi komponen penting untuk **centralized logging**. Jika log hanya disimpan lokal di mesin yang diserang, penyerang yang memperoleh akses root mungkin dapat menghapus atau mengubah log. Dengan remote logging, salinan log dapat tetap tersedia di server lain.

## Mode File Log

Mode file log menentukan permission default ketika rsyslog membuat file log baru. Ini penting karena log yang baru dibuat harus langsung memiliki permission aman. Jika file log dibuat dengan permission terlalu longgar, user biasa dapat membaca informasi sensitif.

Risiko mode file log terlalu permisif:

- informasi autentikasi dapat terbaca;
- alamat IP internal dan username dapat bocor;
- error aplikasi dapat mengungkap konfigurasi;
- penyerang lokal dapat memetakan aktivitas sistem;
- kerahasiaan data operasional terganggu.

Secara konseptual, file log harus dibuat dengan permission yang cukup ketat sejak awal, bukan diperbaiki setelah telanjur terbuka. Ini sejalan dengan prinsip **secure by default**.

## Remote Logging

Remote logging adalah pengiriman log dari satu sistem ke server log pusat. Dalam infrastruktur yang lebih serius, remote logging sangat penting karena log lokal dapat hilang jika sistem rusak, disk gagal, atau penyerang menghapus jejak.

Manfaat remote logging:

- log tetap tersedia meskipun host sumber dikompromikan;
- analisis keamanan dapat dilakukan secara terpusat;
- korelasi antar sistem menjadi lebih mudah;
- mendukung SIEM dan monitoring organisasi;
- memperkuat forensik dan audit.

Contoh skenario penting: jika seorang penyerang berhasil masuk ke server dan mencoba menghapus `/var/log/auth.log`, log yang sudah dikirim ke server pusat masih dapat dipakai untuk investigasi. Dengan demikian, remote logging meningkatkan ketahanan bukti.

## TLS untuk Forwarding Log

Ketika log dikirim melalui jaringan, log dapat melewati media yang tidak sepenuhnya dipercaya. Tanpa enkripsi, log dapat disadap. Tanpa autentikasi server yang benar, log dapat dikirim ke tujuan yang salah atau diserang melalui man-in-the-middle.

TLS dalam forwarding log membantu menyediakan:

- **confidentiality**, agar isi log tidak mudah dibaca pihak lain;
- **integrity**, agar perubahan log saat transit dapat dideteksi;
- **authentication**, agar pengirim dan penerima dapat memverifikasi identitas.

Log sering berisi informasi sensitif, sehingga pengiriman log ke remote server sebaiknya tidak dianggap sebagai lalu lintas biasa. Dalam lingkungan produksi, forwarding log idealnya dilindungi oleh kanal terenkripsi dan dikonfigurasi sesuai kebijakan sertifikat organisasi.

## CA Certificate

CA certificate digunakan untuk memverifikasi sertifikat pada koneksi TLS. Dalam konteks rsyslog, CA certificate membantu memastikan bahwa server log yang dituju memang server yang dipercaya, bukan server palsu.

Jika validasi CA tidak benar, beberapa risiko muncul:

- log dapat dikirim ke server tidak sah;
- penyerang dapat melakukan man-in-the-middle;
- administrator merasa log aman padahal jalurnya tidak tervalidasi;
- bukti audit dapat kehilangan kepercayaan.

Secara konseptual, TLS tanpa validasi identitas yang benar tidak cukup. Enkripsi harus disertai kepercayaan terhadap identitas endpoint.

## Risiko Menerima Log dari Client yang Tidak Seharusnya

CIS juga membahas agar rsyslog tidak dikonfigurasi menerima log dari remote client jika sistem tersebut tidak memang berperan sebagai log server. Ini penting karena menerima log dari jaringan berarti membuka fungsi server tambahan.

Risiko jika host biasa menerima log dari client lain:

- ada port tambahan yang terbuka;
- attack surface jaringan meningkat;
- sistem dapat menerima input log palsu;
- log lokal dapat dipenuhi spam atau data tidak relevan;
- penyerang dapat mencoba mengeksploitasi service logging;
- beban storage dan CPU meningkat tanpa kebutuhan bisnis.

Prinsipnya sama seperti service minimization: jika sebuah host bukan log server, ia tidak perlu menerima log dari jaringan. Ia boleh mengirim log ke pusat, tetapi tidak harus menjadi penerima log untuk host lain.

## Logrotate dalam Konteks rsyslog

`logrotate` adalah komponen yang membantu memutar, mengarsipkan, mengompresi, dan menghapus log sesuai kebijakan. Dalam konteks rsyslog, logrotate penting karena rsyslog dapat menghasilkan banyak file log tradisional di `/var/log`.

Tanpa logrotate:

- file log dapat tumbuh tanpa batas;
- disk dapat penuh;
- service dapat terganggu;
- investigasi menjadi sulit karena file terlalu besar;
- retensi log menjadi tidak konsisten.

Logrotate bukan hanya fitur kenyamanan, tetapi bagian dari manajemen risiko operasional. Sistem yang aman harus tetap stabil dan dapat diaudit dalam jangka panjang.

---

# 6.1.3 Configure Logfiles

## Permission Logfile

File log adalah catatan aktivitas sistem. Karena isinya dapat sensitif dan bernilai sebagai bukti, permission log harus dikonfigurasi dengan benar. File log tidak boleh dapat ditulis oleh user biasa, dan akses baca juga harus dibatasi sesuai kebutuhan.

Konsep utama permission logfile:

- file log harus dimiliki oleh user dan grup yang tepat;
- user biasa tidak boleh dapat mengubah atau menghapus log;
- akses baca harus dibatasi pada pihak yang membutuhkan;
- direktori log juga harus memiliki permission yang aman;
- file log lama hasil rotasi tetap harus dilindungi.

Permission log tidak boleh hanya diperiksa pada satu file. Direktori `/var/log` dapat berisi banyak file dari berbagai service. CIS menekankan perlunya memastikan akses ke seluruh logfile telah dikonfigurasi dengan benar.

## Kenapa Log Harus Dilindungi dari User Biasa

Log dapat mengandung informasi yang tidak layak dibaca user biasa, seperti:

- nama user lain;
- IP address sumber koneksi;
- pesan autentikasi;
- path file sensitif;
- pesan error aplikasi;
- konfigurasi internal;
- indikasi kegagalan keamanan;
- nama service dan proses yang berjalan.

Jika user biasa dapat membaca log secara luas, ia dapat mempelajari sistem dan menyiapkan serangan lebih baik. Jika user biasa dapat menulis ke log, ia dapat membuat kebingungan, menyisipkan pesan palsu, atau mencoba menutupi aktivitasnya.

Karena itu, log harus dipandang sebagai aset keamanan, bukan sekadar file teks biasa.

## Risiko Manipulasi Log

Manipulasi log adalah salah satu teknik yang sering dikaitkan dengan penyembunyian jejak. Setelah penyerang memperoleh akses, ia mungkin mencoba menghapus baris log, mengubah timestamp, menghapus file log, atau membanjiri log dengan pesan palsu.

Dampak manipulasi log:

- investigasi menjadi sulit;
- akar masalah tidak jelas;
- bukti audit melemah;
- organisasi tidak dapat memastikan skala insiden;
- penyerang dapat bertahan lebih lama tanpa terdeteksi.

Perlindungan log harus dipadukan dengan remote logging, permission yang ketat, audit terhadap perubahan file penting, dan integrity checking. Satu kontrol saja tidak cukup untuk menjamin log benar-benar aman.

---

# 6.2 System Auditing

## Konsep System Auditing

System auditing adalah proses mencatat aktivitas yang memiliki nilai keamanan tinggi secara lebih formal dan terstruktur. Di Linux, komponen utama untuk auditing adalah `auditd`, yaitu daemon audit yang bekerja bersama kernel audit subsystem.

Auditing berbeda dari logging umum. Logging umum dapat mencatat banyak informasi operasional, sedangkan auditing berfokus pada kejadian yang perlu dipertanggungjawabkan, misalnya:

- perubahan file identitas user;
- penggunaan privilege tinggi;
- perubahan konfigurasi keamanan;
- login dan logout;
- percobaan akses file yang gagal;
- perubahan permission;
- perubahan modul kernel;
- perubahan konfigurasi audit itu sendiri.

Auditing diperlukan karena dalam keamanan, tidak cukup hanya mengetahui bahwa “service error”. Organisasi juga perlu tahu apakah ada perubahan administratif, siapa yang melakukan perubahan, dan apakah tindakan tersebut sah.

---

# 6.2.1 Configure auditd Service

## Apa itu auditd

`auditd` adalah daemon yang bertugas menerima event audit dari kernel audit subsystem dan menyimpannya ke audit log. Auditd memungkinkan administrator menentukan rule tentang kejadian apa yang harus dicatat.

Secara konseptual, auditd menjadi kamera pengawas untuk aktivitas sensitif di sistem. Ia tidak menggantikan firewall, AppArmor, atau permission file. Auditd terutama berfungsi menyediakan bukti dan visibilitas.

Fungsi auditd:

- mencatat event keamanan penting;
- merekam penggunaan privilege;
- memantau perubahan file sensitif;
- mendukung forensik digital;
- mendukung kepatuhan audit;
- membantu mendeteksi aktivitas mencurigakan.

## Perbedaan Logging Biasa dan Auditing

Perbedaan logging biasa dan auditing dapat dipahami seperti berikut:

| Aspek | Logging Biasa | Auditing |
|---|---|---|
| Fokus | Aktivitas umum sistem dan aplikasi | Aktivitas keamanan dan kepatuhan |
| Contoh | Service restart, error aplikasi, pesan kernel | Perubahan `/etc/passwd`, penggunaan sudo, login/logout |
| Sumber | Aplikasi, service, journald, rsyslog | Kernel audit subsystem dan auditd |
| Tujuan | Troubleshooting dan monitoring | Accountability, compliance, forensic evidence |
| Karakter | Lebih umum dan operasional | Lebih spesifik dan sensitif |

Keduanya saling melengkapi. Logging memberi gambaran luas tentang sistem, sedangkan auditing memberi catatan lebih tajam terhadap tindakan yang berdampak pada keamanan.

## Audit Proses Sebelum auditd Aktif

Salah satu isu penting adalah proses yang berjalan pada fase boot awal, sebelum daemon auditd aktif sepenuhnya. Jika aktivitas pada fase awal tidak tercatat, ada celah visibilitas pada saat sistem mulai berjalan.

Konsep “auditing for processes that start prior to auditd” bertujuan memastikan proses yang muncul sebelum auditd aktif tetap berada dalam cakupan audit. Ini penting karena beberapa perubahan sistem atau eksekusi proses dapat terjadi sangat awal saat boot.

Risiko jika fase awal boot tidak diaudit:

- aktivitas awal sistem tidak memiliki jejak audit;
- malware atau konfigurasi berbahaya yang aktif saat boot lebih sulit dilacak;
- investigasi tidak memiliki kronologi lengkap;
- accountability berkurang.

Secara konseptual, audit harus dimulai sedini mungkin agar tidak ada periode buta saat sistem boot.

## audit_backlog_limit

`audit_backlog_limit` berkaitan dengan kapasitas antrian event audit ketika sistem menghasilkan banyak event. Jika event audit datang lebih cepat daripada kemampuan auditd memprosesnya, event dapat masuk backlog.

Jika backlog terlalu kecil, event audit dapat hilang atau sistem dapat mengalami masalah saat beban tinggi. Dalam konteks keamanan, kehilangan event audit berarti kehilangan bukti.

Konsep pentingnya:

- sistem harus mampu menampung lonjakan event audit;
- audit tidak boleh mudah kehilangan data saat aktivitas tinggi;
- backlog membantu menjaga reliability audit;
- nilai backlog harus disesuaikan dengan profil sistem dan beban kerja.

Audit backlog bukan hanya soal performa, tetapi juga soal keutuhan catatan keamanan.

---

# 6.2.2 Configure Data Retention

## Ukuran Audit Log

Audit log dapat tumbuh besar, terutama pada sistem aktif atau sistem dengan rule audit yang detail. Karena itu, ukuran audit log harus dikonfigurasi agar cukup menampung data penting tetapi tidak menghabiskan storage secara tidak terkendali.

Jika ukuran terlalu kecil:

- log cepat penuh;
- log lama cepat tergeser;
- investigasi kehilangan konteks historis;
- organisasi tidak memenuhi kebutuhan retensi.

Jika ukuran tidak dikelola:

- disk dapat penuh;
- service terganggu;
- sistem dapat menjadi tidak stabil;
- audit justru menjadi risiko operasional.

Konfigurasi ukuran audit log harus mengikuti kebutuhan keamanan, regulasi, kapasitas storage, dan kebijakan retensi organisasi.

## Kebijakan Audit Log Tidak Otomatis Dihapus

CIS menekankan pentingnya memastikan audit log tidak otomatis dihapus begitu saja. Secara konseptual, audit log adalah bukti. Jika sistem menghapus audit log secara otomatis tanpa kebijakan yang jelas, maka bukti insiden dapat hilang sebelum sempat dianalisis.

Audit log yang tidak boleh sembarangan dihapus penting untuk:

- menjaga bukti forensik;
- mendukung investigasi insiden;
- memenuhi kebutuhan audit;
- mempertahankan accountability;
- mencegah penyerang memanfaatkan rotasi/hapus otomatis untuk menghilangkan jejak.

Retensi audit harus direncanakan. Bukan berarti log tidak pernah boleh dihapus, tetapi penghapusan harus mengikuti kebijakan, jadwal, arsip, dan persetujuan yang jelas.

## Sistem Saat Audit Log Penuh

Salah satu pertanyaan penting dalam auditing adalah: apa yang harus dilakukan sistem jika audit log penuh?

Dalam sistem yang sangat sensitif, kehilangan audit event bisa dianggap sangat serius. Karena itu, CIS membahas bagaimana sistem harus merespons ketika audit log penuh. Secara konseptual, ada trade-off antara availability dan accountability.

Jika sistem tetap berjalan saat audit log penuh:

- layanan tetap tersedia;
- tetapi aktivitas sensitif mungkin tidak tercatat;
- penyerang dapat mencoba memenuhi audit log untuk menciptakan blind spot.

Jika sistem dibatasi atau dihentikan saat audit log penuh:

- availability dapat terganggu;
- tetapi sistem tidak terus berjalan tanpa kemampuan audit;
- accountability lebih dijaga.

Organisasi harus memahami dampaknya. Pada server kritis, kebijakan ini harus diuji dan disesuaikan dengan kebutuhan bisnis. Namun secara prinsip, audit log penuh tidak boleh diabaikan.

## Warning Saat Storage Audit Rendah

Sebelum audit log benar-benar penuh, sistem sebaiknya memberi peringatan ketika ruang penyimpanan audit mulai rendah. Warning ini memberi kesempatan administrator untuk bertindak sebelum terjadi kehilangan log atau gangguan sistem.

Manfaat warning storage rendah:

- memberi deteksi dini;
- mencegah kehilangan audit event;
- mencegah disk penuh mendadak;
- membantu operasi proaktif;
- mendukung pemenuhan retensi audit.

Warning harus dipandang sebagai kontrol operasional keamanan. Tanpa peringatan, masalah audit storage mungkin baru diketahui setelah bukti hilang atau sistem bermasalah.

---

# 6.2.3 Configure auditd Rules

## Konsep auditd Rules

Auditd rules menentukan kejadian apa yang harus dicatat. Tanpa rule yang tepat, auditd mungkin aktif tetapi tidak mencatat kejadian penting. Karena itu, konfigurasi audit rule adalah inti dari system auditing.

Audit rule harus seimbang. Jika terlalu sedikit, banyak aktivitas penting tidak terlihat. Jika terlalu banyak dan tidak terarah, sistem dapat menghasilkan log berlebihan, membebani storage, dan membuat analisis sulit.

CIS mengarahkan audit terhadap area yang berhubungan dengan:

- privilege escalation;
- perubahan identitas user dan grup;
- perubahan konfigurasi sistem;
- perubahan waktu dan jaringan;
- aktivitas login;
- akses file gagal;
- perubahan permission;
- penggunaan perintah privileged;
- perubahan audit itu sendiri;
- perubahan kernel module;
- perubahan mekanisme mandatory access control.

Berikut penjelasan konseptual tiap kelompok audit rule.

---

## Audit sudoers

File dan direktori `sudoers` menentukan siapa yang dapat menjalankan perintah dengan privilege lebih tinggi. Perubahan pada konfigurasi sudo dapat memberi user biasa kemampuan administratif.

Audit perubahan sudoers penting karena:

- sudo adalah jalur utama eskalasi privilege;
- perubahan kecil dapat memberi akses root;
- konfigurasi sudo yang salah dapat membuka celah besar;
- setiap perubahan harus dapat dipertanggungjawabkan.

Jika ada user menambahkan dirinya ke aturan sudo tanpa otorisasi, audit rule harus membantu mendeteksi kapan perubahan itu terjadi.

## Audit Tindakan sebagai User Lain

Aktivitas sebagai user lain, terutama melalui mekanisme seperti sudo, sangat penting untuk dicatat. Ketika user menjalankan perintah sebagai root atau akun lain, tanggung jawab tindakan harus tetap dapat dilacak ke user asal.

Tanpa audit tindakan sebagai user lain:

- sulit mengetahui siapa yang sebenarnya melakukan perubahan;
- aktivitas root dapat terlihat anonim;
- investigasi kehilangan konteks;
- accountability melemah.

Audit ini membantu menjawab: user mana yang meminta privilege, kapan, dan aktivitas apa yang dilakukan.

## Audit Sudo Log

Jika sistem memiliki file log khusus untuk sudo, perubahan terhadap file tersebut juga perlu diaudit. Sudo log berisi jejak aktivitas privilege escalation. Jika log ini diubah atau dihapus, jejak administratif dapat hilang.

Audit terhadap sudo log penting untuk:

- mendeteksi penghapusan jejak sudo;
- menjaga integritas bukti privilege escalation;
- membantu investigasi penyalahgunaan hak akses;
- memastikan log privilege tidak dimanipulasi.

## Audit Perubahan Waktu

Waktu sistem adalah fondasi kronologi log. Jika waktu sistem diubah, urutan kejadian dapat menjadi membingungkan. Penyerang dapat mencoba mengubah waktu untuk menyamarkan aktivitas atau mengacaukan korelasi log.

Perubahan waktu perlu diaudit karena:

- timestamp log bergantung pada waktu sistem;
- sertifikat dan autentikasi tertentu bergantung pada waktu;
- forensik membutuhkan kronologi akurat;
- perubahan waktu dapat menjadi indikasi upaya manipulasi.

Audit perubahan waktu membantu memastikan setiap perubahan jam, tanggal, atau konfigurasi waktu dapat dilacak.

## Audit Hostname dan Domain

Hostname dan domain name membantu identifikasi sistem. Dalam lingkungan banyak server, perubahan hostname dapat menyebabkan kebingungan operasional dan investigasi.

Risiko perubahan hostname/domain tanpa audit:

- sistem sulit diidentifikasi di log pusat;
- korelasi SIEM terganggu;
- penyerang dapat menyamarkan host;
- dokumentasi aset tidak sesuai kondisi aktual.

Audit perubahan hostname/domain membantu menjaga integritas identitas sistem.

## Audit Issue Banner

File seperti `/etc/issue` dan `/etc/issue.net` digunakan untuk banner login lokal atau remote tertentu. Perubahan pada file ini perlu diaudit karena banner berhubungan dengan legal notice, kebijakan akses, dan informasi yang ditampilkan sebelum login.

Risiko jika banner diubah sembarangan:

- legal notice hilang;
- sistem menampilkan informasi sensitif;
- pesan login dapat dimanipulasi;
- kebijakan organisasi tidak lagi tercermin.

Audit perubahan banner membantu memastikan pesan login tetap sesuai kebijakan.

## Audit hosts dan hostname

File seperti `/etc/hosts` dan `/etc/hostname` berpengaruh terhadap identitas dan resolusi nama lokal. Manipulasi file ini dapat mengarahkan koneksi ke alamat yang salah atau menyebabkan identifikasi sistem berubah.

Audit perubahan file tersebut penting karena:

- resolusi nama lokal dapat dimanipulasi;
- koneksi internal dapat diarahkan ke host lain;
- hostname sistem dapat berubah tanpa otorisasi;
- troubleshooting dan forensik dapat terganggu.

## Audit Network Config

Konfigurasi jaringan menentukan bagaimana sistem berkomunikasi. Perubahan pada konfigurasi jaringan dapat membuka akses baru, mengubah gateway, mengganti DNS, mengaktifkan interface, atau mengarahkan trafik ke jalur yang tidak aman.

Audit konfigurasi jaringan penting untuk mendeteksi:

- perubahan IP address;
- perubahan route;
- perubahan DNS;
- perubahan interface;
- konfigurasi jaringan yang melemahkan keamanan;
- manipulasi konektivitas oleh penyerang.

Network configuration adalah bagian dari security boundary. Jika boundary berubah, harus ada jejak audit.

## Audit netplan

Walaupun Debian tidak selalu memakai netplan sebagai default seperti beberapa distribusi lain, CIS memasukkan audit terhadap konfigurasi netplan karena pada beberapa sistem atau lingkungan tertentu netplan dapat digunakan untuk mengelola jaringan.

Secara konseptual, jika netplan digunakan, file konfigurasinya harus diaudit karena dapat mengubah alamat IP, gateway, DNS, dan interface. Jika netplan tidak digunakan, bagian ini tetap menunjukkan prinsip umum: semua mekanisme konfigurasi jaringan yang relevan harus dipantau.

## Audit Privileged Commands

Perintah privileged adalah perintah yang dapat dijalankan dengan hak lebih tinggi, misalnya melalui bit SUID/SGID atau mekanisme privilege tertentu. Perintah seperti ini sensitif karena dapat menjadi jalur eskalasi privilege.

Audit penggunaan privileged commands penting karena:

- banyak teknik serangan lokal memanfaatkan binary privileged;
- aktivitas administratif penting dapat dilacak;
- penggunaan tidak wajar dapat menjadi indikator kompromi;
- membantu mendeteksi penyalahgunaan SUID/SGID.

Jika perintah privileged dijalankan oleh user yang tidak biasa atau pada waktu mencurigakan, audit log dapat menjadi petunjuk awal.

## Audit group, passwd, shadow, dan gshadow

File `/etc/group`, `/etc/passwd`, `/etc/shadow`, dan `/etc/gshadow` adalah inti identitas lokal Linux. Perubahan pada file ini dapat menambah user, mengubah password hash, mengubah grup, atau memberi akses tambahan.

Risiko jika perubahan file identitas tidak diaudit:

- akun backdoor dapat dibuat tanpa jejak;
- user dapat diberi akses grup sensitif;
- password dapat diubah diam-diam;
- akun lama dapat diaktifkan kembali;
- akses root tidak sah dapat disiapkan.

Audit file identitas lokal sangat penting karena identity management adalah salah satu target utama penyerang.

## Audit PAM

PAM atau Pluggable Authentication Modules mengendalikan banyak proses autentikasi di Linux. Perubahan konfigurasi PAM dapat melemahkan password policy, mengubah lockout, mengaktifkan metode autentikasi tertentu, atau membuat bypass login.

Audit perubahan PAM penting karena:

- PAM mempengaruhi login, sudo, su, dan autentikasi service lain;
- kesalahan konfigurasi PAM dapat mengunci user sah atau membuka akses tidak sah;
- penyerang dapat mencoba melemahkan autentikasi melalui PAM;
- perubahan PAM harus selalu dapat ditelusuri.

## Audit Unsuccessful File Access

Percobaan akses file yang gagal dapat menjadi sinyal penting. User yang mencoba membaca file tanpa izin mungkin sedang melakukan eksplorasi, enumerasi, atau upaya privilege escalation.

Audit akses file gagal membantu mendeteksi:

- user mencoba membaca file sensitif;
- proses mencurigakan mencoba mengakses path terlarang;
- serangan brute force terhadap permission;
- aktivitas reconnaissance lokal.

Tidak semua akses gagal adalah serangan. Namun pola akses gagal yang berulang terhadap file sensitif layak diperhatikan.

## Audit Permission Changes

Perubahan permission, ownership, ACL, atau atribut file dapat mengubah model keamanan sistem. Jika permission file penting dilonggarkan, user yang sebelumnya tidak punya akses bisa memperoleh akses.

Audit perubahan permission penting untuk:

- mendeteksi file sensitif yang dibuat terlalu terbuka;
- mendeteksi perubahan ownership mencurigakan;
- mengetahui siapa yang mengubah hak akses;
- menjaga integritas discretionary access control.

Permission adalah salah satu kontrol dasar Linux. Perubahannya harus dapat diaudit.

## Audit Mount

Mount filesystem dapat memperkenalkan media baru, partisi baru, network share, atau filesystem dengan opsi tertentu. Penyerang dapat menggunakan mount untuk membawa file berbahaya, mengakses data, atau menjalankan teknik tertentu.

Audit mount penting karena:

- media eksternal atau remote share dapat masuk ke sistem;
- mount option dapat mempengaruhi keamanan;
- filesystem baru dapat membawa SUID file atau executable;
- aktivitas mount sering relevan dalam investigasi.

## Audit Session

Session initiation information membantu mencatat pembukaan sesi user. Ini penting untuk mengetahui kapan user mulai berinteraksi dengan sistem.

Audit session membantu menjawab:

- kapan sesi user dimulai;
- user mana yang membuka sesi;
- dari mana sesi mungkin berasal;
- apakah ada sesi tidak biasa.

Informasi session melengkapi log login/logout dan membantu membentuk kronologi aktivitas user.

## Audit Login dan Logout

Login dan logout adalah event fundamental dalam keamanan. Tanpa catatan login/logout, sulit mengetahui siapa yang berada di sistem pada waktu tertentu.

Audit login/logout penting untuk:

- mendeteksi login tidak sah;
- menganalisis brute force atau percobaan login;
- menghubungkan aktivitas dengan sesi user;
- mendukung forensik;
- membuktikan akses pengguna.

Login sukses dan gagal sama-sama penting. Login gagal dapat menunjukkan serangan, sedangkan login sukses menunjukkan akses aktual.

## Audit File Deletion

Penghapusan file oleh user perlu diaudit karena dapat berkaitan dengan penghapusan bukti, sabotase, kesalahan operasional, atau aktivitas malware.

Audit file deletion membantu mendeteksi:

- penghapusan log;
- penghapusan file konfigurasi;
- penghapusan data penting;
- aktivitas user yang tidak sesuai;
- upaya menutupi jejak.

Penghapusan file tidak selalu buruk, tetapi penghapusan file tertentu pada waktu tertentu dapat sangat penting dalam investigasi.

## Audit MAC Changes

MAC dalam konteks ini adalah **Mandatory Access Control**, seperti AppArmor. Perubahan terhadap konfigurasi MAC dapat mengubah batasan keamanan aplikasi.

Audit perubahan MAC penting karena:

- profile keamanan aplikasi dapat dilemahkan;
- mode enforcing dapat diubah menjadi complain atau disabled;
- aplikasi yang sebelumnya dibatasi dapat menjadi lebih bebas;
- penyerang dapat mencoba menonaktifkan lapisan pertahanan tambahan.

Jika MAC adalah pengaman tambahan, perubahan terhadapnya harus selalu terlihat.

## Audit chcon, setfacl, chacl, dan usermod

Beberapa perintah berpengaruh langsung terhadap label keamanan, ACL, atau identitas user:

- `chcon` berkaitan dengan perubahan konteks keamanan pada sistem yang mendukung label keamanan tertentu;
- `setfacl` mengubah Access Control List;
- `chacl` juga berkaitan dengan ACL;
- `usermod` mengubah properti akun user.

Perintah-perintah ini perlu diaudit karena dapat mengubah siapa yang boleh mengakses apa, atau mengubah kemampuan akun user.

Risiko jika tidak diaudit:

- akses file dapat dilonggarkan tanpa jejak;
- akun user dapat diberi grup tambahan;
- kebijakan akses dapat dimodifikasi diam-diam;
- privilege dapat berubah tanpa bukti.

## Audit Kernel Module Changes

Kernel module memperluas kemampuan kernel. Jika penyerang dapat memuat atau mengubah modul kernel, dampaknya bisa sangat serius. Kernel module dapat digunakan untuk driver, filesystem, network protocol, atau bahkan rootkit.

Audit perubahan kernel module penting karena:

- modul kernel berjalan dengan privilege sangat tinggi;
- rootkit dapat bersembunyi pada level kernel;
- modul baru dapat memperluas attack surface;
- unloading module dapat menonaktifkan fitur keamanan;
- perubahan kernel module merupakan event keamanan tinggi.

Aktivitas load, unload, atau modifikasi module harus dapat dilacak.

## Immutable Audit Config

Konsep immutable audit config berarti konfigurasi audit dibuat sulit diubah saat sistem berjalan. Tujuannya agar penyerang yang memperoleh privilege tidak mudah mematikan atau mengubah audit rule untuk menyembunyikan aktivitas.

Manfaat konfigurasi audit immutable:

- memperkuat integritas audit;
- mencegah perubahan rule audit secara diam-diam;
- menjaga kontinuitas pencatatan event;
- meningkatkan kepercayaan terhadap audit log.

Namun, konfigurasi immutable juga memiliki dampak operasional. Perubahan audit rule mungkin memerlukan reboot atau prosedur administratif tertentu. Karena itu, konfigurasi audit harus direncanakan dengan baik sebelum dikunci.

## Sinkronisasi Konfigurasi Audit Running dan On-Disk

Audit rule dapat ada dalam kondisi running dan dalam file konfigurasi on-disk. Jika keduanya tidak sama, maka sistem mungkin berjalan dengan rule yang berbeda dari yang didokumentasikan.

Risiko ketidaksamaan running dan on-disk configuration:

- administrator mengira rule tertentu aktif padahal tidak;
- perubahan sementara hilang setelah reboot;
- audit tidak konsisten;
- compliance check dapat membingungkan;
- investigasi tidak dapat memastikan cakupan audit.

Secara konseptual, konfigurasi audit yang aktif harus selaras dengan konfigurasi permanen. Sistem yang aman tidak boleh bergantung pada konfigurasi sementara yang tidak terdokumentasi.

---

# 6.2.4 Configure auditd File Access

## Konsep Perlindungan File auditd

Auditd menghasilkan dan menggunakan file yang sangat sensitif: audit log, konfigurasi audit, rule audit, dan tool audit. Jika file-file ini dapat diubah oleh pihak tidak berwenang, maka sistem audit tidak dapat dipercaya.

Perlindungan file auditd bertujuan menjaga:

- kerahasiaan audit log;
- integritas audit log;
- integritas konfigurasi audit;
- integritas tool audit;
- kepercayaan terhadap hasil investigasi.

Audit log bukan hanya data teknis, tetapi bukti. Karena itu, aksesnya harus lebih ketat dibanding file operasional biasa.

## Permission Direktori Audit Log

Direktori audit log biasanya menyimpan file audit utama. Jika permission direktori terlalu longgar, user tidak berwenang dapat melihat, menghapus, mengganti, atau membuat file di lokasi audit.

Risiko direktori audit log tidak aman:

- audit log dapat dihapus;
- file palsu dapat dibuat;
- log dapat diganti;
- user biasa dapat membaca informasi sensitif;
- bukti forensik kehilangan nilai.

Direktori audit harus hanya dapat diakses oleh pihak yang benar-benar berwenang.

## Permission File Audit Log

File audit log berisi catatan aktivitas sensitif. Permission file audit log harus mencegah pembacaan dan perubahan oleh user biasa.

File audit log yang terlalu terbuka dapat menyebabkan:

- bocornya aktivitas user;
- bocornya struktur sistem;
- informasi serangan terlihat oleh penyerang;
- manipulasi atau penghapusan bukti.

Dalam hardening, file audit log harus diperlakukan sebagai aset kritis.

## Owner dan Group Audit Log

Owner dan group menentukan siapa yang memiliki dan dapat mengelola file audit. Jika owner atau group salah, user atau service yang tidak semestinya dapat memperoleh akses.

Konsepnya:

- owner file audit harus entitas yang berwenang;
- group harus dibatasi pada grup yang memang membutuhkan akses;
- perubahan owner/group harus dicegah dari pihak tidak sah;
- kepemilikan file harus konsisten dengan kebijakan sistem.

Kepemilikan yang benar memperkuat permission. Permission yang ketat tetapi owner salah tetap dapat menjadi risiko.

## Permission Konfigurasi Audit

File konfigurasi audit menentukan rule, lokasi log, perilaku auditd, dan kebijakan retention. Jika file konfigurasi ini dapat diubah oleh pihak tidak sah, penyerang dapat mengurangi cakupan audit atau mengubah perilaku audit.

Risiko konfigurasi audit tidak dilindungi:

- rule audit dapat dihapus;
- audit log dapat diarahkan ke lokasi lain;
- retensi dapat dilemahkan;
- audit dapat dibuat tidak mencatat aktivitas tertentu;
- sistem terlihat patuh padahal tidak mencatat event penting.

Karena itu, file konfigurasi audit harus memiliki permission, owner, dan group yang tepat.

## Permission Audit Tools

Audit tools adalah program yang digunakan untuk mengelola, membaca, atau memproses audit. Jika tool audit dimodifikasi, hasil audit dapat dimanipulasi atau administrator dapat diberi output palsu.

Risiko jika audit tools tidak dilindungi:

- tool dapat diganti dengan versi berbahaya;
- hasil audit dapat dipalsukan;
- administrator mendapat informasi salah;
- penyerang dapat menyembunyikan aktivitas;
- integritas forensik terganggu.

Perlindungan audit tools adalah bagian dari menjaga kepercayaan terhadap seluruh sistem audit. Bukan hanya log yang harus aman, alat untuk membaca dan mengelola log juga harus aman.

---

# 6.3 Configure Integrity Checking

## Konsep Integrity Checking

Integrity checking adalah proses memeriksa apakah file berubah dari kondisi yang dikenal baik. Dalam konteks CIS, integrity checking penting karena banyak serangan bekerja dengan mengubah file sistem, mengganti binary, menyisipkan script, mengubah konfigurasi, atau menanam backdoor.

Tanpa integrity checking, administrator mungkin tidak tahu bahwa file penting telah berubah. Logging dan auditing mencatat event, tetapi integrity checking membantu membandingkan keadaan file dengan baseline.

Pertanyaan utama integrity checking:

- Apakah file sistem penting berubah?
- Apakah permission berubah?
- Apakah ownership berubah?
- Apakah hash file berubah?
- Apakah binary sistem masih asli?
- Apakah konfigurasi penting dimodifikasi?

Integrity checking adalah lapisan deteksi. Ia tidak selalu mencegah perubahan, tetapi membantu menemukan perubahan yang tidak sah.

---

## AIDE

AIDE atau **Advanced Intrusion Detection Environment** adalah tool file integrity checking. AIDE membuat database baseline dari file yang dipantau, lalu membandingkan kondisi saat ini dengan baseline tersebut.

AIDE dapat memeriksa atribut seperti:

- checksum atau hash file;
- permission;
- ownership;
- ukuran file;
- timestamp tertentu;
- atribut file lain sesuai konfigurasi.

Konsep kerja AIDE:

```text
Baseline awal file sistem
        ↓
Database integritas
        ↓
Pemeriksaan berkala
        ↓
Perbandingan kondisi saat ini dengan baseline
        ↓
Laporan perubahan
```

AIDE berguna untuk mendeteksi perubahan yang tidak terlihat dari penggunaan normal, misalnya binary sistem yang diganti, file konfigurasi yang berubah, atau permission file yang dilonggarkan.

Namun, AIDE harus dipahami dengan benar. Jika baseline dibuat setelah sistem sudah terkompromi, maka baseline tersebut tidak dapat dipercaya. Baseline sebaiknya dibuat saat sistem berada dalam kondisi bersih dan terverifikasi.

## Filesystem Integrity Checking

Filesystem integrity checking harus dilakukan secara berkala. Jika AIDE hanya diinstal tetapi tidak pernah dijalankan, maka manfaatnya sangat terbatas. Pemeriksaan berkala memungkinkan administrator mendeteksi perubahan dari waktu ke waktu.

Manfaat pemeriksaan integritas berkala:

- mendeteksi perubahan file penting;
- mendeteksi modifikasi binary;
- mendukung investigasi insiden;
- membantu compliance;
- memberikan peringatan dini terhadap perubahan tidak sah.

Pemeriksaan integritas juga harus menghasilkan output yang ditinjau. Tool yang berjalan otomatis tetapi hasilnya tidak pernah dibaca tetap kurang efektif. Dalam program keamanan yang matang, hasil integrity check harus masuk ke proses monitoring atau review rutin.

## Perlindungan Integritas Audit Tools

CIS juga menekankan perlindungan integritas terhadap audit tools. Ini sangat penting karena audit tools adalah alat yang dipakai untuk melihat bukti. Jika tool audit dimodifikasi, administrator dapat menerima hasil palsu.

Contoh risiko:

- binary audit tool diganti agar menyembunyikan event tertentu;
- tool pembaca log dimodifikasi agar tidak menampilkan aktivitas penyerang;
- hash tool berubah tanpa diketahui;
- hasil investigasi menjadi tidak valid.

Karena itu, integrity checking tidak hanya berlaku untuk file konfigurasi umum, tetapi juga untuk tool audit. Secara konseptual, ini adalah prinsip **trust the tools, but verify the tools**. Alat keamanan juga harus menjadi objek pengamanan.

---

# Ringkasan Konseptual Bab 6

Bab 6 CIS Debian Linux 12 Benchmark menekankan bahwa sistem yang aman harus dapat diamati, diaudit, dan diverifikasi integritasnya. Tanpa logging, administrator kehilangan visibilitas. Tanpa auditing, organisasi kehilangan accountability. Tanpa integrity checking, perubahan file penting dapat terjadi tanpa terdeteksi.

Inti konseptual Bab 6 adalah:

1. **journald** menyediakan logging terintegrasi dengan systemd dan perlu dikonfigurasi agar log aktif, persisten, terkelola, dan dapat diteruskan.
2. **rsyslog** menyediakan logging tradisional dan remote logging yang penting untuk monitoring pusat dan SIEM.
3. **permission logfile** harus ketat karena log berisi informasi sensitif dan bukti keamanan.
4. **auditd** menyediakan pencatatan event keamanan yang lebih formal daripada logging biasa.
5. **audit data retention** memastikan audit log cukup lama tersedia dan tidak hilang diam-diam.
6. **auditd rules** menentukan aktivitas penting apa saja yang harus dicatat, seperti sudo, login, perubahan user, perubahan network, perubahan permission, mount, dan kernel module.
7. **auditd file access** melindungi log, konfigurasi, dan tool audit dari manipulasi.
8. **AIDE dan integrity checking** membantu mendeteksi perubahan tidak sah pada file sistem.

Dalam hardening, logging dan auditing bukan fitur tambahan. Keduanya adalah fondasi untuk mengetahui apakah kontrol keamanan benar-benar bekerja dan apakah terjadi aktivitas mencurigakan. Sistem tanpa log dan audit yang baik mungkin terlihat berjalan normal, tetapi tidak memiliki kemampuan memadai untuk membuktikan apa yang sebenarnya terjadi.

---

# Catatan untuk Pembahasan Lanjutan

Bagian ini masih bersifat konseptual. Implementasi teknis seperti konfigurasi `journald`, `rsyslog`, `auditd`, audit rule, `logrotate`, TLS forwarding, dan AIDE sebaiknya dibahas terpisah pada sesi praktik agar setiap command, opsi, risiko, dan dampaknya dapat dijelaskan dengan hati-hati.
