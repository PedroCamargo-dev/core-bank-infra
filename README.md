# Core Bank - Infraestrutura de Observabilidade

Infraestrutura completa de observabilidade para o sistema Core Bank, implementando a stack de logs, métricas e traces distribuídos.

## 📋 Visão Geral

Este repositório contém toda a infraestrutura necessária para executar um ambiente de observabilidade completo, incluindo:

- **Banco de Dados**: PostgreSQL 16
- **Message Broker**: RabbitMQ 3 com Management UI
- **API Gateway**: Kong com Admin UI
- **Monitoramento**: Prometheus + Grafana
- **Logs**: Loki + Promtail
- **Traces**: Tempo + OpenTelemetry Collector
- **Exporters**: Postgres Exporter, RabbitMQ Exporter

## 🚀 Tecnologias

### Componentes Principais

| Componente | Versão | Descrição |
|------------|--------|-----------|
| PostgreSQL | 16 | Banco de dados relacional |
| RabbitMQ | 3-management | Message broker com interface de gerenciamento |
| Kong | latest | API Gateway e gerenciador de APIs |
| Prometheus | 2.53.2 | Sistema de monitoramento e time-series database |
| Grafana | 11.2.0 | Plataforma de analytics e visualização |
| Loki | 3.1.1 | Sistema de agregação de logs |
| Promtail | 3.1.1 | Agente de coleta de logs |
| Tempo | 2.6.1 | Sistema de distributed tracing |
| OpenTelemetry Collector | 0.110.0 | Coletor de telemetria (traces e métricas) |

### Exporters

- **Postgres Exporter** (0.15.0): Exporta métricas do PostgreSQL para o Prometheus
- **RabbitMQ Exporter**: Exporta métricas do RabbitMQ para o Prometheus

## 📦 Pré-requisitos

- Docker Engine
- Docker Compose
- Rede Docker `cloudflared` (externa)

### Criar a Rede Externa

```bash
docker network create cloudflared
```

## 🔧 Configuração

### Portas Expostas

| Serviço | Porta(s) | Descrição |
|---------|----------|-----------|
| PostgreSQL | 5432 | Conexão do banco de dados |
| RabbitMQ | 5672, 15672 | AMQP e Management UI |
| Kong Gateway | 8800, 8443 | HTTP e HTTPS |
| Kong Admin API | 8001, 8444 | Admin API HTTP e HTTPS |
| Kong Admin UI | 8002, 8445 | Admin UI HTTP e HTTPS |
| Prometheus | 9090 | Interface web e API |
| Grafana | 3000 | Interface web |
| Loki | 3100 | API de ingestão e consulta de logs |
| Tempo | 3200, 14318 | API e OTLP HTTP |
| OpenTelemetry Collector | 4317, 4318 | OTLP gRPC e HTTP |
| Postgres Exporter | 9187 | Métricas do PostgreSQL |
| RabbitMQ Exporter | 9419 | Métricas do RabbitMQ |

### Credenciais Padrão

#### PostgreSQL
- **Usuário**: postgres
- **Senha**: securepassword
- **Database**: postgres

#### RabbitMQ
- **Usuário**: rabbitmq
- **Senha**: securepassword
- **Management UI**: http://localhost:15672

#### Kong
- **Senha Admin**: handyshake
- **Admin UI**: http://localhost:8002

#### Grafana
- **Usuário**: admin
- **Senha**: adminpassword
- **URL**: http://localhost:3000

> ⚠️ **IMPORTANTE**: Estas são credenciais de desenvolvimento. **NUNCA** use essas senhas em produção!

## 🏃 Como Executar

### Iniciar todos os serviços

```bash
docker compose up -d
```

### Verificar status dos serviços

```bash
docker compose ps
```

### Ver logs de um serviço específico

```bash
docker compose logs -f <nome-do-serviço>
```

### Parar todos os serviços

```bash
docker compose down
```

### Parar e remover volumes (dados)

```bash
docker compose down -v
```

## 📊 Observabilidade

### Prometheus

Prometheus está configurado para coletar métricas dos seguintes targets:

- Prometheus (auto-monitoramento)
- PostgreSQL (via postgres-exporter)
- RabbitMQ (via rabbitmq-exporter)
- Grafana
- Loki
- Tempo
- OpenTelemetry Collector
- Kong

**Intervalo de scrape**: 10 segundos

Acesse: http://localhost:9090

### Grafana

#### Datasources
- **Prometheus**: Métricas (datasource padrão)
- **Loki**: Logs agregados
- **Tempo**: Distributed tracing

#### Dashboards
- **Infrastructure Health**: Monitoramento de saúde da infraestrutura
- **Logs & Errors**: Análise de logs e erros

#### Correlação de Dados
- Logs com Traces: Links automáticos de trace_id nos logs do Loki para traces no Tempo
- Traces com Logs: Navegação de traces para logs correlacionados

Acesse: http://localhost:3300

### Loki

Sistema de agregação de logs com:

- **Retenção**: 168 horas (7 dias)
- **Storage**: Filesystem local
- **Compactação**: Automática a cada 10 minutos

### Promtail

Agente de coleta configurado para:
- Coletar logs de todos os containers Docker
- Adicionar labels automáticos: container, image, stream, service
- Enviar para Loki em tempo real

### Tempo

Sistema de distributed tracing com:

- **Protocolo**: OTLP (gRPC e HTTP)
- **Retenção**: 168 horas (7 dias)
- **Storage**: Filesystem local

### OpenTelemetry Collector

Coletor centralizado de telemetria:

#### Receivers
- OTLP gRPC (porta 4317)
- OTLP HTTP (porta 4318)

#### Processors
- Batch processing (timeout: 2s, batch size: 1024)

#### Exporters
- **Traces**: Exporta para Tempo via OTLP
- **Metrics**: Exporta para Prometheus
- **Debug**: Logs detalhados para debugging

## 🗄️ Volumes Persistentes

Os seguintes volumes são criados para persistência de dados:

- `pgdata`: Dados do PostgreSQL
- `rabbitmq_data`: Dados do RabbitMQ
- `loki_data`: Chunks e índices do Loki
- `tempo_data`: Traces do Tempo

## 🔄 Health Checks

Todos os serviços críticos possuem health checks configurados:

- **PostgreSQL**: Verificação via `pg_isready`
- **RabbitMQ**: Verificação via `rabbitmq-diagnostics`
- **Kong**: Verificação via `kong health`

**Configuração**:
- Intervalo: 10 segundos
- Timeout: 5 segundos
- Retries: 10
- Start period: 20 segundos

## 🌐 Network

Todos os serviços estão conectados à rede externa `cloudflared`, permitindo comunicação entre containers e integração com outros serviços.

## 📈 Monitoramento de Métricas

### Postgres Exporter

Coleta métricas detalhadas do PostgreSQL:
- Conexões ativas e máximas
- Taxa de transações
- Cache hits/misses
- Estatísticas de tabelas e índices
- Bloqueios e deadlocks

### RabbitMQ Exporter

Coleta métricas do RabbitMQ:
- Mensagens publicadas/consumidas
- Filas e suas estatísticas
- Conexões e canais
- Memória e recursos

## 🔍 Como Usar

### Enviar Traces

Envie traces via OpenTelemetry Collector:

```bash
# Via gRPC
curl -X POST http://localhost:4317

# Via HTTP
curl -X POST http://localhost:4318/v1/traces
```

### Consultar Logs

```bash
# Via Loki API
curl -G -s "http://localhost:3100/loki/api/v1/query" \
  --data-urlencode 'query={service="seu-servico"}'
```

### Consultar Métricas

```bash
# Via Prometheus API
curl -G http://localhost:9090/api/v1/query \
  --data-urlencode 'query=up'
```

## 🔐 Segurança

- Configure variáveis de ambiente para sobrescrever senhas padrão
- Use secrets management em produção
- Configure TLS/SSL para comunicação entre serviços
- Revise as políticas de rede e firewall

## 📝 Configuração Adicional

### Variáveis de Ambiente

Você pode configurar o hostname do Kong Admin GUI:

```bash
export GW_HOST=seu-hostname
docker compose up -d
```

### Arquivos de Configuração

- [prometheus.yml](prometheus.yml): Configuração de scrape do Prometheus
- [loki-config.yaml](loki-config.yaml): Configuração do Loki (storage, retenção)
- [promtail-config.yaml](promtail-config.yaml): Configuração de coleta do Promtail
- [tempo-config.yaml](tempo-config.yaml): Configuração do Tempo (receivers, storage)
- [otel-collector-config.yaml](otel-collector-config.yaml): Pipeline do OpenTelemetry
- [grafana/provisioning/](grafana/provisioning/): Datasources e dashboards

## 🛠️ Troubleshooting

### Verificar logs de um serviço

```bash
docker compose logs -f <serviço>
```

### Reiniciar um serviço específico

```bash
docker compose restart <serviço>
```

### Verificar conectividade de rede

```bash
docker network inspect cloudflared
```

### Kong não inicializa

Verifique se o PostgreSQL está saudável e se as migrations foram executadas:

```bash
docker compose logs kong-bootstrap
docker compose logs postgres
```

---

**Desenvolvido por**: Pedro Camargo  
**Repositório**: core-bank-infra  
**Data**: Janeiro 2026
