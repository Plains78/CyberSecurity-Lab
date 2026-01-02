## Information Disclosure via Misconfigured Backup Files on Apache Web Server ##

## ⚠️ Disclaimer
**For Educational Purposes Only.** All activities documented in this repository were conducted within an isolated virtual environment (VirtualBox/VMware). Unauthorized access to computer systems or networks is strictly prohibited and is a criminal offense under the **Criminal Code of Canada**. The primary purpose of these exercises is to develop skills for defensive security, log analysis, and SOC operations.

### 1. Introduction & Environment & Setting
   - Purpose: To demonstrate how misconfigured backup files can lead to unintended disclosure of sensitive information.
   - Tool: Kali Linux (Attacker) , Ubuntu Server (Victim)
   - Setting:
        a. NETWORK setting: For the communication between attaker and victim server, I use virtual box (Host only Adapter)
        b. vulnerable Source Code in 'config.php'
     <p align="center">
         <img width="500" height="auto" alt="php setting" src="https://github.com/user-attachments/assets/64b12230-10ce-4b8e-9861-50262d7bb8ff" />
    


    This simulates a common administrative mistake where sensitive files are copied and left unprotected in the web root directory.

<p align="center">
  <img src="https://github.com/user-attachments/assets/5b04c9bf-e6bb-4414-9c0d-38c3095ab5a9" width="500" alt="backup" />
</p>

<br>
  
### 2. Step-by-Step Analysis

 (0) IP Identification & Target Selection

* **Victim (Ubuntu Server):** 192.168.59.4
<p align="center">
  <img src="https://github.com/user-attachments/assets/abf88a07-b72a-4eaa-9ea9-fb5d7eb1a0d4" width="600" alt="ipAddress" />
</p>

* **Attacker (Kali Linux):** 192.168.59.3
<p align="center">
  <img src="https://github.com/user-attachments/assets/b6ad8b31-6f2f-4e89-bada-3b4f40253b41" width="600" alt="attacker_ip" />
</p>

---

 (1) Reconnaissance: Service Identification

I used `nmap` and `curl -I` to verify the target's open ports and server version.

* **Port Scanning:** Confirmed port 80/tcp is OPEN.
<p align="center">
  <img src="https://github.com/user-attachments/assets/08702d30-8d77-4812-bd00-6a5a9edc6579" width="600" alt="CheckPortOpenOrNot" />
</p>

* **HTTP Header Analysis:** Identified Apache/2.4.41 on Ubuntu.
<p align="center">
  <img src="https://github.com/user-attachments/assets/9368fa01-ba94-4bbe-a0b0-ad449922e797" width="600" alt="check" />
</p>

---

 (2) Log Forensics

I checked the server's access logs to identify suspicious activity.

- **Log Path:** `/var/log/apache2/access.log`
- **Finding:** Detected a `GET /config.php.bak` request from the attacker's IP (192.168.59.3).

---

 (3) Information Disclosure (Exploitation)

Credentials were exposed due to improper backup file handling. I used `curl` to retrieve the database credentials.

<p align="center">
  <img src="https://github.com/user-attachments/assets/890a2699-8934-44e9-9b16-e6cb3c5ffa75" width="600" alt="image" />
</p>

- **Impact:** Hardcoded credentials (`$user`, `$pw`) were leaked in plaintext.

<br>
<br>

### 3. Remediation & Prevention

 a. File Hygiene
- **Action:** Reduced the attack surface by removing unnecessary files (.bak, .old, .swp) from the web root directory (`/var/www/html/`).

<p align="center">
  <img src="https://github.com/user-attachments/assets/724264f0-64dd-4c16-bdaf-52379c3a9d56" width="600" alt="protection" />
</p>

---

 b. Apache Configuration
- **Action:** Modified `apache2.conf` to block public access to backup and temporary files using the `<FilesMatch>` directive.

<p align="center">
  <img src="https://github.com/user-attachments/assets/179e7a38-e978-4688-90c1-5d2eb9b0865c" width="600" alt="Solve" />
</p>

- **Result:** The server now returns a **403 Forbidden** error, confirming that credentials are no longer exposed.

<p align="center">
  <img src="https://github.com/user-attachments/assets/c57cdd31-e857-4067-83e4-e066da641e51" width="600" alt="finallyPrevent" />
</p>

---

c. Secure Coding & Best Practices
- **Action:** Avoid hardcoding sensitive information (passwords, API keys) in plaintext within source files.
- **Alternative:** Implement **Environment Variables** or use a dedicated **Secret Manager** for secure credential handling.

---

 d. Troubleshooting
- **Configuration Error:** Resolved an Apache service failure by correcting a syntax error (e.g., misspelled directives) in `apache2.conf`.
- **File System Issue:** Identified a directory/file name conflict using `ls -al` and successfully re-created `config.php` as a regular file instead of a directory.
