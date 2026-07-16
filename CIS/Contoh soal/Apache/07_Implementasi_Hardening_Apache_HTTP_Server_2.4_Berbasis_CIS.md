# 07 — Implementasi Hardening Apache HTTP Server 2.4 Berbasis CIS Benchmark

**Basis materi:** CIS Apache HTTP Server 2.4 Benchmark v2.3.0 dan dua modul konseptual yang sudah dibuat sebelumnya.  
**Target praktik:** Apache HTTP Server 2.4 pada Linux, terutama paket **Debian/Ubuntu** dengan service `apache2`.  
**Tujuan:** memberi panduan praktik audit dan remediation dari instalasi sampai validasi akhir.

> **Catatan penting:** Benchmark CIS Apache diuji terhadap Apache 2.4.65 source build, sedangkan perintah dalam file ini disesuaikan untuk Debian/Ubuntu package layout. Pada RHEL/Alma/Rocky, path dan service biasanya berbeda (`httpd`, `/etc/httpd`). Jangan copy-paste ke production tanpa backup, sesi console, dan pengujian.

---

## 0. Prinsip Aman Sebelum Menjalankan Hardening

Apache adalah layanan yang langsung berhadapan dengan client. Perubahan salah pada modul, TLS, access control, atau request limit dapat membuat aplikasi tidak berjalan. Terapkan urutan berikut:

1. Buat backup konfigurasi.
2. Pastikan punya akses console/VM snapshot.
3. Jalankan audit awal.
4. Terapkan satu kelompok hardening.
5. Validasi syntax Apache.
6. Reload/restart Apache.
7. Uji aplikasi.
8. Dokumentasikan exception.

```bash
sudo -i
mkdir -p /root/apache-hardening-backup/$(date +%F)
BACKUP_DIR="/root/apache-hardening-backup/$(date +%F)"
cp -a /etc/apache2 "$BACKUP_DIR/apache2" 2>/dev/null || true
cp -a /var/www "$BACKUP_DIR/var-www" 2>/dev/null || true
```

Validasi syntax sebelum reload/restart:

```bash
apache2ctl configtest
apache2ctl -t
systemctl reload apache2.service
```

---

## 1. Planning and Installation

### 1.1 Pastikan Apache Dipasang dari Repository Resmi

```bash
apt update
apt install -y apache2
systemctl enable --now apache2.service
```

Validasi:

```bash
apache2 -v
apache2ctl -V
systemctl is-enabled apache2.service
systemctl is-active apache2.service
```

### 1.2 Pastikan Server Tidak Multi-Use Tanpa Alasan

Apache server sebaiknya tidak sekaligus menjadi DNS, mail, database, file sharing, proxy publik, atau development server tanpa kebutuhan jelas.

Audit service dan port:

```bash
ss -tulpen
systemctl list-units --type=service --state=running
dpkg -l | egrep 'bind9|mariadb|mysql|postgresql|samba|vsftpd|proftpd|postfix|dovecot|squid|nfs-kernel-server|rpcbind' || true
```

Contoh remediation jika layanan tidak diperlukan:

```bash
systemctl disable --now bind9.service 2>/dev/null || true
systemctl disable --now smbd.service nmbd.service 2>/dev/null || true
systemctl disable --now vsftpd.service 2>/dev/null || true
apt purge -y bind9 samba vsftpd proftpd-basic squid nfs-kernel-server rpcbind 2>/dev/null || true
apt autoremove -y --purge
```

> Jika server memang web+database untuk lab, catat sebagai exception. Untuk production, pisahkan peran bila memungkinkan.

---

## 2. Minimize Apache Modules

### 2.1 Audit Modul Apache Aktif

```bash
apache2ctl -M
apache2ctl -M | sort
ls -1 /etc/apache2/mods-enabled/
```

### 2.2 Pastikan `mod_log_config` Aktif

Pada Debian/Ubuntu, `log_config` biasanya static/built-in. Audit:

```bash
apache2ctl -M | grep -E 'log_config|logio'
```

Jika tidak ada logging request, periksa konfigurasi `CustomLog`.

---

### 2.3 Disable Modul yang Tidak Diperlukan

Gunakan pendekatan aman: hanya disable jika aplikasi tidak memerlukannya.

#### WebDAV

```bash
a2dismod dav dav_fs dav_lock 2>/dev/null || true
```

#### Status dan Info

```bash
a2dismod status info 2>/dev/null || true
```

#### Autoindex

```bash
a2dismod autoindex 2>/dev/null || true
```

#### Proxy jika Apache bukan reverse proxy

```bash
for m in proxy proxy_http proxy_fcgi proxy_ajp proxy_balancer proxy_connect proxy_ftp proxy_wstunnel lbmethod_byrequests; do
  a2dismod "$m" 2>/dev/null || true
done
```

#### UserDir

```bash
a2dismod userdir 2>/dev/null || true
```

#### Basic dan Digest Auth jika tidak dipakai

```bash
for m in auth_basic auth_digest; do
  a2dismod "$m" 2>/dev/null || true
done
```

Validasi:

```bash
apache2ctl configtest
systemctl reload apache2.service
apache2ctl -M | egrep 'dav|status|autoindex|proxy|userdir|info|auth_basic|auth_digest' || true
```

> **Exception umum:** jika Apache memang reverse proxy, jangan disable semua `proxy_*`; batasi hanya modul proxy yang diperlukan dan dokumentasikan.

---

## 3. Principles, Permissions, and Ownership

### 3.1 Apache Berjalan sebagai Non-Root User

Pada Debian/Ubuntu, akun biasanya `www-data`.

Audit:

```bash
grep -E '^export APACHE_RUN_USER|^export APACHE_RUN_GROUP' /etc/apache2/envvars
ps -eo user,group,comm,args | grep '[a]pache2'
getent passwd www-data
```

Remediation:

```bash
sed -i 's/^export APACHE_RUN_USER=.*/export APACHE_RUN_USER=www-data/' /etc/apache2/envvars
sed -i 's/^export APACHE_RUN_GROUP=.*/export APACHE_RUN_GROUP=www-data/' /etc/apache2/envvars
```

### 3.2 Service Account Tidak Bisa Login dan Dikunci

```bash
usermod -s /usr/sbin/nologin www-data 2>/dev/null || true
passwd -l www-data 2>/dev/null || true
getent passwd www-data
passwd -S www-data 2>/dev/null || true
```

### 3.3 Ownership dan Permission Direktori Apache

Audit:

```bash
stat -c '%n %U:%G %a' /etc/apache2 /usr/sbin/apache2 /usr/lib/apache2/modules /var/log/apache2 /var/www
find /etc/apache2 -xdev -type f -perm /022 -ls
find /etc/apache2 -xdev -type d -perm /022 -ls
```

Remediation konservatif:

```bash
chown -R root:root /etc/apache2
find /etc/apache2 -type d -exec chmod 755 {} \;
find /etc/apache2 -type f -exec chmod 644 {} \;
chmod 600 /etc/apache2/envvars 2>/dev/null || true

chown root:root /usr/sbin/apache2 2>/dev/null || true
chmod 755 /usr/sbin/apache2 2>/dev/null || true
chown -R root:root /usr/lib/apache2/modules 2>/dev/null || true
find /usr/lib/apache2/modules -type f -exec chmod 644 {} \; 2>/dev/null || true
```

### 3.4 DocumentRoot Tidak Writable oleh Apache Group

Audit:

```bash
find /var/www -xdev -group www-data -perm /020 -ls 2>/dev/null
find /var/www -xdev -user www-data -ls 2>/dev/null
find /var/www -xdev -perm /002 -ls 2>/dev/null
```

Remediation umum untuk static site:

```bash
chown -R root:root /var/www
find /var/www -type d -exec chmod 755 {} \;
find /var/www -type f -exec chmod 644 {} \;
```

Jika aplikasi butuh upload/cache, buat direktori khusus di luar web root:

```bash
mkdir -p /var/lib/myapp/uploads
chown root:www-data /var/lib/myapp/uploads
chmod 750 /var/lib/myapp/uploads
```

---

## 4. Apache Access Control

### 4.1 Deny OS Root Directory by Default

Buat konfigurasi baseline:

```bash
cat >/etc/apache2/conf-available/00-cis-root-deny.conf <<'EOF'
<Directory />
    Require all denied
    AllowOverride None
    Options None
</Directory>
EOF

a2enconf 00-cis-root-deny
apache2ctl configtest
systemctl reload apache2.service
```

Audit:

```bash
apache2ctl -t -D DUMP_RUN_CFG | sed -n '/Directory/,/Main DocumentRoot/p'
grep -R "<Directory />" -n /etc/apache2/
```

### 4.2 Allow Hanya Web Content yang Sah

Contoh untuk document root `/var/www/html`:

```bash
cat >/etc/apache2/conf-available/10-cis-webroot-access.conf <<'EOF'
<Directory /var/www/html>
    Require all granted
    AllowOverride None
    Options None
</Directory>
EOF

a2enconf 10-cis-webroot-access
apache2ctl configtest
systemctl reload apache2.service
```

Jika admin panel hanya boleh subnet internal:

```apache
<Directory /var/www/html/admin>
    Require ip 192.168.10.0/24
    AllowOverride None
    Options None
</Directory>
```

---

## 5. Minimize Features, Content, and Options

### 5.1 Batasi Options

Root directory:

```apache
<Directory />
    Options None
    AllowOverride None
    Require all denied
</Directory>
```

Web root:

```apache
<Directory /var/www/html>
    Options None
    AllowOverride None
    Require all granted
</Directory>
```

Audit:

```bash
grep -R "Options" /etc/apache2/apache2.conf /etc/apache2/conf-enabled /etc/apache2/sites-enabled 2>/dev/null
grep -R "AllowOverride" /etc/apache2/apache2.conf /etc/apache2/conf-enabled /etc/apache2/sites-enabled 2>/dev/null
```

### 5.2 Hapus Default Content

Audit:

```bash
find /var/www -maxdepth 3 -type f | sort
find /usr/lib/cgi-bin -maxdepth 1 -type f 2>/dev/null | sort
```

Remediation contoh:

```bash
rm -f /var/www/html/index.html
rm -f /usr/lib/cgi-bin/printenv /usr/lib/cgi-bin/test-cgi 2>/dev/null || true
```

> Jangan hapus file aplikasi produksi. Pastikan file yang dihapus memang default/demo/testing.

### 5.3 Batasi HTTP Methods dan Matikan TRACE

```bash
cat >/etc/apache2/conf-available/20-cis-methods.conf <<'EOF'
TraceEnable Off

<Directory /var/www/html>
    <LimitExcept GET POST HEAD OPTIONS>
        Require all denied
    </LimitExcept>
</Directory>
EOF

a2enconf 20-cis-methods
apache2ctl configtest
systemctl reload apache2.service
```

Audit:

```bash
grep -R "TraceEnable" /etc/apache2/
curl -i -X TRACE http://127.0.0.1/ 2>/dev/null | head
```

### 5.4 Tolak HTTP Protocol Lama / Invalid

Apache 2.4 mendukung directive `HttpProtocolOptions`.

```bash
cat >/etc/apache2/conf-available/21-cis-http-protocol.conf <<'EOF'
HttpProtocolOptions Strict
EOF

a2enconf 21-cis-http-protocol
apache2ctl configtest
systemctl reload apache2.service
```

### 5.5 Blokir File Sensitif: `.ht*`, `.git`, `.svn`, Backup, Config

```bash
cat >/etc/apache2/conf-available/30-cis-sensitive-files.conf <<'EOF'
<FilesMatch "^\.ht">
    Require all denied
</FilesMatch>

<DirectoryMatch "(^|/)\.(git|svn)(/|$)">
    Require all denied
</DirectoryMatch>

<FilesMatch "\.(bak|old|orig|save|swp|sql|tar|tgz|gz|zip|7z|env|ini|conf|config|log)$">
    Require all denied
</FilesMatch>
EOF

a2enconf 30-cis-sensitive-files
apache2ctl configtest
systemctl reload apache2.service
```

### 5.6 Tolak Request Berbasis IP Address

Aktifkan rewrite:

```bash
a2enmod rewrite
```

Tambahkan ke virtual host:

```apache
RewriteEngine On
RewriteCond %{HTTP_HOST} ^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$
RewriteRule ^ - [F]
```

Contoh file khusus:

```bash
cat >/etc/apache2/conf-available/31-cis-no-ip-host.conf <<'EOF'
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{HTTP_HOST} ^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$
    RewriteRule ^ - [F]
</IfModule>
EOF

a2enconf 31-cis-no-ip-host
apache2ctl configtest
systemctl reload apache2.service
```

### 5.7 Listen pada IP Eksplisit

Edit `/etc/apache2/ports.conf`:

```apache
Listen 192.0.2.10:80
Listen 192.0.2.10:443
```

Validasi:

```bash
grep -R "^Listen" /etc/apache2/ports.conf /etc/apache2/sites-enabled/ 2>/dev/null
ss -tulpen | grep apache2
```

### 5.8 Header Keamanan Browser

```bash
a2enmod headers
cat >/etc/apache2/conf-available/40-cis-security-headers.conf <<'EOF'
Header always set X-Frame-Options "SAMEORIGIN"
Header always set Content-Security-Policy "frame-ancestors 'self'"
Header always set Referrer-Policy "no-referrer"
Header always set Permissions-Policy "geolocation=(), microphone=(), camera=(), payment=()"
EOF

a2enconf 40-cis-security-headers
apache2ctl configtest
systemctl reload apache2.service
```

Audit:

```bash
curl -I http://127.0.0.1/ 2>/dev/null | egrep -i 'x-frame-options|content-security-policy|referrer-policy|permissions-policy'
```

---

## 6. Logging, Monitoring, and Maintenance

### 6.1 ErrorLog, LogLevel, AccessLog

```bash
cat >/etc/apache2/conf-available/50-cis-logging.conf <<'EOF'
LogLevel warn
ErrorLog ${APACHE_LOG_DIR}/error.log
LogFormat "%v:%p %h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" vhost_combined
CustomLog ${APACHE_LOG_DIR}/access.log vhost_combined
EOF

a2enconf 50-cis-logging
apache2ctl configtest
systemctl reload apache2.service
```

Audit:

```bash
grep -R "LogLevel\|ErrorLog\|CustomLog\|LogFormat" /etc/apache2/
tail -n 20 /var/log/apache2/access.log
tail -n 20 /var/log/apache2/error.log
```

### 6.2 Syslog Facility untuk ErrorLog

Jika organisasi memakai syslog/SIEM:

```apache
ErrorLog syslog:local1
```

Konfigurasi rsyslog contoh:

```bash
cat >/etc/rsyslog.d/30-apache.conf <<'EOF'
local1.*    /var/log/apache2/error_syslog.log
EOF
systemctl restart rsyslog.service
```

### 6.3 Logrotate

Audit:

```bash
cat /etc/logrotate.d/apache2
logrotate -d /etc/logrotate.conf | grep -i apache -A5
```

Contoh baseline:

```bash
cat >/etc/logrotate.d/apache2 <<'EOF'
/var/log/apache2/*.log {
    weekly
    rotate 13
    missingok
    notifempty
    compress
    delaycompress
    sharedscripts
    create 640 root adm
    postrotate
        if systemctl is-active --quiet apache2.service; then
            systemctl reload apache2.service > /dev/null 2>&1 || true
        fi
    endscript
}
EOF
```

### 6.4 Patch Apache

```bash
apt update
apt -y upgrade apache2
apache2 -v
apt list --upgradable | grep apache2 || true
```

### 6.5 ModSecurity dan OWASP CRS

Install:

```bash
apt install -y libapache2-mod-security2 modsecurity-crs
a2enmod security2
```

Aktifkan engine:

```bash
cp -n /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf 2>/dev/null || true
sed -i 's/^SecRuleEngine .*/SecRuleEngine On/' /etc/modsecurity/modsecurity.conf
```

Pastikan CRS diload. Pada Ubuntu/Debian, file integrasi bisa berbeda. Audit:

```bash
ls -l /usr/share/modsecurity-crs/ /etc/modsecurity/ 2>/dev/null
grep -R "modsecurity-crs\|REQUEST-900" /etc/apache2 /etc/modsecurity 2>/dev/null || true
```

Contoh include manual bila belum ada:

```bash
cat >/etc/modsecurity/crs.conf <<'EOF'
IncludeOptional /usr/share/modsecurity-crs/*.load
IncludeOptional /usr/share/modsecurity-crs/rules/*.conf
EOF
```

Validasi:

```bash
apache2ctl configtest
systemctl restart apache2.service
apache2ctl -M | grep security2
grep SecRuleEngine /etc/modsecurity/modsecurity.conf
```

> ModSecurity/CRS perlu tuning. Jangan langsung produksi tanpa monitoring false positive.

---

## 7. SSL/TLS Configuration

### 7.1 Aktifkan SSL dan Header

```bash
a2enmod ssl headers socache_shmcb
systemctl reload apache2.service
```

### 7.2 Sertifikat dan Private Key

Contoh path:

```apache
SSLCertificateFile /etc/ssl/certs/example.fullchain.pem
SSLCertificateKeyFile /etc/ssl/private/example.key
```

Permission private key:

```bash
chown root:root /etc/ssl/private/*.key 2>/dev/null || true
chmod 600 /etc/ssl/private/*.key 2>/dev/null || true
```

Audit:

```bash
find /etc/ssl/private -type f -name '*.key' -exec stat -c '%n %U:%G %a' {} \; 2>/dev/null
```

### 7.3 TLS Protocol, Cipher, Compression, Stapling, HSTS

```bash
cat >/etc/apache2/conf-available/60-cis-tls.conf <<'EOF'
SSLProtocol -all +TLSv1.2 +TLSv1.3
SSLCipherSuite ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256
SSLHonorCipherOrder On
SSLCompression Off
SSLUseStapling On
SSLStaplingCache "shmcb:/var/run/apache2/stapling_cache(128000)"
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
EOF

a2enconf 60-cis-tls
apache2ctl configtest
systemctl reload apache2.service
```

Audit lokal:

```bash
apache2ctl -S
apache2ctl -M | egrep 'ssl|headers|socache'
grep -R "SSLProtocol\|SSLCipherSuite\|SSLCompression\|SSLUseStapling\|Strict-Transport-Security" /etc/apache2/
```

Audit eksternal:

```bash
openssl s_client -connect example.com:443 -servername example.com -tls1_2 </dev/null
openssl s_client -connect example.com:443 -servername example.com -tls1_1 </dev/null
```

> TLSv1.1 seharusnya gagal jika protokol lama berhasil dinonaktifkan.

### 7.4 Redirect HTTP ke HTTPS

Di virtual host port 80:

```apache
<VirtualHost *:80>
    ServerName example.com
    Redirect permanent / https://example.com/
</VirtualHost>
```

---

## 8. Information Leakage

### 8.1 ServerTokens dan ServerSignature

```bash
cat >/etc/apache2/conf-available/70-cis-info-leakage.conf <<'EOF'
ServerTokens Prod
ServerSignature Off
FileETag MTime Size
EOF

a2enconf 70-cis-info-leakage
apache2ctl configtest
systemctl reload apache2.service
```

Audit:

```bash
curl -I http://127.0.0.1/ 2>/dev/null | grep -i '^Server:'
grep -R "ServerTokens\|ServerSignature\|FileETag" /etc/apache2/
```

### 8.2 Hapus Konten Default

```bash
find /var/www -type f \( -name '*default*' -o -name '*example*' -o -name '*test*' \) -print
find /usr/share/apache2 -maxdepth 2 -type f 2>/dev/null | sort
```

Jangan hapus dokumentasi OS sembarangan; pastikan tidak disajikan melalui DocumentRoot.

---

## 9. Denial of Service Mitigations

Aktifkan `reqtimeout`:

```bash
a2enmod reqtimeout
```

Konfigurasi:

```bash
cat >/etc/apache2/conf-available/80-cis-dos.conf <<'EOF'
TimeOut 10
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 15

<IfModule reqtimeout_module>
    RequestReadTimeout header=20-40,MinRate=500 body=20,MinRate=500
</IfModule>
EOF

a2enconf 80-cis-dos
apache2ctl configtest
systemctl reload apache2.service
```

Audit:

```bash
grep -R "TimeOut\|KeepAlive\|MaxKeepAliveRequests\|KeepAliveTimeout\|RequestReadTimeout" /etc/apache2/
apache2ctl -M | grep reqtimeout
```

---

## 10. Request Limits

```bash
cat >/etc/apache2/conf-available/90-cis-request-limits.conf <<'EOF'
LimitRequestLine 8190
LimitRequestFields 100
LimitRequestFieldSize 8190
LimitRequestBody 102400
EOF

a2enconf 90-cis-request-limits
apache2ctl configtest
systemctl reload apache2.service
```

Audit:

```bash
grep -R "LimitRequestLine\|LimitRequestFields\|LimitRequestFieldSize\|LimitRequestBody" /etc/apache2/
```

> Upload aplikasi mungkin membutuhkan limit lebih besar. Jika diubah, dokumentasikan exception dan alasan bisnis.

---

## 11. SELinux

SELinux lebih umum pada RHEL-family. Pada Ubuntu/Debian, Apache biasanya lebih relevan diamankan dengan AppArmor. Jika distribusi memakai SELinux:

```bash
getenforce
sestatus
ps -eZ | grep -E 'httpd|apache2'
semanage boolean -l | grep httpd
```

Untuk Ubuntu/Debian, bagian ini biasanya **Not Applicable** atau diganti AppArmor.

---

## 12. AppArmor untuk Apache

Install tools:

```bash
apt install -y apparmor apparmor-utils apparmor-profiles apparmor-profiles-extra
systemctl enable --now apparmor.service
```

Audit:

```bash
aa-status
aa-status | grep -i apache || true
ls -l /etc/apparmor.d/ | grep -i apache || true
```

Jika profile Apache tersedia:

```bash
aa-enforce /etc/apparmor.d/usr.sbin.apache2 2>/dev/null || true
systemctl restart apache2.service
aa-status | grep -i apache || true
```

Jika belum ada profile, buat/tuning di lab:

```bash
aa-autodep /usr/sbin/apache2
aa-complain /usr/sbin/apache2
# Uji aplikasi, lalu review log:
aa-logprof
aa-enforce /usr/sbin/apache2
```

> Jangan langsung enforce profile custom tanpa uji seluruh fungsi aplikasi.

---

## 13. Validasi Akhir

```bash
# Service
systemctl is-enabled apache2.service
systemctl is-active apache2.service

# Syntax dan virtualhost
apache2ctl configtest
apache2ctl -S
apache2ctl -M | sort

# Modul yang seharusnya tidak aktif
apache2ctl -M | egrep 'dav|status|autoindex|userdir|info|auth_basic|auth_digest' || true

# Permission
stat -c '%n %U:%G %a' /etc/apache2 /usr/sbin/apache2 /var/log/apache2 /var/www
find /etc/apache2 -perm /022 -ls

# Access control dan options
grep -R "Require all denied\|AllowOverride None\|Options None" /etc/apache2/

# Security headers
curl -I http://127.0.0.1/ 2>/dev/null | egrep -i 'server:|x-frame-options|content-security-policy|referrer-policy|permissions-policy|strict-transport-security'

# Logs
tail -n 20 /var/log/apache2/access.log
tail -n 20 /var/log/apache2/error.log
```

---

## 14. Template Exception Register

| Area | Rekomendasi | Status | Alasan Exception | Risiko | Mitigasi | Review |
|---|---|---|---|---|---|---|
| Proxy modules | Disable proxy if not in use | Exception | Apache dipakai reverse proxy | Open proxy jika salah konfigurasi | Batasi ProxyPass, tidak enable forward proxy |  |
| Basic Auth | Disable Basic Auth | Exception | Area admin internal masih pakai Basic Auth | Credential reuse/header exposure | HTTPS wajib, IP allowlist, rencana migrasi SSO |  |
| Request body | LimitRequestBody 102400 | Exception | Aplikasi butuh upload dokumen | DoS/storage abuse | Limit per Location, AV scan, ukuran upload dibatasi aplikasi |  |
| AppArmor | Enforce Apache profile | Planned | Profile belum diuji | Compromise impact lebih luas | Complain mode + aa-logprof + testing |  |

---

## 15. Ringkasan Urutan Implementasi

1. Backup konfigurasi.
2. Install Apache dari repository resmi.
3. Audit service dan pastikan server tidak multi-use tanpa alasan.
4. Audit modul, disable modul tidak perlu.
5. Pastikan Apache berjalan sebagai user non-root dan akun service dikunci.
6. Perbaiki ownership/permission.
7. Terapkan deny OS root dan allow web root eksplisit.
8. Batasi Options, HTTP method, TRACE, file sensitif, IP Host, Listen IP, dan header browser.
9. Konfigurasi logging, logrotate, patching, ModSecurity/CRS bila diperlukan.
10. Konfigurasi TLS dengan protokol/cipher kuat, HSTS, OCSP stapling.
11. Kurangi information leakage.
12. Terapkan DoS mitigation dan request limit.
13. Terapkan AppArmor/SELinux sesuai distro.
14. Validasi akhir dan dokumentasikan exception.
