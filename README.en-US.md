# 🛡 Cybersecurity Home Lab --- Wazuh + Metasploitable 3

A fully documented laboratory environment for offensive security
practice, detection engineering, SOC monitoring, and event analysis.

This repository consolidates **infrastructure, offensive methodology,
telemetry, and detection engineering**, creating a realistic environment
to simulate the full attack and response lifecycle.

------------------------------------------------------------------------

# 📌 Overview

This environment was designed to simulate a real-world operational flow:

Reconnaissance → Enumeration → Attack → Exploitation → Post-Exploitation
→ Log Generation → Detection → Analysis → Rule Creation → Response

The current focus of the lab is the **Linux machine (Metasploitable 3 -
ub1404)**. The Windows VM (win2k8) will be integrated in the future but
is not yet part of the active scenarios.

------------------------------------------------------------------------

# 🎯 Project Objectives

-   Practice penetration testing techniques in an isolated environment
-   Understand log generation in vulnerable systems
-   Analyze how a SIEM correlates events
-   Create custom detection rules in Wazuh
-   Simulate SOC (Tier 1) workflows
-   Technically document each phase of the attack lifecycle
-   Develop an integrated offensive and defensive mindset

------------------------------------------------------------------------

# 🏗 Environment Architecture

Host: Windows 10

-   WSL2 (offensive environment)
-   VMware Workstation Pro
    -   Wazuh Server (Ubuntu 24.04)
        -   NAT (internet access for installation and updates)
        -   Host-only (isolated internal network)
    -   Metasploitable 3
        -   ub1404 (intentionally vulnerable Linux)

Isolated internal network: 192.168.X.0/24

------------------------------------------------------------------------

# 🧱 Components

## 🔹 Wazuh

-   Indexer
-   Server
-   Dashboard

## 🔹 Vulnerable Machine

-   Metasploitable 3 (ub1404)

## 🔹 Infrastructure

-   VMware Workstation Pro 17+
-   Vagrant 2.4+
-   Packer 1.15+
-   Git

## 🔹 Offensive Tools

-   Nmap
-   Hydra
-   Metasploit
-   Nikto
-   Gobuster
-   Burp Suite

------------------------------------------------------------------------

# 💻 Technical Requirements

Recommended hardware:

-   16GB RAM
-   4+ CPU cores
-   SSD

------------------------------------------------------------------------

# 🚀 Full Environment Deployment

## 1️⃣ VMware Network Configuration

Virtual Network Editor:

-   VMnet1 → Host-only → DHCP enabled
-   VMnet8 → NAT → DHCP enabled

------------------------------------------------------------------------

## 2️⃣ Wazuh Installation (Ubuntu 24.04)

Create VM with:

-   6GB RAM
-   2 CPUs
-   30GB Disk
-   NAT + Host-only

### Download installer

``` bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
curl -sO https://packages.wazuh.com/4.14/config.yml
sudo bash wazuh-install.sh --generate-config-files
```

### Single-node cluster configuration

Edit `config.yml`:

``` yaml
nodes:
  indexer:
    - name: node-1
      ip: "192.168.12.128"
  server:
    - name: wazuh-1
      ip: "192.168.12.128"
  dashboard:
    - name: dashboard
      ip: "192.168.12.128"
```

### Install components

``` bash
sudo bash wazuh-install.sh --wazuh-indexer node-1
sudo bash wazuh-install.sh --start-cluster
sudo bash wazuh-install.sh --wazuh-server wazuh-1
sudo bash wazuh-install.sh --wazuh-dashboard dashboard
```

### Firewall configuration

``` bash
sudo ufw allow 1514/tcp
sudo ufw allow 1514/udp
sudo ufw allow 1515/tcp
sudo ufw allow 55000/tcp
sudo ufw allow 9200/tcp
sudo ufw allow 9300/tcp
sudo ufw allow 443/tcp
sudo ufw allow 5601/tcp
sudo ufw --force enable
```

------------------------------------------------------------------------

## 3️⃣ Metasploitable 3 Deployment

``` powershell
vagrant plugin install vagrant-vmware-desktop
```

``` powershell
mkdir C:\Metasploitable3-Workspace
cd C:\Metasploitable3-Workspace
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/rapid7/metasploitable3/master/Vagrantfile" -OutFile "Vagrantfile"
vagrant up --provider=vmware_desktop
```

------------------------------------------------------------------------

## 4️⃣ Static IP Configuration (Linux)

``` bash
sudo ip addr add 192.168.12.130/24 dev eth1
sudo ip link set eth1 up
```

------------------------------------------------------------------------

## 5️⃣ Wazuh Agent Installation

``` bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
echo "deb https://packages.wazuh.com/4.14/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt-get update
sudo WAZUH_MANAGER="192.168.12.128" apt-get install wazuh-agent
sudo service wazuh-agent start
```

Verification:

``` bash
sudo tail -f /var/ossec/logs/ossec.log
```

------------------------------------------------------------------------

# 🔎 Operational Methodology

1.  Surface reconnaissance
2.  Service enumeration
3.  Authentication attacks
4.  Remote exploitation
5.  Post-exploitation
6.  Log analysis
7.  Detection rule creation
8.  Technical documentation

------------------------------------------------------------------------

# 🛰 Enumeration (Nmap)

``` bash
nmap -sS -sV -p445 --script smb-enum-shares,smb-os-discovery <target_ip>
nmap -sV -p21 --script ftp-anon,ftp-vsftpd-backdoor <target_ip>
nmap -sV -p22 --script ssh-auth-methods <target_ip>
nmap -sV -p80 --script http-enum,http-methods,http-title <target_ip>
```

------------------------------------------------------------------------

# 🔐 Authentication Attacks (Hydra)

``` bash
hydra -l msfadmin -P rockyou.txt ssh://<target_ip>
hydra -l anonymous -P rockyou.txt ftp://<target_ip>
hydra -l postgres -P rockyou.txt <target_ip> postgres
```

------------------------------------------------------------------------

# 💣 Remote Exploitation (Metasploit)

``` bash
search samba
exploit/unix/misc/distcc_exec
exploit/multi/http/tomcat_mgr_upload
auxiliary/scanner/postgres/postgres_login
```

------------------------------------------------------------------------

# 🌐 Web Vulnerability Discovery

``` bash
nikto -h http://<target_ip>
gobuster dir -u http://<target_ip> -w wordlist.txt
```

------------------------------------------------------------------------

# 🧠 Detection Engineering (Wazuh)

File:

    /var/ossec/etc/rules/local_rules.xml

Custom rule example:

``` xml
<group name="custom,bruteforce">
  <rule id="100100" level="10">
    <if_sid>5716</if_sid>
    <description>Possible SSH brute force detected</description>
  </rule>
</group>
```

Apply:

``` bash
sudo systemctl restart wazuh-manager
```

------------------------------------------------------------------------

# 📊 Planned Evolution (Roadmap)

-   Windows VM (win2k8) integration
-   Lateral movement simulation
-   MITRE ATT&CK-based alerts
-   Custom dashboards
-   Documented incident response simulation
-   SOC playbook creation

------------------------------------------------------------------------

# ⚖ Ethical Notice

All tests are conducted exclusively in an isolated laboratory
environment for educational purposes. No external systems are targeted.

------------------------------------------------------------------------

Document generated on 2026-02-24 18:36:09 UTC
