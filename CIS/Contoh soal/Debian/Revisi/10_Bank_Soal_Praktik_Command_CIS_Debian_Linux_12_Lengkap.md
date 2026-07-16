# Bank Soal Praktik Command CIS Debian Linux 12 — Pilihan Ganda Jamak

**Fokus:** latihan praktik command, audit, remediation, konfigurasi file, dan pilihan jawaban jamak sesuai materi CIS Debian Linux 12 serta file implementasi Arch berbasis prinsip CIS Debian.

> **Aturan latihan:** pilih semua jawaban yang benar. Jika tidak ada opsi benar, jawab **PASS / kosongkan jawaban**. Jangan jalankan command destruktif di sistem asli; gunakan VM/lab.

## Daftar Cakupan

- Bab 1 Initial Setup: filesystem modules, mount options, package management, AppArmor, GRUB, sysctl, banner, GDM
- Bab 2 Services: service minimization, client services, time sync, cron/at
- Bab 3 Network: perangkat jaringan, kernel modules, IPv4/IPv6 sysctl
- Bab 4 Firewall: UFW resmi CIS Debian dan bonus adaptasi firewalld
- Bab 5 Access Control: SSH, sudo, su, PAM, password policy, user environment
- Bab 6 Logging and Auditing: journald, rsyslog, auditd, audit rules, AIDE
- Bab 7 System Maintenance: permission file, world writable, UID/GID, home, dotfiles
- Bonus implementasi Arch/LUKS/LVM berbasis prinsip CIS Debian


## 0. Cara Menjawab Soal Praktik

### Soal 1

**Pertanyaan:** Dalam ujian pilihan ganda jamak, instruksi mengatakan jawaban bisa lebih dari satu. Bagaimana cara menjawab soal yang semua opsinya salah?

A. `Pilih opsi yang paling mendekati benar`
B. `Pilih semua opsi agar tidak kosong`
C. `PASS / kosongkan jawaban`
D. `Pilih opsi A saja sebagai default`

**Jawaban benar:** C

**Pembahasan:** Jika tidak ada command yang benar, jawab PASS/kosong.

---
### Soal 2

**Pertanyaan:** Pada latihan ini, apa prinsip utama saat memilih command CIS?

A. `Pilih command yang terlihat paling panjang`
B. `Pilih command yang sesuai target OS, service, path, dan tujuan rekomendasi`
C. `Pilih command berbasis distro lain walaupun path berbeda`
D. `Pilih command yang tidak destruktif walaupun tidak mencapai konfigurasi`

**Jawaban benar:** B

**Pembahasan:** CIS sangat bergantung pada target teknologi dan versi.

---

## 1.1.1 Filesystem Kernel Modules

### Soal 3

**Pertanyaan:** Perintah manakah yang benar untuk melakukan audit/remediation agar kernel module `cramfs` tidak tersedia sesuai pola CIS Debian?

A. `lsmod | grep 'cramfs'`
B. `modprobe -r cramfs 2>/dev/null`
C. `printf '%s\n' '' 'install cramfs /bin/false' >> /etc/modprobe.d/60-cramfs.conf`
D. `echo 'enable cramfs' >> /etc/modules-load.d/cramfs.conf`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit memeriksa module loaded; remediation meng-unload dan menambahkan install /bin/false. Opsi D justru memuat module.

---
### Soal 4

**Pertanyaan:** Perintah manakah yang benar untuk melakukan audit/remediation agar kernel module `freevxfs` tidak tersedia sesuai pola CIS Debian?

A. `lsmod | grep 'freevxfs'`
B. `modprobe -r freevxfs 2>/dev/null`
C. `printf '%s\n' '' 'install freevxfs /bin/false' >> /etc/modprobe.d/60-freevxfs.conf`
D. `echo 'enable freevxfs' >> /etc/modules-load.d/freevxfs.conf`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit memeriksa module loaded; remediation meng-unload dan menambahkan install /bin/false. Opsi D justru memuat module.

---
### Soal 5

**Pertanyaan:** Perintah manakah yang benar untuk melakukan audit/remediation agar kernel module `hfs` tidak tersedia sesuai pola CIS Debian?

A. `lsmod | grep 'hfs'`
B. `modprobe -r hfs 2>/dev/null`
C. `printf '%s\n' '' 'install hfs /bin/false' >> /etc/modprobe.d/60-hfs.conf`
D. `echo 'enable hfs' >> /etc/modules-load.d/hfs.conf`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit memeriksa module loaded; remediation meng-unload dan menambahkan install /bin/false. Opsi D justru memuat module.

---
### Soal 6

**Pertanyaan:** Perintah manakah yang benar untuk melakukan audit/remediation agar kernel module `hfsplus` tidak tersedia sesuai pola CIS Debian?

A. `lsmod | grep 'hfsplus'`
B. `modprobe -r hfsplus 2>/dev/null`
C. `printf '%s\n' '' 'install hfsplus /bin/false' >> /etc/modprobe.d/60-hfsplus.conf`
D. `echo 'enable hfsplus' >> /etc/modules-load.d/hfsplus.conf`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit memeriksa module loaded; remediation meng-unload dan menambahkan install /bin/false. Opsi D justru memuat module.

---
### Soal 7

**Pertanyaan:** Perintah manakah yang benar untuk melakukan audit/remediation agar kernel module `jffs2` tidak tersedia sesuai pola CIS Debian?

A. `lsmod | grep 'jffs2'`
B. `modprobe -r jffs2 2>/dev/null`
C. `printf '%s\n' '' 'install jffs2 /bin/false' >> /etc/modprobe.d/60-jffs2.conf`
D. `echo 'enable jffs2' >> /etc/modules-load.d/jffs2.conf`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit memeriksa module loaded; remediation meng-unload dan menambahkan install /bin/false. Opsi D justru memuat module.

---
### Soal 8

**Pertanyaan:** Perintah manakah yang benar untuk melakukan audit/remediation agar kernel module `squashfs` tidak tersedia sesuai pola CIS Debian?

A. `lsmod | grep 'squashfs'`
B. `modprobe -r squashfs 2>/dev/null`
C. `printf '%s\n' '' 'install squashfs /bin/false' >> /etc/modprobe.d/60-squashfs.conf`
D. `echo 'enable squashfs' >> /etc/modules-load.d/squashfs.conf`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit memeriksa module loaded; remediation meng-unload dan menambahkan install /bin/false. Opsi D justru memuat module.

---
### Soal 9

**Pertanyaan:** Perintah manakah yang benar untuk melakukan audit/remediation agar kernel module `udf` tidak tersedia sesuai pola CIS Debian?

A. `lsmod | grep 'udf'`
B. `modprobe -r udf 2>/dev/null`
C. `printf '%s\n' '' 'install udf /bin/false' >> /etc/modprobe.d/60-udf.conf`
D. `echo 'enable udf' >> /etc/modules-load.d/udf.conf`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit memeriksa module loaded; remediation meng-unload dan menambahkan install /bin/false. Opsi D justru memuat module.

---
### Soal 10

**Pertanyaan:** Perintah manakah yang benar untuk melakukan audit/remediation agar kernel module `overlay` tidak tersedia sesuai pola CIS Debian?

A. `lsmod | grep 'overlay'`
B. `modprobe -r overlay 2>/dev/null`
C. `printf '%s\n' '' 'install overlay /bin/false' >> /etc/modprobe.d/60-overlay.conf`
D. `echo 'enable overlay' >> /etc/modules-load.d/overlay.conf`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit memeriksa module loaded; remediation meng-unload dan menambahkan install /bin/false. Opsi D justru memuat module.

---
### Soal 11

**Pertanyaan:** Perintah manakah yang benar untuk memverifikasi `cramfs` tidak loadable melalui konfigurasi modprobe?

A. `modprobe --showconfig | grep -P -- '\b(install|blacklist)\h+cramfs\b'`
B. `lsmod | grep '^cramfs$'`
C. `apt list --installed cramfs`
D. `systemctl status cramfs`

**Jawaban benar:** A

**Pembahasan:** CIS memakai `modprobe --showconfig` untuk melihat blacklist/install rule.

---
### Soal 12

**Pertanyaan:** Manakah konfigurasi yang benar untuk menonaktifkan `usb-storage` jika server tidak membutuhkan media USB storage?

A. `printf '%s\n' 'install usb-storage /bin/false' 'blacklist usb-storage' > /etc/modprobe.d/60-usb-storage.conf`
B. `modprobe -r usb-storage 2>/dev/null`
C. `echo usb-storage >> /etc/modules`
D. `systemctl enable usb-storage.service`

**Jawaban benar:** A, B

**Pembahasan:** A membuat module tidak loadable via modprobe, B meng-unload jika sedang aktif.

---
### Soal 13

**Pertanyaan:** Server dipakai sebagai Docker/container host. Perintah manakah yang sebaiknya dipilih untuk memenuhi CIS Level 2 `overlay` tanpa mengganggu container?

A. `printf '%s\n' 'install overlay /bin/false' >> /etc/modprobe.d/60-overlay.conf`
B. `printf '%s\n' 'blacklist overlay' >> /etc/modprobe.d/60-overlay.conf`
C. `Dokumentasikan exception karena container membutuhkan overlay`
D. `rmmod overlay 2>/dev/null lalu jalankan Docker`

**Jawaban benar:** C

**Pembahasan:** Overlay dibahas Level 2 dan dapat mengganggu Docker/Podman/Kubernetes. Praktik aman adalah exception terdokumentasi.

---

## 1.1.2 Filesystem Partitions

### Soal 14

**Pertanyaan:** Baris `/etc/fstab` manakah yang benar untuk `/tmp` agar sesuai prinsip CIS?

A. `tmpfs /tmp tmpfs rw,nosuid,nodev,noexec,relatime,size=2G,mode=1777 0 0`
B. `UUID=<uuid> /tmp ext4 rw,nodev,nosuid,noexec,relatime 0 2`
C. `UUID=<uuid> /tmp ext4 rw,dev,suid,exec 0 2`
D. `tmpfs /tmp tmpfs defaults,mode=0777 0 0`

**Jawaban benar:** A, B

**Pembahasan:** CIS menerima `/tmp` sebagai tmpfs atau partisi terpisah dengan nodev,nosuid,noexec.

---
### Soal 15

**Pertanyaan:** Baris `/etc/fstab` manakah yang benar untuk `/var/log/audit` sesuai prinsip CIS?

A. `UUID=<uuid-audit> /var/log/audit ext4 rw,nodev,nosuid,noexec,relatime 0 2`
B. `UUID=<uuid-audit> /var/log/audit ext4 rw,dev,suid,exec 0 2`
C. `tmpfs /var/log/audit tmpfs rw,nodev,nosuid,noexec 0 0`
D. `UUID=<uuid-audit> /var/log/audit ext4 defaults,nodev,nosuid,noexec 0 2`

**Jawaban benar:** A, D

**Pembahasan:** Audit log sebaiknya persisten dan dipisah; opsi nodev,nosuid,noexec benar. Tmpfs tidak cocok untuk audit log persisten.

---
### Soal 16

**Pertanyaan:** Perintah manakah yang benar untuk memeriksa mount option `/var/tmp`?

A. `findmnt -no TARGET,OPTIONS /var/tmp`
B. `mount | grep ' /var/tmp '`
C. `cat /etc/fstab | grep '/var/tmp'`
D. `chmod noexec /var/tmp`

**Jawaban benar:** A, B, C

**Pembahasan:** A/B melihat kondisi aktif; C melihat konfigurasi persisten; D bukan command valid untuk mount option.

---
### Soal 17

**Pertanyaan:** Manakah perintah yang benar untuk membuat LUKS2 container pada skema implementasi hardening berbasis prinsip CIS?

A. `cryptsetup luksFormat --type luks2 /dev/nvme0n1p3`
B. `cryptsetup open /dev/nvme0n1p3 cryptlvm`
C. `mkfs.ext4 /dev/nvme0n1p3 sebelum luksFormat`
D. `luksctl create /dev/nvme0n1p3 cryptlvm`

**Jawaban benar:** A, B

**Pembahasan:** A membuat container LUKS2, B membukanya. C salah urutan untuk enkripsi; D bukan command standar cryptsetup.

---
### Soal 18

**Pertanyaan:** Manakah command LVM yang benar setelah LUKS dibuka sebagai `/dev/mapper/cryptlvm`?

A. `pvcreate /dev/mapper/cryptlvm`
B. `vgcreate vgarch /dev/mapper/cryptlvm`
C. `lvcreate -L 40G -n lv_root vgarch`
D. `vgcreate /dev/mapper/cryptlvm vgarch`

**Jawaban benar:** A, B, C

**Pembahasan:** Urutan benar: PV, VG, LV. Opsi D urutan argumennya salah.

---

## 1.2 Package Management

### Soal 19

**Pertanyaan:** Perintah manakah yang benar untuk memasang update, patch, dan software keamanan pada Debian?

A. `apt update`
B. `apt upgrade`
C. `apt install <package>`
D. `pacman -Syu`

**Jawaban benar:** A, B, C

**Pembahasan:** Pada Debian gunakan APT. Pacman adalah Arch.

---
### Soal 20

**Pertanyaan:** Manakah audit/perintah yang tepat untuk memastikan file repository APT memakai konsep signed repository?

A. `grep -R 'Signed-By=' /etc/apt/sources.list /etc/apt/sources.list.d/ 2>/dev/null`
B. `grep -R 'signed-by=' /etc/apt/sources.list /etc/apt/sources.list.d/ 2>/dev/null`
C. `pacman-key --populate`
D. `apt-key adv --recv-keys all`

**Jawaban benar:** A, B

**Pembahasan:** Signed-By/signed-by pada sources APT membatasi key yang digunakan. Pacman-key bukan Debian; apt-key lama tidak sesuai hardening modern.

---
### Soal 21

**Pertanyaan:** Permission mana yang masuk akal untuk file konfigurasi repository APT agar tidak writable oleh user biasa?

A. `chown root:root /etc/apt/sources.list`
B. `chmod 0644 /etc/apt/sources.list`
C. `chmod 0777 /etc/apt/sources.list`
D. `chown nobody:nogroup /etc/apt/sources.list`

**Jawaban benar:** A, B

**Pembahasan:** Konfigurasi repository harus dimiliki root dan tidak writable oleh user biasa.

---

## 1.3 AppArmor

### Soal 22

**Pertanyaan:** Perintah manakah yang benar untuk memasang dan mengaktifkan AppArmor pada Debian?

A. `apt install apparmor apparmor-utils`
B. `systemctl enable --now apparmor.service`
C. `aa-status`
D. `systemctl enable --now selinux.service`

**Jawaban benar:** A, B, C

**Pembahasan:** Debian CIS memakai AppArmor, bukan SELinux service.

---
### Soal 23

**Pertanyaan:** Perintah manakah yang benar untuk mengubah semua profile AppArmor ke enforcing jika profile sudah siap?

A. `aa-enforce /etc/apparmor.d/*`
B. `aa-complain /etc/apparmor.d/*`
C. `apparmor_parser -R /etc/apparmor.d/*`
D. `systemctl stop apparmor`

**Jawaban benar:** A

**Pembahasan:** `aa-enforce` mengubah profile ke enforce. Opsi lain mengurangi/menonaktifkan kontrol.

---
### Soal 24

**Pertanyaan:** Perintah mana yang benar untuk audit bahwa AppArmor tidak dinonaktifkan dari GRUB?

A. `grep '^\s*linux' /boot/grub/grub.cfg | grep 'apparmor=0'`
B. `aa-status`
C. `grep 'selinux=0' /boot/grub/grub.cfg`
D. `ufw status verbose`

**Jawaban benar:** A, B

**Pembahasan:** CIS mengecek tidak ada apparmor=0; aa-status juga berguna untuk status AppArmor.

---

## 1.4 Bootloader

### Soal 25

**Pertanyaan:** Perintah manakah yang benar untuk membuat hash password GRUB pada Debian?

A. `grub-mkpasswd-pbkdf2`
B. `update-grub`
C. `passwd grub`
D. `mkpasswd --method=sha-512 root`

**Jawaban benar:** A

**Pembahasan:** Hash GRUB dibuat dengan `grub-mkpasswd-pbkdf2`; `update-grub` dipakai setelah konfigurasi.

---
### Soal 26

**Pertanyaan:** Perintah mana yang benar untuk mengamankan file konfigurasi bootloader GRUB?

A. `chown root:root /boot/grub/grub.cfg`
B. `chmod 600 /boot/grub/grub.cfg`
C. `chmod 777 /boot/grub/grub.cfg`
D. `chown nobody:nogroup /boot/grub/grub.cfg`

**Jawaban benar:** A, B

**Pembahasan:** GRUB config harus dimiliki root dan tidak readable/writable oleh user biasa.

---

## 1.5 Process Hardening

### Soal 27

**Pertanyaan:** Konfigurasi sysctl mana yang benar untuk perlindungan hardlink/symlink sesuai prinsip CIS?

A. `fs.protected_hardlinks = 1`
B. `fs.protected_symlinks = 1`
C. `fs.protected_hardlinks = 0`
D. `fs.protected_symlinks = 0`

**Jawaban benar:** A, B

**Pembahasan:** Nilai 1 mengaktifkan proteksi.

---
### Soal 28

**Pertanyaan:** Konfigurasi mana yang benar untuk ASLR dan core dump hardening?

A. `kernel.randomize_va_space = 2`
B. `fs.suid_dumpable = 0`
C. `* hard core 0`
D. `kernel.randomize_va_space = 0`

**Jawaban benar:** A, B, C

**Pembahasan:** ASLR penuh memakai 2, SUID core dump dimatikan, limits core 0.

---
### Soal 29

**Pertanyaan:** Perintah manakah yang benar untuk menerapkan konfigurasi sysctl dari `/etc/sysctl.d/*.conf`?

A. `sysctl --system`
B. `sysctl -p /etc/sysctl.d/60-cis-hardening.conf`
C. `systemctl restart procps`
D. `chmod +x /etc/sysctl.conf`

**Jawaban benar:** A, B

**Pembahasan:** `sysctl --system` memuat semua konfigurasi; `sysctl -p file` memuat file tertentu.

---
### Soal 30

**Pertanyaan:** Konfigurasi `systemd-coredump` mana yang benar untuk menonaktifkan penyimpanan core dump?

A. `Storage=none`
B. `ProcessSizeMax=0`
C. `Storage=external`
D. `ProcessSizeMax=infinity`

**Jawaban benar:** A, B

**Pembahasan:** CIS mengarah pada menonaktifkan storage dan membatasi ProcessSizeMax.

---

## 1.6 Warning Banners

### Soal 31

**Pertanyaan:** Perintah mana yang benar untuk menyiapkan banner lokal yang tidak membocorkan versi OS?

A. `printf '%s\n' 'Authorized uses only. Activity may be monitored.' > /etc/issue`
B. `printf '%s\n' 'Authorized uses only. Activity may be monitored.' > /etc/issue.net`
C. `printf '%s\n' 'Debian 12.13 kernel 6.x internal host db01' > /etc/issue`
D. `chmod 644 /etc/issue /etc/issue.net`

**Jawaban benar:** A, B, D

**Pembahasan:** Banner aman tidak membocorkan detail versi/hostname internal.

---
### Soal 32

**Pertanyaan:** Directive SSH mana yang benar agar sshd menampilkan banner peringatan?

A. `Banner /etc/issue.net`
B. `Banner none`
C. `PrintMotd yes`
D. `Banner /etc/shadow`

**Jawaban benar:** A

**Pembahasan:** `Banner /etc/issue.net` adalah directive sshd yang tepat untuk login warning.

---

## 1.7 GDM

### Soal 33

**Pertanyaan:** Manakah konfigurasi yang benar untuk menonaktifkan daftar user pada GDM?

A.
```text
[org/gnome/login-screen]
disable-user-list=true
```
B.
```text
[org/gnome/login-screen]
disable-user-list=false
```
C. `gsettings set org.gnome.login-screen disable-user-list true`
D. `ufw deny gdm`

**Jawaban benar:** A, C

**Pembahasan:** Disable user list mencegah enumerasi akun di layar login.

---
### Soal 34

**Pertanyaan:** Untuk sistem server headless tanpa GUI, command mana yang benar sesuai prinsip minimisasi?

A. `apt purge gdm3`
B. `systemctl disable --now gdm.service`
C. `apt install xserver-xorg gdm3`
D. `systemctl enable --now gdm.service`

**Jawaban benar:** A, B

**Pembahasan:** Jika tidak perlu GUI, hapus/nonaktifkan GDM/X stack.

---

## 2.1 Server Services

### Soal 35

**Pertanyaan:** Perintah manakah yang benar untuk memeriksa service yang sedang listening pada interface jaringan?

A. `ss -tulpen`
B. `ss -lntu`
C. `systemctl list-units --type=service --state=running`
D. `chmod -R 777 /var/log`

**Jawaban benar:** A, B, C

**Pembahasan:** ss melihat port listening; systemctl melihat service aktif.

---
### Soal 36

**Pertanyaan:** Server bukan web server. Perintah mana yang benar untuk memastikan Apache/Nginx tidak digunakan?

A. `systemctl disable --now apache2.service 2>/dev/null`
B. `systemctl disable --now nginx.service 2>/dev/null`
C. `apt purge apache2 nginx`
D. `systemctl enable --now apache2.service`

**Jawaban benar:** A, B, C

**Pembahasan:** Nonaktifkan atau hapus web server jika bukan fungsi sistem.

---
### Soal 37

**Pertanyaan:** Server bukan DHCP server. Manakah tindakan yang benar terhadap layanan DHCP server Kea/ISC?

A. `systemctl disable --now kea-dhcp4-server.service 2>/dev/null`
B. `systemctl disable --now kea-dhcp6-server.service 2>/dev/null`
C. `apt purge isc-dhcp-server kea-dhcp4-server kea-dhcp6-server`
D. `apt install kea-dhcp4-server`

**Jawaban benar:** A, B, C

**Pembahasan:** Hapus/nonaktifkan DHCP server jika tidak diperlukan.

---
### Soal 38

**Pertanyaan:** Perintah mana yang benar untuk menonaktifkan Avahi pada server yang tidak membutuhkan service discovery?

A. `systemctl disable --now avahi-daemon.service`
B. `apt purge avahi-daemon`
C. `systemctl enable --now avahi-daemon.service`
D. `avahi enable secure`

**Jawaban benar:** A, B

**Pembahasan:** Avahi menambah discovery surface; nonaktifkan/hapus jika tidak dibutuhkan.

---
### Soal 39

**Pertanyaan:** Server bukan file server Windows/SMB. Perintah mana yang tepat?

A. `systemctl disable --now smbd.service nmbd.service 2>/dev/null`
B. `apt purge samba`
C. `systemctl enable --now smbd.service`
D. `ufw allow samba`

**Jawaban benar:** A, B

**Pembahasan:** Samba hanya diperlukan pada SMB file server resmi.

---
### Soal 40

**Pertanyaan:** Server bukan NFS server. Command mana yang tepat?

A. `systemctl disable --now nfs-server.service 2>/dev/null`
B. `apt purge nfs-kernel-server`
C. `systemctl enable --now nfs-server.service`
D. `exportfs -ra`

**Jawaban benar:** A, B

**Pembahasan:** NFS server membuka akses filesystem melalui jaringan.

---
### Soal 41

**Pertanyaan:** Manakah command yang benar untuk memastikan Telnet server tidak digunakan?

A. `apt purge telnetd`
B. `systemctl disable --now inetd.service 2>/dev/null`
C. `systemctl enable telnetd`
D. `ufw allow 23/tcp`

**Jawaban benar:** A, B

**Pembahasan:** Telnet server tidak terenkripsi. Kadang dikelola inetd/openbsd-inetd.

---
### Soal 42

**Pertanyaan:** Manakah command yang benar untuk menonaktifkan FTP server jika tidak ada kebutuhan resmi?

A. `apt purge vsftpd proftpd-basic`
B. `systemctl disable --now vsftpd.service 2>/dev/null`
C. `ufw allow 21/tcp`
D. `systemctl enable --now vsftpd.service`

**Jawaban benar:** A, B

**Pembahasan:** FTP server tidak aman secara default dan tidak diperlukan pada baseline umum.

---
### Soal 43

**Pertanyaan:** Command mana yang benar untuk menonaktifkan print server pada server headless?

A. `apt purge cups`
B. `systemctl disable --now cups.service`
C. `systemctl enable --now cups.service`
D. `ufw allow 631/tcp`

**Jawaban benar:** A, B

**Pembahasan:** CUPS hanya diperlukan pada print server/workstation yang membutuhkan printing.

---
### Soal 44

**Pertanyaan:** Command mana yang benar untuk menonaktifkan SNMP service jika tidak digunakan?

A. `systemctl disable --now snmpd.service`
B. `apt purge snmpd`
C. `systemctl enable --now snmpd.service`
D. `ufw allow 161/udp`

**Jawaban benar:** A, B

**Pembahasan:** SNMP bisa membocorkan informasi sistem jika tidak dikonfigurasi aman.

---
### Soal 45

**Pertanyaan:** Command mana yang benar untuk menonaktifkan rpcbind jika tidak ada kebutuhan RPC/NFS?

A. `systemctl disable --now rpcbind.service rpcbind.socket`
B. `apt purge rpcbind`
C. `systemctl enable --now rpcbind.socket`
D. `ufw allow 111/tcp`

**Jawaban benar:** A, B

**Pembahasan:** rpcbind sering diperlukan untuk layanan RPC seperti NFS, tetapi tidak perlu pada server umum.

---

## 2.2 Client Services

### Soal 46

**Pertanyaan:** Manakah command Debian yang benar untuk menghapus client lama yang tidak aman jika tidak dibutuhkan?

A. `apt purge telnet`
B. `apt purge ftp`
C. `apt purge rsh-client`
D. `apt purge talk`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Client protokol lama dapat mendorong penggunaan komunikasi tidak terenkripsi.

---
### Soal 47

**Pertanyaan:** Manakah pilihan yang benar untuk menghapus NIS/LDAP client jika tidak digunakan?

A. `apt purge nis`
B. `apt purge ldap-utils`
C. `apt purge libnss-ldap libpam-ldap`
D. `systemctl enable ypbind`

**Jawaban benar:** A, B, C

**Pembahasan:** Opsi D mengaktifkan layanan NIS/ypbind.

---

## 2.3 Time Synchronization

### Soal 48

**Pertanyaan:** Manakah command yang benar untuk menggunakan chrony sebagai satu-satunya daemon sinkronisasi waktu?

A. `apt install chrony`
B. `systemctl enable --now chrony.service`
C. `systemctl disable --now systemd-timesyncd.service`
D. `systemctl enable --now systemd-timesyncd.service chrony.service`

**Jawaban benar:** A, B, C

**Pembahasan:** CIS menghendaki satu daemon time sync. Jangan jalankan dua bersamaan.

---
### Soal 49

**Pertanyaan:** Command mana yang benar untuk audit chrony aktif dan berjalan?

A. `systemctl is-enabled chrony.service`
B. `systemctl is-active chrony.service`
C. `chronyc sources`
D. `timedatectl set-ntp false`

**Jawaban benar:** A, B, C

**Pembahasan:** A/B cek systemd, C cek sumber chrony.

---
### Soal 50

**Pertanyaan:** Konfigurasi mana yang benar di `/etc/chrony/chrony.conf` untuk timeserver resmi organisasi?

A. `server time.example.org iburst`
B. `pool pool.ntp.org iburst`
C. `server 127.0.0.1 offline`
D. `allow 0.0.0.0/0`

**Jawaban benar:** A, B

**Pembahasan:** Server/pool yang sah boleh dipakai. `allow 0.0.0.0/0` menjadikan host melayani client luas, bukan baseline umum.

---

## 2.4 Job Schedulers

### Soal 51

**Pertanyaan:** Command mana yang benar untuk mengaktifkan cron daemon pada Debian?

A. `apt install cron`
B. `systemctl enable --now cron.service`
C. `systemctl is-active cron.service`
D. `systemctl enable --now crond.service`

**Jawaban benar:** A, B, C

**Pembahasan:** Pada Debian nama service umumnya `cron.service`, bukan `crond.service`.

---
### Soal 52

**Pertanyaan:** Permission mana yang benar untuk `/etc/crontab` dan direktori cron agar tidak dapat diubah user biasa?

A. `chown root:root /etc/crontab`
B. `chmod og-rwx /etc/crontab`
C. `chown root:root /etc/cron.d`
D. `chmod -R 777 /etc/cron.d`

**Jawaban benar:** A, B, C

**Pembahasan:** Cron config harus dikuasai root dan tidak world writable.

---
### Soal 53

**Pertanyaan:** Command mana yang benar untuk membatasi akses `at` hanya melalui allow/deny file yang dikendalikan root?

A. `touch /etc/at.allow`
B. `chown root:root /etc/at.allow`
C. `chmod 640 /etc/at.allow`
D. `chmod 777 /etc/at.allow`

**Jawaban benar:** A, B, C

**Pembahasan:** File allow/deny scheduler harus tidak writable oleh user biasa.

---

## 2 Services

### Soal 54

**Pertanyaan:** Server produksi bukan DNS, DHCP, FTP, mail, print, NFS, Samba, proxy, atau SNMP server. Manakah command tunggal berikut yang sepenuhnya benar dan aman untuk semua konteks?

A. `apt purge apache2 nginx bind9 isc-dhcp-server vsftpd samba snmpd cups -y`
B. `systemctl disable --now '*'`
C. `ufw allow all`
D. `rm -rf /etc/systemd/system`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: opsi A bisa benar pada sebagian konteks tapi tidak aman sebagai jawaban universal tanpa inventaris/exception; B-D jelas salah/berbahaya.

---

## 3.1 Network Devices

### Soal 55

**Pertanyaan:** Command mana yang benar untuk mengidentifikasi interface jaringan dan status wireless?

A. `ip link`
B. `iw dev`
C. `rfkill list`
D. `chmod 000 /sys/class/net`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit perangkat jaringan dimulai dari visibility.

---
### Soal 56

**Pertanyaan:** Server tidak membutuhkan Wi-Fi. Command mana yang benar untuk memblokir wireless?

A. `rfkill block wifi`
B. `systemctl disable --now wpa_supplicant.service 2>/dev/null`
C. `nmcli radio wifi off`
D. `ufw allow wifi`

**Jawaban benar:** A, B, C

**Pembahasan:** Tergantung stack jaringan; rfkill/nmcli/service bisa dipakai. UFW tidak 'allow wifi'.

---
### Soal 57

**Pertanyaan:** Server tidak membutuhkan Bluetooth. Command mana yang benar?

A. `systemctl disable --now bluetooth.service`
B. `rfkill block bluetooth`
C. `apt purge bluez`
D. `systemctl enable --now bluetooth.service`

**Jawaban benar:** A, B, C

**Pembahasan:** Bluetooth menambah local wireless attack surface jika tidak dibutuhkan.

---

## 3.2 Network Kernel Modules

### Soal 58

**Pertanyaan:** Command mana yang benar untuk menonaktifkan module `dccp` jika tidak digunakan?

A. `modprobe -r dccp 2>/dev/null`
B. `printf '%s\n' 'install dccp /bin/false' 'blacklist dccp' > /etc/modprobe.d/60-dccp.conf`
C. `modprobe --showconfig | grep -P -- '\b(install|blacklist)\h+dccp\b'`
D. `echo dccp >> /etc/modules`

**Jawaban benar:** A, B, C

**Pembahasan:** A unload, B disable, C audit. D justru membuat module dimuat.

---
### Soal 59

**Pertanyaan:** Module jaringan mana yang wajar masuk daftar disable jika tidak digunakan sesuai CIS?

A. atm
B. can
C. sctp
D. tipc

**Jawaban benar:** A, B, C, D

**Pembahasan:** CIS membahas module jaringan tidak umum: atm, can, dccp, rds, sctp, tipc.

---

## 3.3 IPv4 Kernel Parameters

### Soal 60

**Pertanyaan:** Sysctl mana yang benar agar host biasa tidak menjadi router IPv4?

A. `net.ipv4.ip_forward = 0`
B. `net.ipv4.conf.all.forwarding = 0`
C. `net.ipv4.conf.default.forwarding = 0`
D. `net.ipv4.ip_forward = 1`

**Jawaban benar:** A, B, C

**Pembahasan:** Nilai 0 menonaktifkan forwarding pada host non-router.

---
### Soal 61

**Pertanyaan:** Sysctl mana yang benar untuk menolak ICMP redirect/source route pada IPv4?

A. `net.ipv4.conf.all.accept_redirects = 0`
B. `net.ipv4.conf.default.accept_redirects = 0`
C. `net.ipv4.conf.all.accept_source_route = 0`
D. `net.ipv4.conf.default.accept_source_route = 0`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua opsi benar untuk hardening routing input.

---
### Soal 62

**Pertanyaan:** Sysctl mana yang benar untuk perlindungan spoofing/SYN flood dan logging paket martian?

A. `net.ipv4.conf.all.rp_filter = 1`
B. `net.ipv4.conf.default.rp_filter = 1`
C. `net.ipv4.tcp_syncookies = 1`
D. `net.ipv4.conf.all.log_martians = 1`

**Jawaban benar:** A, B, C, D

**Pembahasan:** rp_filter, tcp_syncookies, dan log_martians adalah kontrol hardening jaringan.

---
### Soal 63

**Pertanyaan:** Command mana yang benar untuk menyimpan sysctl hardening secara persisten?

A. `printf '%s\n' 'net.ipv4.ip_forward = 0' > /etc/sysctl.d/60-cis-network.conf`
B. `sysctl --system`
C. `sysctl -w net.ipv4.ip_forward=0`
D. `echo net.ipv4.ip_forward=0 > /etc/modules`

**Jawaban benar:** A, B, C

**Pembahasan:** A persisten, B load semua, C runtime. D salah lokasi.

---

## 3.3 IPv6 Kernel Parameters

### Soal 64

**Pertanyaan:** Sysctl mana yang benar jika IPv6 digunakan tetapi perlu di-hardening?

A. `net.ipv6.conf.all.forwarding = 0`
B. `net.ipv6.conf.default.forwarding = 0`
C. `net.ipv6.conf.all.accept_redirects = 0`
D. `net.ipv6.conf.default.accept_source_route = 0`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Jika IPv6 aktif, kontrol keamanan IPv6 tetap harus diterapkan.

---
### Soal 65

**Pertanyaan:** Manakah command yang benar untuk menonaktifkan IPv6 melalui GRUB pada implementasi yang memang tidak memakai IPv6?

A. `Tambahkan `ipv6.disable=1` pada GRUB_CMDLINE_LINUX lalu jalankan `update-grub``
B. `sysctl -w ipv6.disable=1`
C. `ufw disable ipv6`
D. `echo disable > /proc/ipv6`

**Jawaban benar:** A

**Pembahasan:** Penonaktifan IPv6 via boot parameter dilakukan di GRUB, lalu update-grub dan reboot.

---

## 3 Network

### Soal 66

**Pertanyaan:** Pilih command yang benar untuk mengaktifkan forwarding IPv4 pada server biasa sesuai baseline CIS Level 1 non-router.

A. `sysctl -w net.ipv4.ip_forward=1`
B. `echo 'net.ipv4.ip_forward = 1' > /etc/sysctl.d/60-forward.conf`
C. `ufw route allow in on eth0 out on eth1`
D. `iptables -P FORWARD ACCEPT`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: host biasa seharusnya tidak mengaktifkan forwarding kecuali ada exception sebagai router/gateway/container host.

---

## 4.1 UFW Official CIS Debian

### Soal 67

**Pertanyaan:** Perintah manakah yang benar untuk memasang dan mengaktifkan UFW pada Debian sesuai CIS?

A. `apt install ufw`
B. `systemctl unmask ufw.service`
C. `systemctl --now enable ufw.service`
D. `ufw enable`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua benar dalam alur UFW: install, unmask, enable service, enable UFW.

---
### Soal 68

**Pertanyaan:** Sebelum mengaktifkan default deny incoming pada server yang diadministrasi via SSH, command mana yang benar agar tidak terkunci?

A. `ufw allow proto tcp from any to any port 22`
B. `ufw allow ssh`
C. `ufw allow from 192.168.10.20 to any port 22 proto tcp`
D. `ufw deny ssh`

**Jawaban benar:** A, B, C

**Pembahasan:** Allow SSH sebaiknya dibatasi IP admin; deny ssh berisiko lockout.

---
### Soal 69

**Pertanyaan:** Command manakah yang benar untuk default policy UFW sesuai CIS Level 1?

A. `ufw default deny incoming`
B. `ufw default allow outgoing`
C. `ufw default disabled routed`
D. `ufw default allow incoming`

**Jawaban benar:** A, B, C

**Pembahasan:** Level 1: incoming deny, routed disabled. Outgoing default allow; Level 2 dapat menjadi deny outgoing.

---
### Soal 70

**Pertanyaan:** Untuk CIS Level 2 yang sangat ketat terhadap egress, command mana yang benar?

A. `ufw allow out http`
B. `ufw allow out https`
C. `ufw allow out to any port 53`
D. `ufw default deny outgoing`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Allowlist kebutuhan keluar sebelum menerapkan default deny outgoing.

---
### Soal 71

**Pertanyaan:** Command audit mana yang benar untuk melihat status dan default policy UFW?

A. `ufw status verbose`
B. `ufw status verbose | awk -F',' '$1~/Default/ {print $1}'`
C. `systemctl is-active ufw.service`
D. `firewall-cmd --state`

**Jawaban benar:** A, B, C

**Pembahasan:** CIS Debian Bab 4 memakai UFW. `firewall-cmd` adalah firewalld, bukan audit UFW.

---
### Soal 72

**Pertanyaan:** Manakah command berikut yang benar untuk konfigurasi firewall resmi CIS Debian Bab 4 jika pertanyaannya spesifik menyebut UFW?

A. `ufw default deny incoming`
B. `firewall-cmd --set-default-zone=public`
C. `ufw default disabled routed`
D. `ufw enable`

**Jawaban benar:** A, C, D

**Pembahasan:** firewall-cmd bukan UFW, walaupun bisa menjadi adaptasi lain.

---

## 4.2 Adaptasi Firewalld

### Soal 73

**Pertanyaan:** Jika organisasi memilih firewalld sebagai adaptasi prinsip default-zone dan allowlist, command mana yang benar secara sintaks firewalld?

A. `systemctl enable --now firewalld`
B. `firewall-cmd --set-default-zone=public`
C. `firewall-cmd --permanent --zone=public --add-service=ssh`
D. `firewall-cmd --reload`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Ini valid sebagai adaptasi firewalld, tetapi bukan metode resmi Bab 4 CIS Debian yang memakai UFW.

---
### Soal 74

**Pertanyaan:** Pada firewalld, command mana yang benar untuk mengizinkan SSH hanya dari IP admin `192.168.10.20/32` ke port 22?

A. `firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.10.20/32" port protocol="tcp" port="22" accept'`
B. `firewall-cmd --reload`
C. `firewall-cmd --permanent --zone=public --add-service=all`
D. `firewall-cmd --zone=public --open-everything`

**Jawaban benar:** A, B

**Pembahasan:** Rich rule + reload benar. `all`/open-everything tidak sesuai allowlist.

---
### Soal 75

**Pertanyaan:** Jika soal menyebut 'CIS Debian resmi Bab 4', command mana yang benar untuk mengganti UFW dengan firewalld?

A. `apt install firewalld`
B. `systemctl enable --now firewalld`
C. `firewall-cmd --set-default-zone=public`
D. `firewall-cmd --permanent --add-service=all`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: CIS Debian resmi Bab 4 memilih UFW. Firewalld hanya adaptasi lokal, bukan jawaban resmi untuk Bab 4 CIS Debian.

---

## 4 Firewall

### Soal 76

**Pertanyaan:** Manakah command yang salah/tidak sesuai prinsip allowlist?

A. `ufw default allow incoming`
B. `ufw allow 1:65535/tcp`
C. `firewall-cmd --permanent --zone=public --add-service=all`
D. `ufw disable`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua melemahkan kontrol jika dipilih sebagai hardening umum.

---

## 5.1 SSH File Access

### Soal 77

**Pertanyaan:** Command mana yang benar untuk mengamankan `/etc/ssh/sshd_config`?

A. `chown root:root /etc/ssh/sshd_config`
B. `chmod 600 /etc/ssh/sshd_config`
C. `chmod 777 /etc/ssh/sshd_config`
D. `chown nobody:nogroup /etc/ssh/sshd_config`

**Jawaban benar:** A, B

**Pembahasan:** sshd_config harus dimiliki root dan tidak writable oleh user biasa.

---

## 5.1 SSH Host Keys

### Soal 78

**Pertanyaan:** Command mana yang benar untuk private host key SSH?

A. `find /etc/ssh -xdev -type f -name 'ssh_host_*_key' -exec chown root:root {} \;`
B. `find /etc/ssh -xdev -type f -name 'ssh_host_*_key' -exec chmod 600 {} \;`
C. `find /etc/ssh -xdev -type f -name 'ssh_host_*_key.pub' -exec chmod 600 {} \;`
D. `chmod 777 /etc/ssh/ssh_host_ed25519_key`

**Jawaban benar:** A, B

**Pembahasan:** Private key harus root-owned dan mode ketat.

---
### Soal 79

**Pertanyaan:** Command mana yang benar untuk public host key SSH?

A. `find /etc/ssh -xdev -type f -name 'ssh_host_*_key.pub' -exec chown root:root {} \;`
B. `find /etc/ssh -xdev -type f -name 'ssh_host_*_key.pub' -exec chmod 644 {} \;`
C. `chmod 600 /etc/ssh/ssh_host_ed25519_key.pub`
D. `chown user:user /etc/ssh/ssh_host_ed25519_key.pub`

**Jawaban benar:** A, B

**Pembahasan:** Public key tidak rahasia, tetapi ownership harus benar dan tidak writable sembarang.

---

## 5.1 SSH Server Hardening

### Soal 80

**Pertanyaan:** Directive mana yang benar untuk mencegah login root dan password kosong melalui SSH?

A. `PermitRootLogin no`
B. `PermitEmptyPasswords no`
C. `PasswordAuthentication no`
D. `PermitRootLogin yes`

**Jawaban benar:** A, B, C

**Pembahasan:** A/B wajib untuk hardening; PasswordAuthentication no lebih ketat jika key-based auth dipakai.

---
### Soal 81

**Pertanyaan:** Directive mana yang benar untuk membatasi brute force dan koneksi menggantung?

A. `MaxAuthTries 3`
B. `LoginGraceTime 60`
C. `MaxStartups 10:30:60`
D. `MaxAuthTries 99`

**Jawaban benar:** A, B, C

**Pembahasan:** Nilai ketat membatasi percobaan dan koneksi unauthenticated; 99 terlalu longgar.

---
### Soal 82

**Pertanyaan:** Directive mana yang benar untuk menonaktifkan forwarding/tunneling SSH yang tidak diperlukan?

A. `DisableForwarding yes`
B. `AllowTcpForwarding no`
C. `X11Forwarding no`
D. `AllowAgentForwarding no`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua mengurangi penyalahgunaan SSH forwarding sesuai konteks hardening.

---
### Soal 83

**Pertanyaan:** Directive mana yang benar untuk integrasi PAM dan logging SSH?

A. `UsePAM yes`
B. `LogLevel INFO`
C. `Banner /etc/issue.net`
D. `UsePAM no`

**Jawaban benar:** A, B, C

**Pembahasan:** UsePAM yes menghubungkan kebijakan PAM; INFO dan Banner mendukung audit/legal notice.

---
### Soal 84

**Pertanyaan:** Command audit mana yang benar untuk melihat effective configuration sshd?

A. `sshd -T`
B. `sshd -T | grep -Ei 'permitrootlogin|usepam|banner|maxauthtries'`
C. `systemctl reload sshd`
D. `cat /etc/shadow`

**Jawaban benar:** A, B

**Pembahasan:** `sshd -T` menampilkan konfigurasi efektif; reload bukan audit, cat shadow tidak perlu.

---
### Soal 85

**Pertanyaan:** Setelah mengubah konfigurasi SSH, command mana yang benar untuk validasi dan reload?

A. `sshd -t`
B. `systemctl reload sshd.service`
C. `systemctl restart sshd.service`
D. `rm /etc/ssh/sshd_config`

**Jawaban benar:** A, B, C

**Pembahasan:** Validasi syntax dengan sshd -t sebelum reload/restart.

---

## 5.2 Sudo

### Soal 86

**Pertanyaan:** Command mana yang benar untuk memasang dan mengonfigurasi sudo agar command memakai pty dan log file?

A. `apt install sudo`
B. `printf '%s\n' 'Defaults use_pty' 'Defaults logfile="/var/log/sudo.log"' > /etc/sudoers.d/00-cis-hardening`
C. `chmod 440 /etc/sudoers.d/00-cis-hardening`
D. `visudo -cf /etc/sudoers.d/00-cis-hardening`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Pakai visudo -cf untuk validasi file sudoers drop-in.

---
### Soal 87

**Pertanyaan:** Manakah konfigurasi sudo yang salah menurut prinsip CIS karena meniadakan password escalation?

A. `kautsar ALL=(ALL:ALL) NOPASSWD:ALL`
B. `%wheel ALL=(ALL:ALL) NOPASSWD:ALL`
C. `Defaults timestamp_timeout=-1`
D. `Defaults use_pty`

**Jawaban benar:** A, B, C

**Pembahasan:** NOPASSWD dan timestamp tak terbatas melemahkan re-authentication.

---
### Soal 88

**Pertanyaan:** Command mana yang benar untuk membatasi `su` hanya bagi group tertentu melalui PAM?

A. `Buat group khusus misalnya `groupadd sugroup``
B. `Tambahkan user yang boleh ke group tersebut`
C. `Tambahkan `auth required pam_wheel.so use_uid group=sugroup` di `/etc/pam.d/su``
D. `chmod 777 /bin/su`

**Jawaban benar:** A, B, C

**Pembahasan:** CIS membatasi akses su; chmod 777 berbahaya.

---

## 5.3 PAM Packages

### Soal 89

**Pertanyaan:** Paket mana yang benar untuk mendukung kebijakan PAM/password quality pada Debian?

A. `apt install libpam-pwquality`
B. `apt install cracklib-runtime`
C. `apt install libpam-modules`
D. `apt install pam_google_authenticator wajib karena CIS Debian`

**Jawaban benar:** A, B, C

**Pembahasan:** Google Authenticator bukan kewajiban CIS Debian.

---

## 5.3 PAM

### Soal 90

**Pertanyaan:** Command mana yang benar untuk mencari apakah `pam_unix.so` masih memakai `nullok`?

A. `grep -R 'pam_unix.so.*nullok' /etc/pam.d/`
B. `grep -R 'pam_unix.so' /etc/pam.d/`
C. `sed -i 's/nullok//g' /etc/pam.d/common-*`
D. `echo nullok >> /etc/pam.d/common-auth`

**Jawaban benar:** A, B

**Pembahasan:** A/B audit. C bisa remediation tapi terlalu agresif tanpa review; D salah.

---

## 5.3 PAM pwquality

### Soal 91

**Pertanyaan:** Konfigurasi mana yang benar di `/etc/security/pwquality.conf`?

A. `minlen = 14`
B. `dictcheck = 1`
C. enforce_for_root
D. `minlen = 4`

**Jawaban benar:** A, B, C

**Pembahasan:** CIS mengarah pada panjang, dictionary check, dan enforcement termasuk root.

---

## 5.3 PAM faillock

### Soal 92

**Pertanyaan:** Konfigurasi mana yang benar di `/etc/security/faillock.conf` untuk lockout login gagal?

A. `deny = 5`
B. `unlock_time = 900`
C. even_deny_root
D. `deny = 0`

**Jawaban benar:** A, B, C

**Pembahasan:** deny>0, unlock time, dan root included memperkuat lockout.

---

## 5.3 PAM pwhistory

### Soal 93

**Pertanyaan:** Konfigurasi mana yang benar untuk password history?

A. `remember = 24`
B. enforce_for_root
C. use_authtok
D. `remember = 0`

**Jawaban benar:** A, B, C

**Pembahasan:** Password history mencegah reuse; 0 mematikan history.

---

## 5.3 PAM

### Soal 94

**Pertanyaan:** Manakah opsi hashing password yang kuat untuk `pam_unix` atau login.defs?

A. yescrypt
B. sha512
C. md5
D. des

**Jawaban benar:** A, B

**Pembahasan:** yescrypt/sha512 kuat; md5/des lemah.

---

## 5.4 User Accounts

### Soal 95

**Pertanyaan:** Command mana yang benar untuk mengatur masa berlaku password user?

A. `chage --maxdays 365 username`
B. `chage --mindays 1 username`
C. `chage --warndays 7 username`
D. `passwd -d username`

**Jawaban benar:** A, B, C

**Pembahasan:** passwd -d menghapus password user dan berbahaya.

---
### Soal 96

**Pertanyaan:** Command audit mana yang benar untuk memastikan root satu-satunya UID 0?

A. `awk -F: '($3 == 0) { print $1 }' /etc/passwd`
B. `getent passwd 0`
C. `cut -d: -f1,3 /etc/passwd | awk -F: '$2==0{print}'`
D. `grep root /etc/group`

**Jawaban benar:** A, C

**Pembahasan:** Cek UID 0 pada /etc/passwd; root group bukan cek UID 0 user.

---
### Soal 97

**Pertanyaan:** Command mana yang benar untuk mengunci akun sistem yang tidak punya valid login shell?

A. `usermod -L <account>`
B. `passwd -l <account>`
C. `chsh -s /usr/sbin/nologin <account>`
D. `passwd -d <account>`

**Jawaban benar:** A, B, C

**Pembahasan:** Mengunci akun dan memakai nologin benar; menghapus password salah.

---

## 5.4 User Environment

### Soal 98

**Pertanyaan:** Command mana yang benar untuk memastikan `nologin` tidak terdaftar sebagai shell valid?

A. `grep -E '(/nologin|/false)$' /etc/shells`
B. `sed -i '/nologin/d;/false/d' /etc/shells`
C. `echo /usr/sbin/nologin >> /etc/shells`
D. `cat /etc/shells`

**Jawaban benar:** A, B, D

**Pembahasan:** Audit dengan grep/cat; remediation hapus nologin/false dari /etc/shells.

---
### Soal 99

**Pertanyaan:** Konfigurasi mana yang benar untuk default umask user yang lebih ketat?

A. `umask 027`
B. `UMASK 027`
C. `umask 000`
D. `UMASK 000`

**Jawaban benar:** A, B

**Pembahasan:** Umask 027 membatasi akses group/others; 000 terlalu permisif.

---

## 5 Access Control

### Soal 100

**Pertanyaan:** Pilih command yang benar untuk mengaktifkan login root SSH langsung sesuai CIS Debian.

A. `echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config`
B. `sed -i 's/^PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config`
C. `systemctl reload sshd`
D. `passwd -u root`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: CIS menghendaki root SSH login disabled, bukan enabled.

---

## 6.1 journald

### Soal 101

**Pertanyaan:** Command mana yang benar untuk audit journald aktif dan persisten?

A. `systemctl is-active systemd-journald.service`
B. `journalctl --list-boots`
C. `grep -R '^Storage=persistent' /etc/systemd/journald.conf /etc/systemd/journald.conf.d/*.conf 2>/dev/null`
D. `systemctl disable systemd-journald`

**Jawaban benar:** A, B, C

**Pembahasan:** journald harus aktif; Storage=persistent menjaga log lintas boot.

---
### Soal 102

**Pertanyaan:** Konfigurasi mana yang benar di journald untuk logging yang sesuai prinsip CIS?

A. `Storage=persistent`
B. `Compress=yes`
C. `ForwardToSyslog=yes`
D. `Storage=volatile`

**Jawaban benar:** A, B, C

**Pembahasan:** ForwardToSyslog menghubungkan journald ke rsyslog; volatile melemahkan persistensi.

---

## 6.1 rsyslog

### Soal 103

**Pertanyaan:** Command mana yang benar untuk memasang dan mengaktifkan rsyslog?

A. `apt install rsyslog`
B. `systemctl enable --now rsyslog.service`
C. `systemctl is-active rsyslog.service`
D. `systemctl disable --now rsyslog.service`

**Jawaban benar:** A, B, C

**Pembahasan:** Rsyslog perlu installed, enabled, active.

---
### Soal 104

**Pertanyaan:** Konfigurasi mana yang benar untuk mode file log rsyslog agar tidak terlalu permisif?

A. `$FileCreateMode 0640`
B. `$FileCreateMode 0600`
C. `$FileCreateMode 0777`
D. `$FileCreateMode 0666`

**Jawaban benar:** A, B

**Pembahasan:** 0640/0600 membatasi akses; 0777/0666 terlalu terbuka.

---

## 6.1 rsyslog Remote TLS

### Soal 105

**Pertanyaan:** Konfigurasi mana yang benar untuk forwarding rsyslog via TLS/gtls?

A. `$DefaultNetstreamDriver gtls`
B. `$DefaultNetstreamDriverCAFile /etc/ssl/certs/ca-certificates.crt`
C. `*.* @@loghost.example.org:6514`
D. `*.* @loghost.example.org:514 tanpa TLS`

**Jawaban benar:** A, B, C

**Pembahasan:** @@ untuk TCP; gtls+CA untuk TLS. UDP tanpa TLS tidak memenuhi tujuan enkripsi.

---

## 6.1 logrotate

### Soal 106

**Pertanyaan:** Command mana yang benar untuk audit/konfigurasi logrotate?

A. `apt install logrotate`
B. `logrotate -d /etc/logrotate.conf`
C. `grep -R 'rotate\|compress' /etc/logrotate.conf /etc/logrotate.d/`
D. `rm -rf /var/log/*`

**Jawaban benar:** A, B, C

**Pembahasan:** logrotate menjaga ukuran/retensi log. Menghapus log bukan konfigurasi.

---

## 6.2 auditd Service

### Soal 107

**Pertanyaan:** Command mana yang benar untuk memasang dan mengaktifkan auditd pada Debian?

A. `apt install auditd audispd-plugins`
B. `systemctl enable --now auditd.service`
C. `systemctl is-active auditd.service`
D. `systemctl disable --now auditd.service`

**Jawaban benar:** A, B, C

**Pembahasan:** auditd harus installed, enabled, active.

---

## 6.2 auditd Boot

### Soal 108

**Pertanyaan:** Parameter GRUB mana yang benar agar audit aktif sejak awal boot?

A. `audit=1`
B. `audit_backlog_limit=8192`
C. `audit=0`
D. `noaudit`

**Jawaban benar:** A, B

**Pembahasan:** audit=1 dan backlog limit memberi cakupan sejak boot.

---

## 6.2 auditd Rules

### Soal 109

**Pertanyaan:** Rule audit mana yang benar untuk mencatat perubahan sudoers?

A. `-w /etc/sudoers -p wa -k scope`
B. `-w /etc/sudoers.d -p wa -k scope`
C. `-w /etc/passwd -p wa -k identity`
D. `-w /etc/sudoers -p x -k execute`

**Jawaban benar:** A, B

**Pembahasan:** Sudoers perlu watch write/attribute changes.

---
### Soal 110

**Pertanyaan:** Rule audit mana yang benar untuk perubahan identitas user/group?

A. `-w /etc/passwd -p wa -k identity`
B. `-w /etc/group -p wa -k identity`
C. `-w /etc/shadow -p wa -k identity`
D. `-w /etc/gshadow -p wa -k identity`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua file identity sensitif perlu diaudit.

---
### Soal 111

**Pertanyaan:** Rule audit mana yang benar untuk perubahan waktu sistem pada arsitektur 64-bit?

A. `-a always,exit -F arch=b64 -S adjtimex,settimeofday,clock_settime -k time-change`
B. `-w /etc/localtime -p wa -k time-change`
C. `-a never,exit -S clock_settime -k time-change`
D. `-w /etc/passwd -p x -k time-change`

**Jawaban benar:** A, B

**Pembahasan:** A mencatat syscall perubahan waktu, B perubahan zona waktu.

---
### Soal 112

**Pertanyaan:** Rule audit mana yang benar untuk perubahan hostname/domain?

A. `-a always,exit -F arch=b64 -S sethostname,setdomainname -k system-locale`
B. `-w /etc/hostname -p wa -k system-locale`
C. `-w /etc/hosts -p wa -k system-locale`
D. `-a always,exit -S reboot -k hostname`

**Jawaban benar:** A, B, C

**Pembahasan:** Syscall dan file hostname/hosts relevan untuk perubahan identitas network host.

---
### Soal 113

**Pertanyaan:** Rule audit mana yang benar untuk unsuccessful file access attempts?

A. `-a always,exit -F arch=b64 -S open,openat,creat,truncate,ftruncate -F exit=-EACCES -F auid>=1000 -F auid!=unset -k access`
B. `-a always,exit -F arch=b64 -S open,openat,creat,truncate,ftruncate -F exit=-EPERM -F auid>=1000 -F auid!=unset -k access`
C. `-a never,exit -F exit=-EACCES`
D. `-w /etc/passwd -p r -k access`

**Jawaban benar:** A, B

**Pembahasan:** EACCES dan EPERM adalah gagal akses yang penting.

---
### Soal 114

**Pertanyaan:** Rule audit mana yang benar untuk DAC permission modification?

A. `-a always,exit -F arch=b64 -S chmod,fchmod,fchmodat -F auid>=1000 -F auid!=unset -k perm_mod`
B. `-a always,exit -F arch=b64 -S chown,fchown,lchown,fchownat -F auid>=1000 -F auid!=unset -k perm_mod`
C. `-a always,exit -F arch=b64 -S setxattr,lsetxattr,fsetxattr,removexattr,lremovexattr,fremovexattr -F auid>=1000 -F auid!=unset -k perm_mod`
D. `-a never,exit -S chmod`

**Jawaban benar:** A, B, C

**Pembahasan:** chmod/chown/xattr changes relevan untuk DAC modification.

---
### Soal 115

**Pertanyaan:** Rule audit mana yang benar untuk kernel module load/unload?

A. `-a always,exit -F arch=b64 -S init_module,finit_module,delete_module -F auid>=1000 -F auid!=unset -k kernel_modules`
B. `-w /usr/bin/kmod -p x -k kernel_modules`
C. `-e 0`
D. `modprobe --audit all`

**Jawaban benar:** A, B

**Pembahasan:** Syscall module dan eksekusi kmod perlu dicatat; -e 0 mematikan immutable/audit lock.

---
### Soal 116

**Pertanyaan:** Command mana yang benar untuk memuat rule audit dan memeriksa status?

A. `augenrules --load`
B. `auditctl -s`
C. `auditctl -l`
D. `systemctl stop auditd`

**Jawaban benar:** A, B, C

**Pembahasan:** augenrules load; auditctl melihat status/list rules.

---
### Soal 117

**Pertanyaan:** Baris mana yang benar untuk membuat konfigurasi audit immutable setelah rules selesai?

A. `-e 2`
B. `auditctl -e 2`
C. `-e 0`
D. `auditctl -D`

**Jawaban benar:** A, B

**Pembahasan:** -e 2 mengunci audit config sampai reboot; -e 0/D melemahkan/menghapus rules.

---

## 6.2 auditd File Access

### Soal 118

**Pertanyaan:** Command mana yang benar untuk permission audit log dan konfigurasi audit?

A. `chmod 750 /var/log/audit`
B. `find /var/log/audit -type f -exec chmod 640 {} \;`
C. `chown root:root /etc/audit/auditd.conf`
D. `chmod 777 /etc/audit/auditd.conf`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit log/config harus tidak writable oleh user biasa.

---

## 6.3 AIDE

### Soal 119

**Pertanyaan:** Command mana yang benar untuk memasang dan menginisialisasi AIDE pada Debian?

A. `apt install aide aide-common`
B. aideinit
C. `aide --check`
D. `rm -f /var/lib/aide/aide.db`

**Jawaban benar:** A, B, C

**Pembahasan:** AIDE perlu diinstall, init database, dan check berkala.

---

## 6 Logging and Auditing

### Soal 120

**Pertanyaan:** Pilih command yang benar untuk menghapus audit log otomatis agar storage tidak penuh sesuai prinsip CIS.

A. `max_log_file_action = ignore`
B. `rm -f /var/log/audit/audit.log`
C. `service auditd stop && rm -rf /var/log/audit`
D. `sed -i 's/^max_log_file_action.*/max_log_file_action = keep_logs/' /etc/audit/auditd.conf`

**Jawaban benar:** D

**Pembahasan:** CIS mendorong audit log tidak otomatis dihapus; keep_logs adalah arah yang benar.

---

## 7.1 System File Access

### Soal 121

**Pertanyaan:** Command mana yang benar untuk permission `/etc/passwd` dan backup-nya?

A. `chown root:root /etc/passwd /etc/passwd-`
B. `chmod 0644 /etc/passwd /etc/passwd-`
C. `chmod 0666 /etc/passwd`
D. `chown user:user /etc/passwd`

**Jawaban benar:** A, B

**Pembahasan:** passwd boleh readable tetapi tidak writable oleh user biasa.

---
### Soal 122

**Pertanyaan:** Command mana yang benar untuk `/etc/shadow` dan `/etc/shadow-`?

A. `chown root:shadow /etc/shadow /etc/shadow- 2>/dev/null`
B. `chmod 0640 /etc/shadow /etc/shadow- 2>/dev/null`
C. `chmod 0644 /etc/shadow`
D. `cat /etc/shadow > /tmp/shadow.backup`

**Jawaban benar:** A, B

**Pembahasan:** Shadow berisi hash password dan harus sangat dibatasi.

---
### Soal 123

**Pertanyaan:** Command mana yang benar untuk `/etc/group` dan `/etc/group-`?

A. `chown root:root /etc/group /etc/group-`
B. `chmod 0644 /etc/group /etc/group-`
C. `chmod 0777 /etc/group`
D. `chown nobody:nogroup /etc/group`

**Jawaban benar:** A, B

**Pembahasan:** Group file tidak boleh writable user biasa.

---
### Soal 124

**Pertanyaan:** Command mana yang benar untuk `/etc/gshadow`?

A. `chown root:shadow /etc/gshadow /etc/gshadow- 2>/dev/null`
B. `chmod 0640 /etc/gshadow /etc/gshadow- 2>/dev/null`
C. `chmod 0644 /etc/gshadow`
D. `echo '*' > /etc/gshadow`

**Jawaban benar:** A, B

**Pembahasan:** gshadow sensitif seperti shadow group.

---
### Soal 125

**Pertanyaan:** Command mana yang benar untuk `/etc/shells`?

A. `chown root:root /etc/shells`
B. `chmod 0644 /etc/shells`
C. `chmod 0777 /etc/shells`
D. `echo /bin/bash >> /etc/shells`

**Jawaban benar:** A, B

**Pembahasan:** File shells boleh readable tetapi tidak writable user biasa. Menambah shell harus sesuai kebijakan, bukan otomatis benar.

---

## 7.1 World Writable

### Soal 126

**Pertanyaan:** Command audit mana yang benar untuk mencari world writable file/directory pada satu filesystem?

A. `find / -xdev -type f -perm -0002 -print`
B. `find / -xdev -type d -perm -0002 -print`
C. `find / -xdev -type d -perm -0002 ! -perm -1000 -print`
D. `chmod -R 777 /`

**Jawaban benar:** A, B, C

**Pembahasan:** C mencari world-writable directory tanpa sticky bit.

---

## 7.1 No Owner/Group

### Soal 127

**Pertanyaan:** Command mana yang benar untuk mencari file tanpa owner/group?

A. `find / -xdev \( -nouser -o -nogroup \) -print`
B. `find / -nouser -print`
C. `find / -nogroup -print`
D. `chown -R root:root /`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit dahulu; jangan sembarang chown seluruh filesystem.

---

## 7.1 SUID/SGID

### Soal 128

**Pertanyaan:** Command mana yang benar untuk review file SUID/SGID?

A. `find / -xdev -type f -perm /6000 -exec ls -l {} \;`
B. `find / -xdev -type f -perm -4000 -exec ls -l {} \;`
C. `find / -xdev -type f -perm -2000 -exec ls -l {} \;`
D. `chmod -R u+s /usr/bin`

**Jawaban benar:** A, B, C

**Pembahasan:** A semua SUID/SGID, B SUID, C SGID. D sangat berbahaya.

---

## 7.2 Local Users

### Soal 129

**Pertanyaan:** Command mana yang benar untuk memastikan akun di `/etc/passwd` memakai shadowed passwords?

A. `awk -F: '($2 != "x") { print $1 }' /etc/passwd`
B. pwconv
C. `awk -F: '($2 == "") { print $1 }' /etc/shadow`
D. pwunconv

**Jawaban benar:** A, B, C

**Pembahasan:** A audit passwd shadowed, B mengaktifkan shadow, C audit password kosong di shadow. pwunconv melemahkan shadow.

---
### Soal 130

**Pertanyaan:** Command mana yang benar untuk mencari password field kosong di `/etc/shadow`?

A. `awk -F: '($2 == "") { print $1 }' /etc/shadow`
B. `passwd -l <user>`
C. `passwd -d <user>`
D. `grep '::' /etc/shadow`

**Jawaban benar:** A, B, D

**Pembahasan:** A/D audit; B remediation mengunci akun. passwd -d menghapus password dan buruk.

---
### Soal 131

**Pertanyaan:** Command mana yang benar untuk mencari duplicate UID/GID?

A. `cut -d: -f3 /etc/passwd | sort -n | uniq -d`
B. `cut -d: -f3 /etc/group | sort -n | uniq -d`
C. `cut -d: -f1 /etc/passwd | sort | uniq -d`
D. `cut -d: -f1 /etc/group | sort | uniq -d`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua audit duplikasi UID, GID, username, group name.

---
### Soal 132

**Pertanyaan:** Command mana yang benar untuk memastikan semua group utama user ada di `/etc/group`?

A. `awk -F: '{print $4}' /etc/passwd | sort -u`
B. `awk -F: '{print $3}' /etc/group | sort -u`
C. `comm -23 <(awk -F: '{print $4}' /etc/passwd | sort -u) <(awk -F: '{print $3}' /etc/group | sort -u)`
D. `rm /etc/group`

**Jawaban benar:** A, B, C

**Pembahasan:** Bandingkan GID utama user dengan GID group yang ada.

---
### Soal 133

**Pertanyaan:** Command mana yang benar untuk audit home directory user interaktif?

A. `awk -F: '($3>=1000 && $7 !~ /(nologin|false)$/) {print $1,$6}' /etc/passwd`
B. `stat -c '%U %G %A %n' /home/* 2>/dev/null`
C. `chmod -R 777 /home`
D. `chown -R root:root /home`

**Jawaban benar:** A, B

**Pembahasan:** Audit user interaktif dan ownership/permission home; jangan chmod/chown massal tanpa analisis.

---

## 7.2 Dot Files

### Soal 134

**Pertanyaan:** Command mana yang benar untuk mencari dot file berisiko pada home user?

A. `find /home -name .rhosts -o -name .netrc -o -name .forward`
B. `find /home -name .bashrc -delete`
C. `find /home -type f -name '.ssh'`
D. `find /home -name .netrc -exec ls -l {} \;`

**Jawaban benar:** A, D

**Pembahasan:** .rhosts/.netrc/.forward berisiko; jangan menghapus .bashrc sembarangan.

---

## 7 Maintenance

### Soal 135

**Pertanyaan:** Pilih command yang benar untuk memperbaiki semua temuan permission dengan cepat tanpa review manual.

A. `chmod -R 600 /etc`
B. `chown -R root:root /home`
C. `find / -perm /6000 -delete`
D. `chmod -R 777 /var/log`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: semua opsi berbahaya. Maintenance CIS memerlukan audit dan remediation terkontrol.

---

## 8. Bonus Implementasi Arch Berbasis Prinsip CIS

### Soal 136

**Pertanyaan:** Pada panduan implementasi Arch berbasis prinsip CIS Debian, command mana yang benar untuk skema LUKS2 + LVM?

A. `cryptsetup luksFormat --type luks2 /dev/nvme0n1p3`
B. `cryptsetup open /dev/nvme0n1p3 cryptlvm`
C. `pvcreate /dev/mapper/cryptlvm`
D. `vgcreate vgarch /dev/mapper/cryptlvm`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Ini berasal dari file implementasi Arch yang mengadaptasi prinsip CIS Debian.

---
### Soal 137

**Pertanyaan:** Command mana yang benar untuk mount partisi hasil LVM sesuai implementasi Arch?

A. `mount /dev/vgarch/lv_root /mnt`
B. `mount /dev/vgarch/lv_home /mnt/home`
C. `mount /dev/vgarch/lv_log /mnt/var/log`
D. `mount /dev/vgarch/lv_audit /mnt/var/log/audit`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua mount point sesuai rancangan LVM hardening.

---
### Soal 138

**Pertanyaan:** Command mana yang benar untuk memasang paket dasar pada Arch sesuai file implementasi?

A. `pacstrap -K /mnt base linux-hardened linux-firmware`
B. `pacstrap -K /mnt grub efibootmgr cryptsetup lvm2`
C. `pacstrap -K /mnt openssh ufw nftables audit apparmor aide`
D. `apt install linux-hardened pacman`

**Jawaban benar:** A, B, C

**Pembahasan:** Arch memakai pacstrap/pacman, bukan apt.

---
### Soal 139

**Pertanyaan:** Command mana yang benar untuk mkinitcpio pada implementasi Arch terenkripsi?

A. `Tambahkan hook `encrypt` dan `lvm2` pada `/etc/mkinitcpio.conf``
B. `mkinitcpio -P`
C. `update-initramfs -u`
D. `dracut -f`

**Jawaban benar:** A, B

**Pembahasan:** Implementasi Arch memakai mkinitcpio; Debian/other distro command tidak dipakai.

---
### Soal 140

**Pertanyaan:** Command mana yang benar untuk GRUB password pada implementasi Arch?

A. `grub-mkpasswd-pbkdf2`
B. `Tambahkan `set superusers="admin"` dan `password_pbkdf2 admin <hash>``
C. `grub-mkconfig -o /boot/grub/grub.cfg`
D. `passwd grub`

**Jawaban benar:** A, B, C

**Pembahasan:** GRUB password memakai pbkdf2 hash dan regenerate config.

---
### Soal 141

**Pertanyaan:** Command mana yang benar untuk mengaktifkan service dasar pada implementasi Arch?

A. `systemctl enable NetworkManager.service`
B. `systemctl enable sshd.service`
C. `systemctl enable auditd.service`
D. `systemctl enable apparmor.service`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua service dasar tercantum dalam implementasi Arch.

---
### Soal 142

**Pertanyaan:** Pada Arch, command mana yang benar untuk update sistem dan keyring?

A. `pacman -Syu`
B. `pacman-key --init`
C. `pacman-key --populate archlinux`
D. `apt update && apt upgrade`

**Jawaban benar:** A, B, C

**Pembahasan:** Arch menggunakan pacman/pacman-key.

---
### Soal 143

**Pertanyaan:** Pada Arch, konfigurasi firewall resmi dalam file implementasi memakai...

A. `ufw default deny incoming`
B. `ufw default allow outgoing`
C. `ufw default deny routed`
D. `ufw enable`

**Jawaban benar:** A, B, C, D

**Pembahasan:** File implementasi Arch menggunakan UFW sebagai adaptasi CIS Debian.

---

## 8. Bonus Implementasi Firewalld

### Soal 144

**Pertanyaan:** Pada panduan custom berbasis firewalld, command mana yang benar untuk membuat zone admin yang hanya mengizinkan SSH port 10022 dari IP tertentu?

A. `Buat `/etc/firewalld/zones/admin.xml` dengan `<source address="172.27.1.10"/>``
B. `Tambahkan `<port protocol="tcp" port="10022"/>``
C. `systemctl enable firewalld`
D. `Tambahkan `<service name="all"/>``

**Jawaban benar:** A, B, C

**Pembahasan:** Zone berbasis source address dan port spesifik sesuai allowlist; service all tidak sesuai.

---

## 8. Bonus Implementasi

### Soal 145

**Pertanyaan:** Pilih command yang benar untuk menjalankan `apt install ufw` di Arch Linux sesuai file implementasi Arch.

A. `apt install ufw`
B. `apt update`
C. `dpkg-query -s ufw`
D. `systemctl enable apt.service`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: Arch memakai pacman/pacstrap, bukan APT/dpkg.

---