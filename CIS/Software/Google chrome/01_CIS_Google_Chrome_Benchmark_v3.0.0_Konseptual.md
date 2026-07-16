# CIS Google Chrome Benchmark v3.0.0 — Dokumentasi Konseptual

**Sumber utama:** CIS Google Chrome Benchmark v3.0.0 — 01-29-2024.  
**Target pembahasan:** pemahaman konseptual kebijakan hardening Google Chrome, bukan sekadar daftar klik Group Policy.

---

## 1. Gambaran Umum

CIS Google Chrome Benchmark adalah panduan konfigurasi keamanan untuk membangun postur aman pada browser Google Chrome. Fokusnya bukan mengganti antivirus, EDR, firewall, atau patch management, melainkan memberi baseline konfigurasi teknis agar perilaku Chrome lebih terkendali di lingkungan organisasi.

Browser adalah salah satu aplikasi paling sering berinteraksi dengan internet. Karena itu, browser menjadi titik masuk penting untuk phishing, malware, drive-by download, penyalahgunaan ekstensi, pencurian kredensial, pelacakan pengguna, dan kebocoran data. Hardening Chrome berarti mengurangi kebebasan konfigurasi yang berisiko, mengunci kebijakan penting, dan memastikan pengaturan keamanan tidak hanya bergantung pada pilihan pengguna.

Dokumen benchmark ini diuji terhadap **Google Chrome v120**. Sifatnya prescriptive guidance, yaitu memberi arahan konfigurasi yang direkomendasikan, bukan hukum mutlak. Tetap harus ada penyesuaian dengan kebutuhan organisasi, terutama pada fitur yang memengaruhi akses web, ekstensi, remote access, autentikasi, sinkronisasi, dan data loss prevention.

---

## 2. Ruang Lingkup dan Asumsi Penting

Benchmark ini secara eksplisit mengasumsikan penggunaan **Google Chrome dan Google Update ADMX/ADML templates** dalam **Active Directory policy store**. Artinya, konteks paling idealnya adalah lingkungan Windows domain-joined yang dikelola melalui **Group Policy Object (GPO)**.

Konsekuensinya:

- Pengaturan utama diarahkan ke **Computer Configuration → Policies → Administrative Templates → Google**.
- Banyak rekomendasi memiliki registry backend di `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Google\Chrome` atau `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Google\Update`.
- Hasil policy dapat dilihat dari dalam browser melalui `chrome://policy/`.
- Untuk standalone/workgroup, Linux, atau macOS, konsep hardening tetap relevan, tetapi mekanisme implementasinya tidak selalu sama dengan isi benchmark.

Hal penting yang sering disalahpahami: banyak pengaturan Chrome sudah aman secara default, tetapi jika statusnya **Unset**, pengguna masih dapat mengubahnya. CIS mendorong organisasi untuk **mengunci** nilai tersebut agar tidak terjadi configuration drift.

---

## 3. Cara Membaca Struktur Rekomendasi

Setiap rekomendasi pada benchmark umumnya memiliki komponen berikut:

| Komponen | Makna konseptual |
|---|---|
| Title | Nama singkat konfigurasi yang ingin dicapai. |
| Assessment Status | Menunjukkan apakah penilaian dapat diautomasi atau perlu validasi manual. |
| Automated | Bisa dinilai secara teknis menjadi pass/fail, biasanya melalui registry atau policy. |
| Manual | Membutuhkan keputusan organisasi, pengecekan konteks, atau validasi fungsi. |
| Profile | Level keamanan yang dituju, umumnya L1 atau L2. |
| Description | Penjelasan fungsi setting Chrome. |
| Rationale | Alasan keamanan mengapa setting itu direkomendasikan. |
| Impact | Dampak operasional atau usability jika setting diterapkan. |
| Audit | Cara mengecek apakah konfigurasi sudah sesuai. |
| Remediation | Cara memperbaiki agar sesuai baseline. |
| Default Value | Nilai bawaan Chrome jika tidak dikelola policy. |
| CIS Controls | Pemetaan ke CIS Controls v7/v8 jika ada. |

Dalam praktik belajar, bagian paling penting adalah **Rationale** dan **Impact**. Rationale menjawab “mengapa ini aman?”, sedangkan Impact menjawab “apa yang mungkin rusak atau berubah setelah diterapkan?”.

---

## 4. Profile Definitions

### Level 1 (L1) — Corporate/Enterprise Environment

Level 1 adalah baseline umum untuk organisasi. Tujuannya praktis, masuk akal, memberi manfaat keamanan yang jelas, dan tidak mengganggu fungsi teknologi secara berlebihan. L1 cocok untuk mayoritas endpoint perusahaan, lab kampus, lingkungan administrasi, dan workstation pengguna umum.

### Level 2 (L2) — High Security/Sensitive Data Environment

Level 2 memperluas Level 1. L2 bukan profil mandiri; penerapannya berarti **L1 + L2**. Setting L2 biasanya lebih ketat, cocok untuk lingkungan dengan data sensitif, perangkat administrator, sistem kritis, jaringan terbatas, atau organisasi dengan toleransi risiko rendah.

Contoh dampak L2: fitur Bluetooth/USB via Web API diblokir, notifikasi dibatasi, cookies dibuat session-only, Incognito dinonaktifkan, DNS-over-HTTPS diwajibkan tanpa fallback, serta screen/audio/video capture dibatasi.

---

## 5. Konsep Utama dalam CIS Chrome

### 5.1 Policy Enforcement vs Default Setting

Perbedaan besar antara default biasa dan enforced policy adalah kendali. Default hanya berarti Chrome memiliki nilai bawaan tertentu, tetapi user dapat mengubahnya. Enforced policy berarti organisasi mengunci nilai tersebut melalui GPO atau registry policy sehingga user tidak dapat mengubahnya dari UI Chrome.

CIS banyak merekomendasikan setting yang tampaknya sama dengan default Chrome karena tujuan utamanya bukan sekadar memilih nilai default, melainkan mencegah perubahan oleh user, ekstensi, malware, atau kesalahan konfigurasi.

### 5.2 Browser sebagai Attack Surface

Chrome memiliki banyak fitur modern: ekstensi, WebUSB, WebBluetooth, WebRTC, file picker, remote debugging, password manager, sync, payment API, sensor API, serial API, native messaging, dan remote desktop. Semua fitur ini bermanfaat, tetapi juga menambah permukaan serangan.

Prinsip CIS adalah: fitur yang tidak diperlukan organisasi harus dibatasi, dinonaktifkan, atau hanya diizinkan melalui daftar yang jelas.

### 5.3 Privacy dan Data Exfiltration

Privasi dalam konteks enterprise bukan hanya masalah kenyamanan pengguna, tetapi juga perlindungan data organisasi. Telemetry, crash report, sync, search suggestion, translate service, spellcheck web service, payment method query, dan URL-keyed data collection dapat mengirim metadata ke pihak luar. Tidak semua fitur ini selalu berbahaya, tetapi harus diputuskan secara sadar.

### 5.4 Forensics dan Investigasi

Beberapa fitur seperti guest mode, ephemeral profile, disabling history, dan incognito dapat mengurangi jejak lokal. Dari sudut pandang privasi personal, fitur itu tampak menarik. Dari sudut pandang forensik organisasi, fitur tersebut dapat menyulitkan investigasi insiden. Karena itu CIS tidak selalu memaksimalkan privasi individual; CIS menyeimbangkan keamanan organisasi, auditability, dan kemampuan investigasi.

---

## 6. Pembahasan Konseptual per Bab

## Bab 1 — Enforced Defaults

Bab ini berisi setting yang banyak di antaranya merupakan default aman Chrome, tetapi tetap perlu ditegakkan secara enterprise. Tujuannya adalah mencegah pengguna atau proses lain mengubah default tersebut menjadi lebih lemah.

Fokus utamanya meliputi:

- Safe Browsing tetap aktif dan tidak dibypass sembarangan.
- Certificate Transparency tidak dilemahkan melalui allowlist legacy CA, URL, atau SPKI hash.
- HSTS dan insecure origin tidak diberi pengecualian yang membuka peluang downgrade.
- Background apps tidak terus berjalan setelah Chrome ditutup.
- Audio sandbox tetap aktif.
- Browser history tetap tersimpan untuk kebutuhan investigasi.
- WebRTC tidak mengekspos IP lokal secara tidak perlu.
- Komponen Chrome tetap diperbarui.
- Warning terkait command-line flags tetap tampil.
- Import data awal dari browser lain dibatasi agar konfigurasi atau data sensitif tidak ikut terbawa.

Secara konseptual, Bab 1 adalah fondasi agar Chrome tidak menjadi lingkungan yang mudah dimodifikasi oleh user atau aplikasi lain.

## Bab 2 — Attack Surface Reduction

Bab ini adalah bagian paling besar. Tujuannya mengurangi fitur Chrome yang dapat menjadi jalur serangan.

Tema pentingnya:

- **Update management:** Chrome dan komponennya harus diperbarui otomatis agar celah keamanan cepat tertutup.
- **Content settings:** mixed content, WebBluetooth, WebUSB, notifikasi, file access, dan storage partitioning dibatasi.
- **Extensions:** ekstensi eksternal, ekstensi tidak sah, Manifest v2, native messaging, dan unpublished extensions dikontrol ketat.
- **Remote access:** Chrome Remote Desktop dan fitur traversal relay/firewall dikunci agar tidak menjadi jalur akses jarak jauh tidak resmi.
- **TLS dan certificate handling:** user tidak boleh mudah melewati SSL warning atau Safe Browsing warning.
- **Site Isolation dan Renderer App Container:** proses render dipisahkan untuk membatasi dampak exploit browser.
- **Remote debugging:** dinonaktifkan karena bisa membuka kendali browser dari luar.
- **Proxy dan HTTP allowlist:** harus dikonfigurasi secara sadar, bukan auto-detect yang mudah disalahgunakan.

Bab ini sangat penting untuk endpoint enterprise karena banyak serangan modern terjadi melalui browser, ekstensi, konten web, dan API perangkat.

## Bab 3 — Privacy

Bab Privacy mengatur data apa saja yang boleh dikirim, disimpan, atau disinkronkan oleh Chrome.

Fokus utamanya:

- Geolocation default ditolak.
- Third-party cookies diblokir.
- Browser sign-in dan sync dikendalikan.
- Usage/crash reporting dinonaktifkan.
- Spellcheck web service, search suggestion, translate, alternate error pages, dan network prediction dibatasi.
- Google Cast dan payment method query dinonaktifkan jika tidak dibutuhkan.
- Safe Browsing tetap aktif, tetapi trusted-source bypass tidak dibiarkan.

Konsep dasarnya adalah **data minimization**: browser tidak perlu mengirim informasi tambahan ke layanan eksternal kecuali ada kebutuhan bisnis yang jelas.

## Bab 4 — Data Loss Prevention

Bab DLP membatasi saluran teknis yang dapat digunakan situs web atau browser untuk mengambil data dari perangkat pengguna.

Cakupannya:

- Screen capture, audio capture, dan video capture.
- Clipboard read/write.
- File selection dialog.
- Window Management API.
- Serial API dan Sensors API.
- DNS-over-HTTPS tanpa fallback insecure.
- Autofill alamat dan kartu kredit.
- Import saved password dari browser lain.
- Sinkronisasi password dikecualikan.

Bab ini lebih ketat dan banyak berisi L2 karena dapat mengganggu pengalaman pengguna. Namun, untuk perangkat yang menangani data sensitif, pembatasan ini sangat penting agar browser tidak menjadi kanal eksfiltrasi data.

## Bab 5 — Forensics (Post Incident)

Bab ini mendukung investigasi setelah insiden.

Fokusnya:

- Guest mode dimatikan agar pengguna tidak membuat sesi tanpa jejak profil utama.
- Incognito mode dinonaktifkan pada lingkungan high security.
- Disk cache size ditetapkan agar ada artefak yang bisa dianalisis setelah insiden.

Secara konseptual, Bab 5 menunjukkan bahwa keamanan bukan hanya pencegahan, tetapi juga kemampuan menjawab pertanyaan setelah insiden: siapa mengakses apa, kapan, dan jejak apa yang masih tersedia.

---

## 7. Ringkasan Seluruh Rekomendasi

### 1. Enforced Defaults

Penguncian default aman agar pengguna tidak dapat mengubah pengaturan bawaan Chrome menjadi lebih longgar.
| ID | Profil | Status | Ringkasan target konfigurasi | Hal. PDF |
|---|---|---|---|---|
| 1.1.1 | L1 | Automated | (L1) Ensure 'Cross-origin HTTP Authentication prompts' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 19 |
| 1.2.1 | L1 | Automated | (L1) Ensure 'Configure the list of domains on which Safe Browsing will not trigger warnings' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 22 |
| 1.2.2 | L1 | Manual | (L1) Ensure 'Safe Browsing Protection Level' is set to 'Enabled: Safe Browsing is active in the standard mode.' or higher (Manual)<br>**Target:** Safe Browsing is active in the standard mode. (1) or higher | 24 |
| 1.3 | L1 | Automated | (L1) Ensure 'Allow Google Cast to connect to Cast devices on all IP addresses' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 26 |
| 1.4 | L1 | Automated | (L1) Ensure 'Allow queries to a Google time service' is set to 'Enabled' (Automated)<br>**Target:** Enabled(1) | 28 |
| 1.5 | L1 | Automated | (L1) Ensure 'Allow the audio sandbox to run' is set to 'Enabled' (Automated)<br>**Target:** Enabled (1) | 30 |
| 1.6 | L1 | Automated | (L1) Ensure 'Ask where to save each file before downloading' is set to 'Enabled' (Automated)<br>**Target:** Enabled (1) | 32 |
| 1.7 | L1 | Automated | (L1) Ensure 'Continue running background apps when Google Chrome is closed' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 34 |
| 1.8 | L2 | Automated | (L2) Ensure 'Control SafeSites adult content filtering' is set to 'Enabled: Filter top level sites (but not embedded iframes) for adult content' (Automated)<br>**Target:** Enabled with a value of Filter top level sites (but not embedded iframes) for adult content (1) | 36 |
| 1.9 | L1 | Manual | (L1) Ensure 'Determine the availability of variations' is set to 'Enable all variations' (Manual)<br>**Target:** Enable all variations (0) | 38 |
| 1.10 | L1 | Automated | (L1) Ensure 'Disable Certificate Transparency enforcement for a list of Legacy Certificate Authorities' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 40 |
| 1.11 | L1 | Automated | (L1) Ensure 'Disable Certificate Transparency enforcement for a list of subjectPublicKeyInfo hashes' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 42 |
| 1.12 | L1 | Automated | (L1) Ensure 'Disable Certificate Transparency enforcement for a list of URLs' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 44 |
| 1.13 | L1 | Automated | (L1) Ensure 'Disable saving browser history' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 46 |
| 1.14 | L1 | Automated | (L1) Ensure 'DNS interception checks enabled' is set to 'Enabled' (Automated)<br>**Target:** Enabled (1) | 48 |
| 1.15 | L1 | Automated | (L1) Ensure 'Enable component updates in Google Chrome' is set to 'Enabled' (Automated)<br>**Target:** Enabled (1) | 50 |
| 1.16 | L1 | Automated | (L1) Ensure 'Enable globally scoped HTTP auth cache' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 52 |
| 1.17 | L1 | Automated | (L1) Ensure 'Enable online OCSP/CRL checks' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 54 |
| 1.18 | L1 | Automated | (L1) Ensure 'Enable security warnings for command-line flags' is set to 'Enabled' (Automated)<br>**Target:** Enabled (0) | 56 |
| 1.19 | L1 | Automated | (L1) Ensure 'Enable third party software injection blocking' is set to 'Enabled' (Automated)<br>**Target:** Enabled (1) | 58 |
| 1.20 | L1 | Automated | (L1) Ensure 'Enables managed extensions to use the Enterprise Hardware Platform API' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 60 |
| 1.21 | L1 | Automated | (L1) Ensure 'Ephemeral profile' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 62 |
| 1.22 | L1 | Automated | (L1) Ensure 'Import autofill form data from default browser on first run' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 64 |
| 1.23 | L1 | Automated | (L1) Ensure 'Import of homepage from default browser on first run' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 66 |
| 1.24 | L1 | Automated | (L1) Ensure 'Import search engines from default browser on first run' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 68 |
| 1.25 | L1 | Automated | (L1) Ensure 'List of names that will bypass the HSTS policy check' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 70 |
| 1.26 | L1 | Automated | (L1) Ensure 'Origins or hostname patterns for which restrictions on insecure origins should not apply' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 72 |
| 1.27 | L1 | Automated | (L1) Ensure 'Suppress lookalike domain warnings on domains' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 74 |
| 1.28 | L1 | Automated | (L1) Ensure 'Suppress the unsupported OS warning' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 76 |
| 1.29 | L1 | Automated | (L1) Ensure 'URLs for which local IPs are exposed in WebRTC ICE candidates' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 78 |

### 2. Attack Surface Reduction

Pengurangan permukaan serangan browser: update, ekstensi, remote access, insecure content, TLS, proxy, Web APIs, dan debugging.
| ID | Profil | Status | Ringkasan target konfigurasi | Hal. PDF |
|---|---|---|---|---|
| 2.1.1 | L1 | Automated | (L1) Ensure 'Update policy override' is set to 'Enabled' with 'Always allow updates (recommended)' or 'Automatic silent updates' specified (Automated)<br>**Target:** Enabled with a value of Always allow updates (1) or Automatic silent updates (3) | 82 |
| 2.1.2 | L1 | Automated | (L1) Ensure 'Auto-update check period override' is set to any value except '0' (Automated)<br>**Target:** any value except 0. | 84 |
| 2.2.1 | L1 | Automated | (L1) Ensure 'Control use of insecure content exceptions' is set to 'Enabled: Do not allow any site to load mixed content' (Automated)<br>**Target:** Enabled with the value of Do not allow any site to load mixed content (2) | 86 |
| 2.2.2 | L2 | Automated | (L2) Ensure 'Control use of the Web Bluetooth API' is set to 'Enabled: Do not allow any site to request access to Bluetooth devices via the Web Bluetooth API' (Automated)<br>**Target:** Enabled with a value of Do not allow any site to request access to Bluetooth devices via the Web Bluetooth API (2) | 88 |
| 2.2.3 | L2 | Automated | (L2) Ensure 'Control use of the WebUSB API' is set to 'Enabled: Do not allow any site to request access to USB devices via the WebUSB API' (Automated)<br>**Target:** Enabled with a value of Do not allow any site to request access to USB devices via the WebUSB API (2) | 90 |
| 2.2.4 | L2 | Automated | (L2) Ensure 'Default notification setting' is set to 'Enabled: Do not allow any site to show desktop notifications' (Automated)<br>**Target:** Enabled with a value of Do not allow any site to show desktop notifications (2) | 92 |
| 2.2.5 | L1 | Automated | (L1) Ensure 'Allow local file access to file:// URLs on these sites in the PDF Viewer' Is Disabled (Automated)<br>**Target:** Disabled | 94 |
| 2.3.1 | L1 | Automated | (L1) Ensure 'Blocks external extensions from being installed' is set to 'Enabled' (Automated)<br>**Target:** Enabled (1) | 97 |
| 2.3.2 | L1 | Automated | (L1) Ensure 'Configure allowed app/extension types' is set to 'Enabled: extension, hosted_app, platform_app, theme' (Automated)<br>**Target:** Enabled with the values of extension, hosted_app, platform_app, theme. | 99 |
| 2.3.3 | L1 | Automated | (L1) Ensure 'Configure extension installation blocklist' is set to 'Enabled: *' (Automated)<br>**Target:** Enabled with a value of * | 101 |
| 2.3.4 | L2 | Automated | (L2) Ensure 'Default third-party storage partitioning setting' Is Enabled and Blocked (Automated)<br>**Target:** Enabled | 103 |
| 2.3.5 | L1 | Manual | (L1) Ensure 'Block third-party storage partitioning for these origins' Is Configured (Manual)<br>**Target:** Configured per organization | 105 |
| 2.3.6 | L2 | Automated | (L2) Ensure 'Control Manifest v2 extension availability' Is Set to Forced Only (Automated)<br>**Target:** - | 107 |
| 2.3.7 | L1 | Automated | (L1) Ensure 'Control availability of extensions unpublished on the Chrome Web Store' Is Disabled (Automated)<br>**Target:** Disabled | 109 |
| 2.4.1 | L2 | Automated | (L2) Ensure 'Supported authentication schemes' is set to 'Enabled: ntlm, negotiate' (Automated)<br>**Target:** Enabled with the value of ntlm, negotiate | 112 |
| 2.5.1 | L2 | Automated | (L2) Ensure 'Configure native messaging blocklist' is set to 'Enabled: *' (Automated)<br>**Target:** Enabled with a value of * | 115 |
| 2.6.1 | L1 | Manual | (L1) Ensure 'Enable saving passwords to the password manager' is Explicitly Configured (Manual)<br>**Target:** Explicitly set to Enabled (1) or Disabled (0) based on the organization's needs. | 118 |
| 2.7.1 | L1 | Automated | (L1) Ensure 'Enable Google Cloud Print Proxy' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 121 |
| 2.8.1 | L1 | Manual | Ensure 'Allow remote access connections to this machine' is set to 'Disabled' (Manual)<br>**Target:** Disabled (0) | 124 |
| 2.8.2 | L1 | Automated | (L1) Ensure 'Allow remote users to interact with elevated windows in remote assistance sessions' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 126 |
| 2.8.3 | L1 | Manual | (L1) Ensure 'Configure the required domain names for remote access clients' is set to 'Enabled' with a domain defined (Manual)<br>**Target:** Enabled (1) and at least one domain set | 128 |
| 2.8.4 | L1 | Automated | (L1) Ensure 'Enable curtaining of remote access hosts' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 130 |
| 2.8.5 | L1 | Automated | (L1) Ensure 'Enable firewall traversal from remote access host' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 132 |
| 2.8.6 | L1 | Automated | (L1) Ensure 'Enable or disable PIN-less authentication for remote access hosts' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 134 |
| 2.8.7 | L1 | Automated | (L1) Ensure 'Enable the use of relay servers by the remote access host' is set to 'Disabled'. (Automated)<br>**Target:** Disabled (0) | 136 |
| 2.9.1 | L1 | Manual | (L1) Ensure 'Enable First-Party Sets' Is Disabled (Manual)<br>**Target:** Disabled | 139 |
| 2.10.1 | L1 | Manual | (L1) Ensure 'Allow automatic sign-in to Microsoft cloud identity providers' Is Enabled (Manual)<br>**Target:** Enabled | 142 |
| 2.11 | L1 | Automated | (L1) Ensure 'Allow download restrictions' is set to 'Enabled: Block malicious downloads' (Automated)<br>**Target:** Enabled with a value of Block malicious downloads. Recommended. (4) | 144 |
| 2.12 | L2 | Automated | (L2) Ensure 'Allow proceeding from the SSL warning page' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 146 |
| 2.13 | L1 | Automated | (L1) Ensure 'Disable proceeding from the Safe Browsing warning page' is set to 'Enabled' (Automated)<br>**Target:** Enabled (1) | 148 |
| 2.14 | L1 | Automated | (L1) Ensure 'Require Site Isolation for every site' is set to 'Enabled' (Automated)<br>**Target:** Enabled (1) | 150 |
| 2.15 | L2 | Automated | (L2) Ensure 'Force Google SafeSearch' is set to 'Enabled' (Automated)<br>**Target:** Enabled (1) | 152 |
| 2.16 | L1 | Automated | (L1) Ensure 'Notify a user that a browser relaunch or device restart is recommended or required' is set to 'Enabled: Show a recurring prompt to the user indication that a relaunch is required' (Automated)<br>**Target:** Enabled with a value of Show a recurring prompt to the user indicating that a relaunch is required (2) | 154 |
| 2.17 | L1 | Automated | (L1) Ensure 'Proxy settings' is set to 'Enabled' and does not contain "ProxyMode": "auto_detect" (Automated)<br>**Target:** Enabled and the value of ProxyMode is not set to auto_detect | 156 |
| 2.18 | L2 | Automated | (L2) Ensure 'Require online OCSP/CRL checks for local trust anchors' is set to 'Enabled' (Automated)<br>**Target:** Enabled (1) | 158 |
| 2.19 | L1 | Automated | (L1) Ensure 'Set the time period for update notifications' is set to 'Enabled: 86400000' (Automated)<br>**Target:** Enabled with value 86400000 (1 day) | 160 |
| 2.20 | L1 | Automated | (L1) Ensure 'Allow Web Authentication requests on sites with broken TLS certificates' Is Disabled (Automated)<br>**Target:** Disabled | 162 |
| 2.21 | L1 | Automated | (L1) Ensure 'Allow reporting of domain reliability related data' Is Disabled (Automated)<br>**Target:** Disabled | 164 |
| 2.22 | L1 | Automated | (L1) Ensure 'Enable TLS Encrypted ClientHello' Is Enabled (Automated)<br>**Target:** Enabled | 166 |
| 2.23 | L2 | Automated | (L2) Ensure 'Determines whether the built-in certificate verifier will enforce constraints encoded into trust anchors loaded from the platform trust store' Is Enabled (Automated)<br>**Target:** Enabled | 168 |
| 2.24 | L1 | Automated | (L1) Ensure 'Keep browsing data when creating enterprise profile by default' Is Enabled (Automated)<br>**Target:** Enabled | 170 |
| 2.25 | L1 | Automated | (L1) Ensure 'Allow file or directory picker APIs to be called without prior user gesture' Is Disabled (Automated)<br>**Target:** Disabled | 172 |
| 2.26 | L1 | Automated | (L1) Ensure 'Enable Google Search Side Panel' Is Disabled (Automated)<br>**Target:** Disabled | 174 |
| 2.27 | L1 | Manual | (L1) Ensure 'Http Allowlist' Is Properly Configured (Manual)<br>**Target:** - | 176 |
| 2.28 | L1 | Automated | (L1) Ensure 'Enable automatic HTTPS upgrades' Is Enabled (Automated)<br>**Target:** Enabled | 178 |
| 2.29 | L1 | Automated | (L1) Ensure 'Insecure Hashes in TLS Handshakes Enabled' Is Disabled (Automated)<br>**Target:** Disabled | 180 |
| 2.30 | L1 | Automated | (L1) Ensure 'Enable Renderer App Container' Is Enabled (Automated)<br>**Target:** Enabled | 182 |
| 2.31 | L1 | Automated | (L1) Ensure 'Enable strict MIME type checking for worker scripts' Is Enabled (Automated)<br>**Target:** Enabled | 184 |
| 2.32 | L1 | Automated | Ensure 'Allow remote debugging' is set to 'Disabled' (Automated)<br>**Target:** Disabled. | 186 |

### 3. Privacy

Pengendalian data yang keluar dari browser: lokasi, cookies, sync, telemetry, suggestion, translate, dan identitas browser.
| ID | Profil | Status | Ringkasan target konfigurasi | Hal. PDF |
|---|---|---|---|---|
| 3.1.1 | L2 | Automated | (L2) Ensure 'Default cookies setting' is set to 'Enabled: Keep cookies for the duration of the session' (Automated)<br>**Target:** Enabled with a value of Keep cookies for the duration of the session (4) | 190 |
| 3.1.2 | L1 | Automated | (L1) Ensure 'Default geolocation setting' is set to 'Enabled: Do not allow any site to track the users' physical location' (Automated)<br>**Target:** Enabled with a value Do not allow any site to track the users' physical location (2) | 192 |
| 3.2.1 | L1 | Automated | (L1) Ensure 'Enable Google Cast' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 195 |
| 3.3 | L1 | Automated | (L1) Ensure 'Allow websites to query for available payment methods' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 197 |
| 3.4 | L1 | Automated | (L1) Ensure 'Block third party cookies' is set to 'Enabled' (Automated)<br>**Target:** Enabled (1) | 199 |
| 3.5 | L2 | Automated | (L2) Ensure 'Browser sign in settings' is set to 'Enabled: Disabled browser sign-in' (Automated)<br>**Target:** Enabled with a value of Disable browser sign-in (0) | 201 |
| 3.6 | L1 | Automated | (L1) Ensure 'Control how Chrome Cleanup reports data to Google' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 203 |
| 3.7 | L1 | Automated | (L1) Ensure 'Disable synchronization of data with Google' is set to 'Enabled' (Automated)<br>**Target:** Enabled (1) | 205 |
| 3.8 | L1 | Automated | (L1) Ensure 'Enable alternate error pages' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 207 |
| 3.9 | L1 | Automated | (L1) Ensure 'Enable deleting browser and download history' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 209 |
| 3.10 | L1 | Automated | (L1) Ensure 'Enable predict network actions` is set to 'Enabled: Do not predict actions on any network connection' (Automated)<br>**Target:** Enabled with a value of Do not predict network actions on any network connection (2) | 211 |
| 3.11 | L1 | Automated | (L1) Ensure 'Enable or disable spell checking web service' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 213 |
| 3.12 | L1 | Automated | (L1) Ensure 'Enable reporting of usage and crash-related data' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 215 |
| 3.13 | L1 | Automated | (L1) Ensure 'Enable Safe Browsing for trusted sources' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 217 |
| 3.14 | L2 | Automated | (L2) Ensure 'Enable search suggestions' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 219 |
| 3.15 | L2 | Automated | (L2) Ensure 'Enable Translate' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 221 |
| 3.16 | L1 | Automated | (L1) Ensure 'Enable URL-keyed anonymized data collection' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 223 |

### 4. Data Loss Prevention

Pembatasan kanal keluarnya data: clipboard, screen/audio/video capture, file picker, serial/sensor API, DNS-over-HTTPS, autofill, dan sinkronisasi password.
| ID | Profil | Status | Ringkasan target konfigurasi | Hal. PDF |
|---|---|---|---|---|
| 4.2.1 | L2 | Automated | (L2) Ensure 'Control use of the Serial API' is set to 'Enabled: Do not allow any site to request access to serial ports via the Serial API' (Automated)<br>**Target:** Do not allow any site to request access to serial ports via the Serial API (2) | 230 |
| 4.2.2 | L2 | Automated | (L2) Ensure 'Default Sensors Setting' is set to 'Enabled: Do not allow any site to access sensors' (Automated)<br>**Target:** Do not allow any site to access sensors (2) The recommended state for this setting is: Enabled with a value of Do not allow any site to access sensors | 232 |
| 4.2.3 | L1 | Manual | (L1) Ensure 'Allow clipboard for these sites' Is Configured (Manual)<br>**Target:** Configured per organization | 234 |
| 4.2.4 | L1 | Manual | (L1) Ensure 'Block clipboard on these sites' Is Configured (Manual)<br>**Target:** Configured per organization | 236 |
| 4.2.5 | L1 | Automated | (L1) Ensure 'Default clipboard setting' Is 'Enabled' to 'Deny Permissions' (Automated)<br>**Target:** - | 238 |
| 4.2.6 | L2 | Automated | (L2) Ensure 'Default Window Management permissions setting' Is 'Enabled' to 'Deny Permission' (Automated)<br>**Target:** - | 240 |
| 4.2.7 | L2 | Manual | (L2) Ensure 'Allow Window Management permission on these sites' Is Configured (Manual)<br>**Target:** Configured per organization | 242 |
| 4.2.8 | L2 | Manual | (L2) Ensure 'Block Window Management permission on these sites' Is Configured (Manual)<br>**Target:** Configured per organization | 244 |
| 4.3 | L2 | Automated | (L2) Ensure 'Allow invocation of file selection dialogs' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 246 |
| 4.4 | L2 | Automated | (L2) Ensure 'Allow or deny audio capture' is set to 'Disabled' (Automated)<br>**Target:** Disabled | 248 |
| 4.5 | L2 | Automated | (L2) Ensure 'Allow or deny video capture' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 250 |
| 4.6 | L1 | Automated | (L1) Ensure 'Allow user feedback' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 252 |
| 4.7 | L2 | Automated | (L2) Ensure 'Controls the mode of DNS-over-HTTPS' is set to 'Enabled: DNS-over-HTTPS without insecure fallback' (Automated)<br>**Target:** Enabled with a value of Enable DNS-over- HTTPS without insecure fallback (secure) | 254 |
| 4.8 | L2 | Automated | (L2) Ensure 'Enable AutoFill for addresses' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 256 |
| 4.9 | L1 | Automated | (L1) Ensure 'Enable AutoFill for credit cards' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 258 |
| 4.10 | L1 | Automated | (L1) Ensure 'Import saved passwords from default browser on first run' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 260 |
| 4.11 | L1 | Automated | (L1) Ensure 'List of types that should be excluded from synchronization' is set to 'Enabled: passwords' (Automated)<br>**Target:** Enabled with the following text value passwords (Case Sensitive) | 262 |
| 4.12 | L2 | Automated | (L2) Ensure 'Allow or deny screen capture' is set to 'Disabled' (Automated)<br>**Target:** Disabled | 264 |

### 5. Forensics (Post Incident)

Menjaga jejak investigasi pascainsiden: guest mode, incognito, dan cache lokal.
| ID | Profil | Status | Ringkasan target konfigurasi | Hal. PDF |
|---|---|---|---|---|
| 5.1 | L2 | Automated | (L2) Ensure 'Enable guest mode in browser' is set to 'Disabled' (Automated)<br>**Target:** Disabled (0) | 267 |
| 5.2 | L2 | Automated | (L2) Ensure 'Incognito mode availability' is set to 'Enabled: Incognito mode disabled' (Automated)<br>**Target:** Enabled: Incognito mode disabled (1) | 269 |
| 5.3 | L1 | Automated | (L1) Ensure 'Set disk cache size, in bytes' is set to 'Enabled: 250609664' (Automated)<br>**Target:** Enabled: 250609664 or greater | 271 |



---

## 8. Catatan Penerapan

1. Mulai dari **L1** untuk baseline umum.
2. Terapkan **L2** hanya pada endpoint sensitif atau setelah uji dampak.
3. Jangan langsung menerapkan semua setting ke produksi tanpa pilot.
4. Catat exception untuk aplikasi web lama, ekstensi wajib, remote support, proxy, atau fitur yang memang dibutuhkan.
5. Gunakan `chrome://policy/` untuk melihat policy yang benar-benar aktif di browser.
6. Gunakan GPO/MDM/management tool agar konfigurasi konsisten dan tidak bergantung pada user.

---

## 9. Template Exception Register

| ID Rekomendasi | Setting | Alasan Exception | Risiko | Mitigasi | Pemilik Risiko | Tanggal Review |
|---|---|---|---|---|---|---|
| Contoh: 2.12 | Allow proceeding from SSL warning page | Aplikasi internal masih memakai sertifikat lama | User dapat mengakses situs TLS bermasalah | Migrasi sertifikat internal ke CA valid | Tim Infrastruktur | YYYY-MM-DD |

