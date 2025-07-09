# FastTech Foods – MVP Solution

## Índice

- [Desenho da Solução MVP](#desenho-da-solução-mvp)  
- [Diagrama da Arquitetura da Solução (DDD)](#diagrama-da-arquitetura-da-solução-ddd)  
- [Justificativa Técnica das Decisões Arquiteturais](#justificativa-técnica-das-decisoes-arquiteturais)

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

# Justificativa Técnica das Decisões Arquiteturais

## 1. Arquitetura em microsserviços

- Alta coesão e baixo acoplamento de domínios  
- Deploys independentes e escalonamento granular  

## 2. Kubernetes para orquestração

- Escalonamento automático, self-healing e rollouts sem downtime  
- Homogeneidade entre ambientes de dev, staging e produção  

## 3. Mensageria Assíncrona

- Desacoplamento de produtores/consumidores (Pedidos ↔ Cozinha)  
- Tolerância a falhas e retries sem impactar a API  

## 4. API Gateway

- Centralização de segurança (autenticação e autorização)  
- Roteamento e versionamento unificado de APIs  

## 5. Observabilidade Integrada

- Zabbix para monitoramento de saúde e alertas  
- Grafana + Prometheus para métricas de performance  
- Dashboards customizáveis e thresholds automáticos  

## 6. CI/CD Automatizado

- Pipeline único: build, testes (unitários, integração e segurança)  
- Deploy contínuo com rollback automático em falhas  

## 7. Persistência por Contexto

- Bancos SQL isolados para consistência transacional  
- Opção futura de NoSQL para consultas de catálogo e históricos  

## 8. Segurança e Compliance

- JWT para autenticação stateless  
- Criptografia em repouso e em trânsito  
- Role-based Access Control (RBAC) no Gateway  
