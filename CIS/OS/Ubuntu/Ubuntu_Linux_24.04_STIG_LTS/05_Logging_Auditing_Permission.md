# 05 - Logging, Auditing, Permission, dan File Protection

**Sumber utama:** CIS Ubuntu Linux 24.04 LTS STIG Benchmark v1.0.0, 06-12-2025.  
**Target dokumen:** modul belajar tematik untuk memahami hardening Ubuntu 24.04 LTS berbasis STIG.  
**Catatan penting:** file ini adalah dokumentasi pembelajaran dan panduan konseptual. Untuk audit resmi, tetap gunakan dokumen benchmark asli, hasil scanning, dan kebijakan organisasi.

---


## 1. Ruang lingkup modul

Modul ini membahas bagian terbesar dari STIG Ubuntu 24.04 LTS: **logging**, **rsyslog**, **auditd**, **audit rules**, **audit storage**, **audit offloading**, **permission log**, **journalctl**, **audit tools**, **library/system command ownership**, dan perlindungan file sistem. Jika modul 04 menjawab “siapa yang boleh melakukan sesuatu”, modul ini menjawab “apakah tindakan itu tercatat dan apakah buktinya terlindungi”.

## 2. Peta aturan dalam modul ini

| Tema | Aturan terkait | Makna dalam hardening |
|---|---|---|
| System logging dan rsyslog | `UBTU-24-100200` (Automated, CAT II), `UBTU-24-700010` (Automated, CAT II), `UBTU-24-700020` (Automated, CAT II) | Logging harus aktif, preservasi event failure dijaga, dan pesan log tidak boleh membocorkan informasi yang bisa dieksploitasi. |
| Auditd service dan startup audit | `UBTU-24-100400` (Automated, CAT II), `UBTU-24-100410` (Automated, CAT II), `UBTU-24-102010` (Automated, CAT II) | Auditd harus tersedia dan aktif agar sistem menghasilkan audit records sejak startup. |
| Audit offloading, storage, alerting, dan UTC | `UBTU-24-100450` (Automated, CAT III), `UBTU-24-900920` (Manual, CAT III), `UBTU-24-900950` (Manual, CAT III), `UBTU-24-900960` (Manual, CAT III), `UBTU-24-900980` (Automated, CAT III), `UBTU-24-901220` (Automated, CAT III) | Audit log harus cukup kapasitas, bisa dioffload, memberi alert saat masalah, dan menggunakan timestamp yang dapat dipetakan ke UTC. |
| Audit account dan privilege activity | `UBTU-24-200280` (Automated, CAT II), `UBTU-24-200290` (Automated, CAT II), `UBTU-24-200300` (Automated, CAT II), `UBTU-24-200310` (Automated, CAT II), `UBTU-24-200320` (Automated, CAT II), `UBTU-24-200580` (Automated, CAT II), `UBTU-24-500010` (Automated, CAT II), `UBTU-24-900070` (Automated, CAT II), `UBTU-24-900170` (Automated, CAT II), `UBTU-24-900180` (Automated, CAT II), `UBTU-24-900510` (Automated, CAT II), `UBTU-24-900520` (Automated, CAT II) | Perubahan account, fungsi privileged, sudo/sudoedit, su, dan sudoers harus tercatat. |
| Audit file, syscall, mount, login records, kernel module | `UBTU-24-300029` (Automated, CAT II), `UBTU-24-900080` (Automated, CAT II), `UBTU-24-900090` (Automated, CAT II), `UBTU-24-900100` (Automated, CAT II), `UBTU-24-900110` (Automated, CAT II), `UBTU-24-900120` (Automated, CAT II), `UBTU-24-900130` (Automated, CAT II), `UBTU-24-900140` (Automated, CAT II), `UBTU-24-900150` (Automated, CAT II), `UBTU-24-900160` (Automated, CAT II), `UBTU-24-900190` (Automated, CAT II), `UBTU-24-900200` (Automated, CAT II), `UBTU-24-900210` (Automated, CAT II), `UBTU-24-900220` (Automated, CAT II), `UBTU-24-900230` (Automated, CAT II), `UBTU-24-900240` (Automated, CAT II), `UBTU-24-900250` (Automated, CAT II), `UBTU-24-900260` (Automated, CAT II), `UBTU-24-900270` (Automated, CAT II), `UBTU-24-900280` (Automated, CAT II), `UBTU-24-900290` (Automated, CAT II), `UBTU-24-900300` (Automated, CAT II), `UBTU-24-900310` (Automated, CAT II), `UBTU-24-900320` (Automated, CAT II), `UBTU-24-900330` (Automated, CAT II), `UBTU-24-900340` (Automated, CAT II), `UBTU-24-900350` (Automated, CAT II), `UBTU-24-900540` (Automated, CAT II), `UBTU-24-900590` (Automated, CAT II), `UBTU-24-900600` (Automated, CAT II), `UBTU-24-900610` (Automated, CAT II), `UBTU-24-900730` (Automated, CAT II), `UBTU-24-900740` (Automated, CAT II), `UBTU-24-900750` (Automated, CAT II), `UBTU-24-909000` (Automated, CAT II) | Audit rules mencatat perubahan atribut, permission, file access, session record, kernel module, disk tools, dan membuat rule immutable. |
| Permission library, command, journal, log, dan audit tools | `UBTU-24-300006` (Automated, CAT II), `UBTU-24-300007` (Automated, CAT II), `UBTU-24-300008` (Automated, CAT II), `UBTU-24-300009` (Automated, CAT II), `UBTU-24-300010` (Automated, CAT II), `UBTU-24-300011` (Automated, CAT II), `UBTU-24-300012` (Automated, CAT II), `UBTU-24-300013` (Automated, CAT II), `UBTU-24-300030` (Automated, CAT II), `UBTU-24-700030` (Automated, CAT II), `UBTU-24-700040` (Automated, CAT II), `UBTU-24-700050` (Automated, CAT II), `UBTU-24-700060` (Automated, CAT II), `UBTU-24-700070` (Automated, CAT II), `UBTU-24-700080` (Automated, CAT II), `UBTU-24-700090` (Automated, CAT II), `UBTU-24-700100` (Automated, CAT II), `UBTU-24-700110` (Automated, CAT II), `UBTU-24-700120` (Automated, CAT II), `UBTU-24-700130` (Automated, CAT II), `UBTU-24-700140` (Automated, CAT II), `UBTU-24-700150` (Automated, CAT II), `UBTU-24-900040` (Automated, CAT II), `UBTU-24-900050` (Automated, CAT II), `UBTU-24-900060` (Automated, CAT II), `UBTU-24-901230` (Automated, CAT II), `UBTU-24-901240` (Automated, CAT II), `UBTU-24-901250` (Automated, CAT II), `UBTU-24-901260` (Automated, CAT II), `UBTU-24-901270` (Automated, CAT II), `UBTU-24-901280` (Automated, CAT II), `UBTU-24-901300` (Automated, CAT II), `UBTU-24-901310` (Automated, CAT II), `UBTU-24-901350` (Automated, CAT II), `UBTU-24-901380` (Automated, CAT II) | Permission dan ownership file/direktori penting mencegah user tidak sah membaca, menulis, atau mengganti komponen sistem dan audit. |

## 3. Logging biasa vs auditing

Logging dan auditing sering dianggap sama, padahal fokusnya berbeda.

**Logging biasa** mencatat aktivitas sistem dan service, seperti error, warning, login, aktivitas daemon, kernel message, dan pesan aplikasi. Logging membantu troubleshooting dan monitoring.

**Auditing** mencatat aktivitas keamanan yang harus bisa dipertanggungjawabkan. Audit rules dapat menangkap perubahan file penting, penggunaan command privileged, syscall tertentu, perubahan account, perubahan permission, penggunaan sudo, perubahan kernel module, dan tindakan lain yang bernilai forensik.

Dalam STIG, keduanya diperlukan. Logging memberi visibility operasional, auditing memberi accountability formal.

## 4. Rsyslog dan preservasi log

STIG menuntut sistem memiliki service logging yang aktif. Rsyslog digunakan untuk preservasi log records dari event failure dan pengelolaan log tradisional.

Aturan terkait: `UBTU-24-100200` (Automated, CAT II).

Log failure event penting karena ketika sistem gagal, restart, crash, atau service bermasalah, log menjadi sumber utama rekonstruksi kejadian. Tanpa log yang tersedia, troubleshooting dan forensik menjadi spekulatif.

STIG juga mengatur agar error message dan journal entry tidak membocorkan informasi yang bisa dieksploitasi attacker. Pesan log harus cukup informatif untuk corrective action, tetapi tidak berlebihan sampai membuka detail internal yang tidak perlu.

Aturan terkait: `UBTU-24-700010` (Automated, CAT II), `UBTU-24-700020` (Automated, CAT II).

## 5. Auditd sebagai sistem audit formal

Auditd adalah daemon auditing Linux. Ia mencatat event berdasarkan rule yang ditentukan administrator. Auditd berbeda dari syslog karena dapat menangkap syscall, akses file sensitif, perubahan konfigurasi, dan aktivitas privileged dengan detail yang lebih kuat.

Aturan terkait: `UBTU-24-100400` (Automated, CAT II), `UBTU-24-100410` (Automated, CAT II), `UBTU-24-102010` (Automated, CAT II).

Auditd harus aktif sejak startup agar aktivitas awal sistem tidak terlewat. Ini penting karena attacker dapat memanfaatkan fase boot atau service startup untuk mengubah konfigurasi sebelum mekanisme audit berjalan.

Audit record yang baik harus dapat menjawab:

- kapan event terjadi;
- di mana event terjadi;
- user/proses apa yang terlibat;
- event berhasil atau gagal;
- file/command/syscall apa yang terdampak;
- apakah event berkaitan dengan privilege tinggi.

## 6. Audit offloading, storage, alerting, dan UTC

Audit log harus dilindungi dari kehilangan. Jika log hanya tersimpan lokal, attacker yang mendapat privilege tinggi mungkin mencoba menghapus atau memodifikasinya. Karena itu, STIG membahas audit log offloading ke sistem lain, kapasitas penyimpanan, alert ketika storage rendah, alert kegagalan audit process, dan timestamp yang dapat dipetakan ke UTC.

Aturan terkait: `UBTU-24-100450` (Automated, CAT III), `UBTU-24-900920` (Manual, CAT III), `UBTU-24-900950` (Manual, CAT III), `UBTU-24-900960` (Manual, CAT III), `UBTU-24-900980` (Automated, CAT III), `UBTU-24-901220` (Automated, CAT III).

Konsep penting:

- **Offloading** membuat bukti tidak hanya berada di host yang sama dengan target serangan.
- **Storage capacity** memastikan log tidak cepat overwrite.
- **Alerting** memastikan admin tahu saat audit system bermasalah.
- **UTC timestamp** membantu korelasi lintas zona waktu dan sistem.

Untuk standalone system, dokumen juga menyinggung proses berkala untuk memindahkan audit event. Ini penting pada host yang tidak selalu punya koneksi ke server log pusat.

## 7. Audit account dan privilege activity

Perubahan account adalah event keamanan penting. File seperti `/etc/passwd`, `/etc/group`, `/etc/shadow`, `/etc/gshadow`, dan `/etc/security/opasswd` menentukan identitas, group, password hash, dan riwayat password. Perubahan pada file ini harus diaudit.

Aturan terkait account files: `UBTU-24-200280` (Automated, CAT II), `UBTU-24-200290` (Automated, CAT II), `UBTU-24-200300` (Automated, CAT II), `UBTU-24-200310` (Automated, CAT II), `UBTU-24-200320` (Automated, CAT II).

Aktivitas privileged juga harus diaudit. Ini mencakup `su`, `sudo`, `sudoedit`, perubahan `sudoers`, dan eksekusi fungsi privileged. Tanpa audit, administrator tidak dapat membuktikan siapa menaikkan privilege atau mengubah kebijakan sudo.

Aturan terkait privilege: `UBTU-24-200580` (Automated, CAT II), `UBTU-24-500010` (Automated, CAT II), `UBTU-24-900070` (Automated, CAT II), `UBTU-24-900170` (Automated, CAT II), `UBTU-24-900180` (Automated, CAT II), `UBTU-24-900510` (Automated, CAT II), `UBTU-24-900520` (Automated, CAT II).

## 8. Audit rules detail

STIG memiliki banyak aturan audit detail. Tujuannya bukan membuat sistem “ramai log”, tetapi memastikan tindakan sensitif tercatat. Kelompok audit rule mencakup:

- perubahan account dan autentikasi;
- perubahan systemd journal;
- penggunaan `chfn`, `chsh`, `newgrp`, `passwd`, `gpasswd`, `chage`, `usermod`;
- mount dan umount;
- SSH helper seperti `ssh-agent` dan `ssh-keysign`;
- perubahan extended attribute;
- perubahan ownership dan permission (`chown`, `chmod`);
- file creation/open/truncate;
- deletion/rename;
- MAC/AppArmor tools;
- ACL tools (`setfacl`, `chacl`);
- cron modification;
- kernel module load/unload;
- disk utility seperti `fdisk`;
- login records seperti `wtmp`, `utmp`, dan `btmp`;
- immutable audit configuration.

Aturan terkait: `UBTU-24-300029` (Automated, CAT II), `UBTU-24-900080` (Automated, CAT II), `UBTU-24-900090` (Automated, CAT II), `UBTU-24-900100` (Automated, CAT II), `UBTU-24-900110` (Automated, CAT II), `UBTU-24-900120` (Automated, CAT II), `UBTU-24-900130` (Automated, CAT II), `UBTU-24-900140` (Automated, CAT II), `UBTU-24-900150` (Automated, CAT II), `UBTU-24-900160` (Automated, CAT II), `UBTU-24-900190` (Automated, CAT II), `UBTU-24-900200` (Automated, CAT II), `UBTU-24-900210` (Automated, CAT II), `UBTU-24-900220` (Automated, CAT II), `UBTU-24-900230` (Automated, CAT II), `UBTU-24-900240` (Automated, CAT II), `UBTU-24-900250` (Automated, CAT II), `UBTU-24-900260` (Automated, CAT II), `UBTU-24-900270` (Automated, CAT II), `UBTU-24-900280` (Automated, CAT II), `UBTU-24-900290` (Automated, CAT II), `UBTU-24-900300` (Automated, CAT II), `UBTU-24-900310` (Automated, CAT II), `UBTU-24-900320` (Automated, CAT II), `UBTU-24-900330` (Automated, CAT II), `UBTU-24-900340` (Automated, CAT II), `UBTU-24-900350` (Automated, CAT II), `UBTU-24-900540` (Automated, CAT II), `UBTU-24-900590` (Automated, CAT II), `UBTU-24-900600` (Automated, CAT II), `UBTU-24-900610` (Automated, CAT II), `UBTU-24-900730` (Automated, CAT II), `UBTU-24-900740` (Automated, CAT II), `UBTU-24-900750` (Automated, CAT II), `UBTU-24-909000` (Automated, CAT II).

Aturan immutable (`-e 2`) penting karena setelah audit rule dibuat immutable, rule tidak dapat diubah tanpa reboot. Ini menyulitkan attacker yang ingin menonaktifkan audit, melakukan aktivitas jahat, lalu mengembalikan konfigurasi seperti semula.

## 9. Permission dan ownership file sistem penting

STIG juga banyak mengatur permission dan ownership untuk library, command, journal, log, audit config, audit tools, dan audit log. Tujuannya adalah mencegah user tidak sah membaca, menulis, atau mengganti file yang memengaruhi keamanan.

Aturan library dan command: `UBTU-24-300006` (Automated, CAT II), `UBTU-24-300007` (Automated, CAT II), `UBTU-24-300008` (Automated, CAT II), `UBTU-24-300009` (Automated, CAT II), `UBTU-24-300010` (Automated, CAT II), `UBTU-24-300011` (Automated, CAT II), `UBTU-24-300012` (Automated, CAT II), `UBTU-24-300013` (Automated, CAT II), `UBTU-24-901260` (Automated, CAT II), `UBTU-24-901270` (Automated, CAT II), `UBTU-24-901280` (Automated, CAT II).

Jika library atau system command bisa dimodifikasi user biasa, attacker dapat menyisipkan kode, mengganti binary, atau memengaruhi proses privileged. Karena itu, mode, owner, dan group harus ketat.

## 10. Journalctl, system journal, dan /var/log

Journal dan `/var/log` berisi informasi operasional dan keamanan. Permission yang terlalu longgar dapat membocorkan username, alamat IP, path aplikasi, error sensitif, token, atau pola konfigurasi internal.

Aturan terkait journal/log: `UBTU-24-700030` (Automated, CAT II), `UBTU-24-700040` (Automated, CAT II), `UBTU-24-700050` (Automated, CAT II), `UBTU-24-700060` (Automated, CAT II), `UBTU-24-700070` (Automated, CAT II), `UBTU-24-700080` (Automated, CAT II), `UBTU-24-700090` (Automated, CAT II), `UBTU-24-700100` (Automated, CAT II), `UBTU-24-700110` (Automated, CAT II), `UBTU-24-700120` (Automated, CAT II), `UBTU-24-700130` (Automated, CAT II), `UBTU-24-700140` (Automated, CAT II), `UBTU-24-700150` (Automated, CAT II).

Konsepnya:

- `journalctl` tidak boleh bebas digunakan user tidak sah;
- command harus dimiliki root/root;
- direktori journal dan file journal harus memiliki owner/group tepat;
- `/var/log` harus root-owned dengan group yang sesuai;
- `/var/log/syslog` harus memiliki owner/group/mode yang tidak terlalu permisif.

## 11. Audit configuration, audit tools, dan audit log file access

Audit system hanya dapat dipercaya jika konfigurasinya sendiri terlindungi. Jika attacker bisa mengubah rule, mengganti tools, atau memodifikasi audit log, bukti forensik menjadi lemah.

Aturan terkait audit config/tools/log: `UBTU-24-900040` (Automated, CAT II), `UBTU-24-900050` (Automated, CAT II), `UBTU-24-900060` (Automated, CAT II), `UBTU-24-901230` (Automated, CAT II), `UBTU-24-901240` (Automated, CAT II), `UBTU-24-901250` (Automated, CAT II), `UBTU-24-901300` (Automated, CAT II), `UBTU-24-901310` (Automated, CAT II), `UBTU-24-901350` (Automated, CAT II), `UBTU-24-901380` (Automated, CAT II).

Tiga lapisan perlindungan:

1. **Audit configuration protection** - file rule tidak boleh ditulis oleh user tidak sah.
2. **Audit tools protection** - tools audit harus mode/owner/group yang aman.
3. **Audit log protection** - log tidak boleh dibaca/ditulis oleh user tidak sah dan harus dimiliki akun/grup yang benar.

## 12. Default filesystem permission dan shared directory

Default filesystem permission mengatur bagaimana file baru dibuat. Jika default terlalu permisif, user lain bisa membaca atau memodifikasi file yang tidak seharusnya. STIG juga menekankan sticky bit pada direktori publik agar user tidak bisa menghapus file milik user lain di direktori bersama.

Aturan terkait: `UBTU-24-300030` (Automated, CAT II), `UBTU-24-600150` (Automated, CAT II).

Ini terlihat sederhana, tetapi sangat penting di sistem multiuser.

## 13. Checklist pemahaman modul

- Logging biasa memberi visibility operasional; auditd memberi accountability formal.
- Rsyslog harus aktif agar log event terpelihara.
- Auditd harus aktif sejak startup.
- Audit log perlu offloading, storage planning, alerting, dan timestamp yang konsisten.
- Perubahan account file dan sudoers harus diaudit.
- Aktivitas privileged seperti `su`, `sudo`, `sudoedit`, dan command sensitif harus tercatat.
- Permission library, command, log, journal, audit config, audit tools, dan audit logs harus ketat.
- Immutable audit rules membuat perubahan audit rule tidak bisa dilakukan sembarangan saat runtime.
- Log yang tidak terlindungi dapat dibaca, diubah, atau dihapus oleh attacker.

## 14. Kesimpulan

Logging dan auditing adalah pusat pembuktian keamanan. Tanpa kontrol ini, sistem mungkin terlihat berjalan normal tetapi tidak mampu menjelaskan apa yang terjadi saat insiden. Modul ini juga menunjukkan bahwa permission file sistem adalah bagian dari audit assurance: bukti hanya bernilai jika alat, konfigurasi, dan log yang menghasilkan bukti tersebut terlindungi.


## Appendix - Validasi cakupan aturan

Seluruh 188 kode aturan STIG dalam dokumen sumber telah dipetakan ke salah satu modul tematik 02 sampai 05.
