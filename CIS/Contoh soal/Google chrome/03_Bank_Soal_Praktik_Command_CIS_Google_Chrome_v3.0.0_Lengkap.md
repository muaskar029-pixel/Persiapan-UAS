# Bank Soal Praktik Command CIS Google Chrome Benchmark v3.0.0 — Lengkap

**Fokus:** latihan praktik command, policy registry, GPO, audit, validasi, dan pemilihan jawaban jamak berbasis CIS Google Chrome Benchmark v3.0.0.

> **Aturan latihan:** pilih semua jawaban yang benar. Jika tidak ada opsi yang benar, jawab **PASS / kosongkan jawaban**.

## Cakupan

- Implementasi GPO, ADMX/ADML, registry policy, dan validasi `chrome://policy/`.
- Bab 1 Enforced Defaults.
- Bab 2 Attack Surface Reduction.
- Bab 3 Privacy.
- Bab 4 Data Loss Prevention.
- Bab 5 Forensics / Post Incident.
- Soal jebakan command salah dan skenario PASS.

## Catatan Praktik

- Contoh command PowerShell menggunakan variabel:

```powershell
$Chrome = "HKLM:\SOFTWARE\Policies\Google\Chrome"
$Update = "HKLM:\SOFTWARE\Policies\Google\Update"
```

- Contoh helper function yang diasumsikan sudah tersedia:

```powershell
Set-ChromePolicyDword
Set-ChromePolicyString
Set-ChromePolicyListItem
```

- Untuk domain enterprise, GPO tetap lebih disarankan daripada registry lokal manual.


## 0. Cara Menjawab dan Validasi Dasar

### Soal 1

**Pertanyaan:** Dalam latihan pilihan ganda jamak, apa jawaban yang benar jika semua opsi command/policy salah?

A. Pilih opsi yang paling mendekati benar
B. Pilih semua opsi agar tidak kosong
C. Jawab PASS / kosongkan jawaban
D. Pilih opsi A sebagai default

**Jawaban benar:** C

**Pembahasan:** Jika seluruh opsi salah, jawaban yang benar adalah PASS/kosong. Pola ini melatih ketelitian agar tidak memilih command yang terlihat benar tetapi sebenarnya melemahkan hardening.

---

### Soal 2

**Pertanyaan:** Metode implementasi utama CIS Google Chrome Benchmark pada endpoint Windows enterprise adalah...

A. Menerapkan Group Policy Object memakai Google Chrome ADMX/ADML dan Google Update ADMX/ADML
B. Mengubah setting dari UI Chrome satu per satu di setiap laptop user
C. Menghapus Chrome lalu memakai browser portable tanpa policy
D. Menggunakan GPO lalu memvalidasi di `chrome://policy/`

**Jawaban benar:** A, D

**Pembahasan:** Dokumen implementasi menekankan GPO/ADMX sebagai metode utama. UI Chrome bukan cara enterprise karena user preference dapat berubah dan sulit diaudit.

---

### Soal 3

**Pertanyaan:** Command atau lokasi mana yang benar untuk memvalidasi policy Chrome setelah GPO/registry policy diterapkan?

A. `chrome://policy/` lalu klik Reload policies
B. `Get-ItemProperty "HKLM:\SOFTWARE\Policies\Google\Chrome"`
C. `Get-ItemProperty "HKLM:\SOFTWARE\Policies\Google\Update"`
D. `about:config`

**Jawaban benar:** A, B, C

**Pembahasan:** `chrome://policy/` memperlihatkan policy aktif di browser. Registry HKLM Google\Chrome dan Google\Update menyimpan policy machine. `about:config` adalah konsep Firefox, bukan Chrome.

---

### Soal 4

**Pertanyaan:** Command mana yang benar untuk memaksa pembaruan policy di endpoint Windows domain-joined?

A. `gpupdate /force`
B. `chrome://policy/` lalu Reload policies
C. `apt update && apt upgrade`
D. `Set-ExecutionPolicy Bypass` sebagai syarat wajib CIS Chrome

**Jawaban benar:** A, B

**Pembahasan:** `gpupdate /force` memaksa Group Policy refresh. `chrome://policy/` dapat memuat ulang policy di Chrome. APT bukan mekanisme Windows/Chrome GPO.

---

### Soal 5

**Pertanyaan:** PowerShell helper mana yang benar untuk membuat path registry jika belum ada lalu menulis policy DWORD?

A. `New-Item -Path $Path -Force | Out-Null` kemudian `New-ItemProperty -Path $Path -Name $Name -PropertyType DWord -Value $Value -Force | Out-Null`
B. `Remove-Item -Path $Path -Recurse -Force` lalu `New-ItemProperty ...`
C. `New-ItemProperty -Path $Path -Name $Name -PropertyType String -Value $Value` untuk semua policy DWORD
D. `reg delete HKLM\SOFTWARE\Policies\Google\Chrome /f` sebelum setiap policy

**Jawaban benar:** A

**Pembahasan:** Helper yang benar membuat path bila belum ada dan menulis nilai dengan tipe DWORD. Menghapus path atau memakai tipe String untuk policy DWORD bukan implementasi yang aman/tepat.

---

### Soal 6

**Pertanyaan:** Path registry mana yang benar untuk policy machine Google Chrome dan Google Update pada Windows?

A. `HKLM:\SOFTWARE\Policies\Google\Chrome`
B. `HKLM:\SOFTWARE\Policies\Google\Update`
C. `HKCU:\Software\Mozilla\Firefox`
D. `HKLM:\SYSTEM\CurrentControlSet\Services\Chrome`

**Jawaban benar:** A, B

**Pembahasan:** Policy enterprise pada dokumen memakai HKLM untuk Google Chrome dan Google Update. HKCU/Firefox/service registry bukan baseline policy yang dibahas.

---

### Soal 7

**Pertanyaan:** File ADMX/ADML mana yang benar disiapkan pada Central Store untuk Chrome dan Google Update?

A. `chrome.admx` dan file bahasa `chrome.adml`
B. `google.admx` dan file bahasa `google.adml`
C. `GoogleUpdate.admx` dan file bahasa `GoogleUpdate.adml`
D. `firefox.admx` sebagai syarat wajib Chrome

**Jawaban benar:** A, B, C

**Pembahasan:** Implementasi memerlukan template Chrome, Google, dan Google Update. Firefox ADMX bukan bagian CIS Google Chrome.

---

### Soal 8

**Pertanyaan:** GPO mana yang paling rapi untuk baseline CIS Chrome?

A. Membuat GPO terpisah seperti `CIS - Google Chrome v3.0.0 - L1 Baseline`
B. Membuat GPO add-on L2 terpisah untuk endpoint sensitif
C. Menerapkan semua L1 dan L2 langsung ke Domain Controllers tanpa pilot
D. Mengubah registry lokal semua user tanpa dokumentasi

**Jawaban benar:** A, B

**Pembahasan:** Pemisahan L1 dan L2 membuat penerapan lebih fleksibel dan aman. Produksi harus melalui pilot, bukan langsung seluruh domain.

---

### Soal 9

**Pertanyaan:** Command mana yang benar untuk rollback registry policy lokal Chrome di lab?

A. `Remove-Item "HKLM:\SOFTWARE\Policies\Google\Chrome" -Recurse -Force`
B. `gpupdate /force` setelah mengubah/menghapus policy
C. `Remove-Item "C:\Windows\System32" -Recurse -Force`
D. `reg add HKLM\SOFTWARE\Policies\Google\Chrome /v AllowAll /t REG_DWORD /d 1`

**Jawaban benar:** A, B

**Pembahasan:** Rollback lab dapat menghapus key policy Chrome lalu refresh policy/restart Chrome. Opsi destruktif atau policy fiktif tidak benar.

---

### Soal 10

**Pertanyaan:** Bukti audit yang tepat setelah policy diterapkan adalah...

A. Screenshot/ekspor tampilan `chrome://policy/`
B. Output `Get-ItemProperty` dari registry policy
C. Catatan OU/GPO tempat policy diterapkan
D. Riwayat browsing pribadi user sebagai satu-satunya bukti

**Jawaban benar:** A, B, C

**Pembahasan:** Bukti audit harus menunjukkan policy aktif, registry/policy backend, dan cakupan GPO. Riwayat browsing user bukan bukti konfigurasi baseline.

---


## 1. Enforced Defaults

### Soal 11

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan cross-origin HTTP authentication prompts melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "AllowCrossOriginAuthPrompt" 0`
B. `Set-ChromePolicyDword $Chrome "AllowCrossOriginAuthPrompt" 1`
C. `Set-ChromePolicyString $Chrome "AllowCrossOriginAuthPrompt" "0"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "AllowCrossOriginAuthPrompt" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `AllowCrossOriginAuthPrompt` pada baseline implementasi ditulis sebagai DWORD bernilai `0` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 12

**Pertanyaan:** PowerShell mana yang benar untuk mengaktifkan Safe Browsing standard mode atau lebih tinggi melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "SafeBrowsingProtectionLevel" 1`
B. `Set-ChromePolicyDword $Chrome "SafeBrowsingProtectionLevel" 0`
C. `Set-ChromePolicyString $Chrome "SafeBrowsingProtectionLevel" "1"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "SafeBrowsingProtectionLevel" 1`

**Jawaban benar:** A

**Pembahasan:** Policy `SafeBrowsingProtectionLevel` pada baseline implementasi ditulis sebagai DWORD bernilai `1` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 13

**Pertanyaan:** PowerShell mana yang benar untuk mencegah Google Cast terhubung ke semua alamat IP melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "MediaRouterCastAllowAllIPs" 0`
B. `Set-ChromePolicyDword $Chrome "MediaRouterCastAllowAllIPs" 1`
C. `Set-ChromePolicyString $Chrome "MediaRouterCastAllowAllIPs" "0"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "MediaRouterCastAllowAllIPs" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `MediaRouterCastAllowAllIPs` pada baseline implementasi ditulis sebagai DWORD bernilai `0` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 14

**Pertanyaan:** PowerShell mana yang benar untuk mengizinkan query time service yang dibutuhkan Chrome melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "BrowserNetworkTimeQueriesEnabled" 1`
B. `Set-ChromePolicyDword $Chrome "BrowserNetworkTimeQueriesEnabled" 0`
C. `Set-ChromePolicyString $Chrome "BrowserNetworkTimeQueriesEnabled" "1"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "BrowserNetworkTimeQueriesEnabled" 1`

**Jawaban benar:** A

**Pembahasan:** Policy `BrowserNetworkTimeQueriesEnabled` pada baseline implementasi ditulis sebagai DWORD bernilai `1` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 15

**Pertanyaan:** PowerShell mana yang benar untuk memastikan audio sandbox aktif melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "AudioSandboxEnabled" 1`
B. `Set-ChromePolicyDword $Chrome "AudioSandboxEnabled" 0`
C. `Set-ChromePolicyString $Chrome "AudioSandboxEnabled" "1"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "AudioSandboxEnabled" 1`

**Jawaban benar:** A

**Pembahasan:** Policy `AudioSandboxEnabled` pada baseline implementasi ditulis sebagai DWORD bernilai `1` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 16

**Pertanyaan:** PowerShell mana yang benar untuk meminta lokasi penyimpanan setiap download melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "PromptForDownloadLocation" 1`
B. `Set-ChromePolicyDword $Chrome "PromptForDownloadLocation" 0`
C. `Set-ChromePolicyString $Chrome "PromptForDownloadLocation" "1"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "PromptForDownloadLocation" 1`

**Jawaban benar:** A

**Pembahasan:** Policy `PromptForDownloadLocation` pada baseline implementasi ditulis sebagai DWORD bernilai `1` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 17

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan background apps setelah Chrome ditutup melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "BackgroundModeEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "BackgroundModeEnabled" 1`
C. `Set-ChromePolicyString $Chrome "BackgroundModeEnabled" "0"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "BackgroundModeEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `BackgroundModeEnabled` pada baseline implementasi ditulis sebagai DWORD bernilai `0` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 18

**Pertanyaan:** PowerShell mana yang benar untuk mengaktifkan variations sesuai baseline implementasi melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "ChromeVariations" 0`
B. `Set-ChromePolicyDword $Chrome "ChromeVariations" 1`
C. `Set-ChromePolicyString $Chrome "ChromeVariations" "0"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "ChromeVariations" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `ChromeVariations` pada baseline implementasi ditulis sebagai DWORD bernilai `0` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 19

**Pertanyaan:** PowerShell mana yang benar untuk tidak menonaktifkan penyimpanan history agar artefak investigasi tetap ada melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "SavingBrowserHistoryDisabled" 0`
B. `Set-ChromePolicyDword $Chrome "SavingBrowserHistoryDisabled" 1`
C. `Set-ChromePolicyString $Chrome "SavingBrowserHistoryDisabled" "0"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "SavingBrowserHistoryDisabled" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `SavingBrowserHistoryDisabled` pada baseline implementasi ditulis sebagai DWORD bernilai `0` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 20

**Pertanyaan:** PowerShell mana yang benar untuk mengaktifkan DNS interception checks melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "DNSInterceptionChecksEnabled" 1`
B. `Set-ChromePolicyDword $Chrome "DNSInterceptionChecksEnabled" 0`
C. `Set-ChromePolicyString $Chrome "DNSInterceptionChecksEnabled" "1"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "DNSInterceptionChecksEnabled" 1`

**Jawaban benar:** A

**Pembahasan:** Policy `DNSInterceptionChecksEnabled` pada baseline implementasi ditulis sebagai DWORD bernilai `1` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 21

**Pertanyaan:** PowerShell mana yang benar untuk mengaktifkan update komponen Chrome melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "ComponentUpdatesEnabled" 1`
B. `Set-ChromePolicyDword $Chrome "ComponentUpdatesEnabled" 0`
C. `Set-ChromePolicyString $Chrome "ComponentUpdatesEnabled" "1"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "ComponentUpdatesEnabled" 1`

**Jawaban benar:** A

**Pembahasan:** Policy `ComponentUpdatesEnabled` pada baseline implementasi ditulis sebagai DWORD bernilai `1` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 22

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan globally scoped HTTP auth cache melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "GloballyScopeHTTPAuthCacheEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "GloballyScopeHTTPAuthCacheEnabled" 1`
C. `Set-ChromePolicyString $Chrome "GloballyScopeHTTPAuthCacheEnabled" "0"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "GloballyScopeHTTPAuthCacheEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `GloballyScopeHTTPAuthCacheEnabled` pada baseline implementasi ditulis sebagai DWORD bernilai `0` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 23

**Pertanyaan:** PowerShell mana yang benar untuk mengikuti baseline implementasi untuk online OCSP/CRL checks melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "EnableOnlineRevocationChecks" 0`
B. `Set-ChromePolicyDword $Chrome "EnableOnlineRevocationChecks" 1`
C. `Set-ChromePolicyString $Chrome "EnableOnlineRevocationChecks" "0"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "EnableOnlineRevocationChecks" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `EnableOnlineRevocationChecks` pada baseline implementasi ditulis sebagai DWORD bernilai `0` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 24

**Pertanyaan:** PowerShell mana yang benar untuk mengaktifkan warning untuk command-line flags berisiko melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "CommandLineFlagSecurityWarningsEnabled" 1`
B. `Set-ChromePolicyDword $Chrome "CommandLineFlagSecurityWarningsEnabled" 0`
C. `Set-ChromePolicyString $Chrome "CommandLineFlagSecurityWarningsEnabled" "1"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "CommandLineFlagSecurityWarningsEnabled" 1`

**Jawaban benar:** A

**Pembahasan:** Policy `CommandLineFlagSecurityWarningsEnabled` pada baseline implementasi ditulis sebagai DWORD bernilai `1` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 25

**Pertanyaan:** PowerShell mana yang benar untuk mengaktifkan third-party software injection blocking melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "ThirdPartyBlockingEnabled" 1`
B. `Set-ChromePolicyDword $Chrome "ThirdPartyBlockingEnabled" 0`
C. `Set-ChromePolicyString $Chrome "ThirdPartyBlockingEnabled" "1"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "ThirdPartyBlockingEnabled" 1`

**Jawaban benar:** A

**Pembahasan:** Policy `ThirdPartyBlockingEnabled` pada baseline implementasi ditulis sebagai DWORD bernilai `1` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 26

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan Enterprise Hardware Platform API untuk managed extensions melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "EnterpriseHardwarePlatformAPIEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "EnterpriseHardwarePlatformAPIEnabled" 1`
C. `Set-ChromePolicyString $Chrome "EnterpriseHardwarePlatformAPIEnabled" "0"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "EnterpriseHardwarePlatformAPIEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `EnterpriseHardwarePlatformAPIEnabled` pada baseline implementasi ditulis sebagai DWORD bernilai `0` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 27

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan ephemeral profile pada baseline L1 melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "ForceEphemeralProfiles" 0`
B. `Set-ChromePolicyDword $Chrome "ForceEphemeralProfiles" 1`
C. `Set-ChromePolicyString $Chrome "ForceEphemeralProfiles" "0"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "ForceEphemeralProfiles" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `ForceEphemeralProfiles` pada baseline implementasi ditulis sebagai DWORD bernilai `0` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 28

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan import autofill form data melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "ImportAutofillFormData" 0`
B. `Set-ChromePolicyDword $Chrome "ImportAutofillFormData" 1`
C. `Set-ChromePolicyString $Chrome "ImportAutofillFormData" "0"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "ImportAutofillFormData" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `ImportAutofillFormData` pada baseline implementasi ditulis sebagai DWORD bernilai `0` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 29

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan import homepage melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "ImportHomepage" 0`
B. `Set-ChromePolicyDword $Chrome "ImportHomepage" 1`
C. `Set-ChromePolicyString $Chrome "ImportHomepage" "0"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "ImportHomepage" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `ImportHomepage` pada baseline implementasi ditulis sebagai DWORD bernilai `0` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 30

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan import search engine melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "ImportSearchEngine" 0`
B. `Set-ChromePolicyDword $Chrome "ImportSearchEngine" 1`
C. `Set-ChromePolicyString $Chrome "ImportSearchEngine" "0"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "ImportSearchEngine" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `ImportSearchEngine` pada baseline implementasi ditulis sebagai DWORD bernilai `0` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 31

**Pertanyaan:** PowerShell mana yang benar untuk tidak menyembunyikan unsupported OS warning melalui registry policy Chrome?

A. `Set-ChromePolicyDword $Chrome "SuppressUnsupportedOSWarning" 0`
B. `Set-ChromePolicyDword $Chrome "SuppressUnsupportedOSWarning" 1`
C. `Set-ChromePolicyString $Chrome "SuppressUnsupportedOSWarning" "0"`
D. `Set-ChromePolicyDword "HKCU:\Software\Google\Chrome" "SuppressUnsupportedOSWarning" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `SuppressUnsupportedOSWarning` pada baseline implementasi ditulis sebagai DWORD bernilai `0` di `$Chrome` yaitu `HKLM:\SOFTWARE\Policies\Google\Chrome`.

---

### Soal 32

**Pertanyaan:** Perintah mana yang benar untuk memastikan Safe Browsing allowlist domain tidak digunakan secara longgar?

A. Tidak membuat key `HKLM:\SOFTWARE\Policies\Google\Chrome\SafeBrowsingAllowlistDomains` kecuali ada exception resmi
B. Membuat allowlist semua domain dengan nilai `*`
C. Mendokumentasikan exception jika ada domain internal yang harus dikecualikan
D. Menonaktifkan Safe Browsing agar tidak perlu allowlist

**Jawaban benar:** A, C

**Pembahasan:** CIS menolak allowlist luas yang melemahkan Safe Browsing. Jika ada pengecualian, harus sempit dan terdokumentasi.

---

### Soal 33

**Pertanyaan:** Policy mana yang benar untuk menghindari bypass HSTS atau insecure origin secara umum?

A. Tidak mengisi `HSTSPolicyBypassList` secara luas
B. Tidak mengisi `OverrideSecurityRestrictionsOnInsecureOrigin` secara luas
C. Mengisi bypass list dengan `*` agar semua aplikasi lama berjalan
D. Mencatat exception jika ada aplikasi legacy yang harus dimigrasi

**Jawaban benar:** A, B, D

**Pembahasan:** Bypass HSTS dan insecure origin harus dihindari. Exception harus spesifik dan memiliki rencana mitigasi/migrasi.

---

### Soal 34

**Pertanyaan:** Command mana yang benar untuk melihat apakah policy enforced sudah terbaca oleh Chrome, bukan hanya preference user?

A. Buka `chrome://policy/` dan pastikan policy tampil sebagai Machine policy
B. `Get-ItemProperty "HKLM:\SOFTWARE\Policies\Google\Chrome"`
C. `Get-ItemProperty "HKCU:\Software\Google\Chrome\PreferenceMACs"` sebagai sumber utama CIS
D. Klik Settings biasa lalu biarkan user mengubah setting

**Jawaban benar:** A, B

**Pembahasan:** CIS Chrome menekankan managed policy. HKLM policy dan `chrome://policy/` lebih kuat sebagai bukti daripada preference user.

---

### Soal 35

**Pertanyaan:** Semua opsi berikut diklaim sebagai command Enforced Defaults. Mana yang benar?

A. `Set-ChromePolicyDword $Chrome "AudioSandboxEnabled" 1`
B. `Set-ChromePolicyDword $Chrome "ComponentUpdatesEnabled" 1`
C. `Set-ChromePolicyDword $Chrome "BackgroundModeEnabled" 0`
D. `Set-ChromePolicyDword $Chrome "SavingBrowserHistoryDisabled" 1`

**Jawaban benar:** A, B, C

**Pembahasan:** `SavingBrowserHistoryDisabled` pada baseline L1 harus 0 agar penyimpanan history tidak dimatikan.

---


## 2. Attack Surface Reduction

### Soal 36

**Pertanyaan:** PowerShell mana yang benar untuk memastikan Google Chrome update tidak dimatikan?

A. `Set-ChromePolicyDword $Update "Update{8A69D345-D564-463C-AFF1-A69D9E530F96}" 1`
B. `Set-ChromePolicyDword $Update "Update{8A69D345-D564-463C-AFF1-A69D9E530F96}" 3`
C. `Set-ChromePolicyDword $Update "Update{8A69D345-D564-463C-AFF1-A69D9E530F96}" 0`
D. `Set-ChromePolicyDword $Chrome "Update{8A69D345-D564-463C-AFF1-A69D9E530F96}" 0`

**Jawaban benar:** A, B

**Pembahasan:** CIS mengizinkan Always allow updates (1) atau Automatic silent updates (3). Nilai 0 mematikan update dan salah.

---

### Soal 37

**Pertanyaan:** PowerShell mana yang benar untuk memastikan periode auto-update check tidak bernilai 0?

A. `Set-ChromePolicyDword $Update "AutoUpdateCheckPeriodMinutes" 1400`
B. `Set-ChromePolicyDword $Update "AutoUpdateCheckPeriodMinutes" 60`
C. `Set-ChromePolicyDword $Update "AutoUpdateCheckPeriodMinutes" 0`
D. `Set-ChromePolicyString $Update "AutoUpdateCheckPeriodMinutes" "disabled"`

**Jawaban benar:** A, B

**Pembahasan:** Target CIS adalah nilai apa pun kecuali 0. Nilai 0 menonaktifkan pengecekan update.

---

### Soal 38

**Pertanyaan:** PowerShell mana yang benar untuk memblokir mixed/insecure content?

A. `Set-ChromePolicyDword $Chrome "DefaultInsecureContentSetting" 2`
B. `Set-ChromePolicyDword $Chrome "DefaultInsecureContentSetting" 0`
C. `Set-ChromePolicyString $Chrome "DefaultInsecureContentSetting" "2"`
D. `Set-ChromePolicyDword $Update "DefaultInsecureContentSetting" 2`

**Jawaban benar:** A

**Pembahasan:** Policy `DefaultInsecureContentSetting` harus ditulis sebagai DWORD bernilai `2` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 39

**Pertanyaan:** PowerShell mana yang benar untuk memblokir ekstensi eksternal?

A. `Set-ChromePolicyDword $Chrome "BlockExternalExtensions" 1`
B. `Set-ChromePolicyDword $Chrome "BlockExternalExtensions" 0`
C. `Set-ChromePolicyString $Chrome "BlockExternalExtensions" "1"`
D. `Set-ChromePolicyDword $Update "BlockExternalExtensions" 1`

**Jawaban benar:** A

**Pembahasan:** Policy `BlockExternalExtensions` harus ditulis sebagai DWORD bernilai `1` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 40

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan Google Cloud Print Proxy?

A. `Set-ChromePolicyDword $Chrome "CloudPrintProxyEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "CloudPrintProxyEnabled" 1`
C. `Set-ChromePolicyString $Chrome "CloudPrintProxyEnabled" "0"`
D. `Set-ChromePolicyDword $Update "CloudPrintProxyEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `CloudPrintProxyEnabled` harus ditulis sebagai DWORD bernilai `0` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 41

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan koneksi Chrome Remote Desktop host?

A. `Set-ChromePolicyDword $Chrome "RemoteAccessHostAllowRemoteAccessConnections" 0`
B. `Set-ChromePolicyDword $Chrome "RemoteAccessHostAllowRemoteAccessConnections" 1`
C. `Set-ChromePolicyString $Chrome "RemoteAccessHostAllowRemoteAccessConnections" "0"`
D. `Set-ChromePolicyDword $Update "RemoteAccessHostAllowRemoteAccessConnections" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `RemoteAccessHostAllowRemoteAccessConnections` harus ditulis sebagai DWORD bernilai `0` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 42

**Pertanyaan:** PowerShell mana yang benar untuk menolak interaksi dengan elevated windows pada remote assistance?

A. `Set-ChromePolicyDword $Chrome "RemoteAccessHostAllowUiAccessForRemoteAssistance" 0`
B. `Set-ChromePolicyDword $Chrome "RemoteAccessHostAllowUiAccessForRemoteAssistance" 1`
C. `Set-ChromePolicyString $Chrome "RemoteAccessHostAllowUiAccessForRemoteAssistance" "0"`
D. `Set-ChromePolicyDword $Update "RemoteAccessHostAllowUiAccessForRemoteAssistance" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `RemoteAccessHostAllowUiAccessForRemoteAssistance` harus ditulis sebagai DWORD bernilai `0` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 43

**Pertanyaan:** PowerShell mana yang benar untuk mengikuti baseline implementasi untuk curtaining remote host?

A. `Set-ChromePolicyDword $Chrome "RemoteAccessHostRequireCurtain" 0`
B. `Set-ChromePolicyDword $Chrome "RemoteAccessHostRequireCurtain" 1`
C. `Set-ChromePolicyString $Chrome "RemoteAccessHostRequireCurtain" "0"`
D. `Set-ChromePolicyDword $Update "RemoteAccessHostRequireCurtain" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `RemoteAccessHostRequireCurtain` harus ditulis sebagai DWORD bernilai `0` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 44

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan firewall traversal remote access host?

A. `Set-ChromePolicyDword $Chrome "RemoteAccessHostFirewallTraversal" 0`
B. `Set-ChromePolicyDword $Chrome "RemoteAccessHostFirewallTraversal" 1`
C. `Set-ChromePolicyString $Chrome "RemoteAccessHostFirewallTraversal" "0"`
D. `Set-ChromePolicyDword $Update "RemoteAccessHostFirewallTraversal" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `RemoteAccessHostFirewallTraversal` harus ditulis sebagai DWORD bernilai `0` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 45

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan PIN-less/client pairing remote access?

A. `Set-ChromePolicyDword $Chrome "RemoteAccessHostAllowClientPairing" 0`
B. `Set-ChromePolicyDword $Chrome "RemoteAccessHostAllowClientPairing" 1`
C. `Set-ChromePolicyString $Chrome "RemoteAccessHostAllowClientPairing" "0"`
D. `Set-ChromePolicyDword $Update "RemoteAccessHostAllowClientPairing" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `RemoteAccessHostAllowClientPairing` harus ditulis sebagai DWORD bernilai `0` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 46

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan relay connection remote access host?

A. `Set-ChromePolicyDword $Chrome "RemoteAccessHostAllowRelayedConnection" 0`
B. `Set-ChromePolicyDword $Chrome "RemoteAccessHostAllowRelayedConnection" 1`
C. `Set-ChromePolicyString $Chrome "RemoteAccessHostAllowRelayedConnection" "0"`
D. `Set-ChromePolicyDword $Update "RemoteAccessHostAllowRelayedConnection" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `RemoteAccessHostAllowRelayedConnection` harus ditulis sebagai DWORD bernilai `0` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 47

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan First-Party Sets sesuai baseline?

A. `Set-ChromePolicyDword $Chrome "FirstPartySetsEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "FirstPartySetsEnabled" 1`
C. `Set-ChromePolicyString $Chrome "FirstPartySetsEnabled" "0"`
D. `Set-ChromePolicyDword $Update "FirstPartySetsEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `FirstPartySetsEnabled` harus ditulis sebagai DWORD bernilai `0` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 48

**Pertanyaan:** PowerShell mana yang benar untuk mengaktifkan CloudAPAuth sesuai baseline organisasi Microsoft cloud identity?

A. `Set-ChromePolicyDword $Chrome "CloudAPAuthEnabled" 1`
B. `Set-ChromePolicyDword $Chrome "CloudAPAuthEnabled" 0`
C. `Set-ChromePolicyString $Chrome "CloudAPAuthEnabled" "1"`
D. `Set-ChromePolicyDword $Update "CloudAPAuthEnabled" 1`

**Jawaban benar:** A

**Pembahasan:** Policy `CloudAPAuthEnabled` harus ditulis sebagai DWORD bernilai `1` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 49

**Pertanyaan:** PowerShell mana yang benar untuk memblokir malicious downloads sesuai recommended baseline?

A. `Set-ChromePolicyDword $Chrome "DownloadRestrictions" 4`
B. `Set-ChromePolicyDword $Chrome "DownloadRestrictions" 0`
C. `Set-ChromePolicyString $Chrome "DownloadRestrictions" "4"`
D. `Set-ChromePolicyDword $Update "DownloadRestrictions" 4`

**Jawaban benar:** A

**Pembahasan:** Policy `DownloadRestrictions` harus ditulis sebagai DWORD bernilai `4` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 50

**Pertanyaan:** PowerShell mana yang benar untuk mencegah user melanjutkan dari Safe Browsing warning?

A. `Set-ChromePolicyDword $Chrome "DisableSafeBrowsingProceedAnyway" 1`
B. `Set-ChromePolicyDword $Chrome "DisableSafeBrowsingProceedAnyway" 0`
C. `Set-ChromePolicyString $Chrome "DisableSafeBrowsingProceedAnyway" "1"`
D. `Set-ChromePolicyDword $Update "DisableSafeBrowsingProceedAnyway" 1`

**Jawaban benar:** A

**Pembahasan:** Policy `DisableSafeBrowsingProceedAnyway` harus ditulis sebagai DWORD bernilai `1` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 51

**Pertanyaan:** PowerShell mana yang benar untuk mengaktifkan site isolation untuk setiap site?

A. `Set-ChromePolicyDword $Chrome "SitePerProcess" 1`
B. `Set-ChromePolicyDword $Chrome "SitePerProcess" 0`
C. `Set-ChromePolicyString $Chrome "SitePerProcess" "1"`
D. `Set-ChromePolicyDword $Update "SitePerProcess" 1`

**Jawaban benar:** A

**Pembahasan:** Policy `SitePerProcess` harus ditulis sebagai DWORD bernilai `1` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 52

**Pertanyaan:** PowerShell mana yang benar untuk menampilkan prompt relaunch yang berulang?

A. `Set-ChromePolicyDword $Chrome "RelaunchNotification" 2`
B. `Set-ChromePolicyDword $Chrome "RelaunchNotification" 0`
C. `Set-ChromePolicyString $Chrome "RelaunchNotification" "2"`
D. `Set-ChromePolicyDword $Update "RelaunchNotification" 2`

**Jawaban benar:** A

**Pembahasan:** Policy `RelaunchNotification` harus ditulis sebagai DWORD bernilai `2` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 53

**Pertanyaan:** PowerShell mana yang benar untuk mengatur periode notifikasi relaunch 1 hari?

A. `Set-ChromePolicyDword $Chrome "RelaunchNotificationPeriod" 86400000`
B. `Set-ChromePolicyDword $Chrome "RelaunchNotificationPeriod" 0`
C. `Set-ChromePolicyString $Chrome "RelaunchNotificationPeriod" "86400000"`
D. `Set-ChromePolicyDword $Update "RelaunchNotificationPeriod" 86400000`

**Jawaban benar:** A

**Pembahasan:** Policy `RelaunchNotificationPeriod` harus ditulis sebagai DWORD bernilai `86400000` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 54

**Pertanyaan:** PowerShell mana yang benar untuk menolak WebAuthn pada broken TLS certificates?

A. `Set-ChromePolicyDword $Chrome "AllowWebAuthnWithBrokenTlsCerts" 0`
B. `Set-ChromePolicyDword $Chrome "AllowWebAuthnWithBrokenTlsCerts" 1`
C. `Set-ChromePolicyString $Chrome "AllowWebAuthnWithBrokenTlsCerts" "0"`
D. `Set-ChromePolicyDword $Update "AllowWebAuthnWithBrokenTlsCerts" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `AllowWebAuthnWithBrokenTlsCerts` harus ditulis sebagai DWORD bernilai `0` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 55

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan domain reliability reporting?

A. `Set-ChromePolicyDword $Chrome "DomainReliabilityAllowed" 0`
B. `Set-ChromePolicyDword $Chrome "DomainReliabilityAllowed" 1`
C. `Set-ChromePolicyString $Chrome "DomainReliabilityAllowed" "0"`
D. `Set-ChromePolicyDword $Update "DomainReliabilityAllowed" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `DomainReliabilityAllowed` harus ditulis sebagai DWORD bernilai `0` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 56

**Pertanyaan:** PowerShell mana yang benar untuk mengaktifkan TLS Encrypted ClientHello?

A. `Set-ChromePolicyDword $Chrome "EncryptedClientHelloEnabled" 1`
B. `Set-ChromePolicyDword $Chrome "EncryptedClientHelloEnabled" 0`
C. `Set-ChromePolicyString $Chrome "EncryptedClientHelloEnabled" "1"`
D. `Set-ChromePolicyDword $Update "EncryptedClientHelloEnabled" 1`

**Jawaban benar:** A

**Pembahasan:** Policy `EncryptedClientHelloEnabled` harus ditulis sebagai DWORD bernilai `1` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 57

**Pertanyaan:** PowerShell mana yang benar untuk menjaga browsing data saat enterprise profile dibuat?

A. `Set-ChromePolicyDword $Chrome "EnterpriseProfileCreationKeepBrowsingData" 1`
B. `Set-ChromePolicyDword $Chrome "EnterpriseProfileCreationKeepBrowsingData" 0`
C. `Set-ChromePolicyString $Chrome "EnterpriseProfileCreationKeepBrowsingData" "1"`
D. `Set-ChromePolicyDword $Update "EnterpriseProfileCreationKeepBrowsingData" 1`

**Jawaban benar:** A

**Pembahasan:** Policy `EnterpriseProfileCreationKeepBrowsingData` harus ditulis sebagai DWORD bernilai `1` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 58

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan Google Search Side Panel?

A. `Set-ChromePolicyDword $Chrome "GoogleSearchSidePanelEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "GoogleSearchSidePanelEnabled" 1`
C. `Set-ChromePolicyString $Chrome "GoogleSearchSidePanelEnabled" "0"`
D. `Set-ChromePolicyDword $Update "GoogleSearchSidePanelEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `GoogleSearchSidePanelEnabled` harus ditulis sebagai DWORD bernilai `0` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 59

**Pertanyaan:** PowerShell mana yang benar untuk mengaktifkan automatic HTTPS upgrades?

A. `Set-ChromePolicyDword $Chrome "HttpsUpgradesEnabled" 1`
B. `Set-ChromePolicyDword $Chrome "HttpsUpgradesEnabled" 0`
C. `Set-ChromePolicyString $Chrome "HttpsUpgradesEnabled" "1"`
D. `Set-ChromePolicyDword $Update "HttpsUpgradesEnabled" 1`

**Jawaban benar:** A

**Pembahasan:** Policy `HttpsUpgradesEnabled` harus ditulis sebagai DWORD bernilai `1` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 60

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan insecure hashes in TLS handshakes?

A. `Set-ChromePolicyDword $Chrome "InsecureHashesInTLSHandshakesEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "InsecureHashesInTLSHandshakesEnabled" 1`
C. `Set-ChromePolicyString $Chrome "InsecureHashesInTLSHandshakesEnabled" "0"`
D. `Set-ChromePolicyDword $Update "InsecureHashesInTLSHandshakesEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `InsecureHashesInTLSHandshakesEnabled` harus ditulis sebagai DWORD bernilai `0` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 61

**Pertanyaan:** PowerShell mana yang benar untuk mengaktifkan Renderer App Container?

A. `Set-ChromePolicyDword $Chrome "RendererAppContainerEnabled" 1`
B. `Set-ChromePolicyDword $Chrome "RendererAppContainerEnabled" 0`
C. `Set-ChromePolicyString $Chrome "RendererAppContainerEnabled" "1"`
D. `Set-ChromePolicyDword $Update "RendererAppContainerEnabled" 1`

**Jawaban benar:** A

**Pembahasan:** Policy `RendererAppContainerEnabled` harus ditulis sebagai DWORD bernilai `1` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 62

**Pertanyaan:** PowerShell mana yang benar untuk mengaktifkan strict MIME checking untuk worker scripts?

A. `Set-ChromePolicyDword $Chrome "StrictMimetypeCheckForWorkerScriptsEnabled" 1`
B. `Set-ChromePolicyDword $Chrome "StrictMimetypeCheckForWorkerScriptsEnabled" 0`
C. `Set-ChromePolicyString $Chrome "StrictMimetypeCheckForWorkerScriptsEnabled" "1"`
D. `Set-ChromePolicyDword $Update "StrictMimetypeCheckForWorkerScriptsEnabled" 1`

**Jawaban benar:** A

**Pembahasan:** Policy `StrictMimetypeCheckForWorkerScriptsEnabled` harus ditulis sebagai DWORD bernilai `1` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 63

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan remote debugging?

A. `Set-ChromePolicyDword $Chrome "RemoteDebuggingAllowed" 0`
B. `Set-ChromePolicyDword $Chrome "RemoteDebuggingAllowed" 1`
C. `Set-ChromePolicyString $Chrome "RemoteDebuggingAllowed" "0"`
D. `Set-ChromePolicyDword $Update "RemoteDebuggingAllowed" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `RemoteDebuggingAllowed` harus ditulis sebagai DWORD bernilai `0` pada path Chrome policy, kecuali policy Google Update yang memang berada di `$Update`.

---

### Soal 64

**Pertanyaan:** PowerShell mana yang benar untuk memblokir semua extension install kecuali nanti dibuat allowlist resmi?

A. `Set-ChromePolicyListItem "$Chrome\ExtensionInstallBlocklist" 1 "*"`
B. `New-ItemProperty -Path "$Chrome\ExtensionInstallBlocklist" -Name "1" -PropertyType String -Value "*" -Force | Out-Null`
C. `Set-ChromePolicyDword $Chrome "ExtensionInstallBlocklist" 1`
D. `Set-ChromePolicyListItem "$Chrome\ExtensionInstallBlocklist" 1 "allow-all"`

**Jawaban benar:** A, B

**Pembahasan:** Chrome list policy ditulis sebagai subkey dengan item bernama angka dan value string. Blocklist `*` berarti semua ekstensi diblokir kecuali allowlist.

---

### Soal 65

**Pertanyaan:** PowerShell mana yang benar untuk L2: native messaging blocklist semua host?

A. `Set-ChromePolicyListItem "$Chrome\NativeMessagingBlocklist" 1 "*"`
B. `New-ItemProperty -Path "$Chrome\NativeMessagingBlocklist" -Name "1" -PropertyType String -Value "*" -Force | Out-Null`
C. `Set-ChromePolicyDword $Chrome "NativeMessagingBlocklist" 1`
D. `Set-ChromePolicyListItem "$Chrome\NativeMessagingAllowlist" 1 "*"`

**Jawaban benar:** A, B

**Pembahasan:** Native messaging blocklist adalah list policy. Wildcard `*` pada blocklist memblokir native messaging host kecuali exception resmi.

---

### Soal 66

**Pertanyaan:** PowerShell mana yang benar untuk L2: membatasi AuthSchemes hanya `ntlm,negotiate`?

A. `Set-ChromePolicyString $Chrome "AuthSchemes" "ntlm,negotiate"`
B. `New-ItemProperty -Path $Chrome -Name "AuthSchemes" -PropertyType String -Value "ntlm,negotiate" -Force | Out-Null`
C. `Set-ChromePolicyString $Chrome "AuthSchemes" "basic,digest,ntlm,negotiate"`
D. `Set-ChromePolicyDword $Chrome "AuthSchemes" 1`

**Jawaban benar:** A, B

**Pembahasan:** `AuthSchemes` adalah string policy. Memasukkan basic/digest memperluas skema autentikasi dan bukan target L2.

---

### Soal 67

**Pertanyaan:** PowerShell mana yang benar untuk L2: memaksa Manifest V2 extension availability menjadi forced only sesuai baseline implementasi?

A. `Set-ChromePolicyDword $Chrome "ExtensionManifestV2Availability" 3`
B. `Set-ChromePolicyDword $Chrome "ExtensionManifestV2Availability" 0`
C. `Set-ChromePolicyString $Chrome "ExtensionManifestV2Availability" "allow"`
D. `Set-ChromePolicyDword $Update "ExtensionManifestV2Availability" 3`

**Jawaban benar:** A

**Pembahasan:** File implementasi memakai `ExtensionManifestV2Availability` DWORD value 3 pada Chrome policy path.

---

### Soal 68

**Pertanyaan:** PowerShell mana yang benar untuk L2: mencegah user melewati SSL warning page?

A. `Set-ChromePolicyDword $Chrome "SSLErrorOverrideAllowed" 0`
B. `Set-ChromePolicyDword $Chrome "SSLErrorOverrideAllowed" 1`
C. `Set-ChromePolicyDword $Chrome "DisableSafeBrowsingProceedAnyway" 0`
D. `Set-ChromePolicyString $Chrome "SSLErrorOverrideAllowed" "allow"`

**Jawaban benar:** A

**Pembahasan:** `SSLErrorOverrideAllowed` harus disabled/0 agar user tidak bisa melanjutkan dari halaman peringatan SSL.

---

### Soal 69

**Pertanyaan:** Manakah command/policy yang benar untuk mengurangi risiko debugging browser dari luar?

A. `Set-ChromePolicyDword $Chrome "RemoteDebuggingAllowed" 0`
B. `Get-ItemProperty $Chrome | Select-Object RemoteDebuggingAllowed`
C. `Set-ChromePolicyDword $Chrome "RemoteDebuggingAllowed" 1`
D. `chrome.exe --remote-debugging-port=9222` sebagai baseline hardening

**Jawaban benar:** A, B

**Pembahasan:** Remote debugging harus dinonaktifkan. Audit dapat membaca nilai policy. Membuka remote debugging port bukan hardening.

---

### Soal 70

**Pertanyaan:** Jika soal meminta mengizinkan semua situs memuat mixed content demi kompatibilitas, opsi mana yang benar sesuai CIS Chrome?

A. `Set-ChromePolicyDword $Chrome "DefaultInsecureContentSetting" 1`
B. `Set-ChromePolicyDword $Chrome "DefaultInsecureContentSetting" 0`
C. `Set-ChromePolicyDword $Chrome "DefaultInsecureContentSetting" 2` lalu buat exception sempit jika benar-benar perlu
D. `Set-ChromePolicyListItem "$Chrome\HttpAllowlist" 1 "*"`

**Jawaban benar:** C

**Pembahasan:** Baseline memblokir mixed content dengan nilai 2. Jika ada aplikasi legacy, gunakan exception sempit dan terdokumentasi, bukan allow semua.

---

### Soal 71

**Pertanyaan:** Semua opsi berikut diklaim sebagai hardening Chrome Remote Desktop. Mana yang benar?

A. `Set-ChromePolicyDword $Chrome "RemoteAccessHostAllowRemoteAccessConnections" 0`
B. `Set-ChromePolicyDword $Chrome "RemoteAccessHostFirewallTraversal" 0`
C. `Set-ChromePolicyDword $Chrome "RemoteAccessHostAllowClientPairing" 0`
D. `Set-ChromePolicyDword $Chrome "RemoteAccessHostAllowRelayedConnection" 1`

**Jawaban benar:** A, B, C

**Pembahasan:** Remote access yang tidak disetujui harus dikunci. Relayed connection baseline implementasi dinonaktifkan dengan nilai 0.

---

### Soal 72

**Pertanyaan:** Command mana yang benar untuk mengaktifkan semua fitur remote access, firewall traversal, dan relay agar helpdesk lebih mudah masuk ke endpoint?

A. `Set-ChromePolicyDword $Chrome "RemoteAccessHostAllowRemoteAccessConnections" 1`
B. `Set-ChromePolicyDword $Chrome "RemoteAccessHostFirewallTraversal" 1`
C. `Set-ChromePolicyDword $Chrome "RemoteAccessHostAllowRelayedConnection" 1`
D. `Set-ChromePolicyDword $Chrome "RemoteDebuggingAllowed" 1`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: seluruh opsi membuka permukaan serangan remote access/debugging dan tidak sesuai baseline hardening CIS Chrome.

---


## 3. Privacy

### Soal 73

**Pertanyaan:** PowerShell mana yang benar untuk menolak akses geolocation secara default?

A. `Set-ChromePolicyDword $Chrome "DefaultGeolocationSetting" 2`
B. `Set-ChromePolicyDword $Chrome "DefaultGeolocationSetting" 0`
C. `Set-ChromePolicyString $Chrome "DefaultGeolocationSetting" "2"`
D. `Set-ChromePolicyDword $Update "DefaultGeolocationSetting" 2`

**Jawaban benar:** A

**Pembahasan:** `DefaultGeolocationSetting` pada baseline privacy ditulis sebagai DWORD `2` di Chrome policy path.

---

### Soal 74

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan Google Cast/Media Router?

A. `Set-ChromePolicyDword $Chrome "EnableMediaRouter" 0`
B. `Set-ChromePolicyDword $Chrome "EnableMediaRouter" 1`
C. `Set-ChromePolicyString $Chrome "EnableMediaRouter" "0"`
D. `Set-ChromePolicyDword $Update "EnableMediaRouter" 0`

**Jawaban benar:** A

**Pembahasan:** `EnableMediaRouter` pada baseline privacy ditulis sebagai DWORD `0` di Chrome policy path.

---

### Soal 75

**Pertanyaan:** PowerShell mana yang benar untuk menolak website menanyakan payment methods?

A. `Set-ChromePolicyDword $Chrome "PaymentMethodQueryEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "PaymentMethodQueryEnabled" 1`
C. `Set-ChromePolicyString $Chrome "PaymentMethodQueryEnabled" "0"`
D. `Set-ChromePolicyDword $Update "PaymentMethodQueryEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** `PaymentMethodQueryEnabled` pada baseline privacy ditulis sebagai DWORD `0` di Chrome policy path.

---

### Soal 76

**Pertanyaan:** PowerShell mana yang benar untuk memblokir third-party cookies?

A. `Set-ChromePolicyDword $Chrome "BlockThirdPartyCookies" 1`
B. `Set-ChromePolicyDword $Chrome "BlockThirdPartyCookies" 0`
C. `Set-ChromePolicyString $Chrome "BlockThirdPartyCookies" "1"`
D. `Set-ChromePolicyDword $Update "BlockThirdPartyCookies" 1`

**Jawaban benar:** A

**Pembahasan:** `BlockThirdPartyCookies` pada baseline privacy ditulis sebagai DWORD `1` di Chrome policy path.

---

### Soal 77

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan Chrome Cleanup reporting?

A. `Set-ChromePolicyDword $Chrome "ChromeCleanupReportingEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "ChromeCleanupReportingEnabled" 1`
C. `Set-ChromePolicyString $Chrome "ChromeCleanupReportingEnabled" "0"`
D. `Set-ChromePolicyDword $Update "ChromeCleanupReportingEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** `ChromeCleanupReportingEnabled` pada baseline privacy ditulis sebagai DWORD `0` di Chrome policy path.

---

### Soal 78

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan sinkronisasi data dengan Google?

A. `Set-ChromePolicyDword $Chrome "SyncDisabled" 1`
B. `Set-ChromePolicyDword $Chrome "SyncDisabled" 0`
C. `Set-ChromePolicyString $Chrome "SyncDisabled" "1"`
D. `Set-ChromePolicyDword $Update "SyncDisabled" 1`

**Jawaban benar:** A

**Pembahasan:** `SyncDisabled` pada baseline privacy ditulis sebagai DWORD `1` di Chrome policy path.

---

### Soal 79

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan alternate error pages?

A. `Set-ChromePolicyDword $Chrome "AlternateErrorPagesEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "AlternateErrorPagesEnabled" 1`
C. `Set-ChromePolicyString $Chrome "AlternateErrorPagesEnabled" "0"`
D. `Set-ChromePolicyDword $Update "AlternateErrorPagesEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** `AlternateErrorPagesEnabled` pada baseline privacy ditulis sebagai DWORD `0` di Chrome policy path.

---

### Soal 80

**Pertanyaan:** PowerShell mana yang benar untuk mencegah user menghapus browsing/download history?

A. `Set-ChromePolicyDword $Chrome "AllowDeletingBrowserHistory" 0`
B. `Set-ChromePolicyDword $Chrome "AllowDeletingBrowserHistory" 1`
C. `Set-ChromePolicyString $Chrome "AllowDeletingBrowserHistory" "0"`
D. `Set-ChromePolicyDword $Update "AllowDeletingBrowserHistory" 0`

**Jawaban benar:** A

**Pembahasan:** `AllowDeletingBrowserHistory` pada baseline privacy ditulis sebagai DWORD `0` di Chrome policy path.

---

### Soal 81

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan network prediction pada semua koneksi?

A. `Set-ChromePolicyDword $Chrome "NetworkPredictionOptions" 2`
B. `Set-ChromePolicyDword $Chrome "NetworkPredictionOptions" 0`
C. `Set-ChromePolicyString $Chrome "NetworkPredictionOptions" "2"`
D. `Set-ChromePolicyDword $Update "NetworkPredictionOptions" 2`

**Jawaban benar:** A

**Pembahasan:** `NetworkPredictionOptions` pada baseline privacy ditulis sebagai DWORD `2` di Chrome policy path.

---

### Soal 82

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan spell checking web service?

A. `Set-ChromePolicyDword $Chrome "SpellCheckServiceEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "SpellCheckServiceEnabled" 1`
C. `Set-ChromePolicyString $Chrome "SpellCheckServiceEnabled" "0"`
D. `Set-ChromePolicyDword $Update "SpellCheckServiceEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** `SpellCheckServiceEnabled` pada baseline privacy ditulis sebagai DWORD `0` di Chrome policy path.

---

### Soal 83

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan usage/crash reporting?

A. `Set-ChromePolicyDword $Chrome "MetricsReportingEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "MetricsReportingEnabled" 1`
C. `Set-ChromePolicyString $Chrome "MetricsReportingEnabled" "0"`
D. `Set-ChromePolicyDword $Update "MetricsReportingEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** `MetricsReportingEnabled` pada baseline privacy ditulis sebagai DWORD `0` di Chrome policy path.

---

### Soal 84

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan Safe Browsing trusted-source bypass?

A. `Set-ChromePolicyDword $Chrome "SafeBrowsingForTrustedSourcesEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "SafeBrowsingForTrustedSourcesEnabled" 1`
C. `Set-ChromePolicyString $Chrome "SafeBrowsingForTrustedSourcesEnabled" "0"`
D. `Set-ChromePolicyDword $Update "SafeBrowsingForTrustedSourcesEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** `SafeBrowsingForTrustedSourcesEnabled` pada baseline privacy ditulis sebagai DWORD `0` di Chrome policy path.

---

### Soal 85

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan URL-keyed anonymized data collection?

A. `Set-ChromePolicyDword $Chrome "UrlKeyedAnonymizedDataCollectionEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "UrlKeyedAnonymizedDataCollectionEnabled" 1`
C. `Set-ChromePolicyString $Chrome "UrlKeyedAnonymizedDataCollectionEnabled" "0"`
D. `Set-ChromePolicyDword $Update "UrlKeyedAnonymizedDataCollectionEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** `UrlKeyedAnonymizedDataCollectionEnabled` pada baseline privacy ditulis sebagai DWORD `0` di Chrome policy path.

---

### Soal 86

**Pertanyaan:** PowerShell mana yang benar untuk L2: cookies hanya selama sesi?

A. `Set-ChromePolicyDword $Chrome "DefaultCookiesSetting" 4`
B. `Set-ChromePolicyDword $Chrome "DefaultCookiesSetting" 0`
C. `Set-ChromePolicyString $Chrome "DefaultCookiesSetting" "4"`
D. `Set-ChromePolicyDword $Update "DefaultCookiesSetting" 4`

**Jawaban benar:** A

**Pembahasan:** Policy L2 `DefaultCookiesSetting` menggunakan DWORD value `4` pada Chrome policy path.

---

### Soal 87

**Pertanyaan:** PowerShell mana yang benar untuk L2: menonaktifkan browser sign-in?

A. `Set-ChromePolicyDword $Chrome "BrowserSignin" 0`
B. `Set-ChromePolicyDword $Chrome "BrowserSignin" 1`
C. `Set-ChromePolicyString $Chrome "BrowserSignin" "0"`
D. `Set-ChromePolicyDword $Update "BrowserSignin" 0`

**Jawaban benar:** A

**Pembahasan:** Policy L2 `BrowserSignin` menggunakan DWORD value `0` pada Chrome policy path.

---

### Soal 88

**Pertanyaan:** PowerShell mana yang benar untuk L2: menonaktifkan search suggestions?

A. `Set-ChromePolicyDword $Chrome "SearchSuggestEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "SearchSuggestEnabled" 1`
C. `Set-ChromePolicyString $Chrome "SearchSuggestEnabled" "0"`
D. `Set-ChromePolicyDword $Update "SearchSuggestEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** Policy L2 `SearchSuggestEnabled` menggunakan DWORD value `0` pada Chrome policy path.

---

### Soal 89

**Pertanyaan:** PowerShell mana yang benar untuk L2: menonaktifkan Translate?

A. `Set-ChromePolicyDword $Chrome "TranslateEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "TranslateEnabled" 1`
C. `Set-ChromePolicyString $Chrome "TranslateEnabled" "0"`
D. `Set-ChromePolicyDword $Update "TranslateEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** Policy L2 `TranslateEnabled` menggunakan DWORD value `0` pada Chrome policy path.

---

### Soal 90

**Pertanyaan:** Manakah policy privacy yang benar untuk menahan jejak investigasi dan mengurangi sinkronisasi data keluar?

A. `Set-ChromePolicyDword $Chrome "AllowDeletingBrowserHistory" 0`
B. `Set-ChromePolicyDword $Chrome "SyncDisabled" 1`
C. `Set-ChromePolicyDword $Chrome "SavingBrowserHistoryDisabled" 0`
D. `Set-ChromePolicyDword $Chrome "AllowDeletingBrowserHistory" 1`

**Jawaban benar:** A, B, C

**Pembahasan:** History tidak dimatikan dan user tidak boleh menghapusnya sembarangan. Sync dinonaktifkan agar data tidak tersinkron ke akun Google.

---

### Soal 91

**Pertanyaan:** Jika organisasi ingin memaksimalkan privasi enterprise dengan mengurangi layanan eksternal Chrome, opsi mana yang benar?

A. `Set-ChromePolicyDword $Chrome "MetricsReportingEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "SpellCheckServiceEnabled" 0`
C. `Set-ChromePolicyDword $Chrome "SearchSuggestEnabled" 0` pada L2
D. `Set-ChromePolicyDword $Chrome "UrlKeyedAnonymizedDataCollectionEnabled" 1`

**Jawaban benar:** A, B, C

**Pembahasan:** Telemetry, web spellcheck, dan search suggestion dapat mengirim data ke layanan luar. URL-keyed data collection harus 0.

---


## 4. Data Loss Prevention

### Soal 92

**Pertanyaan:** PowerShell mana yang benar untuk menolak akses clipboard secara default?

A. `Set-ChromePolicyDword $Chrome "DefaultClipboardSetting" 2`
B. `Set-ChromePolicyDword $Chrome "DefaultClipboardSetting" 0`
C. `Set-ChromePolicyString $Chrome "DefaultClipboardSetting" "2"`
D. `Set-ChromePolicyDword $Update "DefaultClipboardSetting" 2`

**Jawaban benar:** A

**Pembahasan:** Policy `DefaultClipboardSetting` menggunakan DWORD `2` pada Chrome policy path sesuai baseline implementasi.

---

### Soal 93

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan user feedback?

A. `Set-ChromePolicyDword $Chrome "UserFeedbackAllowed" 0`
B. `Set-ChromePolicyDword $Chrome "UserFeedbackAllowed" 1`
C. `Set-ChromePolicyString $Chrome "UserFeedbackAllowed" "0"`
D. `Set-ChromePolicyDword $Update "UserFeedbackAllowed" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `UserFeedbackAllowed` menggunakan DWORD `0` pada Chrome policy path sesuai baseline implementasi.

---

### Soal 94

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan autofill kartu kredit?

A. `Set-ChromePolicyDword $Chrome "AutofillCreditCardEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "AutofillCreditCardEnabled" 1`
C. `Set-ChromePolicyString $Chrome "AutofillCreditCardEnabled" "0"`
D. `Set-ChromePolicyDword $Update "AutofillCreditCardEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `AutofillCreditCardEnabled` menggunakan DWORD `0` pada Chrome policy path sesuai baseline implementasi.

---

### Soal 95

**Pertanyaan:** PowerShell mana yang benar untuk menonaktifkan import saved passwords?

A. `Set-ChromePolicyDword $Chrome "ImportSavedPasswords" 0`
B. `Set-ChromePolicyDword $Chrome "ImportSavedPasswords" 1`
C. `Set-ChromePolicyString $Chrome "ImportSavedPasswords" "0"`
D. `Set-ChromePolicyDword $Update "ImportSavedPasswords" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `ImportSavedPasswords` menggunakan DWORD `0` pada Chrome policy path sesuai baseline implementasi.

---

### Soal 96

**Pertanyaan:** PowerShell mana yang benar untuk L2: menolak Serial API?

A. `Set-ChromePolicyDword $Chrome "DefaultSerialGuardSetting" 2`
B. `Set-ChromePolicyDword $Chrome "DefaultSerialGuardSetting" 0`
C. `Set-ChromePolicyString $Chrome "DefaultSerialGuardSetting" "2"`
D. `Set-ChromePolicyDword $Update "DefaultSerialGuardSetting" 2`

**Jawaban benar:** A

**Pembahasan:** Policy `DefaultSerialGuardSetting` menggunakan DWORD `2` pada Chrome policy path sesuai baseline implementasi.

---

### Soal 97

**Pertanyaan:** PowerShell mana yang benar untuk L2: menolak Sensors API?

A. `Set-ChromePolicyDword $Chrome "DefaultSensorsSetting" 2`
B. `Set-ChromePolicyDword $Chrome "DefaultSensorsSetting" 0`
C. `Set-ChromePolicyString $Chrome "DefaultSensorsSetting" "2"`
D. `Set-ChromePolicyDword $Update "DefaultSensorsSetting" 2`

**Jawaban benar:** A

**Pembahasan:** Policy `DefaultSensorsSetting` menggunakan DWORD `2` pada Chrome policy path sesuai baseline implementasi.

---

### Soal 98

**Pertanyaan:** PowerShell mana yang benar untuk L2: menolak Window Management permission?

A. `Set-ChromePolicyDword $Chrome "DefaultWindowManagementSetting" 2`
B. `Set-ChromePolicyDword $Chrome "DefaultWindowManagementSetting" 0`
C. `Set-ChromePolicyString $Chrome "DefaultWindowManagementSetting" "2"`
D. `Set-ChromePolicyDword $Update "DefaultWindowManagementSetting" 2`

**Jawaban benar:** A

**Pembahasan:** Policy `DefaultWindowManagementSetting` menggunakan DWORD `2` pada Chrome policy path sesuai baseline implementasi.

---

### Soal 99

**Pertanyaan:** PowerShell mana yang benar untuk L2: menonaktifkan file selection dialogs?

A. `Set-ChromePolicyDword $Chrome "AllowFileSelectionDialogs" 0`
B. `Set-ChromePolicyDword $Chrome "AllowFileSelectionDialogs" 1`
C. `Set-ChromePolicyString $Chrome "AllowFileSelectionDialogs" "0"`
D. `Set-ChromePolicyDword $Update "AllowFileSelectionDialogs" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `AllowFileSelectionDialogs` menggunakan DWORD `0` pada Chrome policy path sesuai baseline implementasi.

---

### Soal 100

**Pertanyaan:** PowerShell mana yang benar untuk L2: menonaktifkan audio capture?

A. `Set-ChromePolicyDword $Chrome "AudioCaptureAllowed" 0`
B. `Set-ChromePolicyDword $Chrome "AudioCaptureAllowed" 1`
C. `Set-ChromePolicyString $Chrome "AudioCaptureAllowed" "0"`
D. `Set-ChromePolicyDword $Update "AudioCaptureAllowed" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `AudioCaptureAllowed` menggunakan DWORD `0` pada Chrome policy path sesuai baseline implementasi.

---

### Soal 101

**Pertanyaan:** PowerShell mana yang benar untuk L2: menonaktifkan video capture?

A. `Set-ChromePolicyDword $Chrome "VideoCaptureAllowed" 0`
B. `Set-ChromePolicyDword $Chrome "VideoCaptureAllowed" 1`
C. `Set-ChromePolicyString $Chrome "VideoCaptureAllowed" "0"`
D. `Set-ChromePolicyDword $Update "VideoCaptureAllowed" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `VideoCaptureAllowed` menggunakan DWORD `0` pada Chrome policy path sesuai baseline implementasi.

---

### Soal 102

**Pertanyaan:** PowerShell mana yang benar untuk L2: menonaktifkan autofill alamat?

A. `Set-ChromePolicyDword $Chrome "AutofillAddressEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "AutofillAddressEnabled" 1`
C. `Set-ChromePolicyString $Chrome "AutofillAddressEnabled" "0"`
D. `Set-ChromePolicyDword $Update "AutofillAddressEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `AutofillAddressEnabled` menggunakan DWORD `0` pada Chrome policy path sesuai baseline implementasi.

---

### Soal 103

**Pertanyaan:** PowerShell mana yang benar untuk L2: menonaktifkan screen capture?

A. `Set-ChromePolicyDword $Chrome "ScreenCaptureAllowed" 0`
B. `Set-ChromePolicyDword $Chrome "ScreenCaptureAllowed" 1`
C. `Set-ChromePolicyString $Chrome "ScreenCaptureAllowed" "0"`
D. `Set-ChromePolicyDword $Update "ScreenCaptureAllowed" 0`

**Jawaban benar:** A

**Pembahasan:** Policy `ScreenCaptureAllowed` menggunakan DWORD `0` pada Chrome policy path sesuai baseline implementasi.

---

### Soal 104

**Pertanyaan:** PowerShell mana yang benar untuk mencegah sinkronisasi password?

A. `Set-ChromePolicyListItem "$Chrome\SyncTypesListDisabled" 1 "passwords"`
B. `New-ItemProperty -Path "$Chrome\SyncTypesListDisabled" -Name "1" -PropertyType String -Value "passwords" -Force | Out-Null`
C. `Set-ChromePolicyDword $Chrome "SyncTypesListDisabled" 1`
D. `Set-ChromePolicyListItem "$Chrome\SyncTypesListDisabled" 1 "Passwords"`

**Jawaban benar:** A, B

**Pembahasan:** Policy ini list item string dan value-nya case sensitive: `passwords`.

---

### Soal 105

**Pertanyaan:** PowerShell mana yang benar untuk mengatur DNS-over-HTTPS L2 tanpa fallback insecure?

A. `Set-ChromePolicyString $Chrome "DnsOverHttpsMode" "secure"`
B. `New-ItemProperty -Path $Chrome -Name "DnsOverHttpsMode" -PropertyType String -Value "secure" -Force | Out-Null`
C. `Set-ChromePolicyString $Chrome "DnsOverHttpsMode" "off"`
D. `Set-ChromePolicyDword $Chrome "DnsOverHttpsMode" 1`

**Jawaban benar:** A, B

**Pembahasan:** `DnsOverHttpsMode` adalah string policy; target L2 adalah `secure`.

---

### Soal 106

**Pertanyaan:** Manakah opsi yang benar untuk mengurangi kanal eksfiltrasi data melalui browser pada endpoint sensitif?

A. `Set-ChromePolicyDword $Chrome "ScreenCaptureAllowed" 0`
B. `Set-ChromePolicyDword $Chrome "AudioCaptureAllowed" 0`
C. `Set-ChromePolicyDword $Chrome "VideoCaptureAllowed" 0`
D. `Set-ChromePolicyDword $Chrome "AllowFileSelectionDialogs" 1`

**Jawaban benar:** A, B, C

**Pembahasan:** Screen/audio/video capture dibatasi pada L2. Mengizinkan file dialog secara umum justru dapat membuka kanal data.

---

### Soal 107

**Pertanyaan:** Policy clipboard mana yang benar jika organisasi ingin deny by default tetapi tetap menyiapkan exception spesifik?

A. `Set-ChromePolicyDword $Chrome "DefaultClipboardSetting" 2`
B. Gunakan `ClipboardAllowedForUrls` hanya untuk URL yang disetujui organisasi
C. Gunakan `ClipboardBlockedForUrls` untuk situs berisiko sesuai kebijakan
D. Gunakan wildcard `*` pada allowlist agar semua situs boleh clipboard

**Jawaban benar:** A, B, C

**Pembahasan:** Default deny lebih aman. Allowlist harus spesifik dan berbasis kebutuhan organisasi, bukan wildcard luas.

---


## 5. Forensics dan Investigasi Pascainsiden

### Soal 108

**Pertanyaan:** PowerShell mana yang benar untuk L2: menonaktifkan guest mode?

A. `Set-ChromePolicyDword $Chrome "BrowserGuestModeEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "BrowserGuestModeEnabled" 1`
C. `Set-ChromePolicyString $Chrome "BrowserGuestModeEnabled" "0"`
D. `Set-ChromePolicyDword $Update "BrowserGuestModeEnabled" 0`

**Jawaban benar:** A

**Pembahasan:** Policy forensics `BrowserGuestModeEnabled` memakai DWORD value `0` sesuai file implementasi.

---

### Soal 109

**Pertanyaan:** PowerShell mana yang benar untuk L2: menonaktifkan incognito mode?

A. `Set-ChromePolicyDword $Chrome "IncognitoModeAvailability" 1`
B. `Set-ChromePolicyDword $Chrome "IncognitoModeAvailability" 0`
C. `Set-ChromePolicyString $Chrome "IncognitoModeAvailability" "1"`
D. `Set-ChromePolicyDword $Update "IncognitoModeAvailability" 1`

**Jawaban benar:** A

**Pembahasan:** Policy forensics `IncognitoModeAvailability` memakai DWORD value `1` sesuai file implementasi.

---

### Soal 110

**Pertanyaan:** PowerShell mana yang benar untuk L1: mengatur disk cache size minimal 250609664 bytes?

A. `Set-ChromePolicyDword $Chrome "DiskCacheSize" 250609664`
B. `Set-ChromePolicyDword $Chrome "DiskCacheSize" 0`
C. `Set-ChromePolicyString $Chrome "DiskCacheSize" "250609664"`
D. `Set-ChromePolicyDword $Update "DiskCacheSize" 250609664`

**Jawaban benar:** A

**Pembahasan:** Policy forensics `DiskCacheSize` memakai DWORD value `250609664` sesuai file implementasi.

---

### Soal 111

**Pertanyaan:** Opsi mana yang benar agar investigasi pascainsiden tidak terlalu kehilangan artefak browser?

A. `Set-ChromePolicyDword $Chrome "BrowserGuestModeEnabled" 0` pada L2
B. `Set-ChromePolicyDword $Chrome "IncognitoModeAvailability" 1` pada L2
C. `Set-ChromePolicyDword $Chrome "DiskCacheSize" 250609664` pada L1
D. `Set-ChromePolicyDword $Chrome "SavingBrowserHistoryDisabled" 1`

**Jawaban benar:** A, B, C

**Pembahasan:** Guest/incognito dapat mengurangi artefak lokal. Disk cache size membantu ketersediaan artefak. Menonaktifkan history tidak sesuai baseline.

---

### Soal 112

**Pertanyaan:** Command mana yang benar untuk mengaktifkan mode yang menghilangkan jejak browsing lokal sebagai baseline forensics CIS?

A. `Set-ChromePolicyDword $Chrome "BrowserGuestModeEnabled" 1`
B. `Set-ChromePolicyDword $Chrome "IncognitoModeAvailability" 0`
C. `Set-ChromePolicyDword $Chrome "SavingBrowserHistoryDisabled" 1`
D. `Set-ChromePolicyDword $Chrome "AllowDeletingBrowserHistory" 1`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: seluruh opsi melemahkan tujuan forensics karena membuat guest/incognito/history deletion lebih mudah atau menghilangkan history.

---


## 6. GPO, Registry, Exception, dan Validasi Implementasi

### Soal 113

**Pertanyaan:** Manakah langkah implementasi yang benar sebelum menerapkan L2 ke produksi?

A. Uji di OU pilot
B. Validasi `chrome://policy/` dan aplikasi web internal
C. Langsung link ke seluruh domain tanpa rollback
D. Backup GPO dengan `Backup-GPO`

**Jawaban benar:** A, B, D

**Pembahasan:** L2 dapat mengganggu fitur WebUSB, WebBluetooth, cookies, notifikasi, capture, dan incognito. Pilot dan backup wajib.

---

### Soal 114

**Pertanyaan:** PowerShell mana yang benar untuk membackup GPO baseline sebelum produksi?

A. `Backup-GPO -Name "CIS - Google Chrome v3.0.0 - L1 Baseline" -Path "D:\GPO-Backup"`
B. `Remove-GPO -Name "CIS - Google Chrome v3.0.0 - L1 Baseline"`
C. `gpupdate /force`
D. `Backup-Chrome -AllUsers`

**Jawaban benar:** A

**Pembahasan:** `Backup-GPO` adalah command yang benar untuk backup GPO. `gpupdate` hanya refresh policy, bukan backup.

---

### Soal 115

**Pertanyaan:** Policy mana yang membutuhkan nilai organisasi dan tidak aman diisi sembarang wildcard?

A. `PasswordManagerEnabled`
B. `ProxySettings` / `ProxyMode`
C. `HttpAllowlist`
D. `ClipboardAllowedForUrls`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Beberapa policy manual bergantung pada keputusan organisasi. Wildcard atau allowlist luas dapat melemahkan kontrol.

---

### Soal 116

**Pertanyaan:** Manakah contoh exception register yang benar?

A. Policy `SSLErrorOverrideAllowed` dikecualikan sementara untuk aplikasi internal sertifikat lama, dengan risiko, mitigasi, owner, dan tanggal review
B. Membuat exception wildcard untuk semua situs HTTP tanpa owner
C. Membuat exception ekstensi password manager resmi berdasarkan extension ID dan disetujui security
D. Menonaktifkan Safe Browsing tanpa alasan karena mengganggu browsing

**Jawaban benar:** A, C

**Pembahasan:** Exception harus spesifik, memiliki alasan bisnis, risiko, mitigasi, owner, dan review. Wildcard tanpa kontrol bukan exception yang baik.

---

### Soal 117

**Pertanyaan:** Manakah registry list policy yang benar secara bentuk?

A. `HKLM:\SOFTWARE\Policies\Google\Chrome\ExtensionInstallBlocklist` berisi property string `1` bernilai `*`
B. `HKLM:\SOFTWARE\Policies\Google\Chrome\SyncTypesListDisabled` berisi property string `1` bernilai `passwords`
C. `HKLM:\SOFTWARE\Policies\Google\Chrome\NativeMessagingBlocklist` berisi property string `1` bernilai `*`
D. `HKLM:\SOFTWARE\Policies\Google\Chrome:NativeMessagingBlocklist` bernilai DWORD 1

**Jawaban benar:** A, B, C

**Pembahasan:** List policy Chrome biasanya memakai subkey dan item bernama angka dengan value string.

---

### Soal 118

**Pertanyaan:** Jika policy tidak muncul di `chrome://policy/`, langkah troubleshoot mana yang masuk akal?

A. Jalankan `gpupdate /force`
B. Restart Chrome atau klik Reload policies
C. Cek apakah ADMX/ADML sudah berada di Central Store yang benar
D. Ubah policy menjadi HKCU preference agar user bebas mengubah

**Jawaban benar:** A, B, C

**Pembahasan:** Refresh policy, restart/reload Chrome, dan validasi template ADMX/ADML adalah langkah yang benar. Mengubah menjadi preference user melemahkan enforcement.

---

### Soal 119

**Pertanyaan:** Manakah konfigurasi proxy yang perlu dihindari dalam baseline CIS jika tidak ada alasan resmi?

A. `ProxyMode` bernilai `auto_detect`
B. Proxy PAC internal yang disetujui organisasi
C. Proxy server eksplisit yang dikelola organisasi
D. Tidak ada proxy karena organisasi memang tidak memakai proxy

**Jawaban benar:** A

**Pembahasan:** Dokumen menyebut proxy setting tidak boleh memakai `auto_detect` tanpa alasan karena bisa disalahgunakan atau tidak terkendali.

---

### Soal 120

**Pertanyaan:** Command/policy mana yang benar untuk membaca policy Chrome dan Google Update dari PowerShell?

A. `Get-ItemProperty "HKLM:\SOFTWARE\Policies\Google\Chrome"`
B. `Get-ItemProperty "HKLM:\SOFTWARE\Policies\Google\Update"`
C. `Get-ItemProperty "HKLM:\SOFTWARE\Policies\Google\Chrome\ExtensionInstallBlocklist"`
D. `cat /etc/chrome/policies/managed/*.json` sebagai command utama Windows GPO

**Jawaban benar:** A, B, C

**Pembahasan:** Pada Windows GPO/registry, HKLM Google Chrome/Update adalah lokasi utama. Path `/etc` adalah konteks Linux, bukan Windows GPO.

---

### Soal 121

**Pertanyaan:** Manakah command yang benar untuk menerapkan baseline registry lokal di lab jika helper function sudah dimuat?

A. `$Chrome = "HKLM:\SOFTWARE\Policies\Google\Chrome"`
B. `$Update = "HKLM:\SOFTWARE\Policies\Google\Update"`
C. `Set-ChromePolicyDword $Chrome "SafeBrowsingProtectionLevel" 1`
D. `Set-ChromePolicyDword $Update "SafeBrowsingProtectionLevel" 1`

**Jawaban benar:** A, B, C

**Pembahasan:** SafeBrowsingProtectionLevel adalah Chrome policy, bukan Google Update policy.

---

### Soal 122

**Pertanyaan:** Manakah statement yang benar terkait GPO vs registry lokal?

A. Registry script berguna untuk lab/standalone test
B. Untuk domain enterprise, GPO lebih rapi dan dapat diaudit
C. Registry lokal adalah satu-satunya metode yang direkomendasikan dan GPO tidak boleh dipakai
D. Setelah GPO diterapkan, validasi tetap dilakukan di `chrome://policy/`

**Jawaban benar:** A, B, D

**Pembahasan:** Dokumen implementasi menyatakan GPO sebagai metode utama; registry script berguna untuk lab atau standalone test.

---

### Soal 123

**Pertanyaan:** Opsi mana yang benar untuk menerapkan policy extension yang aman?

A. Blocklist `*` lalu allowlist extension ID resmi bila ada kebutuhan
B. Izinkan semua extension agar user bebas memilih
C. Blokir external extensions
D. Dokumentasikan exception untuk extension bisnis resmi

**Jawaban benar:** A, C, D

**Pembahasan:** Ekstensi adalah attack surface besar. CIS mengarah pada blocklist/allowlist dan dokumentasi exception.

---

### Soal 124

**Pertanyaan:** Manakah opsi yang salah semua untuk baseline CIS Chrome?

A. `Set-ChromePolicyDword $Chrome "RemoteDebuggingAllowed" 1`
B. `Set-ChromePolicyDword $Chrome "BlockExternalExtensions" 0`
C. `Set-ChromePolicyDword $Chrome "ComponentUpdatesEnabled" 0`
D. `Set-ChromePolicyDword $Update "AutoUpdateCheckPeriodMinutes" 0`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: semua opsi melemahkan baseline. Remote debugging, ekstensi eksternal, komponen update, dan auto-update check tidak boleh dilemahkan.

---

### Soal 125

**Pertanyaan:** Manakah command/policy yang benar untuk L2 endpoint sensitif yang ingin memblokir API perangkat?

A. `Set-ChromePolicyDword $Chrome "DefaultWebBluetoothGuardSetting" 2`
B. `Set-ChromePolicyDword $Chrome "DefaultWebUsbGuardSetting" 2`
C. `Set-ChromePolicyDword $Chrome "DefaultSerialGuardSetting" 2`
D. `Set-ChromePolicyDword $Chrome "DefaultSensorsSetting" 2`

**Jawaban benar:** A, B, C, D

**Pembahasan:** WebBluetooth, WebUSB, Serial API, dan Sensors API dapat menjadi kanal akses perangkat. L2 menolaknya secara default.

---

### Soal 126

**Pertanyaan:** Manakah opsi yang benar untuk kebijakan TLS dan certificate handling?

A. `Set-ChromePolicyDword $Chrome "SSLErrorOverrideAllowed" 0`
B. `Set-ChromePolicyDword $Chrome "AllowWebAuthnWithBrokenTlsCerts" 0`
C. `Set-ChromePolicyDword $Chrome "InsecureHashesInTLSHandshakesEnabled" 0`
D. `Set-ChromePolicyDword $Chrome "SSLErrorOverrideAllowed" 1`

**Jawaban benar:** A, B, C

**Pembahasan:** User tidak boleh mudah melewati SSL warning, WebAuthn tidak boleh pada broken TLS certs, dan insecure hashes harus dinonaktifkan.

---

### Soal 127

**Pertanyaan:** Manakah opsi yang benar untuk mengatur Safe Browsing?

A. `Set-ChromePolicyDword $Chrome "SafeBrowsingProtectionLevel" 1`
B. `Set-ChromePolicyDword $Chrome "DisableSafeBrowsingProceedAnyway" 1`
C. Tidak membuat allowlist domain Safe Browsing secara luas
D. `Set-ChromePolicyDword $Chrome "SafeBrowsingProtectionLevel" 0`

**Jawaban benar:** A, B, C

**Pembahasan:** Safe Browsing harus aktif dan user tidak boleh sembarang melanjutkan dari warning. Mematikan level proteksi tidak sesuai baseline.

---

### Soal 128

**Pertanyaan:** Manakah opsi yang benar untuk DLP autofill dan password import?

A. `Set-ChromePolicyDword $Chrome "AutofillCreditCardEnabled" 0`
B. `Set-ChromePolicyDword $Chrome "AutofillAddressEnabled" 0` pada L2
C. `Set-ChromePolicyDword $Chrome "ImportSavedPasswords" 0`
D. `Set-ChromePolicyDword $Chrome "ImportSavedPasswords" 1`

**Jawaban benar:** A, B, C

**Pembahasan:** Autofill kartu kredit, autofill alamat pada L2, dan import saved passwords harus dibatasi untuk mengurangi kebocoran data.

---

### Soal 129

**Pertanyaan:** Manakah opsi yang benar untuk policy update/relaunch?

A. `Set-ChromePolicyDword $Update "Update{8A69D345-D564-463C-AFF1-A69D9E530F96}" 1`
B. `Set-ChromePolicyDword $Update "AutoUpdateCheckPeriodMinutes" 1400`
C. `Set-ChromePolicyDword $Chrome "RelaunchNotification" 2`
D. `Set-ChromePolicyDword $Chrome "RelaunchNotificationPeriod" 86400000`

**Jawaban benar:** A, B, C, D

**Pembahasan:** Update harus diizinkan, pengecekan update tidak boleh 0, dan relaunch notification membantu memastikan update efektif setelah restart browser.

---


## 7. Soal PASS dan Deteksi Command Salah

### Soal 130

**Pertanyaan:** Semua opsi berikut diklaim sebagai implementasi CIS Chrome pada Windows. Mana yang benar?

A. `sudo ufw enable`
B. `auditctl -l | grep chrome`
C. `aa-status`
D. `systemctl restart chrome.service`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: command tersebut relevan untuk Linux host/service, bukan implementasi utama CIS Google Chrome Benchmark pada Windows GPO/registry.

---

### Soal 131

**Pertanyaan:** Pilih command yang benar untuk menonaktifkan Safe Browsing agar user tidak terganggu warning.

A. `Set-ChromePolicyDword $Chrome "SafeBrowsingProtectionLevel" 0`
B. `Set-ChromePolicyDword $Chrome "DisableSafeBrowsingProceedAnyway" 0`
C. `Set-ChromePolicyListItem "$Chrome\SafeBrowsingAllowlistDomains" 1 "*"`
D. `Set-ChromePolicyDword $Chrome "SafeBrowsingForTrustedSourcesEnabled" 1`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: seluruh opsi melemahkan Safe Browsing atau membuat bypass luas.

---

### Soal 132

**Pertanyaan:** Pilih command yang benar untuk membuka semua extension dan native messaging agar aplikasi user bebas berjalan.

A. `Set-ChromePolicyDword $Chrome "BlockExternalExtensions" 0`
B. `Set-ChromePolicyListItem "$Chrome\ExtensionInstallBlocklist" 1 ""`
C. `Set-ChromePolicyListItem "$Chrome\NativeMessagingBlocklist" 1 ""`
D. `Set-ChromePolicyDword $Chrome "ExtensionManifestV2Availability" 0`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: pendekatan allow semua extension/native messaging tidak sesuai attack surface reduction.

---

### Soal 133

**Pertanyaan:** Pilih command yang benar untuk mengizinkan remote debugging sebagai baseline CIS Chrome.

A. `Set-ChromePolicyDword $Chrome "RemoteDebuggingAllowed" 1`
B. `chrome.exe --remote-debugging-port=9222`
C. `Set-ChromePolicyString $Chrome "RemoteDebuggingAllowed" "enabled"`
D. `netsh advfirewall firewall add rule name=ChromeDebug dir=in action=allow protocol=TCP localport=9222`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: baseline CIS menonaktifkan remote debugging. Membuka port debugging memperbesar risiko kendali browser.

---

### Soal 134

**Pertanyaan:** Pilih command yang benar untuk mengizinkan user bypass SSL warning page sebagai baseline.

A. `Set-ChromePolicyDword $Chrome "SSLErrorOverrideAllowed" 1`
B. `Set-ChromePolicyDword $Chrome "AllowWebAuthnWithBrokenTlsCerts" 1`
C. `Set-ChromePolicyDword $Chrome "InsecureHashesInTLSHandshakesEnabled" 1`
D. `Set-ChromePolicyDword $Chrome "DisableSafeBrowsingProceedAnyway" 0`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: semua opsi melemahkan TLS/Safe Browsing protection.

---

### Soal 135

**Pertanyaan:** Pilih command yang benar untuk menghapus semua policy Chrome agar baseline lebih fleksibel untuk user.

A. `Remove-Item "HKLM:\SOFTWARE\Policies\Google\Chrome" -Recurse -Force` pada produksi tanpa approval
B. `Remove-Item "HKLM:\SOFTWARE\Policies\Google\Update" -Recurse -Force` pada produksi tanpa approval
C. `gpupdate /force` setelah menghapus GPO produksi tanpa dokumentasi
D. `chrome://policy/` lalu abaikan policy yang hilang

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: rollback hanya untuk lab atau perubahan resmi. Menghapus policy produksi tanpa approval/dokumentasi bukan baseline CIS.

---

### Soal 136

**Pertanyaan:** Pilih command yang benar untuk membuat allowlist HTTP semua domain sebagai baseline.

A. `Set-ChromePolicyListItem "$Chrome\HttpAllowlist" 1 "*"`
B. `Set-ChromePolicyListItem "$Chrome\HttpAllowlist" 1 "http://*"`
C. `Set-ChromePolicyDword $Chrome "HttpsUpgradesEnabled" 0`
D. `Set-ChromePolicyDword $Chrome "DefaultInsecureContentSetting" 1`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: HTTP allowlist harus spesifik jika diperlukan. Baseline justru mengaktifkan HTTPS upgrades dan memblokir insecure content.

---

### Soal 137

**Pertanyaan:** Pilih command yang benar untuk membuka screen/audio/video capture default pada endpoint sensitif.

A. `Set-ChromePolicyDword $Chrome "ScreenCaptureAllowed" 1`
B. `Set-ChromePolicyDword $Chrome "AudioCaptureAllowed" 1`
C. `Set-ChromePolicyDword $Chrome "VideoCaptureAllowed" 1`
D. `Set-ChromePolicyDword $Chrome "DefaultClipboardSetting" 1`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: endpoint sensitif L2 membatasi capture dan clipboard karena dapat menjadi kanal eksfiltrasi data.

---

### Soal 138

**Pertanyaan:** Pilih command yang benar untuk mematikan update Chrome karena update sering meminta relaunch.

A. `Set-ChromePolicyDword $Update "Update{8A69D345-D564-463C-AFF1-A69D9E530F96}" 0`
B. `Set-ChromePolicyDword $Update "AutoUpdateCheckPeriodMinutes" 0`
C. `Set-ChromePolicyDword $Chrome "ComponentUpdatesEnabled" 0`
D. `Set-ChromePolicyDword $Chrome "RelaunchNotification" 0` sebagai pengganti patching

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: update Chrome dan komponen harus tetap aktif. Relaunch notification membantu patch benar-benar efektif.

---

### Soal 139

**Pertanyaan:** Pilih command yang benar untuk membuat user bebas menghapus history sebagai baseline forensics.

A. `Set-ChromePolicyDword $Chrome "AllowDeletingBrowserHistory" 1`
B. `Set-ChromePolicyDword $Chrome "SavingBrowserHistoryDisabled" 1`
C. `Set-ChromePolicyDword $Chrome "IncognitoModeAvailability" 0`
D. `Set-ChromePolicyDword $Chrome "BrowserGuestModeEnabled" 1`

**Jawaban benar:** PASS / kosong

**Pembahasan:** PASS: baseline forensics menjaga jejak lokal, bukan menghilangkannya.

---
