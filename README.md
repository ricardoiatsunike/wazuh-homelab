# 🛡 Cybersecurity Home Lab --- Wazuh + Metasploitable 3

Laboratório completo e documentado para prática de segurança ofensiva,
detecção, engenharia de detecção, monitoramento SOC e análise de
eventos.

Este repositório consolida **infraestrutura, metodologia ofensiva,
telemetria e detecção**, formando um ambiente realista para simulação do
ciclo completo de ataque e resposta.

------------------------------------------------------------------------

# 📌 Visão Geral

O ambiente foi construído para simular um fluxo real de operação:

Reconhecimento → Enumeração → Ataque → Exploração → Pós-exploração →
Geração de logs → Detecção → Análise → Criação de regra → Resposta

O foco atual do laboratório está na máquina **Linux (Metasploitable 3 -
ub1404)**.\
A VM Windows (win2k8) será integrada futuramente, mas ainda não faz
parte dos cenários ativos.

------------------------------------------------------------------------

# 🎯 Objetivos do Projeto

-   Praticar técnicas de pentest em ambiente isolado
-   Entender geração de logs em sistemas vulneráveis
-   Analisar como um SIEM correlaciona eventos
-   Criar regras customizadas no Wazuh
-   Simular rotina de SOC (Tier 1)
-   Documentar tecnicamente cada fase do ataque
-   Desenvolver visão ofensiva e defensiva integrada

------------------------------------------------------------------------

# 🏗 Arquitetura do Ambiente

Host: Windows 10

-   WSL2 (ambiente ofensivo)
-   VMware Workstation Pro
    -   Wazuh Server (Ubuntu 24.04)
        -   NAT (internet para instalação e updates)
        -   Host-only (rede interna isolada)
    -   Metasploitable 3
        -   ub1404 (Linux vulnerável)

Rede interna isolada: 192.168.X.0/24

------------------------------------------------------------------------

# 🧱 Componentes

## 🔹 Wazuh

-   Indexer
-   Server
-   Dashboard

## 🔹 Máquina Vulnerável

-   Metasploitable 3 (ub1404)

## 🔹 Infraestrutura

-   VMware Workstation Pro 17+
-   Vagrant 2.4+
-   Packer 1.15+
-   Git

## 🔹 Ferramentas Ofensivas

-   Nmap
-   Hydra
-   Metasploit
-   Nikto
-   Gobuster
-   Burp Suite

------------------------------------------------------------------------

# 💻 Requisitos Técnicos

Hardware recomendado: - 16GB RAM - 4+ cores - SSD

------------------------------------------------------------------------

# 🚀 Deploy Completo do Ambiente

## 1️⃣ Configuração de Rede no VMware

Virtual Network Editor:

-   VMnet1 → Host-only → DHCP habilitado
-   VMnet8 → NAT → DHCP habilitado

------------------------------------------------------------------------

## 2️⃣ Instalação do Wazuh (Ubuntu 24.04)

Criar VM com:

-   6GB RAM
-   2 CPUs
-   30GB Disco
-   NAT + Host-only

### Download do instalador

``` bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
curl -sO https://packages.wazuh.com/4.14/config.yml
sudo bash wazuh-install.sh --generate-config-files
```

### Configuração do cluster (single-node)

Editar `config.yml`:

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

### Instalação dos componentes

``` bash
sudo bash wazuh-install.sh --wazuh-indexer node-1
sudo bash wazuh-install.sh --start-cluster
sudo bash wazuh-install.sh --wazuh-server wazuh-1
sudo bash wazuh-install.sh --wazuh-dashboard dashboard
```

### Configuração de Firewall

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

## 3️⃣ Deploy do Metasploitable 3

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

## 4️⃣ Configuração de IP Estático (Linux)

``` bash
sudo ip addr add 192.168.12.130/24 dev eth1
sudo ip link set eth1 up
```

------------------------------------------------------------------------

## 5️⃣ Instalação do Agente Wazuh

``` bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
echo "deb https://packages.wazuh.com/4.14/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt-get update
sudo WAZUH_MANAGER="192.168.12.128" apt-get install wazuh-agent
sudo service wazuh-agent start
```

Verificação:

``` bash
sudo tail -f /var/ossec/logs/ossec.log
```

------------------------------------------------------------------------

# 🔎 Metodologia Operacional

1.  Reconhecimento de superfície
2.  Enumeração de serviços
3.  Ataques de autenticação
4.  Exploração remota
5.  Pós-exploração
6.  Análise de logs
7.  Criação de regras de detecção
8.  Documentação técnica

------------------------------------------------------------------------

# 🛰 Enumeração (Nmap)

``` bash
nmap -sS -sV -p445 --script smb-enum-shares,smb-os-discovery <ip_alvo>
nmap -sV -p21 --script ftp-anon,ftp-vsftpd-backdoor <ip_alvo>
nmap -sV -p22 --script ssh-auth-methods <ip_alvo>
nmap -sV -p80 --script http-enum,http-methods,http-title <ip_alvo>
```

------------------------------------------------------------------------

# 🔐 Ataques de Autenticação (Hydra)

``` bash
hydra -l msfadmin -P rockyou.txt ssh://<ip_alvo>
hydra -l anonymous -P rockyou.txt ftp://<ip_alvo>
hydra -l postgres -P rockyou.txt <ip_alvo> postgres
```

------------------------------------------------------------------------

# 💣 Exploração Remota (Metasploit)

``` bash
search samba
exploit/unix/misc/distcc_exec
exploit/multi/http/tomcat_mgr_upload
auxiliary/scanner/postgres/postgres_login
```

------------------------------------------------------------------------

# 🌐 Descoberta de Vulnerabilidades Web

``` bash
nikto -h http://<ip_alvo>
gobuster dir -u http://<ip_alvo> -w wordlist.txt
```

------------------------------------------------------------------------

# 🧠 Engenharia de Detecção (Wazuh)

Arquivo:

    /var/ossec/etc/rules/local_rules.xml

Exemplo de regra customizada:

``` xml
<group name="custom,bruteforce">
  <rule id="100100" level="10">
    <if_sid>5716</if_sid>
    <description>Possível brute force SSH detectado</description>
  </rule>
</group>
```

Aplicar:

``` bash
sudo systemctl restart wazuh-manager
```

------------------------------------------------------------------------

# 📊 Evolução Planejada (Roadmap)

-   Integração da VM Windows (win2k8)
-   Simulação de movimentação lateral
-   Implementação de alertas baseados em MITRE ATT&CK
-   Dashboards personalizados
-   Simulação de resposta a incidente documentada
-   Criação de playbooks SOC

------------------------------------------------------------------------

# ⚖ Aviso Ético

Todos os testes são realizados exclusivamente em ambiente isolado para
fins educacionais. Nenhum sistema externo é alvo de testes.

------------------------------------------------------------------------