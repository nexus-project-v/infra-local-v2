# 🚀 Nexus Platform — Arquitetura Local com Docker Compose

Este projeto provisiona um ambiente completo de **microserviços orientados a eventos** utilizando Docker Compose, incluindo:

- API Gateway (Kong)
- Microsserviços (Change API / Deploy API)
- Mensageria (Kafka + Zookeeper)
- Banco de Dados (PostgreSQL + PgAdmin)
- Observabilidade (Zipkin)

---

## 🧱 Arquitetura

### 🔄 Fluxo principal

Frontend → Kong → Change API → Kafka → Deploy API

---

## 📦 Serviços

### 🗄️ Banco de Dados

| Serviço | Porta | Descrição |
|--------|------|----------|
| Postgres | 5432 | Banco principal |
| PgAdmin | 16543 | Interface web de administração |

---

### 📡 Mensageria

| Serviço | Porta | Descrição |
|--------|------|----------|
| Zookeeper | 2181 | Coordenação do Kafka |
| Kafka | 9092 | Broker de eventos |
| Kafka UI | 8085 | Interface para visualizar tópicos |

---

### 🔍 Observabilidade

| Serviço | Porta | Descrição |
|--------|------|----------|
| Zipkin | 9411 | Distributed tracing |

---

### ⚙️ Aplicações

| Serviço | Porta | Descrição |
|--------|------|----------|
| Change API | 9933 | Recebe requests e publica no Kafka |
| Deploy API | 9934 | Consome eventos do Kafka |
| Kong Gateway | 8000 / 8001 | API Gateway |

---

## 🔐 API Gateway (Kong)

- Configuração DB-less via `kong.yml`
- Roteamento de requisições
- Possibilidade de:
  - Rate limiting
  - Autenticação (JWT)
  - Logs e métricas

---

## 🧪 Como subir o ambiente

### ▶️ Subir tudo

```bash
docker compose -f docker-compose.infra.yml -f docker-compose.apps.yml up -d
```

### ⛔ Parar tudo

```bash
docker compose -f docker-compose.infra.yml -f docker-compose.apps.yml down
```

---

## 🌐 Acessos

| Serviço | URL |
|--------|-----|
| Kong Proxy | http://localhost:8000 |
| Kong Admin | http://localhost:8001 |
| Change API | http://localhost:9933 |
| Deploy API | http://localhost:9934 |
| PgAdmin | http://localhost:16543 |
| Kafka UI | http://localhost:8085 |
| Zipkin | http://localhost:9411 |

---

## 🧠 Conceitos Arquiteturais

### 🔹 Event-Driven Architecture

- Comunicação assíncrona via Kafka
- Desacoplamento entre serviços
- Alta resiliência e escalabilidade

---

### 🔹 API Gateway Pattern

- Centraliza entrada no sistema
- Controle de tráfego
- Segurança e governança

---

### 🔹 Observabilidade

- Tracing distribuído com Zipkin
- Monitoramento de fluxo entre serviços

---

## ⚙️ Variáveis importantes

### Change API

- DATABASE_URL
- KAFKA_BOOTSTRAP_SERVERS
- ZIPKIN_URL

### Deploy API

- KAFKA_BOOTSTRAP_SERVERS
- ZIPKIN_URL

---

## 🧪 Teste rápido

```bash
curl http://localhost:8000/change
```

👉 Essa chamada passa pelo Kong → Change API → Kafka → Deploy API

---

## 📌 Observações

- Kafka configurado com listeners interno/externo
- Banco inicializado via init.sql
- Rede Docker compartilhada: nexus-network

---

## 🚀 Próximos passos (evolução)

- 🔐 Adicionar autenticação com JWT (Kong)
- 📊 Integrar Prometheus + Grafana
- ♻️ Implementar DLQ no Kafka
- ☁️ Deploy em Kubernetes (EKS / Minikube)

---

## 👨‍💻 Autor

Projeto desenvolvido para estudo de:

- Arquitetura de microserviços
- Event-driven systems
- API Gateway (Kong)
- Observabilidade distribuída
