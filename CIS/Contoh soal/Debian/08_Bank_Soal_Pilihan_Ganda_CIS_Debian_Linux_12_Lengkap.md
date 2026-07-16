# Bank Soal Pilihan Ganda CIS Debian Linux 12 Benchmark

**Basis materi:** file konseptual CIS Debian Linux 12 Benchmark yang sudah dibuat sebelumnya, mencakup Bagian Pengantar, Bab 1 sampai Bab 7, serta bonus implementasi Arch Linux berbasis prinsip CIS Debian.

**Format:** pilihan ganda A–D, dilengkapi kunci jawaban dan pembahasan singkat.

**Jumlah soal:** 272 soal.


---

## Petunjuk Penggunaan

- Kerjakan terlebih dahulu tanpa melihat bagian **Kunci** dan **Pembahasan**.

- Setelah menjawab, cocokkan dengan kunci yang tersedia di bawah tiap soal.

- Fokus utama latihan ini adalah memahami **konsep**, bukan menghafal command.

- Beberapa soal sengaja dibuat mirip untuk melatih pembedaan konsep yang sering tertukar, misalnya logging vs auditing, server service vs client tool, dan default deny vs default allow.


---


# A. Pengantar CIS Benchmark


## Soal 1

Apa makna utama CIS Benchmark dalam konteks hardening sistem operasi?

A. Software antivirus untuk mendeteksi malware pada endpoint
B. Panduan konfigurasi teknis untuk membentuk baseline keamanan sistem
C. Aplikasi monitoring jaringan real-time
D. Sistem operasi khusus untuk server keamanan

**Kunci:** B

**Pembahasan:** CIS Benchmark adalah panduan konfigurasi teknis. Ia membantu menentukan baseline aman, bukan menggantikan antivirus, EDR, patching, logging, atau monitoring.


## Soal 2

Mengapa CIS Benchmark disebut baseline, bukan satu-satunya standar keamanan?

A. Karena hanya berlaku untuk sistem yang tidak terhubung internet
B. Karena ia adalah titik awal konfigurasi aman yang tetap perlu disesuaikan dengan risiko dan kebutuhan organisasi
C. Karena semua rekomendasinya bersifat opsional tanpa nilai audit
D. Karena hanya membahas teori tanpa konfigurasi teknis

**Kunci:** B

**Pembahasan:** Baseline berarti fondasi awal. CIS perlu digabung dengan risk management, kebijakan organisasi, monitoring, patching, dan standar lain.


## Soal 3

Perbedaan paling tepat antara CIS Benchmark dan EDR adalah...

A. CIS berfokus pada konfigurasi baseline, sedangkan EDR berfokus pada deteksi dan respons aktivitas endpoint
B. CIS menggantikan EDR pada semua server Linux
C. EDR hanya digunakan untuk konfigurasi filesystem
D. CIS hanya berfungsi ketika terjadi insiden

**Kunci:** A

**Pembahasan:** CIS membantu mengeraskan konfigurasi awal dan berkelanjutan. EDR membantu mendeteksi, menganalisis, dan merespons aktivitas mencurigakan.


## Soal 4

Dalam penggunaan CIS, apa arti assessment?

A. Menghapus semua service dari sistem
B. Menilai apakah konfigurasi sistem sudah sesuai rekomendasi
C. Mengganti distribusi Linux dengan Debian
D. Membuat password root baru

**Kunci:** B

**Pembahasan:** Assessment adalah proses memeriksa status konfigurasi sistem terhadap rekomendasi benchmark.


## Soal 5

Apa arti remediation dalam CIS Benchmark?

A. Proses memperbaiki konfigurasi agar sesuai rekomendasi
B. Proses membuat akun user baru
C. Proses menghapus log audit
D. Proses menonaktifkan semua koneksi jaringan

**Kunci:** A

**Pembahasan:** Remediation adalah langkah perbaikan setelah assessment menemukan kondisi yang belum sesuai.


## Soal 6

Mengapa rekomendasi berstatus Manual tidak boleh dianggap tidak penting?

A. Karena Manual berarti otomatis diterapkan oleh CIS-CAT
B. Karena Manual biasanya membutuhkan penilaian konteks manusia yang tidak bisa sepenuhnya diputuskan tool
C. Karena Manual hanya berlaku untuk workstation
D. Karena Manual tidak masuk audit

**Kunci:** B

**Pembahasan:** Manual berarti perlu validasi manusia. Banyak keputusan, misalnya apakah service tertentu sah, bergantung pada fungsi sistem.


## Soal 7

Apa ciri rekomendasi Automated dalam CIS?

A. Tidak dapat diperiksa menggunakan tool apa pun
B. Dapat dinilai secara teknis menjadi pass/fail dengan automation
C. Hanya berlaku untuk user root
D. Selalu lebih penting daripada Manual

**Kunci:** B

**Pembahasan:** Automated berarti kondisi teknisnya dapat diperiksa otomatis, misalnya permission file, status service, atau nilai parameter.


## Soal 8

Mengapa CIS-CAT atau tooling assessment berguna?

A. Agar semua rekomendasi Manual diabaikan
B. Agar assessment banyak sistem lebih skalabel dan konsisten
C. Agar server tidak perlu dipatch
D. Agar firewall otomatis dimatikan

**Kunci:** B

**Pembahasan:** Tooling membantu memeriksa banyak sistem secara konsisten. Namun rekomendasi Manual tetap harus ditangani.


## Soal 9

Siapa stakeholder yang paling tepat dilibatkan saat hardening CIS diterapkan?

A. Hanya administrator sistem
B. Admin, security specialist, auditor, help desk, pemilik layanan, dan manajemen risiko
C. Hanya user biasa
D. Hanya vendor hardware

**Kunci:** B

**Pembahasan:** Hardening dapat berdampak pada operasional. Karena itu perlu koordinasi lintas fungsi.


## Soal 10

Mengapa hardening tidak boleh dilakukan sendirian tanpa koordinasi?

A. Karena semua hardening pasti merusak sistem
B. Karena perubahan keamanan bisa memengaruhi aplikasi, akses user, jaringan, dan operasional
C. Karena CIS hanya boleh dibaca auditor
D. Karena semua konfigurasi harus disetujui antivirus

**Kunci:** B

**Pembahasan:** Hardening dapat menonaktifkan service, mengubah PAM, firewall, SSH, atau permission. Koordinasi mencegah downtime dan lockout.


## Soal 11

Apa risiko menggunakan benchmark yang salah, misalnya Debian 12 memakai benchmark RHEL?

A. Tidak ada risiko karena semua Linux identik
B. Audit bisa salah, remediation gagal, dan konfigurasi bisa tidak relevan atau merusak sistem
C. Sistem otomatis lebih aman
D. Semua path konfigurasi akan tetap sama

**Kunci:** B

**Pembahasan:** Benchmark dibuat untuk produk dan versi tertentu. Debian, Ubuntu, Arch, dan RHEL memiliki paket, service, path, dan default berbeda.


## Soal 12

Mengapa Debian 12 tidak boleh disamakan mentah-mentah dengan Arch Linux?

A. Karena Arch bukan Linux
B. Karena Arch rolling release dan ekosistem paket/konfigurasinya berbeda dari Debian
C. Karena Debian tidak menggunakan kernel
D. Karena semua command Debian ada di Arch

**Kunci:** B

**Pembahasan:** Konsep hardening dapat diadaptasi, tetapi implementasi teknis seperti package manager, PAM, dan service bisa berbeda.


## Soal 13

Apa itu exception dalam CIS?

A. Kondisi ketika rekomendasi tidak diterapkan atau diterapkan berbeda karena alasan yang sah dan terdokumentasi
B. Kesalahan audit yang harus dihapus
C. Fitur untuk mematikan semua kontrol keamanan
D. Rekomendasi yang hanya berlaku untuk cloud

**Kunci:** A

**Pembahasan:** Exception diperlukan ketika ada kebutuhan teknis/bisnis yang sah. Exception harus didokumentasikan beserta risiko dan mitigasi.


## Soal 14

Contoh exception yang benar adalah...

A. Web server tetap aktif pada server yang memang berfungsi sebagai web server dan risikonya didokumentasikan
B. Menonaktifkan auditd tanpa alasan
C. Membuka SSH ke semua jaringan karena mudah
D. Menghapus firewall karena mengganggu

**Kunci:** A

**Pembahasan:** CIS adalah baseline. Jika fungsi utama sistem membutuhkan service tertentu, buat exception dengan mitigasi dan persetujuan.


## Soal 15

Apa isi dokumentasi exception yang baik?

A. Hanya nama admin
B. Rekomendasi, alasan, risiko, mitigasi, pemilik risiko, dan waktu review ulang
C. Hanya tanggal instalasi
D. Hanya output uname -a

**Kunci:** B

**Pembahasan:** Exception harus bisa diaudit dan menjelaskan alasan serta kontrol pengganti.


## Soal 16

Mengapa remediation harus diuji di lab sebelum produksi?

A. Agar skor audit dibuat rendah
B. Agar dampak terhadap service, login, firewall, aplikasi, dan akses admin diketahui sebelum mengganggu produksi
C. Karena production tidak boleh pernah diubah
D. Karena lab lebih penting dari produksi

**Kunci:** B

**Pembahasan:** CIS menekankan testing. Hardening bisa menyebabkan downtime atau lockout bila langsung diterapkan.


## Soal 17

Apa perbedaan lab testing dan pilot deployment?

A. Lab testing dilakukan pada sistem uji; pilot deployment dilakukan pada sebagian kecil sistem produksi
B. Keduanya selalu sama
C. Lab testing hanya untuk antivirus
D. Pilot deployment berarti seluruh produksi langsung diubah

**Kunci:** A

**Pembahasan:** Tahap bertahap membantu menemukan masalah sebelum deployment luas.


## Soal 18

Apa fungsi Impact Statement dalam rekomendasi CIS?

A. Menjelaskan konsekuensi keamanan, fungsi, atau operasional jika rekomendasi diterapkan
B. Menampilkan versi kernel
C. Menyimpan password admin
D. Menentukan vendor hardware

**Kunci:** A

**Pembahasan:** Impact membantu menentukan apakah rekomendasi cocok diterapkan atau perlu exception/mitigasi.


## Soal 19

Apa fungsi Audit Procedure?

A. Memberi langkah menilai apakah sistem compliant
B. Memberi langkah mengganti hostname
C. Menghapus rekomendasi Manual
D. Menjalankan EDR

**Kunci:** A

**Pembahasan:** Audit procedure menjelaskan cara memeriksa kondisi sistem terhadap rekomendasi.


## Soal 20

Apa fungsi Remediation Procedure?

A. Memberi langkah untuk memperbaiki sistem agar sesuai rekomendasi
B. Membuat laporan keuangan
C. Menguji bandwidth
D. Menghapus semua user

**Kunci:** A

**Pembahasan:** Remediation procedure adalah instruksi untuk membawa sistem ke kondisi yang diharapkan.


## Soal 21

Apa maksud Profile dalam CIS?

A. Kumpulan rekomendasi untuk konteks keamanan tertentu, misalnya Level 1 atau Level 2
B. Nama user Linux
C. Profil wallpaper GNOME
D. Daftar file log

**Kunci:** A

**Pembahasan:** Profile mengelompokkan rekomendasi berdasarkan tingkat keamanan dan konteks server/workstation.


## Soal 22

Perbedaan Level 1 dan Level 2 yang paling tepat adalah...

A. Level 1 praktis dan tidak terlalu mengganggu; Level 2 lebih ketat dan dapat berdampak pada fungsi/performa
B. Level 1 hanya untuk Windows; Level 2 hanya untuk Linux
C. Level 1 tidak aman; Level 2 selalu wajib
D. Level 2 tidak perlu diuji

**Kunci:** A

**Pembahasan:** Level 2 lebih defense-in-depth dan cocok untuk lingkungan dengan keamanan tinggi, tetapi perlu testing lebih hati-hati.


## Soal 23

Perbedaan Server dan Workstation dalam profile CIS adalah...

A. Server biasanya menyediakan layanan; workstation digunakan user interaktif
B. Server tidak pernah memakai jaringan
C. Workstation tidak punya user
D. Keduanya selalu identik

**Kunci:** A

**Pembahasan:** Kebutuhan server dan workstation berbeda sehingga rekomendasi dan dampaknya bisa berbeda.


## Soal 24

Kapan organisasi sebaiknya memilih Level 2?

A. Saat keamanan sangat diutamakan dan organisasi siap menerima potensi dampak usability/performa
B. Saat ingin menghindari testing
C. Saat sistem tidak penting
D. Saat tidak ada admin

**Kunci:** A

**Pembahasan:** Level 2 cocok untuk sistem kritikal atau lingkungan sensitif, dengan exception dan testing yang matang.


## Soal 25

Apa kesalahan umum dalam memakai CIS Benchmark?

A. Membaca Impact sebelum remediation
B. Menerapkan semua rekomendasi secara buta tanpa memahami fungsi sistem
C. Mendokumentasikan exception
D. Melibatkan stakeholder

**Kunci:** B

**Pembahasan:** CIS bukan script sekali jalan. Rekomendasi harus dipahami, diuji, dan disesuaikan konteks.


# B. Bab 1 — Initial Setup


## Soal 26

Apa tujuan utama Bab 1 Initial Setup?

A. Membentuk fondasi awal keamanan sebelum sistem dipakai serius
B. Menghapus semua file user
C. Menonaktifkan jaringan secara permanen
D. Mengatur tema desktop

**Kunci:** A

**Pembahasan:** Initial Setup membahas fondasi seperti filesystem, repository, MAC, bootloader, process hardening, banner, dan GDM.


## Soal 27

Apa itu filesystem kernel module?

A. Komponen kernel yang menambah dukungan filesystem tertentu
B. Aplikasi GUI untuk membuka file
C. Akun khusus untuk mount disk
D. Firewall untuk HTTP

**Kunci:** A

**Pembahasan:** Kernel module dapat menambahkan dukungan terhadap filesystem seperti cramfs, hfs, udf, overlay, dan lain-lain.


## Soal 28

Mengapa filesystem yang tidak diperlukan perlu dinonaktifkan?

A. Agar seluruh disk tidak bisa dibaca
B. Untuk mengurangi local attack surface dan kode kernel yang bisa dipicu
C. Agar user tidak bisa login
D. Agar repository APT mati

**Kunci:** B

**Pembahasan:** Dukungan filesystem tambahan berarti parser/driver tambahan di level kernel.


## Soal 29

Apa yang dimaksud local attack surface pada filesystem?

A. Risiko yang dapat dipicu dari user lokal, media eksternal, image filesystem, atau proses lokal
B. Serangan dari luar angkasa
C. Hanya serangan browser
D. Risiko pada DNS server

**Kunci:** A

**Pembahasan:** Filesystem module bisa dipicu oleh mount/media/image dan berjalan dekat kernel.


## Soal 30

Perbedaan native dan non-native filesystem adalah...

A. Native umum dirancang untuk Linux; non-native berasal dari OS/media/kebutuhan lain
B. Native selalu tidak aman
C. Non-native tidak bisa dipakai di Linux
D. Native hanya untuk DVD

**Kunci:** A

**Pembahasan:** Non-native seperti hfs/hfsplus/freevxfs/udf dapat memiliki model berbeda dari Linux standar.


## Soal 31

Mengapa cramfs biasanya dinonaktifkan pada server Debian modern?

A. Karena cramfs adalah web server
B. Karena compressed read-only filesystem ini biasanya tidak dibutuhkan pada server umum
C. Karena cramfs adalah firewall
D. Karena cramfs menyimpan password

**Kunci:** B

**Pembahasan:** Jika tidak digunakan, dukungan cramfs hanya menambah attack surface.


## Soal 32

Mengapa freevxfs termasuk filesystem yang sebaiknya tidak tersedia bila tidak dibutuhkan?

A. Karena terkait filesystem Veritas/HP-UX yang jarang relevan pada Debian umum
B. Karena digunakan untuk SSH
C. Karena wajib untuk AppArmor
D. Karena bagian dari cron

**Kunci:** A

**Pembahasan:** freevxfs adalah contoh filesystem niche yang umumnya tidak dibutuhkan.


## Soal 33

Apa risiko hfs/hfsplus pada server Linux jika tidak diperlukan?

A. Menambah dukungan non-native filesystem Mac OS yang tidak relevan
B. Mempercepat boot
C. Mengaktifkan auditd otomatis
D. Menutup semua port

**Kunci:** A

**Pembahasan:** Non-native filesystem dapat memperluas kode parser kernel yang tidak perlu.


## Soal 34

Mengapa overlay berbeda dari cramfs dalam konteks CIS?

A. Overlay sering dibutuhkan container sehingga penonaktifannya bisa berdampak operasional
B. Overlay tidak pernah digunakan
C. Overlay hanya untuk printer
D. Overlay menggantikan SSH

**Kunci:** A

**Pembahasan:** OverlayFS penting untuk Docker/Podman/container. Jika dibutuhkan, buat exception.


## Soal 35

Mengapa squashfs perlu dipertimbangkan hati-hati sebelum dinonaktifkan?

A. Karena dapat dipakai oleh Snap atau image terkompresi
B. Karena selalu berisi password root
C. Karena wajib untuk sudo
D. Karena membuat DNS berjalan

**Kunci:** A

**Pembahasan:** squashfs pada beberapa lingkungan dibutuhkan. Pada Level 2 dampaknya harus diuji.


## Soal 36

Mengapa udf tidak selalu boleh dimatikan secara buta?

A. Beberapa platform atau kebutuhan media optik/cloud tertentu dapat memerlukannya
B. Karena udf adalah modul SSH
C. Karena semua paket APT membutuhkan udf
D. Karena udf mengatur PAM

**Kunci:** A

**Pembahasan:** UDF digunakan untuk media optik dan pada platform tertentu bisa ada kebutuhan khusus.


## Soal 37

Mengapa usb-storage sering dianggap berisiko pada server?

A. Karena media USB dapat membawa malware atau dipakai eksfiltrasi data
B. Karena USB membuat CPU lambat
C. Karena USB adalah DNS server
D. Karena USB mematikan audit

**Kunci:** A

**Pembahasan:** USB storage membuka jalur media eksternal. Jika dibutuhkan, dokumentasikan exception.


## Soal 38

Apa arti blacklist module secara konseptual?

A. Mencegah module dimuat secara otomatis/aktif, bukan sekadar melihat modul tidak sedang aktif
B. Menghapus user dari sistem
C. Mengubah password root
D. Menghapus semua repository

**Kunci:** A

**Pembahasan:** Hardening module mencakup loaded dan loadable availability.


## Soal 39

Mengapa /tmp sebaiknya dipisahkan atau diberi mount option khusus?

A. Karena /tmp sering world-writable dan bisa dipakai menyimpan payload
B. Karena /tmp menyimpan kernel utama
C. Karena /tmp harus selalu SUID
D. Karena /tmp adalah repository APT

**Kunci:** A

**Pembahasan:** /tmp adalah area sementara yang sering ditulis banyak user/proses.


## Soal 40

Apa fungsi mount option nodev?

A. Mencegah device special file diperlakukan sebagai perangkat valid pada filesystem tersebut
B. Mencegah file dibaca
C. Menghapus semua file saat reboot
D. Mengaktifkan SSH

**Kunci:** A

**Pembahasan:** nodev relevan pada /tmp, /home, /var/log, dan area yang tidak seharusnya memuat device file.


## Soal 41

Apa fungsi mount option nosuid?

A. Membuat bit SUID/SGID tidak berlaku pada filesystem tersebut
B. Mencegah semua network packet
C. Mengaktifkan UFW
D. Menghapus user root

**Kunci:** A

**Pembahasan:** nosuid mengurangi risiko privilege escalation dari filesystem yang bisa ditulis user/aplikasi.


## Soal 42

Apa fungsi mount option noexec?

A. Mencegah eksekusi langsung file dari filesystem tersebut
B. Mencegah file dibaca
C. Membuat filesystem read-only
D. Mengatur DNS

**Kunci:** A

**Pembahasan:** noexec cocok untuk area sementara/log yang tidak seharusnya menjadi tempat menjalankan program.


## Soal 43

Mengapa /var/log sebaiknya dipisahkan?

A. Agar pertumbuhan log tidak memenuhi root filesystem dan log dapat diberi kontrol khusus
B. Agar tidak ada log sama sekali
C. Agar SSH tidak berjalan
D. Agar home user hilang

**Kunci:** A

**Pembahasan:** /var/log berisi catatan sistem/aplikasi yang dapat tumbuh besar.


## Soal 44

Mengapa /var/log/audit perlu perhatian khusus?

A. Karena audit log adalah bukti keamanan dan kepatuhan yang harus tetap tersedia
B. Karena berisi wallpaper
C. Karena menyimpan file ISO
D. Karena menggantikan /etc/passwd

**Kunci:** A

**Pembahasan:** Audit log harus diproteksi dan dipisah agar tidak terganggu oleh log aplikasi biasa.


## Soal 45

Apa risiko jika direktori log dan temporary bercampur dengan root filesystem?

A. Resource exhaustion dapat membuat sistem inti terganggu
B. Sistem pasti lebih cepat
C. Firewall menjadi lebih ketat
D. PAM otomatis aman

**Kunci:** A

**Pembahasan:** Jika root penuh, service, login, update, atau logging bisa gagal.


## Soal 46

Apa konsep repository terpercaya dalam package management?

A. Sumber paket jelas, metadata diverifikasi, dan key/signature dapat dipercaya
B. Repository yang paling banyak iklannya
C. Repository tanpa GPG key
D. Repository dari flashdisk acak

**Kunci:** A

**Pembahasan:** Package manager berjalan dengan privilege tinggi, sehingga sumber paket harus tepercaya.


## Soal 47

Apa risiko package tampering?

A. Paket dimodifikasi pihak tidak sah sehingga sistem dapat memasang software berbahaya
B. Paket menjadi lebih kecil
C. Password otomatis berubah
D. SSH menjadi tidak bisa dihapus

**Kunci:** A

**Pembahasan:** Signed repository dan GPG key membantu mencegah paket palsu/tampering.


## Soal 48

Mengapa file konfigurasi APT harus dibatasi aksesnya?

A. Agar user biasa tidak dapat menambah repository/key berbahaya
B. Agar user bisa mengganti mirror sembarang
C. Agar semua paket gagal update
D. Agar audit tidak berjalan

**Kunci:** A

**Pembahasan:** Konfigurasi repository menentukan sumber software yang dipercaya sistem.


## Soal 49

Apa hubungan patching dengan vulnerability management?

A. Patching menutup kerentanan yang sudah diketahui sebagai bagian dari pengelolaan risiko
B. Patching hanya mengubah tema terminal
C. Patching menggantikan logging
D. Patching membuat semua service terbuka

**Kunci:** A

**Pembahasan:** Update keamanan mengurangi risiko eksploitasi vulnerability yang sudah dipublikasikan.


## Soal 50

Apa itu Mandatory Access Control?

A. Model kontrol akses yang membatasi proses berdasarkan kebijakan sistem, tidak hanya keputusan owner file
B. Sistem password BIOS
C. Aplikasi backup
D. Firewall router

**Kunci:** A

**Pembahasan:** MAC seperti AppArmor membatasi aplikasi meskipun DAC permission mungkin mengizinkan.


## Soal 51

Perbedaan DAC dan MAC adalah...

A. DAC dikendalikan owner/permission tradisional; MAC dikendalikan kebijakan sistem yang lebih wajib
B. DAC hanya untuk jaringan; MAC hanya untuk printer
C. DAC lebih baru dari MAC
D. MAC tidak bisa membatasi proses

**Kunci:** A

**Pembahasan:** Linux permission adalah DAC; AppArmor/SELinux adalah contoh MAC.


## Soal 52

Apa arti profile AppArmor dalam mode enforcing?

A. Kebijakan pembatasan benar-benar diterapkan pada proses
B. Profil hanya dicetak tanpa efek
C. User bebas mengabaikannya
D. Hanya berlaku untuk hostname

**Kunci:** A

**Pembahasan:** Enforcing berarti pelanggaran profile diblokir sesuai kebijakan.


## Soal 53

Mengapa bootloader perlu dilindungi?

A. Akses fisik ke bootloader dapat dipakai mengubah parameter boot atau masuk mode recovery
B. Bootloader menyimpan semua log aplikasi
C. Bootloader adalah web browser
D. Bootloader menggantikan firewall

**Kunci:** A

**Pembahasan:** Bootloader dapat menjadi jalur bypass jika tidak dikunci, terutama pada akses fisik.


## Soal 54

Apa tujuan ptrace restriction?

A. Membatasi proses/user menginspeksi proses lain untuk mengurangi pencurian informasi atau debugging jahat
B. Membuka semua proses untuk user biasa
C. Menghapus kernel
D. Mengaktifkan DNS

**Kunci:** A

**Pembahasan:** ptrace dapat dipakai debugging, tetapi juga dapat dipakai membaca memori proses lain.


## Soal 55

Mengapa core dump sering dibatasi?

A. Core dump dapat menyimpan isi memori sensitif seperti kredensial atau data aplikasi
B. Core dump selalu diperlukan untuk login
C. Core dump adalah firewall
D. Core dump membuat APT aman

**Kunci:** A

**Pembahasan:** Core dump berguna untuk debugging tetapi berisiko membocorkan data sensitif.


## Soal 56

Apa fungsi dmesg restriction?

A. Membatasi user biasa membaca pesan kernel yang dapat membocorkan informasi sistem
B. Membuat semua user root
C. Menghapus log audit
D. Mengaktifkan XDMCP

**Kunci:** A

**Pembahasan:** Pesan kernel dapat membantu attacker fingerprinting atau mencari informasi perangkat.


## Soal 57

Apa itu ASLR?

A. Randomisasi lokasi memori untuk mempersulit eksploitasi memory corruption
B. Sistem backup log
C. Pengganti SSH
D. Repository package

**Kunci:** A

**Pembahasan:** ASLR mengacak alamat memori sehingga exploit lebih sulit diprediksi.


## Soal 58

Mengapa prelink biasanya tidak disarankan dalam hardening modern?

A. Ia dapat bertentangan dengan mekanisme randomisasi dan integritas file tertentu
B. Karena prelink adalah antivirus
C. Karena wajib untuk AppArmor
D. Karena membuat firewall mati

**Kunci:** A

**Pembahasan:** Prelink mengubah binary dan dapat melemahkan manfaat ASLR/integrity checking.


## Soal 59

Apa fungsi login banner?

A. Memberi legal notice bahwa akses terbatas dan aktivitas dapat dimonitor tanpa membocorkan detail sistem
B. Menampilkan semua versi kernel
C. Memberi password tamu
D. Mengaktifkan Wi-Fi

**Kunci:** A

**Pembahasan:** Banner mendukung notice hukum/audit, tetapi tidak boleh membocorkan informasi teknis.


## Soal 60

Perbedaan /etc/issue dan /etc/issue.net adalah...

A. /etc/issue untuk login lokal, /etc/issue.net untuk banner koneksi jaringan seperti SSH/telnet tertentu
B. Keduanya menyimpan password
C. /etc/issue hanya untuk firewall
D. /etc/issue.net hanya untuk cron

**Kunci:** A

**Pembahasan:** Banner berbeda sesuai jalur login lokal atau remote.


## Soal 61

Mengapa daftar user pada GDM sebaiknya dinonaktifkan?

A. Agar penyerang tidak mudah melakukan enumerasi username dari layar login
B. Agar semua user hilang
C. Agar GUI lebih cepat
D. Agar /etc/passwd kosong

**Kunci:** A

**Pembahasan:** Disable user list mengurangi kebocoran informasi akun lokal.


## Soal 62

Apa risiko automount/autorun di lingkungan GUI?

A. Media eksternal dapat otomatis diproses atau dijalankan tanpa kontrol yang cukup
B. Semua media menjadi terenkripsi
C. Auditd otomatis berhenti
D. UFW otomatis membuka semua port

**Kunci:** A

**Pembahasan:** Automount/autorun memperbesar risiko dari removable media.


## Soal 63

Mengapa XDMCP umumnya dinonaktifkan?

A. Karena membuka akses display manager melalui jaringan dan dapat memperbesar attack surface
B. Karena wajib untuk SSH
C. Karena hanya berlaku untuk logrotate
D. Karena menghapus GNOME

**Kunci:** A

**Pembahasan:** XDMCP adalah fitur remote display lama yang jarang dibutuhkan pada hardening modern.


# C. Bab 2 — Services


## Soal 64

Apa prinsip utama Bab 2 Services?

A. Menjalankan hanya layanan yang benar-benar dibutuhkan
B. Mengaktifkan semua daemon untuk kompatibilitas
C. Membuka semua port agar mudah diakses
D. Menghapus package manager

**Kunci:** A

**Pembahasan:** Service minimization adalah inti Bab 2.


## Soal 65

Mengapa daemon yang tidak digunakan tetap berisiko?

A. Karena bisa membuka port, memiliki bug, dan tidak dimonitor secara serius
B. Karena selalu mempercepat sistem
C. Karena membuat password lebih kuat
D. Karena menghapus log

**Kunci:** A

**Pembahasan:** Daemon tidak perlu adalah unmanaged exposure.


## Soal 66

Perbedaan server service dan client tool adalah...

A. Server service menerima koneksi; client tool dipakai mengakses layanan lain
B. Client selalu membuka port 80
C. Server service tidak pernah berisiko
D. Client tool selalu lebih aman

**Kunci:** A

**Pembahasan:** Server meningkatkan risiko incoming attack; client lama bisa membocorkan kredensial.


## Soal 67

Mengapa port terbuka harus punya alasan sah?

A. Karena port adalah titik masuk jaringan ke service
B. Karena port menyimpan password
C. Karena port memformat disk
D. Karena port membuat user baru

**Kunci:** A

**Pembahasan:** Approved listening services harus sesuai fungsi dan terdokumentasi.


## Soal 68

Mengapa autofs berisiko pada server?

A. Automount media eksternal dapat mengurangi kontrol manual terhadap perangkat/berkas
B. autofs adalah DNS server
C. autofs menghapus firewall
D. autofs mengganti sudo

**Kunci:** A

**Pembahasan:** Automount dapat memproses media yang tidak tepercaya.


## Soal 69

Apa konsep MTA local-only?

A. MTA boleh untuk email lokal, tetapi tidak perlu menerima koneksi jaringan jika bukan mail server
B. MTA harus selalu publik
C. MTA menggantikan SSH
D. MTA tidak boleh mengirim email lokal

**Kunci:** A

**Pembahasan:** Kebutuhan notifikasi lokal berbeda dari fungsi mail server publik.


## Soal 70

Mengapa avahi biasanya tidak diperlukan pada server?

A. Discovery otomatis dapat mengumumkan layanan/host ke jaringan lokal
B. Avahi adalah tool patching
C. Avahi wajib untuk auditd
D. Avahi mengamankan root

**Kunci:** A

**Pembahasan:** Zeroconf/mDNS lebih cocok untuk lingkungan desktop/rumah daripada server terkontrol.


## Soal 71

Mengapa DHCP server tidak boleh berjalan tanpa mandat resmi?

A. Ia dapat memberi gateway/DNS/IP yang salah kepada client
B. Ia hanya membaca log
C. Ia menutup semua port
D. Ia membuat AppArmor enforcing

**Kunci:** A

**Pembahasan:** DHCP memengaruhi konfigurasi jaringan client secara langsung.


## Soal 72

Kapan web server menjadi exception yang sah?

A. Saat host memang ditugaskan sebagai web server dan kontrol pengamanannya diterapkan
B. Saat admin lupa mematikannya
C. Saat ingin skor audit turun
D. Saat tidak ada dokumentasi

**Kunci:** A

**Pembahasan:** Web server sah jika sesuai fungsi sistem dan terdokumentasi.


## Soal 73

Mengapa DNS server tidak boleh aktif pada host biasa?

A. DNS dapat membocorkan informasi, membuka rekursi, atau mengganggu resolusi jika tidak resmi
B. DNS selalu aman
C. DNS diperlukan untuk semua login lokal
D. DNS mengganti PAM

**Kunci:** A

**Pembahasan:** DNS adalah layanan inti jaringan yang harus dikelola resmi.


## Soal 74

Mengapa FTP server tidak disarankan?

A. FTP tidak menyediakan enkripsi bawaan untuk kredensial dan data
B. FTP hanya berjalan offline
C. FTP adalah package manager
D. FTP menutup SSH

**Kunci:** A

**Pembahasan:** FTP biasa dapat mengirim username/password dalam bentuk mudah disadap.


## Soal 75

Mengapa dnsmasq perlu dikontrol?

A. Karena dapat menyediakan DNS/DHCP dan memengaruhi jaringan
B. Karena hanya editor teks
C. Karena tidak bisa listening
D. Karena menghapus audit

**Kunci:** A

**Pembahasan:** dnsmasq kecil tetapi dapat berdampak besar pada resolusi dan konfigurasi jaringan.


## Soal 76

Mengapa LDAP server sensitif?

A. Ia dapat menjadi pusat informasi identitas/direktori
B. Ia hanya menyimpan gambar
C. Ia tidak terkait autentikasi
D. Ia adalah driver USB

**Kunci:** A

**Pembahasan:** LDAP sering berhubungan dengan identity management.


## Soal 77

Perbedaan MTA dan message access server adalah...

A. MTA menangani pengiriman/antarserver; message access server seperti IMAP/POP3 memberi akses mailbox user
B. Keduanya firewall
C. IMAP selalu lebih aman dari SSH
D. MTA hanya untuk DNS

**Kunci:** A

**Pembahasan:** IMAP/POP3 hanya dibutuhkan pada mailbox server.


## Soal 78

Mengapa NFS server berisiko bila tidak diperlukan?

A. Karena mengekspos filesystem melalui jaringan
B. Karena hanya membaca hostname
C. Karena menghapus kernel
D. Karena membuat banner

**Kunci:** A

**Pembahasan:** NFS menyentuh akses file langsung via jaringan.


## Soal 79

Mengapa NIS server dianggap legacy dan tidak disarankan?

A. Karena mekanisme identitas lama ini memiliki kelemahan historis dan sudah banyak diganti LDAP/modern identity
B. Karena NIS adalah browser
C. Karena NIS wajib untuk UFW
D. Karena NIS mengaktifkan ASLR

**Kunci:** A

**Pembahasan:** NIS adalah teknologi lama untuk distribusi informasi user/group.


## Soal 80

Mengapa print server tidak perlu pada server umum?

A. Karena CUPS/print service membuka fungsi jaringan dan parser dokumen yang tidak relevan
B. Karena print server membuat password kuat
C. Karena print server mengganti chrony
D. Karena print server diperlukan oleh auditd

**Kunci:** A

**Pembahasan:** Print service hanya perlu pada host dengan fungsi pencetakan.


## Soal 81

Apa fungsi rpcbind dan kenapa berisiko?

A. Memetakan layanan RPC ke port; jika tidak dibutuhkan dapat mengungkap/menopang layanan RPC
B. Mengatur password user
C. Mengompres log
D. Mengatur banner SSH

**Kunci:** A

**Pembahasan:** rpcbind erat dengan layanan RPC seperti NFS.


## Soal 82

Apa bedanya rsync sebagai tool dan rsync daemon?

A. Tool via SSH bisa sah; daemon listening membuka akses sinkronisasi jaringan yang perlu dikontrol
B. Keduanya selalu dilarang
C. Daemon tidak membuka port
D. Tool selalu lebih berbahaya dari daemon

**Kunci:** A

**Pembahasan:** rsync daemon memiliki risiko server service.


## Soal 83

Mengapa Samba harus hanya aktif pada file server resmi?

A. Karena SMB/CIFS membuka akses file lintas jaringan dan sangat bergantung konfigurasi share/permission
B. Karena Samba adalah kernel module
C. Karena Samba hanya untuk AppArmor
D. Karena Samba menghapus /etc/passwd

**Kunci:** A

**Pembahasan:** Samba salah konfigurasi dapat membuka data internal.


## Soal 84

Mengapa SNMP perlu dikonfigurasi hati-hati?

A. Karena dapat membocorkan informasi sistem, terutama dengan versi/community lemah
B. Karena SNMP selalu mengenkripsi semua data
C. Karena SNMP menggantikan SSH
D. Karena SNMP hanya untuk printer lokal

**Kunci:** A

**Pembahasan:** Jika SNMP dibutuhkan, SNMPv3 lebih aman dibanding versi lama.


## Soal 85

Mengapa telnet server harus dihindari?

A. Karena remote login Telnet tidak terenkripsi
B. Karena telnet selalu memakai TLS kuat
C. Karena telnet adalah cron
D. Karena telnet membuat log aman

**Kunci:** A

**Pembahasan:** SSH menggantikan Telnet untuk remote administration aman.


## Soal 86

Mengapa TFTP server berisiko?

A. Karena sederhana, tanpa autentikasi/enkripsi kuat, dan harus dibatasi ketat
B. Karena selalu memerlukan MFA
C. Karena TFTP hanya berjalan lokal
D. Karena TFTP mengaktifkan PAM

**Kunci:** A

**Pembahasan:** TFTP cocok untuk PXE/provisioning, bukan host umum.


## Soal 87

Mengapa web proxy tidak boleh aktif sembarangan?

A. Karena bisa menjadi open proxy atau jalur bypass kontrol jaringan
B. Karena proxy menutup semua trafik
C. Karena proxy adalah shell
D. Karena proxy wajib untuk auditd

**Kunci:** A

**Pembahasan:** Proxy harus resmi, terdokumentasi, dan dikontrol.


## Soal 88

Mengapa xinetd berisiko?

A. Karena super daemon dapat mengaktifkan banyak service jaringan lama/tersembunyi
B. Karena xinetd adalah filesystem
C. Karena xinetd menghapus firewall
D. Karena xinetd menyimpan password hash

**Kunci:** A

**Pembahasan:** xinetd dapat menjadi pintu banyak layanan kecil/legacy.


## Soal 89

Mengapa X Window Server tidak perlu pada server headless?

A. GUI stack menambah paket, library, proses, dan attack surface
B. Karena server tidak boleh punya kernel
C. Karena GUI mematikan semua service
D. Karena X Window sama dengan SSH

**Kunci:** A

**Pembahasan:** Server CLI/headless biasanya tidak membutuhkan GUI.


## Soal 90

Mengapa client tool lama seperti telnet client tetap berbahaya?

A. User dapat tidak sengaja mengirim kredensial lewat protokol tidak terenkripsi
B. Karena client selalu listening
C. Karena client tidak bisa dipakai attacker
D. Karena telnet client mengenkripsi password

**Kunci:** A

**Pembahasan:** Client lama mendorong unsafe usage walau bukan server.


## Soal 91

Mengapa rsh client sebaiknya tidak tersedia?

A. Karena memakai model remote shell lama yang lemah dibanding SSH
B. Karena rsh adalah package manager
C. Karena rsh wajib untuk UFW
D. Karena rsh adalah audit rule

**Kunci:** A

**Pembahasan:** rsh/rlogin/rcp legacy dan tidak sesuai keamanan modern.


## Soal 92

Mengapa talk client tidak umum dipertahankan?

A. Karena protokol lama ini jarang dibutuhkan dan dapat menambah permukaan penggunaan tidak aman
B. Karena talk adalah firewall
C. Karena talk menyimpan log audit
D. Karena talk mengaktifkan ASLR

**Kunci:** A

**Pembahasan:** Client lama yang tidak dipakai sebaiknya dihapus.


## Soal 93

Apa risiko FTP client?

A. User dapat memakai protokol transfer file tidak terenkripsi
B. FTP client selalu memblokir password
C. FTP client adalah AppArmor
D. FTP client tidak pernah mengirim data

**Kunci:** A

**Pembahasan:** SFTP/HTTPS/secure transfer lebih sesuai keamanan modern.


## Soal 94

Mengapa sinkronisasi waktu penting?

A. Log, audit, sertifikat, korelasi insiden, dan forensik bergantung pada waktu akurat
B. Karena waktu akurat menghapus firewall
C. Karena semua service tidak butuh waktu
D. Karena waktu mengganti permission

**Kunci:** A

**Pembahasan:** Timestamp yang benar adalah dasar investigasi dan audit.


## Soal 95

Mengapa hanya satu daemon sinkronisasi waktu dipakai?

A. Agar tidak ada konflik antar daemon yang sama-sama mengatur waktu
B. Agar waktu tidak pernah berubah
C. Agar SSH mati
D. Agar audit log dihapus

**Kunci:** A

**Pembahasan:** Gunakan systemd-timesyncd atau chrony, bukan keduanya bersaing.


## Soal 96

Apa keunggulan chrony secara umum?

A. Cocok untuk sinkronisasi waktu yang fleksibel dan robust pada berbagai kondisi jaringan
B. Menghapus semua log
C. Mengganti firewall
D. Mengatur group user

**Kunci:** A

**Pembahasan:** chrony sering dipakai untuk server dan lingkungan yang membutuhkan sinkronisasi baik.


## Soal 97

Apa itu cron dan at?

A. Mekanisme scheduler untuk menjalankan tugas terjadwal atau sekali waktu
B. Firewall kernel
C. Client FTP
D. Paket AppArmor

**Kunci:** A

**Pembahasan:** Scheduler dapat menjalankan perintah otomatis, sehingga sensitif.


## Soal 98

Mengapa file cron harus dibatasi aksesnya?

A. Karena perubahan job terjadwal dapat menjadi persistence attacker
B. Karena cron harus bisa ditulis semua user
C. Karena cron adalah DNS
D. Karena cron tidak bisa menjalankan command

**Kunci:** A

**Pembahasan:** Attacker sering memakai scheduled tasks untuk persistence.


## Soal 99

Apa hubungan scheduler dengan persistence attacker?

A. Job terjadwal dapat membuat malware/backdoor berjalan ulang secara otomatis
B. Scheduler mencegah semua login
C. Scheduler memformat disk
D. Scheduler menghapus AppArmor

**Kunci:** A

**Pembahasan:** Cron/at yang tidak dikontrol bisa menjaga akses attacker tetap aktif.


# D. Bab 3 — Network


## Soal 100

Apa fokus utama Bab 3 Network?

A. Mengurangi attack surface jaringan dari perangkat, modul kernel, dan parameter IPv4/IPv6
B. Mengatur tema terminal
C. Menghapus semua user
D. Membuat database

**Kunci:** A

**Pembahasan:** Bab 3 membahas perangkat jaringan, modul kernel jaringan, dan parameter kernel.


## Soal 101

Mengapa status IPv6 harus diidentifikasi?

A. Karena IPv6 bisa aktif tanpa dikontrol firewall/monitoring bila organisasi hanya fokus IPv4
B. Karena IPv6 selalu harus dimatikan
C. Karena IPv6 adalah antivirus
D. Karena IPv6 menggantikan PAM

**Kunci:** A

**Pembahasan:** IPv6 tidak selalu harus dimatikan, tetapi harus diketahui dan diamankan.


## Soal 102

Mengapa wireless interface berisiko pada server?

A. Dapat menjadi jalur jaringan tambahan di luar desain segmentasi kabel
B. Karena Wi-Fi membuat password kuat
C. Karena Wi-Fi tidak pernah dipakai attacker
D. Karena Wi-Fi adalah filesystem

**Kunci:** A

**Pembahasan:** Server produksi biasanya tidak membutuhkan Wi-Fi.


## Soal 103

Mengapa Bluetooth service sebaiknya tidak aktif pada server?

A. Bluetooth membuka kanal komunikasi lokal yang biasanya tidak dibutuhkan server
B. Bluetooth adalah DNS server
C. Bluetooth mengganti auditd
D. Bluetooth membuat /tmp aman

**Kunci:** A

**Pembahasan:** Bluetooth memperbesar attack surface fisik/lokal.


## Soal 104

Apa konsep network kernel module?

A. Modul kernel yang menambah dukungan protokol jaringan tertentu
B. Aplikasi GUI jaringan
C. Password jaringan
D. File log web

**Kunci:** A

**Pembahasan:** Module seperti atm, can, dccp, rds, sctp, tipc menambah protokol kernel.


## Soal 105

Mengapa protokol jaringan yang tidak dipakai perlu dinonaktifkan?

A. Untuk mengurangi kode/protokol kernel yang dapat disalahgunakan atau dieksploitasi
B. Untuk membuka semua port
C. Untuk membuat DNS aktif
D. Untuk menghapus SSH

**Kunci:** A

**Pembahasan:** Least functionality berlaku juga untuk protokol kernel.


## Soal 106

Apa perbedaan module loaded dan loadable?

A. Loaded sedang aktif; loadable belum aktif tetapi masih bisa dimuat
B. Keduanya sama persis
C. Loaded berarti sudah dihapus
D. Loadable berarti tidak ada di disk

**Kunci:** A

**Pembahasan:** Hardening ideal mencegah module tidak perlu dimuat ulang.


## Soal 107

ATM dalam konteks network module adalah...

A. Asynchronous Transfer Mode, teknologi jaringan lama/khusus yang jarang dibutuhkan server umum
B. Antivirus Traffic Manager
C. Apache TLS Module
D. Audit Time Monitor

**Kunci:** A

**Pembahasan:** Jika tidak digunakan, ATM module menambah attack surface.


## Soal 108

CAN biasanya relevan untuk...

A. Controller Area Network pada kendaraan/embedded/industri
B. SSH server
C. Package manager
D. Rsyslog

**Kunci:** A

**Pembahasan:** Server umum jarang membutuhkan CAN.


## Soal 109

Mengapa DCCP biasanya dinonaktifkan bila tidak digunakan?

A. Karena protokol datagram dengan congestion control ini jarang dipakai dan dapat luput dari monitoring umum
B. Karena DCCP adalah PAM module
C. Karena DCCP wajib untuk DNS
D. Karena DCCP menghapus logs

**Kunci:** A

**Pembahasan:** DCCP menambah protokol transport di kernel.


## Soal 110

RDS kernel module berkaitan dengan...

A. Reliable Datagram Sockets untuk skenario cluster/database tertentu
B. Remote Desktop Service Windows semata
C. Rotasi log
D. Root password

**Kunci:** A

**Pembahasan:** RDS jarang dibutuhkan server umum.


## Soal 111

SCTP sering ditemukan pada lingkungan...

A. Telekomunikasi/signaling/aplikasi khusus
B. Local shell saja
C. APT repository
D. GNOME wallpaper

**Kunci:** A

**Pembahasan:** SCTP adalah protokol transport khusus dengan multistreaming/multihoming.


## Soal 112

TIPC umumnya digunakan untuk...

A. Inter-process communication dalam cluster/sistem terdistribusi tertentu
B. Mengatur sudo log
C. Membaca /etc/shadow
D. Menjalankan GDM

**Kunci:** A

**Pembahasan:** Jika tidak ada kebutuhan cluster, TIPC tidak diperlukan.


## Soal 113

Apa itu IP forwarding?

A. Kemampuan host meneruskan paket antar-interface seperti router
B. Kemampuan login SSH
C. Kemampuan menyimpan password
D. Kemampuan membaca DVD

**Kunci:** A

**Pembahasan:** Host biasa tidak seharusnya menjadi router tanpa kebutuhan.


## Soal 114

Risiko IP forwarding aktif tanpa kebutuhan adalah...

A. Server dapat menjadi jalur bypass segmentasi jaringan
B. Server menjadi lebih aman otomatis
C. Semua koneksi diblokir
D. Password berubah

**Kunci:** A

**Pembahasan:** Forwarding bisa dipakai pivot/lateral movement.


## Soal 115

Apa risiko menerima ICMP redirect?

A. Routing host dapat dimanipulasi sehingga trafik diarahkan ke gateway berbahaya
B. Log menjadi terkompresi
C. PAM mati
D. Package manager berhenti

**Kunci:** A

**Pembahasan:** ICMP redirect dapat mendukung MITM/manipulasi jalur.


## Soal 116

Apa itu source routing?

A. Paket membawa instruksi jalur yang harus dilalui
B. Enkripsi password
C. Rotasi audit log
D. Autentikasi PAM

**Kunci:** A

**Pembahasan:** Source routing dapat dipakai untuk manipulasi jalur dan sebaiknya ditolak.


## Soal 117

Mengapa broadcast ICMP perlu dibatasi?

A. Untuk mencegah sistem ikut dalam serangan amplifikasi seperti smurf attack
B. Agar semua ping lokal sukses
C. Agar FTP aman
D. Agar AppArmor complain

**Kunci:** A

**Pembahasan:** Broadcast ICMP dapat dimanfaatkan untuk amplifikasi.


## Soal 118

Apa fungsi reverse path filtering?

A. Membantu mendeteksi/menolak paket dengan source address tidak masuk akal/spoofed
B. Menghapus log
C. Mengaktifkan printer
D. Membuat user baru

**Kunci:** A

**Pembahasan:** rp_filter membantu mencegah IP spoofing tertentu.


## Soal 119

Apa itu martian packet logging?

A. Pencatatan paket dengan alamat sumber/tujuan yang tidak valid atau mencurigakan
B. Logging browser
C. Logging password plaintext
D. Logging file home

**Kunci:** A

**Pembahasan:** Martian packet log memberi visibilitas terhadap trafik mencurigakan.


## Soal 120

Apa fungsi TCP SYN cookies?

A. Membantu mitigasi SYN flood saat backlog koneksi penuh
B. Membuat SSH tanpa password
C. Menghapus daemon
D. Mengaktifkan GUI

**Kunci:** A

**Pembahasan:** SYN cookies mempertahankan availability terhadap serangan SYN flood.


## Soal 121

Apa risiko salah konfigurasi IPv6?

A. IPv6 dapat menjadi jalur bypass jika firewall, redirect, RA, dan forwarding tidak dikontrol
B. IPv6 tidak pernah melewati jaringan
C. IPv6 hanya dipakai untuk file lokal
D. IPv6 selalu mati di semua sistem

**Kunci:** A

**Pembahasan:** Dual-stack perlu hardening di IPv4 dan IPv6.


## Soal 122

Mengapa IPv6 forwarding harus dimatikan pada host biasa?

A. Agar host tidak menjadi router IPv6 tanpa kebutuhan
B. Agar semua IPv6 selalu aktif
C. Agar password hash hilang
D. Agar AIDE mati

**Kunci:** A

**Pembahasan:** Forwarding hanya untuk router/gateway/VPN/container tertentu.


## Soal 123

Mengapa IPv6 router advertisement perlu diperhatikan?

A. RA dari jaringan tidak tepercaya dapat memengaruhi konfigurasi routing host
B. RA adalah log audit
C. RA hanya untuk cron
D. RA membuat sudo aman

**Kunci:** A

**Pembahasan:** Host tidak seharusnya sembarang menerima instruksi routing.


## Soal 124

Apa prinsip utama Bab 3 sebelum firewall dibahas?

A. Kurangi perangkat/protokol/perilaku jaringan yang tidak diperlukan
B. Buka semua protokol agar kompatibel
C. Matikan semua logging
D. Jalankan semua client lama

**Kunci:** A

**Pembahasan:** Firewall penting, tetapi perangkat dan parameter jaringan juga harus dikendalikan.


# E. Bab 4 — Host Based Firewall


## Soal 125

Apa itu host-based firewall?

A. Firewall yang berjalan langsung pada sistem operasi host
B. Firewall yang hanya ada di router ISP
C. Antivirus lokal
D. Paket DNS

**Kunci:** A

**Pembahasan:** Host firewall melindungi endpoint/server itu sendiri.


## Soal 126

Perbedaan firewall host dan firewall jaringan adalah...

A. Firewall host melindungi satu host; firewall jaringan melindungi segmen/jalur antarjaringan
B. Firewall host selalu menggantikan semua firewall jaringan
C. Firewall jaringan hanya untuk file lokal
D. Keduanya tidak memfilter paket

**Kunci:** A

**Pembahasan:** Keduanya saling melengkapi dalam defense in depth.


## Soal 127

Mengapa firewall lokal tetap penting walaupun ada firewall router?

A. Karena dapat membatasi lateral movement dan service yang tidak sengaja terbuka di host
B. Karena router tidak pernah berfungsi
C. Karena firewall lokal mengganti patching
D. Karena firewall lokal menghapus SSH

**Kunci:** A

**Pembahasan:** Firewall host tetap berguna jika attacker sudah berada di jaringan internal.


## Soal 128

Apa prinsip default deny?

A. Semua koneksi ditolak kecuali ada aturan eksplisit yang mengizinkan
B. Semua koneksi diizinkan kecuali diblokir
C. Semua user menjadi root
D. Semua log dihapus

**Kunci:** A

**Pembahasan:** Default deny berbasis allowlist lebih aman daripada menebak semua trafik buruk.


## Soal 129

Mengapa allowlist lebih aman daripada denylist?

A. Lebih mudah mendefinisikan trafik sah daripada seluruh kemungkinan trafik jahat
B. Karena allowlist mengizinkan semua hal
C. Karena denylist selalu lengkap
D. Karena allowlist mematikan firewall

**Kunci:** A

**Pembahasan:** Allowlist memulai dari kondisi tertutup, lalu membuka kebutuhan sah.


## Soal 130

Apa itu incoming traffic?

A. Trafik yang masuk menuju host
B. Trafik yang dibuat host ke luar
C. Trafik antar dua host lain yang tidak melalui host
D. Trafik file lokal

**Kunci:** A

**Pembahasan:** Incoming harus ketat karena menuju service host.


## Soal 131

Mengapa incoming default sebaiknya deny/reject?

A. Agar service tidak sengaja terbuka tidak otomatis dapat diakses
B. Agar semua aplikasi keluar diblokir
C. Agar user tidak bisa login lokal
D. Agar disk tidak penuh

**Kunci:** A

**Pembahasan:** Default deny incoming membatasi titik masuk jaringan.


## Soal 132

Perbedaan deny/drop dan reject adalah...

A. Drop diam-diam; reject memberi respons penolakan eksplisit
B. Drop mengizinkan trafik; reject menyimpan file
C. Keduanya selalu sama
D. Reject menghapus paket dari disk

**Kunci:** A

**Pembahasan:** Pilihan drop/reject bergantung kebijakan keamanan dan troubleshooting.


## Soal 133

Apa itu outgoing traffic?

A. Trafik yang dibuat host menuju luar
B. Trafik yang datang ke host
C. Trafik yang tidak punya jaringan
D. Trafik dari BIOS

**Kunci:** A

**Pembahasan:** Egress control penting untuk mencegah eksfiltrasi dan command-and-control.


## Soal 134

Mengapa outgoing default deny lebih sulit diterapkan?

A. Karena banyak fungsi sah seperti DNS, NTP, update, backup, API, dan logging remote membutuhkan koneksi keluar
B. Karena outgoing tidak pernah dipakai
C. Karena semua outgoing berbahaya
D. Karena outgoing tidak bisa difilter

**Kunci:** A

**Pembahasan:** Egress allowlist membutuhkan inventaris kebutuhan koneksi.


## Soal 135

Apa itu routed traffic?

A. Trafik yang diteruskan melewati host dari satu interface ke interface lain
B. Trafik local loopback saja
C. Trafik password PAM
D. Trafik file shadow

**Kunci:** A

**Pembahasan:** Routed traffic relevan jika host menjadi router/gateway.


## Soal 136

Mengapa routed traffic default sebaiknya deny pada host biasa?

A. Agar host tidak menjadi jalur bypass segmentasi jaringan
B. Agar host menjadi router otomatis
C. Agar semua log hilang
D. Agar USB storage aktif

**Kunci:** A

**Pembahasan:** Routing antar-interface hanya boleh jika fungsi sistem memang membutuhkannya.


## Soal 137

Apa peran UFW pada Debian?

A. Frontend sederhana untuk mengelola aturan firewall host
B. Package manager
C. PAM module
D. Web server

**Kunci:** A

**Pembahasan:** UFW menyederhanakan pengelolaan firewall dibanding aturan low-level.


## Soal 138

Mengapa memasang UFW saja belum cukup?

A. Karena firewall harus aktif dan memiliki default policy/rule yang benar
B. Karena UFW otomatis menghapus semua service
C. Karena UFW hanya bekerja jika tidak ada jaringan
D. Karena UFW adalah antivirus

**Kunci:** A

**Pembahasan:** Installed != configured/enabled.


## Soal 139

Apa risiko utama mengaktifkan firewall dari remote SSH tanpa aturan allow?

A. Administrator bisa terkunci dari server
B. Server otomatis reboot
C. Password root hilang
D. AIDE berhenti

**Kunci:** A

**Pembahasan:** Rule untuk akses administrasi sah harus dibuat sebelum default deny.


## Soal 140

Mengapa sebaiknya tidak mencampur banyak metode firewall tanpa koordinasi?

A. Aturan bisa saling menimpa dan audit menjadi membingungkan
B. Karena firewall tidak bisa punya aturan
C. Karena semua metode identik
D. Karena UFW harus dipakai bersama firewalld selalu

**Kunci:** A

**Pembahasan:** Pilih metode utama seperti UFW/nftables/firewalld dan dokumentasikan.


## Soal 141

Apa contoh incoming rule sah untuk web server?

A. Izinkan HTTP/HTTPS sesuai kebutuhan dan SSH dari sumber admin
B. Izinkan semua port dari semua sumber
C. Blokir loopback
D. Izinkan telnet publik

**Kunci:** A

**Pembahasan:** Rule harus sesuai fungsi sistem.


## Soal 142

Mengapa firewall lokal membantu pada workstation/laptop?

A. Perangkat berpindah jaringan sehingga tidak selalu dilindungi firewall router yang sama
B. Karena laptop tidak punya jaringan
C. Karena firewall lokal menghapus file user
D. Karena laptop tidak butuh update

**Kunci:** A

**Pembahasan:** Host firewall memberi baseline proteksi di berbagai jaringan.


## Soal 143

Mengapa IPv6 perlu dipertimbangkan dalam firewall?

A. Karena sistem bisa memiliki trafik IPv6 yang tidak terblokir oleh aturan IPv4
B. Karena IPv6 adalah file log
C. Karena IPv6 selalu nonaktif
D. Karena firewall tidak mendukung IPv6

**Kunci:** A

**Pembahasan:** Hardening harus mencakup IPv4 dan IPv6 jika aktif.


## Soal 144

Apa inti Bab 4 Host Based Firewall?

A. Menyaring trafik host berdasarkan kebutuhan sah sebagai lapisan defense in depth
B. Mengganti semua kontrol keamanan lain
C. Membuat semua service aktif
D. Menghapus audit

**Kunci:** A

**Pembahasan:** Firewall host membatasi komunikasi masuk, keluar, dan routed sesuai kebijakan.


# F. Bab 5 — Access Control


## Soal 145

Mengapa SSH dianggap pintu akses administrasi utama?

A. Karena admin dapat mengelola server jarak jauh melalui shell
B. Karena SSH menyimpan semua paket APT
C. Karena SSH hanya untuk GUI
D. Karena SSH menghapus sudo

**Kunci:** A

**Pembahasan:** Kompromi SSH bisa menjadi kompromi sistem, terutama bila akun punya sudo/root.


## Soal 146

Risiko PermitRootLogin aktif adalah...

A. Penyerang cukup menarget akun root yang namanya pasti diketahui dan akuntabilitas admin melemah
B. Semua user otomatis terkunci
C. Log menjadi lebih rinci
D. Firewall mati

**Kunci:** A

**Pembahasan:** Login root langsung sebaiknya diganti login user personal + sudo.


## Soal 147

Mengapa permission file sshd_config penting?

A. Agar user biasa tidak dapat melemahkan konfigurasi SSH
B. Agar semua user bisa mengedit SSH
C. Agar SSH selalu publik
D. Agar host key hilang

**Kunci:** A

**Pembahasan:** sshd_config menentukan akses remote.


## Soal 148

Apa fungsi SSH host private key?

A. Identitas rahasia server untuk autentikasi server ke client
B. Password user biasa
C. Log systemd
D. File konfigurasi UFW

**Kunci:** A

**Pembahasan:** Jika private host key bocor, risiko impersonasi/MITM meningkat.


## Soal 149

Apa fungsi banner SSH?

A. Memberi legal notice/consent tanpa membocorkan informasi teknis
B. Menampilkan password root
C. Menampilkan daftar user lengkap
D. Mengaktifkan FTP

**Kunci:** A

**Pembahasan:** Banner mendukung aspek legal dan audit.


## Soal 150

Cipher pada SSH berfungsi untuk...

A. Menjaga kerahasiaan data sesi SSH
B. Membuat user baru
C. Menyimpan log
D. Mengatur cron

**Kunci:** A

**Pembahasan:** Cipher adalah algoritma enkripsi.


## Soal 151

MAC pada SSH berfungsi untuk...

A. Menjaga integritas pesan
B. Mengatur alamat IP
C. Menghapus firewall
D. Membuat passphrase

**Kunci:** A

**Pembahasan:** MAC mendeteksi perubahan pesan saat transit.


## Soal 152

KexAlgorithms pada SSH berkaitan dengan...

A. Pertukaran kunci sesi antara client dan server
B. Rotasi log
C. Akses USB
D. Package update

**Kunci:** A

**Pembahasan:** Key exchange adalah fondasi sesi terenkripsi.


## Soal 153

Apa fungsi ClientAliveInterval?

A. Mendeteksi/memutus sesi SSH idle setelah periode tertentu
B. Mengizinkan password kosong
C. Membuka XDMCP
D. Menghapus host key

**Kunci:** A

**Pembahasan:** Idle session dapat ditinggalkan dan disalahgunakan.


## Soal 154

Mengapa DisableForwarding penting?

A. Mencegah SSH dipakai sebagai tunnel/forwarding yang tidak diawasi
B. Agar SSH tidak bisa login sama sekali
C. Agar semua traffic keluar bebas
D. Agar cron aktif

**Kunci:** A

**Pembahasan:** Forwarding dapat menjadi jalur bypass kontrol jaringan.


## Soal 155

Mengapa GSSAPIAuthentication biasanya dimatikan jika tidak dipakai?

A. Mengurangi mekanisme autentikasi tambahan yang tidak diperlukan
B. Karena GSSAPI selalu wajib
C. Karena GSSAPI adalah firewall
D. Karena GSSAPI menyimpan log audit

**Kunci:** A

**Pembahasan:** Least functionality berlaku pada metode autentikasi.


## Soal 156

Mengapa HostbasedAuthentication berisiko?

A. Kepercayaan pada host asal dapat disalahgunakan jika host tersebut kompromi
B. Karena selalu lebih kuat dari MFA
C. Karena tidak terkait SSH
D. Karena membuat log aman

**Kunci:** A

**Pembahasan:** Host-based trust sulit diaudit dan dapat menjadi jalur bypass.


## Soal 157

Mengapa IgnoreRhosts perlu diaktifkan?

A. Agar SSH mengabaikan mekanisme trust lama seperti .rhosts/.shosts
B. Agar semua host dipercaya
C. Agar password root kosong
D. Agar firewall dimatikan

**Kunci:** A

**Pembahasan:** rhosts adalah model trust lama yang tidak sesuai hardening modern.


## Soal 158

Apa tujuan LoginGraceTime?

A. Membatasi waktu koneksi SSH boleh berada dalam keadaan belum login
B. Membuat sesi SSH tidak pernah timeout
C. Mengaktifkan FTP
D. Mengatur DNS

**Kunci:** A

**Pembahasan:** Koneksi setengah terbuka tidak boleh dibiarkan terlalu lama.


## Soal 159

Apa fungsi MaxAuthTries?

A. Membatasi jumlah percobaan autentikasi per koneksi SSH
B. Membatasi jumlah file log
C. Membatasi jumlah repository
D. Membatasi jumlah partisi

**Kunci:** A

**Pembahasan:** Membantu mengurangi ruang brute force.


## Soal 160

Apa fungsi MaxStartups?

A. Membatasi koneksi SSH yang belum selesai autentikasi
B. Membatasi user home
C. Mengatur bootloader
D. Mengatur AppArmor

**Kunci:** A

**Pembahasan:** Melindungi SSH dari banyak koneksi unauthenticated.


## Soal 161

Apa fungsi MaxSessions?

A. Membatasi jumlah sesi dalam satu koneksi SSH
B. Menghapus semua sesi login
C. Mengatur systemd-journald
D. Membuat user root

**Kunci:** A

**Pembahasan:** Mengendalikan resource dan penyalahgunaan sesi.


## Soal 162

Mengapa PermitEmptyPasswords harus no?

A. Akun tanpa password tidak boleh login remote
B. Agar password kosong diizinkan
C. Karena password kosong lebih kuat
D. Karena SSH hanya menerima user anonymous

**Kunci:** A

**Pembahasan:** Password kosong adalah risiko kritis.


## Soal 163

Apa risiko PermitUserEnvironment aktif?

A. User dapat memengaruhi environment variable seperti PATH dan perilaku sesi
B. User tidak bisa login
C. SSH otomatis lebih aman
D. Firewall mati

**Kunci:** A

**Pembahasan:** Environment dapat dimanipulasi untuk memengaruhi eksekusi program.


## Soal 164

Mengapa UsePAM penting pada SSH?

A. Agar kebijakan PAM seperti account/session/authentication checks berlaku pada SSH
B. Agar PAM dilewati
C. Agar SSH tidak perlu autentikasi
D. Agar user list muncul

**Kunci:** A

**Pembahasan:** UsePAM membantu konsistensi kontrol autentikasi.


## Soal 165

Apa makna post-quantum key exchange?

A. Kesiapan algoritma pertukaran kunci terhadap risiko komputasi kuantum masa depan
B. Membuat password dua kata
C. Menghapus TLS
D. Mengaktifkan FTP

**Kunci:** A

**Pembahasan:** Ini bagian dari crypto agility dan keamanan jangka panjang.


## Soal 166

Mengapa ListenAddress perlu dikontrol?

A. Agar SSH hanya mendengarkan pada interface/IP administrasi yang diperlukan
B. Agar SSH terbuka di semua jaringan
C. Agar semua port web tertutup
D. Agar root bisa login

**Kunci:** A

**Pembahasan:** Least exposure membatasi permukaan akses SSH.


## Soal 167

Apa itu privilege escalation?

A. Proses memperoleh hak lebih tinggi, misalnya user biasa menjalankan perintah sebagai root
B. Proses menghapus log
C. Proses membuka port
D. Proses memasang wallpaper

**Kunci:** A

**Pembahasan:** Privilege escalation diperlukan tetapi harus dikontrol.


## Soal 168

Mengapa sudo lebih baik daripada login root bersama?

A. Sudo dapat mencatat user asal dan mendukung least privilege
B. Sudo tidak perlu password sama sekali
C. Sudo selalu membuka semua perintah untuk semua user
D. Sudo menghapus akuntabilitas

**Kunci:** A

**Pembahasan:** Sudo memungkinkan audit trail per-user.


## Soal 169

Apa tujuan requiretty/use_pty pada sudo?

A. Membuat eksekusi sudo lebih terkontrol dan membantu logging
B. Membuat sudo tidak bisa berjalan
C. Menghapus password
D. Mengubah DNS

**Kunci:** A

**Pembahasan:** PTY membantu keamanan input/output dan auditabilitas.


## Soal 170

Mengapa sudo log file penting?

A. Mencatat aktivitas eskalasi privilege untuk audit dan investigasi
B. Menyimpan file ISO
C. Membuat banner login
D. Membaca filesystem

**Kunci:** A

**Pembahasan:** Log sudo meningkatkan akuntabilitas.


## Soal 171

Apa risiko sudo NOPASSWD terlalu luas?

A. Kompromi akun user langsung dapat menjalankan perintah root tanpa autentikasi ulang
B. User tidak bisa eskalasi
C. Audit menjadi sempurna
D. Sistem otomatis terkunci

**Kunci:** A

**Pembahasan:** Password untuk eskalasi memberi lapisan verifikasi tambahan.


## Soal 172

Apa fungsi timestamp_timeout pada sudo?

A. Mengatur berapa lama sudo mengingat autentikasi sebelum meminta password ulang
B. Mengatur jam sistem
C. Menghapus log
D. Membuka port

**Kunci:** A

**Pembahasan:** Timeout terlalu panjang meningkatkan risiko sesi privileged ditinggalkan.


## Soal 173

Mengapa su dibatasi ke group tertentu?

A. Agar tidak semua user dapat mencoba berpindah ke root/user lain
B. Agar semua user bisa su tanpa password
C. Agar root hilang
D. Agar SSH mati

**Kunci:** A

**Pembahasan:** Pembatasan su mengurangi jalur eskalasi dan brute force root.


## Soal 174

Apa itu PAM?

A. Framework autentikasi modular Linux
B. Firewall paket
C. Filesystem kernel
D. Tool backup

**Kunci:** A

**Pembahasan:** PAM mengatur autentikasi, account, password, dan session berbagai service.


## Soal 175

Apa fungsi pam_faillock?

A. Mengunci/menunda akun setelah gagal login berulang sesuai kebijakan
B. Mengganti DNS
C. Menghapus password
D. Menjalankan cron

**Kunci:** A

**Pembahasan:** Lockout gagal login mengurangi brute force.


## Soal 176

Apa fungsi pam_pwquality?

A. Menerapkan kebijakan kualitas/kompleksitas password
B. Membuka SSH root
C. Menghapus log
D. Mengatur firewall

**Kunci:** A

**Pembahasan:** Password quality membantu mencegah password lemah.


## Soal 177

Apa fungsi pam_pwhistory?

A. Mencegah penggunaan ulang password lama
B. Menonaktifkan auditd
C. Membuka Telnet
D. Mengatur IPv6

**Kunci:** A

**Pembahasan:** Password history mengurangi reuse password.


## Soal 178

Mengapa opsi nullok berbahaya?

A. Dapat mengizinkan password kosong pada PAM tertentu
B. Karena memaksa MFA
C. Karena membuat password lebih panjang
D. Karena mengaktifkan AppArmor

**Kunci:** A

**Pembahasan:** Password kosong tidak boleh diizinkan.


## Soal 179

Apa fungsi use_authtok dalam konteks PAM password?

A. Memastikan modul memakai token/password yang sudah diberikan sehingga alur perubahan password konsisten
B. Menghapus password history
C. Mengaktifkan telnet
D. Mengubah hostname

**Kunci:** A

**Pembahasan:** use_authtok membantu konsistensi antar modul password.


## Soal 180

Apa itu password expiration?

A. Kebijakan masa berlaku password sebelum harus diganti
B. Penghapusan semua akun
C. Pembukaan firewall
D. Mode GUI

**Kunci:** A

**Pembahasan:** Password aging mengurangi risiko kredensial lama dipakai terus tanpa review.


## Soal 181

Mengapa minimum password age penting?

A. Mencegah user mengganti password berkali-kali untuk melewati password history
B. Agar user tidak pernah ganti password
C. Agar password selalu kosong
D. Agar user menjadi root

**Kunci:** A

**Pembahasan:** Minimum age mencegah bypass history.


## Soal 182

Mengapa UID 0 harus unik untuk root?

A. UID 0 memiliki hak superuser; akun lain dengan UID 0 setara root
B. UID 0 hanya untuk tamu
C. UID 0 tidak punya privilege
D. UID 0 adalah group biasa

**Kunci:** A

**Pembahasan:** Akun UID 0 tambahan adalah risiko kritis.


## Soal 183

Mengapa root PATH integrity penting?

A. PATH root yang mengarah ke direktori writable dapat membuat root menjalankan binary palsu
B. PATH root hanya untuk warna shell
C. PATH root tidak memengaruhi keamanan
D. PATH root menghapus SSH

**Kunci:** A

**Pembahasan:** PATH root harus bebas dari direktori tidak aman/kosong relatif.


## Soal 184

Mengapa akun sistem sebaiknya memakai shell non-login?

A. Akun service tidak seharusnya dipakai login interaktif
B. Akun sistem harus selalu bisa SSH
C. Akun sistem wajib punya bash
D. Akun sistem harus UID 0

**Kunci:** A

**Pembahasan:** Locked non-login accounts membatasi penyalahgunaan service account.


## Soal 185

Apa fungsi default umask?

A. Menentukan permission default file/direktori baru
B. Mengatur DNS
C. Menghapus user
D. Menyimpan password

**Kunci:** A

**Pembahasan:** Umask yang terlalu longgar dapat membuat file user mudah dibaca.


# G. Bab 6 — Logging and Auditing


## Soal 186

Apa fungsi utama logging dan auditing?

A. Memberi visibility, accountability, dan bukti untuk monitoring, audit, dan forensik
B. Menghapus semua data lama
C. Membuka port
D. Mengganti antivirus

**Kunci:** A

**Pembahasan:** Logging/auditing memungkinkan analisis kejadian keamanan.


## Soal 187

Perbedaan logging biasa dan auditing adalah...

A. Logging umum mencatat aktivitas operasional; auditing fokus pada aktivitas keamanan/kepatuhan yang sensitif
B. Auditing tidak mencatat apa pun
C. Logging selalu lebih formal dari auditd
D. Keduanya tidak berbeda

**Kunci:** A

**Pembahasan:** Keduanya saling melengkapi.


## Soal 188

Apa fungsi systemd-journald?

A. Mengumpulkan log dari kernel, service, stdout/stderr, dan metadata systemd
B. Membuka firewall
C. Membuat user
D. Mengatur DNS

**Kunci:** A

**Pembahasan:** journald adalah komponen logging systemd.


## Soal 189

Mengapa persistensi log journald penting?

A. Agar log tetap tersedia setelah reboot
B. Agar log hilang setelah reboot
C. Agar log bisa ditulis semua user
D. Agar auditd mati

**Kunci:** A

**Pembahasan:** Log volatile hilang saat reboot dan merugikan investigasi.


## Soal 190

Mengapa forwarding journald ke rsyslog berguna?

A. Agar log bisa dikelola sebagai file log tradisional atau diteruskan ke server pusat
B. Agar log dihapus
C. Agar SSH terbuka
D. Agar /tmp dieksekusi

**Kunci:** A

**Pembahasan:** rsyslog kuat untuk file log dan remote forwarding.


## Soal 191

Mengapa permission log journald harus dibatasi?

A. Log dapat berisi informasi sensitif dan bernilai bukti
B. Karena log harus bisa diedit semua user
C. Karena log tidak pernah sensitif
D. Karena log hanya gambar

**Kunci:** A

**Pembahasan:** Least privilege berlaku pada akses log.


## Soal 192

Apa tujuan log rotation?

A. Mencegah log memenuhi disk dan menjaga retensi terkelola
B. Menghapus semua log setiap menit
C. Membuka port baru
D. Mengatur PAM

**Kunci:** A

**Pembahasan:** Rotasi menjaga availability dan auditability.


## Soal 193

Apa fungsi compression log?

A. Menghemat storage untuk log lama
B. Membuat log tidak perlu permission
C. Menghapus bukti
D. Membuka auditd

**Kunci:** A

**Pembahasan:** Kompresi mendukung retensi lebih lama tetapi bukan pengganti security.


## Soal 194

Apa fungsi rsyslog?

A. Menerima, memproses, menyimpan, dan meneruskan log
B. Mengatur SSH cipher
C. Mengatur AppArmor profile
D. Memformat disk

**Kunci:** A

**Pembahasan:** rsyslog adalah sistem logging fleksibel.


## Soal 195

Mengapa mode file log penting?

A. Agar file log baru dibuat dengan permission aman sejak awal
B. Agar log bisa ditulis siapa saja
C. Agar log menjadi executable
D. Agar firewall terbuka

**Kunci:** A

**Pembahasan:** Secure by default mencegah log baru terlalu terbuka.


## Soal 196

Mengapa remote logging penting?

A. Log tetap tersedia di server pusat meskipun host sumber dikompromikan
B. Agar log hanya lokal
C. Agar auditd dihapus
D. Agar semua log bisa diedit attacker

**Kunci:** A

**Pembahasan:** Remote logging memperkuat forensik dan SIEM.


## Soal 197

Mengapa TLS untuk forwarding log penting?

A. Melindungi confidentiality, integrity, dan authentication saat log dikirim lewat jaringan
B. Agar log tidak tersimpan
C. Agar password kosong
D. Agar semua port diterima

**Kunci:** A

**Pembahasan:** Log berisi data sensitif dan harus dilindungi saat transit.


## Soal 198

Apa peran CA certificate pada TLS rsyslog?

A. Memverifikasi identitas endpoint/server log yang dipercaya
B. Menghapus key SSH
C. Mengatur password age
D. Menjalankan cron

**Kunci:** A

**Pembahasan:** TLS tanpa validasi identitas lemah terhadap MITM.


## Soal 199

Mengapa host biasa tidak perlu menerima log dari client lain?

A. Karena membuka fungsi server/port tambahan tanpa kebutuhan
B. Karena semua host wajib menjadi log server
C. Karena menerima log selalu gratis risiko
D. Karena log client tidak dapat menyerang

**Kunci:** A

**Pembahasan:** Jika bukan log server, jangan buka penerimaan log remote.


## Soal 200

Mengapa logfile harus dilindungi dari user biasa?

A. User biasa dapat membaca informasi sensitif atau memanipulasi jejak
B. Karena log tidak penting
C. Karena semua user harus edit log
D. Karena log adalah wallpaper

**Kunci:** A

**Pembahasan:** Log adalah aset keamanan dan bukti.


## Soal 201

Apa risiko manipulasi log?

A. Investigasi dan akuntabilitas menjadi lemah karena jejak dapat dihapus/dipalsukan
B. Sistem otomatis lebih aman
C. Audit makin akurat
D. Password menjadi kompleks

**Kunci:** A

**Pembahasan:** Attacker sering mencoba menghapus jejak.


## Soal 202

Apa itu auditd?

A. Daemon yang menerima event audit kernel dan menyimpannya sebagai audit log
B. Web server
C. Package manager
D. Client FTP

**Kunci:** A

**Pembahasan:** auditd mendukung audit keamanan formal.


## Soal 203

Mengapa proses sebelum auditd aktif perlu diaudit?

A. Agar tidak ada blind spot pada fase boot awal
B. Agar boot lebih lambat saja
C. Agar semua log volatile
D. Agar SSH mati

**Kunci:** A

**Pembahasan:** Aktivitas awal boot juga dapat bernilai keamanan.


## Soal 204

Apa fungsi audit_backlog_limit?

A. Menentukan kapasitas antrean event audit saat beban tinggi
B. Membatasi user login
C. Mengatur firewall
D. Membuat kernel module

**Kunci:** A

**Pembahasan:** Backlog membantu mencegah kehilangan event audit.


## Soal 205

Mengapa ukuran audit log perlu diatur?

A. Agar retensi cukup tanpa memenuhi storage secara tak terkendali
B. Agar audit log selalu kosong
C. Agar semua user bisa edit audit log
D. Agar SSH mati

**Kunci:** A

**Pembahasan:** Audit log yang terlalu kecil atau tak dikelola dapat menghilangkan bukti/memenuhi disk.


## Soal 206

Mengapa audit log tidak boleh otomatis dihapus tanpa kebijakan?

A. Karena audit log adalah bukti insiden/kepatuhan
B. Karena log tidak pernah berguna
C. Karena penghapusan selalu aman
D. Karena log hanya cache

**Kunci:** A

**Pembahasan:** Retensi audit harus jelas dan terdokumentasi.


## Soal 207

Apa dilema saat audit log penuh?

A. Menjaga availability vs memastikan sistem tidak berjalan tanpa audit
B. Memilih warna terminal
C. Menghapus semua user
D. Mengaktifkan Telnet

**Kunci:** A

**Pembahasan:** Sistem sensitif mungkin harus berhenti/masuk single-user jika audit tidak bisa mencatat.


## Soal 208

Mengapa warning storage audit rendah penting?

A. Memberi deteksi dini sebelum audit log penuh/hilang
B. Agar storage langsung dihapus
C. Agar log tidak perlu disimpan
D. Agar user bisa edit audit log

**Kunci:** A

**Pembahasan:** Peringatan membantu tindakan proaktif.


## Soal 209

Apa fungsi audit rule?

A. Menentukan kejadian keamanan apa yang harus dicatat auditd
B. Menentukan tema desktop
C. Mengatur DNS resolver
D. Menghapus kernel

**Kunci:** A

**Pembahasan:** Tanpa rule tepat, auditd aktif tetapi tidak mencatat hal penting.


## Soal 210

Mengapa sudoers perlu diaudit?

A. Perubahan sudoers dapat memberi akses privilege tinggi
B. Karena sudoers menyimpan gambar
C. Karena sudoers tidak terkait privilege
D. Karena sudoers hanya milik user biasa

**Kunci:** A

**Pembahasan:** sudoers adalah jalur utama privilege escalation.


## Soal 211

Mengapa tindakan sebagai user lain perlu diaudit?

A. Agar aktivitas root/sudo dapat ditelusuri ke user asal
B. Agar semua aksi anonim
C. Agar root login langsung dibuka
D. Agar cron mati

**Kunci:** A

**Pembahasan:** Audit meningkatkan non-repudiation.


## Soal 212

Mengapa perubahan waktu sistem diaudit?

A. Timestamp log dan audit bergantung pada waktu yang benar
B. Karena waktu tidak penting
C. Karena waktu hanya dekorasi
D. Karena waktu menghapus log

**Kunci:** A

**Pembahasan:** Perubahan waktu dapat mengacaukan kronologi insiden.


## Soal 213

Mengapa perubahan hostname/domain/network config diaudit?

A. Karena perubahan identitas/jaringan dapat memengaruhi konektivitas dan forensik
B. Karena hostname hanya hiasan
C. Karena network config tidak sensitif
D. Karena auditd tidak bisa mencatat file

**Kunci:** A

**Pembahasan:** Konfigurasi jaringan adalah kontrol keamanan penting.


## Soal 214

Mengapa privileged commands diaudit?

A. Binary privileged dapat dipakai untuk melakukan aksi sensitif/eskalasi
B. Karena semua command privileged tidak berisiko
C. Karena command privileged hanya untuk membaca wallpaper
D. Karena auditd harus dimatikan

**Kunci:** A

**Pembahasan:** Audit privileged commands membantu mendeteksi penyalahgunaan.


## Soal 215

Mengapa file /etc/passwd, /etc/group, /etc/shadow, /etc/gshadow diaudit?

A. Perubahan file identitas lokal dapat mengubah user, group, dan autentikasi
B. Karena file ini tidak penting
C. Karena file ini hanya sementara
D. Karena file ini milik semua user untuk diedit

**Kunci:** A

**Pembahasan:** Akun dan group adalah fondasi access control.


## Soal 216

Mengapa unsuccessful file access perlu diaudit?

A. Gagal akses file bisa menandakan probing atau percobaan akses tidak sah
B. Karena akses gagal selalu normal
C. Karena audit harus hanya sukses
D. Karena file gagal tidak penting

**Kunci:** A

**Pembahasan:** Failed access memberi sinyal aktivitas mencurigakan.


## Soal 217

Mengapa perubahan permission/mount/file deletion perlu diaudit?

A. Dapat menunjukkan upaya mengubah akses, menyembunyikan data, atau persistence
B. Karena perubahan file tidak memengaruhi keamanan
C. Karena mount selalu aman
D. Karena deletion tidak bisa disalahgunakan

**Kunci:** A

**Pembahasan:** Perubahan filesystem sering bernilai forensik.


## Soal 218

Mengapa perubahan MAC seperti AppArmor/SELinux diaudit?

A. Karena dapat melemahkan pembatasan proses
B. Karena MAC hanya alamat jaringan
C. Karena MAC tidak terkait security
D. Karena auditd tidak bisa melihat konfigurasi

**Kunci:** A

**Pembahasan:** MAC policy adalah kontrol keamanan penting.


## Soal 219

Mengapa audit rule dibuat immutable?

A. Agar attacker/admin tidak mudah mengubah rule audit tanpa reboot/prosedur
B. Agar auditd tidak mencatat
C. Agar semua log bisa dihapus
D. Agar firewall terbuka

**Kunci:** A

**Pembahasan:** Immutable audit config melindungi integritas audit policy.


## Soal 220

Mengapa file/direktori audit log dan tool audit perlu permission ketat?

A. Jika bisa diubah user biasa, bukti dan konfigurasi audit tidak dapat dipercaya
B. Agar semua user bisa baca/tulis audit
C. Agar audit log menjadi executable
D. Agar auditd berhenti

**Kunci:** A

**Pembahasan:** Audit artifacts harus dilindungi sebagai evidence.


## Soal 221

Apa fungsi AIDE?

A. Filesystem integrity checking untuk mendeteksi perubahan file penting
B. Firewall host
C. SSH cipher
D. Package manager

**Kunci:** A

**Pembahasan:** AIDE membuat baseline integritas dan membandingkan perubahan.


## Soal 222

Mengapa audit tools perlu dilindungi integritasnya?

A. Jika tool audit dimodifikasi, hasil audit bisa dipalsukan
B. Karena tool audit tidak penting
C. Karena tool audit harus bisa diedit user
D. Karena audit tools adalah GUI

**Kunci:** A

**Pembahasan:** Integrity checking terhadap audit tools memperkuat kepercayaan hasil audit.


# H. Bab 7 — System Maintenance


## Soal 223

Apa fokus utama Bab 7 System Maintenance?

A. Menjaga permission file penting, konsistensi user/group, dan kondisi filesystem tetap aman
B. Mengatur wallpaper
C. Membuka semua service
D. Menghapus firewall

**Kunci:** A

**Pembahasan:** Bab 7 adalah maintenance berkala terhadap file dan identitas sistem.


## Soal 224

Mengapa /etc/passwd harus dilindungi?

A. Berisi informasi akun; jika writable user biasa dapat mengubah identitas/UID/shell
B. Karena menyimpan password plaintext
C. Karena berisi log audit
D. Karena file sementara

**Kunci:** A

**Pembahasan:** /etc/passwd biasanya readable tetapi tidak boleh writable user biasa.


## Soal 225

Mengapa /etc/passwd- juga harus diamankan?

A. File backup tetap berisi informasi akun dan dapat membantu enumerasi/perbandingan perubahan
B. Karena backup tidak pernah sensitif
C. Karena hanya milik user biasa
D. Karena isinya kosong

**Kunci:** A

**Pembahasan:** Backup file identitas harus diproteksi seperti file utama.


## Soal 226

Apa fungsi /etc/group?

A. Menyimpan daftar group lokal dan anggotanya
B. Menyimpan audit log
C. Menyimpan firewall rule
D. Menyimpan certificate TLS

**Kunci:** A

**Pembahasan:** /etc/group memengaruhi otorisasi berbasis group.


## Soal 227

Apa risiko /etc/group writable oleh user biasa?

A. User bisa menambahkan dirinya ke group dengan akses lebih tinggi
B. User tidak bisa login
C. Semua log hilang
D. Firewall mati

**Kunci:** A

**Pembahasan:** Group membership menentukan banyak hak akses.


## Soal 228

Mengapa /etc/shadow sangat sensitif?

A. Menyimpan password hash dan informasi aging akun
B. Menyimpan hostname
C. Menyimpan service list
D. Menyimpan file temporary

**Kunci:** A

**Pembahasan:** Hash password dapat dicrack offline bila bocor.


## Soal 229

Mengapa /etc/shadow- dapat lebih berbahaya dari yang terlihat?

A. Bisa menyimpan hash password lama yang mungkin lebih lemah
B. Karena selalu kosong
C. Karena tidak pernah ada
D. Karena hanya berisi log

**Kunci:** A

**Pembahasan:** Backup shadow tetap rahasia autentikasi.


## Soal 230

Apa fungsi /etc/gshadow?

A. Menyimpan informasi sensitif group seperti admin/anggota group dan password group bila ada
B. Menyimpan DNS zone
C. Menyimpan config UFW
D. Menyimpan kernel module

**Kunci:** A

**Pembahasan:** /etc/gshadow adalah bagian sensitif kontrol group.


## Soal 231

Apa fungsi /etc/shells?

A. Daftar shell login valid
B. Daftar firewall port
C. Daftar audit rules
D. Daftar package repository

**Kunci:** A

**Pembahasan:** Jika dimanipulasi, shell tidak sah bisa dianggap valid.


## Soal 232

Apa risiko /etc/security/opasswd terbaca user biasa?

A. Hash/riwayat password lama dapat dianalisis atau dicrack
B. Semua user otomatis lock
C. Firewall terbuka
D. AIDE mati

**Kunci:** A

**Pembahasan:** Password history adalah data autentikasi sensitif.


## Soal 233

Apa itu world writable file?

A. File yang dapat ditulis oleh semua user
B. File yang hanya root bisa tulis
C. File tanpa owner
D. File immutable

**Kunci:** A

**Pembahasan:** World writable file berbahaya karena integritasnya tidak terjaga.


## Soal 234

Mengapa world writable directory tanpa sticky bit berbahaya?

A. User dapat menghapus/mengganti file milik user lain
B. Semua user otomatis terlindungi
C. Direktori jadi read-only
D. Log menjadi terenkripsi

**Kunci:** A

**Pembahasan:** Sticky bit diperlukan pada direktori bersama seperti /tmp.


## Soal 235

Apa risiko file tanpa owner/group?

A. UID/GID lama bisa dipakai ulang dan user baru mewarisi akses file lama
B. File otomatis lebih aman
C. File tidak bisa dibaca siapa pun
D. File menjadi audit rule

**Kunci:** A

**Pembahasan:** No owner/group menunjukkan inkonsistensi identity/filesystem.


## Soal 236

Apa itu SUID?

A. Bit permission yang membuat executable berjalan dengan hak owner file
B. File log audit
C. Network protocol
D. Shell default

**Kunci:** A

**Pembahasan:** SUID root dapat menjadi jalur privilege escalation.


## Soal 237

Apa itu SGID?

A. Bit permission yang membuat executable/direktori memakai hak group tertentu
B. Sistem DNS
C. Firewall rule
D. Paket update

**Kunci:** A

**Pembahasan:** SGID perlu direview seperti SUID.


## Soal 238

Mengapa review SUID/SGID tidak sekadar menghapus semua?

A. Beberapa binary sah memang membutuhkan SUID/SGID untuk fungsi tertentu
B. Karena semua SUID pasti malware
C. Karena SUID tidak punya risiko
D. Karena SGID hanya untuk GUI

**Kunci:** A

**Pembahasan:** Review perlu membedakan legitimate binary dan temuan aneh.


## Soal 239

Apa itu shadowed passwords?

A. Password hash disimpan di /etc/shadow, bukan di /etc/passwd
B. Password disimpan di /etc/group
C. Password dikirim lewat FTP
D. Password selalu kosong

**Kunci:** A

**Pembahasan:** Shadowed passwords memisahkan identitas umum dari rahasia autentikasi.


## Soal 240

Mengapa empty password field berbahaya?

A. Akun berpotensi login tanpa password
B. Akun otomatis terkunci selamanya
C. Password menjadi sangat kompleks
D. PAM otomatis mati

**Kunci:** A

**Pembahasan:** Akun tanpa password adalah temuan kritis.


## Soal 241

Apa maksud konsistensi group dalam Bab 7?

A. GID utama user di /etc/passwd harus ada di /etc/group
B. Semua user harus punya GID 0
C. Semua group harus sama namanya
D. Tidak boleh ada group sama sekali

**Kunci:** A

**Pembahasan:** Inkonsistensi GID melemahkan identitas dan permission.


## Soal 242

Mengapa group shadow tidak boleh berisi user tidak semestinya?

A. Akses ke group shadow dapat memberi kemampuan membaca data password hash
B. Karena shadow group untuk semua user
C. Karena shadow group hanya untuk printer
D. Karena shadow group membuat firewall

**Kunci:** A

**Pembahasan:** Group shadow berkaitan dengan akses file shadow.


## Soal 243

Apa risiko duplicate UID?

A. Dua username dapat merujuk ke identitas kernel yang sama sehingga akuntabilitas rusak
B. Sistem otomatis lebih aman
C. Password menjadi unik
D. Tidak ada dampak

**Kunci:** A

**Pembahasan:** Kernel mengenali user berdasarkan UID, bukan nama.


## Soal 244

Apa risiko duplicate GID?

A. Beberapa group berbeda dapat berbagi identitas numerik yang sama
B. Semua group hilang
C. Akses menjadi lebih ketat otomatis
D. Tidak ada risiko

**Kunci:** A

**Pembahasan:** GID ganda membingungkan kontrol akses group.


## Soal 245

Apa risiko duplicate username?

A. Identitas manusia dan tool administrasi menjadi ambigu
B. Sistem menjadi lebih cepat
C. Password tidak diperlukan
D. Firewall aktif otomatis

**Kunci:** A

**Pembahasan:** Username harus unik agar administrasi/audit jelas.


## Soal 246

Mengapa home directory user interaktif perlu diperiksa?

A. Harus ada, dimiliki user yang benar, dan tidak terlalu terbuka
B. Agar semua user dapat menulis home user lain
C. Agar user tidak punya file
D. Agar SSH mati

**Kunci:** A

**Pembahasan:** Home directory berisi data dan konfigurasi user.


## Soal 247

Mengapa dot files user interaktif berisiko?

A. File seperti .rhosts, .netrc, atau permission terlalu longgar dapat membuka akses/credential leakage
B. Dot files selalu aman
C. Dot files hanya gambar
D. Dot files tidak pernah dibaca program

**Kunci:** A

**Pembahasan:** Dot files dapat memengaruhi shell, login, trust, dan kredensial.


# I. Bonus — Implementasi Arch Linux Berbasis Prinsip CIS Debian


## Soal 248

Mengapa panduan Arch berbasis CIS Debian disebut adaptasi, bukan penerapan resmi?

A. Karena CIS Debian dibuat untuk Debian 12, sedangkan Arch punya ekosistem paket dan konfigurasi berbeda
B. Karena Arch bukan Linux
C. Karena CIS tidak punya konsep hardening
D. Karena Debian tidak bisa diinstal

**Kunci:** A

**Pembahasan:** Prinsip dapat diadaptasi, tetapi command/path/package berbeda.


## Soal 249

Apa perbedaan package management Debian dan Arch dalam hardening?

A. Debian memakai apt/dpkg; Arch memakai pacman/pacman-key
B. Keduanya selalu identik
C. Arch memakai apt secara default
D. Debian memakai pacman secara default

**Kunci:** A

**Pembahasan:** Penerjemahan implementasi harus mengikuti distro target.


## Soal 250

Mengapa LUKS2 + LVM dipakai dalam dokumentasi instalasi?

A. Untuk enkripsi disk dan fleksibilitas pemisahan logical volume
B. Untuk mematikan firewall
C. Untuk menghapus /home
D. Untuk mengganti SSH

**Kunci:** A

**Pembahasan:** LUKS memberi confidentiality; LVM membantu partisi hardening.


## Soal 251

Mengapa /var/lib/docker atau data container sebaiknya dipartisi terpisah pada container host?

A. Data container bisa tumbuh cepat dan memenuhi root filesystem
B. Karena Docker tidak menulis file
C. Karena container tidak butuh storage
D. Karena auditd tidak mencatat Docker

**Kunci:** A

**Pembahasan:** CIS Docker juga menekankan partisi terpisah untuk data container.


## Soal 252

Mengapa linux-hardened dipilih dalam contoh Arch?

A. Sebagai kernel dengan konfigurasi hardening tambahan dibanding kernel umum
B. Karena wajib di semua CIS resmi
C. Karena menggantikan AppArmor
D. Karena membuat user root hilang

**Kunci:** A

**Pembahasan:** Ini pilihan implementasi Arch, bukan instruksi literal CIS Debian.


## Soal 253

Mengapa tetap memasang kernel cadangan seperti linux-lts?

A. Sebagai opsi recovery jika kernel utama bermasalah
B. Agar semua modul aktif
C. Agar firewall mati
D. Agar audit log dihapus

**Kunci:** A

**Pembahasan:** Hardening juga harus mempertimbangkan recoverability.


## Soal 254

Mengapa GRUB diberi password?

A. Untuk membatasi modifikasi parameter boot/recovery oleh pihak tidak berwenang
B. Agar user tidak bisa login normal
C. Agar disk tidak terenkripsi
D. Agar SSH terbuka

**Kunci:** A

**Pembahasan:** Bootloader dapat menjadi jalur bypass jika fisik diakses.


## Soal 255

Apa fungsi kernel parameter audit=1?

A. Mengaktifkan audit sejak boot awal
B. Menghapus auditd
C. Mengaktifkan FTP
D. Mengatur GDM

**Kunci:** A

**Pembahasan:** Audit sejak boot mengurangi blind spot.


## Soal 256

Mengapa AppArmor memerlukan kernel parameter LSM yang benar?

A. Agar AppArmor masuk urutan Linux Security Modules dan aktif
B. Agar firewall mengizinkan semua port
C. Agar user root dihapus
D. Agar cron mati

**Kunci:** A

**Pembahasan:** Pada Arch, AppArmor perlu paket, service, dan dukungan kernel parameter.


## Soal 257

Mengapa fstab perlu diedit setelah genfstab?

A. Untuk memastikan mount option seperti nodev,nosuid,noexec sesuai hardening
B. Agar semua filesystem executable
C. Agar /tmp menyatu dengan /
D. Agar semua UUID dihapus

**Kunci:** A

**Pembahasan:** genfstab tidak selalu memberi opsi hardening yang diinginkan.


## Soal 258

Apa risiko menjalankan pacman -Rns orphan tanpa review?

A. Paket yang tampak orphan bisa masih dibutuhkan manual oleh admin/aplikasi
B. Tidak ada risiko sama sekali
C. Semua paket sistem pasti aman dihapus
D. Pacman tidak bisa menghapus paket

**Kunci:** A

**Pembahasan:** Service minimization perlu review kebutuhan.


## Soal 259

Mengapa overlay module menjadi exception pada host Docker/Podman?

A. Container runtime sering membutuhkan OverlayFS untuk storage layer
B. Overlay selalu malware
C. Overlay adalah SSH cipher
D. Overlay menyimpan user password

**Kunci:** A

**Pembahasan:** Menonaktifkan overlay bisa membuat container gagal berjalan.


## Soal 260

Mengapa usb-storage tidak langsung dimatikan jika backup offline dibutuhkan?

A. Karena kebutuhan bisnis/operasional dapat menjadi exception terdokumentasi
B. Karena usb-storage wajib untuk AppArmor
C. Karena usb-storage selalu aman
D. Karena backup tidak perlu media

**Kunci:** A

**Pembahasan:** Exception harus disertai mitigasi.


## Soal 261

Apa tujuan sysctl hardening pada Arch adaptasi CIS?

A. Mengatur perilaku kernel seperti hardlink/symlink, ptrace, dmesg, ASLR, IPv4/IPv6
B. Mengubah tema shell
C. Menghapus /etc/passwd
D. Mengaktifkan telnet

**Kunci:** A

**Pembahasan:** Sysctl mengontrol parameter kernel runtime/persistent.


## Soal 262

Mengapa rfkill Bluetooth/Wi-Fi relevan untuk server?

A. Untuk menonaktifkan perangkat jaringan nirkabel yang tidak dibutuhkan
B. Untuk mengatur password root
C. Untuk membuat audit rule
D. Untuk mengubah repository

**Kunci:** A

**Pembahasan:** Ini adaptasi prinsip network devices.


## Soal 263

Mengapa UFW rule SSH sebaiknya dibatasi ke IP admin jika memungkinkan?

A. Untuk mengurangi exposure SSH dari seluruh jaringan
B. Agar semua orang bisa SSH
C. Agar SSH tidak perlu password
D. Agar log dihapus

**Kunci:** A

**Pembahasan:** Allow from sumber admin menerapkan least exposure.


## Soal 264

Mengapa file ssh_host_*_key harus permission ketat?

A. Private host key adalah rahasia identitas server
B. Karena public key tidak boleh dibaca siapa pun
C. Karena key adalah log
D. Karena key mengatur firewall

**Kunci:** A

**Pembahasan:** Private key bocor dapat melemahkan kepercayaan SSH.


## Soal 265

Mengapa sudoers.d perlu divalidasi dengan visudo -cf?

A. Untuk mencegah kesalahan sintaks yang dapat merusak akses sudo
B. Untuk menghapus semua user
C. Untuk membuka telnet
D. Untuk menonaktifkan AppArmor

**Kunci:** A

**Pembahasan:** Kesalahan sudoers bisa menyebabkan lockout privilege.


## Soal 266

Mengapa audit rules Docker/host perlu diterapkan bila Docker digunakan?

A. Docker daemon dan socket memberi akses sangat tinggi dan perlu accountability
B. Karena Docker tidak punya risiko
C. Karena Docker selalu non-root
D. Karena Docker mengganti auditd

**Kunci:** A

**Pembahasan:** Docker control plane setara privilege tinggi.


## Soal 267

Mengapa AIDE database harus diinisialisasi setelah baseline aman?

A. Agar baseline integritas mencerminkan kondisi sistem yang sudah dipercaya
B. Agar AIDE mengabaikan semua perubahan
C. Agar AIDE membuka port
D. Agar AIDE mengganti password

**Kunci:** A

**Pembahasan:** Integrity checking membutuhkan baseline yang benar.


## Soal 268

Mengapa exception register penting dalam implementasi?

A. Agar penyimpangan dari baseline dapat diaudit dan memiliki mitigasi
B. Agar semua rekomendasi diabaikan
C. Agar hardening tidak perlu dokumentasi
D. Agar auditor tidak melihat sistem

**Kunci:** A

**Pembahasan:** Exception bukan kelalaian jika terdokumentasi dan disetujui.


## Soal 269

Mengapa dokumentasi implementasi harus memiliki checklist validasi?

A. Agar admin dapat membuktikan konfigurasi akhir sesuai target
B. Agar semua command dilupakan
C. Agar log dihapus
D. Agar user tidak bisa audit

**Kunci:** A

**Pembahasan:** Checklist memudahkan audit dan post-install verification.


## Soal 270

Apa prinsip utama adaptasi CIS ke distro lain?

A. Pertahankan konsep risiko, tetapi sesuaikan command, package, service, dan path dengan distro target
B. Salin semua command Debian mentah-mentah
C. Abaikan semua rekomendasi
D. Gunakan benchmark versi acak

**Kunci:** A

**Pembahasan:** Konsep sama, implementasi berbeda.


## Soal 271

Mengapa hardening pasca-instalasi harus dilakukan bertahap?

A. Untuk menguji dampak, mencegah lockout, dan memudahkan rollback
B. Agar sistem tidak pernah aman
C. Agar semua user bisa root
D. Agar audit log penuh

**Kunci:** A

**Pembahasan:** Pendekatan bertahap adalah praktik aman.


## Soal 272

Apa hubungan dokumen implementasi Arch dengan CIS Debian konseptual?

A. Ia menerjemahkan prinsip CIS Debian ke mekanisme Arch, bukan menggantikan benchmark resmi
B. Ia membuktikan Arch sama persis dengan Debian
C. Ia menghapus kebutuhan pemahaman konsep
D. Ia membuat semua rekomendasi otomatis

**Kunci:** A

**Pembahasan:** Dokumen implementasi adalah adaptasi operasional dari konsep CIS.


---
# Rekap Kunci Jawaban Cepat


1. B | 2. B | 3. A | 4. B | 5. A | 6. B | 7. B | 8. B | 9. B | 10. B | 11. B | 12. B | 13. A | 14. A | 15. B | 16. B | 17. A | 18. A | 19. A | 20. A | 21. A | 22. A | 23. A | 24. A | 25. B


26. A | 27. A | 28. B | 29. A | 30. A | 31. B | 32. A | 33. A | 34. A | 35. A | 36. A | 37. A | 38. A | 39. A | 40. A | 41. A | 42. A | 43. A | 44. A | 45. A | 46. A | 47. A | 48. A | 49. A | 50. A


51. A | 52. A | 53. A | 54. A | 55. A | 56. A | 57. A | 58. A | 59. A | 60. A | 61. A | 62. A | 63. A | 64. A | 65. A | 66. A | 67. A | 68. A | 69. A | 70. A | 71. A | 72. A | 73. A | 74. A | 75. A


76. A | 77. A | 78. A | 79. A | 80. A | 81. A | 82. A | 83. A | 84. A | 85. A | 86. A | 87. A | 88. A | 89. A | 90. A | 91. A | 92. A | 93. A | 94. A | 95. A | 96. A | 97. A | 98. A | 99. A | 100. A


101. A | 102. A | 103. A | 104. A | 105. A | 106. A | 107. A | 108. A | 109. A | 110. A | 111. A | 112. A | 113. A | 114. A | 115. A | 116. A | 117. A | 118. A | 119. A | 120. A | 121. A | 122. A | 123. A | 124. A | 125. A


126. A | 127. A | 128. A | 129. A | 130. A | 131. A | 132. A | 133. A | 134. A | 135. A | 136. A | 137. A | 138. A | 139. A | 140. A | 141. A | 142. A | 143. A | 144. A | 145. A | 146. A | 147. A | 148. A | 149. A | 150. A


151. A | 152. A | 153. A | 154. A | 155. A | 156. A | 157. A | 158. A | 159. A | 160. A | 161. A | 162. A | 163. A | 164. A | 165. A | 166. A | 167. A | 168. A | 169. A | 170. A | 171. A | 172. A | 173. A | 174. A | 175. A


176. A | 177. A | 178. A | 179. A | 180. A | 181. A | 182. A | 183. A | 184. A | 185. A | 186. A | 187. A | 188. A | 189. A | 190. A | 191. A | 192. A | 193. A | 194. A | 195. A | 196. A | 197. A | 198. A | 199. A | 200. A


201. A | 202. A | 203. A | 204. A | 205. A | 206. A | 207. A | 208. A | 209. A | 210. A | 211. A | 212. A | 213. A | 214. A | 215. A | 216. A | 217. A | 218. A | 219. A | 220. A | 221. A | 222. A | 223. A | 224. A | 225. A


226. A | 227. A | 228. A | 229. A | 230. A | 231. A | 232. A | 233. A | 234. A | 235. A | 236. A | 237. A | 238. A | 239. A | 240. A | 241. A | 242. A | 243. A | 244. A | 245. A | 246. A | 247. A | 248. A | 249. A | 250. A


251. A | 252. A | 253. A | 254. A | 255. A | 256. A | 257. A | 258. A | 259. A | 260. A | 261. A | 262. A | 263. A | 264. A | 265. A | 266. A | 267. A | 268. A | 269. A | 270. A | 271. A | 272. A
