🛡️ Docker Cybersecurity Lab | Offensive & Defensive Security Practice
A sandboxed, educational cybersecurity lab built with Docker containers to simulate real-world attack and defense scenarios.
 Strictly for academic and learning purposes only.

📋 Table of Contents
Overview
Lab Architecture
Prerequisites
Setup & Installation
Lab Tasks
Task 1 — Network Connectivity Test
Task 2 — Brute Force Attack
Task 3 — SQL Injection
Task 4 — Command Injection
Tools & Technologies
AI/ML Algorithms for Edge Devices
Learning Outcomes
Disclaimer

Overview
This lab creates an isolated Docker-based environment with two containers:
Container
Role
Image
attacker
Offensive machine (Kali Linux)
kalilinux/kali-rolling
victim
Vulnerable web application (DVWA)
vulnerables/web-dvwa

Both containers communicate over a private Docker bridge network (lab-network), simulating a real attacker-victim network scenario — completely isolated from the host system.

Lab Architecture
┌─────────────────────────────────────────────┐
│             Host Machine (Windows)           │
│                                             │
│  ┌──────────────┐      ┌─────────────────┐  │
│  │   attacker   │      │     victim      │  │
│  │  Kali Linux  │◄────►│   DVWA (PHP)    │  │
│  │              │      │  Port 80 (HTTP) │  │
│  └──────────────┘      └─────────────────┘  │
│          │                     │            │
│          └─────────────────────┘            │
│                 lab-network                  │
│              (Docker bridge)                 │
└─────────────────────────────────────────────┘


Prerequisites
OS: Windows 10/11 (64-bit)
Docker Desktop — Download here
WSL2 (Windows Subsystem for Linux 2) — Required for Docker on Windows
RAM: Minimum 4 GB free
Storage: ~1.5 GB free (for Docker images)
Internet Connection — For pulling Docker images
Enable WSL2 (if not already enabled)
Open PowerShell as Administrator and run:
wsl --install --no-distribution
wsl --update
wsl --set-default-version 2

Restart your PC after installation.

Setup & Installation
1. Clone or Create the Project Folder
mkdir cyber-lab
cd cyber-lab

2. Create docker-compose.yml
Create a file named docker-compose.yml (not docker-compose.yml.txt) with the following content:
version: '3.8'

services:
  attacker:
    image: kalilinux/kali-rolling
    container_name: attacker
    command: sleep infinity
    networks:
      - lab-network
    tty: true
    stdin_open: true

  victim:
    image: vulnerables/web-dvwa
    container_name: victim
    ports:
      - "80:80"
    networks:
      - lab-network

networks:
  lab-network:
    driver: bridge

💡 Tip: Use VS Code or Notepad++ to create this file so the .txt extension is not added automatically.
3. Start the Lab
docker compose up -d

Expected output:
✔ Network cyber-lab_lab-network  Created
✔ Container victim               Started
✔ Container attacker             Started

4. Verify Containers Are Running
docker ps
<img width="1006" height="507" alt="Screenshot 2026-09-21 001948" src="https://github.com/user-attachments/assets/ddfd6aed-6741-48ec-9c8a-a0c6169a323e" />

Both attacker and victim should show Up status.
5. Set Up DVWA (Victim Web App)
Open your browser and go to: http://localhost:80
Login with default credentials:
Username: admin
Password: password
Click "Create / Reset Database"
Log in again
Go to DVWA Security → Set level to Low → Click Submit
<img width="1267" height="917" alt="Screenshot 2026-09-21 002419" src="https://github.com/user-attachments/assets/3b74d650-e6cc-45e9-b288-213b4b06e5e9" />

Lab Tasks
Task 1 — Network Connectivity Test (Ping)
Objective: Verify attacker can communicate with the victim over the lab network.
# Access attacker container
docker exec -it attacker bash

# Install ping tool
apt-get update && apt-get install -y iputils-ping

# Ping victim (discover its IP first)
getent hosts victim

# Ping victim container
ping -c 4 <victim-ip>

Expected Result:
4 packets transmitted, 4 received, 0% packet loss


Task 2 — Brute Force Attack
Objective: Use Hydra to perform a dictionary-based brute force attack against the DVWA login page.
# Install Hydra and wordlist
apt-get install -y hydra wordlists
gunzip /usr/share/wordlists/rockyou.txt.gz

# Run brute force attack
hydra -l admin -P /usr/share/wordlists/rockyou.txt <victim-ip> \
  http-get-form \
  "/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:incorrect"

What This Demonstrates:
How weak passwords are vulnerable to automated guessing
Why rate-limiting and account lockout policies are essential
The power of dictionary attacks on systems with no security controls

Task 3 — SQL Injection
Objective: Extract database records by injecting malicious SQL into a web form.
In browser: navigate to DVWA → SQL Injection
In the User ID field, enter:
1' OR '1'='1

Click Submit
What Happens Behind the Scenes:
-- Normal query
SELECT * FROM users WHERE id = '1'

-- Injected query (always TRUE)
SELECT * FROM users WHERE id = '1' OR '1'='1'
-- Returns ALL rows from the users table

What This Demonstrates:
How unsanitized input leads to unauthorized database access
Why parameterized queries / prepared statements are critical
The risk of exposing sensitive user data through web forms

Task 4 — Command Injection
Objective: Execute arbitrary OS commands on the server by injecting system commands into a web input.
In browser: navigate to DVWA → Command Injection
In the IP address field, enter:
127.0.0.1; whoami

Or to read system files:
127.0.0.1; cat /etc/passwd

What This Demonstrates:
How web apps that pass user input to OS commands are critically vulnerable
Why server-side input validation and sanitization is mandatory
How attackers can escalate from web access to system-level access

Stopping the Lab
# Stop all containers
docker compose down

# Or just pause (keeps data)
docker compose stop


Tools & Technologies
Tool
Purpose
Docker Desktop
Container management platform
Kali Linux
Attacker OS with pre-installed security tools
DVWA
Intentionally vulnerable web application for practice
Hydra
Network login brute-force tool
Nmap
Network scanner and port discovery
iputils-ping
Network connectivity testing
rockyou.txt
Common password wordlist for dictionary attacks
WSL2
Windows Subsystem for Linux — enables Linux containers on Windows



Learning Outcomes
After completing this lab, students are able to:
✅ Set up and manage multi-container environments using Docker Compose
✅ Understand the attacker-victim model in cybersecurity
✅ Perform and document a brute force attack using Hydra
✅ Identify and exploit SQL Injection vulnerabilities
✅ Demonstrate Command Injection on a vulnerable web application
✅ Understand why security controls like input validation, rate limiting, and parameterized queries are essential

Common Troubleshooting
Error
Solution
no configuration file provided
Rename docker-compose.yml.txt → docker-compose.yml
Virtualization not detected
Enable Hyper-V and WSL2 via Windows Features
WSL not installed
Run wsl --install in PowerShell (Admin)
Port 80 already in use
Change "80:80" to "8080:80" in compose file
command not found (ping/nmap)
Run apt-get update && apt-get install -y <tool>
rockyou.txt not found
Run apt-get install -y wordlists && gunzip /usr/share/wordlists/rockyou.txt.gz


Disclaimer
⚠️ This lab is strictly for educational purposes only.
All attacks demonstrated in this lab are performed inside an isolated Docker environment on a local machine. No real systems, networks, or individuals are targeted.
Performing these attacks on real systems, networks, or applications without explicit written permission is illegal and unethical under cybercrime laws in most countries.
The author takes no responsibility for any misuse of the techniques demonstrated here.

License
This project is for educational use only as part of a university assignment.
 DVWA is maintained by digininja — all credit to the original authors.

Author:
Fatima Yousaf
BSIT Student
Lab completed as part of a cybersecurity university assignment using Docker Desktop, Kali Linux, and DVWA.

