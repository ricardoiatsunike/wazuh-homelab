# Wazuh Home Lab

Documentação técnica para implantação de um **Wazuh Home Lab** em ambiente virtualizado, voltado para **monitoramento**, **SIEM**, **HIDS** e **treinamento prático de SOC**.

---

## Objetivo

Construir um laboratório funcional do Wazuh para estudo prático de:

- Monitoramento de endpoints
- Coleta e análise de logs
- Detecção de incidentes
- Rotina operacional de SOC (Tier 1 / Tier 2)

---

## Ambiente

- Hypervisor: VirtualBox / VMware
- Sistema Operacional: Ubuntu Server LTS
- Wazuh: versão 4.14
- Endpoints monitorados: Windows

---

## Pós-instalação do Ubuntu Server

### Expansão de disco (LVM)

Por padrão, o Ubuntu Server não utiliza todo o espaço disponível do disco virtual.

```bash
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
df -h
```

---

### Configuração de fuso horário (Brasília)

```bash
sudo timedatectl set-timezone America/Sao_Paulo
timedatectl
sudo systemctl restart systemd-timesyncd
```

---

## Configuração de Firewall (UFW)

```bash
sudo ufw --force enable
```

### Portas – Wazuh Indexer
```bash
sudo ufw allow 9200/tcp
sudo ufw allow 9300/tcp
```

### Portas – Wazuh Manager
```bash
sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp
sudo ufw allow 1516/tcp
sudo ufw allow 55000/tcp
```

### Portas – Wazuh Dashboard
```bash
sudo ufw allow 443/tcp
sudo ufw allow 5601/tcp
```

Verificação:
```bash
sudo ufw status
```

---

## Instalação do Wazuh (Modelo Distribuído)

### Download do instalador e configuração inicial

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
curl -sO https://packages.wazuh.com/4.14/config.yml
```

---

## Configuração do `config.yml`

Edite o arquivo `config.yml` e ajuste os nomes e endereços IP conforme o ambiente:

```yaml
nodes:
  indexer:
    - name: node-1
      ip: "<indexer-node-ip>"

  server:
    - name: wazuh-1
      ip: "<wazuh-manager-ip>"

  dashboard:
    - name: dashboard
      ip: "<dashboard-node-ip>"
```

---

## Geração de certificados e chaves

```bash
bash wazuh-install.sh --generate-config-files
```

O comando gera o arquivo:

```text
wazuh-install-files.tar
```

Copie este arquivo para todos os nós do ambiente.

---

## Instalação do Wazuh Indexer

```bash
bash wazuh-install.sh --wazuh-indexer node-1
```

---

## Inicialização do cluster

```bash
bash wazuh-install.sh --start-cluster
```

---

## Instalação do Wazuh Manager

```bash
bash wazuh-install.sh --wazuh-server wazuh-1
```

---

## Instalação do Wazuh Dashboard

```bash
bash wazuh-install.sh --wazuh-dashboard dashboard
```

---

## Deploy do Wazuh Agent (Windows)

### Instalação via linha de comando

CMD:
```cmd
wazuh-agent-4.14.2-1.msi /q WAZUH_MANAGER="10.0.0.2"
```

PowerShell:
```powershell
.\wazuh-agent-4.14.2-1.msi /q WAZUH_MANAGER="10.0.0.2"
```

### Inicialização do serviço

CMD:
```cmd
NET START WazuhSvc
```

PowerShell:
```powershell
Start-Service wazuhsvc
```

---

## Resultado esperado

- Agentes conectados ao Manager
- Logs indexados no Wazuh Indexer
- Dashboard operacional
- Ambiente pronto para testes e simulações de incidentes

---

## Referências

- https://documentation.wazuh.com
