# Bank Soal Praktik Command CIS MariaDB 10.6 — Pilihan Ganda Jamak

**Fokus:** latihan praktik command, SQL audit, remediation, konfigurasi file, permission, TLS, audit plugin, authentication, privilege, network, backup, dan replication security berdasarkan CIS MariaDB 10.6 Benchmark v1.1.0 serta file implementasi hardening MariaDB.

> **Aturan latihan:** pilih semua jawaban yang benar. Jika tidak ada opsi benar, jawab **PASS / kosongkan jawaban**. Jangan menjalankan command destruktif di server asli; gunakan VM/lab.

## Daftar Cakupan

- Bab 1 Operating System Level Configuration
- Bab 2 Installation and Planning
- Bab 3 File Permissions
- Bab 4 General
- Bab 5 MariaDB Permissions
- Bab 6 Auditing and Logging
- Bab 7 Authentication
- Bab 8 Network
- Bab 9 Replication
- Validasi akhir dan exception register


## 0. Cara Menjawab Soal Praktik

### Soal 1

**Pertanyaan:** Dalam latihan pilihan ganda jamak ini, apa yang harus dilakukan jika semua opsi command salah?

A. `Pilih opsi yang paling mendekati benar`
B. `Pilih semua opsi agar tidak kosong`
C. `PASS / kosongkan jawaban`
D. `Pilih opsi A sebagai default`

**Jawaban benar:** C

**Pembahasan:** Jika tidak ada opsi command yang benar, jawaban harus PASS/kosong.

---
### Soal 2

**Pertanyaan:** Prinsip apa yang paling tepat saat memilih command hardening MariaDB berbasis CIS?

A. `Pilih command yang sesuai target MariaDB 10.6, path Linux, service, dan tujuan rekomendasi`
B. `Pilih command distro lain walaupun path/service berbeda`
C. `Pilih command yang terlihat paling panjang`
D. `Pilih command yang mengubah produksi tanpa backup`

**Jawaban benar:** A

**Pembahasan:** CIS MariaDB sangat bergantung pada versi, path, service name, dan konteks sistem operasi.

---

## 1. Operating System Level Configuration

### Soal 3

**Pertanyaan:** Perintah manakah yang benar untuk memeriksa versi MariaDB sebelum melakukan hardening?

A. `mariadb --version || true`
B. `mysql --version || true`
C. `sudo systemctl status mariadb --no-pager || true`
D. `mariadb version get --cis`

**Jawaban benar:** A, B, C

**Pembahasan:** Versi dapat dicek dengan client binary dan status service. Opsi D bukan command standar.

---
### Soal 4

**Pertanyaan:** Perintah manakah yang benar untuk memastikan service MariaDB berjalan sebagai user `mysql`?

A. `ps -ef | grep -E 'mariadbd|mysqld' | grep -v grep`
B. `getent passwd mysql`
C. `sudo systemctl status mariadb --no-pager`
D. `mysql-user --audit service`

**Jawaban benar:** A, B, C

**Pembahasan:** Proses, entry passwd, dan status service membantu audit service account. Opsi D tidak valid.

---
### Soal 5

**Pertanyaan:** Perintah manakah yang benar untuk menonaktifkan interactive login pada user service `mysql`?

A. `sudo usermod -s /usr/sbin/nologin mysql`
B. `sudo usermod -s /bin/false mysql`
C. `getent passwd mysql`
D. `sudo usermod -s /bin/bash mysql`

**Jawaban benar:** A, B, C

**Pembahasan:** nologin/bin false benar untuk service account. getent digunakan untuk validasi. Bash justru mengaktifkan login interaktif.

---
### Soal 6

**Pertanyaan:** Perintah manakah yang benar untuk menonaktifkan MariaDB command history pada akun root?

A. `sudo rm -f /root/.mysql_history /root/.mariadb_history`
B. `sudo ln -sf /dev/null /root/.mysql_history`
C. `sudo ln -sf /dev/null /root/.mariadb_history`
D. `sudo chmod 777 /root/.mysql_history`

**Jawaban benar:** A, B, C

**Pembahasan:** History dihapus lalu diarahkan ke /dev/null. chmod 777 berbahaya.

---
### Soal 7

**Pertanyaan:** Perintah manakah yang benar untuk mencari penggunaan environment variable `MYSQL_PWD`?

A. `sudo grep -a MYSQL_PWD /proc/*/environ 2>/dev/null || true`
B. `sudo grep -R "MYSQL_PWD" /root /home /etc/profile /etc/profile.d 2>/dev/null || true`
C. `printenv | grep MYSQL_PWD || true`
D. `mysql_secure_pwd --scan`

**Jawaban benar:** A, B, C

**Pembahasan:** A memeriksa environment proses, B memeriksa profile, C memeriksa environment shell aktif. Opsi D bukan command standar.

---
### Soal 8

**Pertanyaan:** Jika ditemukan `MYSQL_PWD` pada script login user, tindakan manakah yang sesuai prinsip CIS?

A. `Hapus `MYSQL_PWD` dari profile/script`
B. `Ganti dengan autentikasi yang lebih aman seperti unix_socket, certificate, atau secret management`
C. `Simpan password di file global world-readable agar mudah dipakai aplikasi`
D. `Tambahkan `export MYSQL_PWD=password` ke `/etc/profile``

**Jawaban benar:** A, B

**Pembahasan:** MYSQL_PWD tidak boleh dipakai karena credential bisa bocor. Opsi C/D memperburuk risiko.

---
### Soal 9

**Pertanyaan:** Command manakah yang benar untuk membuat direktori data/log/import/backup hardening MariaDB?

A. `sudo mkdir -p /srv/mariadb/{data,logs,backups,import}`
B. `sudo mkdir -p /etc/mysql/{ssl,encryption}`
C. `sudo chown -R mysql:mysql /srv/mariadb/data /srv/mariadb/logs /srv/mariadb/import`
D. `sudo chmod -R 777 /srv/mariadb`

**Jawaban benar:** A, B, C

**Pembahasan:** Direktori dibuat dan owner dibatasi. chmod 777 salah karena membuka akses terlalu luas.

---
### Soal 10

**Pertanyaan:** Permission manakah yang benar untuk direktori utama MariaDB pada contoh implementasi?

A. `sudo chmod 750 /srv/mariadb/data`
B. `sudo chmod 750 /srv/mariadb/logs`
C. `sudo chmod 700 /srv/mariadb/backups`
D. `sudo chmod 777 /srv/mariadb/import`

**Jawaban benar:** A, B, C

**Pembahasan:** Datadir/logs dibatasi, backup lebih ketat. Import tidak boleh 777.

---
### Soal 11

**Pertanyaan:** Perintah manakah yang benar untuk memindahkan datadir dari `/var/lib/mysql` ke `/srv/mariadb/data` secara aman?

A. `sudo systemctl stop mariadb`
B. `sudo rsync -aHAX --numeric-ids /var/lib/mysql/ /srv/mariadb/data/`
C. `sudo chown -R mysql:mysql /srv/mariadb/data`
D. `sudo rm -rf /var/lib/mysql tanpa backup lalu start service`

**Jawaban benar:** A, B, C

**Pembahasan:** Service dihentikan, data disalin dengan metadata, owner diperbaiki. Menghapus tanpa backup berbahaya.

---
### Soal 12

**Pertanyaan:** Baris AppArmor manakah yang benar jika datadir/log/import/encryption/ssl MariaDB dipindah sesuai implementasi?

A. `/srv/mariadb/data/ r,`
B. `/srv/mariadb/data/** rwk,`
C. `/etc/mysql/ssl/** r,`
D. `/srv/mariadb/data/** x,`

**Jawaban benar:** A, B, C

**Pembahasan:** MariaDB membutuhkan read/write/lock pada data, read pada ssl/encryption. Eksekusi luas tidak perlu.

---
### Soal 13

**Pertanyaan:** Perintah manakah yang benar untuk reload AppArmor setelah mengubah profile MariaDB?

A. `sudo systemctl reload apparmor`
B. `sudo apparmor_parser -r /etc/apparmor.d/usr.sbin.mariadbd`
C. `sudo systemctl restart apparmor || true`
D. `sudo apparmor-disable-all`

**Jawaban benar:** A, B, C

**Pembahasan:** Reload/restart atau apparmor_parser bisa dipakai. Menonaktifkan semua AppArmor tidak sesuai hardening.

---
### Soal 14

**Pertanyaan:** Konfigurasi systemd sandboxing mana yang benar untuk hardening MariaDB?

A. `NoNewPrivileges=true`
B. `PrivateTmp=true`
C. `ProtectSystem=full`
D. `User=root`

**Jawaban benar:** A, B, C

**Pembahasan:** NoNewPrivileges, PrivateTmp, dan ProtectSystem dapat memperkuat service. User root tidak sesuai prinsip least privilege.

---
### Soal 15

**Pertanyaan:** Jika MariaDB gagal start setelah systemd override, command manakah yang benar untuk troubleshooting?

A. `journalctl -u mariadb -xe --no-pager`
B. `sudo systemctl status mariadb --no-pager`
C. `sudo tail -n 100 /srv/mariadb/logs/mariadb.err`
D. `sudo rm -rf /etc/mysql`

**Jawaban benar:** A, B, C

**Pembahasan:** Gunakan journal, status service, dan error log. Menghapus konfigurasi tidak benar.

---

## 2. Installation and Planning

### Soal 16

**Pertanyaan:** Perintah manakah yang benar untuk instalasi paket MariaDB pada Debian/Ubuntu-family?

A. `sudo apt update`
B. `sudo apt -y install mariadb-server mariadb-client mariadb-backup`
C. `sudo systemctl enable mariadb`
D. `sudo pacman -S mariadb-server`

**Jawaban benar:** A, B, C

**Pembahasan:** Target implementasi memakai Debian/Ubuntu style. Pacman untuk Arch, bukan contoh ini.

---
### Soal 17

**Pertanyaan:** Perintah manakah yang benar untuk menjalankan hardening awal MariaDB setelah instalasi?

A. `sudo mariadb-secure-installation`
B. `Hapus anonymous user dan test database saat wizard meminta`
C. `Reload privilege tables jika diminta`
D. `Aktifkan anonymous user agar aplikasi mudah login`

**Jawaban benar:** A, B, C

**Pembahasan:** mariadb-secure-installation membantu menghapus default tidak aman. Anonymous user tidak boleh diaktifkan.

---
### Soal 18

**Pertanyaan:** Perintah manakah yang benar untuk membuat backup konfigurasi sebelum hardening?

A. `sudo mkdir -p /root/pre-cis-backup`
B. `sudo cp -a /etc/mysql /root/pre-cis-backup/mysql-etc-$(date +%F) 2>/dev/null || true`
C. `sudo systemctl status mariadb --no-pager || true`
D. `sudo rm -rf /etc/mysql`

**Jawaban benar:** A, B, C

**Pembahasan:** Backup konfigurasi dan catat status sebelum hardening. Menghapus /etc/mysql salah.

---
### Soal 19

**Pertanyaan:** Perintah manakah yang benar untuk backup penuh terenkripsi menggunakan `mariabackup` sesuai contoh implementasi?

A. sudo mariabackup --backup --user=root --stream=xbstream | openssl enc -aes-256-cbc -pbkdf2 -salt -out /srv/mariadb/backups/full-$(date +%F).xb.enc
B. `sudo mkdir -p /srv/mariadb/backups`
C. `sudo chmod 700 /srv/mariadb/backups`
D. `mysqldump --password=rahasia --all-databases > /tmp/db.sql`

**Jawaban benar:** A, B, C

**Pembahasan:** Backup terenkripsi dan direktori backup ketat. Password di command line berisiko.

---
### Soal 20

**Pertanyaan:** Command manakah yang benar untuk backup konfigurasi penting MariaDB?

A. sudo tar --xattrs --acls -czf /srv/mariadb/backups/mysql-config-$(date +%F).tar.gz /etc/mysql /etc/apparmor.d/usr.sbin.mariadbd /srv/mariadb/logs 2>/dev/null
B. `sudo chmod 600 /srv/mariadb/backups/mysql-config-$(date +%F).tar.gz`
C. `cp -a /etc/mysql /tmp/mysql-public-backup && chmod -R 777 /tmp/mysql-public-backup`
D. `Jangan backup TLS/encryption material karena tidak penting saat restore`

**Jawaban benar:** A, B

**Pembahasan:** Konfigurasi, AppArmor, log, TLS/encryption material penting untuk restore. Public backup salah.

---
### Soal 21

**Pertanyaan:** Perintah manakah yang benar untuk melakukan restore test dari backup terenkripsi secara umum?

A. `openssl enc -d -aes-256-cbc -pbkdf2 -in full-YYYY-MM-DD.xb.enc -out restore.xbstream`
B. `mbstream -x < restore.xbstream`
C. `mariabackup --prepare --target-dir=/path/restore`
D. `rm -rf /srv/mariadb/data lalu restore langsung di produksi tanpa test`

**Jawaban benar:** A, B, C

**Pembahasan:** Decrypt, extract, prepare adalah langkah umum restore test. Menghapus produksi tanpa test salah.

---
### Soal 22

**Pertanyaan:** Manakah pilihan yang benar terkait password di command line MariaDB?

A. `Gunakan prompt `-p` tanpa menulis password langsung`
B. `Gunakan file konfigurasi user-specific dengan permission 0600 bila diperlukan`
C. `Gunakan secret manager atau certificate authentication bila tersedia`
D. `Gunakan `mariadb -uroot -pPasswordRahasia` agar otomatis`

**Jawaban benar:** A, B, C

**Pembahasan:** Password literal di command line dapat muncul di process list/history.

---
### Soal 23

**Pertanyaan:** Manakah konfigurasi yang benar untuk password lifetime MariaDB sesuai baseline CIS?

A. `default_password_lifetime=365`
B. `SET GLOBAL default_password_lifetime=365;`
C. `ALTER USER 'app'@'10.10.10.20' PASSWORD EXPIRE INTERVAL 365 DAY;`
D. `default_password_lifetime=0 agar password tidak pernah kedaluwarsa`

**Jawaban benar:** A, B, C

**Pembahasan:** CIS merekomendasikan lifetime tidak lebih dari 365 hari. Nilai 0 berarti tidak kedaluwarsa.

---
### Soal 24

**Pertanyaan:** Perintah SQL manakah yang benar untuk mengunci akun MariaDB yang tidak digunakan?

A. `ALTER USER 'old_user'@'localhost' ACCOUNT LOCK;`
B. `ALTER USER 'old_user'@'10.10.10.20' ACCOUNT LOCK;`
C. `DROP USER 'old_user'@'localhost'; jika sudah dipastikan tidak dibutuhkan dan sudah ada approval`
D. `GRANT ALL ON *.* TO 'old_user'@'%';`

**Jawaban benar:** A, B, C

**Pembahasan:** Akun dormant dikunci atau dihapus setelah verifikasi. Grant all ke wildcard host salah.

---
### Soal 25

**Pertanyaan:** Command mana yang benar untuk memeriksa bind address MariaDB?

A. `SELECT VARIABLE_NAME, VARIABLE_VALUE FROM information_schema.global_variables WHERE VARIABLE_NAME = 'bind_address';`
B. `SHOW VARIABLES LIKE 'bind_address';`
C. `sudo ss -lntp | grep -E '3306|mariadbd|mysql' || true`
D. `SHOW BIND *;`

**Jawaban benar:** A, B, C

**Pembahasan:** Variabel dan socket listening dapat dipakai audit. SHOW BIND bukan SQL MariaDB valid.

---
### Soal 26

**Pertanyaan:** Nilai `bind_address` manakah yang paling aman untuk server local-only menurut contoh implementasi?

A. `bind_address=127.0.0.1`
B. `bind_address=10.10.10.10 jika itu IP database segment yang disetujui`
C. `bind_address=*`
D. `bind_address=::`

**Jawaban benar:** A, B

**Pembahasan:** Bind ke localhost atau IP segment yang disetujui. `*` dan `::` terlalu luas.

---

## 3. File Permissions

### Soal 27

**Pertanyaan:** Perintah manakah yang benar untuk audit dan remediation permission datadir MariaDB?

A. `SHOW VARIABLES WHERE variable_name = 'datadir';`
B. `sudo ls -ld /srv/mariadb/data`
C. `sudo chown mysql:mysql /srv/mariadb/data && sudo chmod 750 /srv/mariadb/data`
D. `sudo chmod 777 /srv/mariadb/data`

**Jawaban benar:** A, B, C

**Pembahasan:** Datadir harus terbatas untuk user/group mysql. chmod 777 sangat salah.

---
### Soal 28

**Pertanyaan:** Perintah mana yang benar untuk permission error log MariaDB?

A. `sudo touch /srv/mariadb/logs/mariadb.err`
B. `sudo chown mysql:mysql /srv/mariadb/logs/mariadb.err`
C. `sudo chmod 600 /srv/mariadb/logs/mariadb.err`
D. `sudo chmod 666 /srv/mariadb/logs/mariadb.err`

**Jawaban benar:** A, B, C

**Pembahasan:** Error log dibatasi. 666 membuat world-writable.

---
### Soal 29

**Pertanyaan:** Perintah mana yang benar untuk permission audit log MariaDB?

A. `sudo touch /srv/mariadb/logs/server_audit.log`
B. `sudo chown mysql:mysql /srv/mariadb/logs/server_audit.log`
C. `sudo chmod 660 /srv/mariadb/logs/server_audit.log`
D. `sudo chown nobody:nogroup /srv/mariadb/logs/server_audit.log`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit log harus bisa ditulis MariaDB tetapi tidak bebas diubah user lain.

---
### Soal 30

**Pertanyaan:** Perintah manakah yang benar untuk melindungi private key TLS MariaDB?

A. `sudo chown -R mysql:mysql /etc/mysql/ssl`
B. `sudo chmod 400 /etc/mysql/ssl/*key.pem`
C. `sudo chmod 444 /etc/mysql/ssl/*cert.pem /etc/mysql/ssl/ca.pem`
D. `sudo chmod 777 /etc/mysql/ssl/server-key.pem`

**Jawaban benar:** A, B, C

**Pembahasan:** Private key sangat ketat, cert publik boleh read-only. 777 salah.

---
### Soal 31

**Pertanyaan:** Perintah manakah yang benar untuk plugin directory MariaDB setelah mengetahui nilai `plugin_dir`?

A. `SHOW VARIABLES WHERE variable_name = 'plugin_dir';`
B. `PLUGIN_DIR="/usr/lib/mysql/plugin"  # sesuaikan hasil audit`
C. `sudo chown mysql:mysql "$PLUGIN_DIR" && sudo chmod 550 "$PLUGIN_DIR"`
D. `sudo chmod 777 "$PLUGIN_DIR"`

**Jawaban benar:** A, B, C

**Pembahasan:** Plugin directory tidak boleh writable oleh user biasa. chmod 777 membuka risiko plugin berbahaya.

---
### Soal 32

**Pertanyaan:** Perintah manakah yang benar untuk melindungi file key management encryption?

A. `sudo chown -R mysql:mysql /etc/mysql/encryption`
B. `sudo chmod 750 /etc/mysql/encryption`
C. `sudo chmod 640 /etc/mysql/encryption/keyfile*`
D. `sudo chmod 777 /etc/mysql/encryption/keyfile.key`

**Jawaban benar:** A, B, C

**Pembahasan:** Key management harus ketat. 777 membocorkan key material.

---
### Soal 33

**Pertanyaan:** Perintah manakah yang benar untuk memeriksa permission akhir file penting?

A. `sudo ls -ld /srv/mariadb/data /srv/mariadb/logs /srv/mariadb/import /srv/mariadb/backups`
B. `sudo ls -l /srv/mariadb/logs`
C. `sudo ls -l /etc/mysql/ssl`
D. `sudo chmod -R 777 /etc/mysql`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit permission memakai ls. chmod 777 /etc/mysql salah.

---
### Soal 34

**Pertanyaan:** Command berikut ditujukan untuk memperbaiki semua permission MariaDB dengan cepat. Manakah yang benar dan aman secara universal?

A. `sudo chmod -R 777 /srv/mariadb`
B. `sudo chown -R root:root /srv/mariadb/data`
C. `sudo find / -name '*mysql*' -exec chmod 600 {} \;`
D. `sudo rm -rf /srv/mariadb/logs`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: semua opsi destruktif/keliru. Permission harus spesifik per path dan fungsi.

---

## 4. General Configuration

### Soal 35

**Pertanyaan:** Directive mana yang benar dalam file `60-cis-hardening.cnf` untuk mengurangi risiko local file import?

A. `local-infile=0`
B. `secure_file_priv=/srv/mariadb/import`
C. `skip-grant-tables=FALSE`
D. `local-infile=1`

**Jawaban benar:** A, B, C

**Pembahasan:** local_infile dimatikan, secure_file_priv dibatasi, skip-grant-tables tidak boleh aktif.

---
### Soal 36

**Pertanyaan:** Directive mana yang benar untuk symbolic link dan strict SQL mode?

A. `skip-symbolic-links`
B. sql_mode=STRICT_ALL_TABLES,ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
C. `sql_mode=''`
D. `symbolic-links=1`

**Jawaban benar:** A, B

**Pembahasan:** Symbolic links dinonaktifkan dan STRICT_ALL_TABLES diaktifkan. Opsi C/D melemahkan kontrol.

---
### Soal 37

**Pertanyaan:** Perintah SQL manakah yang benar untuk memvalidasi `sql_mode` dan `secure_file_priv`?

A. `SHOW VARIABLES LIKE 'sql_mode';`
B. `SHOW VARIABLES LIKE 'secure_file_priv';`
C. `SHOW VARIABLES WHERE Variable_name IN ('secure_file_priv','sql_mode');`
D. `SHOW SECURE FILE PRIVILEGE;`

**Jawaban benar:** A, B, C

**Pembahasan:** SHOW VARIABLES benar. Opsi D tidak valid.

---
### Soal 38

**Pertanyaan:** Perintah SQL manakah yang benar untuk memvalidasi `old_passwords` dan `secure_auth`?

A. `SHOW VARIABLES WHERE Variable_name = 'old_passwords';`
B. `SHOW VARIABLES WHERE Variable_name = 'secure_auth';`
C. `SHOW VARIABLES WHERE Variable_name IN ('old_passwords','secure_auth');`
D. `SHOW PASSWORD LEGACY STATUS;`

**Jawaban benar:** A, B, C

**Pembahasan:** SHOW VARIABLES digunakan untuk audit variable. Opsi D tidak valid.

---
### Soal 39

**Pertanyaan:** Konfigurasi manakah yang salah jika tujuannya hardening MariaDB?

A. `skip-grant-tables=TRUE`
B. `local-infile=1`
C. `secure_file_priv=''`
D. `old_passwords=1`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua opsi memperlemah keamanan: bypass privilege, local infile aktif, secure_file_priv kosong, dan old password aktif.

---
### Soal 40

**Pertanyaan:** Manakah command yang benar untuk menghapus test database dan anonymous user melalui secure installation atau SQL manual?

A. `sudo mariadb-secure-installation`
B. `DROP DATABASE test;`
C. `SELECT user,host FROM mysql.user WHERE user = '';`
D. `CREATE USER ''@'%' IDENTIFIED BY '';`

**Jawaban benar:** A, B, C

**Pembahasan:** secure-installation atau SQL dapat menghapus default tidak aman. Anonymous user tidak boleh dibuat.

---
### Soal 41

**Pertanyaan:** Perintah manakah yang benar untuk membuat file konfigurasi CIS MariaDB dan mengamankan permission-nya?

A. `sudo tee /etc/mysql/mariadb.conf.d/60-cis-hardening.cnf >/dev/null <<'CONF'`
B. `sudo chown root:root /etc/mysql/mariadb.conf.d/60-cis-hardening.cnf`
C. `sudo chmod 640 /etc/mysql/mariadb.conf.d/60-cis-hardening.cnf`
D. `sudo chmod 777 /etc/mysql/mariadb.conf.d/60-cis-hardening.cnf`

**Jawaban benar:** A, B, C

**Pembahasan:** Config harus root-owned dan tidak world-writable.

---
### Soal 42

**Pertanyaan:** Command mana yang benar untuk restart bertahap dan cek error setelah konfigurasi MariaDB?

A. `sudo systemctl restart mariadb`
B. `sudo systemctl status mariadb --no-pager`
C. `journalctl -u mariadb -xe --no-pager`
D. `sudo systemctl disable mariadb`

**Jawaban benar:** A, B, C

**Pembahasan:** Restart dan cek status/log benar. Disable service bukan validasi hardening.

---
### Soal 43

**Pertanyaan:** Perintah manakah yang benar untuk memvalidasi variable encryption MariaDB?

A. sudo mariadb -e "SELECT VARIABLE_NAME, VARIABLE_VALUE FROM information_schema.global_variables WHERE variable_name LIKE '%ENCRYPT%';"
B. `SHOW VARIABLES LIKE '%encrypt%';`
C. `SELECT VARIABLE_NAME, VARIABLE_VALUE FROM information_schema.global_variables WHERE variable_name LIKE '%ENCRYPT%';`
D. `mariadb-encrypt --status`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit encryption variable memakai SQL. mariadb-encrypt bukan command standar.

---

## 5. TLS dan Data-at-Rest Encryption

### Soal 44

**Pertanyaan:** Perintah manakah yang benar untuk membuat direktori TLS MariaDB?

A. `sudo mkdir -p /etc/mysql/ssl`
B. `cd /etc/mysql/ssl`
C. `sudo chown -R mysql:mysql /etc/mysql/ssl`
D. `sudo chmod 777 /etc/mysql/ssl`

**Jawaban benar:** A, B, C

**Pembahasan:** Direktori TLS dibuat dan owner dibatasi. 777 tidak aman.

---
### Soal 45

**Pertanyaan:** Perintah OpenSSL manakah yang benar untuk membuat CA lokal pada contoh implementasi?

A. `sudo openssl genrsa 4096 | sudo tee ca-key.pem >/dev/null`
B. `sudo openssl req -new -x509 -nodes -days 3650 -key ca-key.pem -out ca.pem -subj "/CN=MariaDB-CIS-Local-CA"`
C. `sudo openssl genrsa 4096 > /tmp/public-key.txt && chmod 777 /tmp/public-key.txt`
D. `sudo openssl x509 -in ca.pem -text -noout`

**Jawaban benar:** A, B, D

**Pembahasan:** A/B membuat CA; D bisa validasi certificate. Menaruh key di /tmp world-readable salah.

---
### Soal 46

**Pertanyaan:** Directive TLS manakah yang benar untuk MariaDB CIS baseline?

A. `require_secure_transport=ON`
B. `tls_version=TLSv1.2,TLSv1.3`
C. `ssl-ca=/etc/mysql/ssl/ca.pem`
D. `tls_version=TLSv1,TLSv1.1`

**Jawaban benar:** A, B, C

**Pembahasan:** TLS secure transport aktif, versi lama tidak dipakai.

---
### Soal 47

**Pertanyaan:** Perintah SQL manakah yang benar untuk audit TLS MariaDB?

A. `SELECT @@require_secure_transport;`
B. `SHOW variables WHERE variable_name = 'have_ssl';`
C. `SELECT @@tls_version;`
D. `SHOW TLS CERTIFICATE ALL;`

**Jawaban benar:** A, B, C

**Pembahasan:** Variable TLS dapat diaudit dengan SELECT/SHOW VARIABLES. Opsi D tidak valid.

---
### Soal 48

**Pertanyaan:** Perintah manakah yang benar untuk membuat file key management encryption?

A. `sudo mkdir -p /etc/mysql/encryption`
B. `sudo bash -c '(echo -n "1;" ; openssl rand -hex 32) > /etc/mysql/encryption/keyfile'`
C. `sudo openssl rand -hex 128 | sudo tee /etc/mysql/encryption/keyfile.key >/dev/null`
D. `sudo chmod 777 /etc/mysql/encryption/keyfile.key`

**Jawaban benar:** A, B, C

**Pembahasan:** Key dibuat di path khusus. Permission 777 salah.

---
### Soal 49

**Pertanyaan:** Directive mana yang benar untuk mengaktifkan file key management dan encryption at rest pada MariaDB?

A. `plugin_load_add=file_key_management`
B. `file_key_management_filename=/etc/mysql/encryption/keyfile.enc`
C. `file_key_management_filekey=FILE:/etc/mysql/encryption/keyfile.key`
D. `innodb_encrypt_tables=OFF`

**Jawaban benar:** A, B, C

**Pembahasan:** A-C benar untuk plugin dan key file. innodb_encrypt_tables harus ON bila menerapkan at-rest encryption.

---
### Soal 50

**Pertanyaan:** Directive mana yang benar untuk enkripsi binary log, redo log, temporary file, dan table?

A. `encrypt_binlog=ON`
B. `innodb_encrypt_log=ON`
C. `encrypt_tmp_files=ON`
D. `innodb_encrypt_tables=ON`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua mendukung hardening data/log at rest.

---
### Soal 51

**Pertanyaan:** Perintah SQL manakah yang benar untuk mengenkripsi tabel yang sudah ada secara bertahap?

A. `ALTER TABLE nama_database.nama_tabel ENCRYPTED=YES ENCRYPTION_KEY_ID=1;`
B. `SHOW VARIABLES LIKE '%ENCRYPT%';`
C. `ALTER TABLE nama_database.nama_tabel ENCRYPTED=NO;`
D. `DROP TABLE nama_database.nama_tabel;`

**Jawaban benar:** A, B

**Pembahasan:** A melakukan enkripsi tabel, B audit. Opsi C mematikan enkripsi, D destruktif.

---

## 6. Authentication dan Password Policy

### Soal 52

**Pertanyaan:** Perintah SQL manakah yang benar untuk mengaktifkan plugin password check?

A. `INSTALL SONAME 'simple_password_check';`
B. `INSTALL SONAME 'cracklib_password_check';`
C. `SHOW PLUGINS;`
D. `INSTALL SONAME 'weak_password_allow_all';`

**Jawaban benar:** A, B, C

**Pembahasan:** simple_password_check dan cracklib_password_check benar. Opsi D tidak valid dan tidak sesuai keamanan.

---
### Soal 53

**Pertanyaan:** Directive mana yang benar untuk password complexity di MariaDB?

A. `plugin_load_add=simple_password_check`
B. `simple_password_check=FORCE_PLUS_PERMANENT`
C. `simple_password_check_minimal_length=14`
D. `strict_password_validation=OFF`

**Jawaban benar:** A, B, C

**Pembahasan:** Password plugin dipaksa aktif dan minimal 14 karakter. strict_password_validation harus ON.

---
### Soal 54

**Pertanyaan:** Perintah SQL mana yang benar agar root lokal memakai `unix_socket`?

A. `ALTER USER 'root'@'localhost' IDENTIFIED VIA unix_socket;`
B. `SELECT User,Host,plugin FROM mysql.user WHERE User='root';`
C. `ALTER USER 'root'@'%' IDENTIFIED BY 'root';`
D. `CREATE USER 'root'@'%' IDENTIFIED BY '';`

**Jawaban benar:** A, B

**Pembahasan:** Root lokal dapat memakai unix_socket. Root wildcard/password kosong salah.

---
### Soal 55

**Pertanyaan:** Query manakah yang benar untuk audit authentication plugin lemah?

A. `SELECT User,host FROM mysql.user WHERE plugin IN('mysql_native_password', 'mysql_old_password','');`
B. `SHOW VARIABLES WHERE Variable_name = 'old_passwords';`
C. `SHOW VARIABLES WHERE Variable_name = 'secure_auth';`
D. `SELECT * FROM mysql.user WHERE user='';`

**Jawaban benar:** A, B, C, D

**Pembahasan:** A-C audit auth plugin/variabel; D audit anonymous account yang juga bagian authentication.

---
### Soal 56

**Pertanyaan:** Perintah SQL manakah yang benar untuk menggunakan plugin `ed25519` bila tersedia?

A. `INSTALL SONAME 'auth_ed25519';`
B. `ALTER USER 'app_user'@'10.10.10.20' IDENTIFIED VIA ed25519 USING PASSWORD('GantiPasswordSangatKuat');`
C. `SHOW PLUGINS;`
D. `ALTER USER 'app_user'@'%' IDENTIFIED VIA mysql_old_password;`

**Jawaban benar:** A, B, C

**Pembahasan:** ed25519 lebih kuat jika tersedia. mysql_old_password tidak sesuai.

---
### Soal 57

**Pertanyaan:** Perintah SQL manakah yang benar untuk menghapus anonymous account?

A. `SELECT user,host FROM mysql.user WHERE user = '';`
B. `DROP USER ''@'localhost';`
C. `DROP USER ''@'%';`
D. `CREATE USER ''@'%' IDENTIFIED BY '';`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit anonymous user lalu drop sesuai host hasil audit. Membuat anonymous user salah.

---
### Soal 58

**Pertanyaan:** Perintah SQL manakah yang benar untuk menghilangkan wildcard host pada user aplikasi?

A. `SELECT user, host FROM mysql.user WHERE host = '%';`
B. `CREATE USER 'app_user'@'10.10.10.20' IDENTIFIED BY 'GantiPasswordSangatKuat';`
C. `DROP USER 'app_user'@'%';`
D. `CREATE USER 'app_user'@'%' IDENTIFIED BY 'password';`

**Jawaban benar:** A, B, C

**Pembahasan:** Wildcard host diaudit lalu diganti host/IP spesifik. Membuat wildcard baru salah.

---
### Soal 59

**Pertanyaan:** Manakah perintah yang benar untuk menghapus password user database agar mudah login sesuai CIS?

A. `ALTER USER 'app_user'@'10.10.10.20' IDENTIFIED BY '';`
B. `UPDATE mysql.user SET authentication_string='' WHERE User='app_user';`
C. `CREATE USER 'anonymous'@'%' IDENTIFIED BY '';`
D. `SET PASSWORD FOR 'app_user'@'10.10.10.20' = PASSWORD('');`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: password kosong tidak sesuai prinsip CIS.

---
### Soal 60

**Pertanyaan:** Perintah mana yang benar untuk audit password tidak disimpan di global config?

A. `sudo grep -R -i 'password' /etc/mysql 2>/dev/null`
B. `sudo grep -R -i 'MYSQL_PWD' /root /home /etc/profile /etc/profile.d 2>/dev/null || true`
C. `Review hasil grep agar tidak ada password cleartext di global config`
D. `Tambahkan password aplikasi ke `/etc/mysql/mariadb.cnf``

**Jawaban benar:** A, B, C

**Pembahasan:** Audit config/profile diperlukan. Menambahkan password ke global config salah.

---

## 7. User, Privilege, dan Access Control

### Soal 61

**Pertanyaan:** Query manakah yang benar untuk audit full database access dan privilege sensitif?

A. `SELECT * FROM information_schema.user_privileges WHERE grantee NOT LIKE ('\'mysql.%localhost\'');`
B. `SELECT GRANTEE FROM INFORMATION_SCHEMA.USER_PRIVILEGES WHERE PRIVILEGE_TYPE = 'FILE';`
C. `SELECT GRANTEE FROM INFORMATION_SCHEMA.USER_PRIVILEGES WHERE PRIVILEGE_TYPE = 'SUPER';`
D. `GRANT ALL PRIVILEGES ON *.* TO 'app_user'@'%';`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit privilege luas/sensitif benar. Grant all ke app wildcard salah.

---
### Soal 62

**Pertanyaan:** Perintah SQL manakah yang benar untuk mencabut privilege sensitif dari user non-admin?

A. `REVOKE FILE ON *.* FROM 'app_user'@'10.10.10.20';`
B. `REVOKE PROCESS ON *.* FROM 'app_user'@'10.10.10.20';`
C. `REVOKE SUPER ON *.* FROM 'app_user'@'10.10.10.20';`
D. `GRANT SUPER ON *.* TO 'app_user'@'10.10.10.20';`

**Jawaban benar:** A, B, C

**Pembahasan:** FILE/PROCESS/SUPER harus dibatasi untuk non-admin. Grant SUPER salah.

---
### Soal 63

**Pertanyaan:** Perintah SQL manakah yang benar untuk mencabut privilege administratif lain dari user non-admin?

A. `REVOKE SHUTDOWN ON *.* FROM 'app_user'@'10.10.10.20';`
B. `REVOKE CREATE USER ON *.* FROM 'app_user'@'10.10.10.20';`
C. `REVOKE GRANT OPTION ON *.* FROM 'app_user'@'10.10.10.20';`
D. `GRANT GRANT OPTION ON *.* TO 'app_user'@'10.10.10.20';`

**Jawaban benar:** A, B, C

**Pembahasan:** SHUTDOWN, CREATE USER, GRANT OPTION sangat sensitif. Grant option salah.

---
### Soal 64

**Pertanyaan:** Query manakah yang benar untuk audit DML/DDL grants di database-level?

A. SELECT User,Host,Db FROM mysql.db WHERE Select_priv='Y' OR Insert_priv='Y' OR Update_priv='Y' OR Delete_priv='Y' OR Create_priv='Y' OR Drop_priv='Y' OR Alter_priv='Y';
B. `SELECT DISTINCT GRANTEE FROM INFORMATION_SCHEMA.USER_PRIVILEGES WHERE IS_GRANTABLE = 'YES';`
C. `SHOW GRANTS FOR 'app_user'@'10.10.10.20';`
D. `DROP DATABASE mysql;`

**Jawaban benar:** A, B, C

**Pembahasan:** A-C audit privilege. Menghapus database mysql destruktif fatal.

---
### Soal 65

**Pertanyaan:** Perintah SQL mana yang benar untuk grant least privilege pada user aplikasi?

A. `GRANT SELECT, INSERT, UPDATE, DELETE ON appdb.* TO 'app_user'@'10.10.10.20';`
B. `GRANT ALL PRIVILEGES ON *.* TO 'app_user'@'%';`
C. `CREATE USER 'app_user'@'10.10.10.20' IDENTIFIED BY 'GantiPasswordSangatKuat';`
D. `FLUSH PRIVILEGES;`

**Jawaban benar:** A, C, D

**Pembahasan:** Grant DB-level terbatas ke host spesifik benar. ALL global ke wildcard salah.

---
### Soal 66

**Pertanyaan:** Query manakah yang benar untuk review stored procedure/function dan DEFINER/INVOKER?

A. `SHOW PROCEDURE STATUS;`
B. `SHOW FUNCTION STATUS;`
C. `SELECT * FROM information_schema.routines;`
D. `SELECT * FROM information_schema.parameters;`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua membantu review routines, parameters, definer, dan kebutuhan privilege.

---
### Soal 67

**Pertanyaan:** Command manakah yang benar untuk memberi `SUPER` dan `GRANT OPTION` kepada user aplikasi biasa sesuai CIS?

A. `GRANT SUPER ON *.* TO 'app_user'@'10.10.10.20';`
B. `GRANT GRANT OPTION ON *.* TO 'app_user'@'10.10.10.20';`
C. `GRANT FILE ON *.* TO 'app_user'@'10.10.10.20';`
D. `GRANT PROCESS ON *.* TO 'app_user'@'10.10.10.20';`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: semua privilege tersebut sensitif dan tidak boleh diberikan ke user aplikasi biasa tanpa kebutuhan khusus/exception.

---

## 8. Auditing and Logging

### Soal 68

**Pertanyaan:** Directive logging mana yang benar dalam konfigurasi MariaDB CIS?

A. `log_error=/srv/mariadb/logs/mariadb.err`
B. `log_warnings=2`
C. `server_audit_logging=ON`
D. `server_audit_logging=OFF`

**Jawaban benar:** A, B, C

**Pembahasan:** Error log ke file, log_warnings=2, audit aktif. Audit OFF salah.

---
### Soal 69

**Pertanyaan:** Directive audit plugin mana yang benar agar plugin tidak mudah di-unload?

A. `plugin_load_add=server_audit`
B. `server_audit=FORCE_PLUS_PERMANENT`
C. `server_audit_events=CONNECT`
D. `server_audit=OFF`

**Jawaban benar:** A, B, C

**Pembahasan:** server_audit FORCE_PLUS_PERMANENT membuat audit lebih sulit dimatikan. OFF salah.

---
### Soal 70

**Pertanyaan:** Query manakah yang benar untuk validasi audit plugin MariaDB?

A. `SHOW VARIABLES LIKE '%audit%';`
B. `SELECT LOAD_OPTION FROM information_schema.plugins WHERE PLUGIN_NAME='SERVER_AUDIT';`
C. `SHOW PLUGINS;`
D. `UNINSTALL SONAME 'server_audit';`

**Jawaban benar:** A, B, C

**Pembahasan:** A-C audit status plugin. Uninstall audit plugin salah.

---
### Soal 71

**Pertanyaan:** Harapan yang benar dari validasi audit plugin adalah...

A. `server_audit_logging = ON`
B. `server_audit_events minimal mencatat CONNECT`
C. `LOAD_OPTION = FORCE_PLUS_PERMANENT`
D. `server_audit_logging = OFF`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit harus aktif, minimal event penting, dan tidak mudah dimatikan.

---
### Soal 72

**Pertanyaan:** Perintah mana yang benar untuk audit error log dan warning level?

A. `SHOW variables LIKE 'log_error';`
B. `SHOW GLOBAL VARIABLES LIKE 'log_warnings';`
C. `sudo tail -n 100 /srv/mariadb/logs/mariadb.err`
D. `SET GLOBAL log_warnings=0;`

**Jawaban benar:** A, B, C

**Pembahasan:** A-C audit/inspeksi. log_warnings=0 melemahkan logging.

---
### Soal 73

**Pertanyaan:** Perintah SQL manakah yang benar untuk memvalidasi encrypted binary/relay log?

A. SELECT VARIABLE_NAME, VARIABLE_VALUE, 'BINLOG - At Rest Encryption' as Note FROM information_schema.global_variables WHERE variable_name LIKE '%ENCRYPT_LOG%';
B. `SHOW VARIABLES LIKE '%encrypt%';`
C. `SHOW VARIABLES LIKE 'encrypt_binlog';`
D. `DELETE FROM mysql.general_log;`

**Jawaban benar:** A, B, C

**Pembahasan:** A-C audit enkripsi log. DELETE general log bukan validasi.

---
### Soal 74

**Pertanyaan:** Command mana yang benar untuk mematikan audit agar query admin tidak tercatat sesuai CIS?

A. `SET GLOBAL server_audit_logging=OFF;`
B. `UNINSTALL SONAME 'server_audit';`
C. `server_audit=OFF`
D. `systemctl stop mariadb`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: semua opsi bertentangan dengan CIS. Audit harus aktif dan tidak mudah dimatikan.

---

## 9. Network dan Connection Limits

### Soal 75

**Pertanyaan:** Directive network mana yang benar sesuai CIS MariaDB?

A. `bind_address=127.0.0.1`
B. `require_secure_transport=ON`
C. `tls_version=TLSv1.2,TLSv1.3`
D. `bind_address=*`

**Jawaban benar:** A, B, C

**Pembahasan:** Bind address harus terbatas, secure transport aktif, TLS versi modern.

---
### Soal 76

**Pertanyaan:** Query manakah yang benar untuk audit user remote wajib TLS?

A. `SELECT user, host, ssl_type FROM mysql.user WHERE NOT HOST IN ('::1', '127.0.0.1', 'localhost');`
B. `ALTER USER 'app_user'@'10.10.10.20' REQUIRE SSL;`
C. `ALTER USER 'app_user'@'10.10.10.20' REQUIRE X509;`
D. `ALTER USER 'app_user'@'%' REQUIRE NONE;`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit ssl_type lalu wajibkan SSL/X509. Wildcard require none salah.

---
### Soal 77

**Pertanyaan:** Perintah SQL mana yang benar untuk connection limits?

A. SELECT VARIABLE_NAME, VARIABLE_VALUE FROM information_schema.global_variables WHERE VARIABLE_NAME LIKE 'max_%connections';
B. `SET GLOBAL max_user_connections=50;`
C. `SET GLOBAL max_connections=300;`
D. `SET GLOBAL max_connections=0;`

**Jawaban benar:** A, B, C

**Pembahasan:** A audit, B/C remediation contoh. max_connections=0 bukan baseline.

---
### Soal 78

**Pertanyaan:** Perintah SQL mana yang benar untuk limit koneksi per user aplikasi?

A. `ALTER USER 'app_user'@'10.10.10.20' WITH MAX_CONNECTIONS_PER_HOUR 1000 MAX_USER_CONNECTIONS 20;`
B. SELECT user, host, max_connections, max_user_connections FROM mysql.user WHERE user NOT LIKE 'mysql.%' AND user NOT LIKE 'root';
C. `ALTER USER 'app_user'@'10.10.10.20' WITH MAX_USER_CONNECTIONS 0;`
D. `GRANT ALL ON *.* TO 'app_user'@'%';`

**Jawaban benar:** A, B

**Pembahasan:** A menetapkan limit, B audit. 0 berarti tidak terbatas, dan grant all wildcard salah.

---
### Soal 79

**Pertanyaan:** Command manakah yang benar untuk validasi port MariaDB yang listening?

A. `sudo ss -lntp | grep -E '3306|mariadbd|mysql' || true`
B. `SHOW VARIABLES LIKE 'port';`
C. `SELECT VARIABLE_NAME, VARIABLE_VALUE FROM information_schema.global_variables WHERE VARIABLE_NAME='bind_address';`
D. `ufw allow 3306/tcp from any`

**Jawaban benar:** A, B, C

**Pembahasan:** A-C audit. Membuka 3306 dari any tidak sesuai prinsip least exposure.

---
### Soal 80

**Pertanyaan:** Command firewall mana yang paling sesuai jika MariaDB harus diakses hanya dari app server `10.10.10.20` pada port 3306 di Ubuntu/Debian dengan UFW?

A. `sudo ufw allow from 10.10.10.20 to any port 3306 proto tcp`
B. `sudo ufw deny 3306/tcp`
C. `sudo ufw status verbose`
D. `sudo ufw allow 3306/tcp`

**Jawaban benar:** A, C

**Pembahasan:** Allow source spesifik dan audit UFW benar. Allow 3306 dari semua jaringan terlalu luas; deny bisa memutus kebutuhan aplikasi jika belum ada rule tepat.

---
### Soal 81

**Pertanyaan:** Perintah mana yang benar untuk membuka MariaDB ke semua interface dan semua jaringan sesuai CIS?

A. `bind_address=*`
B. `sudo ufw allow 3306/tcp`
C. `ALTER USER 'app_user'@'%' REQUIRE NONE;`
D. `require_secure_transport=OFF`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: semua opsi melemahkan network security.

---

## 10. Replication Security

### Soal 82

**Pertanyaan:** Perintah SQL mana yang benar untuk mengaktifkan TLS pada replication traffic?

A. `STOP REPLICA;`
B. `CHANGE MASTER TO MASTER_SSL=1;`
C. `START REPLICA;`
D. `CHANGE MASTER TO MASTER_SSL=0;`

**Jawaban benar:** A, B, C

**Pembahasan:** TLS replication diaktifkan dengan MASTER_SSL=1 di antara stop/start replica.

---
### Soal 83

**Pertanyaan:** Perintah SQL mana yang benar agar replica memverifikasi certificate primary?

A. `SHOW REPLICA STATUS\G;`
B. `STOP REPLICA;`
C. `CHANGE MASTER TO MASTER_SSL_VERIFY_SERVER_CERT=1;`
D. `START REPLICA;`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Audit status dan set verify server certificate benar.

---
### Soal 84

**Pertanyaan:** Perintah SQL mana yang benar untuk audit dan remediation replication user agar tidak memiliki `SUPER`?

A. `SELECT user, host FROM mysql.user WHERE user='repl' AND Super_priv = 'Y';`
B. `REVOKE SUPER ON *.* FROM 'repl'@'replica1.example.com';`
C. `GRANT SUPER ON *.* TO 'repl'@'replica1.example.com';`
D. `SHOW GRANTS FOR 'repl'@'replica1.example.com';`

**Jawaban benar:** A, B, D

**Pembahasan:** Audit privilege dan revoke SUPER benar. Grant SUPER salah.

---
### Soal 85

**Pertanyaan:** Perintah SQL mana yang benar untuk menetapkan cipher TLS replication yang disetujui?

A. `STOP REPLICA;`
B. `CHANGE MASTER TO MASTER_SSL_CIPHER='ECDHE-ECDSA-AES128-GCM-SHA256';`
C. `START REPLICA;`
D. `CHANGE MASTER TO MASTER_SSL_CIPHER='NULL-MD5';`

**Jawaban benar:** A, B, C

**Pembahasan:** Cipher kuat diset saat replication dihentikan lalu start. NULL-MD5 lemah/tidak sesuai.

---
### Soal 86

**Pertanyaan:** Perintah SQL mana yang benar untuk mutual TLS replication?

A. `CHANGE MASTER TO MASTER_SSL_CERT='/etc/mysql/ssl/client-cert.pem', MASTER_SSL_KEY='/etc/mysql/ssl/client-key.pem';`
B. `ALTER USER 'repl'@'replica1.example.com' REQUIRE X509;`
C. `SHOW REPLICA STATUS\G;`
D. `ALTER USER 'repl'@'%' REQUIRE NONE;`

**Jawaban benar:** A, B, C

**Pembahasan:** Replica memakai client cert/key dan primary mewajibkan X509. Wildcard require none salah.

---
### Soal 87

**Pertanyaan:** Jika replication tidak digunakan pada server MariaDB, jawaban yang paling tepat untuk kontrol Bab 9 adalah...

A. `Tandai Not Applicable / Exception sesuai dokumentasi`
B. `Jangan membuat user repl tanpa kebutuhan`
C. `Jangan membuka port replication tambahan`
D. `Aktifkan Swarm overlay network`

**Jawaban benar:** A, B, C

**Pembahasan:** Jika tidak ada replication, kontrol replication didokumentasikan N/A dan jangan membuat attack surface baru.

---
### Soal 88

**Pertanyaan:** Command mana yang benar untuk membuat replication user wildcard dengan SUPER dan tanpa TLS sesuai CIS?

A. `CREATE USER 'repl'@'%' IDENTIFIED BY 'password';`
B. `GRANT REPLICATION SLAVE, SUPER ON *.* TO 'repl'@'%';`
C. `ALTER USER 'repl'@'%' REQUIRE NONE;`
D. `CHANGE MASTER TO MASTER_SSL=0;`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: semua opsi tidak sesuai hardening replication.

---

## 11. Validasi Akhir dan Exception

### Soal 89

**Pertanyaan:** Command gabungan mana yang benar untuk validasi akhir service dan port MariaDB?

A. `sudo systemctl status mariadb --no-pager`
B. `sudo ss -lntp | grep -E '3306|mariadbd|mysql' || true`
C. `sudo ls -ld /srv/mariadb/data /srv/mariadb/logs /srv/mariadb/import /srv/mariadb/backups`
D. `sudo chmod -R 777 /srv/mariadb`

**Jawaban benar:** A, B, C

**Pembahasan:** A-C validasi status, port, dan permission. chmod 777 salah.

---
### Soal 90

**Pertanyaan:** SQL audit akhir manakah yang benar untuk variabel penting MariaDB?

A. SHOW VARIABLES WHERE Variable_name IN ('old_passwords','secure_auth','have_ssl','require_secure_transport','tls_version','ssl_cipher','bind_address','default_password_lifetime','log_error','log_warnings','secure_file_priv','sql_mode');
B. `SHOW VARIABLES LIKE '%audit%';`
C. `SHOW PLUGINS;`
D. `DROP USER root@localhost;`

**Jawaban benar:** A, B, C

**Pembahasan:** A-C audit konfigurasi dan plugin. Drop root berbahaya.

---
### Soal 91

**Pertanyaan:** SQL audit akhir mana yang benar untuk user, host, TLS, dan privilege?

A. `SELECT user,host FROM mysql.user WHERE user = '';`
B. `SELECT user,host FROM mysql.user WHERE host = '%';`
C. SELECT GRANTEE FROM INFORMATION_SCHEMA.USER_PRIVILEGES WHERE PRIVILEGE_TYPE IN ('FILE','PROCESS','SUPER','SHUTDOWN','CREATE USER','REPLICATION SLAVE');
D. `SELECT DISTINCT GRANTEE FROM INFORMATION_SCHEMA.USER_PRIVILEGES WHERE IS_GRANTABLE = 'YES';`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua query berguna untuk audit akun anonymous, wildcard host, privilege sensitif, dan grant option.

---
### Soal 92

**Pertanyaan:** Manakah item yang benar untuk checklist implementasi CIS MariaDB?

A. `MariaDB 10.6 terpasang`
B. `User `mysql` tidak punya interactive shell`
C. ``require_secure_transport=ON` dan `have_ssl=YES``
D. `Anonymous account dan wildcard host dibiarkan`

**Jawaban benar:** A, B, C

**Pembahasan:** Anonymous account dan wildcard host harus dihapus/dibatasi.

---
### Soal 93

**Pertanyaan:** Manakah contoh exception register yang benar untuk MariaDB?

A. `bind_address tidak local-only karena aplikasi remote perlu koneksi; mitigasi firewall + TLS + user host spesifik`
B. `general_log aktif sementara untuk debug; mitigasi batas waktu, permission ketat, review log`
C. `Semua rule tidak diterapkan tanpa alasan`
D. `Exception tidak perlu review date`

**Jawaban benar:** A, B

**Pembahasan:** Exception harus punya alasan, risiko, mitigasi, owner, dan review date.

---
### Soal 94

**Pertanyaan:** Urutan implementasi mana yang benar dan aman?

A. `Backup/snapshot terlebih dahulu`
B. `Uji di lab sebelum produksi`
C. `Terapkan TLS/auth/privilege secara bertahap dan validasi aplikasi`
D. `Langsung ubah semua di production tanpa rollback plan`

**Jawaban benar:** A, B, C

**Pembahasan:** Hardening database harus bertahap dan memiliki rollback plan.

---
### Soal 95

**Pertanyaan:** Command mana yang benar untuk menjalankan validasi akhir TLS dan audit plugin sekaligus?

A. `SHOW VARIABLES WHERE Variable_name IN ('have_ssl','require_secure_transport','tls_version');`
B. `SELECT LOAD_OPTION FROM information_schema.plugins WHERE PLUGIN_NAME='SERVER_AUDIT';`
C. `SHOW VARIABLES LIKE '%audit%';`
D. `SET GLOBAL require_secure_transport=OFF;`

**Jawaban benar:** A, B, C

**Pembahasan:** A-C audit. Mematikan secure transport salah.

---

## 12. Soal Campuran Praktik Detail

### Soal 96

**Pertanyaan:** Manakah baris konfigurasi yang benar untuk membatasi `secure_file_priv` pada direktori import khusus?

A. `secure_file_priv=/srv/mariadb/import`
B. `sudo mkdir -p /srv/mariadb/import && sudo chown mysql:mysql /srv/mariadb/import && sudo chmod 750 /srv/mariadb/import`
C. `secure_file_priv=`
D. `secure_file_priv=/`

**Jawaban benar:** A, B

**Pembahasan:** secure_file_priv harus diarahkan ke direktori spesifik yang permission-nya ketat. Kosong atau root filesystem tidak aman.

---
### Soal 97

**Pertanyaan:** Manakah command yang benar untuk memastikan MariaDB tidak menggunakan `skip-grant-tables`?

A. `grep -R 'skip-grant-tables' /etc/mysql /etc/my.cnf* 2>/dev/null || true`
B. `SHOW VARIABLES LIKE 'skip_grant_tables';`
C. `Pastikan konfigurasi memuat `skip-grant-tables=FALSE` atau tidak mengaktifkan opsi tersebut`
D. `Tambahkan `skip-grant-tables` agar root mudah recovery setiap saat`

**Jawaban benar:** A, B, C

**Pembahasan:** skip-grant-tables hanya untuk recovery khusus, bukan konfigurasi produksi.

---
### Soal 98

**Pertanyaan:** Manakah command yang benar untuk memeriksa dan mengatur `log_warnings=2`?

A. `SHOW GLOBAL VARIABLES LIKE 'log_warnings';`
B. `SET GLOBAL log_warnings=2;`
C. `Tambahkan `log_warnings=2` di `/etc/mysql/mariadb.conf.d/60-cis-hardening.cnf``
D. `SET GLOBAL log_warnings=0;`

**Jawaban benar:** A, B, C

**Pembahasan:** log_warnings=2 sesuai benchmark. 0 mengurangi visibility.

---
### Soal 99

**Pertanyaan:** Manakah perintah yang benar untuk membuat client certificate MariaDB untuk X.509/mTLS?

A. sudo openssl req -newkey rsa:4096 -days 3650 -nodes -keyout client-key.pem -out client-req.pem -subj "/CN=mariadb-client"
B. `sudo openssl x509 -req -in client-req.pem -days 3650 -CA ca.pem -CAkey ca-key.pem -set_serial 02 -out client-cert.pem`
C. `ALTER USER 'app_user'@'10.10.10.20' REQUIRE X509;`
D. `chmod 777 client-key.pem`

**Jawaban benar:** A, B, C

**Pembahasan:** A/B membuat sertifikat, C mewajibkan X509. Private key tidak boleh 777.

---
### Soal 100

**Pertanyaan:** Manakah command yang benar untuk membuat konfigurasi MariaDB gagal lebih aman saat JSON tidak relevan? (Pertanyaan jebakan)

A. `jq . /etc/mysql/mariadb.conf.d/60-cis-hardening.cnf`
B. `mariadb --validate-json /etc/mysql/mariadb.conf.d/60-cis-hardening.cnf`
C. `docker info --format '{{ .SecurityOptions }}'`
D. `kubectl apply -f 60-cis-hardening.cnf`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: file MariaDB `.cnf` bukan JSON dan bukan Kubernetes/Docker config.

---
### Soal 101

**Pertanyaan:** Manakah command yang benar untuk memvalidasi bahwa MariaDB memakai datadir baru setelah restart?

A. `sudo mariadb -e "SHOW VARIABLES WHERE variable_name = 'datadir';"`
B. `sudo ls -ld /srv/mariadb/data`
C. `sudo ss -lntp | grep -E '3306|mariadbd|mysql' || true`
D. `cat /etc/passwd | grep '^datadir:'`

**Jawaban benar:** A, B, C

**Pembahasan:** A validasi variabel, B permission, C service listening. datadir bukan user di passwd.

---
### Soal 102

**Pertanyaan:** Manakah command yang benar untuk memastikan MariaDB audit log tidak world-readable/writable?

A. `stat -c '%U:%G %a %n' /srv/mariadb/logs/server_audit.log`
B. `sudo chmod 660 /srv/mariadb/logs/server_audit.log`
C. `sudo chown mysql:mysql /srv/mariadb/logs/server_audit.log`
D. `sudo chmod 777 /srv/mariadb/logs/server_audit.log`

**Jawaban benar:** A, B, C

**Pembahasan:** Mode 660 dan owner mysql:mysql sesuai contoh; 777 salah.

---
### Soal 103

**Pertanyaan:** Manakah command SQL yang benar untuk mewajibkan TLS, tetapi bukan X.509, bagi remote user aplikasi?

A. `ALTER USER 'app_user'@'10.10.10.20' REQUIRE SSL;`
B. `SELECT user, host, ssl_type FROM mysql.user WHERE NOT HOST IN ('::1', '127.0.0.1', 'localhost');`
C. `ALTER USER 'app_user'@'10.10.10.20' REQUIRE X509;`
D. `ALTER USER 'app_user'@'10.10.10.20' REQUIRE NONE;`

**Jawaban benar:** A, B

**Pembahasan:** REQUIRE SSL mewajibkan TLS biasa. REQUIRE X509 lebih kuat tapi tidak sesuai pertanyaan 'bukan X.509'.

---
### Soal 104

**Pertanyaan:** Manakah command yang benar untuk mempersiapkan rollback sebelum hardening MariaDB?

A. `Buat snapshot VM jika tersedia`
B. `sudo cp -a /etc/mysql /root/pre-cis-backup/mysql-etc-$(date +%F) 2>/dev/null || true`
C. `Catat `systemctl status mariadb --no-pager` sebelum perubahan`
D. `Hapus backup lama tanpa verifikasi`

**Jawaban benar:** A, B, C

**Pembahasan:** Snapshot/backup/status mempermudah rollback. Menghapus backup tanpa verifikasi salah.

---
### Soal 105

**Pertanyaan:** Manakah command yang benar untuk mengaktifkan plugin `server_audit` secara sementara tanpa force permanent sesuai CIS produksi?

A. `INSTALL SONAME 'server_audit';`
B. `server_audit_logging=ON`
C. `server_audit=FORCE_PLUS_PERMANENT`
D. `server_audit=OFF`

**Jawaban benar:** A, B, C

**Pembahasan:** Pada produksi CIS, plugin harus aktif dan FORCE_PLUS_PERMANENT. Kata 'sementara' tidak ideal, tetapi opsi A/B/C masih benar sebagai komponen implementasi; D salah.

---
### Soal 106

**Pertanyaan:** Manakah command yang benar untuk memastikan MariaDB tidak menyimpan log pada partisi sistem secara tidak terkendali?

A. `log_error=/srv/mariadb/logs/mariadb.err`
B. `server_audit_file_path=/srv/mariadb/logs/server_audit.log`
C. `sudo ls -ld /srv/mariadb/logs`
D. `log_error=/var/log/syslog-only-no-file`

**Jawaban benar:** A, B, C

**Pembahasan:** Log diarahkan ke direktori khusus dan divalidasi. Opsi D tidak sesuai contoh file log MariaDB.

---
### Soal 107

**Pertanyaan:** Command manakah yang benar untuk membuat database server multi-use dengan banyak service supaya mudah dikelola sesuai CIS?

A. `sudo apt install apache2 postfix samba bind9 mariadb-server`
B. `sudo systemctl enable --now apache2 postfix smbd bind9 mariadb`
C. `sudo ufw allow 80,443,25,445,53,3306/tcp`
D. `Gabungkan semua service produksi di satu host tanpa dokumentasi`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: CIS mendorong dedicated machine dan service minimization, bukan multi-use tanpa kebutuhan.

---
### Soal 108

**Pertanyaan:** Manakah command yang benar untuk mengaudit apakah user remote masih memakai wildcard host dan tidak memakai TLS?

A. `SELECT user, host FROM mysql.user WHERE host = '%';`
B. `SELECT user, host, ssl_type FROM mysql.user WHERE NOT HOST IN ('::1', '127.0.0.1', 'localhost');`
C. `SHOW GRANTS FOR 'app_user'@'%';`
D. `ALTER USER 'app_user'@'%' REQUIRE NONE;`

**Jawaban benar:** A, B, C

**Pembahasan:** A-C audit. REQUIRE NONE melemahkan TLS.

---
### Soal 109

**Pertanyaan:** Manakah command yang benar untuk mengurangi risiko user non-admin membaca file server melalui MariaDB?

A. `REVOKE FILE ON *.* FROM 'app_user'@'10.10.10.20';`
B. `secure_file_priv=/srv/mariadb/import`
C. `sudo chmod 750 /srv/mariadb/import`
D. `GRANT FILE ON *.* TO 'app_user'@'%';`

**Jawaban benar:** A, B, C

**Pembahasan:** FILE privilege dicabut, secure_file_priv dibatasi, direktori import ketat. Grant FILE wildcard salah.

---
### Soal 110

**Pertanyaan:** Manakah command yang benar untuk memastikan root OS dapat login MariaDB secara lokal tanpa menyimpan password root database?

A. `ALTER USER 'root'@'localhost' IDENTIFIED VIA unix_socket;`
B. `sudo mariadb`
C. `Simpan password root di `/etc/mysql/my.cnf``
D. `export MYSQL_PWD=rootpassword`

**Jawaban benar:** A, B

**Pembahasan:** unix_socket dan sudo mariadb menghindari password di file/global env. Opsi C/D salah.

---
### Soal 111

**Pertanyaan:** Manakah command yang benar untuk validasi `server_audit` tidak bisa di-unload dengan mudah?

A. `SELECT LOAD_OPTION FROM information_schema.plugins WHERE PLUGIN_NAME='SERVER_AUDIT';`
B. `Harapan output: FORCE_PLUS_PERMANENT`
C. `SHOW VARIABLES LIKE '%audit%';`
D. `UNINSTALL PLUGIN SERVER_AUDIT;`

**Jawaban benar:** A, B, C

**Pembahasan:** LOAD_OPTION harus FORCE_PLUS_PERMANENT. Uninstall plugin salah.

---
### Soal 112

**Pertanyaan:** Manakah command yang benar untuk operasi impor/ekspor file yang dibatasi oleh `secure_file_priv`?

A. `Pastikan direktori `/srv/mariadb/import` ada dan dimiliki mysql:mysql`
B. `Pastikan `secure_file_priv=/srv/mariadb/import``
C. `Letakkan file import yang disetujui di `/srv/mariadb/import` dengan permission sesuai`
D. `Set `secure_file_priv=/` agar semua path bisa diakses`

**Jawaban benar:** A, B, C

**Pembahasan:** Direktori import khusus membatasi akses file. Root filesystem sebagai secure_file_priv tidak aman.

---
### Soal 113

**Pertanyaan:** Manakah command yang benar untuk membatasi efek DoS akibat koneksi database terlalu banyak?

A. `max_connections=300`
B. `max_user_connections=50`
C. `ALTER USER 'app_user'@'10.10.10.20' WITH MAX_USER_CONNECTIONS 20;`
D. `max_connections=999999999`

**Jawaban benar:** A, B, C

**Pembahasan:** Connection limit membantu mengurangi resource exhaustion. Nilai sangat besar tidak sesuai baseline.

---
### Soal 114

**Pertanyaan:** Manakah command yang benar untuk menerapkan perubahan konfigurasi MariaDB setelah edit file `.cnf`?

A. `sudo systemctl daemon-reload jika systemd unit/override berubah`
B. `sudo systemctl restart mariadb`
C. `sudo systemctl status mariadb --no-pager`
D. `sudo reboot -f tanpa validasi`

**Jawaban benar:** A, B, C

**Pembahasan:** daemon-reload diperlukan untuk override systemd; restart dan status untuk validasi. Reboot paksa tidak ideal.

---
### Soal 115

**Pertanyaan:** Manakah command yang benar untuk mengecek service MariaDB local-only jika bind_address 127.0.0.1?

A. `sudo ss -lntp | grep ':3306'`
B. `SHOW VARIABLES LIKE 'bind_address';`
C. `Pastikan output listening pada 127.0.0.1:3306, bukan 0.0.0.0:3306`
D. `Pastikan output listening pada *:3306 agar semua host mudah konek`

**Jawaban benar:** A, B, C

**Pembahasan:** Untuk local-only, bind ke 127.0.0.1, bukan semua interface.

---
### Soal 116

**Pertanyaan:** Manakah command yang benar untuk menandai pengecualian jika aplikasi lama belum mendukung TLSv1.2/TLSv1.3?

A. `Catat exception dengan alasan, risiko, mitigasi, owner, dan review date`
B. `Rencanakan upgrade client agar mendukung TLS modern`
C. `Sementara gunakan segmentasi jaringan/firewall ketat jika benar-benar disetujui`
D. `Aktifkan TLSv1 dan TLSv1.1 permanen tanpa dokumentasi`

**Jawaban benar:** A, B, C

**Pembahasan:** Exception harus terdokumentasi dan ada mitigasi. TLS lama permanen tanpa dokumentasi salah.

---
### Soal 117

**Pertanyaan:** Manakah command yang benar untuk memilih cipher yang kuat pada konfigurasi TLS MariaDB sesuai contoh?

A. `ssl_cipher=ECDHE-ECDSA-AES128-GCM-SHA256`
B. `tls_version=TLSv1.2,TLSv1.3`
C. `require_secure_transport=ON`
D. `ssl_cipher=NULL`

**Jawaban benar:** A, B, C

**Pembahasan:** Cipher kuat dan TLS modern sesuai hardening. NULL cipher tidak aman.

---
### Soal 118

**Pertanyaan:** Perintah manakah yang benar untuk mengaktifkan `local_infile` dan `skip-grant-tables` supaya aplikasi lama tidak error sesuai CIS?

A. `local-infile=1`
B. `skip-grant-tables`
C. `SET GLOBAL local_infile=1;`
D. `--skip-grant-tables saat service normal`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: semua opsi bertentangan dengan baseline hardening kecuali ada recovery/exception sangat khusus.

---

## 13. Simulasi Ujian Praktik

### Soal 119

**Pertanyaan:** MariaDB berada di `/srv/mariadb/data`, tetapi AppArmor memblokir start. Opsi mana yang benar?

A. `Tambahkan rule `/srv/mariadb/data/** rwk,` pada profile MariaDB`
B. `Reload AppArmor atau jalankan `apparmor_parser -r` pada profile`
C. `Cek `journalctl -u mariadb -xe --no-pager``
D. `Matikan AppArmor permanen tanpa dokumentasi`

**Jawaban benar:** A, B, C

**Pembahasan:** Sesuaikan profile dan validasi log. Mematikan AppArmor permanen bukan jawaban hardening.

---
### Soal 120

**Pertanyaan:** Aplikasi remote perlu user database. Pilihan mana yang paling sesuai prinsip CIS?

A. `CREATE USER 'app_user'@'10.10.10.20' IDENTIFIED BY 'GantiPasswordSangatKuat';`
B. `GRANT SELECT, INSERT, UPDATE, DELETE ON appdb.* TO 'app_user'@'10.10.10.20';`
C. `ALTER USER 'app_user'@'10.10.10.20' REQUIRE SSL;`
D. `CREATE USER 'app_user'@'%' IDENTIFIED BY '123'; GRANT ALL ON *.* TO 'app_user'@'%';`

**Jawaban benar:** A, B, C

**Pembahasan:** User host spesifik, privilege terbatas, dan TLS wajib. Wildcard/ALL/password lemah salah.

---
### Soal 121

**Pertanyaan:** DBA ingin membuka akses `PROCESS` untuk developer agar bisa melihat query semua user. Apa command yang sesuai CIS?

A. `SELECT GRANTEE FROM INFORMATION_SCHEMA.USER_PRIVILEGES WHERE PRIVILEGE_TYPE = 'PROCESS';`
B. `Dokumentasikan exception bila benar-benar diperlukan`
C. `Berikan hanya pada role/admin tepercaya jika disetujui`
D. `GRANT PROCESS ON *.* TO 'developer'@'%'; tanpa review`

**Jawaban benar:** A, B, C

**Pembahasan:** PROCESS sensitif karena bisa membocorkan query user lain; perlu review/exception.

---
### Soal 122

**Pertanyaan:** Akun backup membutuhkan akses. Pilihan mana yang benar secara prinsip?

A. `Beri privilege minimal yang dibutuhkan untuk backup sesuai tool dan dokumentasikan`
B. `Simpan credential backup di secret manager atau file 0600`
C. `Pastikan backup file terenkripsi dan permission ketat`
D. `Simpan password backup di command line cron`

**Jawaban benar:** A, B, C

**Pembahasan:** Backup credential sangat sensitif. Password di command line cron salah.

---
### Soal 123

**Pertanyaan:** Manakah command yang benar untuk membuat audit SQL melihat user dengan `GRANT OPTION`?

A. `SELECT DISTINCT GRANTEE FROM INFORMATION_SCHEMA.USER_PRIVILEGES WHERE IS_GRANTABLE = 'YES';`
B. `SHOW GRANTS FOR 'app_user'@'10.10.10.20';`
C. `REVOKE GRANT OPTION ON *.* FROM 'app_user'@'10.10.10.20'; jika tidak diperlukan`
D. `GRANT GRANT OPTION ON *.* TO 'app_user'@'%';`

**Jawaban benar:** A, B, C

**Pembahasan:** Grant option bisa menjadi eskalasi privilege; audit dan revoke bila tidak perlu.

---
### Soal 124

**Pertanyaan:** Manakah command yang benar untuk mengecek dan menghapus anonymous user secara hati-hati?

A. `SELECT user,host FROM mysql.user WHERE user = '';`
B. `DROP USER ''@'localhost';`
C. `DROP USER ''@'%';`
D. `DROP USER ''@'host-yang-muncul-di-hasil-audit';`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Host anonymous perlu disesuaikan dengan hasil audit. DROP USER harus diarahkan ke entry yang benar.

---
### Soal 125

**Pertanyaan:** Manakah command yang benar untuk audit keamanan runtime MariaDB dari sisi OS?

A. `ps -ef | grep -E 'mariadbd|mysqld' | grep -v grep`
B. `getent passwd mysql`
C. `sudo ss -lntp | grep -E '3306|mariadbd|mysql' || true`
D. `sudo chmod 777 /var/run/mysqld`

**Jawaban benar:** A, B, C

**Pembahasan:** A-C audit proses, user, dan port. chmod 777 runtime dir salah.

---
### Soal 126

**Pertanyaan:** Command mana yang benar untuk memastikan konfigurasi password complexity tidak membuat aplikasi langsung rusak?

A. `Uji perubahan di lab/staging`
B. `Uji rotasi password akun aplikasi secara terkontrol`
C. `Siapkan rollback plan dan exception register`
D. `Terapkan langsung ke produksi tanpa koordinasi`

**Jawaban benar:** A, B, C

**Pembahasan:** Password policy dapat memengaruhi aplikasi/service account, jadi perlu uji dan rollback.

---
### Soal 127

**Pertanyaan:** Manakah command/SQL yang benar untuk menguji `require_secure_transport=ON`?

A. `SELECT @@require_secure_transport;`
B. `SHOW variables WHERE variable_name = 'have_ssl';`
C. `Uji koneksi client tanpa TLS dan pastikan ditolak sesuai kebijakan`
D. `SET GLOBAL require_secure_transport=OFF untuk memudahkan koneksi`

**Jawaban benar:** A, B, C

**Pembahasan:** Validasi mencakup variable dan uji koneksi. Mematikan secure transport salah.

---
### Soal 128

**Pertanyaan:** Manakah command yang benar untuk menjaga password tidak masuk history MariaDB?

A. `Arahkan `/root/.mysql_history` ke `/dev/null``
B. `Arahkan `/root/.mariadb_history` ke `/dev/null``
C. `Hindari mengetik password dalam query/command line`
D. `Simpan semua password di query history agar mudah audit`

**Jawaban benar:** A, B, C

**Pembahasan:** History harus diminimalkan untuk secret. Menyimpan password di history salah.

---
