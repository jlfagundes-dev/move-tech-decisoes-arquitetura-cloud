# ADR 0002: Estratégia de Exposição do Serviço

- **Status:** Aprovado
- **Data:** 2026-08-14

## Contexto
O sistema precisa de um ponto de acesso externo estável e confiável para usuários e ferramentas de automação que precisam interagir com o serviço. Como os endereços IP dos Pods no Kubernetes são efêmeros e mudam constantemente, é obrigatório utilizar um recurso (service) que forneça uma entrada estática para o serviço.
Foram avaliadas duas abordagens:
- Load Balancer: Exposição direta via balanceamento de carga.
- Ingress Controller: Gerenciamento centralizado de rotas e múltiplos domínios, utilizando soluções como Traefik ou Nginx.

## Decisão
Decidimos adotar o serviço de Load Balancer da Magalu Cloud para expor a aplicação diretamente aos usuários e ferramentas de automação que precisam interagir com o serviço. Encaminhando o tráfego para os Pods.

Esta decisão reduz o esforço de configuração de infraestrutura intermediária no estágio atual do projeto.


## Consequências

- **Positivas:**
    - Agilidade na Entrega: Elimina etapas complexas de configuração de software intermediário, permitindo que a aplicação entre no ar em menos tempo.
    - Integração Nativa: A solução de Load Balancer na nuvem (Magalu Cloud) gerencia o provisionamento automático de um IP público vinculado ao serviço, garantindo que a aplicação esteja disponível imediatamente após o deploy.
    - Solução Única: Essa abordagem minimiza a superfície de configuração e reduz possíveis pontos de falha na camada de roteamento.

- **Negativas:**
    - Custo: Caso a solução evolua para múltiplos serviços/microsserviços expostos publicamente. Instâncias dedicadas geram custo mensal fixo para cada serviço de load balancer.
    - Complexidade na Gestão de Segurança: A gestão de certificados de segurança (SSL/TLS) e regras de proteção contra ataques precisa ser feita de forma manual e individual para cada serviço. Isso aumenta a chance de erros humanos.
    - Limitação de Roteamento por Caminho: Diferente do Ingress, o Load Balancer não suporta separar tráfego por URLs específicas.
        - Exemplo: O endpoint `/api/v1/products` usado pelos clientes para consultar catálogo e carrinho, enquanto `/admin/orders` é usado pelo time interno para gerenciar pedidos. Com Load Balancer direto, todo o tráfego chega no mesmo IP público e a aplicação precisa diferenciar internamente as rotas. Se o negócio decidir expor publicamente um novo serviço, como `/analytics`, pode ser necessário provisionar outro IP público e Load Balancer, aumentando custo e complexidade.

