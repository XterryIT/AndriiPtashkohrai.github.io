---
layout: default
title: "Advanced Threat Hunting Lab"
permalink: /projects/threat-hunting/
---

# Advanced Threat Hunting Lab: Detecting APT29 (Cozy Bear) Tactics

In response to the increasing sophistication of state-sponsored cyber threats, I designed an advanced threat hunting laboratory to emulate and detect the tactics, techniques, and procedures (TTPs) of **APT29 (Cozy Bear)**. This project focuses on simulating real-world attacks using **Atomic Red Team**, capturing telemetry with **Sysmon**, and engineering robust detection rules in **Splunk**.

## 🏗️ Lab Architecture & Tooling

The environment was built using isolated virtual machines to safely execute malicious payloads while capturing comprehensive logs:
* **Victim Machine:** Windows 10 running Sysmon v15.15 (SwiftOnSecurity profile with custom additions) and Splunk Universal Forwarder.
* **SIEM & Analytics:** Kali Linux hosting Splunk Enterprise v9.3.2 (acting as Indexer and Search Head).
* **Emulation Framework:** Atomic Red Team for executing MITRE ATT&CK techniques.

## 🕵️‍♂️ Threat Profiling: APT29 (Cozy Bear)
APT29 is a highly disciplined threat actor attributed to Russia's SVR, known for long-term espionage and "living-off-the-land" (LotL) techniques. To effectively hunt this adversary, I focused on their preferred stealth mechanisms, such as avoiding malware drops in favor of native binaries (WMI, PowerShell), bypassing UAC, and exfiltrating data via alternative protocols.

## 🛡️ Attack Emulation & SPL Detection Rules

Here are some of the key APT29 techniques I simulated, along with the custom Splunk Processing Language (SPL) queries I authored to detect them.

### 1. Application Layer Protocol: DNS (T1071.004)
**The Attack:** APT29 often uses DNS for Command and Control (C2) communication to bypass standard HTTP/HTTPS filtering. I simulated a high-volume DNS query attack (fast-flux/DGA behavior) to a target domain (`file.io` / `nip.io`).
**The Detection:** The following SPL correlates PowerShell execution with a high volume of DNS queries to suspicious subdomains within a 1-minute window using the `transaction` command.

```spl
index=main source="WinEventLog:Microsoft-Windows-Sysmon/Operational"
(EventCode=22 QueryName="atomicredteam-*.127.0.0.nip.io")
OR ( EventCode=1 Image="*\\powershell.exe" CommandLine="*Resolve-DnsName *nip.io*" )
| transaction ProcessGuid startswith="EventCode=1" maxspan=1m
| table _time, host, User, QueryName, CommandLine, eventcount
```

### 2. Impair Defenses: Disable Windows Event Logging (T1562.002)
**The Attack:** To blind defenders, attackers may terminate the EventLog service threads. I executed `Invoke-Phant0m`, a script designed to kill Windows Event Log Service threads without stopping the service itself.
**The Detection:** I monitored Sysmon EventCode 11 (Process Access) to catch unauthorized scripts accessing the EventLog service.

```spl
index=main source="WinEventLog:Microsoft-Windows-Sysmon/Operational" 
EventCode=11 Image="*\\powershell*" TargetFilename="*Invoke-Phant0m*" 
User!="SYSTEM" User!="Administrator"
| table _time, host, User, Image, TargetFilename
```

### 3. Bypass User Account Control (T1548.002)
**The Attack:** Bypassing UAC by hijacking the registry keys associated with `eventvwr.msc` (`HKCU\Software\Classes\mscfile\shell\open\command`). This allows arbitrary code execution with `SYSTEM` privileges without prompting the user.
**The Detection:** The rule flags instances where `mmc.exe` or `cmd.exe` launches `eventvwr.msc`, explicitly filtering out standard system accounts to minimize false positives.

```spl
index=main sourcetype="WinEventLog:Microsoft-Windows-Sysmon/Operational" 
EventCode=1 
(Image="*\\mmc.exe" AND CommandLine="*eventvwr.msc*") OR 
(Image="*\\cmd.exe" AND CommandLine="*/c eventvwr.msc*")
| table _time, host, User, EventCode, Image, CommandLine
```

### 4. HTML Smuggling (T1027.006)
**The Attack:** Delivering malicious payloads (like ISOs or EXEs) hidden inside HTML/JavaScript, decrypted client-side via `mshta.exe` to bypass network proxies.
**The Detection:** Direct monitoring of `mshta.exe` execution combined with suspicious file names originating from web requests.

```spl
index=main source="WinEventLog:Microsoft-Windows-Sysmon/Operational"
EventCode=1 Image="*\\mshta.exe" CommandLine="*T1027_006_remote.html*"
| where NOT User IN ("SYSTEM","Administrator")
| table _time, host, User, Image, CommandLine, ParentCommandLine
```

## 📉 Conclusions & Takeaways

By emulating APT29, I demonstrated that relying purely on static signatures is insufficient. The use of "Living off the Land" binaries (like `mshta.exe` and `wmic.exe`) requires behavioral analysis and correlation across multiple Sysmon event types. 

A critical lesson learned was the necessity of rigorous filtering: initial rules generated significant noise from standard administrative tasks. By chaining events with Splunk's `transaction` command and filtering out `SYSTEM` accounts, I drastically reduced false positives, creating high-fidelity alerts suitable for a production SOC environment.

<div class="back-link-container">
  <a href="/" class="btn-back">← Back to Home</a>
</div>