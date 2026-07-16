# 08 — Bank Soal Praktik Command CIS Apache HTTP Server 2.4 — Lengkap

**Fokus:** latihan pilihan ganda jamak berbasis praktik command, audit, remediation, konfigurasi file Apache, dan validasi akhir.

**Basis materi:** dua file konseptual CIS Apache HTTP Server 2.4 yang sudah dibuat sebelumnya serta file implementasi hardening Apache.

> **Aturan latihan:** pilih semua jawaban yang benar. Jika tidak ada opsi yang benar, jawab **PASS / kosongkan jawaban**. Jangan menjalankan command destruktif di sistem asli. Gunakan VM/lab.

## Daftar Cakupan

- Planning and Installation
- Minimize Apache Modules
- Principles, Permissions, and Ownership
- Apache Access Control
- Minimize Features, Content, and Options
- Logging, Monitoring, and Maintenance
- SSL/TLS Configuration
- Information Leakage
- Denial of Service Mitigations
- Request Limits
- SELinux dan AppArmor
- Validasi akhir dan exception
- Bonus adaptasi firewalld


## 0. Cara Menjawab Soal Praktik

### Soal 1

**Pertanyaan:** Dalam latihan pilihan ganda jamak, apa jawaban yang benar jika semua opsi command salah atau tidak sesuai konteks?

A. Pilih opsi yang paling mendekati benar
B. Pilih semua opsi agar tidak kosong
C. `PASS / kosongkan jawaban`
D. Pilih A sebagai default

**Jawaban benar:** C

**Pembahasan:** Jika tidak ada command yang benar, jawaban harus PASS/kosong.

---


## 1. Planning and Installation

### Soal 2

**Pertanyaan:** Command mana yang benar untuk memasang dan mengaktifkan Apache pada Ubuntu/Debian?

A. `apt update`
B. `apt install -y apache2`
C. `systemctl enable --now apache2.service`
D. `systemctl enable --now httpd.service`

**Jawaban benar:** A, B, C

**Pembahasan:** Pada Ubuntu/Debian service Apache bernama apache2.service, bukan httpd.service.

---

### Soal 3

**Pertanyaan:** Command mana yang benar untuk validasi versi dan build Apache?

A. `apache2 -v`
B. `apache2ctl -V`
C. `httpd -V`
D. `apache2ctl -M`

**Jawaban benar:** A, B, D

**Pembahasan:** apache2 -v/V dan apache2ctl relevan di Debian/Ubuntu; httpd biasanya RHEL-family.

---

### Soal 4

**Pertanyaan:** Command mana yang benar untuk melihat apakah server Apache juga menjalankan service lain yang memperbesar attack surface?

A. `ss -tulpen`
B. `systemctl list-units --type=service --state=running`
C. `dpkg -l | egrep 'bind9|samba|vsftpd|postfix|mysql|mariadb|postgresql|squid' || true`
D. `chmod -R 777 /etc/apache2`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit port, service, dan paket membantu mendeteksi multi-use server. chmod 777 jelas salah.

---

### Soal 5

**Pertanyaan:** Server Apache produksi tidak boleh merangkap DNS/Samba/FTP tanpa alasan. Command mana yang bisa dipakai jika layanan tersebut tidak diperlukan?

A. `systemctl disable --now bind9.service 2>/dev/null || true`
B. `systemctl disable --now smbd.service nmbd.service 2>/dev/null || true`
C. `apt purge -y bind9 samba vsftpd proftpd-basic 2>/dev/null || true`
D. `ufw allow 21/tcp`

**Jawaban benar:** A, B, C

**Pembahasan:** Disable/purge layanan tidak perlu. Membuka port FTP bukan hardening.

---

### Soal 6

**Pertanyaan:** Command mana yang benar untuk membuat backup konfigurasi Apache sebelum hardening?

A. `mkdir -p /root/apache-hardening-backup/$(date +%F)`
B. `cp -a /etc/apache2 /root/apache-hardening-backup/$(date +%F)/apache2`
C. `cp -a /var/www /root/apache-hardening-backup/$(date +%F)/var-www 2>/dev/null || true`
D. `rm -rf /etc/apache2`

**Jawaban benar:** A, B, C

**Pembahasan:** Backup wajib sebelum remediation; menghapus /etc/apache2 merusak konfigurasi.

---


## 2. Minimize Apache Modules

### Soal 7

**Pertanyaan:** Command mana yang benar untuk audit modul Apache aktif?

A. `apache2ctl -M`
B. `apache2ctl -M | sort`
C. `ls -1 /etc/apache2/mods-enabled/`
D. `a2dismod all`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit modul dilakukan dengan apache2ctl -M dan mods-enabled.

---

### Soal 8

**Pertanyaan:** Command mana yang benar untuk menonaktifkan WebDAV jika tidak dipakai?

A. `a2dismod dav dav_fs dav_lock 2>/dev/null || true`
B. `apache2ctl configtest`
C. `systemctl reload apache2.service`
D. `a2enmod dav dav_fs`

**Jawaban benar:** A, B, C

**Pembahasan:** WebDAV dimatikan dengan a2dismod, lalu validasi dan reload.

---

### Soal 9

**Pertanyaan:** Command mana yang benar untuk menonaktifkan modul status/info yang tidak diperlukan?

A. `a2dismod status info 2>/dev/null || true`
B. `apache2ctl -M | egrep 'status|info' || true`
C. `systemctl reload apache2.service`
D. `a2enmod status info`

**Jawaban benar:** A, B, C

**Pembahasan:** status/info dapat membocorkan informasi operasional jika terbuka.

---

### Soal 10

**Pertanyaan:** Command mana yang benar untuk menonaktifkan autoindex agar directory listing tidak aktif?

A. `a2dismod autoindex 2>/dev/null || true`
B. `apache2ctl configtest`
C. `systemctl reload apache2.service`
D. `a2enmod autoindex`

**Jawaban benar:** A, B, C

**Pembahasan:** autoindex membuat directory listing; disable jika tidak dibutuhkan.

---

### Soal 11

**Pertanyaan:** Apache bukan reverse proxy. Command mana yang tepat untuk disable modul proxy yang tidak diperlukan?

A. `a2dismod proxy proxy_http proxy_fcgi proxy_ajp proxy_balancer proxy_connect proxy_wstunnel 2>/dev/null || true`
B. `apache2ctl -M | grep proxy || true`
C. `systemctl reload apache2.service`
D. `a2enmod proxy proxy_http proxy_connect`

**Jawaban benar:** A, B, C

**Pembahasan:** Jika bukan reverse proxy, modul proxy tidak perlu aktif.

---

### Soal 12

**Pertanyaan:** Apache memang reverse proxy produksi. Opsi mana yang benar menurut prinsip CIS?

A. Disable seluruh proxy module tanpa uji
B. Aktifkan hanya modul proxy yang benar-benar dipakai
C. Dokumentasikan exception dan alasan bisnis
D. Aktifkan forward proxy terbuka untuk semua client

**Jawaban benar:** B, C

**Pembahasan:** Proxy boleh aktif jika fungsi server memang reverse proxy, tetapi harus dibatasi dan didokumentasikan.

---

### Soal 13

**Pertanyaan:** Command mana yang benar untuk menonaktifkan UserDir jika konten tidak boleh disajikan dari home user?

A. `a2dismod userdir 2>/dev/null || true`
B. `apache2ctl -M | grep userdir || true`
C. `systemctl reload apache2.service`
D. `a2enmod userdir`

**Jawaban benar:** A, B, C

**Pembahasan:** UserDir memungkinkan URL /~user; disable jika tidak dibutuhkan.

---

### Soal 14

**Pertanyaan:** Command mana yang benar untuk menonaktifkan Basic/Digest Auth jika aplikasi memakai SSO atau auth internal modern?

A. `a2dismod auth_basic auth_digest 2>/dev/null || true`
B. `apache2ctl -M | egrep 'auth_basic|auth_digest' || true`
C. `systemctl reload apache2.service`
D. `a2enmod auth_basic auth_digest`

**Jawaban benar:** A, B, C

**Pembahasan:** Basic/Digest lama tidak perlu aktif jika tidak digunakan.

---

### Soal 15

**Pertanyaan:** Pilih command yang benar untuk menonaktifkan `mod_log_config` sesuai CIS Apache.

A. `a2dismod log_config`
B. `sed -i '/CustomLog/d' /etc/apache2/apache2.conf`
C. `rm -f /var/log/apache2/access.log`
D. `systemctl stop apache2`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: mod_log_config justru diperlukan untuk access logging.

---


## 3. Principles, Permissions, and Ownership

### Soal 16

**Pertanyaan:** Command mana yang benar untuk memastikan Apache worker berjalan sebagai user/group non-root `www-data` pada Ubuntu/Debian?

A. `grep -E '^export APACHE_RUN_USER|^export APACHE_RUN_GROUP' /etc/apache2/envvars`
B. `sed -i 's/^export APACHE_RUN_USER=.*/export APACHE_RUN_USER=www-data/' /etc/apache2/envvars`
C. `sed -i 's/^export APACHE_RUN_GROUP=.*/export APACHE_RUN_GROUP=www-data/' /etc/apache2/envvars`
D. `sed -i 's/^export APACHE_RUN_USER=.*/export APACHE_RUN_USER=root/' /etc/apache2/envvars`

**Jawaban benar:** A, B, C

**Pembahasan:** Apache worker tidak boleh berjalan sebagai root.

---

### Soal 17

**Pertanyaan:** Command mana yang benar untuk membuat akun service Apache tidak dapat login interaktif?

A. `usermod -s /usr/sbin/nologin www-data 2>/dev/null || true`
B. `passwd -l www-data 2>/dev/null || true`
C. `getent passwd www-data`
D. `passwd -u www-data`

**Jawaban benar:** A, B, C

**Pembahasan:** Service account harus nologin dan terkunci.

---

### Soal 18

**Pertanyaan:** Command mana yang benar untuk mengamankan konfigurasi Apache di `/etc/apache2`?

A. `chown -R root:root /etc/apache2`
B. `find /etc/apache2 -type d -exec chmod 755 {} \;`
C. `find /etc/apache2 -type f -exec chmod 644 {} \;`
D. `chmod -R 777 /etc/apache2`

**Jawaban benar:** A, B, C

**Pembahasan:** Konfigurasi harus root-owned dan tidak writable oleh user biasa.

---

### Soal 19

**Pertanyaan:** Command audit mana yang benar untuk mencari file konfigurasi Apache yang writable oleh group/other?

A. `find /etc/apache2 -xdev -type f -perm /022 -ls`
B. `find /etc/apache2 -xdev -type d -perm /022 -ls`
C. `stat -c '%n %U:%G %a' /etc/apache2`
D. `find /etc/apache2 -type f -delete`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit permission tidak boleh langsung menghapus file.

---

### Soal 20

**Pertanyaan:** Untuk static site, command mana yang benar agar DocumentRoot tidak writable oleh user Apache?

A. `chown -R root:root /var/www`
B. `find /var/www -type d -exec chmod 755 {} \;`
C. `find /var/www -type f -exec chmod 644 {} \;`
D. `chown -R www-data:www-data /var/www`

**Jawaban benar:** A, B, C

**Pembahasan:** Akun Apache sebaiknya membaca konten, bukan memiliki/menulis seluruh web root.

---

### Soal 21

**Pertanyaan:** Aplikasi butuh direktori upload. Command mana yang lebih sesuai prinsip CIS?

A. `mkdir -p /var/lib/myapp/uploads`
B. `chown root:www-data /var/lib/myapp/uploads`
C. `chmod 750 /var/lib/myapp/uploads`
D. `chmod 777 /var/www/html/uploads`

**Jawaban benar:** A, B, C

**Pembahasan:** Writable directory sebaiknya khusus, dibatasi, dan idealnya di luar document root.

---

### Soal 22

**Pertanyaan:** Pilih command yang benar untuk memberi Apache akses tulis penuh ke seluruh `/var/www` sesuai CIS.

A. `chown -R www-data:www-data /var/www`
B. `chmod -R 777 /var/www`
C. `chmod -R g+w /var/www`
D. `setfacl -Rm u:www-data:rwx /var/www`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: memberi akses tulis penuh ke web root bertentangan dengan least privilege.

---


## 4. Apache Access Control

### Soal 23

**Pertanyaan:** Konfigurasi mana yang benar untuk menolak akses ke OS root directory secara default?

A.

```text
<Directory />
    Require all denied
    AllowOverride None
    Options None
</Directory>
```
B.

```text
<Directory />
    Require all granted
</Directory>
```
C. `a2enconf 00-cis-root-deny`
D. `apache2ctl configtest`

**Jawaban benar:** A, C, D

**Pembahasan:** OS root harus deny by default; opsi B membuka semua.

---

### Soal 24

**Pertanyaan:** Konfigurasi mana yang benar untuk mengizinkan web content publik pada `/var/www/html` setelah OS root ditolak?

A.

```text
<Directory /var/www/html>
    Require all granted
    AllowOverride None
    Options None
</Directory>
```
B.

```text
<Directory /var/www/html>
    Require all denied
</Directory>
```
C. `a2enconf 10-cis-webroot-access`
D. `systemctl reload apache2.service`

**Jawaban benar:** A, C, D

**Pembahasan:** Web root sah boleh granted, tetapi tetap AllowOverride None dan Options minimal.

---

### Soal 25

**Pertanyaan:** Command mana yang benar untuk audit `AllowOverride` dan konfigurasi Directory?

A. `grep -R 'AllowOverride' /etc/apache2/apache2.conf /etc/apache2/conf-enabled /etc/apache2/sites-enabled 2>/dev/null`
B. `grep -R '<Directory' /etc/apache2/`
C. `apache2ctl -t -D DUMP_RUN_CFG`
D. `chmod 777 /var/www/html/.htaccess`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit mencari direktif efektif; chmod .htaccess salah.

---

### Soal 26

**Pertanyaan:** Konfigurasi admin panel hanya boleh subnet internal. Opsi mana yang benar?

A.

```text
<Directory /var/www/html/admin>
    Require ip 192.168.10.0/24
</Directory>
```
B.

```text
<Directory /var/www/html/admin>
    Require all granted
</Directory>
```
C.

```text
<Directory /var/www/html/admin>
    AllowOverride None
</Directory>
```
D. `Require ip any`

**Jawaban benar:** A, C

**Pembahasan:** Admin panel perlu pembatasan IP; Require all granted terlalu luas.

---


## 5. Minimize Features, Content, and Options

### Soal 27

**Pertanyaan:** Konfigurasi `Options` mana yang paling sesuai CIS untuk OS root directory?

A. Options None
B. Options Indexes FollowSymLinks ExecCGI
C. `AllowOverride None`
D. `Require all denied`

**Jawaban benar:** A, C, D

**Pembahasan:** Root directory harus paling ketat.

---

### Soal 28

**Pertanyaan:** Command mana yang benar untuk menghapus default content Apache jika benar-benar hanya file default/demo?

A. `rm -f /var/www/html/index.html`
B. `rm -f /usr/lib/cgi-bin/printenv /usr/lib/cgi-bin/test-cgi 2>/dev/null || true`
C. `find /var/www -maxdepth 3 -type f | sort`
D. `rm -rf /var/www`

**Jawaban benar:** A, B, C

**Pembahasan:** Hapus file default/demo setelah audit; jangan hapus seluruh /var/www sembarangan.

---

### Soal 29

**Pertanyaan:** Konfigurasi mana yang benar untuk mematikan TRACE?

A. `TraceEnable Off`
B. `TraceEnable On`
C. `curl -i -X TRACE http://127.0.0.1/ | head`
D. `a2enconf 20-cis-methods`

**Jawaban benar:** A, C, D

**Pembahasan:** TraceEnable Off adalah remediation; curl audit.

---

### Soal 30

**Pertanyaan:** Konfigurasi mana yang benar untuk membatasi method hanya GET, POST, HEAD, OPTIONS pada web root?

A.

```text
<LimitExcept GET POST HEAD OPTIONS>
    Require all denied
</LimitExcept>
```
B.

```text
<LimitExcept DELETE PUT TRACE>
    Require all granted
</LimitExcept>
```
C. `TraceEnable Off`
D. `Require all granted untuk semua method`

**Jawaban benar:** A, C

**Pembahasan:** LimitExcept menolak method di luar allowlist; TRACE juga harus dimatikan khusus.

---

### Soal 31

**Pertanyaan:** Konfigurasi mana yang benar untuk menolak HTTP protocol lama/invalid pada Apache 2.4?

A. `HttpProtocolOptions Strict`
B. `HttpProtocolOptions Unsafe`
C. `a2enconf 21-cis-http-protocol`
D. `apache2ctl configtest`

**Jawaban benar:** A, C, D

**Pembahasan:** Strict mengurangi penerimaan request invalid/abnormal.

---

### Soal 32

**Pertanyaan:** Konfigurasi mana yang benar untuk memblokir file `.ht*`?

A.

```text
<FilesMatch "^\.ht">
    Require all denied
</FilesMatch>
```
B.

```text
<FilesMatch "^\.ht">
    Require all granted
</FilesMatch>
```
C. `a2enconf 30-cis-sensitive-files`
D. `apache2ctl configtest`

**Jawaban benar:** A, C, D

**Pembahasan:** .htaccess/.htpasswd tidak boleh disajikan.

---

### Soal 33

**Pertanyaan:** Konfigurasi mana yang benar untuk memblokir metadata repository `.git` dan `.svn`?

A.

```text
<DirectoryMatch "(^|/)\.(git|svn)(/|$)">
    Require all denied
</DirectoryMatch>
```
B.

```text
<DirectoryMatch "(^|/)\.(git|svn)(/|$)">
    Require all granted
</DirectoryMatch>
```
C. `find /var/www -name .git -o -name .svn`
D. `apache2ctl configtest`

**Jawaban benar:** A, C, D

**Pembahasan:** Metadata VCS tidak boleh terbuka lewat web.

---

### Soal 34

**Pertanyaan:** Konfigurasi mana yang benar untuk membatasi file backup/config agar tidak tersaji?

A.

```text
<FilesMatch "\.(bak|old|orig|save|swp|sql|tar|tgz|gz|zip|7z|env|ini|conf|config|log)$">
    Require all denied
</FilesMatch>
```
B.

```text
<FilesMatch "\.(bak|old|sql|env)$">
    Require all granted
</FilesMatch>
```
C. `a2enconf 30-cis-sensitive-files`
D. `systemctl reload apache2.service`

**Jawaban benar:** A, C, D

**Pembahasan:** File backup/config/log berisiko membocorkan rahasia.

---

### Soal 35

**Pertanyaan:** Konfigurasi mana yang benar untuk menolak request dengan Host header berupa IPv4 langsung?

A.

```text
RewriteEngine On
RewriteCond %{HTTP_HOST} ^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$
RewriteRule ^ - [F]
```
B. `a2enmod rewrite`
C. `RewriteRule ^ - [L]`
D. `a2enconf 31-cis-no-ip-host`

**Jawaban benar:** A, B, D

**Pembahasan:** Rewrite rule dengan [F] menolak IP-based Host header.

---

### Soal 36

**Pertanyaan:** Konfigurasi Listen mana yang lebih sesuai prinsip CIS pada server multi-interface?

A. `Listen 192.0.2.10:80`
B. `Listen 192.0.2.10:443`
C. `Listen 0.0.0.0:80 untuk semua interface tanpa alasan`
D. `grep -R '^Listen' /etc/apache2/ports.conf`

**Jawaban benar:** A, B, D

**Pembahasan:** Listen IP eksplisit mengurangi exposure tidak sengaja.

---

### Soal 37

**Pertanyaan:** Header mana yang benar untuk membatasi framing/clickjacking?

A. Header always set X-Frame-Options "SAMEORIGIN"
B. Header always set Content-Security-Policy "frame-ancestors 'self'"
C. Header always set X-Frame-Options "ALLOWALL"
D. `a2enmod headers`

**Jawaban benar:** A, B, D

**Pembahasan:** X-Frame-Options dan CSP frame-ancestors membantu membatasi framing.

---

### Soal 38

**Pertanyaan:** Header mana yang benar untuk membatasi referrer dan fitur browser yang tidak diperlukan?

A. Header always set Referrer-Policy "no-referrer"
B. Header always set Permissions-Policy "geolocation=(), microphone=(), camera=(), payment=()"
C. Header always set Referrer-Policy "unsafe-url"
D. `a2enconf 40-cis-security-headers`

**Jawaban benar:** A, B, D

**Pembahasan:** no-referrer dan Permissions-Policy ketat mengurangi kebocoran dan misuse fitur browser.

---


## 6. Logging, Monitoring, and Maintenance

### Soal 39

**Pertanyaan:** Konfigurasi mana yang benar untuk logging dasar Apache?

A. `LogLevel warn`
B. `ErrorLog ${APACHE_LOG_DIR}/error.log`
C. `CustomLog ${APACHE_LOG_DIR}/access.log combined`
D. `LogLevel debug permanen tanpa kebutuhan`

**Jawaban benar:** A, B, C

**Pembahasan:** Log harus cukup informatif; debug permanen dapat sangat noisy.

---

### Soal 40

**Pertanyaan:** Command mana yang benar untuk audit log Apache?

A. `tail -n 20 /var/log/apache2/access.log`
B. `tail -n 20 /var/log/apache2/error.log`
C. `grep -R 'CustomLog\|ErrorLog\|LogLevel' /etc/apache2/`
D. `rm -f /var/log/apache2/*.log`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit log bukan menghapus log.

---

### Soal 41

**Pertanyaan:** Konfigurasi logrotate mana yang sesuai untuk retensi sekitar 13 minggu?

A. weekly
B. `rotate 13`
C. compress
D. `create 777 root root`

**Jawaban benar:** A, B, C

**Pembahasan:** Mode create harus ketat seperti 640 root adm, bukan 777.

---

### Soal 42

**Pertanyaan:** Command mana yang benar untuk menguji konfigurasi logrotate tanpa menjalankan rotasi nyata?

A. `logrotate -d /etc/logrotate.conf`
B. `cat /etc/logrotate.d/apache2`
C. `rm -rf /var/log/apache2`
D. `systemctl reload apache2.service`

**Jawaban benar:** A, B, D

**Pembahasan:** logrotate -d adalah debug/dry run; reload Apache relevan setelah rotasi.

---

### Soal 43

**Pertanyaan:** Command mana yang benar untuk patching Apache dari repository Ubuntu/Debian?

A. `apt update`
B. `apt -y upgrade apache2`
C. `apache2 -v`
D. `wget random-apache.tar.gz && make install tanpa verifikasi`

**Jawaban benar:** A, B, C

**Pembahasan:** Patching vendor package dilakukan via APT.

---

### Soal 44

**Pertanyaan:** Command mana yang benar untuk memasang dan mengaktifkan ModSecurity pada Ubuntu/Debian?

A. `apt install -y libapache2-mod-security2 modsecurity-crs`
B. `a2enmod security2`
C. `sed -i 's/^SecRuleEngine .*/SecRuleEngine On/' /etc/modsecurity/modsecurity.conf`
D. `a2dismod security2`

**Jawaban benar:** A, B, C

**Pembahasan:** ModSecurity harus terpasang, modul aktif, dan engine On.

---

### Soal 45

**Pertanyaan:** Command mana yang benar untuk audit ModSecurity/CRS?

A. `apache2ctl -M | grep security2`
B. `grep SecRuleEngine /etc/modsecurity/modsecurity.conf`
C. `ls -l /usr/share/modsecurity-crs/ /etc/modsecurity/ 2>/dev/null`
D. `rm -rf /usr/share/modsecurity-crs`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit mengecek modul, engine, dan CRS; menghapus CRS salah.

---

### Soal 46

**Pertanyaan:** Pilih command yang benar untuk menjadikan ModSecurity efektif tanpa rule set apa pun.

A. `a2enmod security2 lalu tidak memuat CRS/rule apa pun`
B. `SecRuleEngine Off`
C. `a2dismod security2`
D. `rm -rf /etc/modsecurity`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: ModSecurity tanpa rule set tidak memberi perlindungan berarti; SecRuleEngine Off juga tidak efektif.

---


## 7. SSL/TLS Configuration

### Soal 47

**Pertanyaan:** Command mana yang benar untuk mengaktifkan modul TLS dan header pada Apache Ubuntu/Debian?

A. `a2enmod ssl headers socache_shmcb`
B. `apache2ctl configtest`
C. `systemctl reload apache2.service`
D. `a2dismod ssl`

**Jawaban benar:** A, B, C

**Pembahasan:** mod_ssl dan headers diperlukan untuk HTTPS/header keamanan.

---

### Soal 48

**Pertanyaan:** Permission mana yang benar untuk private key TLS?

A. `chown root:root /etc/ssl/private/*.key 2>/dev/null || true`
B. `chmod 600 /etc/ssl/private/*.key 2>/dev/null || true`
C. `find /etc/ssl/private -type f -name '*.key' -exec stat -c '%n %U:%G %a' {} \;`
D. `chmod 644 /etc/ssl/private/*.key`

**Jawaban benar:** A, B, C

**Pembahasan:** Private key harus sangat dibatasi.

---

### Soal 49

**Pertanyaan:** Konfigurasi TLS mana yang benar untuk menonaktifkan protokol lama?

A. SSLProtocol -all +TLSv1.2 +TLSv1.3
B. SSLProtocol all
C. SSLProtocol +TLSv1 +TLSv1.1
D. `grep -R 'SSLProtocol' /etc/apache2/`

**Jawaban benar:** A, D

**Pembahasan:** TLSv1.0/1.1 harus tidak ditawarkan.

---

### Soal 50

**Pertanyaan:** Konfigurasi mana yang benar untuk cipher kuat dan forward secrecy?

A. SSLCipherSuite ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256
B. `SSLHonorCipherOrder On`
C. SSLCipherSuite ALL:NULL:EXPORT:RC4
D. SSLCompression Off

**Jawaban benar:** A, B, D

**Pembahasan:** ECDHE GCM mendukung forward secrecy; NULL/EXPORT/RC4 salah.

---

### Soal 51

**Pertanyaan:** Konfigurasi mana yang benar untuk HSTS dan OCSP Stapling?

A. Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
B. SSLUseStapling On
C. `SSLStaplingCache "shmcb:/var/run/apache2/stapling_cache(128000)"`
D. Header always unset Strict-Transport-Security

**Jawaban benar:** A, B, C

**Pembahasan:** HSTS dan OCSP Stapling adalah hardening TLS; jangan unset HSTS jika diwajibkan.

---

### Soal 52

**Pertanyaan:** Command mana yang benar untuk menguji apakah TLS 1.1 ditolak?

A. `openssl s_client -connect example.com:443 -servername example.com -tls1_1 </dev/null`
B. `openssl s_client -connect example.com:443 -servername example.com -tls1_2 </dev/null`
C. `curl -I https://example.com/`
D. `chmod 777 /etc/ssl/private`

**Jawaban benar:** A, B, C

**Pembahasan:** openssl/curl untuk audit koneksi; chmod 777 private key salah.

---

### Soal 53

**Pertanyaan:** Konfigurasi mana yang benar untuk redirect HTTP ke HTTPS?

A.

```text
<VirtualHost *:80>
    ServerName example.com
    Redirect permanent / https://example.com/
</VirtualHost>
```
B. `Redirect permanent / http://example.com/`
C. `a2ensite example.conf`
D. `apache2ctl configtest`

**Jawaban benar:** A, C, D

**Pembahasan:** Redirect harus menuju HTTPS, bukan HTTP.

---

### Soal 54

**Pertanyaan:** Pilih konfigurasi yang benar untuk menyimpan private key TLS di DocumentRoot agar mudah diakses Apache.

A. `SSLCertificateKeyFile /var/www/html/server.key`
B. `chmod 644 /var/www/html/server.key`
C. `chown www-data:www-data /var/www/html/server.key`
D. `Alias /key /var/www/html/server.key`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: private key tidak boleh berada di web root atau readable publik.

---


## 8. Information Leakage

### Soal 55

**Pertanyaan:** Konfigurasi mana yang benar untuk mengurangi informasi versi Apache?

A. ServerTokens Prod
B. ServerSignature Off
C. FileETag MTime Size
D. ServerTokens Full

**Jawaban benar:** A, B, C

**Pembahasan:** Prod/Off dan ETag tanpa inode mengurangi fingerprinting.

---

### Soal 56

**Pertanyaan:** Command mana yang benar untuk audit header Server?

A. `curl -I http://127.0.0.1/ 2>/dev/null | grep -i '^Server:'`
B. `grep -R 'ServerTokens\|ServerSignature\|FileETag' /etc/apache2/`
C. `apache2ctl configtest`
D. `cat /etc/shadow`

**Jawaban benar:** A, B, C

**Pembahasan:** Header/configtest relevan; shadow tidak.

---

### Soal 57

**Pertanyaan:** Command mana yang benar untuk mencari default/example/test content yang berisiko?

A. `find /var/www -type f \( -name '*default*' -o -name '*example*' -o -name '*test*' \) -print`
B. `find /usr/share/apache2 -maxdepth 2 -type f 2>/dev/null | sort`
C. `curl -I http://127.0.0.1/`
D. `chmod -R 777 /usr/share/apache2`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit konten/default dan header; chmod 777 salah.

---


## 9. Denial of Service Mitigations

### Soal 58

**Pertanyaan:** Konfigurasi mana yang benar untuk mitigasi koneksi lambat/DoS ringan?

A. `TimeOut 10`
B. `KeepAlive On`
C. `MaxKeepAliveRequests 100`
D. `KeepAliveTimeout 15`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua sesuai prinsip pembatasan resource.

---

### Soal 59

**Pertanyaan:** Command mana yang benar untuk mengaktifkan dan audit module request timeout?

A. `a2enmod reqtimeout`
B. `apache2ctl -M | grep reqtimeout`
C. `RequestReadTimeout header=20-40,MinRate=500 body=20,MinRate=500`
D. `a2dismod reqtimeout`

**Jawaban benar:** A, B, C

**Pembahasan:** reqtimeout membantu mitigasi Slowloris/slow POST.

---

### Soal 60

**Pertanyaan:** Konfigurasi mana yang salah karena memperbesar risiko koneksi idle menahan resource?

A. `TimeOut 300`
B. `KeepAliveTimeout 300`
C. `MaxKeepAliveRequests 0`
D. `TimeOut 10`

**Jawaban benar:** A, B, C

**Pembahasan:** Timeout besar/unlimited request bisa menghabiskan resource.

---

### Soal 61

**Pertanyaan:** Pilih konfigurasi yang benar untuk mematikan semua timeout agar client lambat tetap terlayani sesuai CIS.

A. `TimeOut 0`
B. `KeepAliveTimeout 0`
C. `RequestReadTimeout header=0 body=0`
D. `MaxKeepAliveRequests 0`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: mematikan/bikin unlimited timeout bukan mitigasi DoS.

---


## 10. Request Limits

### Soal 62

**Pertanyaan:** Konfigurasi request limit mana yang sesuai baseline CIS Apache?

A. `LimitRequestLine 8190`
B. `LimitRequestFields 100`
C. `LimitRequestFieldSize 8190`
D. `LimitRequestBody 102400`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Keempatnya membatasi ukuran input HTTP.

---

### Soal 63

**Pertanyaan:** Konfigurasi mana yang salah karena membuat request body tidak terbatas?

A. `LimitRequestBody 0`
B. `LimitRequestLine 0`
C. `LimitRequestFields 0`
D. `LimitRequestFieldSize 8190`

**Jawaban benar:** A, B, C

**Pembahasan:** Nilai 0 pada limit dapat berarti tidak terbatas/tidak sesuai rekomendasi.

---

### Soal 64

**Pertanyaan:** Aplikasi upload dokumen membutuhkan body lebih dari 102400 byte. Apa tindakan paling sesuai CIS?

A. Naikkan limit hanya pada Location/Directory yang membutuhkan
B. Dokumentasikan exception
C. Uji di staging sebelum production
D. `Set LimitRequestBody 0 global`

**Jawaban benar:** A, B, C

**Pembahasan:** Exception/tuning spesifik lebih aman daripada unlimited global.

---


## 11. SELinux dan 12. AppArmor

### Soal 65

**Pertanyaan:** Pada Ubuntu/Debian, command mana yang benar untuk AppArmor Apache?

A. `apt install -y apparmor apparmor-utils apparmor-profiles apparmor-profiles-extra`
B. `systemctl enable --now apparmor.service`
C. aa-status
D. getenforce

**Jawaban benar:** A, B, C

**Pembahasan:** Ubuntu/Debian umumnya memakai AppArmor; getenforce untuk SELinux.

---

### Soal 66

**Pertanyaan:** Jika profile Apache tersedia, command mana yang benar untuk enforcing mode?

A. `aa-enforce /etc/apparmor.d/usr.sbin.apache2 2>/dev/null || true`
B. `systemctl restart apache2.service`
C. `aa-status | grep -i apache || true`
D. `aa-complain /etc/apparmor.d/usr.sbin.apache2 sebagai mode produksi final`

**Jawaban benar:** A, B, C

**Pembahasan:** Enforce untuk produksi setelah uji; complain hanya testing.

---

### Soal 67

**Pertanyaan:** Command mana yang benar jika distro memakai SELinux dan ingin audit context Apache?

A. getenforce
B. sestatus
C. `ps -eZ | grep -E 'httpd|apache2'`
D. `semanage boolean -l | grep httpd`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Ini audit SELinux, terutama relevan pada RHEL-family.

---

### Soal 68

**Pertanyaan:** Pilih command yang benar untuk menjalankan SELinux dan AppArmor bersamaan pada Ubuntu production agar perlindungan ganda pasti aman.

A. `apt install selinux-basics && selinux-activate`
B. `aa-disable /etc/apparmor.d/usr.sbin.apache2`
C. `setenforce 0`
D. `chmod 777 /etc/apparmor.d/usr.sbin.apache2`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: CIS menjelaskan SELinux/AppArmor adalah kontrol serupa; menjalankan dua mekanisme tanpa desain matang dapat menyulitkan troubleshooting.

---


## 13. Validasi Akhir

### Soal 69

**Pertanyaan:** Command validasi akhir mana yang benar setelah hardening Apache?

A. `systemctl is-active apache2.service`
B. `apache2ctl configtest`
C. `apache2ctl -S`
D. `apache2ctl -M | sort`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua command berguna untuk validasi service, syntax, vhost, dan modul.

---

### Soal 70

**Pertanyaan:** Command mana yang benar untuk memastikan modul berisiko tidak aktif?

A. `apache2ctl -M | egrep 'dav|status|autoindex|userdir|info|auth_basic|auth_digest' || true`
B. `ls -1 /etc/apache2/mods-enabled/`
C. `apache2ctl configtest`
D. `a2enmod dav status info autoindex`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit modul aktif; a2enmod justru mengaktifkan modul.

---

### Soal 71

**Pertanyaan:** Command mana yang benar untuk audit header keamanan HTTP?

A. curl -I http://127.0.0.1/ 2>/dev/null | egrep -i 'server:|x-frame-options|content-security-policy|referrer-policy|permissions-policy'
B. `curl -Ik https://example.com/ | egrep -i 'strict-transport-security|server:'`
C. `grep -R 'Header always set' /etc/apache2/`
D. `cat /etc/ssl/private/example.key`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit header boleh dengan curl/grep; membaca private key tidak perlu dan berisiko.

---

### Soal 72

**Pertanyaan:** Command mana yang benar untuk audit permission Apache?

A. `stat -c '%n %U:%G %a' /etc/apache2 /usr/sbin/apache2 /var/log/apache2 /var/www`
B. `find /etc/apache2 -perm /022 -ls`
C. `find /var/www -xdev -perm /002 -ls`
D. `chmod -R 777 /var/www`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit permission penting; chmod 777 salah.

---

### Soal 73

**Pertanyaan:** Jika `apache2ctl configtest` gagal setelah perubahan, tindakan mana yang benar?

A. `Jangan reload/restart dulu`
B. `Baca pesan error dan file/line yang disebut`
C. `Rollback dari backup jika perlu`
D. `Paksa restart dengan `systemctl restart apache2 --force``

**Jawaban benar:** A, B, C

**Pembahasan:** Syntax gagal harus diperbaiki sebelum reload/restart.

---

### Soal 74

**Pertanyaan:** Command mana yang benar untuk membuka semua akses agar website pasti bisa diakses saat ujian?

A. `Require all granted pada <Directory />`
B. Options Indexes FollowSymLinks ExecCGI pada root
C. `chmod -R 777 /var/www`
D. `TraceEnable On`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: semua opsi melemahkan hardening CIS.

---


## 2. Minimize Apache Modules

### Soal 75

**Pertanyaan:** Command mana yang benar untuk memeriksa apakah `mod_status` masih aktif setelah hardening?

A. `apache2ctl -M | grep status || true`
B. `ls /etc/apache2/mods-enabled/status.load 2>/dev/null`
C. `curl -s http://127.0.0.1/server-status | head`
D. `a2enmod status`

**Jawaban benar:** A, B, C

**Pembahasan:** A/B/C audit; a2enmod status mengaktifkan modul.

---


## 5. Minimize Features, Content, and Options

### Soal 76

**Pertanyaan:** Command mana yang benar untuk memastikan `headers` module aktif sebelum menyetel security headers?

A. `a2enmod headers`
B. `apache2ctl -M | grep headers`
C. `apache2ctl configtest`
D. `a2dismod headers`

**Jawaban benar:** A, B, C

**Pembahasan:** Security header membutuhkan mod_headers.

---


## 6. Logging, Monitoring, and Maintenance

### Soal 77

**Pertanyaan:** Jika Apache berada di belakang reverse proxy, opsi logging mana yang perlu dipertimbangkan?

A. `Menambahkan format log yang mencatat Host/vhost`
B. `Mempertimbangkan header client IP seperti X-Forwarded-For jika dipercaya`
C. `Membiarkan access log kosong`
D. `Menonaktifkan CustomLog`

**Jawaban benar:** A, B

**Pembahasan:** Di belakang proxy, IP asli bisa berada di header terpercaya; logging tidak boleh dimatikan.

---


## 7. SSL/TLS Configuration

### Soal 78

**Pertanyaan:** Command mana yang benar untuk melihat virtual host TLS yang aktif?

A. `apache2ctl -S`
B. `grep -R 'SSLCertificateFile\|SSLCertificateKeyFile' /etc/apache2/sites-enabled /etc/apache2/conf-enabled 2>/dev/null`
C. `ss -tulpen | grep ':443'`
D. `rm /etc/ssl/private/*.key`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit TLS vhost/cert/listen; menghapus key salah.

---


## 8. Information Leakage

### Soal 79

**Pertanyaan:** Konfigurasi mana yang benar jika ingin menghindari inode pada ETag?

A. FileETag MTime Size
B. FileETag None
C. FileETag INode MTime Size
D. ServerSignature Off

**Jawaban benar:** A, B, D

**Pembahasan:** FileETag MTime Size atau None menghindari inode; ServerSignature Off mengurangi leakage lain.

---


## 9. Denial of Service Mitigations

### Soal 80

**Pertanyaan:** Manakah konfigurasi `RequestReadTimeout` yang benar secara sintaks dan tujuan?

A. `RequestReadTimeout header=20-40,MinRate=500 body=20,MinRate=500`
B. <IfModule reqtimeout_module> RequestReadTimeout header=20-40,MinRate=500 body=20,MinRate=500 </IfModule>
C. `RequestReadTimeout Off untuk hardening`
D. `a2enmod reqtimeout`

**Jawaban benar:** A, B, D

**Pembahasan:** RequestReadTimeout perlu aktif untuk membatasi slow request.

---


## 10. Request Limits

### Soal 81

**Pertanyaan:** Command mana yang benar untuk menerapkan file konfigurasi request limit?

A.

```text
cat >/etc/apache2/conf-available/90-cis-request-limits.conf <<'EOF'
LimitRequestLine 8190
LimitRequestFields 100
LimitRequestFieldSize 8190
LimitRequestBody 102400
EOF
```
B. `a2enconf 90-cis-request-limits`
C. `apache2ctl configtest`
D. `systemctl reload apache2.service`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua adalah alur valid konfigurasi + enable + reload.

---


## 4. Apache Access Control

### Soal 82

**Pertanyaan:** Konfigurasi mana yang benar untuk endpoint monitoring hanya localhost?

A.

```text
<Location /server-health>
    Require local
</Location>
```
B.

```text
<Location /server-health>
    Require all granted
</Location>
```
C.

```text
<Location /server-health>
    Require ip 127.0.0.1 ::1
</Location>
```
D. `Require ip 0.0.0.0/0`

**Jawaban benar:** A, C

**Pembahasan:** Endpoint internal harus dibatasi local/IP tertentu.

---


## 3. Principles, Permissions, and Ownership

### Soal 83

**Pertanyaan:** Command mana yang benar untuk audit apakah akun Apache punya shell valid?

A. `getent passwd www-data`
B. `awk -F: '$1=="www-data"{print $7}' /etc/passwd`
C. `usermod -s /bin/bash www-data`
D. `passwd -S www-data`

**Jawaban benar:** A, B, D

**Pembahasan:** Audit shell/status; jangan memberi /bin/bash.

---


## 1. Planning and Installation

### Soal 84

**Pertanyaan:** Command mana yang benar untuk menguji Apache setelah reload?

A. `curl -I http://127.0.0.1/`
B. `journalctl -u apache2.service -n 50 --no-pager`
C. `systemctl status apache2.service --no-pager`
D. `rm -rf /var/log/apache2`

**Jawaban benar:** A, B, C

**Pembahasan:** curl/journalctl/status untuk validasi.

---


## Bonus — Adaptasi Firewalld

### Soal 85

**Pertanyaan:** Jika organisasi memilih firewalld sebagai adaptasi lokal, command mana yang benar secara sintaks firewalld untuk membuka HTTPS?

A. `apt install -y firewalld`
B. `systemctl enable --now firewalld`
C. `firewall-cmd --permanent --zone=public --add-service=https`
D. `firewall-cmd --reload`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Valid secara firewalld, tetapi bukan metode utama pada modul Apache ini.

---

### Soal 86

**Pertanyaan:** Command firewalld mana yang salah karena membuka layanan terlalu luas?

A. `firewall-cmd --permanent --zone=public --add-service=all`
B. `firewall-cmd --permanent --zone=trusted --add-source=0.0.0.0/0`
C. `firewall-cmd --permanent --zone=public --add-port=1-65535/tcp`
D. `firewall-cmd --permanent --zone=public --add-service=https`

**Jawaban benar:** A, B, C

**Pembahasan:** A/B/C melemahkan allowlist; HTTPS spesifik bisa benar jika layanan butuh.

---

### Soal 87

**Pertanyaan:** Jika soal menyebut implementasi resmi Ubuntu/Debian pada file ini memakai UFW atau konfigurasi Apache, command mana yang benar untuk menjawab dengan firewalld sebagai pengganti wajib?

A. `firewall-cmd --set-default-zone=public`
B. `firewall-cmd --permanent --add-service=http`
C. `firewall-cmd --reload`
D. `firewall-cmd --permanent --add-service=all`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: firewalld hanya adaptasi organisasi; bukan jawaban wajib jika konteksnya konfigurasi Apache/Ubuntu default pada file ini.

---


## 1. Planning and Installation

### Soal 88

**Pertanyaan:** Pilih langkah yang benar setelah mengubah konfigurasi Apache.

A. `apache2ctl configtest`
B. `apache2ctl -t`
C. `systemctl reload apache2.service jika syntax OK`
D. `reload walaupun configtest gagal`

**Jawaban benar:** A, B, C

**Pembahasan:** Jangan reload jika syntax gagal.

---

### Soal 89

**Pertanyaan:** Command mana yang benar untuk melihat package Apache yang terpasang di Ubuntu/Debian?

A. `dpkg -l | grep apache2`
B. `apt policy apache2`
C. `apache2 -v`
D. `rpm -qa | grep httpd`

**Jawaban benar:** A, B, C

**Pembahasan:** rpm untuk RHEL-family, bukan Ubuntu/Debian.

---


## 2. Minimize Apache Modules

### Soal 90

**Pertanyaan:** Command mana yang benar untuk mengecek modul proxy secara spesifik?

A. `apache2ctl -M | grep proxy || true`
B. `ls /etc/apache2/mods-enabled/proxy*.load 2>/dev/null`
C. `a2query -m proxy 2>/dev/null || true`
D. `a2enmod proxy`

**Jawaban benar:** A, B, C

**Pembahasan:** A/B/C audit; a2enmod mengaktifkan proxy.

---

### Soal 91

**Pertanyaan:** Command mana yang benar untuk disable modul `cgi` jika server tidak menjalankan CGI?

A. `a2dismod cgi 2>/dev/null || true`
B. `a2dismod cgid 2>/dev/null || true`
C. `apache2ctl configtest`
D. `a2enmod cgi cgid`

**Jawaban benar:** A, B, C

**Pembahasan:** CGI/cgid sebaiknya nonaktif jika tidak dipakai.

---

### Soal 92

**Pertanyaan:** Server tidak memakai konten dari home user. Apa command yang benar?

A. `a2dismod userdir`
B. `grep -R 'UserDir' /etc/apache2/`
C. `apache2ctl -M | grep userdir || true`
D. `a2enmod userdir`

**Jawaban benar:** A, B, C

**Pembahasan:** UserDir dinonaktifkan dan diaudit.

---


## 3. Principles, Permissions, and Ownership

### Soal 93

**Pertanyaan:** Command mana yang benar untuk audit file Apache yang dimiliki user selain root pada konfigurasi/system path?

A. `find /etc/apache2 ! -user root -ls`
B. `find /usr/lib/apache2/modules ! -user root -ls`
C. `find /usr/sbin/apache2 ! -user root -ls 2>/dev/null`
D. `chown -R www-data:www-data /etc/apache2`

**Jawaban benar:** A, B, C

**Pembahasan:** File konfigurasi/module Apache seharusnya root-owned.

---

### Soal 94

**Pertanyaan:** Command mana yang benar untuk audit permission world-writable pada web root?

A. `find /var/www -xdev -perm /002 -ls`
B. `find /var/www -xdev -type f -perm /002 -ls`
C. `find /var/www -xdev -type d -perm /002 -ls`
D. `chmod -R o+w /var/www`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit world-writable; jangan memberi o+w.

---

### Soal 95

**Pertanyaan:** Command mana yang benar untuk mengamankan `/var/log/apache2` agar log tidak writable sembarang user?

A. `chown root:adm /var/log/apache2 2>/dev/null || true`
B. `chmod 750 /var/log/apache2 2>/dev/null || true`
C. `find /var/log/apache2 -type f -exec chmod 640 {} \;`
D. `chmod -R 777 /var/log/apache2`

**Jawaban benar:** A, B, C

**Pembahasan:** Log harus terbatas, bukan 777.

---


## 4. Apache Access Control

### Soal 96

**Pertanyaan:** Dalam Apache 2.4, directive modern mana yang benar untuk deny/allow access?

A. `Require all denied`
B. `Require all granted`
C. `Require ip 192.168.10.0/24`
D. `Order allow,deny wajib dipakai di Apache 2.4`

**Jawaban benar:** A, B, C

**Pembahasan:** `Require` adalah pendekatan modern Apache 2.4.

---

### Soal 97

**Pertanyaan:** Pilih konfigurasi yang benar untuk mematikan `.htaccess` override di semua direktori yang dikelola.

A. `AllowOverride None`
B. `AllowOverride All`
C. `AllowOverrideList Options`
D. `grep -R 'AllowOverride' /etc/apache2/`

**Jawaban benar:** A, D

**Pembahasan:** AllowOverride None memusatkan konfigurasi; All/List membuka override.

---

### Soal 98

**Pertanyaan:** Command mana yang benar untuk mengaktifkan file konfigurasi access control yang dibuat di conf-available?

A. `a2enconf 00-cis-root-deny`
B. `a2enconf 10-cis-webroot-access`
C. `apache2ctl configtest`
D. `a2dismod 00-cis-root-deny`

**Jawaban benar:** A, B, C

**Pembahasan:** a2enconf mengaktifkan conf; a2dismod untuk modul, bukan conf.

---


## 5. Minimize Features, Content, and Options

### Soal 99

**Pertanyaan:** Options mana yang berisiko jika aktif di DocumentRoot tanpa kebutuhan?

A. Indexes
B. ExecCGI
C. Includes
D. FollowSymLinks

**Jawaban benar:** A, B, C, D

**Pembahasan:** Keempatnya bisa memperluas risiko jika tidak dibutuhkan.

---

### Soal 100

**Pertanyaan:** Konfigurasi mana yang benar untuk security headers dalam file Apache conf?

A. Header always set X-Frame-Options "SAMEORIGIN"
B. Header always set Referrer-Policy "no-referrer"
C. Header always set Permissions-Policy "geolocation=(), microphone=(), camera=()"
D. Header always set X-Powered-By "Apache 2.4.65"

**Jawaban benar:** A, B, C

**Pembahasan:** Header X-Powered-By versi server justru leakage.

---

### Soal 101

**Pertanyaan:** Command mana yang benar untuk menguji akses `.git` tidak terbuka?

A. `curl -i http://127.0.0.1/.git/config`
B. `curl -i http://127.0.0.1/.svn/entries`
C. `find /var/www -name .git -o -name .svn`
D. `git clone http://127.0.0.1/.git`

**Jawaban benar:** A, B, C

**Pembahasan:** curl/find audit; git clone bukan audit umum dan bisa menciptakan dampak tidak perlu.

---

### Soal 102

**Pertanyaan:** Jika website API membutuhkan `PUT`, apa tindakan paling benar?

A. Buat exception terdokumentasi
B. `Izinkan PUT hanya pada Location/API yang perlu`
C. Uji authorization aplikasi
D. Aktifkan PUT global untuk semua direktori

**Jawaban benar:** A, B, C

**Pembahasan:** Allowlist spesifik lebih aman daripada global.

---


## 6. Logging, Monitoring, and Maintenance

### Soal 103

**Pertanyaan:** Pilih command yang benar untuk mengirim error log Apache ke syslog facility local1.

A. `ErrorLog syslog:local1`
B.

```text
cat >/etc/rsyslog.d/30-apache.conf <<'EOF'
local1.* /var/log/apache2/error_syslog.log
EOF
```
C. `systemctl restart rsyslog.service`
D. `ErrorLog /dev/null`

**Jawaban benar:** A, B, C

**Pembahasan:** Syslog membantu integrasi log; /dev/null membuang bukti.

---

### Soal 104

**Pertanyaan:** Command mana yang benar untuk mengecek Apache error dari systemd journal?

A. `journalctl -u apache2.service -n 100 --no-pager`
B. `systemctl status apache2.service --no-pager`
C. `tail -n 50 /var/log/apache2/error.log`
D. `rm -f /var/log/apache2/error.log`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit/troubleshooting log; jangan hapus log.

---

### Soal 105

**Pertanyaan:** File log access/error Apache harus punya permission yang tidak terlalu terbuka. Opsi mana yang benar?

A. `create 640 root adm dalam logrotate`
B. `chmod 640 /var/log/apache2/*.log`
C. `chown root:adm /var/log/apache2/*.log`
D. `chmod 666 /var/log/apache2/*.log`

**Jawaban benar:** A, B, C

**Pembahasan:** Log tidak boleh world-writable/readable luas.

---

### Soal 106

**Pertanyaan:** Apa tindakan benar jika ModSecurity menghasilkan false positive besar di production?

A. Tune CRS/rule secara bertahap
B. Mulai dari detection/logging di staging
C. Dokumentasikan rule exclusion yang spesifik
D. `Matikan seluruh logging Apache`

**Jawaban benar:** A, B, C

**Pembahasan:** Tuning WAF harus spesifik; logging jangan dimatikan.

---


## 7. SSL/TLS Configuration

### Soal 107

**Pertanyaan:** Sertifikat HTTPS publik yang benar harus memenuhi apa?

A. CN/SAN sesuai hostname
B. Belum expired
C. Chain CA valid
D. Private key disimpan di document root

**Jawaban benar:** A, B, C

**Pembahasan:** Private key tidak boleh di document root.

---

### Soal 108

**Pertanyaan:** Command mana yang benar untuk audit sertifikat remote?

A. `openssl s_client -connect example.com:443 -servername example.com </dev/null`
B. echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -dates -subject -issuer
C. `curl -Iv https://example.com/`
D. `cat /etc/ssl/private/example.key`

**Jawaban benar:** A, B, C

**Pembahasan:** Private key tidak dibutuhkan untuk audit remote certificate.

---

### Soal 109

**Pertanyaan:** Konfigurasi mana yang berisiko dan harus dihindari?

A. SSLProtocol all
B. SSLCompression On
C. SSLCipherSuite ALL
D. SSLUseStapling On

**Jawaban benar:** A, B, C

**Pembahasan:** Stapling On justru baik; protokol/cipher semua dan compression On berisiko.

---

### Soal 110

**Pertanyaan:** Pilih header HSTS yang benar untuk production yang HTTPS-nya sudah stabil.

A. Strict-Transport-Security: max-age=31536000; includeSubDomains
B. Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
C. Strict-Transport-Security: max-age=0
D. Header always unset Strict-Transport-Security

**Jawaban benar:** A, B

**Pembahasan:** max-age besar dan includeSubDomains bisa baik jika siap; max-age=0/unset menonaktifkan.

---


## 8. Information Leakage

### Soal 111

**Pertanyaan:** Konfigurasi mana yang salah karena memperlihatkan terlalu banyak informasi server?

A. ServerTokens Full
B. ServerSignature On
C. FileETag INode MTime Size
D. ServerTokens Prod

**Jawaban benar:** A, B, C

**Pembahasan:** Prod lebih aman; Full/Signature/INode leakage.

---

### Soal 112

**Pertanyaan:** Command mana yang benar untuk memastikan default page tidak masih disajikan?

A. `curl -s http://127.0.0.1/ | head`
B. `grep -R 'Apache2 Ubuntu Default Page\|It works' /var/www /usr/share/apache2 2>/dev/null || true`
C. `find /var/www -maxdepth 3 -type f -print`
D. `a2enmod autoindex`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit konten default; autoindex membuka listing.

---


## 9. Denial of Service Mitigations

### Soal 113

**Pertanyaan:** Nilai mana yang sesuai untuk `KeepAliveTimeout` menurut prinsip CIS?

A. `KeepAliveTimeout 15`
B. `KeepAliveTimeout 5`
C. `KeepAliveTimeout 300`
D. `KeepAliveTimeout 0`

**Jawaban benar:** A, B

**Pembahasan:** 15 detik atau kurang lebih sesuai; 300/0 tidak ideal.

---

### Soal 114

**Pertanyaan:** Pilih command yang benar untuk menerapkan DoS conf.

A.

```text
cat >/etc/apache2/conf-available/80-cis-dos.conf <<'EOF'
TimeOut 10
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 15
EOF
```
B. `a2enconf 80-cis-dos`
C. `apache2ctl configtest`
D. `systemctl reload apache2.service`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Alur conf-available -> a2enconf -> configtest -> reload benar.

---


## 10. Request Limits

### Soal 115

**Pertanyaan:** Limit mana yang kemungkinan perlu exception pada aplikasi upload file?

A. `LimitRequestBody 102400`
B. `LimitRequestLine 8190`
C. `LimitRequestFields 100`
D. ServerTokens Prod

**Jawaban benar:** A

**Pembahasan:** Upload besar terutama dipengaruhi LimitRequestBody.

---

### Soal 116

**Pertanyaan:** Command mana yang benar untuk audit request limit aktif di konfigurasi?

A. `grep -R 'LimitRequestLine\|LimitRequestFields\|LimitRequestFieldSize\|LimitRequestBody' /etc/apache2/`
B. `apache2ctl configtest`
C. `apache2ctl -S`
D. `rm -f /etc/apache2/conf-enabled/90-cis-request-limits.conf`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit grep/configtest/vhost; remove conf melemahkan.

---


## 12. AppArmor

### Soal 117

**Pertanyaan:** Apa langkah aman saat membuat profile AppArmor custom untuk Apache?

A. `aa-autodep /usr/sbin/apache2`
B. `aa-complain /usr/sbin/apache2 untuk fase uji`
C. `aa-logprof setelah aplikasi diuji`
D. `aa-enforce tanpa testing pada production kritis`

**Jawaban benar:** A, B, C

**Pembahasan:** Enforce production perlu testing dulu.

---

### Soal 118

**Pertanyaan:** Command mana yang benar untuk memastikan AppArmor service aktif?

A. `systemctl is-active apparmor.service`
B. `systemctl is-enabled apparmor.service`
C. aa-status
D. `systemctl disable apparmor.service`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit service/status; disable bukan hardening.

---


## 13. Validasi Akhir

### Soal 119

**Pertanyaan:** Pilih output/command yang perlu dicek setelah reload Apache.

A. `journalctl -u apache2.service -n 50 --no-pager`
B. `curl -I http://127.0.0.1/`
C. `apache2ctl -M`
D. `ss -tulpen | grep apache2`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua relevan untuk memastikan Apache berjalan dan listen sesuai rencana.

---

### Soal 120

**Pertanyaan:** Jika setelah TLS hardening client lama gagal connect, apa tindakan sesuai CIS?

A. `Analisis kebutuhan bisnis client lama`
B. Buat exception jika benar-benar diperlukan
C. Dokumentasikan risiko dan rencana migrasi
D. Aktifkan SSLv3 global tanpa dokumentasi

**Jawaban benar:** A, B, C

**Pembahasan:** Exception harus disadari; jangan aktifkan protokol lama global tanpa kontrol.

---

### Soal 121

**Pertanyaan:** Command mana yang benar untuk rollback cepat dari backup konfigurasi Apache?

A. `cp -a /root/apache-hardening-backup/<tanggal>/apache2 /etc/apache2`
B. `apache2ctl configtest setelah rollback`
C. `systemctl reload apache2.service jika syntax OK`
D. `rm -rf /`

**Jawaban benar:** A, B, C

**Pembahasan:** Rollback dari backup, validasi, reload; rm -rf jelas salah.

---
