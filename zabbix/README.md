---

# Zabbix Monitoring Stack

Este repositório contém uma stack Docker Compose que provisiona uma infraestrutura de monitoramento baseada no Zabbix com suporte a PostgreSQL como banco de dados e Grafana para visualizações avançadas.

## Visão Geral

A stack é composta pelos seguintes serviços:

- **PostgreSQL**: Banco de dados relacional usado pelo Zabbix Server para armazenar dados de monitoramento.
- **Zabbix Server**: Servidor central de monitoramento, responsável por coletar dados dos agentes e processar eventos.
- **Zabbix Web (NGINX)**: Interface web do Zabbix, acessível via navegador para visualização e gerenciamento.
- **Zabbix Agent**: Agente responsável por coletar métricas de servidores e aplicações e enviá-las para o Zabbix Server.
- **Grafana**: Ferramenta de visualização que permite a criação de dashboards customizados usando os dados coletados pelo Zabbix.

## Estrutura dos Serviços

### PostgreSQL Server

- **Imagem**: `postgres:latest`
- **Função**: Banco de dados para o Zabbix Server.
- **Portas**: Não expostas externamente.
- **Volumes**: Persistência de dados em `/var/lib/postgresql/data`.
- **Rede**: `zabbix-network`

### Zabbix Server

- **Imagem**: `zabbix/zabbix-server-pgsql:latest`
- **Função**: Processa os dados de monitoramento e interage com o banco de dados PostgreSQL.
- **Portas**: 
  - `10051:10051` (Zabbix trapper process)
- **Volumes**: Persistência de dados e logs em `/var/lib/zabbix`.
- **Rede**: `zabbix-network`

### Zabbix Web (NGINX)

- **Imagem**: `zabbix/zabbix-web-nginx-pgsql:latest`
- **Função**: Interface web para o Zabbix.
- **Portas**:
  - `8082:8080` (Acesso à interface web do Zabbix)
- **Volumes**: Persistência de arquivos web em `/usr/share/zabbix`.
- **Rede**: `zabbix-network`

### Zabbix Agent

- **Imagem**: `zabbix/zabbix-agent:latest`
- **Função**: Coleta de dados de performance e status dos servidores.
- **Configuração**:
  - Conecta ao Zabbix Server via `zabbix-server` (nome do contêiner na rede Docker).
- **Rede**: `zabbix-network`

### Grafana

- **Imagem**: `grafana/grafana:latest`
- **Função**: Visualização e análise dos dados do Zabbix.
- **Portas**:
  - `3000:3000` (Acesso à interface web do Grafana)
- **Volumes**: Persistência de dados em `/var/lib/grafana`.
- **Plugins**: O plugin Zabbix (`alexanderzobnin-zabbix-app`) é instalado automaticamente.
- **Rede**: `zabbix-network`

## Como Usar

1. **Clone o repositório**:
   ```bash
   git clone https://github.com/seu-repositorio/zabbix-monitoring-stack.git
   cd zabbix-monitoring-stack
   ```

2. **Inicie os contêineres**:
   ```bash
   docker-compose up -d
   ```

3. **Acesse as interfaces**:
   - **Zabbix Web**: `http://localhost:8082` (usuário padrão: `Admin`, senha padrão: `zabbix`)
   - **Grafana**: `http://localhost:3000` (usuário padrão: `admin`, senha padrão: `admin`)

4. **Configurar Zabbix como Fonte de Dados no Grafana**:
   - No Grafana, vá até "Configuration" > "Data Sources" > "Add data source".
   - Escolha "Zabbix" e configure com o URL da API: `http://zabbix-web:8080/zabbix/api_jsonrpc.php`.
   - Preencha com as credenciais de login do Zabbix.

## Persistência de Dados

Os dados dos contêineres são persistidos em volumes Docker, garantindo que as informações não sejam perdidas entre reinicializações:

- **PostgreSQL Data**: `postgresql-data:/var/lib/postgresql/data`
- **Zabbix Server Data**: `zabbix-server-data:/var/lib/zabbix`
- **Zabbix Web Data**: `zabbix-web-data:/usr/share/zabbix`
- **Grafana Data**: `grafana-data:/var/lib/grafana`

## Customizações

Você pode ajustar as configurações dos serviços alterando as variáveis de ambiente no arquivo `docker-compose.yml` conforme necessário. 

## Encerramento

Para parar e remover todos os contêineres e volumes associados, execute:

```bash
docker-compose down -v
```

---
