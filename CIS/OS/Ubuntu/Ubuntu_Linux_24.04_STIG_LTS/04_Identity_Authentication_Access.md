# 04 - Identity, Authentication, dan Access Control

**Sumber utama:** CIS Ubuntu Linux 24.04 LTS STIG Benchmark v1.0.0, 06-12-2025.  
**Target dokumen:** modul belajar tematik untuk memahami hardening Ubuntu 24.04 LTS berbasis STIG.  
**Catatan penting:** file ini adalah dokumentasi pembelajaran dan panduan konseptual. Untuk audit resmi, tetap gunakan dokumen benchmark asli, hasil scanning, dan kebijakan organisasi.

---


## 1. Ruang lingkup modul

Modul ini membahas kontrol STIG terkait **identitas user**, **PAM**, **password policy**, **MFA**, **SSSD**, **smart card**, **PIV/PKI**, **session control**, **root login**, **sudo**, dan **privilege escalation**. Bagian ini adalah inti access control: siapa boleh masuk, bagaimana mereka membuktikan identitas, kapan sesi dikunci, dan bagaimana privilege tinggi diberikan.

## 2. Peta aturan dalam modul ini

| Tema | Aturan terkait | Makna dalam hardening |
|---|---|---|
| PAM dan password quality | `UBTU-24-100600` (Automated, CAT II), `UBTU-24-300014` (Automated, CAT II), `UBTU-24-300016` (Automated, CAT II), `UBTU-24-300017` (Automated, CAT III), `UBTU-24-400220` (Automated, CAT II), `UBTU-24-400260` (Automated, CAT II), `UBTU-24-400270` (Automated, CAT II), `UBTU-24-400280` (Automated, CAT II), `UBTU-24-400290` (Automated, CAT II), `UBTU-24-400300` (Automated, CAT II), `UBTU-24-400310` (Automated, CAT II), `UBTU-24-400320` (Automated, CAT II), `UBTU-24-400330` (Automated, CAT II), `UBTU-24-400340` (Automated, CAT III), `UBTU-24-400400` (Automated, CAT II) | PAM dan pwquality mengatur password strength, lifetime, hashing, delay login gagal, dan larangan cached authentication terlalu lama. |
| Blank password, account lock, dan lifecycle akun | `UBTU-24-200250` (Manual, CAT II), `UBTU-24-200260` (Automated, CAT II), `UBTU-24-200610` (Automated, CAT III), `UBTU-24-300027` (Automated, CAT I), `UBTU-24-300028` (Automated, CAT I) | Akun sementara, akun inactive, login gagal, dan blank password harus dikendalikan. |
| Root, user identity, dan privilege escalation | `UBTU-24-300021` (Automated, CAT II), `UBTU-24-400000` (Automated, CAT II), `UBTU-24-400110` (Automated, CAT II) | Identitas user harus unik, root login langsung dicegah, dan privilege escalation harus memerlukan reauthentication. |
| Sudo, su, dan akses fungsi keamanan | `UBTU-24-600130` (Manual, CAT I), `UBTU-24-500010` (Automated, CAT II), `UBTU-24-500050` (Automated, CAT II) | Hanya user yang membutuhkan akses boleh berada dalam sudo group; aktivitas privileged dan maintenance session harus kuat dan tercatat. |
| MFA, SSSD, Smart Card, PIV, dan PKI | `UBTU-24-100650` (Automated, CAT II), `UBTU-24-100660` (Automated, CAT II), `UBTU-24-100900` (Automated, CAT II), `UBTU-24-100910` (Automated, CAT II), `UBTU-24-400020` (Automated, CAT II), `UBTU-24-400030` (Automated, CAT II), `UBTU-24-400060` (Automated, CAT II), `UBTU-24-400360` (Automated, CAT II), `UBTU-24-400370` (Manual, CAT I), `UBTU-24-400375` (Automated, CAT II), `UBTU-24-400380` (Automated, CAT II), `UBTU-24-600060` (Automated, CAT II) | STIG menekankan multifactor authentication, smart card/PIV, certificate validation, revocation cache, dan CA tepercaya. |
| Session control dan local/graphical access | `UBTU-24-101000` (Automated, CAT II), `UBTU-24-200000` (Automated, CAT III), `UBTU-24-200020` (Automated, CAT II), `UBTU-24-200040` (Automated, CAT II), `UBTU-24-200060` (Automated, CAT II), `UBTU-24-300024` (Automated, CAT III), `UBTU-24-300025` (Automated, CAT I), `UBTU-24-300026` (Automated, CAT I), `UBTU-24-200640` (Manual, CAT II), `UBTU-24-200650` (Automated, CAT II), `UBTU-24-200660` (Manual, CAT II) | Session lock, inactivity timeout, concurrent session limit, last login display, Ctrl-Alt-Delete, dan login banner mengurangi risiko akses tidak sah. |

## 3. Identitas user dan prinsip accountability

Access control dimulai dari identitas yang jelas. Jika satu akun dipakai bersama, atau user tidak unik, sistem tidak bisa membuktikan siapa melakukan tindakan tertentu. STIG menuntut sistem mampu mengidentifikasi user interaktif secara unik.

Aturan terkait: `UBTU-24-400000` (Automated, CAT II).

Accountability bergantung pada tiga hal:

- user unik;
- autentikasi kuat;
- aktivitas tercatat.

Tanpa identitas unik, audit log kehilangan nilai forensik. Misalnya, jika beberapa orang memakai satu akun admin bersama, log hanya menunjukkan akun tersebut, bukan individu yang bertindak.

## 4. PAM dan password policy

PAM adalah lapisan autentikasi modular di Linux. Melalui PAM, Ubuntu dapat menerapkan password quality, account lockout, password history, smart card, SSSD, dan aturan autentikasi lain. `libpam-pwquality` digunakan untuk memeriksa kekuatan password saat password dibuat atau diganti.

Aturan terkait password/PAM: `UBTU-24-100600` (Automated, CAT II), `UBTU-24-300014` (Automated, CAT II), `UBTU-24-300016` (Automated, CAT II), `UBTU-24-300017` (Automated, CAT III), `UBTU-24-400220` (Automated, CAT II), `UBTU-24-400260` (Automated, CAT II), `UBTU-24-400270` (Automated, CAT II), `UBTU-24-400280` (Automated, CAT II), `UBTU-24-400290` (Automated, CAT II), `UBTU-24-400300` (Automated, CAT II), `UBTU-24-400310` (Automated, CAT II), `UBTU-24-400320` (Automated, CAT II), `UBTU-24-400330` (Automated, CAT II), `UBTU-24-400340` (Automated, CAT III), `UBTU-24-400400` (Automated, CAT II).

Password policy dalam STIG mencakup:

- larangan dictionary words;
- penggunaan `pwquality` saat password dibuat/diganti;
- delay setelah login gagal;
- password disimpan dalam bentuk hash terenkripsi;
- syarat uppercase, lowercase, numeric, special character;
- perubahan minimal karakter saat password diganti;
- minimum password lifetime;
- maximum password lifetime;
- minimum panjang password;
- pembatasan cached authentication;
- penggunaan algoritma hashing sesuai baseline/FIPS.

Kebijakan password bukan hanya soal “panjang dan kompleks”. Ia juga membatasi password reuse, memperlambat brute force, mencegah password kosong, dan memastikan penyimpanan password tidak mudah dibalik menjadi plaintext.

## 5. Blank/null password dan login gagal

Blank password adalah risiko kritis karena user bisa masuk tanpa secret. Dalam STIG, larangan blank password muncul pada konfigurasi akun dan PAM.

Aturan terkait: `UBTU-24-300027` (Automated, CAT I), `UBTU-24-300028` (Automated, CAT I), `UBTU-24-200610` (Automated, CAT III).

Account lockout setelah beberapa kegagalan login membantu mengurangi brute force. Namun lockout juga harus dirancang hati-hati agar tidak mudah dipakai untuk denial of service terhadap user sah. Dalam lingkungan produksi, lockout policy perlu dikombinasikan dengan monitoring dan prosedur unlock yang jelas.

## 6. Lifecycle akun: temporary dan inactive accounts

Akun sementara harus otomatis kedaluwarsa, dan akun inactive harus dinonaktifkan. Ini mencegah akun lama menjadi backdoor administratif yang terlupakan.

Aturan terkait: `UBTU-24-200250` (Manual, CAT II), `UBTU-24-200260` (Automated, CAT II).

Risiko akun tidak terkelola:

- mantan user masih punya akses;
- akun proyek sementara tetap aktif setelah proyek selesai;
- akun vendor tidak dicabut;
- attacker menemukan akun lama dengan password lemah;
- audit tidak bisa membedakan akun aktif dan tidak aktif.

## 7. Root login dan privilege escalation

Root adalah akun paling sensitif. STIG menuntut pencegahan direct root login dan reauthentication ketika privilege escalation atau pergantian role dilakukan.

Aturan terkait: `UBTU-24-400110` (Automated, CAT II), `UBTU-24-300021` (Automated, CAT II).

Melarang direct root login meningkatkan accountability karena admin masuk menggunakan akun pribadi, lalu melakukan eskalasi privilege melalui mekanisme terkontrol seperti sudo. Dengan demikian, tindakan administratif dapat dikaitkan dengan user tertentu.

Reauthentication penting karena sesi yang sudah terbuka tidak boleh otomatis memberi akses privilege tinggi tanpa validasi ulang. Ini mengurangi risiko jika terminal admin ditinggalkan atau sesi dicuri.

## 8. Sudo, su, dan akses fungsi keamanan

Sudo dan su adalah jalur menuju privilege tinggi. STIG menekankan pembatasan anggota sudo group dan audit atas aktivitas privileged.

Aturan terkait: `UBTU-24-600130` (Manual, CAT I), `UBTU-24-500010` (Automated, CAT II), `UBTU-24-500050` (Automated, CAT II).

Prinsip yang digunakan:

- hanya user yang membutuhkan privilege yang boleh menjadi anggota sudo;
- penggunaan sudo/su harus dicatat;
- maintenance session nonlocal harus memakai authenticator kuat;
- konfigurasi sudoers harus dilindungi dan diaudit.

Kontrol ini tidak berdiri sendiri. Ia berhubungan dengan audit rules dalam modul 05, terutama audit penggunaan `sudo`, `sudoedit`, perubahan `/etc/sudoers`, dan privileged commands.

## 9. MFA, SSSD, Smart Card, PIV, dan PKI

STIG Ubuntu 24.04 LTS memberi perhatian besar pada MFA dan identitas berbasis sertifikat. SSSD berperan sebagai layanan identitas yang menghubungkan sistem dengan sumber identitas seperti LDAP, Active Directory, FreeIPA, Kerberos, atau mekanisme identity provider lain. Smart card/PIV dan PKI dipakai untuk meningkatkan assurance autentikasi.

Aturan terkait: `UBTU-24-100650` (Automated, CAT II), `UBTU-24-100660` (Automated, CAT II), `UBTU-24-100900` (Automated, CAT II), `UBTU-24-100910` (Automated, CAT II), `UBTU-24-400020` (Automated, CAT II), `UBTU-24-400030` (Automated, CAT II), `UBTU-24-400060` (Automated, CAT II), `UBTU-24-400360` (Automated, CAT II), `UBTU-24-400370` (Manual, CAT I), `UBTU-24-400375` (Automated, CAT II), `UBTU-24-400380` (Automated, CAT II), `UBTU-24-600060` (Automated, CAT II).

Konsep yang perlu dipahami:

- MFA memakai minimal dua faktor: sesuatu yang diketahui, dimiliki, atau melekat pada user.
- Smart card/PIV mengikat identitas ke credential fisik/kriptografis.
- PKI membutuhkan certificate path validation menuju trust anchor.
- Revocation checking/cache penting agar sertifikat yang dicabut tidak tetap dipercaya.
- PAM dan SSSD perlu selaras agar login lokal, SSH, dan privilege flow konsisten.

Aturan PKI/MFA termasuk area yang paling kontekstual. Implementasinya bergantung pada infrastruktur organisasi. Dalam lab pribadi, beberapa aturan bisa tidak applicable atau perlu pengecualian, tetapi dalam audit STIG formal, pengecualian harus punya alasan dan bukti.

## 10. Session control

Session control membatasi risiko ketika user sudah berhasil login. Fokusnya bukan lagi “boleh masuk atau tidak”, tetapi “apa yang terjadi setelah masuk”.

Aturan terkait: `UBTU-24-101000` (Automated, CAT II), `UBTU-24-200000` (Automated, CAT III), `UBTU-24-200020` (Automated, CAT II), `UBTU-24-200040` (Automated, CAT II), `UBTU-24-200060` (Automated, CAT II), `UBTU-24-300024` (Automated, CAT III).

Kontrol session meliputi:

- user dapat memulai session lock;
- graphical session terkunci setelah idle;
- lock tetap bertahan sampai user autentikasi ulang;
- sesi idle otomatis berakhir;
- jumlah sesi bersamaan dibatasi;
- last successful login ditampilkan.

Ini penting terutama pada workstation, server dengan console access, dan lingkungan multiuser.

## 11. Ctrl-Alt-Delete dan login banner

Ctrl-Alt-Delete dapat menyebabkan reboot atau tindakan sistem yang tidak diinginkan jika tidak dibatasi. Pada sistem kritis, akses fisik atau GUI tidak boleh memberi shortcut untuk mengganggu availability.

Aturan terkait Ctrl-Alt-Delete: `UBTU-24-300025` (Automated, CAT I), `UBTU-24-300026` (Automated, CAT I).

Login banner adalah notice hukum/kebijakan sebelum akses diberikan. STIG mengatur banner pada local graphical logon dan SSH. Banner membantu memperjelas bahwa sistem hanya boleh digunakan oleh pihak berwenang dan aktivitas dapat dipantau.

Aturan terkait banner: `UBTU-24-200640` (Manual, CAT II), `UBTU-24-200650` (Automated, CAT II), `UBTU-24-200660` (Manual, CAT II), `UBTU-24-200680` (Manual, CAT II).

Banner harus konsisten, tidak membocorkan informasi sistem, dan sesuai kebijakan organisasi.

## 12. Checklist pemahaman modul

- User interaktif harus unik agar audit log bermakna.
- PAM adalah pusat banyak kontrol autentikasi.
- Password policy mencakup complexity, length, lifetime, hashing, dan lockout.
- Blank/null password termasuk risiko kritis.
- Root login langsung harus dicegah untuk menjaga accountability.
- Privilege escalation harus membutuhkan reauthentication.
- Sudo group harus berisi user yang memang membutuhkan akses.
- MFA, smart card, PIV, dan PKI membutuhkan infrastruktur identitas yang matang.
- Session lock, timeout, last login, dan banner melindungi tahap setelah user berhasil login.

## 13. Kesimpulan

Access control dalam STIG bukan hanya password. Ia menggabungkan identitas unik, password policy, PAM, MFA, PKI, root restriction, sudo governance, session control, dan legal notice. Jika modul ini diterapkan tanpa perencanaan, risiko terbesar adalah user terkunci dari sistem. Karena itu, perubahan PAM, SSSD, smart card, root login, dan sudo harus selalu diuji di lab dan pilot terlebih dahulu.
