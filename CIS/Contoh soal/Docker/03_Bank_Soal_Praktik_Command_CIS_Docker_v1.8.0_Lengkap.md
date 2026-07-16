# Bank Soal Praktik Command CIS Docker Benchmark v1.8.0 — Pilihan Ganda Jamak

**Basis materi:** `01_CIS_Docker_Benchmark_v1.8.0_Konseptual.md` dan `02_Implementasi_Hardening_Docker_Berbasis_CIS_v1.8.0.md`.

**Aturan latihan:** pilih semua jawaban yang benar. Jika tidak ada opsi benar, jawab **PASS / kosongkan jawaban**. Jangan jalankan command destruktif di host asli; gunakan VM/lab.

**Total soal:** 141 soal.

## Cakupan

- Host configuration dan partisi `/var/lib/docker`
- Instalasi Docker Engine dan validasi daemon
- Pembatasan group `docker`
- Audit rules untuk Docker
- `/etc/docker/daemon.json`
- Permission `docker.service`, `docker.socket`, `/etc/docker`, `daemon.json`, dan `docker.sock`
- Dockerfile, image, secret, scanning, dan content trust
- Network, runtime hardening, Compose, AppArmor/SELinux, seccomp
- Logging, operasi keamanan, image/container sprawl
- Docker Swarm jika digunakan


## 0. Cara Menjawab Soal Praktik

### Soal 1
**Pertanyaan:** Dalam latihan pilihan ganda jamak CIS Docker, apa yang harus dilakukan jika semua opsi command salah?
A. Pilih opsi yang terlihat paling mendekati benar
B. `Pilih semua opsi agar tidak kosong`
C. `PASS / kosongkan jawaban`
D. Pilih opsi A sebagai default

**Jawaban benar:** C

**Pembahasan:** Jika tidak ada opsi yang benar, jawaban harus PASS/kosong agar tidak memilih command yang salah.

---
### Soal 2
**Pertanyaan:** Prinsip utama memilih command hardening Docker yang benar adalah...
A. Memilih command yang paling panjang
B. Memilih command yang sesuai host OS, Docker Engine, path, dan tujuan kontrol CIS
C. Memilih command Kubernetes karena semua container sama
D. `Memilih command yang terlihat aman meski tidak mencapai konfigurasi`

**Jawaban benar:** B

**Pembahasan:** Command harus sesuai konteks Docker host, bukan sekadar terlihat aman.

---

## 1. Host Configuration

### Soal 3
**Pertanyaan:** Command manakah yang benar untuk melakukan update dasar host Ubuntu/Debian sebelum instalasi Docker?
A. `sudo apt update`
B. `sudo apt full-upgrade -y`
C. `sudo apt install -y ca-certificates curl gnupg lsb-release auditd audispd-plugins apparmor apparmor-utils ufw jq`
D. `sudo pacman -Syu docker`

**Jawaban benar:** A, B, C

**Pembahasan:** Pada target Ubuntu/Debian digunakan APT. Pacman adalah package manager Arch.

---
### Soal 4
**Pertanyaan:** Command mana yang benar untuk mengaktifkan auditd dan AppArmor pada host Docker Ubuntu/Debian?
A. `sudo systemctl enable --now auditd`
B. `sudo systemctl enable --now apparmor`
C. `sudo aa-status || true`
D. `sudo systemctl enable --now selinux`

**Jawaban benar:** A, B, C

**Pembahasan:** Ubuntu/Debian memakai AppArmor. SELinux umumnya bukan service bernama selinux pada Ubuntu/Debian.

---
### Soal 5
**Pertanyaan:** Command mana yang benar untuk konfigurasi firewall host dasar sebelum Docker, dengan hanya mengizinkan SSH?
A. `sudo ufw default deny incoming`
B. `sudo ufw default allow outgoing`
C. `sudo ufw allow OpenSSH`
D. `sudo ufw allow 1:65535/tcp`

**Jawaban benar:** A, B, C

**Pembahasan:** UFW default deny incoming dan allow outgoing adalah baseline. Membuka semua port TCP tidak sesuai prinsip allowlist.

---
### Soal 6
**Pertanyaan:** Command mana yang benar untuk mengecek apakah `/var/lib/docker` sudah menjadi mount point terpisah?
A. `mountpoint -- /var/lib/docker || true`
B. `df -h`
C. `docker info -f '{{ .DockerRootDir }}'`
D. `chmod +x /var/lib/docker`

**Jawaban benar:** A, B, C

**Pembahasan:** mountpoint dan df memeriksa storage. DockerRootDir memberi lokasi root Docker. chmod +x bukan audit partisi.

---
### Soal 7
**Pertanyaan:** Manakah baris `/etc/fstab` yang benar untuk memisahkan `/var/lib/docker` pada LV ext4 sesuai prinsip CIS Docker?
A. `/dev/vg0/lv_docker /var/lib/docker ext4 defaults,nodev,nosuid 0 2`
B. `/dev/vg0/lv_docker /var/lib/docker ext4 defaults,nodev,nosuid,noexec 0 2`
C. `/dev/vg0/lv_docker /var/lib/docker ext4 defaults 0 2`
D. `tmpfs /var/lib/docker tmpfs defaults 0 0`

**Jawaban benar:** A

**Pembahasan:** Dokumentasi implementasi memakai defaults,nodev,nosuid. noexec tidak disarankan sembarang pada Docker root karena layer container bisa membutuhkan eksekusi.

---
### Soal 8
**Pertanyaan:** Command mana yang benar untuk membuat logical volume Docker pada contoh LVM?
A. `sudo lvcreate -L 50G -n lv_docker vg0`
B. `sudo mkfs.ext4 /dev/vg0/lv_docker`
C. `sudo mkdir -p /var/lib/docker`
D. `sudo mkfs.ext4 /var/lib/docker`

**Jawaban benar:** A, B, C

**Pembahasan:** LV dibuat, diformat, lalu mount point dibuat. mkfs dilakukan pada block device, bukan path direktori mount.

---
### Soal 9
**Pertanyaan:** Command mana yang benar untuk mengecek identitas dan OS sebelum hardening Docker?
A. `id`
B. `cat /etc/os-release`
C. `uname -a`
D. `docker destroy host`

**Jawaban benar:** A, B, C

**Pembahasan:** id, os-release, dan uname adalah audit dasar; opsi D bukan command benar dan berbahaya.

---
### Soal 10
**Pertanyaan:** Manakah tindakan yang sesuai untuk Docker host sebelum mengaktifkan workload production?
A. Patch OS terlebih dahulu
B. `Aktifkan auditd dan AppArmor/SELinux sesuai distro`
C. Aktifkan firewall host dengan allowlist port yang dibutuhkan
D. Matikan semua log agar performa container cepat

**Jawaban benar:** A, B, C

**Pembahasan:** Docker host tetap Linux host yang harus di-hardening. Logging tidak boleh dimatikan.

---

## 2. Instalasi Docker Engine

### Soal 11
**Pertanyaan:** Command mana yang benar dalam alur instalasi Docker Engine dari repository resmi/tepercaya di Ubuntu/Debian setelah repository tersedia?
A. `sudo apt update`
B. `sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`
C. `sudo systemctl enable --now docker`
D. `sudo systemctl enable --now containerd`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Setelah repo tersedia, Docker Engine dan plugin dipasang lalu service diaktifkan.

---
### Soal 12
**Pertanyaan:** Command mana yang benar untuk validasi instalasi Docker Engine?
A. `docker version`
B. `docker info`
C. `systemctl status docker --no-pager`
D. kubectl get pods

**Jawaban benar:** A, B, C

**Pembahasan:** Validasi Docker memakai docker version/info dan status service. kubectl adalah Kubernetes, bukan validasi Docker Engine dasar.

---
### Soal 13
**Pertanyaan:** Pilih command yang benar untuk membuat direktori keyring APT pada Ubuntu/Debian sebelum mengikuti prosedur repo resmi Docker.
A. `sudo install -m 0755 -d /etc/apt/keyrings`
B. `sudo mkdir -p /etc/apt/keyrings`
C. `sudo chmod 0755 /etc/apt/keyrings`
D. `sudo install -m 0777 -d /etc/apt/keyrings`

**Jawaban benar:** A, B, C

**Pembahasan:** Direktori keyring boleh dibuat dengan mode aman. Mode 0777 terlalu permisif.

---
### Soal 14
**Pertanyaan:** Command mana yang benar untuk memasang Docker Engine pada Ubuntu/Debian tanpa repository Docker resmi/organisasi yang sudah tersedia?
A. `sudo apt install -y docker-ce docker-ce-cli containerd.io`
B. `sudo apt update && sudo apt install -y docker-ce`
C. `sudo systemctl enable --now docker`
D. `sudo docker production-hardening --cis`

**Jawaban benar:** PASS / kosong

**Pembahasan:** Paket docker-ce berasal dari repository Docker/resmi organisasi. Tanpa repository tersedia, opsi A/B belum tentu valid; D tidak ada.

---

## 3. Batasi User yang Mengontrol Docker Daemon

### Soal 15
**Pertanyaan:** Command mana yang benar untuk mengecek dan mengelola anggota group `docker`?
A. getent group docker
B. `sudo usermod -aG docker "$ADMIN_USER"`
C. `sudo gpasswd -d nama_user docker`
D. `sudo usermod -aG docker semua_user`

**Jawaban benar:** A, B, C

**Pembahasan:** Group docker harus terbatas pada admin tepercaya. Menambahkan semua user sangat berbahaya.

---
### Soal 16
**Pertanyaan:** Mengapa anggota group `docker` harus dibatasi?
A. Akses Docker daemon dapat setara akses root pada host
B. `User docker bisa menjalankan container yang memount filesystem host`
C. Docker group aman untuk semua user karena bukan sudo
D. `Docker socket tidak berpengaruh pada host`

**Jawaban benar:** A, B

**Pembahasan:** Akses Docker daemon dapat digunakan untuk privilege escalation ke host.

---
### Soal 17
**Pertanyaan:** Command manakah yang benar untuk menghapus user `budi` dari group docker?
A. `sudo gpasswd -d budi docker`
B. `sudo deluser budi docker`
C. `sudo usermod -G '' budi`
D. `sudo rm -rf /home/budi`

**Jawaban benar:** A, B

**Pembahasan:** gpasswd -d atau deluser user group bisa menghapus membership. Mengosongkan semua group atau menghapus home tidak tepat.

---
### Soal 18
**Pertanyaan:** Pilih pernyataan/command yang benar jika user biasa meminta akses Docker untuk menjalankan container lab.
A. `Tambahkan ke group docker hanya jika user dipercaya seperti admin host`
B. `Pertimbangkan rootless mode atau workflow terbatas jika sesuai`
C. `Langsung `usermod -aG docker semua_mahasiswa``
D. Dokumentasikan alasan pemberian akses

**Jawaban benar:** A, B, D

**Pembahasan:** Akses docker group berisiko tinggi sehingga harus dibatasi dan terdokumentasi.

---

## 4. Audit Rules Docker

### Soal 19
**Pertanyaan:** Rule audit mana yang benar untuk memantau `/usr/bin/dockerd` dalam konteks CIS Docker?
A. `-w /usr/bin/dockerd -k docker`
B. `auditctl -w /usr/bin/dockerd -k docker`
C. `rm -rf /usr/bin/dockerd`
D. `chmod 777 /usr/bin/dockerd`

**Jawaban benar:** A, B

**Pembahasan:** Rule watch audit dapat ditulis ke file rules atau diterapkan dengan auditctl. Menghapus atau chmod 777 jelas salah.

---
### Soal 20
**Pertanyaan:** Rule audit mana yang benar untuk memantau `/run/containerd` dalam konteks CIS Docker?
A. `-w /run/containerd -k docker`
B. `auditctl -w /run/containerd -k docker`
C. `rm -rf /run/containerd`
D. `chmod 777 /run/containerd`

**Jawaban benar:** A, B

**Pembahasan:** Rule watch audit dapat ditulis ke file rules atau diterapkan dengan auditctl. Menghapus atau chmod 777 jelas salah.

---
### Soal 21
**Pertanyaan:** Rule audit mana yang benar untuk memantau `/var/lib/docker` dalam konteks CIS Docker?
A. `-w /var/lib/docker -k docker`
B. `auditctl -w /var/lib/docker -k docker`
C. `rm -rf /var/lib/docker`
D. `chmod 777 /var/lib/docker`

**Jawaban benar:** A, B

**Pembahasan:** Rule watch audit dapat ditulis ke file rules atau diterapkan dengan auditctl. Menghapus atau chmod 777 jelas salah.

---
### Soal 22
**Pertanyaan:** Rule audit mana yang benar untuk memantau `/etc/docker` dalam konteks CIS Docker?
A. `-w /etc/docker -k docker`
B. `auditctl -w /etc/docker -k docker`
C. `rm -rf /etc/docker`
D. `chmod 777 /etc/docker`

**Jawaban benar:** A, B

**Pembahasan:** Rule watch audit dapat ditulis ke file rules atau diterapkan dengan auditctl. Menghapus atau chmod 777 jelas salah.

---
### Soal 23
**Pertanyaan:** Rule audit mana yang benar untuk memantau `/var/run/docker.sock` dalam konteks CIS Docker?
A. `-w /var/run/docker.sock -k docker`
B. `auditctl -w /var/run/docker.sock -k docker`
C. `rm -rf /var/run/docker.sock`
D. `chmod 777 /var/run/docker.sock`

**Jawaban benar:** A, B

**Pembahasan:** Rule watch audit dapat ditulis ke file rules atau diterapkan dengan auditctl. Menghapus atau chmod 777 jelas salah.

---
### Soal 24
**Pertanyaan:** Rule audit mana yang benar untuk memantau `/etc/docker/daemon.json` dalam konteks CIS Docker?
A. `-w /etc/docker/daemon.json -k docker`
B. `auditctl -w /etc/docker/daemon.json -k docker`
C. `rm -rf /etc/docker/daemon.json`
D. `chmod 777 /etc/docker/daemon.json`

**Jawaban benar:** A, B

**Pembahasan:** Rule watch audit dapat ditulis ke file rules atau diterapkan dengan auditctl. Menghapus atau chmod 777 jelas salah.

---
### Soal 25
**Pertanyaan:** Rule audit mana yang benar untuk memantau `/etc/containerd/config.toml` dalam konteks CIS Docker?
A. `-w /etc/containerd/config.toml -k docker`
B. `auditctl -w /etc/containerd/config.toml -k docker`
C. `rm -rf /etc/containerd/config.toml`
D. `chmod 777 /etc/containerd/config.toml`

**Jawaban benar:** A, B

**Pembahasan:** Rule watch audit dapat ditulis ke file rules atau diterapkan dengan auditctl. Menghapus atau chmod 777 jelas salah.

---
### Soal 26
**Pertanyaan:** Rule audit mana yang benar untuk memantau `/usr/bin/containerd` dalam konteks CIS Docker?
A. `-w /usr/bin/containerd -k docker`
B. `auditctl -w /usr/bin/containerd -k docker`
C. `rm -rf /usr/bin/containerd`
D. `chmod 777 /usr/bin/containerd`

**Jawaban benar:** A, B

**Pembahasan:** Rule watch audit dapat ditulis ke file rules atau diterapkan dengan auditctl. Menghapus atau chmod 777 jelas salah.

---
### Soal 27
**Pertanyaan:** Rule audit mana yang benar untuk memantau `/usr/bin/runc` dalam konteks CIS Docker?
A. `-w /usr/bin/runc -k docker`
B. `auditctl -w /usr/bin/runc -k docker`
C. `rm -rf /usr/bin/runc`
D. `chmod 777 /usr/bin/runc`

**Jawaban benar:** A, B

**Pembahasan:** Rule watch audit dapat ditulis ke file rules atau diterapkan dengan auditctl. Menghapus atau chmod 777 jelas salah.

---
### Soal 28
**Pertanyaan:** Command mana yang benar untuk membuat dan memuat ulang audit rules Docker setelah file `/etc/audit/rules.d/99-docker.rules` dibuat?
A. `sudo augenrules --load`
B. `sudo systemctl restart auditd`
C. `sudo auditctl -l | grep docker`
D. `sudo auditctl -D`

**Jawaban benar:** A, B, C

**Pembahasan:** augenrules atau restart auditd memuat rules, auditctl -l memeriksa. auditctl -D menghapus rules.

---
### Soal 29
**Pertanyaan:** File mana yang wajar masuk audit rules CIS Docker?
A. `/usr/bin/dockerd`
B. `/etc/docker/daemon.json`
C. `/var/run/docker.sock`
D. `/etc/passwd.bak.random`

**Jawaban benar:** A, B, C

**Pembahasan:** Docker daemon, daemon.json, dan docker.sock adalah objek penting Docker. File random tidak spesifik CIS Docker.

---
### Soal 30
**Pertanyaan:** Jika beberapa path audit rules Docker tidak ada pada distro tertentu, tindakan yang benar adalah...
A. `Catat sebagai not applicable bila path memang tidak ada`
B. `Sesuaikan path dengan lokasi service/socket aktual`
C. Buat file palsu agar audit pass
D. `Matikan auditd karena hasilnya tidak sempurna`

**Jawaban benar:** A, B

**Pembahasan:** Beberapa path berbeda antar distro. Jangan membuat file palsu atau mematikan auditd.

---

## 5. Docker Daemon Configuration

### Soal 31
**Pertanyaan:** Command mana yang benar untuk mengecek mode/security options Docker daemon?
A. `ps -fe | grep '[d]ockerd'`
B. `docker info --format '{{ .SecurityOptions }}'`
C. `docker info`
D. `cat /etc/shadow`

**Jawaban benar:** A, B, C

**Pembahasan:** ps dan docker info membantu audit daemon/security options. Membaca shadow tidak relevan dan sensitif.

---
### Soal 32
**Pertanyaan:** Konfigurasi `/etc/docker/daemon.json` mana yang sesuai baseline rootful hardened dari modul implementasi?
A. `"icc": false`
B. `"no-new-privileges": true`
C. `"live-restore": true`
D. `"experimental": true`

**Jawaban benar:** A, B, C

**Pembahasan:** Experimental harus false pada production baseline.

---
### Soal 33
**Pertanyaan:** Konfigurasi daemon mana yang benar untuk logging dan rotasi log Docker?
A. `"log-driver": "json-file"`
B. `"log-opts": { "max-size": "10m", "max-file": "5" }`
C. `"log-level": "info"`
D. `"log-driver": "none"`

**Jawaban benar:** A, B, C

**Pembahasan:** json-file dengan log-opts dan log-level info sesuai baseline. log-driver none menghilangkan log container.

---
### Soal 34
**Pertanyaan:** Konfigurasi daemon mana yang benar untuk storage driver dan iptables?
A. `"storage-driver": "overlay2"`
B. `"iptables": true`
C. `"storage-driver": "aufs"`
D. `"iptables": false`

**Jawaban benar:** A, B

**Pembahasan:** overlay2 modern dan Docker tetap mengelola iptables. aufs deprecated; iptables false dapat merusak network jika tidak dirancang matang.

---
### Soal 35
**Pertanyaan:** Command mana yang benar untuk validasi JSON dan restart Docker setelah mengubah `/etc/docker/daemon.json`?
A. `jq . /etc/docker/daemon.json`
B. `sudo systemctl daemon-reload`
C. `sudo systemctl restart docker`
D. `sudo rm /etc/docker/daemon.json`

**Jawaban benar:** A, B, C

**Pembahasan:** Validasi JSON sebelum restart, reload systemd, lalu restart Docker.

---
### Soal 36
**Pertanyaan:** Command mana yang benar untuk memeriksa live-restore dan experimental feature?
A. `docker info --format 'LiveRestore={{ .LiveRestoreEnabled }}'`
B. `docker version --format 'Server experimental={{ .Server.Experimental }}'`
C. `docker info --format 'LoggingDriver={{ .LoggingDriver }}'`
D. `docker experimental enable production`

**Jawaban benar:** A, B, C

**Pembahasan:** A/B/C adalah audit. Opsi D bukan command hardening Docker yang benar.

---
### Soal 37
**Pertanyaan:** Apa konfigurasi yang tepat untuk default ulimits pada daemon?
A. `"default-ulimits" dapat mengatur nofile dan nproc`
B. `nofile Hard/Soft dapat dibatasi, misalnya 65535`
C. nproc dapat dibatasi, misalnya Hard 4096 Soft 2048
D. ulimit harus dihapus agar container bebas memakai resource

**Jawaban benar:** A, B, C

**Pembahasan:** Default ulimits mencegah abuse resource. Menghapus semua batas tidak sesuai hardening.

---
### Soal 38
**Pertanyaan:** Command mana yang benar untuk mengecek insecure registries?
A. `docker info | sed -n '/Insecure Registries:/,/Live Restore Enabled:/p'`
B. `docker info`
C. `grep -R 'insecure-registries' /etc/docker/daemon.json 2>/dev/null || true`
D. `docker registry --allow-insecure all`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit insecure registries lewat docker info atau daemon.json. Opsi D bukan hardening dan membuka risiko.

---
### Soal 39
**Pertanyaan:** Jika remote Docker API tidak diperlukan, command/cek mana yang benar?
A. `ss -lntp | grep -E 'dockerd|2375|2376' || true`
B. `ps -ef | grep '[d]ockerd'`
C. `Pastikan tidak ada `-H tcp://0.0.0.0:2375``
D. Buka TCP 2375 tanpa TLS agar admin mudah

**Jawaban benar:** A, B, C

**Pembahasan:** Remote API sebaiknya tidak diekspos. Port 2375 plaintext sangat berbahaya.

---
### Soal 40
**Pertanyaan:** Jika remote Docker API wajib digunakan, opsi parameter mana yang benar?
A. `--tlsverify`
B. `--tlscacert=/path/ca.pem`
C. `--tlscert=/path/server-cert.pem`
D. `--tlskey=/path/server-key.pem`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Remote API wajib memakai TLS mutual authentication dan firewall ketat.

---
### Soal 41
**Pertanyaan:** Command mana yang benar untuk membuat Docker daemon mendengar di semua IP pada port 2375 tanpa TLS sesuai CIS?
A. `dockerd -H tcp://0.0.0.0:2375`
B. `echo '{"hosts":["tcp://0.0.0.0:2375"]}' > /etc/docker/daemon.json`
C. `ufw allow 2375/tcp`
D. `systemctl restart docker`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: Docker remote API tanpa TLS di 2375 tidak sesuai hardening CIS.

---
### Soal 42
**Pertanyaan:** Manakah alasan benar memilih rootless mode?
A. Mengurangi dampak root-level compromise
B. Cocok jika workload kompatibel dengan batasan rootless
C. Selalu cocok untuk semua workload tanpa uji
D. Mengurangi kebutuhan membatasi group docker

**Jawaban benar:** A, B

**Pembahasan:** Rootless kuat tetapi tidak universal dan tetap perlu kontrol akses.

---
### Soal 43
**Pertanyaan:** Command audit mana yang benar untuk rootless mode?
A. `ps -fe | grep '[d]ockerd'`
B. `docker info --format '{{ .SecurityOptions }}'`
C. `loginctl enable-linger <user> jika mengikuti prosedur rootless resmi`
D. `chmod 777 /var/run/docker.sock`

**Jawaban benar:** A, B, C

**Pembahasan:** Rootless mengikuti prosedur resmi dan path berbeda. chmod 777 docker.sock salah.

---

## 6. Permission dan Ownership File Docker

### Soal 44
**Pertanyaan:** Command mana yang benar untuk mengamankan `/etc/docker` dengan ownership/mode baseline `root:root 755`?
A. `sudo chown root:root /etc/docker`
B. `sudo chmod 755 /etc/docker`
C. `stat -c '%U:%G %a %n' /etc/docker`
D. `sudo chmod 777 /etc/docker`

**Jawaban benar:** A, B, C

**Pembahasan:** chown/chmod sesuai baseline dan stat memvalidasi. chmod 777 terlalu permisif.

---
### Soal 45
**Pertanyaan:** Command mana yang benar untuk mengamankan `/etc/docker/daemon.json` dengan ownership/mode baseline `root:root 644`?
A. `sudo chown root:root /etc/docker/daemon.json`
B. `sudo chmod 644 /etc/docker/daemon.json`
C. `stat -c '%U:%G %a %n' /etc/docker/daemon.json`
D. `sudo chmod 777 /etc/docker/daemon.json`

**Jawaban benar:** A, B, C

**Pembahasan:** chown/chmod sesuai baseline dan stat memvalidasi. chmod 777 terlalu permisif.

---
### Soal 46
**Pertanyaan:** Command mana yang benar untuk mengamankan `/var/run/docker.sock` dengan ownership/mode baseline `root:docker 660`?
A. `sudo chown root:docker /var/run/docker.sock`
B. `sudo chmod 660 /var/run/docker.sock`
C. `stat -c '%U:%G %a %n' /var/run/docker.sock`
D. `sudo chmod 777 /var/run/docker.sock`

**Jawaban benar:** A, B, C

**Pembahasan:** chown/chmod sesuai baseline dan stat memvalidasi. chmod 777 terlalu permisif.

---
### Soal 47
**Pertanyaan:** Command mana yang benar untuk menemukan path unit file `docker.service` sebelum memperbaiki permission?
A. `systemctl show -p FragmentPath docker.service`
B. `DOCKER_SERVICE_PATH="$(systemctl show -p FragmentPath --value docker.service)"`
C. `stat -c '%U:%G %a %n' "$DOCKER_SERVICE_PATH"`
D. `find / -name docker.service -delete`

**Jawaban benar:** A, B, C

**Pembahasan:** Gunakan systemctl show untuk path aktual. Menghapus unit file salah.

---
### Soal 48
**Pertanyaan:** Command mana yang benar untuk mengamankan `docker.service` jika path ditemukan?
A. `sudo chown root:root "$DOCKER_SERVICE_PATH"`
B. `sudo chmod 644 "$DOCKER_SERVICE_PATH"`
C. `stat -c '%U:%G %a %n' "$DOCKER_SERVICE_PATH"`
D. `sudo chown docker:docker "$DOCKER_SERVICE_PATH"`

**Jawaban benar:** A, B, C

**Pembahasan:** Systemd unit Docker harus dikuasai root dan tidak writable oleh group docker.

---
### Soal 49
**Pertanyaan:** Command mana yang benar untuk certificate publik registry di `/etc/docker/certs.d`?
A. `sudo find /etc/docker/certs.d -type f -exec chown root:root {} \;`
B. `sudo find /etc/docker/certs.d -type f -exec chmod 0444 {} \;`
C. `sudo find /etc/docker/certs.d -type f -exec stat -c '%U:%G %a %n' {} \;`
D. `sudo find /etc/docker/certs.d -type f -exec chmod 0777 {} \;`

**Jawaban benar:** A, B, C

**Pembahasan:** Certificate publik boleh read-only; tidak boleh writable semua user.

---
### Soal 50
**Pertanyaan:** Command mana yang benar untuk private key Docker TLS?
A. `sudo chown root:root /path/to/server-key.pem`
B. `sudo chmod 400 /path/to/server-key.pem`
C. `sudo chmod 644 /path/to/server-key.pem`
D. `sudo cp /path/to/server-key.pem /var/www/html/`

**Jawaban benar:** A, B

**Pembahasan:** Private key harus sangat ketat dan tidak boleh dipublikasikan.

---
### Soal 51
**Pertanyaan:** Command mana yang benar untuk membuat semua file Docker dapat diubah semua user demi kemudahan development sesuai CIS?
A. `sudo chmod -R 777 /etc/docker`
B. `sudo chmod 777 /var/run/docker.sock`
C. `sudo chown -R user:user /etc/docker`
D. `sudo chmod 666 /etc/docker/daemon.json`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: semua opsi melemahkan permission objek sensitif Docker.

---

## 7. Dockerfile dan Image Hardening

### Soal 52
**Pertanyaan:** Instruksi Dockerfile mana yang benar untuk menjalankan aplikasi sebagai non-root user?
A. `RUN groupadd -r appuser && useradd -r -g appuser -d /app -s /usr/sbin/nologin appuser`
B. `COPY --chown=appuser:appuser ./app /app`
C. `USER appuser`
D. `USER root`

**Jawaban benar:** A, B, C

**Pembahasan:** Buat user khusus, set ownership, lalu USER appuser. USER root bukan hardening.

---
### Soal 53
**Pertanyaan:** Praktik Dockerfile mana yang benar untuk image minimal dan bersih?
A. Gunakan base image minimal seperti debian:stable-slim jika sesuai
B. `Install paket dengan --no-install-recommends`
C. `Hapus cache APT dengan `rm -rf /var/lib/apt/lists/*``
D. Install semua tools debugging production secara permanen

**Jawaban benar:** A, B, C

**Pembahasan:** Image minimal mengurangi attack surface. Tools debugging harus dipertimbangkan terpisah.

---
### Soal 54
**Pertanyaan:** Instruksi mana yang benar untuk Dockerfile bila hanya menyalin file lokal aplikasi?
A. `COPY --chown=appuser:appuser ./app /app`
B. `ADD https://example.com/script.sh /app/script.sh untuk semua kasus`
C. `COPY ./app /app`
D. `RUN curl http://unknown.local/install.sh | sh`

**Jawaban benar:** A, C

**Pembahasan:** COPY lebih eksplisit untuk file lokal. ADD URL dan pipe curl ke shell berisiko.

---
### Soal 55
**Pertanyaan:** Command/Instruksi mana yang benar untuk mengecek dan mengurangi SUID/SGID dalam image?
A. `RUN find / -perm /6000 -type f -exec chmod a-s {} + 2>/dev/null || true`
B. `docker run --rm appku:1.0.0 id`
C. `find / -perm /6000 -type f`
D. `chmod -R u+s /usr/bin`

**Jawaban benar:** A, B, C

**Pembahasan:** SUID/SGID dapat diaudit/dikurangi. Menambahkan SUID massal sangat berbahaya.

---
### Soal 56
**Pertanyaan:** Manakah contoh yang salah karena menyimpan secret di Dockerfile?
A. `ENV DB_PASSWORD=rahasia`
B. `RUN echo "token-api" > /root/token.txt`
C. `ARG SECRET_TOKEN=rahasia lalu dipakai dalam layer build`
D. `USER appuser`

**Jawaban benar:** A, B, C

**Pembahasan:** Secrets dapat tersimpan dalam image layer/history. USER appuser justru baik.

---
### Soal 57
**Pertanyaan:** Command mana yang benar untuk mendeteksi kemungkinan secret yang sudah masuk image history?
A. `docker history nama-image:tag`
B. `docker inspect nama-image:tag`
C. `grep -R password Dockerfile .`
D. `docker run --privileged nama-image:tag`

**Jawaban benar:** A, B, C

**Pembahasan:** history/inspect/grep membantu audit. Menjalankan privileged tidak membantu dan berbahaya.

---
### Soal 58
**Pertanyaan:** Command mana yang benar untuk build, cek user efektif, dan scan image?
A. `docker build -t appku:1.0.0 .`
B. `docker run --rm appku:1.0.0 id`
C. `trivy image appku:1.0.0`
D. `docker scan --disable-security appku:1.0.0`

**Jawaban benar:** A, B, C

**Pembahasan:** Build, run id, dan scan image sesuai praktik. Opsi D bukan hardening.

---
### Soal 59
**Pertanyaan:** Command mana yang benar untuk mengaktifkan Docker Content Trust saat pull image bila organisasi memakai signing?
A. `export DOCKER_CONTENT_TRUST=1`
B. `docker pull nama-image:tag`
C. `unset DOCKER_CONTENT_TRUST agar semua image bebas`
D. `docker pull --allow-unsigned-only nama-image:tag`

**Jawaban benar:** A, B

**Pembahasan:** Content trust membantu validasi image/artifact. Allow unsigned tidak sesuai supply chain hardening.

---
### Soal 60
**Pertanyaan:** Instruksi mana yang benar untuk HEALTHCHECK pada Dockerfile?
A. `HEALTHCHECK --interval=30s --timeout=5s --retries=3 CMD curl -f http://127.0.0.1:8080/health || exit 1`
B. `HEALTHCHECK NONE untuk semua production`
C. `CMD ["./start.sh"]`
D. `EXPOSE 8080`

**Jawaban benar:** A, C, D

**Pembahasan:** HEALTHCHECK membantu availability/monitoring; CMD dan EXPOSE valid. Menghapus semua HEALTHCHECK tidak selalu benar.

---
### Soal 61
**Pertanyaan:** Manakah praktik source image yang benar?
A. Gunakan base image resmi atau internal registry terpercaya
B. `Pin versi/tag bila memungkinkan`
C. Scan image sebelum deploy
D. Pull image acak dari internet karena ukurannya kecil

**Jawaban benar:** A, B, C

**Pembahasan:** Sumber image harus tepercaya dan divalidasi.

---
### Soal 62
**Pertanyaan:** Command mana yang benar untuk menyimpan password database ke image agar mudah dipakai semua environment?
A. `ENV DB_PASSWORD=rahasia`
B. `RUN echo rahasia > /app/.env`
C. `docker commit container-dengan-secret image:prod`
D. `Gunakan secret manager/runtime secret, bukan image layer`

**Jawaban benar:** D

**Pembahasan:** Secret tidak boleh disimpan di Dockerfile/layer/image.

---

## 8. Network Docker

### Soal 63
**Pertanyaan:** Command mana yang benar untuk membuat custom bridge network per aplikasi?
A. `docker network create --driver bridge app_net`
B. `docker network inspect app_net`
C. `docker run -d --name appku --network app_net appku:1.0.0`
D. `docker run -d --network host appku:1.0.0`

**Jawaban benar:** A, B, C

**Pembahasan:** Custom network lebih baik daripada default bridge/host network. Host network melemahkan isolasi.

---
### Soal 64
**Pertanyaan:** Command mana yang benar untuk audit container pada default bridge?
A. `docker network inspect bridge --format '{{json .Containers}}' | jq .`
B. `docker network inspect bridge`
C. `docker ps --format 'table {{.Names}}	{{.Networks}}'`
D. `docker network rm bridge`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit default bridge benar. Menghapus default bridge bukan langkah audit aman umum.

---
### Soal 65
**Pertanyaan:** Command mapping port mana yang sesuai prinsip bind ke interface tertentu?
A. -p 127.0.0.1:8080:8080
B. `--publish 127.0.0.1:8080:8080`
C. -p 0.0.0.0:8080:8080 tanpa kebutuhan publik
D. `--network host`

**Jawaban benar:** A, B

**Pembahasan:** Bind localhost membatasi exposure. 0.0.0.0 dan host network perlu alasan/exception.

---
### Soal 66
**Pertanyaan:** Command mana yang benar untuk mengaktifkan inter-container communication bebas pada default bridge sesuai hardening ketat?
A. `Set `"icc": true` di daemon.json`
B. `Gunakan semua container di default bridge`
C. `Gunakan `--network host` agar mudah`
D. Buka semua port container ke 0.0.0.0

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: hardening mengarah ke segmentasi, custom network, dan `icc=false` jika kompatibel.

---

## 9. Runtime Hardening

### Soal 67
**Pertanyaan:** Opsi `docker run` mana yang benar untuk menjalankan container sebagai user non-root?
A. `--user 1000:1000`
B. `---user 1000:1000`
C. `--privileged`
D. `--security-opt seccomp=unconfined`

**Jawaban benar:** A

**Pembahasan:** Opsi `--user 1000:1000` sesuai baseline runtime hardening. Opsi lain salah atau melemahkan isolasi.

---
### Soal 68
**Pertanyaan:** Opsi `docker run` mana yang benar untuk membuat root filesystem container read-only?
A. `--read-only`
B. `---read-only`
C. `--privileged`
D. `--security-opt seccomp=unconfined`

**Jawaban benar:** A

**Pembahasan:** Opsi `--read-only` sesuai baseline runtime hardening. Opsi lain salah atau melemahkan isolasi.

---
### Soal 69
**Pertanyaan:** Opsi `docker run` mana yang benar untuk memberi tmpfs aman untuk `/tmp`?
A. `--tmpfs /tmp:rw,noexec,nosuid,nodev,size=64m`
B. `---tmpfs /tmp:rw,noexec,nosuid,nodev,size=64m`
C. `--privileged`
D. `--security-opt seccomp=unconfined`

**Jawaban benar:** A

**Pembahasan:** Opsi `--tmpfs /tmp:rw,noexec,nosuid,nodev,size=64m` sesuai baseline runtime hardening. Opsi lain salah atau melemahkan isolasi.

---
### Soal 70
**Pertanyaan:** Opsi `docker run` mana yang benar untuk menurunkan semua Linux capabilities?
A. `--cap-drop ALL`
B. `---cap-drop ALL`
C. `--privileged`
D. `--security-opt seccomp=unconfined`

**Jawaban benar:** A

**Pembahasan:** Opsi `--cap-drop ALL` sesuai baseline runtime hardening. Opsi lain salah atau melemahkan isolasi.

---
### Soal 71
**Pertanyaan:** Opsi `docker run` mana yang benar untuk mencegah privilege tambahan?
A. `--security-opt no-new-privileges:true`
B. `---security-opt no-new-privileges:true`
C. `--privileged`
D. `--security-opt seccomp=unconfined`

**Jawaban benar:** A

**Pembahasan:** Opsi `--security-opt no-new-privileges:true` sesuai baseline runtime hardening. Opsi lain salah atau melemahkan isolasi.

---
### Soal 72
**Pertanyaan:** Opsi `docker run` mana yang benar untuk memakai profile AppArmor default?
A. `--security-opt apparmor=docker-default`
B. `---security-opt apparmor=docker-default`
C. `--privileged`
D. `--security-opt seccomp=unconfined`

**Jawaban benar:** A

**Pembahasan:** Opsi `--security-opt apparmor=docker-default` sesuai baseline runtime hardening. Opsi lain salah atau melemahkan isolasi.

---
### Soal 73
**Pertanyaan:** Opsi `docker run` mana yang benar untuk membatasi jumlah proses?
A. `--pids-limit 256`
B. `---pids-limit 256`
C. `--privileged`
D. `--security-opt seccomp=unconfined`

**Jawaban benar:** A

**Pembahasan:** Opsi `--pids-limit 256` sesuai baseline runtime hardening. Opsi lain salah atau melemahkan isolasi.

---
### Soal 74
**Pertanyaan:** Opsi `docker run` mana yang benar untuk membatasi memory?
A. `--memory 512m`
B. `---memory 512m`
C. `--privileged`
D. `--security-opt seccomp=unconfined`

**Jawaban benar:** A

**Pembahasan:** Opsi `--memory 512m` sesuai baseline runtime hardening. Opsi lain salah atau melemahkan isolasi.

---
### Soal 75
**Pertanyaan:** Opsi `docker run` mana yang benar untuk membatasi CPU?
A. `--cpus 1.0`
B. `---cpus 1.0`
C. `--privileged`
D. `--security-opt seccomp=unconfined`

**Jawaban benar:** A

**Pembahasan:** Opsi `--cpus 1.0` sesuai baseline runtime hardening. Opsi lain salah atau melemahkan isolasi.

---
### Soal 76
**Pertanyaan:** Opsi `docker run` mana yang benar untuk membatasi restart loop?
A. `--restart on-failure:5`
B. `---restart on-failure:5`
C. `--privileged`
D. `--security-opt seccomp=unconfined`

**Jawaban benar:** A

**Pembahasan:** Opsi `--restart on-failure:5` sesuai baseline runtime hardening. Opsi lain salah atau melemahkan isolasi.

---
### Soal 77
**Pertanyaan:** Command `docker run` mana yang paling sesuai baseline aman dari modul implementasi?
A. `docker run -d --name appku --network app_net --user 1000:1000 --read-only --tmpfs /tmp:rw,noexec,nosuid,nodev,size=64m --cap-drop ALL --security-opt no-new-privileges:true --pids-limit 256 --memory 512m --cpus 1.0 -p 127.0.0.1:8080:8080 appku:1.0.0`
B. `docker run -d --privileged -v /:/host --network host appku:1.0.0`
C. `docker run -d -v /var/run/docker.sock:/var/run/docker.sock appku:1.0.0`
D. `docker run -d --pid host --ipc host --uts host appku:1.0.0`

**Jawaban benar:** A

**Pembahasan:** Opsi A menerapkan banyak kontrol runtime; B/C/D melemahkan isolasi host.

---
### Soal 78
**Pertanyaan:** Runtime option mana yang harus dihindari kecuali ada exception sangat kuat?
A. `--privileged`
B. `-v /:/host`
C. `-v /var/run/docker.sock:/var/run/docker.sock`
D. `--network host`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Keempat opsi ini memperluas akses container ke host atau melemahkan isolasi.

---
### Soal 79
**Pertanyaan:** Command mana yang benar untuk audit container berjalan dari sisi privileged/network/namespace/capabilities?
A. `docker ps --quiet | xargs -r docker inspect --format '{{.Name}} Privileged={{.HostConfig.Privileged}} Network={{.HostConfig.NetworkMode}} PidMode={{.HostConfig.PidMode}} IpcMode={{.HostConfig.IpcMode}} CapAdd={{.HostConfig.CapAdd}} CapDrop={{.HostConfig.CapDrop}}'`
B. `docker ps --quiet | xargs -r docker inspect --format '{{.Name}} Readonly={{.HostConfig.ReadonlyRootfs}} Binds={{.HostConfig.Binds}} SecurityOpt={{.HostConfig.SecurityOpt}}'`
C. `docker ps --quiet | xargs -r docker inspect --format '{{.Name}} {{.HostConfig.Binds}}' | grep docker.sock || true`
D. `docker ps -q | xargs docker rm -f`

**Jawaban benar:** A, B, C

**Pembahasan:** Inspect dipakai untuk audit. Menghapus paksa container bukan audit.

---
### Soal 80
**Pertanyaan:** Command mana yang benar untuk masuk ke container production sebagai root privileged sesuai CIS?
A. `docker exec --privileged appku sh`
B. `docker exec -u root appku sh`
C. `docker exec --user 0 --privileged appku bash`
D. `docker exec --cap-add SYS_ADMIN appku bash`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: akses root/privileged dalam container production harus dihindari kecuali break-glass/exception terdokumentasi.

---
### Soal 81
**Pertanyaan:** Manakah opsi yang benar untuk resource limit container?
A. `--memory 512m`
B. `--memory-swap 512m`
C. `--cpus 1.0`
D. `--ulimit nofile=1024:2048`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua opsi membatasi resource agar container tidak menghabiskan host.

---
### Soal 82
**Pertanyaan:** Manakah opsi yang benar untuk memberi capability minimal jika aplikasi memang butuh satu capability tertentu?
A. `--cap-drop ALL lalu --cap-add NET_BIND_SERVICE jika benar-benar dibutuhkan`
B. `--cap-add ALL`
C. `--privileged`
D. `--cap-add SYS_ADMIN untuk semua container`

**Jawaban benar:** A

**Pembahasan:** Strategi aman adalah drop all lalu add hanya capability spesifik yang dibutuhkan.

---
### Soal 83
**Pertanyaan:** Manakah command yang benar untuk menjalankan container dengan seccomp default Docker?
A. `docker run appku:1.0.0`
B. `docker run --security-opt seccomp=/path/seccomp-profile.json appku:1.0.0 jika memakai custom profile yang diuji`
C. `docker run --security-opt seccomp=unconfined appku:1.0.0`
D. `docker run --privileged appku:1.0.0`

**Jawaban benar:** A, B

**Pembahasan:** Default seccomp aktif secara default; custom profile boleh jika diuji. Jangan unconfined/privileged.

---
### Soal 84
**Pertanyaan:** Command mana yang benar untuk mengecek security options Docker terkait AppArmor/seccomp/userns?
A. `docker info --format '{{ .SecurityOptions }}'`
B. `sudo aa-status`
C. `sestatus jika host RHEL family memakai SELinux`
D. `chmod 777 /etc/apparmor.d`

**Jawaban benar:** A, B, C

**Pembahasan:** Security options dan MAC status perlu diaudit; chmod 777 salah.

---

## 10. Docker Compose Hardening

### Soal 85
**Pertanyaan:** Field Compose mana yang sesuai baseline runtime hardening?
A. `user: "1000:1000"`
B. read_only: true
C.
```text
cap_drop:
  - ALL
```
D. privileged: true

**Jawaban benar:** A, B, C

**Pembahasan:** Compose dapat menerapkan user non-root, read-only, dan drop capabilities. privileged melemahkan isolasi.

---
### Soal 86
**Pertanyaan:** Field Compose mana yang benar untuk tmpfs aman dan security_opt?
A.
```text
tmpfs:
  - /tmp:rw,noexec,nosuid,nodev,size=64m
```
B.
```text
security_opt:
  - no-new-privileges:true
```
C.
```text
security_opt:
  - apparmor=docker-default
```
D.
```text
security_opt:
  - seccomp=unconfined
```

**Jawaban benar:** A, B, C

**Pembahasan:** tmpfs aman dan no-new-privileges/AppArmor benar. seccomp unconfined tidak sesuai hardening.

---
### Soal 87
**Pertanyaan:** Field Compose mana yang benar untuk membatasi resource dan restart loop?
A. `pids_limit: 256`
B. mem_limit: 512m
C. cpus: 1.0
D. `restart: "on-failure:5"`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua field sesuai contoh Compose hardening.

---
### Soal 88
**Pertanyaan:** Mapping port Compose mana yang paling aman jika aplikasi hanya perlu diakses reverse proxy lokal?
A. `"127.0.0.1:8080:8080"`
B. `"0.0.0.0:8080:8080" tanpa kebutuhan publik`
C. `"8080:8080" tanpa pembatasan interface`
D. `network_mode: "host"`

**Jawaban benar:** A

**Pembahasan:** Bind ke localhost membatasi exposure. Opsi lain lebih luas.

---
### Soal 89
**Pertanyaan:** Compose manakah yang benar untuk semua container production sesuai CIS agar mudah debugging?
A. privileged: true
B. network_mode: host
C. `pid: host`
D.
```text
volumes:
  - /var/run/docker.sock:/var/run/docker.sock
```

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: semua field melemahkan isolasi dan harus dihindari kecuali exception sangat kuat.

---

## 11. Logging dan Operasi Keamanan

### Soal 90
**Pertanyaan:** Command mana yang benar untuk mengecek logging driver Docker?
A. `docker info --format '{{ .LoggingDriver }}'`
B. `docker info`
C. `grep -R 'log-driver' /etc/docker/daemon.json 2>/dev/null || true`
D. `docker logs --disable all`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit logging driver melalui docker info atau daemon.json. Opsi D tidak benar.

---
### Soal 91
**Pertanyaan:** Konfigurasi daemon mana yang benar untuk container logging rotation dengan json-file?
A. `"log-driver": "json-file"`
B. `"log-opts": { "max-size": "10m", "max-file": "5" }`
C. `"log-driver": "none"`
D. `"max-size": "infinite"`

**Jawaban benar:** A, B

**Pembahasan:** json-file dengan max-size/max-file mencegah log tumbuh tak terbatas.

---
### Soal 92
**Pertanyaan:** Command mana yang benar untuk inventory Docker resources?
A. `docker images`
B. `docker ps -a`
C. `docker volume ls`
D. `docker network ls`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Inventory image/container/volume/network penting untuk mencegah sprawl.

---
### Soal 93
**Pertanyaan:** Command prune mana yang valid tetapi harus digunakan hati-hati setelah verifikasi?
A. `docker container prune`
B. `docker image prune`
C. `docker network prune`
D. `docker volume prune`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Prune valid, tetapi volume/image bisa berisi data penting sehingga harus diverifikasi.

---
### Soal 94
**Pertanyaan:** Command mana yang lebih agresif dan harus sangat hati-hati karena bisa menghapus image yang belum dipakai?
A. `docker system prune -a`
B. `docker images`
C. `docker ps -a`
D. `docker info`

**Jawaban benar:** A

**Pembahasan:** docker system prune -a agresif dan dapat menghapus image unused yang masih dibutuhkan.

---
### Soal 95
**Pertanyaan:** Command mana yang benar untuk memberi label owner/environment/app saat menjalankan container?
A. `docker run -d --label owner="perpustakaan" --label env="production" --label app="himmah" appku:1.0.0`
B. `docker ps --format 'table {{.Names}}	{{.Image}}	{{.Labels}}'`
C. `docker run --label env=production appku:1.0.0`
D. `docker run --privileged --label secure=true appku:1.0.0`

**Jawaban benar:** A, B, C

**Pembahasan:** Label membantu inventory. Label secure=true tidak mengimbangi risiko --privileged.

---
### Soal 96
**Pertanyaan:** Command mana yang benar untuk audit cepat host/daemon Docker?
A. `mountpoint -- "$(docker info -f '{{ .DockerRootDir }}')"`
B. getent group docker
C. `docker info --format 'SecurityOptions={{ .SecurityOptions }}'`
D. `docker info --format 'LiveRestore={{ .LiveRestoreEnabled }}'`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua termasuk audit cepat host/daemon.

---
### Soal 97
**Pertanyaan:** Command mana yang benar untuk audit file permission Docker secara cepat?
A. `stat -c '%U:%G %a %n' /etc/docker`
B. `[ -f /etc/docker/daemon.json ] && stat -c '%U:%G %a %n' /etc/docker/daemon.json`
C. `stat -c '%U:%G %a %n' /var/run/docker.sock`
D. `systemctl show -p FragmentPath docker.service`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua memeriksa objek penting Docker.

---
### Soal 98
**Pertanyaan:** Command mana yang benar untuk mematikan logging container production agar disk tidak penuh sesuai CIS?
A. Set `log-driver` menjadi `none` untuk semua container
B. `Hapus `/var/lib/docker/containers/*/*-json.log` tanpa restart`
C. `Matikan rsyslog/auditd`
D. Gunakan log rotation atau centralized logging

**Jawaban benar:** D

**Pembahasan:** Solusi benar adalah rotasi/centralized logging, bukan mematikan log.

---

## 12. Docker Swarm Configuration

### Soal 99
**Pertanyaan:** Jika Swarm tidak digunakan, command mana yang benar untuk audit dan meninggalkan Swarm?
A. `docker info --format '{{ .Swarm }}'`
B. `docker swarm leave --force`
C. `docker node ls`
D. `docker swarm init --advertise-addr 0.0.0.0`

**Jawaban benar:** A, B

**Pembahasan:** Jika Swarm tidak digunakan, audit status dan leave. node ls hanya untuk manager, init tidak diperlukan.

---
### Soal 100
**Pertanyaan:** Jika Swarm dipakai, command mana yang benar untuk bind Swarm ke interface tertentu?
A. `docker swarm init --advertise-addr 10.10.10.10 --listen-addr 10.10.10.10:2377`
B. `docker swarm init --advertise-addr 0.0.0.0 --listen-addr 0.0.0.0:2377 tanpa alasan`
C. `docker swarm update --autolock=true`
D. `docker swarm leave --force setelah init production`

**Jawaban benar:** A, C

**Pembahasan:** Bind ke IP tertentu dan autolock benar untuk Swarm. 0.0.0.0 luas; leave bukan setup production.

---
### Soal 101
**Pertanyaan:** Command mana yang benar untuk overlay network terenkripsi di Swarm?
A. `docker network create --driver overlay --opt encrypted secure_overlay`
B. `docker network ls --filter driver=overlay --quiet | xargs -r docker network inspect --format '{{.Name}} {{.Options}}'`
C. `docker network create --driver bridge --opt encrypted secure_overlay`
D. `docker network create --driver overlay insecure_overlay`

**Jawaban benar:** A, B

**Pembahasan:** Overlay encrypted menggunakan driver overlay dan opt encrypted; audit dengan inspect.

---
### Soal 102
**Pertanyaan:** Command mana yang benar untuk Docker secrets pada Swarm?
A. `printf 'password-rahasia' | docker secret create db_password -`
B. `docker service create --name appku --secret db_password --network secure_overlay appku:1.0.0`
C. `ENV DB_PASSWORD=password-rahasia di Dockerfile`
D. `docker config create db_password password.txt untuk password rahasia`

**Jawaban benar:** A, B

**Pembahasan:** Docker secret dirancang untuk secret. ENV Dockerfile dan docker config untuk password bukan pilihan aman.

---
### Soal 103
**Pertanyaan:** Command mana yang benar untuk auto-lock dan rotasi unlock key Swarm manager?
A. `docker swarm update --autolock=true`
B. `docker swarm unlock-key`
C. `docker swarm unlock-key --rotate`
D. `docker swarm update --autolock=false untuk hardening`

**Jawaban benar:** A, B, C

**Pembahasan:** Autolock dan rotasi unlock key melindungi key material manager.

---
### Soal 104
**Pertanyaan:** Command mana yang benar untuk rotasi CA dan mengatur masa berlaku certificate node Swarm?
A. `docker swarm ca --rotate`
B. `docker swarm update --cert-expiry 720h`
C. `docker swarm cert --disable`
D. `rm -rf /var/lib/docker/swarm/certificates`

**Jawaban benar:** A, B

**Pembahasan:** CA rotation dan cert expiry valid. Menghapus cert manual salah.

---
### Soal 105
**Pertanyaan:** Manakah prinsip benar untuk Swarm management plane dan data plane?
A. `Pisahkan interface/VLAN/subnet manajemen dan data jika memungkinkan`
B. Batasi port manajemen hanya antar node sah
C. Expose management API ke internet agar mudah
D. Gunakan firewall untuk node yang sah

**Jawaban benar:** A, B, D

**Pembahasan:** Management plane harus dibatasi dan dipisahkan, bukan diekspos publik.

---
### Soal 106
**Pertanyaan:** Jika Swarm tidak digunakan pada host lab, command mana yang benar untuk mengaktifkan semua fitur Swarm agar CIS lengkap?
A. `docker swarm init`
B. `docker network create --driver overlay public_overlay`
C. `docker secret create semua_secret -`
D. `docker swarm update --autolock=false`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: Swarm controls hanya relevan jika Swarm dipakai. Jangan aktifkan Swarm tanpa kebutuhan.

---

## 13. Soal Analisis Praktik Campuran

### Soal 107
**Pertanyaan:** Sebuah container menjalankan web app. Opsi mana yang paling sesuai CIS untuk exposure lokal melalui reverse proxy host?
A. `--network app_net`
B. -p 127.0.0.1:8080:8080
C. `--read-only --tmpfs /tmp:rw,noexec,nosuid,nodev,size=64m`
D. `--privileged`

**Jawaban benar:** A, B, C

**Pembahasan:** Custom network, bind localhost, dan read-only/tmpfs bagus. privileged tidak.

---
### Soal 108
**Pertanyaan:** Aplikasi membutuhkan menulis file sementara. Pilihan mana yang benar jika root filesystem dibuat read-only?
A. `Tambahkan tmpfs khusus seperti `/tmp``
B. Mount volume khusus hanya untuk path data yang diperlukan
C. Nonaktifkan read-only untuk semua container tanpa dokumentasi
D. `Gunakan `--tmpfs /tmp:rw,noexec,nosuid,nodev,size=64m``

**Jawaban benar:** A, B, D

**Pembahasan:** Read-only bisa dipertahankan dengan tmpfs/volume spesifik.

---
### Soal 109
**Pertanyaan:** Developer ingin mount `/var/run/docker.sock` ke container CI agar build Docker mudah. Jawaban CIS yang tepat adalah...
A. `Hindari karena container bisa mengontrol daemon/host`
B. Gunakan alternatif build yang lebih aman bila memungkinkan
C. Jika tetap wajib, dokumentasikan exception dan batasi environment
D. Aman karena hanya socket, bukan filesystem root

**Jawaban benar:** A, B, C

**Pembahasan:** Docker socket adalah objek sangat sensitif. Mount socket setara memberi kontrol host.

---
### Soal 110
**Pertanyaan:** Container butuh port 80 publik. Pilihan mana yang lebih sesuai hardening?
A. `Gunakan reverse proxy/load balancer host yang bind 80/443, container tetap di port non-privileged`
B. Jika map port 80 langsung, dokumentasikan exception dan batasi firewall
C. Jalankan container privileged agar bisa port 80
D. `Berikan capability/konfigurasi minimal bila benar-benar perlu`

**Jawaban benar:** A, B, D

**Pembahasan:** Gunakan reverse proxy atau exception terkontrol, bukan privileged.

---
### Soal 111
**Pertanyaan:** Command mana yang benar untuk audit container yang tidak read-only?
A. `docker ps --quiet | xargs -r docker inspect --format '{{.Name}} Readonly={{.HostConfig.ReadonlyRootfs}}'`
B. `docker inspect <container>`
C. `docker ps --format 'table {{.Names}}	{{.Image}}'`
D. `chmod -R 555 /var/lib/docker`

**Jawaban benar:** A, B, C

**Pembahasan:** Inspect dapat melihat ReadonlyRootfs. chmod massal Docker root berbahaya.

---
### Soal 112
**Pertanyaan:** Command mana yang benar untuk audit container resource limits?
A. `docker ps --quiet | xargs -r docker inspect --format '{{.Name}} Memory={{.HostConfig.Memory}} NanoCPUs={{.HostConfig.NanoCpus}} PidsLimit={{.HostConfig.PidsLimit}}'`
B. `docker stats --no-stream`
C. `docker inspect <container>`
D. ulimit -a di host saja sudah cukup untuk semua container

**Jawaban benar:** A, B, C

**Pembahasan:** Resource limit container perlu inspect/stats. Host ulimit saja tidak cukup.

---
### Soal 113
**Pertanyaan:** Command mana yang benar untuk audit apakah Docker daemon memakai user namespace remapping?
A. `docker info --format '{{ .SecurityOptions }}'`
B. `grep -R 'userns-remap' /etc/docker/daemon.json 2>/dev/null || true`
C. `docker info | grep -i userns`
D. `docker run --userns=host semua_container`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit lewat docker info dan daemon.json. --userns=host melemahkan user namespace.

---
### Soal 114
**Pertanyaan:** Opsi mana yang benar bila aplikasi rusak setelah `userns-remap`: false?
A. Rollback dari backup daemon.json sementara lalu dokumentasikan exception
B. Uji ownership volume dan compatibility di staging
C. Matikan semua hardening Docker
D. Tetap deploy tanpa uji karena CIS harus dipaksa

**Jawaban benar:** A, B

**Pembahasan:** userns-remap dapat mengganggu workload lama; rollback/exception/testing perlu.

---
### Soal 115
**Pertanyaan:** Command mana yang benar untuk mengaktifkan semua fitur experimental Docker di production sesuai CIS?
A. `Set `"experimental": true` di daemon.json`
B. `systemctl restart docker`
C. `docker version --format 'Server experimental={{ .Server.Experimental }}'`
D. Dokumentasikan bahwa experimental wajib aktif

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: experimental features seharusnya false pada production baseline kecuali exception sangat jelas.

---
### Soal 116
**Pertanyaan:** Jika organisasi memakai private registry internal, konfigurasi yang benar adalah...
A. `Gunakan TLS dengan sertifikat/CA valid`
B. `Letakkan CA di path yang benar seperti `/etc/docker/certs.d/<registry>/ca.crt` sesuai kebutuhan`
C. Tambahkan ke insecure-registries agar cepat
D. `Validasi image dengan scanning/signing bila tersedia`

**Jawaban benar:** A, B, D

**Pembahasan:** Private registry tetap harus pakai TLS dan validasi artifact, bukan insecure registry.

---
### Soal 117
**Pertanyaan:** Command mana yang benar untuk membuka Docker socket agar semua user bisa build image tanpa sudo sesuai CIS?
A. `sudo chmod 666 /var/run/docker.sock`
B. `sudo chmod 777 /var/run/docker.sock`
C. `sudo chown root:root /var/run/docker.sock && sudo chmod 666 /var/run/docker.sock`
D. Tambahkan semua user ke group docker

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: semua opsi memperluas akses Docker daemon secara berbahaya.

---
### Soal 118
**Pertanyaan:** Manakah kontrol prioritas awal CIS Docker untuk praktik lab?
A. Batasi group docker
B. `Jangan gunakan --privileged`
C. Jangan mount docker.sock ke container
D. Jalankan container sebagai non-root jika aplikasi mendukung

**Jawaban benar:** A, B, C, D

**Pembahasan:** Keempatnya termasuk prioritas implementasi awal dalam modul.

---
### Soal 119
**Pertanyaan:** Command mana yang benar untuk validasi post-hardening Docker secara ringkas?
A. `docker info --format 'SecurityOptions={{ .SecurityOptions }}'`
B. `sudo auditctl -l | grep docker`
C. `stat -c '%U:%G %a %n' /var/run/docker.sock`
D. `docker ps --quiet | xargs -r docker inspect --format '{{.Name}} Privileged={{.HostConfig.Privileged}} Readonly={{.HostConfig.ReadonlyRootfs}} Binds={{.HostConfig.Binds}}'`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua command memvalidasi sisi daemon, audit, socket, dan runtime.

---
### Soal 120
**Pertanyaan:** Manakah command yang termasuk sangat destruktif dan bukan jawaban implementasi CIS?
A. `sudo rm -rf /var/lib/docker`
B. `sudo chmod -R 777 /etc/docker`
C. `docker ps -q | xargs -r docker rm -f tanpa persetujuan`
D. `sudo auditctl -D`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Semua berisiko merusak host, permission, container, atau audit rules.

---
### Soal 121
**Pertanyaan:** Manakah kombinasi yang benar untuk container web stateless yang sehat?
A. custom network
B. non-root user
C. `read-only root filesystem + tmpfs /tmp`
D. healthcheck

**Jawaban benar:** A, B, C, D

**Pembahasan:** Kombinasi ini mendukung isolation, least privilege, dan monitoring.

---
### Soal 122
**Pertanyaan:** Manakah alasan benar menghindari default bridge untuk semua aplikasi?
A. Segmentasi antar aplikasi lebih lemah
B. Custom network memberi kontrol isolasi lebih baik
C. `Semua container default bridge selalu aman karena Docker otomatis memfilter semua komunikasi`
D. `Default bridge memudahkan lateral movement jika tidak dikendalikan`

**Jawaban benar:** A, B, D

**Pembahasan:** Default bridge tidak ideal untuk segmentasi semua workload.

---
### Soal 123
**Pertanyaan:** Pilih opsi benar untuk exception register Docker.
A. `Catat ID rekomendasi/kontrol`
B. Catat alasan bisnis dan risiko
C. `Catat mitigasi, owner, tanggal review, target eliminasi/accepted risk`
D. `Gunakan exception agar semua kontrol tidak perlu diuji`

**Jawaban benar:** A, B, C

**Pembahasan:** Exception harus terdokumentasi dan direview, bukan alasan mengabaikan semua kontrol.

---

## 14. Daemon JSON dan Validasi Detail

### Soal 124
**Pertanyaan:** Manakah potongan `daemon.json` yang benar untuk user namespace remapping default?
A. `"userns-remap": "default"`
B. `"userns-remap": false`
C. `"userns-remap": "host"`
D. `"userns-remap": "root"`

**Jawaban benar:** A

**Pembahasan:** `userns-remap: default` membuat mapping default. Host/root tidak sesuai tujuan remap.

---
### Soal 125
**Pertanyaan:** Manakah potongan `daemon.json` yang benar untuk mengurangi attack surface userland proxy?
A. `"userland-proxy": false`
B. `"userland-proxy": true`
C. `"userland-proxy": "all"`
D. `"proxy": "open"`

**Jawaban benar:** A

**Pembahasan:** Menonaktifkan userland proxy dapat mengurangi komponen tambahan jika kompatibel.

---
### Soal 126
**Pertanyaan:** Manakah potongan `daemon.json` yang benar untuk mematikan experimental features?
A. `"experimental": false`
B. `"experimental": true`
C. `"features": {"experimental": true}`
D. `"production": false`

**Jawaban benar:** A

**Pembahasan:** Experimental features sebaiknya tidak aktif di production.

---
### Soal 127
**Pertanyaan:** Command mana yang benar untuk mengecek DockerRootDir dan mountpoint-nya?
A. `docker info -f '{{ .DockerRootDir }}'`
B. `mountpoint -- "$(docker info -f '{{ .DockerRootDir }}')"`
C. `df -h "$(docker info -f '{{ .DockerRootDir }}')"`
D. `rm -rf "$(docker info -f '{{ .DockerRootDir }}')"`

**Jawaban benar:** A, B, C

**Pembahasan:** A/B/C audit storage Docker; D destruktif.

---
### Soal 128
**Pertanyaan:** Command mana yang benar untuk melihat apakah Docker menggunakan storage driver overlay2?
A. `docker info --format '{{ .Driver }}'`
B. `docker info | grep -i 'Storage Driver'`
C. `grep -R 'storage-driver' /etc/docker/daemon.json 2>/dev/null || true`
D. `docker driver enable aufs`

**Jawaban benar:** A, B, C

**Pembahasan:** Audit storage driver via docker info atau daemon.json.

---

## 15. Soal PASS / Kosong

### Soal 129
**Pertanyaan:** Pilih command yang benar untuk memperkuat container dengan memberi akses penuh ke host.
A. `docker run --privileged appku:1.0.0`
B. `docker run -v /:/host appku:1.0.0`
C. `docker run --pid host --network host appku:1.0.0`
D. `docker run -v /var/run/docker.sock:/var/run/docker.sock appku:1.0.0`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: semua opsi memberi akses host berlebihan.

---
### Soal 130
**Pertanyaan:** Pilih command yang benar untuk menghapus semua kontrol AppArmor/seccomp Docker agar aplikasi tidak error sesuai CIS.
A. `docker run --security-opt apparmor=unconfined appku:1.0.0`
B. `docker run --security-opt seccomp=unconfined appku:1.0.0`
C. `sudo systemctl stop apparmor`
D. `sudo chmod 777 /etc/apparmor.d`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: semua opsi melemahkan MAC/seccomp.

---
### Soal 131
**Pertanyaan:** Pilih command yang benar untuk menyimpan semua secrets dalam Dockerfile agar image portable sesuai CIS.
A. `ENV API_KEY=rahasia`
B. `RUN echo rahasia > /root/key.txt`
C. `ARG DB_PASSWORD=rahasia`
D. `docker commit container-dengan-secret app:prod`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: secret tidak boleh masuk Dockerfile/image layer/history.

---
### Soal 132
**Pertanyaan:** Pilih command yang benar untuk menjalankan semua container tanpa limit resource sesuai CIS.
A. `docker run --memory 0 --pids-limit -1 appku:1.0.0`
B. `docker run --ulimit nofile=999999999:999999999 appku:1.0.0`
C. `docker run --cpus 0 appku:1.0.0`
D. Hapus semua default-ulimits dari daemon karena limit mengganggu

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: hardening menghendaki resource limit yang masuk akal, bukan tanpa batas.

---
### Soal 133
**Pertanyaan:** Pilih command yang benar untuk mengizinkan insecure registry HTTP di production tanpa exception.
A. `Tambahkan `"insecure-registries": ["registry.local:5000"]``
B. Buka firewall ke registry HTTP publik
C. Matikan TLS pada registry
D. `Pull image dari registry tidak tepercaya tanpa scan`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: insecure registry menimbulkan risiko tampering/interception.

---

## 16. Prinsip CIS Docker

### Soal 134
**Pertanyaan:** Pilih prinsip yang benar dari CIS Docker Benchmark.
A. Host Linux tetap harus di-hardening
B. Docker daemon harus dikonfigurasi hati-hati karena memengaruhi semua container
C. Image dan Dockerfile harus aman sebelum runtime
D. `Container isolation adalah batas absolut sehingga host tidak penting`

**Jawaban benar:** A, B, C

**Pembahasan:** Docker bukan pengganti hardening host; semua lapisan perlu diamankan.

---
### Soal 135
**Pertanyaan:** Manakah risiko umum Docker yang benar?
A. Docker group terlalu luas dapat menjadi privilege escalation
B. Mount docker.sock ke container memberi kontrol daemon
C. `--privileged melemahkan isolasi`
D. `Tidak ada resource limit dapat menyebabkan DoS host`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Keempat risiko ini disebut dalam dokumentasi konseptual.

---
### Soal 136
**Pertanyaan:** Manakah urutan implementasi yang masuk akal sesuai modul?
A. Hardening host Linux dahulu
B. Install Docker dari sumber tepercaya
C. `Batasi group docker dan audit rules`
D. Jalankan semua container privileged terlebih dahulu baru hardening belakangan

**Jawaban benar:** A, B, C

**Pembahasan:** Hardening dimulai dari host dan daemon sebelum workload production.

---
### Soal 137
**Pertanyaan:** Manakah konsep yang benar tentang `no-new-privileges`?
A. `Mencegah proses memperoleh privilege tambahan melalui mekanisme seperti SUID/SGID`
B. Dapat disetel di daemon.json sebagai default
C. `Dapat disetel saat runtime dengan `--security-opt no-new-privileges:true``
D. Membuat container otomatis rootless

**Jawaban benar:** A, B, C

**Pembahasan:** no-new-privileges bukan rootless, tetapi mencegah privilege escalation tambahan.

---
### Soal 138
**Pertanyaan:** Manakah konsep yang benar tentang seccomp?
A. Membatasi syscall yang dapat digunakan proses container
B. Default seccomp Docker membantu mengurangi kernel attack surface
C. Custom seccomp harus diuji karena bisa memblokir syscall aplikasi
D. `seccomp=unconfined adalah baseline production CIS`

**Jawaban benar:** A, B, C

**Pembahasan:** seccomp unconfined tidak sesuai baseline.

---
### Soal 139
**Pertanyaan:** Manakah konsep benar tentang AppArmor/SELinux untuk container?
A. `Keduanya adalah lapisan MAC/mandatory access control`
B. `Ubuntu/Debian umumnya memakai AppArmor`
C. RHEL family umum memakai SELinux
D. `Keduanya tidak perlu jika container sudah non-root`

**Jawaban benar:** A, B, C

**Pembahasan:** Non-root tidak menggantikan MAC.

---
### Soal 140
**Pertanyaan:** Manakah alasan benar memakai custom network per aplikasi?
A. Segmentasi lebih jelas
B. Audit network lebih mudah
C. `Mengurangi komunikasi bebas antar container yang tidak perlu`
D. Memastikan semua container langsung terbuka ke internet

**Jawaban benar:** A, B, C

**Pembahasan:** Custom network membantu isolasi, bukan membuka semua container.

---
### Soal 141
**Pertanyaan:** Manakah praktik benar untuk image lifecycle?
A. Build image dari base tepercaya
B. Scan image
C. Rebuild ketika base image atau paket mendapat patch keamanan
D. Gunakan image lama selamanya karena pernah lolos scan

**Jawaban benar:** A, B, C

**Pembahasan:** Image harus dipelihara dan dibangun ulang secara berkala.

---
