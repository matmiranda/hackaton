# FastTech Foods – MVP Solution

## Índice

- [Desenho da Solução MVP](#desenho-da-solução-mvp)  
- [Diagrama da Arquitetura da Solução (DDD)](#diagrama-da-arquitetura-da-solução-ddd)  
- [Justificativa Técnica das Decisões Arquiteturais](#justificativa-técnica-das-decisoes-arquiteturais)  
- [Próximos Passos e Sugestões de Evolução](#próximos-passos-e-sugestões-de-evolução)  

---

## Desenho da Solução MVP

O MVP é composto por quatro microsserviços independentes, containerizados e orquestrados via Kubernetes, comunicando-se assíncronamente por mensageria. Componentes de suporte garantem segurança, observabilidade e entrega contínua.

- **Microsserviço de Autenticação**  
  - Login de funcionários (e-mail corporativo) e clientes (CPF ou e-mail)  
  - Emissão e validação de JWT  
  - Persistência em banco SQL dedicado  

- **Microsserviço de Cardápio**  
  - CRUD de itens: nome, descrição, preço, disponibilidade  
  - API REST para front-end e outros serviços  
  - Banco SQL próprio  

- **Microsserviço de Pedidos**  
  - Criação, atualização e cancelamento de pedidos  
  - Estados: pendente, em preparo, pronto  
  - Publicação de eventos “PedidoCriado” e “PedidoCancelado”  

- **Microsserviço de Cozinha**  
  - Consome eventos de pedidos  
  - Fluxo de aceite ou rejeição  
  - Emissão de eventos “PedidoAceito” e “PedidoRecusado”  

- **Gateway API**  
  - Ponto único de entrada HTTP  
  - Autenticação, autorização e roteamento de requisições  

- **Mensageria (RabbitMQ ou Kafka)**  
  - Fila de eventos entre Pedidos e Cozinha  
  - Tópicos para notificações em tempo real  

- **Observabilidade**  
  - Zabbix para health checks e alertas  
  - Prometheus Exporter + Grafana para métricas e dashboards  

- **CI/CD**  
  - GitHub Actions ou Azure DevOps  
  - Build, testes automatizados, security scans e deploy no cluster  

---

## Diagrama da Arquitetura da Solução (DDD)

```ascii
                   +--------------------+
                   |     API Gateway    |
                   +---------+----------+
                             |
  +--------------------------+-------------------------+
  |                          |                         |
+-----+    +---------+    +--------+    +-------------+
| Auth|--->| Cardápio|--->| Pedidos|--->|  Cozinha    |
| MS  |    |   MS    |    |   MS   |    |    MS       |
+--+--+    +---------+    +---+----+    +------+------+
   |                          |                 |
   |    +---------------------+                 |
   |    |                                       |
   v    v                                       v
 Users DB (SQL)                               Kitchen DB (SQL)
            \                                  /
             \                                /
          Events Topic (RabbitMQ / Kafka) ----

Observability                  CI/CD
+ Zabbix    + Grafana          + GitHub Actions
+ Prometheus
```

Justificativa Técnica das Decisões Arquiteturais
Arquitetura em microsserviços

Alta coesão e baixo acoplamento de domínios

Deploys independentes e escalonamento granular

Kubernetes para orquestração

Escalonamento automático e self-healing

Rolling updates sem downtime

Consistência entre ambientes de dev, staging e produção

Mensageria Assíncrona

Desacoplamento de produtores e consumidores (Pedidos ↔ Cozinha)

Garantia de entrega de eventos e retries automáticos

Suporta picos de carga sem degradar APIs

API Gateway

Centralização de políticas de autenticação e autorização

Roteamento dinâmico e versionamento de endpoints

Throttling e rate limiting integrados

Observabilidade Integrada

Zabbix para health checks, uptime e alertas críticos

Prometheus + Grafana para coleta de métricas e dashboards customizáveis

Alertas configuráveis por thresholds e regras de escalonamento

CI/CD Automatizado

Pipeline unificado: build, testes unitários/integrados, security scan

Deploy contínuo com rollback automático em caso de falhas

Gatilhos baseados em pull requests e branch policies

Persistência por Contexto

Bancos SQL isolados para cada microsserviço, garantindo ACID

Evita compartilhamento de schema e flutuação de performance

Possibilidade futura de adotar NoSQL para cache e consultas de alta velocidade

Segurança e Compliance

Autenticação stateless via JWT

Criptografia de dados em trânsito (TLS) e em repouso

Role-based Access Control (RBAC) aplicado no Gateway API
