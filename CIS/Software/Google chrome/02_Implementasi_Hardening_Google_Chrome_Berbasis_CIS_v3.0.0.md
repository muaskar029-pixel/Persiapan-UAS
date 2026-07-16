# Implementasi Hardening Google Chrome Berbasis CIS Benchmark v3.0.0

**Sumber utama:** CIS Google Chrome Benchmark v3.0.0 — 01-29-2024.  
**Fokus:** implementasi teknis melalui Group Policy/registry policy pada endpoint Windows enterprise.

---

## 1. Prinsip Implementasi

Benchmark ini paling sesuai diterapkan pada lingkungan **Windows Active Directory domain-joined** menggunakan **Google Chrome ADMX/ADML** dan **Google Update ADMX/ADML**. Untuk organisasi, cara paling rapi adalah memakai Group Policy Object (GPO), bukan mengubah setting satu per satu dari UI Chrome.

Alur ideal:

1. Siapkan ADMX/ADML Chrome dan Google Update.
2. Buat GPO khusus Chrome baseline.
3. Terapkan ke OU uji/lab.
4. Validasi dengan `chrome://policy/`, registry, dan uji fungsi aplikasi web internal.
5. Lakukan pilot ke sebagian user.
6. Terapkan produksi bertahap.
7. Dokumentasikan exception.

---

## 2. Prasyarat

- Endpoint Windows domain-joined.
- Google Chrome terpasang dan dikelola organisasi.
- Akses administrator domain atau delegated GPO admin.
- Google Chrome ADMX/ADML template.
- Google Update ADMX/ADML template.
- Group Policy Management Console.
- OU target untuk lab/pilot/produksi.

Direktori Central Store biasanya:

```text
\\<domain>\\SYSVOL\\<domain>\\Policies\\PolicyDefinitions
```

File yang perlu tersedia antara lain:

```text
chrome.admx
google.admx
GoogleUpdate.admx
<language>\\chrome.adml
<language>\\google.adml
<language>\\GoogleUpdate.adml
```

---

## 3. Strategi Profil

### 3.1 Baseline L1

Gunakan L1 untuk endpoint umum. L1 relatif aman untuk kebanyakan organisasi karena tujuannya tidak terlalu mengganggu fungsi browser.

### 3.2 Baseline L2

Gunakan L2 untuk perangkat yang menangani data sensitif, administrator, endpoint audit, endpoint riset internal, atau area yang membutuhkan kontrol ketat. L2 harus diuji lebih dulu karena dapat membatasi fitur seperti WebUSB, WebBluetooth, notifikasi, cookies, capture, dan Incognito.

---

## 4. Tahap Implementasi GPO

### 4.1 Buat GPO

Contoh nama:

```text
CIS - Google Chrome v3.0.0 - L1 Baseline
CIS - Google Chrome v3.0.0 - L2 Add-on
```

Pisahkan L1 dan L2 agar penerapan lebih fleksibel.

### 4.2 Link GPO ke OU Pilot

Jangan langsung link ke seluruh domain. Gunakan OU pilot, misalnya:

```text
OU=Pilot-Chrome-CIS,OU=Workstations,DC=example,DC=local
```

### 4.3 Terapkan Policy

Gunakan Group Policy Management Editor:

```text
Computer Configuration
└─ Policies
   └─ Administrative Templates
      └─ Google
         ├─ Google Chrome
         └─ Google Update
```

### 4.4 Force Update Policy di Client

```powershell
gpupdate /force
```

Restart Chrome, lalu cek:

```text
chrome://policy/
```

Klik **Reload policies** untuk memuat ulang policy.

---

## 5. Audit dan Validasi

### 5.1 Cek Policy dari Chrome

Buka:

```text
chrome://policy/
```

Pastikan policy muncul sebagai **Machine policy** atau policy organisasi, bukan sekadar user preference.

### 5.2 Cek Registry Policy

Sebagian besar policy berada di:

```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Policies\Google\Chrome"
Get-ItemProperty "HKLM:\SOFTWARE\Policies\Google\Update"
```

### 5.3 Cek Update Chrome

Buka:

```text
chrome://settings/help
chrome://components
```

`chrome://components` berguna untuk melihat komponen Chrome yang ikut diperbarui.

---

## 6. PowerShell Helper untuk Registry Policy Lokal

GPO tetap metode utama. Script berikut berguna untuk lab, standalone test, atau pembuatan baseline registry sebelum ditransformasikan ke GPO. Jalankan sebagai Administrator.

```powershell
# Helper dasar: membuat path registry jika belum ada, lalu menulis nilai policy.
function Set-ChromePolicyDword {
    param(
        [string]$Path,
        [string]$Name,
        [int]$Value
    )
    if (-not (Test-Path $Path)) {
        New-Item -Path $Path -Force | Out-Null
    }
    New-ItemProperty -Path $Path -Name $Name -PropertyType DWord -Value $Value -Force | Out-Null
}

function Set-ChromePolicyString {
    param(
        [string]$Path,
        [string]$Name,
        [string]$Value
    )
    if (-not (Test-Path $Path)) {
        New-Item -Path $Path -Force | Out-Null
    }
    New-ItemProperty -Path $Path -Name $Name -PropertyType String -Value $Value -Force | Out-Null
}

function Set-ChromePolicyListItem {
    param(
        [string]$Path,
        [int]$Index,
        [string]$Value
    )
    if (-not (Test-Path $Path)) {
        New-Item -Path $Path -Force | Out-Null
    }
    New-ItemProperty -Path $Path -Name "$Index" -PropertyType String -Value $Value -Force | Out-Null
}
```

---

## 7. Contoh Baseline Registry L1 Inti

Script ini bukan pengganti GPO penuh. Gunakan sebagai contoh implementasi lab untuk sebagian besar kontrol L1 yang bernilai numerik sederhana.

```powershell
$Chrome = "HKLM:\SOFTWARE\Policies\Google\Chrome"
$Update = "HKLM:\SOFTWARE\Policies\Google\Update"

# Bab 1 - Enforced Defaults
Set-ChromePolicyDword $Chrome "AllowCrossOriginAuthPrompt" 0
Set-ChromePolicyDword $Chrome "SafeBrowsingProtectionLevel" 1
Set-ChromePolicyDword $Chrome "MediaRouterCastAllowAllIPs" 0
Set-ChromePolicyDword $Chrome "BrowserNetworkTimeQueriesEnabled" 1
Set-ChromePolicyDword $Chrome "AudioSandboxEnabled" 1
Set-ChromePolicyDword $Chrome "PromptForDownloadLocation" 1
Set-ChromePolicyDword $Chrome "BackgroundModeEnabled" 0
Set-ChromePolicyDword $Chrome "ChromeVariations" 0
Set-ChromePolicyDword $Chrome "SavingBrowserHistoryDisabled" 0
Set-ChromePolicyDword $Chrome "DNSInterceptionChecksEnabled" 1
Set-ChromePolicyDword $Chrome "ComponentUpdatesEnabled" 1
Set-ChromePolicyDword $Chrome "GloballyScopeHTTPAuthCacheEnabled" 0
Set-ChromePolicyDword $Chrome "EnableOnlineRevocationChecks" 0
Set-ChromePolicyDword $Chrome "CommandLineFlagSecurityWarningsEnabled" 1
Set-ChromePolicyDword $Chrome "ThirdPartyBlockingEnabled" 1
Set-ChromePolicyDword $Chrome "EnterpriseHardwarePlatformAPIEnabled" 0
Set-ChromePolicyDword $Chrome "ForceEphemeralProfiles" 0
Set-ChromePolicyDword $Chrome "ImportAutofillFormData" 0
Set-ChromePolicyDword $Chrome "ImportHomepage" 0
Set-ChromePolicyDword $Chrome "ImportSearchEngine" 0
Set-ChromePolicyDword $Chrome "SuppressUnsupportedOSWarning" 0

# Bab 2 - Attack Surface Reduction
Set-ChromePolicyDword $Update "Update{8A69D345-D564-463C-AFF1-A69D9E530F96}" 1
Set-ChromePolicyDword $Update "AutoUpdateCheckPeriodMinutes" 1400
Set-ChromePolicyDword $Chrome "DefaultInsecureContentSetting" 2
Set-ChromePolicyDword $Chrome "BlockExternalExtensions" 1
Set-ChromePolicyListItem "$Chrome\ExtensionInstallBlocklist" 1 "*"
Set-ChromePolicyDword $Chrome "ExtensionUnpublishedAvailability" 1
Set-ChromePolicyDword $Chrome "CloudPrintProxyEnabled" 0
Set-ChromePolicyDword $Chrome "RemoteAccessHostAllowRemoteAccessConnections" 0
Set-ChromePolicyDword $Chrome "RemoteAccessHostAllowUiAccessForRemoteAssistance" 0
Set-ChromePolicyDword $Chrome "RemoteAccessHostRequireCurtain" 0
Set-ChromePolicyDword $Chrome "RemoteAccessHostFirewallTraversal" 0
Set-ChromePolicyDword $Chrome "RemoteAccessHostAllowClientPairing" 0
Set-ChromePolicyDword $Chrome "RemoteAccessHostAllowRelayedConnection" 0
Set-ChromePolicyDword $Chrome "FirstPartySetsEnabled" 0
Set-ChromePolicyDword $Chrome "CloudAPAuthEnabled" 1
Set-ChromePolicyDword $Chrome "DownloadRestrictions" 4
Set-ChromePolicyDword $Chrome "DisableSafeBrowsingProceedAnyway" 1
Set-ChromePolicyDword $Chrome "SitePerProcess" 1
Set-ChromePolicyDword $Chrome "RelaunchNotification" 2
Set-ChromePolicyDword $Chrome "RelaunchNotificationPeriod" 86400000
Set-ChromePolicyDword $Chrome "AllowWebAuthnWithBrokenTlsCerts" 0
Set-ChromePolicyDword $Chrome "DomainReliabilityAllowed" 0
Set-ChromePolicyDword $Chrome "EncryptedClientHelloEnabled" 1
Set-ChromePolicyDword $Chrome "EnterpriseProfileCreationKeepBrowsingData" 1
Set-ChromePolicyDword $Chrome "GoogleSearchSidePanelEnabled" 0
Set-ChromePolicyDword $Chrome "HttpsUpgradesEnabled" 1
Set-ChromePolicyDword $Chrome "InsecureHashesInTLSHandshakesEnabled" 0
Set-ChromePolicyDword $Chrome "RendererAppContainerEnabled" 1
Set-ChromePolicyDword $Chrome "StrictMimetypeCheckForWorkerScriptsEnabled" 1
Set-ChromePolicyDword $Chrome "RemoteDebuggingAllowed" 0

# Bab 3 - Privacy
Set-ChromePolicyDword $Chrome "DefaultGeolocationSetting" 2
Set-ChromePolicyDword $Chrome "EnableMediaRouter" 0
Set-ChromePolicyDword $Chrome "PaymentMethodQueryEnabled" 0
Set-ChromePolicyDword $Chrome "BlockThirdPartyCookies" 1
Set-ChromePolicyDword $Chrome "ChromeCleanupReportingEnabled" 0
Set-ChromePolicyDword $Chrome "SyncDisabled" 1
Set-ChromePolicyDword $Chrome "AlternateErrorPagesEnabled" 0
Set-ChromePolicyDword $Chrome "AllowDeletingBrowserHistory" 0
Set-ChromePolicyDword $Chrome "NetworkPredictionOptions" 2
Set-ChromePolicyDword $Chrome "SpellCheckServiceEnabled" 0
Set-ChromePolicyDword $Chrome "MetricsReportingEnabled" 0
Set-ChromePolicyDword $Chrome "SafeBrowsingForTrustedSourcesEnabled" 0
Set-ChromePolicyDword $Chrome "UrlKeyedAnonymizedDataCollectionEnabled" 0

# Bab 4 - DLP L1 inti
Set-ChromePolicyDword $Chrome "DefaultClipboardSetting" 2
Set-ChromePolicyDword $Chrome "UserFeedbackAllowed" 0
Set-ChromePolicyDword $Chrome "AutofillCreditCardEnabled" 0
Set-ChromePolicyDword $Chrome "ImportSavedPasswords" 0
Set-ChromePolicyListItem "$Chrome\SyncTypesListDisabled" 1 "passwords"

# Bab 5 - Forensics L1 inti
Set-ChromePolicyDword $Chrome "DiskCacheSize" 250609664

Write-Host "Baseline Chrome policy diterapkan. Jalankan gpupdate /force atau restart Chrome, lalu cek chrome://policy/."
```

---

## 8. Contoh Add-on L2

Terapkan hanya setelah uji dampak.

```powershell
$Chrome = "HKLM:\SOFTWARE\Policies\Google\Chrome"

Set-ChromePolicyDword $Chrome "SafeSitesFilterBehavior" 1
Set-ChromePolicyDword $Chrome "DefaultWebBluetoothGuardSetting" 2
Set-ChromePolicyDword $Chrome "DefaultWebUsbGuardSetting" 2
Set-ChromePolicyDword $Chrome "DefaultNotificationsSetting" 2
Set-ChromePolicyDword $Chrome "DefaultThirdPartyStoragePartitioningSetting" 2
Set-ChromePolicyDword $Chrome "ExtensionManifestV2Availability" 3
Set-ChromePolicyString $Chrome "AuthSchemes" "ntlm,negotiate"
Set-ChromePolicyListItem "$Chrome\NativeMessagingBlocklist" 1 "*"
Set-ChromePolicyDword $Chrome "SSLErrorOverrideAllowed" 0
Set-ChromePolicyDword $Chrome "ForceGoogleSafeSearch" 1
Set-ChromePolicyDword $Chrome "RequireOnlineRevocationChecksForLocalAnchors" 1
Set-ChromePolicyDword $Chrome "EnforceLocalAnchorConstraintsEnabled" 1
Set-ChromePolicyDword $Chrome "DefaultCookiesSetting" 4
Set-ChromePolicyDword $Chrome "BrowserSignin" 0
Set-ChromePolicyDword $Chrome "SearchSuggestEnabled" 0
Set-ChromePolicyDword $Chrome "TranslateEnabled" 0
Set-ChromePolicyDword $Chrome "ScreenCaptureAllowed" 0
Set-ChromePolicyDword $Chrome "DefaultSerialGuardSetting" 2
Set-ChromePolicyDword $Chrome "DefaultSensorsSetting" 2
Set-ChromePolicyDword $Chrome "DefaultWindowManagementSetting" 2
Set-ChromePolicyDword $Chrome "AllowFileSelectionDialogs" 0
Set-ChromePolicyDword $Chrome "AudioCaptureAllowed" 0
Set-ChromePolicyDword $Chrome "VideoCaptureAllowed" 0
Set-ChromePolicyString $Chrome "DnsOverHttpsMode" "secure"
Set-ChromePolicyDword $Chrome "AutofillAddressEnabled" 0
Set-ChromePolicyDword $Chrome "BrowserGuestModeEnabled" 0
Set-ChromePolicyDword $Chrome "IncognitoModeAvailability" 1
```

---

## 9. Policy yang Perlu Nilai Organisasi

Beberapa rekomendasi tidak bisa diisi universal karena tergantung domain, aplikasi internal, proxy, atau allowlist organisasi:

| Area | Contoh Policy | Keputusan yang Dibutuhkan |
|---|---|---|
| Password Manager | `PasswordManagerEnabled` | Apakah organisasi memakai password manager Chrome atau solusi lain. |
| Proxy | `ProxySettings` | Mode proxy, server, PAC URL, dan larangan `auto_detect`. |
| Remote Access Domain | `RemoteAccessHostClientDomainList` | Domain resmi remote support jika Chrome Remote Desktop disetujui. |
| HTTP Allowlist | `HttpAllowlist` | Situs HTTP internal yang masih diperlukan, idealnya dikurangi. |
| Clipboard allow/block list | `ClipboardAllowedForUrls`, `ClipboardBlockedForUrls` | Situs yang boleh/tidak boleh akses clipboard. |
| Window Management allow/block | `WindowManagementAllowedForUrls`, `WindowManagementBlockedForUrls` | Situs enterprise yang memang butuh fitur window management. |
| File picker without gesture | `FileOrDirectoryPickerWithoutGestureAllowedForOrigins` | Umumnya kosong/dihapus, kecuali aplikasi internal jelas butuh. |

---

## 10. Checklist Validasi Akhir

| Area | Validasi |
|---|---|
| Policy masuk | `chrome://policy/` menampilkan policy sebagai machine policy. |
| Registry terbentuk | `HKLM:\SOFTWARE\Policies\Google\Chrome` berisi nilai yang diterapkan. |
| Update aktif | Google Update policy tidak mematikan update. |
| Safe Browsing aktif | Safe Browsing standard/enhanced aktif dan tidak mudah dibypass. |
| Ekstensi terkendali | Ekstensi non-approved diblokir atau dibatasi. |
| Remote access | Chrome Remote Desktop tidak menjadi jalur remote access liar. |
| TLS warning | User tidak bisa sembarang bypass SSL/Safe Browsing warning. |
| Privacy | Sync, telemetry, geolocation, suggestions, dan data collection sesuai kebijakan. |
| DLP | Clipboard/capture/file picker/Web APIs sesuai kebutuhan organisasi. |
| Forensics | Guest/incognito/cache sesuai kebutuhan investigasi. |

---

## 11. Rollback

Sebelum produksi, ekspor GPO:

```powershell
Backup-GPO -Name "CIS - Google Chrome v3.0.0 - L1 Baseline" -Path "D:\GPO-Backup"
```

Untuk rollback registry lokal di lab, hapus policy Chrome:

```powershell
Remove-Item "HKLM:\SOFTWARE\Policies\Google\Chrome" -Recurse -Force
```

Lalu restart Chrome dan cek ulang `chrome://policy/`.

---

## 12. Exception Register

| ID | Policy | Exception | Risiko | Mitigasi | Pemilik | Review |
|---|---|---|---|---|---|---|
| 2.12 | SSLErrorOverrideAllowed | Aplikasi internal sertifikat lama | User bisa masuk situs TLS bermasalah | Migrasi sertifikat ke CA internal valid | Infra | YYYY-MM-DD |
| 2.3.3 | ExtensionInstallBlocklist | Ekstensi password manager resmi perlu allowlist | Ekstensi lain tetap diblokir | Allowlist hanya extension ID resmi | Security | YYYY-MM-DD |

---

## 13. Matriks Implementasi Rekomendasi

### 1. Enforced Defaults

| ID | Profil | Target | Registry/Policy indicator | Catatan implementasi |
|---|---|---|---|---|
| 1.1.1 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:AllowCrossOriginAuthPrompt` | GPO lebih disarankan daripada edit registry langsung. |
| 1.2.1 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\SafeBrowsingAllowlistDomains\\` | GPO lebih disarankan daripada edit registry langsung. |
| 1.2.2 | L1 | Safe Browsing is active in the standard mode. (1) or higher | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:SafeBrowsingProtectionLevel` | Manual: perlu keputusan organisasi dan validasi fungsional. |
| 1.3 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:MediaRouterCastAllowAllIPs` | GPO lebih disarankan daripada edit registry langsung. |
| 1.4 | L1 | Enabled(1) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:BrowserNetworkTimeQueriesEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 1.5 | L1 | Enabled (1) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:AudioSandboxEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 1.6 | L1 | Enabled (1) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:PromptForDownloadLocation` | GPO lebih disarankan daripada edit registry langsung. |
| 1.7 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:BackgroundModeEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 1.8 | L2 | Enabled with a value of Filter top level sites (but not embedded iframes) for adult content (1) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:SafeSitesFilterBehavior` | GPO lebih disarankan daripada edit registry langsung. |
| 1.9 | L1 | Enable all variations (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:ChromeVariations` | Manual: perlu keputusan organisasi dan validasi fungsional. |
| 1.10 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\CertificateTransparencyEnforcementDisabledForLegacyCas\\` | GPO lebih disarankan daripada edit registry langsung. |
| 1.11 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\CertificateTransparencyEnforcementDisabledForCas` | GPO lebih disarankan daripada edit registry langsung. |
| 1.12 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\CertificateTransparencyEnforcementDisabledForUrls` | GPO lebih disarankan daripada edit registry langsung. |
| 1.13 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:SavingBrowserHistoryDisabled` | GPO lebih disarankan daripada edit registry langsung. |
| 1.14 | L1 | Enabled (1) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:DNSInterceptionChecksEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 1.15 | L1 | Enabled (1) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:ComponentUpdatesEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 1.16 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:GloballyScopeHTTPAuthCacheEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 1.17 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:EnableOnlineRevocationChecks` | GPO lebih disarankan daripada edit registry langsung. |
| 1.18 | L1 | Enabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:CommandLineFlagSecurityWarningsEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 1.19 | L1 | Enabled (1) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:ThirdPartyBlockingEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 1.20 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:EnterpriseHardwarePlatformAPIEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 1.21 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:ForceEphemeralProfiles` | GPO lebih disarankan daripada edit registry langsung. |
| 1.22 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:ImportAutofillFormData` | GPO lebih disarankan daripada edit registry langsung. |
| 1.23 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:ImportHomepage` | GPO lebih disarankan daripada edit registry langsung. |
| 1.24 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:ImportSearchEngine` | GPO lebih disarankan daripada edit registry langsung. |
| 1.25 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\HSTSPolicyBypassList` | GPO lebih disarankan daripada edit registry langsung. |
| 1.26 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Chrome\\OverrideSecurityRestrictionsOnInsecureOrigin` | GPO lebih disarankan daripada edit registry langsung. |
| 1.27 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\LookalikeWarningAllowlistDomains` | GPO lebih disarankan daripada edit registry langsung. |
| 1.28 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:SuppressUnsupportedOSWarning` | GPO lebih disarankan daripada edit registry langsung. |
| 1.29 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\WebRtcLocalIpsAllowedUrls` | GPO lebih disarankan daripada edit registry langsung. |

### 2. Attack Surface Reduction

| ID | Profil | Target | Registry/Policy indicator | Catatan implementasi |
|---|---|---|---|---|
| 2.1.1 | L1 | Enabled with a value of Always allow updates (1) or Automatic silent updates (3) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Update:Update{8A69D345-D564-463C-AFF1-A69D9E530F96}` | GPO lebih disarankan daripada edit registry langsung. |
| 2.1.2 | L1 | any value except 0. | `HKEY_LOCAL_MACHINE\\Software\\Policies\\Google\\Update:AutoUpdateCheckPeriodMinutes` | GPO lebih disarankan daripada edit registry langsung. |
| 2.2.1 | L1 | Enabled with the value of Do not allow any site to load mixed content (2) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:DefaultInsecureContentSetting` | GPO lebih disarankan daripada edit registry langsung. |
| 2.2.2 | L2 | Enabled with a value of Do not allow any site to request access to Bluetooth devices via the Web Bluetooth API (2) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:DefaultWebBluetoothGuardSetting` | GPO lebih disarankan daripada edit registry langsung. |
| 2.2.3 | L2 | Enabled with a value of Do not allow any site to request access to USB devices via the WebUSB API (2) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:DefaultWebUsbGuardSetting` | GPO lebih disarankan daripada edit registry langsung. |
| 2.2.4 | L2 | Enabled with a value of Do not allow any site to show desktop notifications (2) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:DefaultNotificationsSetting` | GPO lebih disarankan daripada edit registry langsung. |
| 2.2.5 | L1 | Disabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\PdfLocalFileAccessAllowedForDomains\\` | GPO lebih disarankan daripada edit registry langsung. |
| 2.3.1 | L1 | Enabled (1) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:BlockExternalExtensions` | GPO lebih disarankan daripada edit registry langsung. |
| 2.3.2 | L1 | Enabled with the values of extension, hosted_app, platform_app, theme. | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\ExtensionAllowedTypes:` | GPO lebih disarankan daripada edit registry langsung. |
| 2.3.3 | L1 | Enabled with a value of * | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\ExtensionInstallBlocklist:1` | GPO lebih disarankan daripada edit registry langsung. |
| 2.3.4 | L2 | Enabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\DefaultThirdPartyStoragePartitioningSetting:2` | GPO lebih disarankan daripada edit registry langsung. |
| 2.3.5 | L1 | Configured per organization | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\ThirdPartyStoragePartitioningBlockedForOrigins\\<number>=<url><br>HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\ThirdPartyStoragePartitioningBlockedForOrigins\\1=https://www.example.com` | Manual: perlu keputusan organisasi dan validasi fungsional. |
| 2.3.6 | L2 | (L2) Ensure 'Control Manifest v2 extension availability' Is Set to Forced Only (Automated) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:ExtensionManifestV2Availability` | GPO lebih disarankan daripada edit registry langsung. |
| 2.3.7 | L1 | Disabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:ExtensionUnpublishedAvailability` | GPO lebih disarankan daripada edit registry langsung. |
| 2.4.1 | L2 | Enabled with the value of ntlm, negotiate | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:AuthSchemes` | GPO lebih disarankan daripada edit registry langsung. |
| 2.5.1 | L2 | Enabled with a value of * | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\NativeMessagingBlocklist:1` | GPO lebih disarankan daripada edit registry langsung. |
| 2.6.1 | L1 | Explicitly set to Enabled (1) or Disabled (0) based on the organization's needs. | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:PasswordManagerEnabled` | Manual: perlu keputusan organisasi dan validasi fungsional. |
| 2.7.1 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:CloudPrintProxyEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 2.8.1 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:RemoteAccessHostAllowRemoteAccessConnections` | Manual: perlu keputusan organisasi dan validasi fungsional. |
| 2.8.2 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:RemoteAccessHostAllowUiAccessForRemoteAssistance` | GPO lebih disarankan daripada edit registry langsung. |
| 2.8.3 | L1 | Enabled (1) and at least one domain set | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\RemoteAccessHostClientDomainList` | Manual: perlu keputusan organisasi dan validasi fungsional. |
| 2.8.4 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:RemoteAccessHostRequireCurtain` | GPO lebih disarankan daripada edit registry langsung. |
| 2.8.5 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:RemoteAccessHostFirewallTraversal` | GPO lebih disarankan daripada edit registry langsung. |
| 2.8.6 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:RemoteAccessHostAllowClientPairing` | GPO lebih disarankan daripada edit registry langsung. |
| 2.8.7 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:RemoteAccessHostAllowRelayedConnection` | GPO lebih disarankan daripada edit registry langsung. |
| 2.9.1 | L1 | Disabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:FirstPartySetsEnabled` | Manual: perlu keputusan organisasi dan validasi fungsional. |
| 2.10.1 | L1 | Enabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:CloudAPAuthEnabled` | Manual: perlu keputusan organisasi dan validasi fungsional. |
| 2.11 | L1 | Enabled with a value of Block malicious downloads. Recommended. (4) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:DownloadRestrictions` | GPO lebih disarankan daripada edit registry langsung. |
| 2.12 | L2 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:SSLErrorOverrideAllowed` | GPO lebih disarankan daripada edit registry langsung. |
| 2.13 | L1 | Enabled (1) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:DisableSafeBrowsingProceedAnyway` | GPO lebih disarankan daripada edit registry langsung. |
| 2.14 | L1 | Enabled (1) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:SitePerProcess` | GPO lebih disarankan daripada edit registry langsung. |
| 2.15 | L2 | Enabled (1) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:ForceGoogleSafeSearch` | GPO lebih disarankan daripada edit registry langsung. |
| 2.16 | L1 | Enabled with a value of Show a recurring prompt to the user indicating that a relaunch is required (2) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:RelaunchNotification` | GPO lebih disarankan daripada edit registry langsung. |
| 2.17 | L1 | Enabled and the value of ProxyMode is not set to auto_detect | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:ProxyMode` | GPO lebih disarankan daripada edit registry langsung. |
| 2.18 | L2 | Enabled (1) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:RequireOnlineRevocationChecksForLocalAnchors` | GPO lebih disarankan daripada edit registry langsung. |
| 2.19 | L1 | Enabled with value 86400000 (1 day) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:RelaunchNotificationPeriod` | GPO lebih disarankan daripada edit registry langsung. |
| 2.20 | L1 | Disabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:AllowWebAuthnWithBrokenTlsCerts` | GPO lebih disarankan daripada edit registry langsung. |
| 2.21 | L1 | Disabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:DomainReliabilityAllowed` | GPO lebih disarankan daripada edit registry langsung. |
| 2.22 | L1 | Enabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:EncryptedClientHelloEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 2.23 | L2 | Enabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:EnforceLocalAnchorConstraintsEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 2.24 | L1 | Enabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:EnterpriseProfileCreationKeepBrowsingData` | GPO lebih disarankan daripada edit registry langsung. |
| 2.25 | L1 | Disabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\FileOrDirectoryPickerWithoutGestureAllowedForOrigins\\<number>=<url>` | Butuh nilai organisasi/allowlist/blocklist. Jangan isi sembarang. |
| 2.26 | L1 | Disabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:GoogleSearchSidePanelEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 2.27 | L1 | (L1) Ensure 'Http Allowlist' Is Properly Configured (Manual) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\HttpAllowlist\\<number>=<url><br>HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\HttpAllowlist\\1=www.example.com` | Manual: perlu keputusan organisasi dan validasi fungsional. |
| 2.28 | L1 | Enabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:HttpsUpgradesEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 2.29 | L1 | Disabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:InsecureHashesInTLSHandshakesEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 2.30 | L1 | Enabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:RendererAppContainerEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 2.31 | L1 | Enabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:StrictMimetypeCheckForWorkerScriptsEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 2.32 | L1 | Disabled. | `HKEY_LOCAL_MACHINE\\Software\\Policies\\Google\\Chrome:RemoteDebuggingAllowed` | GPO lebih disarankan daripada edit registry langsung. |

### 3. Privacy

| ID | Profil | Target | Registry/Policy indicator | Catatan implementasi |
|---|---|---|---|---|
| 3.1.1 | L2 | Enabled with a value of Keep cookies for the duration of the session (4) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:DefaultCookiesSetting` | GPO lebih disarankan daripada edit registry langsung. |
| 3.1.2 | L1 | Enabled with a value Do not allow any site to track the users' physical location (2) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:DefaultGeolocationSetting` | GPO lebih disarankan daripada edit registry langsung. |
| 3.2.1 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:EnableMediaRouter` | GPO lebih disarankan daripada edit registry langsung. |
| 3.3 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:PaymentMethodQueryEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 3.4 | L1 | Enabled (1) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:BlockThirdPartyCookies` | GPO lebih disarankan daripada edit registry langsung. |
| 3.5 | L2 | Enabled with a value of Disable browser sign-in (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:BrowserSignin` | GPO lebih disarankan daripada edit registry langsung. |
| 3.6 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:ChromeCleanupReportingEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 3.7 | L1 | Enabled (1) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:SyncDisabled` | GPO lebih disarankan daripada edit registry langsung. |
| 3.8 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:AlternateErrorPagesEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 3.9 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:AllowDeletingBrowserHistory` | GPO lebih disarankan daripada edit registry langsung. |
| 3.10 | L1 | Enabled with a value of Do not predict network actions on any network connection (2) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:NetworkPredictionOptions` | GPO lebih disarankan daripada edit registry langsung. |
| 3.11 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:SpellCheckServiceEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 3.12 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:MetricsReportingEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 3.13 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:SafeBrowsingForTrustedSourcesEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 3.14 | L2 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:SearchSuggestEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 3.15 | L2 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:TranslateEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 3.16 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:UrlKeyedAnonymizedDataCollectionEnabled<br>HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:ScreenCaptureAllowed` | GPO lebih disarankan daripada edit registry langsung. |

### 4. Data Loss Prevention

| ID | Profil | Target | Registry/Policy indicator | Catatan implementasi |
|---|---|---|---|---|
| 4.2.1 | L2 | Do not allow any site to request access to serial ports via the Serial API (2) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:DefaultSerialGuardSetting` | GPO lebih disarankan daripada edit registry langsung. |
| 4.2.2 | L2 | Do not allow any site to access sensors (2) The recommended state for this setting is: Enabled with a value of Do not allow any site to access sensors | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:DefaultSensorsSetting` | GPO lebih disarankan daripada edit registry langsung. |
| 4.2.3 | L1 | Configured per organization | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\ClipboardAllowedForUrls\\<number>=<url><br>HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\ClipboardAllowedForUrls\\1=https://www.example.com` | Manual: perlu keputusan organisasi dan validasi fungsional. |
| 4.2.4 | L1 | Configured per organization | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\ClipboardBlockedForUrls\\<number>=<url><br>HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\ClipboardBlockedForUrls\\1=https://www.example.com` | Manual: perlu keputusan organisasi dan validasi fungsional. |
| 4.2.5 | L1 | (L1) Ensure 'Default clipboard setting' Is 'Enabled' to 'Deny Permissions' (Automated) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:DefaultClipboardSetting` | GPO lebih disarankan daripada edit registry langsung. |
| 4.2.6 | L2 | (L2) Ensure 'Default Window Management permissions setting' Is 'Enabled' to 'Deny Permission' (Automated) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:DefaultWindowManagementSetting` | GPO lebih disarankan daripada edit registry langsung. |
| 4.2.7 | L2 | Configured per organization | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\WindowManagementAllowedForUrls\\<number>=<url><br>HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\WindowManagementAllowedForUrls\\1=https://www.example.com` | Manual: perlu keputusan organisasi dan validasi fungsional. |
| 4.2.8 | L2 | Configured per organization | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\WindowManagementBlockedForUrls\\<number>=<url><br>HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\WindowManagementBlockedForUrls\\1=https://www.example.com` | Manual: perlu keputusan organisasi dan validasi fungsional. |
| 4.3 | L2 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:AllowFileSelectionDialogs` | GPO lebih disarankan daripada edit registry langsung. |
| 4.4 | L2 | Disabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:AudioCaptureAllowed` | GPO lebih disarankan daripada edit registry langsung. |
| 4.5 | L2 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:VideoCaptureAllowed` | GPO lebih disarankan daripada edit registry langsung. |
| 4.6 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:UserFeedbackAllowed` | GPO lebih disarankan daripada edit registry langsung. |
| 4.7 | L2 | Enabled with a value of Enable DNS-over- HTTPS without insecure fallback (secure) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:DnsOverHttpsMode` | GPO lebih disarankan daripada edit registry langsung. |
| 4.8 | L2 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:AutofillAddressEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 4.9 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:AutofillCreditCardEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 4.10 | L1 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:ImportSavedPasswords` | GPO lebih disarankan daripada edit registry langsung. |
| 4.11 | L1 | Enabled with the following text value passwords (Case Sensitive) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome\\SyncTypesListDisabled:1` | GPO lebih disarankan daripada edit registry langsung. |
| 4.12 | L2 | Disabled | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:ScreenCaptureAllowed` | GPO lebih disarankan daripada edit registry langsung. |

### 5. Forensics (Post Incident)

| ID | Profil | Target | Registry/Policy indicator | Catatan implementasi |
|---|---|---|---|---|
| 5.1 | L2 | Disabled (0) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:BrowserGuestModeEnabled` | GPO lebih disarankan daripada edit registry langsung. |
| 5.2 | L2 | Enabled: Incognito mode disabled (1) | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:IncognitoModeAvailability` | GPO lebih disarankan daripada edit registry langsung. |
| 5.3 | L1 | Enabled: 250609664 or greater | `HKEY_LOCAL_MACHINE\\SOFTWARE\\Policies\\Google\\Chrome:DiskCacheSize` | GPO lebih disarankan daripada edit registry langsung. |



---

## 14. Catatan Penting

- Registry script harus diuji di lab. Untuk domain, ubah ke GPO agar terkelola dan dapat diaudit.
- Setting L2 dapat mengganggu situs/aplikasi internal. Gunakan pilot.
- Jangan membuat allowlist luas seperti `*` untuk exception keamanan kecuali benar-benar dipahami risikonya.
- Setelah semua policy diterapkan, dokumentasikan hasil `chrome://policy/` sebagai bukti audit.
