# ADR 0001: Adoção de Banco de Dados Gerenciado (DBaaS) Externo ao Cluster

## Status
Em revisão — aguardando aprovação da Supervisão de Arquitetura e da área de Negócio.

## Data
14/08/2026

## Contexto
A aplicação de pedidos exige persistência dos dados além do ciclo de vida efêmero dos containers.  
Foram avaliadas duas opções:
1. **Self-managed:** PostgreSQL rodando como StatefulSet dentro do cluster Kubernetes.
2. **DBaaS (Managed):** Serviço de banco de dados gerenciado da Magalu Cloud.

A decisão deve atender requisitos de alta disponibilidade e segurança, mantendo o banco isolado da internet pública.

## Decisão
Adotar o **PostgreSQL gerenciado (DBaaS)** da Magalu Cloud, provisionado na zona `br-se1-a`.  
A aplicação se conectará ao banco via rede interna (VPC), utilizando uma **Connection String** (`DATABASE_URL`) injetada com segurança por meio de **Kubernetes Secrets**.

## Revisão dos pontos positivos e negativos da solução adotada

### Positivos
- Reduz Carga Operacional: O provedor (Magalu Cloud) assume a gestão de patches, atualizações de SO e manutenção de hardware.
- Alta Disponibilidade Nativa: O serviço garante snapshots automáticos diários (realizados às 02h UTC com 7 dias de retenção) e failover gerenciado. Isso permite atingir SLA de 99,5% definido para o projeto, reduz o RTO (tempo de recuperação) em caso de falhas.
- Segurança: O banco nasce isolado em uma sub-rede privada, tornando-se invisível para a internet pública.
- Persistência Desacoplada: Os dados sobrevivem independentemente do ciclo de vida efêmero dos pods ou do próprio cluster Kubernetes.

### Negativos
- Custo: Instâncias dedicadas geram custo mensal fixo.  
- Latência: Embora mitigado pelo uso da mesma zona (br-se1-a), o tráfego entre a aplicação e o banco agora depende de chamadas de rede internas, exigindo o gerenciamento de timeouts e retries para evitar falhas em cascata.  
- Rede: O isolamento em sub-rede privada exige uma configuração minuciosa de regras de Security Group para liberar a porta 5432 apenas para o tráfego originado no cluster. Erros nessa configuração resultam em erros de connection refused ou timeout, tornando o diagnóstico inicial mais complexo do que em bancos rodando localmente via Docker Compose.
