### Overview
A combined external threat actor and insider threat attack against a 600-employee financial firm. The attacker compromised a low-privilege user's account (Barry Shmelly), escalated through credential theft via Mimikatz, seized the domain controller, and deployed ransomware to 306 machines. Simultaneously, an insider (Jane Smith) conducted independent crypto wallet theft and data exfiltration over 27 days via FTP.

### Attack Timeline

```
Feb 2, 2024  03:32  → First recon: 8 machines run "systeminfo"
Feb 1, 2024         → Actor (143.38.175.105) logs into Barry Shmelly's account
Feb 1, 2024         → 9 phishing emails sent from Barry's compromised account
Feb 1, 2024         → 41QI-LAPTOP first infected laptop
Feb 5, 2024         → 12,716 bytes of data sent to 182.56.23.121 (Jane Smith / FTP)
Feb 17, 2024 02:30  → files_go_byebye.exe appears on network (UL8R-MACHINE first)
Feb 17, 2024 02:34  → Ransom note YOU_GOT_CRYTOED_SO_GIMME_CRYPTO.txt on 306 machines
                       50 files encrypted with .umadbro extension
```
> **Ransomware was active for 15 days before detection.**

### Attack Chain — External Threat Actor

```
Compromised account: barry_shmelly@encryptoderafinancial.com
Actor IP: 143.38.175.105
        ↓
Phishing → 9 employees targeted (including Robin Kirby, Valerie Orozco)
File lure: Employee_Contact_List_Updated_March_2024.docx.exe
        ↓
Lateral movement: Robin Kirby → Valerie Orozco's machine (lysmith/vaorozco = 10.10.0.138 / 10.10.0.18)
        ↓
Credential dump: totally_not_mimikatz.exe "sekurlsa::logonpasswords"
        ↓
Domain Controller compromise: DOMAIN_CONTROLLER_SERVER (lihenry_domain_admin)
        ↓
Group Policy abuse: gpupdate /force on 306 devices
PowerShell payload (Base64): pulls files_go_byebye.exe from notification-finance-services.com
        ↓
Ransomware execution: start /b C:\ProgramData\files_go_byebye.exe -encrypt -target C:\Users\ -ext .umadbro
        ↓
306 machines encrypted | Ransom note: YOU_GOT_CRYTOED_SO_GIMME_CRYPTO.txt
Contact: OopsALLYOURFileSAreWithUs@snailmail.com
```

### Attack Chain — Insider Threat (Jane Smith)

```
jane_smith@encryptoderafinancial.com | Blockchain Contractor | GOTI-LAPTOP
        ↓
Recon: cold storage crypto wallet locations
Contact: elboss@westealurcrypto.com
        ↓
Downloads: ftp_client.exe + crypto_stealer.exe
Target directory: C:\Users\jasmith\ToTheMoon\
        ↓
FTP exfiltration → 182.56.23.121 (27 days | 208,138 bytes total)
Scheduled daily, password: Ugot2muchCRYTOw3llt4k3it0FFurH4ND5
All config encoded in PowerShell (Base64)
```

### Indicators of Compromise (IOCs)

| Type | Value |
|------|-------|
| Actor IP | `143.38.175.105` |
| Actor IP (Distrib.) | `211.152.115.93` |
| Exfil IP (Jane) | `182.56.23.121` |
| Phishing Sender | `bitbingersbanking.net` |
| C2 Domain | `notification-finance-services.com` |
| Ransom Contact | `OopsALLYOURFileSAreWithUs@snailmail.com` |
| Malicious Executables | `files_go_byebye.exe`, `totally_not_mimikatz.exe`, `screenconnect_client.exe`, `ftp_client.exe`, `crypto_stealer.exe`, `Company_Financials_Q1_2024_Review.xlsx.exe` |
| Ransomware Extension | `.umadbro` |
| Compromised Accounts | `bashmelly`, `lysmith`, `vaorozco`, `lihenry_domain_admin`, `jasmith` |
| Sensitive Files Staged | `SECRET_MergersAndAcquisitions_Strategy2025.docx`, `ExecutiveSalaryNegotiations.docx`, `Encryptodera_Proprietary_Algorithms.zip` |

### Tools & Techniques (MITRE ATT&CK)

| Technique | ID | Tool / Method |
|-----------|-----|---------------|
| Phishing: Malicious Attachment | T1566.001 | .docx.exe double-extension lures |
| Credential Dumping | T1003.001 | totally_not_mimikatz.exe (Mimikatz) |
| Domain Controller Compromise | T1078.002 | lihenry_domain_admin credentials |
| Group Policy Modification | T1484.001 | gpupdate /force → ransomware GPO |
| Obfuscated Files (Base64) | T1027 | PowerShell Base64 payload |
| Remote Access Software | T1219 | screenconnect_client.exe |
| Exfiltration via FTP | T1048.003 | ftp_client.exe → 182.56.23.121 |
| Data Encrypted for Impact | T1486 | files_go_byebye.exe (.umadbro) |
| Insider Threat | — | Jane Smith — crypto theft + data exfil |
