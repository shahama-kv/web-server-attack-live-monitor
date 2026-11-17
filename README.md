# web-server-attack-live-monitor
A beginner cybersecurity project demonstrating Nginx web server attack stimulation an real-time log monitoring using Ubuntu, Kali Linux an a Windows web browser.

Project Overview:
The goal of this project is to stimulate a real-world attack-and-defense scenario in an isolate virtual environment while optimizing system resource on an **8GB RAM** host machine.

In this lab:
An **Ubuntu VM** hosts a live web server.
A **Kali Linux VM** performs automated directory brute-forcing to discover hidden endpoints.
The **Windows Host** acts as a legitimate client and monitors server behavior.
**Fail2Ban** is configured on ubuntu to detect suspicious '404' spikes in real time and automatically block the attacker's IP without disrupting legitimate users.

-------------

Lab Architecture:

All three machines communicates over an isolated **VirtualBox Host-Only Network** ('192.168.0/24'), while both VMs retain access via a separate **NAT** for package installation.
Victim lab - Ubuntu: Hosts **Nginx** web server
Legitimate user - Windows: Hypervisor host & normal web client
Attacker - Kali Linux: Executes directory brute-force scans

--------------

Tools & Technologies Use:

**Virtualization**: Oracle VertualBox
**Web Server**: Nginx
**Attack tool**: 'dirb' (Web Content Scanner on Kali Linux)
**Log Analysis**: Linux 'tail' utility ('/var/log/nginx/access.log')
**Intrusion Prevention**: Fail2Ban

--------------

## Lab Notes

**1. Network Setup**
I connected both Ubuntu and Kali using two network adapters: **NAT** and **Host-Only**. 
* *Why:* I needed NAT so the VMs could reach the internet to download tools, and Host-Only so Windows, Ubuntu, and Kali could talk to each other in a safe, private bubble away from my home Wi-Fi.

**2. Spinning Up the Web Server**
I installed **Nginx** on the Ubuntu VM (`sudo apt install nginx`) and opened its IP in my Windows browser to make sure it loaded.
* *Why:* I needed a realistic target to attack. Testing it from my Windows host instead of opening a browser inside Ubuntu saved a ton of RAM on my 8GB laptop.

**3. Running the Attack & Watching Logs**
I opened Ubuntu's live visitor log (`sudo tail -f /var/log/nginx/access.log`) and launched a `dirb` scan from Kali (`dirb http://192.168.56.101`).
* *Why:* I wanted to see what a directory brute-force attack actually looks like from the defender's seat. Watching the logs scroll live made it obvious how noisy automated tools are—Kali's IP flooded the screen with `404` errors in seconds.

<img width="816" height="455" alt="Screenshot 2026-11-17 195056" src="https://github.com/user-attachments/assets/2af82695-7057-4223-980d-4eb49bd5aa16" />


**4. Automating the Defense**
I installed **Fail2Ban** and added a rule in `/etc/fail2ban/jail.local` to ban any IP that triggers 5 `404` errors within 60 seconds.
* *Why:* Reading logs manually is too slow to stop an automated script. I wanted Ubuntu to react on its own and drop the attacker at the firewall level without me having to step in.

```ini
[nginx-botsearch]
enabled = true
port = http,https
filter = nginx-botsearch
logpath = /var/log/nginx/access.log
maxretry = 5
findtime = 60
bantime = 600
```
<img width="812" height="615" alt="Screenshot 2026-11-17 194015" src="https://github.com/user-attachments/assets/4324d8ea-86d8-4b3b-bbfa-6e3823190355" />

**5. Testing the Block**
I ran `dirb` from Kali again, and the scan froze right away. Checking `sudo fail2ban-client status nginx-botsearch` confirmed Kali's IP (`192.168.56.102`) was banned.
* *Why:* I refreshed the site from my Windows host right after, and it still loaded fine. That proved my rule only blocked malicious scanning behavior while keeping normal traffic working.

<img width="712" height="552" alt="Screenshot 2026-11-17 195117" src="https://github.com/user-attachments/assets/5a9201e4-2f21-4d69-851e-8349bbf88946" />

<img width="701" height="540" alt="Screenshot 2026-11-17 194804" src="https://github.com/user-attachments/assets/99bdc459-6251-41de-b71a-f5089c749928" />


<img width="721" height="127" alt="Screenshot 2026-11-17 195039" src="https://github.com/user-attachments/assets/8e49a392-22d2-4015-b64b-b3b4fa747ed7" />

**What I Learned**
* A sudden burst of `404` errors in access logs is an immediate giveaway for directory scanning.
* Logs give visibility, while tools like Fail2Ban turn that visibility into instant automated defense.
* Offloading tasks to the Windows host makes multi-VM labs run smoothly even on 8GB of RAM.

