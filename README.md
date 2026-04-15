# INCIDENT RESPONSE REPORT

**Azuki Import/Export Trading Co.**

*Threat Hunt Exercise — Training Purpose*

---

| Field | Detail |
|---|---|
| **Date of Report:** | 2026-04-10 |
| **Incident ID:** | IR-2025-001 |
| **Analyst:** | Ramzy Aboughlia |
| **Severity Level:** | CRITICAL |
| **Report Status:** | Contained |
| **Escalated To:** | SOC Manager / CISO |
| **Affected Organization:** | Azuki Import/Export Trading Co. |
| **Affected System:** | AZUKI-SL (IT Admin Workstation) |
| **Incident Date:** | 2025-11-19 |

---

## SUMMARY OF FINDINGS

- An external threat actor gained initial access to the IT admin workstation (AZUKI-SL) by compromising the account **kenji.sato** via brute-force from IP `88.97.178.12`.
- The attacker deployed a malicious PowerShell script (`wupdate.ps1`) downloaded from a C2 server at `78.141.196.6:8080` to automate the attack chain.
- Windows Defender was disabled by adding exclusions for `.bat`, `.ps1`, `.exe` extensions and the staging directory `C:\ProgramData\WindowsCache`, allowing payloads to execute undetected.
- Credentials were stolen from LSASS memory using a renamed Mimikatz binary (`mm.exe`) with the `sekurlsa::logonpasswords` module.
- Collected data was compressed into `export-data.zip` and exfiltrated to a Discord webhook over HTTPS using `curl.exe`.
- A backdoor account named `support` was created and elevated to administrator for persistent access.
- A scheduled task named `Windows Update Check` was created to execute a malicious `svchost.exe` daily at 02:00 as SYSTEM.
- The attacker cleared Security, System, and Application event logs using `wevtutil.exe` to destroy forensic evidence.
- Lateral movement was performed to internal target `10.1.0.188` using stolen credentials and Remote Desktop Protocol (RDP) via `mstsc.exe`.

---

## WHO, WHAT, WHEN, WHERE, WHY, HOW

### WHO

**Attacker**

- Initial access source IP: `88.97.178.12`
- Secondary brute-force IP: `115.247.157.74`
- Internal pivot IP: `10.0.8.9` (remote device: `vm00000b`)
- C2 Server: `78.141.196.6:8080`
- Exfiltration platform: Discord (`discord.com/api/webhooks`)

**Compromised**

- **Primary Account:** `kenji.sato` — IT admin account used as initial foothold
- **Created Backdoor Account:** `support` — local account created and elevated to Administrator
- **Primary System:** `AZUKI-SL` — IT admin workstation
- **Lateral Movement Secondary Target:** `10.1.0.188` — Internal file server (`fileadmin` account)

### WHAT

1. Brute-force attack on `kenji.sato` account from external IP `88.97.178.12`.
2. Successful logon established — attacker gains foothold on AZUKI-SL.
3. PowerShell script `wupdate.ps1` downloaded and executed from C2 server — attack chain automated.
4. Windows Defender exclusions added for `.bat`, `.ps1`, `.exe` and staging directory path.
5. Payloads downloaded via `certutil.exe` (LOLBin) — `mm.exe` (Mimikatz) and fake `svchost.exe`.
6. Network reconnaissance performed using `ipconfig` and `arp` commands.
7. Backdoor account `support` created and elevated to Administrator.
8. Credentials dumped from LSASS using `mm.exe` with `sekurlsa::logonpasswords`.
9. Collected data compressed into `export-data.zip` using PowerShell `Compress-Archive`.
10. Data exfiltrated to Discord webhook via `curl.exe` over HTTPS (port 443).
11. Scheduled task `Windows Update Check` created for daily persistence at 02:00 SYSTEM.
12. Event logs cleared (Security, System, Application) using `wevtutil.exe`.
13. Lateral movement to `10.1.0.188` using `cmdkey` + `mstsc` with stolen `fileadmin` credentials.

### WHEN

| Date/Time (UTC) | Event | Details |
|---|---|---|
| 2025-11-19 (Early) | Initial Brute-Force | IP `88.97.178.12` begins login attempts against `kenji.sato` |
| 2025-11-19 (Early) | Initial Access Confirmed | `kenji.sato` successfully authenticated from `88.97.178.12` |
| 2025-11-19 ~18:00 | Script Download | `wupdate.ps1` downloaded from `78.141.196.6:8080` to `AppData\Local\Temp` |
| 2025-11-19 18:49:27 | AV Evasion | Windows Defender exclusions added for `.bat`, `.ps1`, `.exe` |
| 2025-11-19 18:49:29 | Path Exclusion Added | WindowsCache and Temp paths excluded from Defender scanning |
| 2025-11-19 ~18:50 | Payload Download | `certutil` downloads `mm.exe` and `svchost.exe` from C2 to staging dir |
| 2025-11-19 ~19:00 | Recon | `ipconfig` and `arp` commands executed for network discovery |
| 2025-11-19 ~19:00 | Persistence #1 | Backdoor account `support` created and added to Administrators |
| 2025-11-19 ~19:10 | Credential Dump | `mm.exe` executed — `sekurlsa::logonpasswords` dumps LSASS memory |
| 2025-11-19 ~19:15 | Collection | `export-data.zip` created in `C:\ProgramData\WindowsCache` |
| 2025-11-19 ~19:20 | Exfiltration | `curl.exe` uploads `export-data.zip` to Discord webhook over HTTPS |
| 2025-11-19 ~19:25 | Persistence #2 | Scheduled task `Windows Update Check` created — daily 02:00 SYSTEM |
| 2025-11-19 ~19:30 | Log Clearing | `wevtutil` clears Security, System, Application logs |
| 2025-11-19 19:05:11 | Lateral Movement Recon | `cmdkey /list` executed to enumerate saved credentials |
| 2025-11-19 19:10:37 | Credential Staging | `cmdkey` stores `fileadmin` credentials for `10.1.0.188` |
| 2025-11-19 19:10:41 | Lateral Movement | `mstsc /v:10.1.0.188` — RDP session established to file server |

### WHERE

**Compromised System**

- Hostname: `AZUKI-SL` (IT Admin Workstation)

**Attacker Infrastructure**

- Initial access IP: `88.97.178.12` (External — Remote desktop/brute-force)
- Secondary attack IP: `115.247.157.74` (External — Failed admin brute-force)
- Internal pivot: `10.0.8.9` / `vm00000b`
- C2 Server: `78.141.196.6:8080` (Payload hosting and command)
- Exfiltration endpoint: `discord.com/api/webhooks` (HTTPS port 443)

**Malware Locations**

- `C:\ProgramData\WindowsCache\mm.exe` — Renamed Mimikatz credential dumper
- `C:\ProgramData\WindowsCache\svchost.exe` — Malicious persistence payload
- `C:\ProgramData\WindowsCache\export-data.zip` — Compressed stolen data
- `C:\Users\kenji.sato\AppData\Local\Temp\wupdate.ps1` — Initial attack script
- `C:\Users\kenji.sato\Downloads\WindowsUpdate.bat` — Hidden batch file
- `C:\Users\kenji.sato\WindowsUpdate.bat` — Hidden batch file (copy)

**Lateral Movement Target**

- `10.1.0.188` — Internal file server (accessed via RDP using `fileadmin` credentials)

### WHY

**Root Cause**

- Weak or reused credentials on the `kenji.sato` IT admin account allowed successful brute-force from an external IP.
- No account lockout policy or MFA was enforced on the IT admin workstation.
- Overly permissive Windows Defender configuration allowed registry-based exclusions to be added by a non-SYSTEM process.
- No network egress filtering to block outbound connections to raw IP addresses and non-standard ports.

**Attacker Objective**

- **Primary:** Data theft from Azuki Import/Export — shipping logistics, client data, financial records.
- **Secondary:** Establish persistent access for continued long-term surveillance and re-entry.
- **Tertiary:** Lateral movement to internal file server (`10.1.0.188`) to access sensitive company files.

### HOW — Attack Chain

1. **[Initial Access]** Brute-force attack from `88.97.178.12` against `kenji.sato` account. Successful logon via Remote Desktop. Account confirmed as IT admin with high privileges.
2. **[Execution]** PowerShell script `wupdate.ps1` downloaded from C2 server (`78.141.196.6:8080`) using `Invoke-WebRequest` with `-WindowStyle Hidden` and `-ExecutionPolicy Bypass`. Script saved to Temp directory disguised as Windows Update.
3. **[Defense Evasion]** Windows Defender exclusions added via registry for `.bat`, `.ps1`, `.exe` extensions and staging directory paths. This allowed all subsequent payloads to execute without AV detection.
4. **[C2 Download]** `certutil.exe` (LOLBin) used to download `mm.exe` (renamed Mimikatz) and fake `svchost.exe` from C2 server to `C:\ProgramData\WindowsCache` staging directory.
5. **[Discovery]** Network reconnaissance performed using `ipconfig` and `arp` commands to map the internal network and identify lateral movement targets.
6. **[Persistence #1]** Backdoor local account `support` created and added to Administrators group for guaranteed re-entry independent of `kenji.sato` account.
7. **[Credential Access]** `mm.exe` executed with `privilege::debug` and `sekurlsa::logonpasswords` modules to dump all credentials from LSASS memory including plaintext passwords and NTLM hashes.
8. **[Collection]** Sensitive files and credential dump output compressed into `export-data.zip` using PowerShell `Compress-Archive` stored in staging directory.
9. **[Exfiltration]** `curl.exe` used to upload `export-data.zip` to a Discord webhook over HTTPS (port 443) — traffic blending with normal web traffic to avoid detection.
10. **[Persistence #2]** Scheduled task `Windows Update Check` created to execute `C:\ProgramData\WindowsCache\svchost.exe` daily at 02:00 as SYSTEM — ensures payload survives reboots.
11. **[Defense Evasion #2]** `wevtutil.exe` used to clear Security, System, and Application event logs — destroying forensic evidence of the entire attack.
12. **[Lateral Movement]** `cmdkey` used to store stolen `fileadmin` credentials for `10.1.0.188`. `mstsc.exe` then opened RDP session to internal file server using those credentials, pivoting from AZUKI-SL as a launchpad.

---

## IMPACT ASSESSMENT

**Actual Impact**

- **Credential compromise:** All accounts logged into AZUKI-SL had credentials stolen from LSASS memory — including domain accounts, potentially giving attacker access to multiple systems.
- **Data exfiltration:** Company data compressed and successfully sent to external Discord webhook — scope of stolen data unknown without file server forensics.
- **File server accessed:** Lateral movement to `10.1.0.188` (`fileadmin`) means attacker had access to shipping logistics, client data, and financial records of Azuki Import/Export.
- **Persistent access maintained:** Two persistence mechanisms remain (backdoor account `support` + scheduled task) — attacker can re-enter at any time.
- **Forensic evidence destroyed:** Event log clearing means full scope of attacker activity may never be completely recovered.
- **Windows Defender compromised:** AV exclusions remain in registry until manually removed — endpoint protection is effectively disabled for key file types.

| Category | Impact | Severity |
|---|---|---|
| Confidentiality | Data stolen — shipping, client, financial records | CRITICAL |
| Integrity | Backdoor account + scheduled task installed | HIGH |
| Availability | Forensic evidence destroyed via log clearing | HIGH |
| Credential Security | All LSASS credentials compromised | CRITICAL |
| Network Security | Lateral movement to internal file server | CRITICAL |

**Overall Risk Level:**

> **CRITICAL — Full system compromise with data exfiltration, persistent access, and lateral movement confirmed.**

---

## RECOMMENDATIONS

### IMMEDIATE (Within 24 Hours)

- Isolate AZUKI-SL from the network immediately to prevent further lateral movement or C2 communication.
- Disable and delete the backdoor account `support` from the system.
- Remove the malicious scheduled task `Windows Update Check` from Task Scheduler.
- Block all outbound connections to `78.141.196.6` and `88.97.178.12` at the firewall level.
- Reset `kenji.sato` password and all other accounts whose credentials were exposed on AZUKI-SL.
- Remove Windows Defender exclusions added by the attacker from the registry.
- Preserve disk image of AZUKI-SL for forensic analysis before any remediation.
- Investigate `10.1.0.188` (file server) for signs of compromise — check logs, new accounts, file access.

### SHORT-TERM (1–7 Days)

- Deploy MFA on all remote access methods — especially RDP and VPN for IT admin accounts.
- Implement account lockout policy: lock accounts after 5 failed login attempts within 10 minutes.
- Block outbound HTTPS to Discord webhooks and raw IP addresses at the network perimeter.
- Enable Windows Defender Tamper Protection to prevent registry-based AV exclusion changes.
- Audit all local administrator accounts across all systems — remove unnecessary privileged accounts.
- Deploy a SIEM alert for `certutil.exe`, `mstsc.exe`, `cmdkey.exe`, and `wevtutil.exe` executions.
- Review and audit all scheduled tasks across the network for malicious entries.

### LONG-TERM

- Implement Privileged Access Workstations (PAW) for IT admin tasks — separate admin from daily-use machines.
- Deploy a network-based IDS/IPS to detect lateral movement and C2 communication patterns.
- Enable LSASS protection (RunAsPPL) to prevent credential dumping via tools like Mimikatz.
- Conduct regular threat hunting exercises and red team assessments to identify gaps before attackers do.
- Implement Credential Guard on all Windows systems to protect credentials in memory.
- Establish a formal Incident Response plan with defined playbooks for common attack types.
- Provide security awareness training for all employees — phishing and credential hygiene focus.

---

## APPENDIX

### A. Indicators of Compromise (IOCs)

| Category | Indicator | Description |
|---|---|---|
| Attacker IP | `88.97.178.12` | Initial access brute-force source IP |
| Attacker IP | `115.247.157.74` | Secondary brute-force attempt IP |
| Internal IP | `10.0.8.9` | Internal pivot point (`vm00000b`) |
| C2 Server | `78.141.196.6:8080` | Payload hosting server — certutil downloads |
| C2 URL | `http://78.141.196.6:8080/AdobeGC.exe` | Mimikatz download URL (disguised as Adobe) |
| C2 URL | `http://78.141.196.6:8080/svchost.exe` | Persistence payload download URL |
| C2 URL | `http://78.141.196.6:8080/wupdate.ps1` | Initial attack script download URL |
| Exfiltration | `discord.com/api/webhooks/...` | Discord webhook used for data exfiltration |
| Lateral Target | `10.1.0.188` | Internal file server — lateral movement target |
| Malicious File | `mm.exe` | Renamed Mimikatz in `C:\ProgramData\WindowsCache` |
| Malicious File | `svchost.exe` (fake) | Persistence payload in `C:\ProgramData\WindowsCache` |
| Malicious File | `wupdate.ps1` | Initial PowerShell attack script in Temp dir |
| Malicious File | `export-data.zip` | Compressed stolen data staged for exfiltration |
| Malicious File | `WindowsUpdate.bat` | Hidden batch file in kenji.sato Downloads |
| Backdoor Account | `support` | Local admin backdoor account created by attacker |
| Scheduled Task | `Windows Update Check` | Malicious persistence scheduled task — daily 02:00 |
| Staging Dir | `C:\ProgramData\WindowsCache` | Attacker staging directory for tools and data |
| Registry Key | `HKLM\...\Defender\Exclusions\Extensions` | `.bat` `.ps1` `.exe` excluded from AV scanning |
| Registry Key | `HKLM\...\Defender\Exclusions\Paths` | Staging and Temp paths excluded from AV |
| Compromised Account | `kenji.sato` | Primary compromised IT admin account |
| Compromised Account | `fileadmin` | File server admin account — credentials stolen via Mimikatz |

### B. MITRE ATT&CK Mapping

| Tactic | Technique | ID | Evidence |
|---|---|---|---|
| Initial Access | Valid Accounts — Brute Force | T1078 / T1110 | `kenji.sato` compromised from `88.97.178.12` |
| Execution | Command & Scripting: PowerShell | T1059.001 | `wupdate.ps1` — ExecutionPolicy Bypass |
| Execution | Command & Scripting: Windows Cmd | T1059.003 | `WindowsUpdate.bat` executed with hide flag |
| Persistence | Create Account: Local Account | T1136.001 | Backdoor account `support` created + admin |
| Persistence | Scheduled Task/Job | T1053.005 | `schtasks` `Windows Update Check` — daily 02:00 |
| Defense Evasion | Impair Defenses: Disable AV | T1562.001 | Registry exclusions for `.bat` `.ps1` `.exe` |
| Defense Evasion | Masquerading | T1036 | `mm.exe`, fake `svchost.exe`, `wupdate.ps1` |
| Defense Evasion | Clear Windows Event Logs | T1070.001 | `wevtutil cl` Security/System/Application |
| Discovery | System Network Config Discovery | T1016 | `ipconfig` and `arp` commands executed |
| Credential Access | OS Credential Dumping: LSASS | T1003.001 | `mm.exe` `sekurlsa::logonpasswords` |
| Collection | Archive Collected Data | T1560.001 | `Compress-Archive` `export-data.zip` |
| Command & Control | Ingress Tool Transfer | T1105 | `certutil` LOLBin downloads from C2 |
| Exfiltration | Exfil Over Web Service | T1567 | `curl.exe` upload to Discord webhook HTTPS |
| Lateral Movement | Use Alternate Auth Material | T1550 | `cmdkey` + `mstsc` with stolen `fileadmin` creds |
| Lateral Movement | Remote Desktop Protocol | T1021.001 | `mstsc /v:10.1.0.188` |

### C. Investigation Timeline

| Time (UTC) | Event | Details |
|---|---|---|
| 2025-11-19 Early AM | Brute-force begins | Multiple failed logins against `kenji.sato` from `88.97.178.12` |
| 2025-11-19 Early AM | Initial access confirmed | Successful logon — `kenji.sato` authenticated via RDP |
| 2025-11-19 ~18:00 | Attack script deployed | `wupdate.ps1` downloaded from C2 with hidden PowerShell |
| 2025-11-19 18:49:27 | AV exclusions added | Defender registry exclusions for `.bat` `.ps1` `.exe` |
| 2025-11-19 18:49:29 | Path exclusions added | Defender path exclusions for staging and Temp dirs |
| 2025-11-19 ~18:50 | Payloads downloaded | `certutil` downloads `mm.exe` and `svchost.exe` to WindowsCache |
| 2025-11-19 ~19:00 | Network recon | `ipconfig` + `arp` executed for internal network mapping |
| 2025-11-19 ~19:00 | Backdoor created | Account `support` created and added to Administrators |
| 2025-11-19 ~19:10 | Credential dumping | `mm.exe` runs Mimikatz `sekurlsa::logonpasswords` vs LSASS |
| 2025-11-19 ~19:15 | Data collection | `export-data.zip` created in staging directory |
| 2025-11-19 ~19:20 | Data exfiltration | `curl.exe` uploads zip to Discord webhook over HTTPS |
| 2025-11-19 ~19:25 | Scheduled task created | Persistence task `Windows Update Check` — daily 02:00 SYSTEM |
| 2025-11-19 ~19:28 | Log clearing | `wevtutil` clears Security, System, Application logs |
| 2025-11-19 19:05:11 | Lateral recon | `cmdkey /list` — enumerate stored credentials |
| 2025-11-19 19:10:37 | Lateral cred staging | `cmdkey` stores `fileadmin` creds for `10.1.0.188` |
| 2025-11-19 19:10:41 | Lateral movement | `mstsc /v:10.1.0.188` — RDP to internal file server |

### D. Key Investigation Queries

**Query 1: Initial Access — Logon Events**

```kql
DeviceLogonEvents
| where DeviceName == "azuki-sl"
| where Timestamp between (datetime(2025-11-19) .. datetime(2025-11-20))
| sort by Timestamp asc
| project AccountName, ActionType, LogonType, RemoteIP, RemoteDeviceName, RemotePort, RemoteIPType, IsLocalAdmin
```

> **Result:** Identified `88.97.178.12` as initial access IP and `kenji.sato` as the compromised account.

**Query 2: AV Exclusions — Registry Extensions**

```kql
DeviceRegistryEvents
| where DeviceName == "azuki-sl"
| where Timestamp between (datetime(2025-11-19) .. datetime(2025-11-20))
| where RegistryKey contains "Windows Defender\\Exclusions\\Extensions"
| sort by Timestamp asc
| project Timestamp, RegistryKey, RegistryValueName, RegistryValueData, InitiatingProcessCommandLine
```

> **Result:** Confirmed exclusions added for `.bat`, `.ps1`, `.exe` at 18:49:27–29 UTC.

**Query 3: AV Exclusions — Paths**

```kql
DeviceRegistryEvents
| where DeviceName == "azuki-sl"
| where Timestamp between (datetime(2025-11-19) .. datetime(2025-11-20))
| where RegistryKey contains "Windows Defender\\Exclusions\\Paths"
| sort by Timestamp asc
| project Timestamp, RegistryKey, RegistryValueName, RegistryValueData, InitiatingProcessCommandLine
```

> **Result:** `C:\Users\KENJI~1.SAT\AppData\Local\Temp` and `C:\ProgramData\WindowsCache` excluded from scanning.

**Query 4: Payload Downloads — Process Commands**

```kql
DeviceProcessEvents
| where DeviceName == "azuki-sl"
| where AccountName == "kenji.sato"
| where Timestamp between (datetime(2025-11-19) .. datetime(2025-11-20))
| sort by Timestamp asc
| distinct ProcessCommandLine
```

> **Result:** Identified `certutil` LOLBin downloads, `mm.exe` execution, `schtasks` persistence, `wupdate.ps1` execution.

**Query 5: Credential Dumping Tool**

```kql
DeviceFileEvents
| where DeviceName == "azuki-sl"
| where FolderPath contains "WindowsCache"
| where Timestamp between (datetime(2025-11-19) .. datetime(2025-11-20))
| project Timestamp, FileName, FolderPath, ActionType, InitiatingProcessCommandLine
```

> **Result:** `mm.exe` identified as renamed Mimikatz in `C:\ProgramData\WindowsCache` staging directory.

**Query 6: Data Exfiltration — Network Events**

```kql
DeviceNetworkEvents
| where DeviceName == "azuki-sl"
| where Timestamp between (datetime(2025-11-19) .. datetime(2025-11-20))
| where InitiatingProcessAccountName == "kenji.sato"
| where RemotePort == 443
| project Timestamp, RemoteUrl, RemoteIP, RemotePort, InitiatingProcessFileName, InitiatingProcessCommandLine
| sort by Timestamp asc
```

> **Result:** `curl.exe` uploading `export-data.zip` to `discord.com/api/webhooks` over HTTPS port 443.

**Query 7: Event Log Clearing**

```kql
DeviceProcessEvents
| where DeviceName == "azuki-sl"
| where FileName == "wevtutil.exe"
| where Timestamp between (datetime(2025-11-19) .. datetime(2025-11-20))
| where InitiatingProcessAccountName == "kenji.sato"
| distinct ProcessCommandLine
```

> **Result:** Security log cleared first, followed by System and Application — attacker prioritized hiding authentication events.

**Query 8: Lateral Movement**

```kql
DeviceProcessEvents
| where DeviceName == "azuki-sl"
| where Timestamp between (datetime(2025-11-19) .. datetime(2025-11-20))
| where InitiatingProcessAccountName == "kenji.sato"
| where FileName in ("cmdkey.exe", "mstsc.exe")
| project Timestamp, FileName, ProcessCommandLine, AccountName
| sort by Timestamp asc
```

> **Result:** `cmdkey /list` → `cmdkey` stores `fileadmin` creds → `mstsc /v:10.1.0.188` confirms RDP lateral movement to file server.

### E. Evidence Reference

- **Screenshot 1:** Windows Defender registry exclusions for `.bat`, `.ps1`, `.exe` extensions (18:49:27 UTC)
- **Screenshot 2:** Windows Defender path exclusions for Temp and WindowsCache directories
- **Screenshot 3:** `wevtutil.exe` log clearing commands — Security cleared first
- **Screenshot 4:** `cmdkey` + `mstsc` lateral movement commands showing `10.1.0.188` as target

---

***END OF REPORT — CONFIDENTIAL — FOR TRAINING PURPOSES ONLY***

*Analyst: Ramzy Aboughlia | Date: 2026-04-10 | Incident ID: IR-2025-001*
