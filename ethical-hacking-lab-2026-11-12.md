# Ethical Hacking Lab — November 12, 2026

## Overview

**Environment:** Kali Linux ARM virtual machine in VMware Fusion on macOS  
**Scope:** Self-owned local lab, `127.0.0.1` (loopback)  
**Tools:** Python 3 HTTP server, Nmap, curl, Gobuster  
**Goal:** Practice service discovery, HTTP inspection, directory enumeration, simulated information exposure, remediation, and verification.

> All tests were performed against a local practice server. The exposed files contained fabricated training text, not actual confidential data.

## 1. Build a local web server

Created a practice directory and simple webpage:

```bash
mkdir -p ~/hacking-lab
cd ~/hacking-lab
echo "Welcome to my hacking lab" > index.html
python3 -m http.server 8000 --bind 127.0.0.1
```

The server listened on TCP port **8000**, bound to the loopback interface so it was not exposed to other machines.

## 2. Discover the service with Nmap

```bash
nmap -sV -p 8000 127.0.0.1
```

Observed:

```text
8000/tcp open  http  SimpleHTTPServer 0.6 (Python 3.13.14)
```

**Learned:** An open port indicates a listening service; service fingerprinting can identify software, but does not by itself establish a vulnerability.

## 3. Inspect HTTP headers

```bash
curl -I http://127.0.0.1:8000
```

Observed an `HTTP/1.0 200 OK` response and a `Server: SimpleHTTP/0.6 Python/3.13.14` header. The response also included content type, content length, and modification metadata.

**Learned:** HTTP headers reveal how a server responds and sometimes identify software versions. Version disclosure alone does not prove insecurity.

## 4. Simulate an exposed document

```bash
cd ~/hacking-lab
echo "CONFIDENTIAL - Employee payroll information" > payroll.txt
curl http://127.0.0.1:8000/payroll.txt
```

The server returned the fabricated document without authentication. Moved it outside the web root and retested:

```bash
mkdir -p ~/private-lab
mv ~/hacking-lab/payroll.txt ~/private-lab/
curl -i http://127.0.0.1:8000/payroll.txt
```

Observed **404 File not found** after remediation.

**Learned:** Files placed inside a served directory can be directly retrieved. Moving sensitive content outside the web root prevents this simple form of accidental publication.

## 5. Discover directories with Gobuster

Created test directories:

```bash
mkdir -p ~/hacking-lab/admin ~/hacking-lab/backups ~/hacking-lab/uploads
printf "admin\nbackups\nuploads\nsecret\nlogin\n" > ~/hacking-lab/wordlist.txt
```

Ran a small, controlled enumeration:

```bash
gobuster dir -u http://127.0.0.1:8000 -w ~/hacking-lab/wordlist.txt -t 2
```

Gobuster **v3.8.2** discovered `admin`, `backups`, and `uploads`, each returning **301** redirects to the trailing-slash URL.

**Learned:** A 301 redirect is not itself a vulnerability. Directory enumeration can identify unlinked paths from a supplied wordlist.

## 6. Demonstrate directory listing and direct file access

```bash
echo "INTERNAL BACKUP - Training data only" > ~/hacking-lab/backups/database-backup.txt
curl http://127.0.0.1:8000/backups/
curl http://127.0.0.1:8000/backups/database-backup.txt
```

The directory listing displayed `database-backup.txt`, and a direct request returned its training contents.

**Learned:** Directory listing can reveal filenames, while direct file access is a separate concern. Disabling directory listings alone does not necessarily prevent access to a known URL.

## 7. Remediate and retest

```bash
mv ~/hacking-lab/backups/database-backup.txt ~/private-lab/
curl -I http://127.0.0.1:8000/backups/database-backup.txt
curl http://127.0.0.1:8000/backups/
```

Observed:

- Direct file request: **404 File not found**.
- Directory listing: still enabled, but the listing was **empty**.

**Conclusion:** The simulated sensitive file was no longer accessible at its former URL. The lab server still supports directory listing; that feature would need separate configuration or a different production-grade web server if it were undesirable.

## Skills practiced

- Local lab setup and safe scoping
- TCP port and service discovery with Nmap
- HTTP response header inspection with curl
- Directory enumeration with Gobuster
- Distinguishing 200, 301, and 404 HTTP responses
- Demonstrating accidental file exposure
- Remediation and verification
- Recording evidence and distinguishing observations from assumptions

## Next steps

1. Learn how to capture local HTTP requests using Wireshark on the loopback interface.
2. Practice writing short findings with impact, evidence, remediation, and retest results.
3. Set up an intentionally vulnerable training application such as OWASP Juice Shop in an isolated lab.
4. Explore authentication and authorization flaws only in owned or explicitly authorized systems.

## Lab cleanup

Stop the Python HTTP server with **Ctrl+C** in its terminal. Keep fabricated files for future practice, or remove them when no longer needed.

---

*Learning journal based on commands and terminal screenshots observed during the November 12, 2026 session.*
