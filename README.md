# CyberShield Windows v3.0

A comprehensive Windows security suite for defense-force and enterprise use. CyberShield consolidates eight specialized security tools into a single, unified interface — enabling rapid threat assessment, malware defense, mobile device auditing, network anomaly detection, and more.

> **Security Scope:** This tool is strictly defensive. All capabilities are designed for authorized monitoring, auditing, and protection of systems you own or have explicit permission to assess. No offensive capabilities are included.

---

## Quick Start

### Pre-built Binary
```
CyberShield-Windows/bin/Release/net8.0-windows/CyberShield.exe
```

### Rebuild from Source
```bash
dotnet publish CyberShield-Windows/CyberShield.csproj \
  -c Release \
  -r win-x64 \
  --self-contained true \
  -p:PublishSingleFile=true \
  -p:PublishTrimmed=false \
  -o ./publish
```

---

## Features

| Tab | Name | Description |
|-----|------|-------------|
| 1 | **System Scanner** | Deep scan of running processes, services, startup entries, and scheduled tasks for anomalies |
| 2 | **Ransomware Guard** | Real-time file-system watcher covering 40+ extension families; kills suspicious processes automatically |
| 3 | **Mobile Auditor** | Parses Android APK manifests for dangerous permissions, security flags, and code-protection checks |
| 4 | **IOC Feed** | Built-in threat intelligence feed of IPs, domains, hashes, and YARA rules with source attribution |
| 5 | **Net Anomaly** | Captures live traffic and scores each connection with a weighted risk model |
| 6 | **Log Analyzer** | Ingests Windows Event Logs and flat-file logs; surfaces high-severity events with filters |
| 7 | **Vuln Scanner** | CVE-based vulnerability scanner cross-referencing installed software against NVD/NIST data |
| 8 | **Report Engine** | Generates PDF/HTML audit reports aggregating findings from all other tabs |

---

## Project Structure

```
CyberShield-Windows/
├── CyberShield.sln
├── CyberShield.csproj          # .NET 8 Windows target, NuGet refs
├── Program.cs                  # Entry point → MainForm
├── MainForm.cs / .Designer.cs  # Tab host shell
│
├── Tabs/
│   ├── SystemScannerTab.cs     # Process/service/startup/task scanner
│   ├── RansomwareGuardTab.cs   # FileSystemWatcher + kill logic
│   ├── MobileAuditorTab.cs     # APK parser → permission/flag checks
│   ├── IOCFeedTab.cs           # IOC database + matching engine
│   ├── NetAnomalyTab.cs        # Packet capture + risk scoring
│   ├── LogAnalyzerTab.cs       # EventLog + flat-file ingestion
│   ├── VulnScannerTab.cs       # CVE cross-reference engine
│   └── ReportEngineTab.cs      # PDF/HTML report builder
│
├── Core/
│   ├── IOCDatabase.cs          # Embedded IOC store (IP, domain, hash, YARA)
│   ├── RiskScorer.cs           # Shared weighted scoring logic
│   ├── ProcessHelper.cs        # Win32 process utilities
│   └── ReportBuilder.cs        # iTextSharp/HTML report generation
│
├── Models/
│   ├── ScanResult.cs
│   ├── IOCEntry.cs
│   ├── NetworkConnection.cs
│   └── AuditReport.cs
│
└── Resources/
    ├── ioc_feed.json           # Default built-in IOC data
    ├── yara_rules/             # Bundled YARA rule files
    └── cvss_weights.json       # CVSS scoring configuration

Dependency chain:
  MainForm → Tab classes → Core/* → Models/*
  ReportEngineTab → ReportBuilder → iTextSharp / RazorLight
  NetAnomalyTab → SharpPcap / PacketDotNet
  MobileAuditorTab → ICSharpCode.SharpZipLib (APK = ZIP)
```

---

## Mobile Auditor — Detail

### Permission Risk Table

| Permission | Risk Level | Notes |
|------------|-----------|-------|
| READ_CONTACTS | High | PII exposure |
| READ_SMS | Critical | Credential harvesting vector |
| RECEIVE_SMS | Critical | SMS interception |
| READ_CALL_LOG | High | Behavioral profiling |
| RECORD_AUDIO | Critical | Covert audio capture |
| CAMERA | High | Visual surveillance |
| ACCESS_FINE_LOCATION | High | Precise tracking |
| ACCESS_BACKGROUND_LOCATION | Critical | Silent background tracking |
| READ_EXTERNAL_STORAGE | Medium | Broad file access |
| WRITE_EXTERNAL_STORAGE | Medium | Data exfiltration path |
| PROCESS_OUTGOING_CALLS | High | Call interception |
| BIND_DEVICE_ADMIN | Critical | Administrative control |
| INSTALL_PACKAGES | Critical | Dropper capability |
| REQUEST_INSTALL_PACKAGES | Critical | Sideload enablement |
| BIND_ACCESSIBILITY_SERVICE | Critical | Input capture / overlay |
| SYSTEM_ALERT_WINDOW | High | Overlay phishing |
| GET_ACCOUNTS | High | Account enumeration |
| USE_CREDENTIALS | Critical | Direct credential theft |
| CHANGE_NETWORK_STATE | Medium | Network manipulation |
| RECEIVE_BOOT_COMPLETED | Medium | Persistence mechanism |

### Security Flags Checked

| Flag / Attribute | Risk if Absent/Set |
|------------------|--------------------|
| `android:debuggable="true"` | Debug access enabled — High |
| `android:allowBackup="true"` | Full data backup possible — Medium |
| `android:networkSecurityConfig` absent | Cleartext traffic allowed — High |
| `android:usesCleartextTraffic="true"` | HTTP traffic allowed — High |
| `android:exported="true"` on Activities | Unauthorized component access — Medium |
| `android:sharedUserId` set | UID sharing attack surface — Medium |
| `minSdkVersion` < 21 | Missing modern security APIs — Low |
| `targetSdkVersion` < 29 | Missing scoped storage + perms — Medium |

### Code Protection Checks

| Check | Detection Method |
|-------|-----------------|
| Obfuscation | Class/method name entropy analysis |
| Root detection | Known root-check API call patterns |
| Emulator detection | Known emulator fingerprint checks |
| SSL pinning | TrustManager / OkHttp pinner patterns |
| Anti-tamper | Signature verification call patterns |
| Native libraries | Presence of .so files in APK |
| Dynamic code loading | DexClassLoader / PathClassLoader refs |
| Reflection abuse | High-frequency java.lang.reflect usage |

---

## Ransomware Guard — Covered Extensions

The watcher monitors creation/modification of files with any of the following extensions (grouped by family):

**Documents:** .doc .docx .xls .xlsx .ppt .pptx .pdf .odt .ods .odp .rtf .txt .csv .xml .json .yaml .yml .toml .ini .cfg .conf

**Media:** .jpg .jpeg .png .gif .bmp .tiff .mp3 .mp4 .avi .mov .mkv .flac .wav

**Archives / Backup:** .zip .rar .7z .tar .gz .bak .backup .sql .db .sqlite .mdb .accdb

**Code / Dev:** .py .js .ts .cs .java .cpp .c .h .php .rb .go .rs .swift .kt

**System / Keys:** .pem .key .crt .cer .pfx .p12 .kdbx .wallet .dat

**Ransomware note extensions:** .encrypted .locked .crypt .ransom .zepto .cerber .locky .wncry .petya .ryuk .revil .conti .lockbit .blackcat .dharma

> Suspicious process heuristics: processes writing to >50 unique files in <30 seconds are flagged and optionally terminated after user confirmation.

---

## IOC Feed — Built-in Indicators

| Category | Count | Sources |
|----------|-------|---------|
| Malicious IPs | 150+ | Emerging Threats, AbuseIPDB snapshots |
| Malicious Domains | 200+ | URLhaus, OpenPhish, PhishTank |
| File Hashes (MD5/SHA256) | 300+ | MalwareBazaar, VirusTotal public feed |
| YARA Rules | 25+ | Yara-Rules project, Neo23x0, custom |
| User-Agent Strings | 50+ | Known C2 / RAT beacons |
| Registry Keys (persistence) | 40+ | ATT&CK T1547 mappings |

### Sample Built-in IOC Table

| Type | Indicator | Threat Family | Confidence |
|------|-----------|--------------|------------|
| Domain | update-srv[.]top | Emotet C2 | High |
| Domain | cdn-delivery[.]xyz | Qakbot loader | High |
| IP | 185.220.101.0/24 | Tor exit / C2 range | Medium |
| Hash (SHA256) | 44d88612fea8a8f36de82e1278abb02f | EICAR test (reference) | Known |
| YARA | rule Mirai_Botnet | IoT botnet detection | High |
| User-Agent | `python-requests/2.18.4` pattern | Generic beacon | Medium |

---

## Net Anomaly — Risk Signals

All signals are weighted and combined into a 0–100 risk score per connection. Connections scoring ≥ 60 are flagged; ≥ 80 are auto-alerted.

| Signal | Weight | Description |
|--------|--------|-------------|
| IOC match (IP/domain) | 40 | Direct hit against built-in IOC feed |
| Known malicious port | 10 | Ports: 4444, 6666, 31337, 1337, 8080 (non-browser), etc. |
| Beaconing pattern | 15 | Regular interval connections (±5% jitter) to same host |
| High data volume outbound | 10 | >50 MB in single session to external IP |
| Non-standard protocol on standard port | 10 | e.g. non-HTTP traffic on port 80 |
| DNS over non-53 port | 8 | DNS tunneling indicator |
| Connection to Tor exit node | 15 | Tor exit node IP list match |
| GeoIP anomaly | 7 | First-ever connection to country code |
| Low TTL / fast-flux DNS | 8 | DNS TTL < 300s on repeated queries |
| Self-signed / expired certificate | 7 | TLS cert validation failure |
| DGA-like domain | 12 | High entropy domain name (Shannon > 3.5) |
| Lateral movement pattern | 20 | Internal IP scanning >5 hosts in <60s |
| Process name mismatch | 15 | e.g. svchost.exe connecting on non-standard port |
| Parent process anomaly | 10 | Unexpected parent (e.g. Word spawning netcat) |

---

## Tech Stack

| Component | Library / API | Version |
|-----------|--------------|---------|
| UI Framework | Windows Forms (.NET 8) | 8.0 |
| Packet Capture | SharpPcap | 6.3.0 |
| Packet Decoding | PacketDotNet | 1.4.7 |
| APK Parsing | ICSharpCode.SharpZipLib | 1.3.3 |
| PDF Reports | iTextSharp (LGPL) | 5.5.13 |
| HTML Reports | RazorLight | 2.3.1 |
| YARA Matching | dnYara | 1.2.0 |
| JSON Handling | System.Text.Json | 8.0 |
| Logging | NLog | 5.2.8 |
| Charting | LiveCharts2 | 0.9.8 |
| GeoIP | MaxMind GeoLite2 | DB: monthly |
| Win32 APIs | P/Invoke (kernel32, advapi32, ntdll) | OS native |
| CVE Data | NVD JSON feeds (NIST) | Daily sync |
| Threat Intel | Emerging Threats rules (open) | Weekly sync |

---

## Security Scope

CyberShield Windows is a **defensive security tool** built for:

- **Defense-force / military IT security teams** conducting authorized endpoint assessments
- **Enterprise SOC analysts** monitoring for threats on managed infrastructure
- **Security researchers** auditing mobile applications and network traffic in lab environments

**This tool does NOT and is NOT designed to:**
- Exploit vulnerabilities
- Bypass authentication on systems you do not own
- Intercept communications without authorization
- Conduct offensive operations of any kind

All network capture requires appropriate OS permissions (WinPcap/Npcap driver) and should only be used on networks you are authorized to monitor. Mobile APK auditing is intended for APKs you own, have been provided for assessment, or are analyzing in a sandboxed lab.

**Compliance note:** Users are responsible for ensuring their use complies with applicable laws and regulations, including the Computer Fraud and Abuse Act (CFAA), the Computer Misuse Act (UK), and equivalent legislation in their jurisdiction.

---

*CyberShield v3.0 — Built for defenders.*
