# 02 - Baseline, Package, Service, dan System Integrity

**Sumber utama:** CIS Ubuntu Linux 24.04 LTS STIG Benchmark v1.0.0, 06-12-2025.  
**Target dokumen:** modul belajar tematik untuk memahami hardening Ubuntu 24.04 LTS berbasis STIG.  
**Catatan penting:** file ini adalah dokumentasi pembelajaran dan panduan konseptual. Untuk audit resmi, tetap gunakan dokumen benchmark asli, hasil scanning, dan kebijakan organisasi.

---


## 1. Ruang lingkup modul

Modul ini membahas aturan STIG Ubuntu 24.04 LTS yang berkaitan dengan **baseline konfigurasi**, **integritas sistem**, **kebersihan package**, **service minimum**, **sinkronisasi waktu**, **AppArmor**, dan **proteksi kernel/memory**. Tema ini menjadi fondasi sebelum masuk ke firewall, SSH, autentikasi, dan audit.

Hardening selalu dimulai dari pertanyaan sederhana: **apa yang benar-benar harus ada pada sistem?** Komponen yang tidak diperlukan sebaiknya tidak dipasang, tidak aktif, atau dibatasi. Komponen yang wajib ada harus bisa diverifikasi integritasnya. Inilah alasan AIDE, Chrony, AppArmor, package verification, dan kontrol kernel menjadi bagian awal yang penting.

## 2. Peta aturan dalam modul ini

| Tema | Aturan terkait | Makna dalam hardening |
|---|---|---|
| Integritas audit tools dan baseline AIDE | `UBTU-24-90890` (Automated, CAT II), `UBTU-24-100100` (Automated, CAT II), `UBTU-24-100110` (Manual, CAT II), `UBTU-24-100120` (Manual, CAT II), `UBTU-24-100130` (Automated, CAT II) | AIDE digunakan untuk memeriksa integritas file, melindungi audit tools, menjalankan pemeriksaan berkala, dan memberi notifikasi ketika baseline berubah. |
| Service lama/tidak aman dan time synchronization | `UBTU-24-100010` (Automated, CAT III), `UBTU-24-100020` (Automated, CAT III), `UBTU-24-100030` (Automated, CAT I), `UBTU-24-100040` (Automated, CAT I), `UBTU-24-100700` (Automated, CAT III), `UBTU-24-600160` (Automated, CAT III), `UBTU-24-600180` (Automated, CAT III) | STIG memilih Chrony dan menghindari service lama/tidak aman seperti Telnet, RSH, NTP lama, dan systemd-timesyncd dalam baseline ini. |
| Package integrity dan lifecycle Ubuntu | `UBTU-24-300001` (Automated, CAT III), `UBTU-24-700320` (Automated, CAT II), `UBTU-24-700400` (Manual, CAT II) | APT harus memverifikasi tanda tangan paket, komponen lama dibersihkan setelah update, dan rilis Ubuntu harus masih didukung vendor. |
| AppArmor dan kontrol eksekusi | `UBTU-24-100500` (Automated, CAT II), `UBTU-24-100510` (Automated, CAT II) | AppArmor berfungsi sebagai mandatory access control untuk membatasi perilaku aplikasi berdasarkan profil. |
| Boot/maintenance mode, USB, kernel, dan memory protection | `UBTU-24-102000` (Automated, CAT I), `UBTU-24-300039` (Automated, CAT II), `UBTU-24-600030` (Automated, CAT I), `UBTU-24-600070` (Automated, CAT II), `UBTU-24-600140` (Automated, CAT III), `UBTU-24-600150` (Automated, CAT II), `UBTU-24-700300` (Manual, CAT II), `UBTU-24-700310` (Automated, CAT II) | Penguatan sistem meliputi autentikasi maintenance mode, pembatasan USB storage, FIPS, core dump, kernel message buffer, sticky bit, NX memory, dan ASLR. |

## 3. Konsep baseline konfigurasi

Baseline adalah kondisi awal yang dianggap aman dan disetujui. Dalam konteks STIG, baseline bukan hanya daftar paket dan konfigurasi, tetapi juga keadaan yang dapat dibuktikan. Sistem yang sudah di-hardening harus bisa menjawab:

- paket penting apa saja yang dipasang;
- service apa saja yang aktif;
- konfigurasi mana yang dianggap standar;
- file penting mana yang harus dipantau integritasnya;
- siapa yang diberi notifikasi saat baseline berubah;
- apakah perubahan terjadi secara sah atau tidak sah.

Tanpa baseline, administrator sulit membedakan perubahan normal dari perubahan mencurigakan. Misalnya, perubahan pada binary audit tools, konfigurasi PAM, SSH, atau audit rule bisa jadi bagian dari administrasi sah, tetapi bisa juga merupakan upaya attacker menurunkan kemampuan deteksi sistem.

## 4. AIDE dan integrity checking

AIDE atau Advanced Intrusion Detection Environment adalah file integrity checker. Tugas utamanya adalah membuat database baseline dari file penting, lalu membandingkan kondisi saat ini dengan baseline tersebut. STIG menempatkan AIDE sebagai kontrol penting karena banyak serangan pasca-kompromi berusaha mengubah file konfigurasi, binary sistem, atau audit tools.

AIDE penting untuk tiga tujuan:

1. **Deteksi perubahan tidak sah.** Jika file penting berubah tanpa proses change management, administrator harus mengetahuinya.
2. **Perlindungan audit tools.** Tool seperti `auditctl`, `auditd`, `ausearch`, dan `aureport` harus dipantau karena penyerang bisa menggantinya untuk menyembunyikan jejak.
3. **Bukti compliance.** Audit formal membutuhkan bukti bahwa sistem memiliki mekanisme pemeriksaan integritas.

Aturan terkait: `UBTU-24-90890` (Automated, CAT II), `UBTU-24-100100` (Automated, CAT II), `UBTU-24-100110` (Manual, CAT II), `UBTU-24-100120` (Manual, CAT II), `UBTU-24-100130` (Automated, CAT II).

Secara konseptual, AIDE bukan pengganti EDR atau antivirus. AIDE tidak selalu tahu apakah sebuah perubahan “jahat”. Ia memberi tahu bahwa perubahan terjadi. Interpretasi perubahan tetap membutuhkan admin, security analyst, atau proses monitoring.

## 5. Notifikasi perubahan baseline

Salah satu poin penting STIG adalah sistem harus memberi tahu personel yang ditunjuk ketika baseline berubah secara tidak sah. Ini penting karena file integrity checking tanpa notifikasi hanya menjadi arsip pasif. Jika hasil AIDE tidak pernah dibaca, kontrolnya tidak efektif.

Dalam praktik organisasi, hasil integrity check biasanya diarahkan ke:

- email administrator;
- sistem monitoring;
- SIEM;
- tiket insiden;
- laporan berkala compliance.

Perlu dibedakan antara perubahan sah dan tidak sah. Update paket resmi bisa mengubah binary sistem. Perubahan konfigurasi oleh admin juga bisa sah. Karena itu, integrity checking harus dikaitkan dengan change management.

## 6. Service minimization

Service minimization berarti hanya memasang dan menjalankan service yang benar-benar diperlukan. Setiap service tambahan memperbesar attack surface. Pada STIG Ubuntu 24.04 LTS, beberapa service lama atau tidak sesuai baseline harus tidak terpasang.

Telnet dan RSH mendapat perhatian khusus karena keduanya merupakan mekanisme akses jarak jauh yang tidak memberikan perlindungan modern seperti SSH. Jika kredensial atau sesi administratif lewat kanal tidak terenkripsi, confidentiality dan integrity dapat rusak.

Aturan terkait service lama: `UBTU-24-100030` (Automated, CAT I), `UBTU-24-100040` (Automated, CAT I).

Prinsipnya bukan “semua service server buruk”, tetapi **service harus sesuai misi sistem**. Web server boleh aktif jika mesin memang web server. Database boleh aktif jika mesin memang database server. Namun service yang tidak punya kebutuhan bisnis harus dihapus, dinonaktifkan, atau diberi pengecualian tertulis.

## 7. Time synchronization dan Chrony

Waktu sistem adalah dasar logging, audit, forensik, korelasi SIEM, validasi sertifikat, dan rekonstruksi insiden. Jika waktu antarhost berbeda jauh, urutan kejadian bisa salah. Dalam insiden keamanan, kesalahan urutan waktu dapat membuat analisis keliru.

STIG Ubuntu 24.04 LTS menekankan penggunaan Chrony dan menghindari time daemon tertentu seperti `systemd-timesyncd` dan `ntp` dalam baseline ini. Aturan terkait: `UBTU-24-100010` (Automated, CAT III), `UBTU-24-100020` (Automated, CAT III), `UBTU-24-100700` (Automated, CAT III), `UBTU-24-600160` (Automated, CAT III), `UBTU-24-600180` (Automated, CAT III).

Secara konsep, hanya satu mekanisme sinkronisasi waktu utama yang sebaiknya aktif. Menjalankan beberapa daemon waktu secara bersamaan dapat menyebabkan konflik, drift yang tidak konsisten, atau konfigurasi yang sulit diaudit.

## 8. Package verification dan vendor-supported release

Package manager adalah jalur utama masuknya software ke sistem. Karena itu, STIG menuntut APT mencegah pemasangan paket yang tidak diverifikasi tanda tangan digitalnya. Signed package membantu memastikan paket berasal dari sumber tepercaya dan tidak dimodifikasi di tengah jalan.

Aturan terkait: `UBTU-24-300001` (Automated, CAT III), `UBTU-24-700320` (Automated, CAT II), `UBTU-24-700400` (Manual, CAT II).

Ada tiga ide utama:

- **package authenticity:** paket harus berasal dari repository tepercaya;
- **package integrity:** paket tidak boleh berubah tanpa terdeteksi;
- **lifecycle support:** Ubuntu yang dipakai harus masih mendapat dukungan vendor.

Menggunakan rilis yang sudah tidak didukung berarti sistem mungkin tidak menerima patch keamanan. Dalam konteks compliance, ini biasanya menjadi temuan serius karena organisasi tidak bisa mengandalkan security update resmi.

## 9. AppArmor sebagai Mandatory Access Control

AppArmor adalah mekanisme Mandatory Access Control di Ubuntu. Berbeda dengan permission Linux biasa yang banyak bergantung pada owner/group/mode, AppArmor membatasi perilaku aplikasi berdasarkan profil. Sebuah proses bisa dibatasi agar hanya mengakses path tertentu, menjalankan kemampuan tertentu, atau melakukan operasi tertentu.

Aturan terkait: `UBTU-24-100500` (Automated, CAT II), `UBTU-24-100510` (Automated, CAT II).

AppArmor penting karena eksploitasi aplikasi tidak otomatis memberi akses penuh ke seluruh sistem. Jika sebuah service dikompromikan, profil AppArmor dapat membatasi dampaknya. Ini adalah bentuk containment.

Namun AppArmor efektif hanya jika:

- service AppArmor aktif;
- profil tersedia;
- profil berada dalam mode enforcing bila dibutuhkan;
- profil disesuaikan dengan aplikasi;
- pengecualian dicatat jika suatu aplikasi membutuhkan akses tambahan.

## 10. Boot, maintenance mode, USB, kernel, dan memory protection

STIG juga membahas penguatan area sistem yang sering menjadi celah lokal:

- single-user/maintenance mode harus membutuhkan autentikasi;
- USB mass storage perlu dibatasi untuk mengurangi risiko data exfiltration atau malware lokal;
- kernel core dump perlu dikendalikan karena bisa memuat data sensitif;
- kernel message buffer tidak boleh bebas dibaca user biasa;
- sticky bit pada direktori publik mencegah user menghapus file milik user lain;
- NX/non-executable memory dan ASLR membantu mengurangi eksploitasi memory corruption.

Aturan terkait: `UBTU-24-102000` (Automated, CAT I), `UBTU-24-300039` (Automated, CAT II), `UBTU-24-600030` (Automated, CAT I), `UBTU-24-600070` (Automated, CAT II), `UBTU-24-600140` (Automated, CAT III), `UBTU-24-600150` (Automated, CAT II), `UBTU-24-700300` (Manual, CAT II), `UBTU-24-700310` (Automated, CAT II).

Bagian ini memperlihatkan bahwa hardening tidak hanya tentang network. Banyak serangan memanfaatkan akses lokal, kesalahan permission, device removable, atau celah eksploitasi proses. Karena itu, baseline sistem harus melindungi boot path, memory, kernel information exposure, dan shared directories.

## 11. Checklist pemahaman modul

- AIDE digunakan untuk mendeteksi perubahan file penting dan audit tools.
- Baseline harus terhubung dengan notifikasi dan change management.
- Service lama seperti Telnet dan RSH memperbesar attack surface dan tidak aman untuk kredensial.
- Chrony dipilih sebagai mekanisme sinkronisasi waktu; konflik time daemon harus dihindari.
- APT harus memakai verifikasi tanda tangan paket.
- Ubuntu harus berada pada rilis yang masih didukung vendor.
- AppArmor membatasi perilaku aplikasi agar kompromi tidak mudah melebar.
- Proteksi boot, USB, kernel, memory, dan sticky bit adalah bagian dari hardening lokal.

## 12. Kesimpulan

Modul ini adalah pondasi. Sistem yang belum memiliki baseline, belum memeriksa integritas file, masih menjalankan service lama, tidak sinkron waktu, tidak memakai AppArmor, atau memakai rilis tidak didukung akan sulit disebut siap untuk hardening lanjutan. Setelah fondasi ini dipahami, barulah kontrol network, SSH, autentikasi, dan audit dapat dibangun dengan lebih konsisten.
