# infra-monitoring

Stack de observabilidade da infraestrutura DEV2B.

## O que está incluído

| Serviço        | Função                                          | Acesso              |
|----------------|-------------------------------------------------|---------------------|
| Grafana        | Visualização de métricas e logs (dashboards)    | https://grafana.dev2b.tec.br |
| Prometheus     | Coleta e armazenamento de métricas              | interno             |
| cAdvisor       | Métricas de containers (CPU, RAM, rede, I/O)    | interno             |
| node-exporter  | Métricas do host (disco, CPU, memória, rede)    | interno             |
| Loki           | Armazenamento e consulta de logs                | interno             |

## Arquitetura

```
Internet
   │
Traefik (proxy)
   │  https://grafana.dev2b.tec.br
   ▼
Grafana
   ├── Prometheus ◄── cAdvisor (métricas de containers)
   │             ◄── node-exporter (métricas do host)
   └── Loki      ◄── logs via loki-docker-driver (opcional)
```

## Pré-requisitos

### Rede Docker externa
```bash
docker network create proxy
```

### DNS
```
grafana.dev2b.tec.br → 129.121.51.116
```

## Variáveis de ambiente

Crie o secret `APP_ENV` no GitHub:

```env
GRAFANA_USER=admin
GRAFANA_PASSWORD=senha_segura_aqui
```

| Variável            | Descrição                         |
|---------------------|-----------------------------------|
| `GRAFANA_USER`      | Usuário administrador do Grafana  |
| `GRAFANA_PASSWORD`  | Senha do administrador            |

## Deploy

Automático via GitHub Actions a cada push na `main`.

### Secrets necessários no GitHub

| Secret            | Valor                        |
|-------------------|------------------------------|
| `VPS_HOST`        | `129.121.51.116`             |
| `VPS_USER`        | `root`                       |
| `VPS_PASSWORD`    | Senha SSH                    |
| `VPS_PORT`        | `22022`                      |
| `APP_ENV`         | Conteúdo do `.env.app`       |

## Dashboards recomendados

Após o primeiro acesso ao Grafana, importe os dashboards prontos do Grafana Labs:

| Dashboard                              | ID    |
|----------------------------------------|-------|
| Docker + cAdvisor                      | 13946 |
| Docker and system monitoring           | 13496 |
| cAdvisor Docker Insights               | 19908 |
| Docker and Host Monitoring (Prometheus)| 179   |

Para importar: **Dashboards → New → Import → ID**

## Plugin loki-docker-driver (opcional)

Para enviar logs de todos os containers ao Loki automaticamente, instale o plugin na VPS:

```bash
docker plugin install grafana/loki-docker-driver:latest --alias loki --grant-all-permissions
```

Adicione ao `/etc/docker/daemon.json`:

```json
{
  "log-driver": "loki",
  "log-opts": {
    "loki-url": "http://localhost:3100/loki/api/v1/push",
    "loki-batch-size": "400"
  }
}
```

```bash
systemctl restart docker
```

> Novos containers criados após isso enviarão logs ao Loki automaticamente.

## Deploy manual

```bash
cd /opt/monitoring

cat > .env.app <<EOF
GRAFANA_USER=admin
GRAFANA_PASSWORD=senha_aqui
EOF
chmod 600 .env.app

docker compose pull
docker compose up -d
docker compose ps
```
