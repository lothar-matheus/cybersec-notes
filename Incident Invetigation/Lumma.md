# Incident Investigation Report: Lumma Stealer via Click Fix Phishing
**Event ID:** SOC338 - 316  
**Analysis Date:** March 13, 2025  
**Analyst:** [Matheus Lemos]  
**Verdict:** True Positive (TP)  

---

## 1. Executive Summary
A highly targeted phishing campaign was identified using the **"Click Fix"** (also known as *Pastejacking*) technique to distribute the **Lumma Stealer** malware. The attack disguised itself as a legitimate Windows 11 Pro system upgrade notification. Investigation confirmed that the user `dylan@letsdefend.io` interacted with the malicious link, leading to host compromise and data exfiltration to the IP address `132.232.40.201`.

## 2. Attack Vector Analysis (Click Fix Phishing)

The **Click Fix** technique differs from traditional phishing by leveraging psychological engineering rather than direct file attachments. The identified workflow was:

1.  **The Lure:** An email from `update@windows-update.site` offered a "Free Windows 11 Pro Upgrade." It utilized urgency and financial incentive as primary triggers.
2.  **Deceptive Landing Page:** Upon clicking the link, the user was redirected to a page simulating a system error or a Windows security verification.
3.  **The Manipulation (Click Fix):** The page featured a "Fix It" or "Copy Code" button. Interaction with this button automatically copied a malicious, often Base64-encoded PowerShell script to the user’s **clipboard**.
4.  **Induced Execution:** On-screen instructions guided the user to press `Win + R`, type `cmd` or `powershell`, and press `Ctrl + V` followed by `Enter`.
    * *Technical Note:* This bypasses Secure Email Gateways (SEG) because the malicious payload is never "delivered" via email; it is generated in the browser and manually executed by the user.

## 3. Malware Behavior: DLL Side-Loading
Once the command was executed, Lumma Stealer employed **DLL Side-Loading (MITRE T1574.002)**:
* The script downloaded a ZIP archive containing a legitimate, signed executable and a malicious DLL renamed to match a real system library.
* When the legitimate executable was launched, it automatically loaded the malicious DLL from the same directory, allowing the malware to run under the context of a trusted process, evading signature-based EDR detections.

## 4. Evidence & Findings

### 4.1. Network Analysis
* **SMTP IP:** `132.232.40.201`
    * **Geolocation:** Tencent Cloud (China).
    * **Reputation:** Confirmed C2 (Command & Control) infrastructure in threat intelligence databases (VirusTotal/AbuseIPDB).
    * **Role:** Acted as both the phishing source and the final destination for stolen data (Exfiltration Point).

### 4.2. Host Investigation (Endpoint)
* **Web History:** Confirmed access to the domain `windows-update.site` at the time of the alert.
* **Process Logs:** Identified `cmd.exe` execution correlating with the alert timestamp, with evidence of PowerShell commands used to fetch the initial payload.

## 5. Indicators of Compromise (IOCs)

| Type | Value | Description |
| :--- | :--- | :--- |
| **IP Address** | `132.232.40.201` | C2 Infrastructure / Malicious SMTP |
| **Domain** | `windows-update.site` | Click Fix Landing Page Host |
| **Email** | `update@windows-update.site` | Phishing Sender |
| **Malware** | `Lumma Stealer` | Infostealer Family |

## 6. Containment & Mitigation
* **Immediate Action:** The host was isolated from the network to halt active data exfiltration.
* **Remediation:** Deleted all temporary files and unauthorized executables within the user's `%AppData%` and `%Temp%` directories.
* **Prevention:** * Blocked the malicious IP and domain at the edge firewall and proxy.
    * Recommended implementation of Attack Surface Reduction (ASR) rules to block child process creation by browsers.
    * Scheduled Security Awareness training focused on "Click Fix" and "Pastejacking" tactics.

---
**Conclusion:** The incident was successfully contained before lateral movement occurred.