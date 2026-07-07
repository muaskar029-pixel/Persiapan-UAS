# Penjelasan Konseptual CIS Debian Linux 12 Benchmark
# H. Bab 7 — System Maintenance

**Sumber utama:** CIS Debian Linux 12 Benchmark v2.0.0, Bab 7 — *System Maintenance*  
**Fokus pembahasan:** konseptual, bukan panduan praktik command-by-command.  
**Tujuan:** memahami mengapa pemeliharaan file sistem, permission, user lokal, group lokal, home directory, dan dot files menjadi bagian penting dari hardening Debian Linux 12.

---

## Gambaran Umum Bab 7 — System Maintenance

Bab **System Maintenance** membahas pekerjaan pemeliharaan keamanan yang harus diperiksa secara berkala. Berbeda dengan beberapa bab sebelumnya yang banyak membahas konfigurasi service, firewall, SSH, atau audit, Bab 7 berfokus pada kondisi dasar sistem yang sering berubah seiring penggunaan harian: permission file, ownership file, user lokal, group lokal, file tanpa pemilik, file world writable, SUID/SGID, home directory user, dan file konfigurasi tersembunyi milik user.

Dalam CIS Debian Linux 12 Benchmark, bagian ini ditekankan sebagai bagian **maintenance** karena banyak rekomendasinya bukan sekadar “atur sekali lalu selesai”. Sistem Linux adalah sistem yang hidup. Administrator bisa menambah user, menghapus user, memasang aplikasi, menghapus paket, memindahkan file, melakukan restore backup, mengubah permission, atau membuat script otomatis. Semua tindakan tersebut dapat meninggalkan kondisi yang tidak aman.

Contoh kondisi yang bisa muncul setelah sistem berjalan lama:

1. file penting seperti `/etc/shadow` berubah permission menjadi terlalu terbuka;
2. file backup seperti `/etc/passwd-` atau `/etc/gshadow-` ikut terekspos;
3. ada file world writable yang bisa diubah semua user;
4. ada file tanpa owner karena user lama sudah dihapus;
5. ada binary SUID/SGID baru yang tidak dikenal;
6. ada UID atau GID ganda;
7. ada user lokal dengan field password kosong;
8. home directory user tidak dimiliki oleh user yang benar;
9. dot files seperti `.rhosts`, `.netrc`, atau `.forward` muncul dan membuka risiko akses tidak sah.

Secara konseptual, Bab 7 adalah bagian yang memastikan bahwa **identitas, kepemilikan, dan izin akses sistem tetap konsisten**. Jika bab Access Control mengatur siapa boleh masuk dan bagaimana autentikasi dilakukan, maka Bab 7 memastikan objek-objek penting di filesystem tidak memberi jalan pintas bagi user biasa atau penyerang lokal.

Bab ini dapat dipahami dalam dua kelompok besar:

- **7.1 Configure System File and Directory Access**, yaitu pengamanan file sistem penting dan kondisi filesystem yang berisiko.
- **7.2 Local User and Group Settings**, yaitu validasi konsistensi akun lokal, group lokal, home directory, dan dot files.

---

# 7.1 Configure System File and Directory Access

## Konsep Permission File Sistem Penting

Pada Linux, keamanan file sangat bergantung pada tiga hal utama:

1. **owner**, yaitu user pemilik file;
2. **group owner**, yaitu group yang terkait dengan file;
3. **mode permission**, yaitu hak baca, tulis, dan eksekusi untuk owner, group, dan others.

File sistem penting seperti `/etc/passwd`, `/etc/group`, `/etc/shadow`, dan `/etc/gshadow` adalah fondasi identitas lokal Linux. File-file tersebut menentukan siapa saja user yang ada, group apa yang ada, password hash disimpan di mana, shell login apa yang valid, dan riwayat password lama disimpan di mana.

Jika file-file ini dapat dimodifikasi oleh pihak yang tidak berwenang, maka dampaknya bisa sangat serius. Penyerang dapat membuat user baru, mengubah UID, memasukkan akun ke group tertentu, melemahkan kontrol password, atau membaca hash password untuk cracking offline.

Karena itu, prinsip utamanya adalah:

- file yang perlu dibaca oleh sistem boleh readable, tetapi tidak boleh writable oleh user biasa;
- file yang berisi rahasia seperti hash password harus sangat dibatasi;
- file backup harus dilindungi sama seriusnya dengan file utama;
- owner file identitas sistem harus dikendalikan oleh root;
- permission yang terlalu longgar harus dianggap sebagai temuan keamanan.

CIS juga memetakan bagian ini ke konsep **data access control list**, yaitu pengaturan akses berdasarkan kebutuhan. Hanya pihak yang membutuhkan akses yang seharusnya mendapat akses.

---

## `/etc/passwd`

`/etc/passwd` berisi informasi akun user lokal. File ini mencatat nama user, UID, GID utama, informasi komentar, home directory, dan shell login. Banyak program sistem membutuhkan file ini agar dapat menerjemahkan UID menjadi nama user, mengetahui home directory, atau memeriksa shell login.

Karena dibutuhkan oleh banyak utilitas, `/etc/passwd` secara normal harus dapat dibaca oleh banyak proses. Namun, yang sangat penting adalah file ini **tidak boleh dapat ditulis oleh user biasa**.

Risiko jika `/etc/passwd` dapat ditulis sembarang user:

- penyerang dapat menambah akun baru;
- penyerang dapat mengubah UID user menjadi `0` sehingga setara root;
- penyerang dapat mengubah shell akun tertentu;
- penyerang dapat memanipulasi relasi UID/GID;
- sistem kehilangan akuntabilitas karena identitas user berubah.

Secara konseptual, `/etc/passwd` boleh readable, tetapi harus tetap dikendalikan oleh root. Mode yang umum adalah `0644` dengan owner `root` dan group `root`. Artinya semua user bisa membaca, tetapi hanya root yang bisa menulis.

---

## `/etc/passwd-`

`/etc/passwd-` adalah file backup dari `/etc/passwd`. Dalam banyak kasus, administrator hanya fokus mengamankan file utama dan lupa bahwa file backup juga berisi informasi sensitif tentang struktur akun sistem.

Meskipun `/etc/passwd-` bukan file utama yang aktif digunakan untuk autentikasi harian, isinya tetap dapat memberi informasi tentang user lokal, UID, GID, home directory, dan shell. Jika file backup ini memiliki permission terlalu terbuka atau owner yang salah, penyerang dapat menggunakannya untuk memahami struktur sistem atau membandingkan perubahan akun.

Prinsip CIS: **file backup identitas harus diamankan seperti file identitas utama**. Jika `/etc/passwd` dikunci dari perubahan tidak sah, maka `/etc/passwd-` juga harus dikunci.

Risiko utama `/etc/passwd-`:

- membocorkan informasi akun lama;
- membantu enumerasi user;
- memperlihatkan perubahan akun sebelum dan sesudah modifikasi;
- menjadi indikator bahwa backup sistem tidak ikut diamankan.

---

## `/etc/group`

`/etc/group` berisi daftar group lokal dan anggota group. File ini penting karena banyak keputusan akses Linux tidak hanya bergantung pada user, tetapi juga pada group.

Contoh fungsi group:

- menentukan siapa dapat membaca file tertentu;
- menentukan siapa dapat mengakses device tertentu;
- menentukan anggota group administratif tertentu;
- mengelompokkan service account;
- mengatur akses kolaboratif ke direktori atau aplikasi.

Seperti `/etc/passwd`, file `/etc/group` perlu readable oleh sistem dan program non-privileged tertentu. Namun, file ini tidak boleh writable oleh user biasa. Jika user biasa dapat mengubah `/etc/group`, ia bisa menambahkan dirinya ke group yang lebih tinggi privilege-nya.

Contoh risiko:

- user biasa menambahkan diri ke group administratif;
- user mendapat akses ke file yang seharusnya terbatas;
- service account masuk group yang salah;
- group permission menjadi tidak dapat dipercaya.

Secara konseptual, `/etc/group` adalah peta otorisasi berbasis group. Jika peta ini bisa dimanipulasi, maka kontrol akses filesystem bisa runtuh.

---

## `/etc/group-`

`/etc/group-` adalah backup dari `/etc/group`. Seperti `/etc/passwd-`, file ini tetap penting karena berisi informasi group lokal. Walaupun bukan file aktif utama, ia masih dapat membocorkan struktur group, anggota group, dan perubahan administrasi.

File backup sering menjadi celah karena dianggap tidak penting. Padahal dalam audit keamanan, file backup yang menyimpan informasi sensitif tetap dihitung sebagai aset yang harus dilindungi.

Risiko jika `/etc/group-` tidak diamankan:

- struktur group lama dapat terbaca;
- perubahan privilege group dapat dianalisis penyerang;
- informasi anggota group dapat dipakai untuk privilege targeting;
- backup yang writable dapat menandakan praktik administrasi yang lemah.

Prinsip konseptualnya: **backup file sistem penting harus mengikuti standar perlindungan yang sama dengan file aslinya**.

---

## `/etc/shadow`

`/etc/shadow` adalah salah satu file paling sensitif pada Linux. File ini menyimpan password hash dan informasi keamanan akun seperti masa berlaku password, tanggal perubahan password terakhir, peringatan kedaluwarsa, dan status lock tertentu.

Walaupun password di `/etc/shadow` tidak disimpan dalam bentuk plaintext, hash password tetap sangat sensitif. Jika penyerang dapat membaca file ini, ia dapat melakukan **offline password cracking** menggunakan wordlist, brute force, atau teknik cracking lain tanpa perlu terus menyerang sistem secara langsung.

Risiko jika `/etc/shadow` dapat dibaca user biasa:

- hash password dapat dicuri;
- password lemah dapat dipecahkan offline;
- informasi expiration dapat digunakan untuk menyerang akun tertentu;
- akun service atau akun lama dapat menjadi target;
- kontrol autentikasi lokal menjadi terekspos.

Berbeda dengan `/etc/passwd` yang perlu world-readable, `/etc/shadow` harus sangat terbatas. Secara umum, hanya root atau mekanisme sistem tertentu yang boleh mengaksesnya. CIS menekankan mode yang lebih ketat, misalnya `0640` atau lebih restriktif, dengan owner root dan group yang sesuai seperti `root` atau `shadow`.

Konsep pentingnya: `/etc/shadow` bukan sekadar file konfigurasi. Ia adalah **database rahasia autentikasi lokal**.

---

## `/etc/shadow-`

`/etc/shadow-` adalah backup dari `/etc/shadow`. Karena file ini juga berisi password hash dan informasi keamanan akun, risikonya sama seriusnya dengan `/etc/shadow`.

Bahkan dalam beberapa kondisi, file backup bisa lebih berbahaya karena mungkin berisi password hash lama. Jika user pernah mengganti password dari password lemah ke password kuat, backup lama mungkin masih menyimpan hash lama yang lebih mudah dipecahkan. Karena itu, backup shadow tidak boleh dianggap aman hanya karena bukan file aktif.

Risiko `/etc/shadow-`:

- hash password lama dapat dibaca;
- informasi account aging lama dapat dianalisis;
- penyerang dapat membandingkan perubahan password;
- backup yang tidak dikunci menjadi titik bocor kredensial.

Prinsip CIS: **file backup yang berisi rahasia harus dilindungi seperti file rahasia utama**.

---

## `/etc/gshadow`

`/etc/gshadow` menyimpan informasi sensitif tentang group, termasuk password group jika digunakan, administrator group, dan anggota group tertentu. Walaupun group password jarang digunakan dalam praktik modern, file ini tetap mengandung informasi keamanan group.

Jika `/etc/gshadow` dapat dibaca oleh user biasa, penyerang dapat memperoleh informasi yang membantu subversi group. Jika dapat ditulis, risikonya lebih berat: penyerang bisa memanipulasi administrator group atau keanggotaan group.

CIS mencatat bahwa `/etc/gshadow` mungkin tidak selalu ada secara default. Namun, praktik keamanan yang baik mendorong adanya pemisahan informasi group sensitif dari `/etc/group`, sebagaimana `/etc/shadow` memisahkan hash password dari `/etc/passwd`.

Risiko `/etc/gshadow`:

- membocorkan informasi sensitif group;
- memungkinkan analisis struktur privilege;
- jika writable, dapat mengubah kontrol group;
- dapat menjadi jalur privilege escalation melalui group membership.

---

## `/etc/gshadow-`

`/etc/gshadow-` adalah backup dari `/etc/gshadow`. Karena isinya dapat menyimpan informasi sensitif group, file ini harus dibatasi aksesnya. Walaupun file ini mungkin tidak selalu ada, jika ada maka ia harus diperlakukan sebagai file sensitif.

Risiko umum file backup group shadow:

- membocorkan informasi group lama;
- memperlihatkan perubahan anggota group;
- menyimpan data sensitif yang tidak lagi terlihat pada file utama;
- menjadi bukti bahwa file backup tidak masuk cakupan hardening.

Konsep pentingnya sama dengan file backup lain: **backup bukan pengecualian dari kontrol keamanan**.

---

## `/etc/shells`

`/etc/shells` berisi daftar shell login yang valid. File ini digunakan oleh utilitas seperti `chsh` dan dapat dipakai oleh program lain untuk memverifikasi apakah sebuah shell dianggap valid.

File ini tidak berisi password hash, tetapi tetap penting untuk keamanan. Jika penyerang dapat mengubah `/etc/shells`, ia bisa memasukkan shell yang tidak seharusnya dianggap valid, atau mengubah perilaku program yang bergantung pada daftar shell valid.

Risiko jika `/etc/shells` dapat dimodifikasi:

- shell tidak sah dapat dianggap valid;
- kebijakan login shell menjadi tidak konsisten;
- user dapat diarahkan memakai shell yang tidak sesuai kebijakan;
- kontrol terhadap akun non-login dapat melemah.

Secara konseptual, `/etc/shells` adalah daftar referensi sistem. Ia boleh dibaca, tetapi tidak boleh dimodifikasi oleh user biasa.

---

## `/etc/security/opasswd`

`/etc/security/opasswd` dan backup-nya seperti `/etc/security/opasswd.old` menyimpan riwayat password lama jika mekanisme seperti `pam_unix` atau `pam_pwhistory` digunakan. File ini digunakan agar user tidak dapat memakai ulang password lama.

Karena berhubungan dengan riwayat password, file ini sangat sensitif. Walaupun isinya bukan plaintext password, data yang tersimpan dapat menjadi bahan serangan cracking atau analisis pola password.

Risiko jika file ini terbaca:

- hash password lama dapat dianalisis;
- pola rotasi password user dapat dipelajari;
- password lama yang lemah dapat dipecahkan;
- kontrol password history kehilangan kerahasiaannya.

Risiko jika file ini writable:

- penyerang dapat menghapus jejak password history;
- user dapat menghindari kebijakan larangan reuse password;
- integritas kebijakan password menjadi rusak.

Karena itu, file `opasswd` harus sangat ketat, biasanya hanya root yang dapat membaca dan menulis. Konsepnya: **password history adalah bagian dari kontrol autentikasi, bukan file biasa**.

---

## World Writable Files

**World writable files** adalah file yang dapat ditulis oleh semua user. Ini adalah kondisi berbahaya karena setiap user lokal, termasuk user dengan privilege rendah, dapat mengubah isi file tersebut.

Pada sistem multi-user, permission world writable dapat membuka banyak risiko:

- user biasa mengubah file konfigurasi aplikasi;
- file script dimodifikasi untuk menjalankan perintah berbahaya;
- data user lain dirusak;
- file dijadikan tempat menyisipkan payload;
- integritas sistem menjadi tidak dapat dipercaya.

Tidak semua world writable directory otomatis salah. Contohnya `/tmp` memang dirancang untuk dapat ditulis banyak user. Namun, direktori seperti ini perlu dilindungi dengan konsep seperti **sticky bit**, agar user tidak dapat menghapus atau mengganti file milik user lain.

Perbedaan penting:

- **world writable file**: hampir selalu berisiko karena isi file bisa diubah semua user;
- **world writable directory tanpa sticky bit**: berisiko karena user bisa menghapus atau mengganti file user lain;
- **world writable directory dengan sticky bit**: masih perlu dikendalikan, tetapi lebih aman untuk direktori bersama seperti `/tmp`.

Dalam konteks hardening, pencarian world writable file bukan sekadar checklist. Temuan seperti ini bisa menunjukkan:

- kesalahan instalasi aplikasi;
- script deployment yang buruk;
- permission hasil restore backup yang salah;
- percobaan penyerang membuat lokasi persistence.

---

## File Tanpa Owner atau Group

File tanpa owner atau tanpa group biasanya muncul ketika user atau group dihapus, tetapi file miliknya tidak ikut dipindahkan atau dihapus. Akibatnya, file tersebut masih menyimpan UID atau GID numerik yang tidak lagi memiliki nama user/group yang valid.

Masalahnya muncul ketika suatu saat UID atau GID tersebut digunakan kembali oleh user atau group baru. User baru itu bisa tiba-tiba “mewarisi” file lama yang sebenarnya bukan miliknya.

Contoh risiko:

1. user lama `dev1` dengan UID 1500 dihapus;
2. file milik UID 1500 masih ada;
3. beberapa bulan kemudian user baru mendapat UID 1500;
4. user baru tersebut bisa memiliki akses ke file lama milik `dev1`.

Risiko file tanpa owner/group:

- akses tidak sengaja ke data lama;
- akuntabilitas file hilang;
- file peninggalan aplikasi tidak terkontrol;
- potensi privilege escalation jika file memiliki permission khusus;
- kebingungan dalam investigasi forensik.

CIS menekankan bahwa file tanpa owner atau group harus diselidiki. Remediasinya tidak selalu sekadar menghapus file. Administrator harus memahami asal file, fungsi file, apakah masih dibutuhkan, dan siapa pemilik yang benar.

---

## Review SUID dan SGID

**SUID** dan **SGID** adalah permission khusus pada Linux. Jika sebuah file executable memiliki bit SUID, maka ketika dijalankan, proses dapat berjalan dengan hak owner file tersebut, bukan hak user yang menjalankan. Jika owner-nya root, program tersebut dapat menjalankan fungsi tertentu dengan privilege root.

SGID bekerja mirip, tetapi terkait group. Program dengan SGID dapat berjalan dengan hak group pemilik file.

SUID/SGID tidak selalu salah. Beberapa program memang membutuhkan SUID agar user biasa dapat melakukan operasi terbatas yang memerlukan privilege lebih tinggi. Contoh konseptualnya adalah perubahan password, karena user perlu mengubah data autentikasi yang disimpan dalam file sensitif.

Namun, SUID/SGID sangat berbahaya jika:

- binary tidak dikenal muncul;
- binary resmi diganti oleh penyerang;
- script atau program custom diberi SUID root;
- permission SUID diberikan tanpa alasan bisnis;
- file SUID berada di lokasi yang tidak umum;
- binary memiliki vulnerability.

Karena itu CIS menandai review SUID/SGID sebagai aktivitas yang membutuhkan penilaian. Tidak cukup hanya “ada atau tidak ada”. Administrator harus memahami apakah file tersebut legitimate.

Yang perlu diperiksa secara konseptual:

- apakah file berasal dari paket resmi;
- apakah checksum binary sesuai paket;
- apakah lokasi file wajar;
- apakah owner dan group wajar;
- apakah permission khusus memang dibutuhkan;
- apakah ada perubahan setelah instalasi;
- apakah file muncul setelah insiden atau aktivitas mencurigakan.

Review SUID/SGID berkaitan langsung dengan privilege escalation. Banyak serangan lokal berusaha menemukan binary SUID yang bisa disalahgunakan untuk mendapatkan root shell atau membaca file sensitif.

---

# 7.2 Local User and Group Settings

## Konsep Local User and Group Settings

Bagian ini membahas konsistensi user dan group lokal pada sistem Debian. CIS memberi catatan bahwa pemeriksaan ini berfokus pada user dan group lokal. Jika organisasi memakai sumber identitas eksternal seperti LDAP, Active Directory, atau domain identity provider, maka pemeriksaan serupa harus dilakukan pada sumber identitas tersebut juga.

Di Linux, user dan group bukan hanya nama. Sistem terutama mengenali identitas melalui angka:

- **UID** untuk user;
- **GID** untuk group.

Nama user dan group membantu manusia membaca sistem, tetapi kernel dan filesystem menggunakan UID/GID untuk menentukan ownership dan akses. Karena itu, inkonsistensi UID/GID dapat menyebabkan masalah besar.

Tujuan bagian ini:

- memastikan password lokal disimpan secara aman;
- memastikan tidak ada akun tanpa password;
- memastikan group di `/etc/passwd` benar-benar ada di `/etc/group`;
- memastikan group sensitif seperti `shadow` tidak berisi user tidak semestinya;
- memastikan UID, GID, username, dan group name unik;
- memastikan home directory user interaktif aman;
- memastikan dot files user tidak membuka jalur akses berbahaya.

---

## Shadowed Passwords

**Shadowed passwords** berarti password hash tidak disimpan di `/etc/passwd`, melainkan di `/etc/shadow`. Pada sistem modern, `/etc/passwd` tetap readable untuk kebutuhan sistem, tetapi field password di `/etc/passwd` biasanya berisi `x`, yang menandakan bahwa password hash sebenarnya berada di `/etc/shadow`.

Konsep ini penting karena `/etc/passwd` harus bisa dibaca banyak program. Jika password hash disimpan langsung di `/etc/passwd`, maka semua user lokal dapat membaca hash tersebut dan melakukan cracking offline.

Dengan shadowed passwords:

- `/etc/passwd` tetap dapat dipakai untuk informasi akun umum;
- hash password dipindahkan ke `/etc/shadow`;
- `/etc/shadow` diberi permission lebih ketat;
- risiko kebocoran hash password berkurang.

Risiko jika akun tidak memakai shadowed password:

- hash password bisa terekspos di file world-readable;
- password cracking menjadi lebih mudah;
- standar keamanan autentikasi lokal menurun;
- sistem menunjukkan konfigurasi lama atau salah.

Secara konseptual, shadowed passwords adalah pemisahan antara **informasi identitas umum** dan **rahasia autentikasi**.

---

## Empty Password Fields

Field password kosong di `/etc/shadow` berarti akun dapat berpotensi login tanpa password. Ini adalah kondisi kritis karena siapa pun yang mengetahui username dapat mencoba masuk tanpa autentikasi yang layak.

Akun tanpa password berbeda dengan akun yang terkunci. Akun terkunci biasanya memiliki penanda tertentu yang membuat login tidak bisa dilakukan. Sementara field kosong menunjukkan tidak ada rahasia autentikasi yang dibutuhkan.

Risiko empty password:

- akses tanpa kredensial;
- privilege escalation jika akun memiliki hak tinggi;
- akses lateral oleh user lokal;
- bypass kebijakan password;
- membuka celah pada service yang menerima autentikasi lokal.

Dalam hardening, semua akun harus memiliki password yang valid atau dikunci. Jika ditemukan akun tanpa password, itu harus diperlakukan sebagai temuan serius dan perlu investigasi: apakah akun tersebut dibuat sengaja, akibat kesalahan administrasi, atau hasil kompromi.

---

## Konsistensi Group

Setiap user di `/etc/passwd` memiliki GID utama. GID tersebut seharusnya menunjuk ke group yang benar-benar ada di `/etc/group`. Jika GID di `/etc/passwd` tidak ada padanannya di `/etc/group`, maka terjadi inkonsistensi identitas.

Dampak inkonsistensi group:

- permission berbasis group menjadi tidak jelas;
- file baru yang dibuat user dapat memiliki group tidak valid;
- audit ownership menjadi sulit;
- aplikasi yang memeriksa group dapat gagal atau salah membaca akses;
- manajemen akses menjadi tidak dapat dipercaya.

Kondisi ini biasanya muncul karena:

- group dihapus tetapi user masih menunjuk ke GID tersebut;
- file `/etc/passwd` diedit manual;
- migrasi akun tidak lengkap;
- restore backup sebagian;
- script administrasi gagal.

Prinsip konseptualnya: setiap referensi group harus menunjuk ke group yang valid. Sistem identitas lokal harus konsisten agar kontrol akses dapat bekerja dengan benar.

---

## Shadow Group

Group `shadow` memungkinkan program tertentu membaca `/etc/shadow`. Karena `/etc/shadow` berisi password hash dan informasi keamanan akun, group ini sangat sensitif.

CIS menekankan bahwa tidak boleh ada user yang menjadi anggota group `shadow` secara tidak semestinya. Jika user biasa masuk group `shadow`, ia dapat memperoleh kemampuan membaca `/etc/shadow`, lalu melakukan password cracking offline.

Risiko user dalam group `shadow`:

- membaca hash password semua user;
- mempelajari informasi expiration akun;
- menyerang akun lain melalui cracking;
- meningkatkan risiko kompromi seluruh sistem.

Konsepnya sederhana: akses ke `/etc/shadow` harus diberikan hanya kepada proses sistem yang benar-benar membutuhkan, bukan kepada user biasa. Group sensitif bukan tempat “mempermudah administrasi”.

---

## Duplicate UID

UID adalah identitas numerik user. Linux menggunakan UID untuk menentukan ownership file dan hak akses. Jika dua user memiliki UID yang sama, maka sistem dapat memperlakukan keduanya sebagai identitas yang sama pada level filesystem.

Walaupun program seperti `useradd` biasanya mencegah UID ganda, administrator masih bisa menciptakan UID ganda dengan mengedit file secara manual atau melalui migrasi yang buruk.

Risiko duplicate UID:

- dua user berbagi akses file yang sama;
- akuntabilitas audit menjadi rusak;
- log bisa sulit dibedakan secara konseptual;
- user dapat mengakses data user lain;
- ownership file menjadi tidak bermakna.

Contoh konseptual: jika `userA` dan `userB` memiliki UID yang sama, maka file yang dimiliki UID tersebut dapat tampak sebagai milik salah satu nama user tergantung interpretasi sistem. Ini bukan sekadar masalah tampilan, tetapi masalah kontrol akses.

Prinsipnya: **satu UID hanya untuk satu identitas user**.

---

## Duplicate GID

GID adalah identitas numerik group. Jika dua group memiliki GID yang sama, maka keduanya secara efektif berbagi identitas group pada level filesystem.

Risiko duplicate GID:

- group berbeda dapat mengakses file yang sama;
- batas akses antar group hilang;
- audit group membership menjadi membingungkan;
- aplikasi dapat salah menafsirkan group;
- privilege berbasis group dapat melebar tanpa sengaja.

Duplicate GID sering terjadi karena edit manual, migrasi akun, atau sinkronisasi identitas yang buruk. Seperti UID, GID harus unik agar ownership dan permission dapat dipercaya.

Prinsipnya: **satu GID hanya untuk satu group**.

---

## Duplicate Usernames

Username ganda terjadi ketika dua entry di `/etc/passwd` memakai nama user yang sama. Walaupun UID-nya berbeda, username ganda tetap berbahaya karena banyak tool dan proses administrasi mengandalkan nama user sebagai identitas manusia.

CIS menjelaskan bahwa jika nama user ganda ada, login dengan nama tersebut dapat menggunakan entry pertama yang ditemukan. Ini menciptakan kebingungan dan berpotensi menyebabkan akses yang tidak diharapkan.

Risiko duplicate usernames:

- login tidak mengarah ke identitas yang diharapkan;
- file ownership terlihat membingungkan;
- audit aktivitas user menjadi tidak jelas;
- administrator salah mengubah akun;
- penyerang dapat menyamarkan akun berbahaya dengan nama yang sama.

Prinsipnya: username harus unik agar identitas manusia dan identitas sistem tetap konsisten.

---

## Duplicate Group Names

Duplicate group names terjadi ketika dua entry di `/etc/group` memakai nama group yang sama. Meskipun GID-nya berbeda, group name ganda dapat membuat administrasi dan audit menjadi salah.

Risiko duplicate group names:

- administrator mengira mengubah satu group, padahal ada entry lain;
- aplikasi salah membaca group;
- permission berbasis group sulit diaudit;
- user dapat masuk ke group yang tidak dimaksud;
- GID efektif yang dipakai bisa tidak sesuai harapan.

Group name adalah label manusia untuk GID. Jika label ini tidak unik, maka proses administrasi menjadi rawan kesalahan.

Prinsipnya: group name harus unik agar kebijakan akses berbasis group dapat dipahami dan diaudit.

---

## Home Directory User Interaktif

Home directory adalah ruang kerja user untuk menyimpan file pribadi, konfigurasi shell, konfigurasi aplikasi, dan environment lokal. Untuk user interaktif, home directory sangat penting karena banyak proses login akan membaca file di sana.

CIS menekankan beberapa konsep penting:

1. home directory user interaktif harus ada;
2. home directory harus dimiliki oleh user yang benar;
3. permission home directory harus cukup restriktif;
4. home directory tidak boleh group/world-writable secara berlebihan.

Risiko jika home directory tidak ada:

- user bisa ditempatkan di `/` atau lokasi tidak semestinya;
- environment login tidak terbentuk dengan benar;
- aplikasi gagal menyimpan konfigurasi;
- perilaku login menjadi tidak konsisten.

Risiko jika home directory dimiliki user lain:

- user tidak dapat mengelola file sendiri;
- pemilik lain dapat membaca atau mengubah konfigurasi user;
- terjadi kebingungan ownership;
- potensi privilege abuse antar user.

Risiko jika home directory terlalu terbuka:

- user lain dapat membaca file pribadi;
- user lain dapat mengubah konfigurasi shell;
- file startup seperti `.bashrc` dapat dimodifikasi;
- penyerang lokal dapat menanam persistence;
- data sensitif user bocor.

Secara konseptual, home directory adalah batas pribadi user. Jika batas ini rusak, maka isolasi antar user melemah.

---

## Dot Files User Interaktif

**Dot files** adalah file tersembunyi di home directory yang namanya diawali titik, seperti `.bashrc`, `.profile`, `.bash_history`, `.netrc`, `.forward`, dan `.rhosts`. File-file ini sering memengaruhi perilaku login, shell, autentikasi, command history, dan integrasi aplikasi.

Dot files berbahaya karena sering dieksekusi atau dibaca otomatis. Jika penyerang dapat mengubah dot files user, ia bisa memengaruhi sesi login user tersebut.

Contoh risiko dot files:

- `.bashrc` atau `.profile` dimodifikasi untuk menjalankan perintah berbahaya saat user login;
- `.bash_history` terbaca dan mengungkap perintah sensitif;
- `.netrc` menyimpan kredensial untuk layanan remote atau API;
- `.forward` mengalihkan email user ke alamat lain;
- `.rhosts` dapat melemahkan autentikasi remote lama.

CIS memberi perhatian khusus pada beberapa dot files:

### `.forward`

`.forward` dapat digunakan untuk meneruskan email user ke alamat lain. Dalam konteks keamanan, ini berisiko karena email sistem atau notifikasi sensitif dapat dialihkan tanpa sepengetahuan administrator.

Risiko `.forward`:

- kebocoran notifikasi sistem;
- pengalihan email internal;
- persistence untuk exfiltration informasi;
- penyalahgunaan akun email lokal.

### `.rhosts`

`.rhosts` berkaitan dengan mekanisme remote trust lama seperti `rsh`, `rlogin`, atau `rcp`. File ini dapat mengizinkan host/user tertentu dianggap tepercaya tanpa password. Mekanisme seperti ini sudah sangat tidak sesuai dengan praktik keamanan modern.

Risiko `.rhosts`:

- bypass autentikasi password;
- akses remote berbasis trust yang lemah;
- sulit diaudit;
- dapat menjadi backdoor lama yang terlupakan.

### `.netrc`

`.netrc` dapat menyimpan informasi login ke host remote atau API. Jika permission-nya terlalu terbuka, kredensial dapat dibaca user lain.

Risiko `.netrc`:

- kredensial remote bocor;
- akses ke layanan eksternal disalahgunakan;
- token atau password API terekspos;
- lateral movement dari host lokal ke sistem lain.

Jika `.netrc` memang diperlukan oleh kebijakan lokal, file ini harus sangat restriktif, misalnya hanya dapat dibaca oleh pemiliknya.

### `.bash_history`

`.bash_history` menyimpan riwayat perintah shell. File ini sering dianggap sepele, padahal dapat berisi informasi sensitif seperti nama host, path rahasia, token yang tidak sengaja diketik, perintah database, atau pola administrasi.

Risiko `.bash_history`:

- mengungkap perintah administratif;
- membocorkan token/password yang pernah salah diketik;
- membantu penyerang memahami struktur sistem;
- mempercepat privilege escalation.

Karena itu, permission `.bash_history` harus dibatasi agar user lain tidak dapat membacanya.

---

## Hubungan Bab 7 dengan Persistence Attacker

Bab 7 sangat terkait dengan konsep **persistence**, yaitu kemampuan penyerang mempertahankan akses setelah berhasil masuk. Banyak teknik persistence lokal memanfaatkan permission yang salah, dot files, SUID/SGID, user/group palsu, atau file world writable.

Contoh pola persistence yang relevan:

- menambahkan user baru di `/etc/passwd`;
- mengubah group membership;
- membuat UID 0 tambahan;
- menanam script di dot files user;
- membuat binary SUID palsu;
- mengubah file world writable agar menjalankan payload;
- menyimpan kredensial di `.netrc`;
- memakai file tanpa owner untuk menyembunyikan jejak.

Karena itu, System Maintenance bukan hanya “rapi-rapi file”. Ini adalah pemeriksaan rutin terhadap tanda-tanda bahwa sistem mungkin sudah berubah dari baseline aman.

---

## Hubungan Bab 7 dengan Audit dan Forensik

Bab 7 juga mendukung audit dan forensik. Ketika terjadi insiden, investigator perlu menjawab:

- apakah ada akun baru yang tidak sah?
- apakah ada UID/GID ganda?
- apakah ada file SUID/SGID mencurigakan?
- apakah file penting berubah permission?
- apakah ada file tanpa owner?
- apakah home directory user dimodifikasi?
- apakah dot files digunakan untuk persistence?

Jika kondisi Bab 7 tidak dipantau, maka sulit membedakan perubahan normal dan perubahan berbahaya. Sistem yang memiliki permission berantakan juga lebih sulit diaudit karena tidak ada baseline yang jelas.

---

## Ringkasan Konseptual Bab 7

Bab 7 — **System Maintenance** menekankan bahwa keamanan Linux tidak berhenti pada instalasi awal. Sistem harus terus diperiksa agar file penting, permission, user, group, dan home directory tetap sesuai baseline.

Inti dari Bab 7:

1. File identitas seperti `/etc/passwd` dan `/etc/group` harus readable tetapi tidak writable oleh user biasa.
2. File rahasia seperti `/etc/shadow`, `/etc/gshadow`, dan `/etc/security/opasswd` harus sangat dibatasi.
3. File backup seperti `/etc/passwd-`, `/etc/group-`, `/etc/shadow-`, dan `/etc/gshadow-` harus diamankan seperti file utama.
4. World writable files berbahaya karena semua user dapat memodifikasi isi file.
5. World writable directories perlu sticky bit agar user tidak dapat menghapus file milik user lain.
6. File tanpa owner/group menunjukkan inkonsistensi administrasi dan dapat menyebabkan akses tidak sengaja jika UID/GID dipakai ulang.
7. SUID/SGID perlu direview karena dapat menjadi jalur privilege escalation.
8. Password lokal harus memakai shadowed passwords.
9. Akun dengan password kosong harus dianggap sebagai risiko kritis.
10. UID, GID, username, dan group name harus unik untuk menjaga akuntabilitas.
11. Home directory user interaktif harus ada, dimiliki user yang benar, dan permission-nya tidak terlalu terbuka.
12. Dot files harus diamankan karena dapat memengaruhi login, shell, autentikasi remote, command history, dan persistence.

Secara sederhana, Bab 7 memastikan bahwa sistem Debian tidak hanya aman dari sisi konfigurasi service, tetapi juga aman dari sisi **struktur identitas dan integritas filesystem lokal**.

---

## Peta Singkat Bab 7

| Area | Fokus Konseptual | Risiko Jika Diabaikan |
|---|---|---|
| `/etc/passwd` dan `/etc/group` | Identitas user dan group harus dilindungi dari perubahan tidak sah | Manipulasi akun, group, UID/GID, dan akses |
| `/etc/shadow` dan `/etc/gshadow` | Rahasia autentikasi dan group harus dibatasi | Password cracking, kebocoran hash, subversi akun |
| File backup `-` | Backup file sensitif tetap harus aman | Kebocoran informasi lama, bypass kontrol file utama |
| `/etc/security/opasswd` | Riwayat password harus dilindungi | Cracking password lama, bypass password history |
| World writable files | File tidak boleh dapat ditulis semua user | Modifikasi data, payload, persistence |
| File tanpa owner/group | Ownership harus valid dan konsisten | Akses tidak sengaja, akuntabilitas hilang |
| SUID/SGID | Program privilege khusus harus direview | Privilege escalation lokal |
| Shadowed passwords | Hash password harus dipisah dari `/etc/passwd` | Hash terbaca semua user |
| Empty password fields | Akun tidak boleh login tanpa password | Akses tanpa autentikasi |
| Duplicate UID/GID/name | Identitas harus unik | Audit kacau, akses silang antar user/group |
| Home directory | Ruang user harus ada dan aman | Data user bocor, konfigurasi login dimodifikasi |
| Dot files | File tersembunyi user harus dibatasi | Backdoor, kebocoran kredensial, persistence |

---

## Penutup Bab 7

Bab 7 adalah pengingat bahwa hardening bukan hanya menonaktifkan service atau mengatur firewall. Hardening juga berarti menjaga agar struktur paling dasar dari sistem tetap bersih, konsisten, dan dapat dipercaya.

Jika Bab 7 diabaikan, sistem bisa terlihat aman dari luar tetapi rapuh dari dalam. SSH mungkin sudah aman, firewall mungkin sudah aktif, auditd mungkin sudah berjalan, tetapi jika `/etc/shadow` terbaca, UID ganda muncul, file SUID asing ada, atau `.rhosts` tertanam di home user, maka penyerang lokal tetap memiliki banyak peluang.

Karena itu, Bab 7 perlu dipahami sebagai **kontrol pemeliharaan integritas sistem**. Tujuannya bukan hanya lolos audit, tetapi memastikan bahwa identitas, permission, ownership, dan file lokal tidak berubah menjadi celah keamanan.
