# Funbox-1 Writeup

## Objective
Gain root access on the target machine.

## Tools Used
- Netdiscover
- Nmap
- Gobuster
- WPScan
- pspy

## Reconnaissance
Performed target identification using `netdiscover`

`sudo netdiscover -r 192.168.11.0/24`

![Netdiscover](screenshots/Funbox-1/netdiscover.png)

Performed initial port scanning using Nmap:

`nmap -sS -sV 192.168.11.4 -T4`

Discovered the following open ports:
- 21 (FTP)
- 22 (SSH)
- 80 (HTTP)

## Enumeration
Started web enumeration on port 80.

- Used Gobuster for directory brute-forcing
- Discovered:
  - /robots.txt (no useful data)
  - /secret (no useful data)
  - /wp-admin (indicating WordPress application)

Performed WordPress enumeration using WPScan:

- Enumerated valid users:
  - admin
  - joe

- Conducted password brute-force using rockyou.txt
- Successfully found valid credentials for both users

## Exploitation
Attempted SSH login using discovered credentials.

- User `joe` was able to login via SSH using the same password obtained from WPScan

## Privilege Escalation
After gaining SSH access:

- Explored system directories
- Found a directory belonging to another user (`funny`)
- Located a file: `.backup.sh`

Key observations:
- File had **777 permissions (world writable)**
- Suspected it was executed by a cron job

Used `pspy` tool to monitor background processes:
- Confirmed `.backup.sh` was being executed periodically by `root` user and user `funny` himself

### Exploitation Steps:
- Modified `.backup.sh` to include a reverse shell payload
- Set up a listener on Kali Linux
- Waited for cron job execution
- Got shell for user `funny` and removed him from cronjob using `crontab -r`
- Waited for cron job execution for `root` user 

## Result
- Successfully received a reverse shell as a `root`
- Changed directory to root and achieved `root flag`

## Key Learnings
- Importance of proper enumeration (especially WordPress)
- Weak credential reuse across services can lead to compromise
- Misconfigured file permissions (777) are critical vulnerabilities
- Cron jobs can be abused for privilege escalation
- Tools like `pspy` are very effective for detecting scheduled tasks
