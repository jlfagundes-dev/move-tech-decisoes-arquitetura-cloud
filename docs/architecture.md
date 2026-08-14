**Aluno:** José Luiz Fagundes
**Repositório do Projeto:** https://github.com/jlfagundes-dev/move-tech-decisoes-arquitetura-cloud

# Documentação de Arquitetura da Solução

## 1. Mapeamento de Recursos (Cluster & Cloud)
- **Cluster Kubernetes (Magalu Cloud):**
	- App API (Container em FastAPI)
	- Service `LoadBalancer` (expõe porta pública)
	- HorizontalPodAutoscaler (HPA)
	- ServiceMonitor (Prometheus)
- **Serviços Externos / Managed Services:**
	- PostgreSQL Gerenciado (DBaaS)
	- Magalu Cloud Container Registry (MCR)
	- Prometheus + Grafana (observability stack)

## 2. Diagrama C2 (Nível de Containers)

```mermaid
graph TD
    User([Usuário / Cliente HTTP]) -->|HTTPS/HTTP :80| LB[Service cloud-application<br/>LoadBalancer - MGC Cloud]
    LB -->|HTTP/TCP :8000| App1[Pod: API de Pedidos<br/>FastAPI/Uvicorn - réplica 1]
    LB -->|HTTP/TCP :8000| App2[Pod: API de Pedidos<br/>FastAPI/Uvicorn - réplica N até 6]

    App1 -->|TLS/TCP :5432 - psycopg2| DB[(PostgreSQL Gerenciado<br/>DBaaS - fora do cluster)]
    App2 -->|TLS/TCP :5432 - psycopg2| DB

    Prometheus[Prometheus Server<br/>Prometheus Operator] -->|HTTP scrape /metrics :8000<br/>a cada 15s| App1
    Prometheus -->|HTTP scrape /metrics :8000<br/>a cada 15s| App2
    Grafana[Grafana] -->|HTTP/PromQL :9090| Prometheus

    HPA[HorizontalPodAutoscaler] -.->|API K8s - leitura de métricas de CPU| App1
    HPA -.->|API K8s - leitura de métricas de CPU| App2

    CI[GitHub Actions - CI/CD] -->|HTTPS - docker push| Registry[Magalu Container Registry]
    CI -->|HTTPS - kubectl apply| K8sAPI[API do Kubernetes]
    K8sAPI -->|HTTPS - image pull| Registry
    Registry -.->|imagePullSecrets| App1
    Registry -.->|imagePullSecrets| App2

    K6[k6 - Teste de carga<br/>GitHub Actions manual] -->|HTTP/HTTPS :80| LB
```

Estilo Arquitetural e RNFs
- **Estilo:** Monolito Modular em Camadas com Implantação Cloud-Native em Contêineres.
- **Disponibilidade Alvo:** 99.9% (SLA)
- **Latência P95:** < 200ms sob carga normal.
- **RPS Alvo:** 500 requisições por segundo.
- **Teto de Custo (FinOps):** R$ 150,00/mês (alocação otimizada de recursos).
---
