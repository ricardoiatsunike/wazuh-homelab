## Homelab de Cibersegurança com Wazuh e Metasploitable 3

Documentação do desenvolvimento e uso de um ambiente controlado voltado ao estudo prático ofensivo e monitoramento defensivo.

O ambiente integra:

- Wazuh como plataforma SIEM
- VM Metasploitable 3 como alvo vulnerável
- Ferramentas ofensivas executadas via WSL2
- VMware Workstation Pro
  
---

## Objetivos do Projeto

- Praticar técnicas de pentest em ambiente isolado
- Entender a geração de logs em diferentes serviços
- Analisar como um SIEM correlaciona eventos
- Criar regras customizadas de detecção
- Simular perspectiva de SOC
- Elaborar documentação técnica

---

<details>
<summary><h2><b>Clique aqui para visualizar a arquitetura do projeto</b></h2></summary>

• Host: Windows  
• WSL2 - ambiente ofensivo
• VMware Workstation Pro
• Wazuh Server (Ubuntu 24.04)  
• NAT
• Host-only - rede interna isolada  
• Metasploitable 3 (ub1404) – alvo

**Rede interna isolada:** `192.168.12.0/24`

---

## Componentes

### Wazuh
- Indexer
- Manager
- Dashboard

### Máquina Vulnerável
- Metasploitable 3 (ub1404)

### Infraestrutura
- VMware Workstation Pro 17+
- Vagrant 2.4+
- Packer 1.15+
- Git

### Ferramentas Ofensivas
- Nmap
- Hydra
- Metasploit
- Nikto
- Gobuster
- Burp Suite

---

## Requisitos Técnicos

**Hardware recomendado:**  
- 16 GB RAM  
- 4+ núcleos de CPU  
- SSD
</details>
---

## Deploy do Ambiente

### Configuração de Rede no VMware

No **Virtual Network Editor**:

- **VMnet1** → Host-only → DHCP habilitado  
- **VMnet8** → NAT → DHCP habilitado

---

### Instalação do Wazuh (Ubuntu 24.04)

Criar VM com:

- 6 GB RAM
- 2 CPUs
- 30 GB de disco
- Adaptador 1: NAT
- Adaptador 2: Host-only (VMnet1)

#### Download do instalador

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
curl -sO https://packages.wazuh.com/4.14/config.yml
sudo bash wazuh-install.sh --generate-config-files
```

#### Configuração do cluster (single-node)

Editar `config.yml`:

```yaml
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

#### Instalação dos componentes

```bash
sudo bash wazuh-install.sh --wazuh-indexer node-1
sudo bash wazuh-install.sh --start-cluster
sudo bash wazuh-install.sh --wazuh-server wazuh-1
sudo bash wazuh-install.sh --wazuh-dashboard dashboard
```

#### Configuração de Firewall

```bash
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

---

### Deploy do Metasploitable 3

```powershell
vagrant plugin install vagrant-vmware-desktop
```

```powershell
mkdir C:\Metasploitable3-Workspace
cd C:\Metasploitable3-Workspace
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/rapid7/metasploitable3/master/Vagrantfile" -OutFile "Vagrantfile"
vagrant up --provider=vmware_desktop
```

---

### Configuração de IP Estático (Linux)

```bash
sudo ip addr add 192.168.12.100/24 dev eth1
sudo ip link set eth1 up
```

---

### Instalação do Agente Wazuh

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
echo "deb https://packages.wazuh.com/4.14/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt-get update
sudo WAZUH_MANAGER="192.168.12.128" apt-get install wazuh-agent
sudo service wazuh-agent start
```

Verificação:

```bash
sudo tail -f /var/ossec/logs/ossec.log
```

---

## Metodologia

1. Reconhecimento de superfície  
2. Enumeração de serviços  
3. Ataques de autenticação  
4. Exploração remota  
5. Pós-exploração  
6. Análise de logs  
7. Criação de regras de detecção  
8. Documentação técnica

---

## Enumeração com Nmap

```bash
nmap -sS -sV -p445 --script smb-enum-shares,smb-os-discovery 192.168.12.100
nmap -sV -p21 --script ftp-anon,ftp-vsftpd-backdoor 192.168.12.100
nmap -sV -p22 --script ssh-auth-methods 192.168.12.100
nmap -sV -p80 --script http-enum,http-methods,http-title 192.168.12.100
```

---

## Ataques de Autenticação com Hydra

<img width="844" height="251" alt="image" src="https://github.com/user-attachments/assets/e6c5f533-13ac-4b23-8066-f94dbfafbc82" />

```bash
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt ssh://192.168.12.100
hydra -l anonymous -P /usr/share/wordlists/rockyou.txt ftp://192.168.12.100
hydra -l postgres -P /usr/share/wordlists/rockyou.txt 192.168.12.100 postgres
```

## Perspectiva defensiva no servidor:
<img width="810" height="154" alt="image" src="https://github.com/user-attachments/assets/8e942d35-f897-408d-833f-ca97fb3342f5" />

---

## Exploração Remota com Metasploit

```bash
msfconsole
search samba
use exploit/unix/misc/distcc_exec
use exploit/multi/http/tomcat_mgr_upload
use auxiliary/scanner/postgres/postgres_login
```

<img width="313" height="151" alt="image" src="https://github.com/user-attachments/assets/d61e0c0b-bb79-4a39-af7c-d64938440aef" />


---

## Descoberta de Vulnerabilidades Web

```bash
nikto -h http://192.168.12.100
gobuster dir -u http://192.168.12.100 -w /usr/share/wordlists/dirb/common.txt
```

---

## Detecção com Wazuh

Arquivo de regras customizadas:

```
/var/ossec/etc/rules/local_rules.xml
```

Exemplo de regra para força bruta SSH:

```xml
<group name="custom,bruteforce">
  <rule id="100100" level="10">
    <if_sid>5716</if_sid>
    <description>Possível brute force SSH detectado</description>
  </rule>
</group>
```

Aplicar regra:

```bash
sudo systemctl restart wazuh-manager
```

---

## Roadmap

- Integração da VM Windows (win2k8)
- Implementação de alertas baseados em MITRE ATT&CK
- Dashboards personalizados no Wazuh
- Simulação de resposta a incidente
- Criação de playbooks SOC

---

## Referências

- [🔗 Documentação Oficial Wazuh](https://documentation.wazuh.com)
- [🔗 Metasploitable 3 – Rapid7](https://github.com/rapid7/metasploitable3/)
- [🔗 Nmap Documentation](https://nmap.org/docs.html)
- [🔗 Hydra – Kali Tools](https://www.kali.org/tools/hydra/)
- [🔗 Metasploit Documentation](https://docs.metasploit.com/)
