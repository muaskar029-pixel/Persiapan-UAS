# Implementasi Instalasi Ubuntu 24.04 LTS dan Post-Installation Hardening Berbasis CIS Ubuntu Linux 24.04 LTS STIG Benchmark

> **Dokumen kerja:** `06_Implementasi_Instalasi_Post_Installation_Hardening_Ubuntu_24.04_STIG.md`  
> **Basis utama:** CIS Ubuntu Linux 24.04 LTS STIG Benchmark v1.0.0  
> **Sasaran:** Ubuntu 24.04 LTS Server/Workstation  
> **Tujuan:** Membuat panduan implementasi dari tahap instalasi sampai post-installation hardening dengan pendekatan STIG/CIS.

---

## 0. Catatan Penting Sebelum Implementasi

Dokumen ini dibuat sebagai **panduan implementasi praktis**, bukan pengganti audit resmi. CIS Ubuntu Linux 24.04 LTS STIG Benchmark berisi aturan STIG untuk Canonical Ubuntu 24.04 LTS. Banyak aturan dapat langsung diterapkan, tetapi sebagian aturan membutuhkan keputusan organisasi, misalnya FIPS, smart card, PIV/CAC, DoD PKI, remote audit server, serta banner legal yang memakai teks DoD.

Prinsip implementasi:

1. **Uji di lab terlebih dahulu.** Jangan langsung menerapkan semua hardening ke sistem produksi.
2. **Buat backup konfigurasi.** Banyak perubahan menyentuh SSH, PAM, auditd, AppArmor, firewall, dan login policy.
3. **Pastikan ada akses console/recovery.** Kesalahan konfigurasi SSH, UFW, atau PAM dapat membuat admin terkunci.
4. **Gunakan exception register.** Aturan yang tidak relevan harus dicatat alasannya, bukan diabaikan tanpa dokumentasi.
5. **Bedakan baseline akademik dan compliance penuh.** Untuk tugas/lab kampus, cukup jelaskan aturan yang tidak bisa diterapkan karena membutuhkan infrastruktur organisasi.

---

## 1. Pemetaan Tujuan Implementasi

| Area | Tujuan Hardening | Contoh Rule STIG |
|---|---|---|
| Baseline & integritas | Memastikan sistem, konfigurasi, dan audit tools tidak berubah diam-diam | UBTU-24-90890, UBTU-24-100100, UBTU-24-100110, UBTU-24-100120, UBTU-24-100130 |
| Service minimization | Menghapus layanan lama/tidak aman | UBTU-24-100010, UBTU-24-100020, UBTU-24-100030, UBTU-24-100040 |
| Time sync | Waktu konsisten untuk log, audit, dan forensik | UBTU-24-100700, UBTU-24-600160, UBTU-24-600180 |
| Logging & audit | Mengaktifkan rsyslog, auditd, audit rule, audit storage, dan proteksi file audit | UBTU-24-100200, UBTU-24-100400, UBTU-24-100410, UBTU-24-900040 s.d. UBTU-24-909000 |
| Firewall & network | Membatasi akses jaringan dan mengurangi permukaan serangan | UBTU-24-100300, UBTU-24-100310, UBTU-24-600190, UBTU-24-600200, UBTU-24-600230 |
| SSH | Mengamankan pintu administrasi jarak jauh | UBTU-24-100800 s.d. UBTU-24-100860, UBTU-24-600000, UBTU-24-600010 |
| Authentication | Password policy, lockout, root login, sudo, MFA/SSSD bila relevan | UBTU-24-100600, UBTU-24-200250, UBTU-24-400260 s.d. UBTU-24-400400 |
| MAC & kernel | AppArmor, FIPS, ASLR, coredump, USB storage | UBTU-24-100500, UBTU-24-100510, UBTU-24-600030, UBTU-24-700300, UBTU-24-700310 |
| File permission | Permission file sistem, log, library, audit tools, sticky bit | UBTU-24-300006 s.d. UBTU-24-300013, UBTU-24-700030 s.d. UBTU-24-700150, UBTU-24-901230 s.d. UBTU-24-901380 |

---

## 2. Checklist Keputusan Sebelum Instalasi

Isi bagian ini sebelum instalasi agar hardening tidak asal diterapkan.

| Pertanyaan | Jawaban / Catatan |
|---|---|
| Sistem ini server atau workstation? |  |
| Apakah sistem memakai GUI/GNOME? |  |
| Apakah SSH diperlukan? |  |
| IP/subnet admin yang boleh SSH? |  |
| Apakah perlu FIPS mode? |  |
| Apakah organisasi memakai MFA/SSSD/smart card/PIV/PKI? |  |
| Apakah ada remote syslog/audit server? |  |
| Apakah ada layanan bisnis yang harus dibuka di firewall? |  |
| Apakah sistem memakai wireless/Bluetooth? |  |
| Apakah ada pengecualian yang sudah disetujui? |  |

---

## 3. Tahap Instalasi Ubuntu 24.04 LTS

### 3.1 Pilih Edisi dan Mode Instalasi

Gunakan **Ubuntu 24.04 LTS** yang masih vendor-supported. STIG menekankan bahwa rilis yang tidak didukung vendor tidak boleh digunakan karena tidak akan menerima perbaikan keamanan.

Rekomendasi instalasi:

- Untuk server: gunakan **Ubuntu Server 24.04 LTS**.
- Pilih instalasi minimal bila tersedia.
- Jangan memasang paket server yang tidak diperlukan.
- Jangan mengaktifkan layanan jaringan sebelum kebutuhannya jelas.
- Aktifkan OpenSSH hanya jika administrasi jarak jauh memang dibutuhkan.

Validasi setelah instalasi:

```bash
cat /etc/os-release
grep DISTRIB_DESCRIPTION /etc/lsb-release
```

Hasil yang diharapkan mengarah ke Ubuntu 24.04 LTS.

---

### 3.2 Rekomendasi Skema Partisi Hardening

CIS Ubuntu STIG lebih banyak membahas rule operasional, permission, audit, dan konfigurasi. Untuk tahap instalasi, pemisahan partisi berikut memakai prinsip CIS hardening umum agar containment lebih baik.

Contoh skema server:

| Mount Point | Ukuran Contoh | Filesystem | Mount Option Konseptual |
|---|---:|---|---|
| `/boot/efi` | 512 MB | FAT32 | default |
| `/boot` | 1 GB | ext4 | `nodev,nosuid,noexec` bila kompatibel |
| `/` | 20–40 GB | ext4/xfs | default |
| `/tmp` | 2–8 GB | ext4/tmpfs | `nodev,nosuid,noexec` |
| `/var` | 10–30 GB | ext4/xfs | `nodev` |
| `/var/tmp` | 2–8 GB | ext4 | `nodev,nosuid,noexec` |
| `/var/log` | 5–20 GB | ext4/xfs | `nodev,nosuid,noexec` |
| `/var/log/audit` | 5–20 GB | ext4/xfs | `nodev,nosuid,noexec` |
| `/home` | sesuai kebutuhan | ext4/xfs | `nodev,nosuid` |
| swap | sesuai RAM/kebijakan | swap | tidak dieksekusi |

Alasan:

- `/tmp` dan `/var/tmp` sering dipakai aplikasi dan user sebagai tempat file sementara.
- `/var/log` dan `/var/log/audit` perlu dipisahkan agar log tidak memenuhi root filesystem.
- `/home` dipisahkan agar data user tidak bercampur dengan sistem.
- `/var/log/audit` dipisahkan agar audit log punya ruang khusus.

Contoh entri `/etc/fstab` setelah instalasi:

```fstab
UUID=<uuid-tmp>        /tmp             ext4    defaults,nodev,nosuid,noexec      0 2
UUID=<uuid-var>        /var             ext4    defaults,nodev                    0 2
UUID=<uuid-vartmp>     /var/tmp         ext4    defaults,nodev,nosuid,noexec      0 2
UUID=<uuid-varlog>     /var/log         ext4    defaults,nodev,nosuid,noexec      0 2
UUID=<uuid-audit>      /var/log/audit   ext4    defaults,nodev,nosuid,noexec      0 2
UUID=<uuid-home>       /home            ext4    defaults,nodev,nosuid             0 2
```

Validasi:

```bash
findmnt -no TARGET,OPTIONS /tmp /var /var/tmp /var/log /var/log/audit /home
```

---

### 3.3 Enkripsi Disk

Untuk server/laptop yang menyimpan data sensitif, gunakan enkripsi disk saat instalasi. Di Ubuntu installer, opsi ini biasanya tersedia sebagai encrypted LVM/ZFS bergantung mode instalasi.

Catatan:

- STIG tidak selalu memaksa enkripsi disk untuk semua sistem, tetapi enkripsi penting untuk melindungi data saat perangkat hilang atau dicuri.
- Untuk server tanpa akses fisik mudah, disk encryption perlu dirancang agar reboot jarak jauh tetap memungkinkan.
- Simpan recovery key di tempat aman.

---

### 3.4 FIPS Mode Saat Instalasi

Rule STIG UBTU-24-600030 menekankan FIPS-validated cryptography untuk konteks yang membutuhkan standar tersebut. Dokumen STIG menyebut konfigurasi FIPS dilakukan dengan menambahkan parameter `fips=1` saat instalasi, dan mengarahkan ke dokumentasi Ubuntu Pro security certification untuk sistem yang sudah berjalan.

Keputusan implementasi:

| Kondisi | Tindakan |
|---|---|
| Sistem untuk lab/kampus biasa | Catat sebagai **Not Applicable / Exception** bila tidak ada kebutuhan FIPS formal |
| Sistem compliance DoD/federal | Rancang sejak instalasi, gunakan Ubuntu Pro/FIPS sesuai dokumentasi vendor |
| Sistem sudah terpasang | Jangan asal menambahkan `fips=1`; lakukan sesuai dokumentasi vendor dan uji penuh |

Validasi bila FIPS diterapkan:

```bash
cat /proc/sys/crypto/fips_enabled
```

Nilai `1` berarti FIPS mode aktif.

---

### 3.5 Akun Administrator Awal

Saat instalasi:

- Buat akun admin personal, jangan menggunakan akun bersama.
- Gunakan password kuat.
- Jangan aktifkan login root langsung.
- Setelah instalasi, gunakan `sudo` untuk administrasi.

Contoh setelah instalasi:

```bash
id
sudo -v
sudo passwd -l root
```

---

## 4. Post-Installation Hardening — Tahap Awal Aman

Masuk sebagai user admin lalu gunakan root shell sementara:

```bash
sudo -i
```

Buat folder backup konfigurasi:

```bash
mkdir -p /root/hardening-backup/$(date +%F)
BACKUP_DIR="/root/hardening-backup/$(date +%F)"
```

Backup file penting:

```bash
cp -a /etc/ssh "$BACKUP_DIR/ssh" 2>/dev/null || true
cp -a /etc/pam.d "$BACKUP_DIR/pam.d" 2>/dev/null || true
cp -a /etc/audit "$BACKUP_DIR/audit" 2>/dev/null || true
cp -a /etc/rsyslog.conf "$BACKUP_DIR/rsyslog.conf" 2>/dev/null || true
cp -a /etc/rsyslog.d "$BACKUP_DIR/rsyslog.d" 2>/dev/null || true
cp -a /etc/login.defs "$BACKUP_DIR/login.defs" 2>/dev/null || true
cp -a /etc/security "$BACKUP_DIR/security" 2>/dev/null || true
cp -a /etc/default "$BACKUP_DIR/default" 2>/dev/null || true
```

Update sistem:

```bash
apt update
apt -y full-upgrade
apt -y autoremove --purge
apt -y autoclean
```

Reboot bila kernel atau library penting diperbarui:

```bash
reboot
```

---

## 5. Package Management dan Baseline Sistem

### 5.1 Pastikan Ubuntu 24.04 LTS Masih Didukung

```bash
grep DISTRIB_DESCRIPTION /etc/lsb-release
ubuntu-security-status 2>/dev/null || true
```

Bila sistem bukan Ubuntu 24.04 LTS atau sudah tidak didukung, sistem tidak sesuai baseline.

---

### 5.2 Konfigurasi Unattended Upgrades dan Remove Unused Components

Install paket:

```bash
apt install -y unattended-upgrades apt-listchanges
```

Pastikan komponen lama dihapus setelah update:

```bash
cat >/etc/apt/apt.conf.d/52-hardening-remove-unused <<'EOF2'
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Remove-New-Unused-Dependencies "true";
EOF2
```

Validasi:

```bash
grep -R "Remove-Unused" /etc/apt/apt.conf.d/
```

---

### 5.3 Nonaktifkan Weak Dependencies APT — Tambahan CIS Baseline

Ini bukan fokus utama STIG Ubuntu, tetapi sering dipakai dalam hardening CIS untuk mengurangi paket tambahan yang tidak perlu.

```bash
cat >/etc/apt/apt.conf.d/60-no-weak-dependencies <<'EOF2'
APT::Install-Recommends "0";
APT::Install-Suggests "0";
EOF2
```

Validasi:

```bash
apt-config dump | grep 'APT::Install-'
```

---

## 6. Service Minimization dan Time Synchronization

### 6.1 Hapus Layanan Lama/Tidak Aman

STIG menolak layanan lama seperti Telnet dan RSH karena tidak mengenkripsi kredensial dan sesi.

```bash
apt purge -y telnetd rsh-server ntp systemd-timesyncd || true
apt autoremove -y --purge
```

Tambahan paket client/daemon lama yang sebaiknya tidak ada kecuali ada kebutuhan khusus:

```bash
apt purge -y nis talk talkd rsh-client telnet ldap-utils ftp tnftp xinetd || true
apt autoremove -y --purge
```

Validasi:

```bash
dpkg -l | egrep 'telnetd|rsh-server|ntp|systemd-timesyncd|nis|talkd|xinetd' || true
```

Bila ada paket muncul, lakukan review apakah memang dibutuhkan. Jika dibutuhkan, buat exception.

---

### 6.2 Gunakan Chrony sebagai Time Synchronization

Install dan aktifkan Chrony:

```bash
apt install -y chrony
systemctl enable --now chrony.service
```

Edit konfigurasi sumber waktu:

```bash
cp -a /etc/chrony/chrony.conf "$BACKUP_DIR/chrony.conf" 2>/dev/null || true
nano /etc/chrony/chrony.conf
```

Contoh sumber waktu lokal/organisasi:

```conf
pool ntp.ubuntu.com iburst maxsources 4
# atau gunakan NTP internal organisasi:
# server 192.168.10.1 iburst
```

Restart dan validasi:

```bash
systemctl restart chrony.service
systemctl is-enabled chrony.service
systemctl is-active chrony.service
chronyc tracking
chronyc sources -v
```

---

## 7. AppArmor dan Process Protection

### 7.1 Install dan Aktifkan AppArmor

```bash
apt install -y apparmor apparmor-utils apparmor-profiles apparmor-profiles-extra
systemctl enable --now apparmor.service
```

Validasi:

```bash
systemctl is-enabled apparmor.service
systemctl is-active apparmor.service
aa-status
```

### 7.2 Jadikan Profil AppArmor Enforcing

Lihat profil:

```bash
aa-status
```

Ubah profil complain ke enforce jika sesuai dengan kebutuhan aplikasi:

```bash
aa-enforce /etc/apparmor.d/* 2>/dev/null || true
```

Catatan:

- Jangan langsung memaksa semua profil ke enforcing pada server produksi tanpa uji.
- Aplikasi seperti database, web server, container runtime, dan layanan custom bisa gagal bila profil terlalu ketat.

---

### 7.3 Kernel Hardening Dasar

Buat file sysctl:

```bash
cat >/etc/sysctl.d/60-stig-hardening.conf <<'EOF2'
# Network / DoS mitigation
net.ipv4.tcp_syncookies = 1

# IPv4 hardening
net.ipv4.ip_forward = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.default.secure_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1

# IPv6 hardening; sesuaikan bila host ini router IPv6
net.ipv6.conf.all.forwarding = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0
net.ipv6.conf.all.accept_source_route = 0
net.ipv6.conf.default.accept_source_route = 0
net.ipv6.conf.all.accept_ra = 0
net.ipv6.conf.default.accept_ra = 0

# Process and information exposure hardening
kernel.randomize_va_space = 2
kernel.kptr_restrict = 2
kernel.dmesg_restrict = 1
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
EOF2

sysctl --system
```

Validasi:

```bash
sysctl net.ipv4.tcp_syncookies kernel.randomize_va_space kernel.kptr_restrict kernel.dmesg_restrict
```

---

### 7.4 Batasi Core Dump

```bash
cat >/etc/security/limits.d/60-hardening-coredump.conf <<'EOF2'
* hard core 0
EOF2

cat >/etc/sysctl.d/61-coredump.conf <<'EOF2'
fs.suid_dumpable = 0
EOF2

sysctl --system
```

Jika `systemd-coredump` tidak dibutuhkan:

```bash
systemctl disable --now systemd-coredump.socket 2>/dev/null || true
apt purge -y systemd-coredump 2>/dev/null || true
```

---

### 7.5 USB Mass Storage

Jika server tidak membutuhkan USB storage, nonaktifkan modul:

```bash
cat >/etc/modprobe.d/usb-storage.conf <<'EOF2'
install usb-storage /bin/false
blacklist usb-storage
EOF2

modprobe -r usb-storage 2>/dev/null || true
```

Validasi:

```bash
modprobe -n -v usb-storage
lsmod | grep usb_storage || true
```

---

## 8. Firewall dan Network Access Control

### 8.1 Install dan Aktifkan UFW

```bash
apt install -y ufw
```

Aturan dasar:

```bash
ufw default deny incoming
ufw default allow outgoing
ufw default deny routed
```

Jika SSH diperlukan, izinkan dari subnet admin saja. Ganti `192.168.10.0/24` sesuai jaringanmu.

```bash
ufw allow from 192.168.10.0/24 to any port 22 proto tcp comment 'Allow SSH from admin subnet'
ufw limit 22/tcp comment 'Rate limit SSH'
```

Jika server web:

```bash
ufw allow 80/tcp comment 'HTTP'
ufw allow 443/tcp comment 'HTTPS'
```

Aktifkan:

```bash
ufw --force enable
ufw status verbose
systemctl enable --now ufw.service
```

Validasi:

```bash
ufw status numbered
ss -tulpen
```

Catatan penting:

- Sebelum mengaktifkan UFW di server remote, pastikan rule SSH sudah benar.
- Semua port yang terbuka harus memiliki alasan bisnis/operasional.
- Rule firewall harus cocok dengan daftar layanan yang memang disetujui.

---

### 8.2 Nonaktifkan Wireless/Bluetooth pada Server Bila Tidak Dibutuhkan

Untuk server, wireless dan Bluetooth biasanya tidak diperlukan.

```bash
systemctl disable --now bluetooth.service 2>/dev/null || true
apt purge -y bluez 2>/dev/null || true
```

Opsional blacklist modul wireless/Bluetooth bila benar-benar tidak dipakai:

```bash
cat >/etc/modprobe.d/disable-wireless-bluetooth.conf <<'EOF2'
# Sesuaikan dengan hardware. Jangan aktifkan jika sistem memang memakai Wi-Fi/Bluetooth.
blacklist bluetooth
blacklist btusb
EOF2
```

Validasi:

```bash
rfkill list 2>/dev/null || true
systemctl status bluetooth.service 2>/dev/null || true
```

---

## 9. SSH Hardening

### 9.1 Install dan Aktifkan SSH Jika Dibutuhkan

```bash
apt install -y ssh
systemctl enable --now ssh.service
```

Validasi:

```bash
systemctl is-enabled ssh.service
systemctl is-active ssh.service
ss -tulpen | grep ':22'
```

Jika SSH tidak dibutuhkan:

```bash
systemctl disable --now ssh.service
apt purge -y openssh-server
```

---

### 9.2 Permission File SSH

```bash
chown root:root /etc/ssh/sshd_config
chmod 600 /etc/ssh/sshd_config

find /etc/ssh -xdev -type f -name 'ssh_host_*_key' -exec chown root:root {} \; -exec chmod 600 {} \;
find /etc/ssh -xdev -type f -name 'ssh_host_*_key.pub' -exec chown root:root {} \; -exec chmod 644 {} \;
```

---

### 9.3 Konfigurasi SSH Server

Buat file drop-in agar perubahan mudah dikelola:

```bash
mkdir -p /etc/ssh/sshd_config.d
cat >/etc/ssh/sshd_config.d/60-stig-hardening.conf <<'EOF2'
# CIS Ubuntu 24.04 LTS STIG-oriented SSH hardening

# Legal/login banner
Banner /etc/issue.net

# Session timeout
ClientAliveInterval 600
ClientAliveCountMax 1

# Authentication hardening
PermitRootLogin no
PermitEmptyPasswords no
HostbasedAuthentication no
IgnoreRhosts yes
GSSAPIAuthentication no
PermitUserEnvironment no
UsePAM yes
LoginGraceTime 60
MaxAuthTries 4
MaxSessions 10
MaxStartups 10:30:60
LogLevel VERBOSE

# Forwarding and GUI related restrictions
X11Forwarding no
AllowTcpForwarding no
AllowAgentForwarding no
PermitTunnel no
PermitOpen none

# Cryptographic algorithms from STIG guidance
Ciphers aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes128-ctr
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512,hmac-sha2-256
KexAlgorithms ecdh-sha2-nistp521,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group-exchange-sha256,diffie-hellman-group16-sha512,diffie-hellman-group14-sha256
EOF2
```

Validasi syntax sebelum restart:

```bash
sshd -t
```

Restart:

```bash
systemctl restart ssh.service
```

Validasi efektif:

```bash
sshd -T | egrep 'banner|clientaliveinterval|clientalivecountmax|permitrootlogin|permitemptypasswords|x11forwarding|ciphers|macs|kexalgorithms|loglevel|maxauthtries|maxsessions|maxstartups'
```

Catatan:

- `AllowTcpForwarding no`, `AllowAgentForwarding no`, dan `PermitOpen none` bisa mengganggu jump host, port forwarding, deployment, dan tunneling. Jika ada kebutuhan bisnis, catat exception.
- Jika memakai `Match` block di `sshd_config`, pastikan drop-in tidak konflik.

---

### 9.4 Konfigurasi SSH Client

STIG juga memuat penguatan SSH client. Buat drop-in:

```bash
mkdir -p /etc/ssh/ssh_config.d
cat >/etc/ssh/ssh_config.d/60-stig-hardening.conf <<'EOF2'
Host *
    Ciphers aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes128-ctr
    MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512,hmac-sha2-256
EOF2
```

Validasi:

```bash
ssh -G localhost | egrep 'ciphers|macs' | head
```

---

### 9.5 Banner SSH dan Login Banner

Buat banner singkat non-informatif. Hindari menampilkan versi OS, hostname detail, kernel, atau informasi internal.

```bash
cat >/etc/issue <<'EOF2'
Authorized users only. All activity may be monitored and reported.
EOF2

cat >/etc/issue.net <<'EOF2'
Authorized users only. All activity may be monitored and reported.
EOF2

cat >/etc/motd <<'EOF2'
Authorized use only.
EOF2

chmod 644 /etc/issue /etc/issue.net /etc/motd
chown root:root /etc/issue /etc/issue.net /etc/motd
```

Catatan STIG:

- Dalam lingkungan DoD, teks banner harus mengikuti Standard Mandatory DoD Notice and Consent Banner.
- Untuk lab/kampus, banner bisa disesuaikan selama tidak membocorkan informasi sistem.

---

### 9.6 Acknowledgement Banner untuk SSH — Opsional/DoD-Specific

STIG memiliki rule manual yang meminta user melakukan acknowledgement untuk banner SSH. Untuk lab non-DoD, bagian ini bisa dicatat sebagai exception.

Contoh sederhana:

```bash
cat >/etc/profile.d/ssh_confirm.sh <<'EOF2'
#!/bin/bash
if [ -n "$SSH_CLIENT" ] || [ -n "$SSH_TTY" ]; then
  while true; do
    read -r -p "Authorized use only. Activity may be monitored. Do you agree? [y/N] " yn
    case "$yn" in
      [Yy]*) break ;;
      [Nn]*|"") exit 1 ;;
      *) echo "Please answer yes or no." ;;
    esac
  done
fi
EOF2
chmod 644 /etc/profile.d/ssh_confirm.sh
```

---

## 10. Authentication, PAM, Password Policy, dan Account Lockout

### 10.1 Install PAM Password Quality

```bash
apt install -y libpam-pwquality cracklib-runtime
```

Validasi:

```bash
dpkg -l | grep libpam-pwquality
```

---

### 10.2 Password Complexity

Konfigurasi `/etc/security/pwquality.conf`:

```bash
cp -a /etc/security/pwquality.conf "$BACKUP_DIR/pwquality.conf" 2>/dev/null || true
cat >/etc/security/pwquality.conf <<'EOF2'
# CIS/STIG-oriented password quality baseline
minlen = 15
dcredit = -1
ucredit = -1
ocredit = -1
lcredit = -1
maxrepeat = 3
maxclassrepeat = 4
dictcheck = 1
enforcing = 1
retry = 3
EOF2
```

Penjelasan ringkas:

- `minlen = 15`: panjang minimum password.
- `dcredit/ucredit/ocredit/lcredit = -1`: mensyaratkan angka, huruf besar, simbol, dan huruf kecil.
- `dictcheck = 1`: cek dictionary.
- `retry = 3`: batas percobaan saat mengganti password.

---

### 10.3 Password Aging

Edit `/etc/login.defs`:

```bash
cp -a /etc/login.defs "$BACKUP_DIR/login.defs.before-aging"
sed -i 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS   60/' /etc/login.defs
sed -i 's/^PASS_MIN_DAYS.*/PASS_MIN_DAYS   1/' /etc/login.defs
sed -i 's/^PASS_WARN_AGE.*/PASS_WARN_AGE   7/' /etc/login.defs
```

Terapkan ke user interaktif yang sudah ada:

```bash
awk -F: '($3>=1000)&&($1!="nobody"){print $1}' /etc/passwd | while read -r user; do
  chage --maxdays 60 --mindays 1 --warndays 7 "$user"
done
```

Validasi:

```bash
grep -E '^PASS_MAX_DAYS|^PASS_MIN_DAYS|^PASS_WARN_AGE' /etc/login.defs
chage -l <nama_user>
```

---

### 10.4 Account Lockout untuk Login Gagal

Ubuntu 24.04 dapat memakai `pam_faillock`. Karena perubahan PAM bisa berisiko mengunci sistem, lakukan dari console atau pastikan ada sesi root/admin cadangan.

Install bila perlu:

```bash
apt install -y libpam-modules
```

Buat file konfigurasi faillock:

```bash
cat >/etc/security/faillock.conf <<'EOF2'
deny = 3
fail_interval = 900
unlock_time = 0
even_deny_root
root_unlock_time = 900
EOF2
```

Catatan:

- `deny = 3`: lock setelah 3 kegagalan.
- `unlock_time = 0`: tetap terkunci sampai admin membuka, cocok untuk kebijakan ketat.
- Untuk lab agar tidak sering terkunci, bisa gunakan `unlock_time = 900` dan catat sebagai penyesuaian.

Validasi manual:

```bash
faillock --user <nama_user>
```

Reset lockout:

```bash
faillock --user <nama_user> --reset
```

---

### 10.5 Password History

Install modul bila tersedia:

```bash
apt install -y libpam-pwquality
```

Tambahkan `pam_pwhistory` pada `/etc/pam.d/common-password` secara hati-hati. Contoh baris yang biasa dipakai:

```pam
password requisite pam_pwhistory.so remember=5 use_authtok
```

Catatan:

- Jangan menambahkan duplikasi baris PAM.
- Backup file sebelum edit.
- Uji dengan user non-kritis terlebih dahulu.

---

### 10.6 Null Password dan Blank Password

Pastikan tidak ada akun dengan password kosong:

```bash
awk -F: '($2==""){print $1}' /etc/shadow
```

Jika ada:

```bash
passwd -l <nama_user>
```

Pastikan `nullok` tidak dipakai di PAM:

```bash
grep -R "nullok" /etc/pam.d/ || true
```

Jika ditemukan, review dan hapus dari konfigurasi PAM yang relevan.

---

## 11. Privilege Escalation: sudo dan su

### 11.1 Install dan Batasi sudo

```bash
apt install -y sudo
```

Buat konfigurasi sudo hardening:

```bash
cat >/etc/sudoers.d/60-stig-hardening <<'EOF2'
Defaults use_pty
Defaults logfile="/var/log/sudo.log"
Defaults timestamp_timeout=5
Defaults passwd_timeout=1
EOF2
chmod 440 /etc/sudoers.d/60-stig-hardening
visudo -cf /etc/sudoers.d/60-stig-hardening
```

Validasi:

```bash
sudo -l
ls -l /var/log/sudo.log 2>/dev/null || true
```

### 11.2 Restrict `su`

Pastikan hanya group tertentu yang boleh `su`. Contoh menggunakan group `sugroup`:

```bash
groupadd -f sugroup
```

Edit `/etc/pam.d/su`, pastikan ada baris:

```pam
auth required pam_wheel.so use_uid group=sugroup
```

Tambahkan user admin yang benar-benar boleh `su`:

```bash
usermod -aG sugroup <nama_admin>
```

Validasi:

```bash
grep pam_wheel /etc/pam.d/su
getent group sugroup
```

---

## 12. Logging: rsyslog dan Log Permission

### 12.1 Install dan Aktifkan rsyslog

```bash
apt install -y rsyslog
systemctl enable --now rsyslog.service
```

Validasi:

```bash
systemctl is-enabled rsyslog.service
systemctl is-active rsyslog.service
```

### 12.2 Mode File Log

Buat konfigurasi file mode:

```bash
cat >/etc/rsyslog.d/60-stig-filecreate.conf <<'EOF2'
$FileCreateMode 0640
EOF2
systemctl restart rsyslog.service
```

Validasi:

```bash
grep -R "FileCreateMode" /etc/rsyslog.conf /etc/rsyslog.d/*.conf
```

### 12.3 Remote Logging — Jika Ada Server Log

Jika organisasi memiliki server log:

```bash
cat >/etc/rsyslog.d/70-remote-log.conf <<'EOF2'
*.* @@192.168.10.50:6514
EOF2
systemctl restart rsyslog.service
```

Catatan:

- `@@` berarti TCP.
- Untuk compliance kuat, gunakan TLS dan CA certificate.
- Jika tidak ada remote log server, catat sebagai exception/roadmap.

### 12.4 Permission Logfile

Audit permission:

```bash
find /var/log -type f -perm /137 -ls
```

Perbaikan konservatif:

```bash
find /var/log -type f -exec chmod go-wx {} \;
find /var/log -type d -exec chmod go-w {} \;
```

Jangan ubah permission secara membabi buta pada aplikasi yang punya kebutuhan khusus tanpa uji.

---

## 13. Auditing: auditd, Audit Storage, dan Audit Rules

### 13.1 Install dan Aktifkan auditd

```bash
apt install -y auditd audispd-plugins
systemctl enable --now auditd.service
```

Validasi:

```bash
systemctl is-enabled auditd.service
systemctl is-active auditd.service
auditctl -s
```

---

### 13.2 Audit Kernel Parameter Saat Boot

Pastikan audit aktif sejak boot awal. Edit `/etc/default/grub`:

```bash
cp -a /etc/default/grub "$BACKUP_DIR/grub.before-audit"
sed -i 's/GRUB_CMDLINE_LINUX="/GRUB_CMDLINE_LINUX="audit=1 audit_backlog_limit=8192 /' /etc/default/grub
update-grub
```

Validasi setelah reboot:

```bash
cat /proc/cmdline | grep -E 'audit=1|audit_backlog_limit=8192'
```

---

### 13.3 Audit Log Retention dan Reaksi Saat Penuh

Edit `/etc/audit/auditd.conf`:

```bash
cp -a /etc/audit/auditd.conf "$BACKUP_DIR/auditd.conf.before-retention"
sed -i 's/^max_log_file .*/max_log_file = 100/' /etc/audit/auditd.conf
sed -i 's/^max_log_file_action .*/max_log_file_action = keep_logs/' /etc/audit/auditd.conf
sed -i 's/^space_left_action .*/space_left_action = email/' /etc/audit/auditd.conf
sed -i 's/^admin_space_left_action .*/admin_space_left_action = halt/' /etc/audit/auditd.conf
sed -i 's/^disk_full_action .*/disk_full_action = halt/' /etc/audit/auditd.conf
sed -i 's/^disk_error_action .*/disk_error_action = halt/' /etc/audit/auditd.conf
systemctl restart auditd.service
```

Catatan:

- `halt` sangat ketat dan bisa menghentikan server jika storage audit penuh.
- Untuk lab, bisa gunakan `single` atau `syslog`, tetapi catat sebagai deviasi.

Validasi:

```bash
grep -E 'max_log_file|max_log_file_action|space_left_action|admin_space_left_action|disk_full_action|disk_error_action' /etc/audit/auditd.conf
```

---

### 13.4 Remote Audit Offloading — Jika Ada Server Audit

Jika organisasi memiliki audit server:

```bash
sed -i -E 's/^active\s*=.*/active = yes/' /etc/audit/plugins.d/au-remote.conf
sed -i -E 's/^remote_server\s*=.*/remote_server = 192.168.10.60/' /etc/audit/audisp-remote.conf
systemctl restart auditd.service
```

Validasi:

```bash
grep -i '^active' /etc/audit/plugins.d/au-remote.conf
grep -i '^remote_server' /etc/audit/audisp-remote.conf
```

Jika tidak ada audit server, buat exception.

---

### 13.5 Audit Rules Dasar

Buat file rules. Ini ringkasan implementatif untuk mencakup area STIG: identitas, sudo, waktu, network environment, permission changes, file deletion, mount, session, login/logout, kernel module, dan privileged commands.

```bash
UID_MIN=$(awk '/^\s*UID_MIN/{print $2}' /etc/login.defs)
[ -z "$UID_MIN" ] && UID_MIN=1000

cat >/etc/audit/rules.d/50-stig-hardening.rules <<EOF2
# Identity and account files
-w /etc/passwd -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/gshadow -p wa -k identity
-w /etc/security/opasswd -p wa -k identity

# Sudoers and sudo log
-w /etc/sudoers -p wa -k scope
-w /etc/sudoers.d/ -p wa -k scope
-w /var/log/sudo.log -p wa -k actions

# Time changes
-a always,exit -F arch=b64 -S adjtimex,settimeofday,clock_settime -k time-change
-a always,exit -F arch=b32 -S adjtimex,settimeofday,clock_settime -k time-change
-w /etc/localtime -p wa -k time-change

# Network environment
-a always,exit -F arch=b64 -S sethostname,setdomainname -k system-locale
-a always,exit -F arch=b32 -S sethostname,setdomainname -k system-locale
-w /etc/issue -p wa -k system-locale
-w /etc/issue.net -p wa -k system-locale
-w /etc/hosts -p wa -k system-locale
-w /etc/netplan/ -p wa -k system-locale

# Permission changes
-a always,exit -F arch=b64 -S chmod,fchmod,fchmodat -F auid>=${UID_MIN} -F auid!=unset -k perm_mod
-a always,exit -F arch=b32 -S chmod,fchmod,fchmodat -F auid>=${UID_MIN} -F auid!=unset -k perm_mod
-a always,exit -F arch=b64 -S chown,fchown,lchown,fchownat -F auid>=${UID_MIN} -F auid!=unset -k perm_mod
-a always,exit -F arch=b32 -S chown,fchown,lchown,fchownat -F auid>=${UID_MIN} -F auid!=unset -k perm_mod
-a always,exit -F arch=b64 -S setxattr,lsetxattr,fsetxattr,removexattr,lremovexattr,fremovexattr -F auid>=${UID_MIN} -F auid!=unset -k perm_mod
-a always,exit -F arch=b32 -S setxattr,lsetxattr,fsetxattr,removexattr,lremovexattr,fremovexattr -F auid>=${UID_MIN} -F auid!=unset -k perm_mod

# File deletion
-a always,exit -F arch=b64 -S unlink,unlinkat,rename,renameat,renameat2 -F auid>=${UID_MIN} -F auid!=unset -k delete
-a always,exit -F arch=b32 -S unlink,unlinkat,rename,renameat,renameat2 -F auid>=${UID_MIN} -F auid!=unset -k delete

# Mounts
-a always,exit -F arch=b64 -S mount -F auid>=${UID_MIN} -F auid!=unset -k mounts
-a always,exit -F arch=b32 -S mount -F auid>=${UID_MIN} -F auid!=unset -k mounts

# Login/logout and session
-w /var/log/faillog -p wa -k logins
-w /var/log/lastlog -p wa -k logins
-w /var/log/tallylog -p wa -k logins
-w /var/run/utmp -p wa -k session
-w /var/log/wtmp -p wa -k session
-w /var/log/btmp -p wa -k session

# Kernel module changes
-w /sbin/insmod -p x -k modules
-w /sbin/rmmod -p x -k modules
-w /sbin/modprobe -p x -k modules
-a always,exit -F arch=b64 -S init_module,finit_module,delete_module -k modules
-a always,exit -F arch=b32 -S init_module,delete_module -k modules
EOF2
```

Tambahkan audit untuk privileged commands secara dinamis:

```bash
find / -xdev \( -perm -4000 -o -perm -2000 \) -type f 2>/dev/null | while read -r file; do
  printf -- "-a always,exit -F path=%s -F perm=x -F auid>=%s -F auid!=unset -k privileged\n" "$file" "$UID_MIN"
done >/etc/audit/rules.d/50-privileged.rules
```

Load rules:

```bash
augenrules --load
```

Validasi:

```bash
auditctl -l | head -50
```

---

### 13.6 Immutable Audit Configuration

Set audit rules immutable setelah konfigurasi stabil:

```bash
echo '-e 2' >/etc/audit/rules.d/99-finalize.rules
augenrules --load
```

Catatan:

- Setelah `-e 2`, perubahan audit rules membutuhkan reboot.
- Terapkan bagian ini paling akhir setelah semua rules benar.

Validasi:

```bash
grep -E '^-e 2' /etc/audit/audit.rules
```

---

### 13.7 Permission File Audit

```bash
chown root:root /etc/audit/auditd.conf /etc/audit/rules.d/*.rules 2>/dev/null || true
chmod 640 /etc/audit/auditd.conf /etc/audit/rules.d/*.rules 2>/dev/null || true
chown root:root /sbin/auditctl /sbin/auditd /sbin/ausearch /sbin/aureport /sbin/autrace /sbin/augenrules 2>/dev/null || true
chmod 750 /sbin/auditctl /sbin/auditd /sbin/ausearch /sbin/aureport /sbin/autrace /sbin/augenrules 2>/dev/null || true
chown root:adm /var/log/audit 2>/dev/null || true
chmod 750 /var/log/audit 2>/dev/null || true
find /var/log/audit -type f -exec chown root:adm {} \; -exec chmod 640 {} \; 2>/dev/null || true
```

Validasi:

```bash
ls -ld /var/log/audit
ls -l /var/log/audit | head
ls -l /etc/audit /etc/audit/rules.d
```

---

## 14. AIDE: File Integrity Monitoring

### 14.1 Install AIDE

```bash
apt install -y aide aide-common
```

Validasi:

```bash
dpkg -l | grep aide
```

### 14.2 Lindungi Integritas Audit Tools

Tambahkan entri audit tools di `/etc/aide/aide.conf` bila belum ada:

```bash
cat >>/etc/aide/aide.conf <<'EOF2'

# Audit Tools - CIS/STIG
/sbin/auditctl p+i+n+u+g+s+b+acl+xattrs+sha512
/sbin/auditd p+i+n+u+g+s+b+acl+xattrs+sha512
/sbin/ausearch p+i+n+u+g+s+b+acl+xattrs+sha512
/sbin/aureport p+i+n+u+g+s+b+acl+xattrs+sha512
/sbin/autrace p+i+n+u+g+s+b+acl+xattrs+sha512
/sbin/augenrules p+i+n+u+g+s+b+acl+xattrs+sha512
EOF2
```

Validasi:

```bash
egrep '(/sbin/(audit|au))' /etc/aide/aide.conf
```

### 14.3 Inisialisasi Database AIDE

```bash
aideinit
cp -p /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

Cek manual:

```bash
aide -c /etc/aide/aide.conf --check
```

### 14.4 Pastikan AIDE Memberi Notifikasi

```bash
sed -i 's/^SILENTREPORTS=.*/SILENTREPORTS=no/' /etc/default/aide
[ -z "$(grep '^SILENTREPORTS=' /etc/default/aide)" ] && echo 'SILENTREPORTS=no' >> /etc/default/aide
```

Validasi:

```bash
grep SILENTREPORTS /etc/default/aide
systemctl list-timers | grep aide || true
grep -r aide /etc/cron* /etc/crontab 2>/dev/null || true
```

---

## 15. File Permission, Ownership, dan Filesystem Protection

### 15.1 Permission File Account Penting

```bash
chown root:root /etc/passwd /etc/passwd- /etc/group /etc/group- 2>/dev/null || true
chmod 644 /etc/passwd /etc/passwd- /etc/group /etc/group- 2>/dev/null || true

chown root:shadow /etc/shadow /etc/shadow- /etc/gshadow /etc/gshadow- 2>/dev/null || true
chmod 640 /etc/shadow /etc/shadow- /etc/gshadow /etc/gshadow- 2>/dev/null || true

chown root:root /etc/shells 2>/dev/null || true
chmod 644 /etc/shells 2>/dev/null || true
```

Validasi:

```bash
stat -c '%n %U:%G %a' /etc/passwd /etc/group /etc/shadow /etc/gshadow /etc/shells
```

---

### 15.2 World-Writable Files dan Sticky Bit

Cari file world-writable:

```bash
find / -xdev -type f -perm -0002 -print 2>/dev/null
```

Perbaiki file world-writable yang tidak seharusnya:

```bash
find / -xdev -type f -perm -0002 -exec chmod o-w {} \; 2>/dev/null
```

Pastikan direktori world-writable punya sticky bit:

```bash
find / -xdev -type d -perm -0002 ! -perm -1000 -print 2>/dev/null
find / -xdev -type d -perm -0002 ! -perm -1000 -exec chmod a+t {} \; 2>/dev/null
```

---

### 15.3 File Tanpa Owner/Group

Cari file tanpa owner/group:

```bash
find / -xdev \( -nouser -o -nogroup \) -print 2>/dev/null
```

Perbaikan harus case-by-case. Jangan otomatis chown semua file tanpa investigasi.

---

### 15.4 Review SUID/SGID

```bash
find / -xdev \( -perm -4000 -o -perm -2000 \) -type f -print 2>/dev/null | sort
```

Tindakan:

- Bandingkan dengan baseline awal.
- Hapus SUID/SGID dari binary yang tidak diperlukan.
- Jangan hapus SUID/SGID dari tool sistem penting tanpa uji.

---

## 16. User dan Group Consistency

### 16.1 Shadowed Passwords

Pastikan `/etc/passwd` memakai `x` pada field password:

```bash
awk -F: '($2 != "x") {print $1 ": password field is not x"}' /etc/passwd
```

### 16.2 Empty Password Fields

```bash
awk -F: '($2 == "") {print $1}' /etc/shadow
```

Lock akun jika ditemukan:

```bash
passwd -l <nama_user>
```

### 16.3 Duplicate UID/GID/User/Group

```bash
cut -d: -f3 /etc/passwd | sort | uniq -d
cut -d: -f3 /etc/group | sort | uniq -d
cut -d: -f1 /etc/passwd | sort | uniq -d
cut -d: -f1 /etc/group | sort | uniq -d
```

Jika ada duplikasi, investigasi sebelum mengubah.

### 16.4 Home Directory User Interaktif

```bash
awk -F: '($3>=1000)&&($1!="nobody"){print $1,$6}' /etc/passwd | while read -r user home; do
  [ -d "$home" ] || echo "Missing home: $user $home"
done
```

Permission home:

```bash
awk -F: '($3>=1000)&&($1!="nobody"){print $1,$6}' /etc/passwd | while read -r user home; do
  [ -d "$home" ] && chmod go-w "$home"
done
```

### 16.5 Dot Files User

Cari dotfile berbahaya:

```bash
find /home -name '.rhosts' -o -name '.netrc' -o -name '.forward' 2>/dev/null
```

Review dan hapus bila tidak diperlukan.

---

## 17. GUI/GNOME Hardening Jika Workstation Memakai GNOME

Jika sistem server tanpa GUI, bagian ini bisa ditandai **Not Applicable**.

### 17.1 GDM Login Banner

```bash
mkdir -p /etc/dconf/db/gdm.d
cat >/etc/dconf/db/gdm.d/01-banner-message <<'EOF2'
[org/gnome/login-screen]
banner-message-enable=true
banner-message-text='Authorized users only. All activity may be monitored and reported.'
EOF2

dconf update
```

### 17.2 Disable User List

```bash
cat >/etc/dconf/db/gdm.d/00-login-screen <<'EOF2'
[org/gnome/login-screen]
disable-user-list=true
EOF2

dconf update
```

### 17.3 Screen Lock dan Automount

Untuk user profile dconf, terapkan sesuai kebijakan organisasi. Contoh umum:

```bash
mkdir -p /etc/dconf/db/local.d /etc/dconf/db/local.d/locks
cat >/etc/dconf/db/local.d/00-security-settings <<'EOF2'
[org/gnome/desktop/session]
idle-delay=uint32 900

[org/gnome/desktop/screensaver]
lock-enabled=true
lock-delay=uint32 0

[org/gnome/desktop/media-handling]
automount=false
autorun-never=true
EOF2

dconf update
```

---

## 18. Validasi Akhir Sistem

### 18.1 Validasi Paket dan Service

```bash
systemctl is-active ssh ufw apparmor chrony rsyslog auditd
systemctl is-enabled ssh ufw apparmor chrony rsyslog auditd
```

### 18.2 Validasi Port Terbuka

```bash
ss -tulpen
ufw status verbose
```

Pastikan hanya port yang disetujui yang listening.

### 18.3 Validasi SSH

```bash
sshd -t
sshd -T | egrep 'permitrootlogin|permitemptypasswords|clientaliveinterval|clientalivecountmax|x11forwarding|ciphers|macs|kexalgorithms|banner'
```

### 18.4 Validasi Audit

```bash
auditctl -s
auditctl -l | wc -l
grep -E '^-e 2' /etc/audit/audit.rules || true
```

### 18.5 Validasi AIDE

```bash
aide -c /etc/aide/aide.conf --check
```

### 18.6 Validasi AppArmor

```bash
aa-status
```

### 18.7 Validasi Password Policy

```bash
grep -v '^#' /etc/security/pwquality.conf | sed '/^$/d'
grep -E '^PASS_MAX_DAYS|^PASS_MIN_DAYS|^PASS_WARN_AGE' /etc/login.defs
```

### 18.8 Validasi Permission File Penting

```bash
stat -c '%n %U:%G %a' /etc/passwd /etc/group /etc/shadow /etc/gshadow /etc/shells
find /var/log -type f -perm /137 -ls | head
```

---

## 19. Template Exception Register

Gunakan tabel ini untuk aturan yang tidak diterapkan.

| Rule / Area | Status | Alasan | Risiko | Mitigasi | Disetujui Oleh | Tanggal Review |
|---|---|---|---|---|---|---|
| FIPS Mode | Exception | Tidak ada kebutuhan compliance FIPS/DoD pada lab | Crypto tidak diklaim FIPS validated | Pakai crypto kuat OpenSSH; update rutin |  |  |
| Smart Card/PIV/PKI | Exception | Tidak ada infrastruktur smart card/PKI | Tidak memakai MFA berbasis kartu | Gunakan password kuat + SSH key + sudo policy |  |  |
| Remote audit offloading | Planned | Belum ada server audit pusat | Audit log lokal bisa hilang bila host compromise | Backup log, rencana syslog/audit server |  |  |
| GDM banner | N/A | Server tanpa GUI | Tidak ada login GUI | SSH/local banner tetap aktif |  |  |
| Wireless/Bluetooth | N/A/Disabled | Server tidak memakai wireless/Bluetooth | Attack surface radio | Service disabled/purged |  |  |

---

## 20. Urutan Implementasi yang Disarankan

Gunakan urutan berikut agar risiko terkunci lebih kecil:

1. Install Ubuntu 24.04 LTS minimal.
2. Update sistem.
3. Backup konfigurasi.
4. Konfigurasi repository, package cleanup, dan service minimization.
5. Konfigurasi Chrony.
6. Konfigurasi AppArmor.
7. Konfigurasi UFW, tetapi pastikan rule SSH benar sebelum enable.
8. Konfigurasi SSH, lalu validasi `sshd -t` sebelum restart.
9. Konfigurasi PAM/password policy dari console atau sesi admin cadangan.
10. Konfigurasi rsyslog.
11. Konfigurasi auditd dan audit rules.
12. Konfigurasi AIDE.
13. Hardening permission file dan user/group consistency.
14. Reboot dan validasi ulang.
15. Baru aktifkan audit immutable `-e 2` setelah konfigurasi stabil.

---

## 21. Checklist Ringkas Pasca-Reboot

```bash
# OS version
cat /etc/os-release

# Services
systemctl is-active ssh ufw apparmor chrony rsyslog auditd

# Firewall
ufw status verbose
ss -tulpen

# SSH
sshd -t
sshd -T | egrep 'permitrootlogin|clientaliveinterval|clientalivecountmax|ciphers|macs|kexalgorithms|x11forwarding|banner'

# AppArmor
aa-status

# Time
chronyc tracking

# Audit
auditctl -s
auditctl -l | head

# AIDE
aide -c /etc/aide/aide.conf --check

# Permissions
stat -c '%n %U:%G %a' /etc/passwd /etc/group /etc/shadow /etc/gshadow
```

---

## 22. Penutup

Panduan ini menempatkan CIS Ubuntu Linux 24.04 LTS STIG Benchmark sebagai **baseline implementasi**. Untuk lab dan pembelajaran, fokus utama adalah memahami kenapa konfigurasi diterapkan dan bisa membuktikan hasilnya melalui audit command. Untuk lingkungan produksi, setiap perubahan harus melewati lab testing, pilot deployment, dokumentasi exception, serta review berkala.

Bagian yang paling berisiko mengganggu operasional adalah:

- PAM dan account lockout.
- SSH ciphers/MACs/KexAlgorithms bila client lama masih dipakai.
- UFW bila rule akses admin belum benar.
- AppArmor enforcing pada aplikasi custom.
- Audit immutable `-e 2` karena perubahan audit rule butuh reboot.
- FIPS mode karena harus dirancang sejak instalasi dan terkait dukungan vendor.

