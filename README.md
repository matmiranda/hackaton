# FastTech Foods – MVP Solution

## Índice

- [Linguagem Ubíqua](#linguagem-ubíqua)
- [Desenho da Solução MVP](#desenho-da-solução-mvp)  
- [Diagrama da Arquitetura da Solução (DDD)](#diagrama-da-arquitetura-da-solução-ddd)  
- [Justificativa Técnica das Decisões Arquiteturais](#justificativa-técnica-das-decisões-arquiteturais)
- [Detalhamento dos microsserviços](#detalhamento-dos-microsserviços)
- [Script Mysql](#script-mysql--fasttech-foods)

---

## Linguagem Ubíqua

Este projeto utiliza um vocabulário compartilhado entre desenvolvedores e especialistas de domínio para garantir clareza e alinhamento:

- Cliente (Customer)  
- Funcionário (Employee)  
- Gerente (Manager)  
- Cardápio (Menu)  
- Pedido (Order)  
- Entrega (Delivery)  
- Autenticação (Authentication)  
- Disponibilidade (Availability)  

---

## System Contexts

### Client Side

- Cadastro do Cliente: registro via e-mail ou CPF  
- Autenticação do Cliente: e-mail ou CPF + senha  
- Catálogo Online: visualização e busca de produtos  
- Montagem de Pedido: seleção de itens, cálculo de total, escolha de modalidade de entrega  

### Employee Side

- Autenticação do Funcionário: e-mail corporativo + senha  
- Gerência de Cardápio: cadastrar, editar nome, descrição, preço e disponibilidade  
- Equipe de Cozinha: recepção de pedidos, opção de aceitar ou recusar  

---

## Actors

- Chefe da Cozinha  
- Gerente do FastTech Foods  
- Atendente (Balcão)  
- Atendente (Drive-Thru)  
- Entregador (Delivery)  

---

## Features

### Cliente

1. Cadastro de conta  
2. Login via e-mail ou CPF  
3. Busca e filtragem de produtos (lanche, sobremesa, bebida)  
4. Montagem de pedido e visualização de total  
5. Seleção de modalidade de entrega (balcão, drive-thru, delivery)  

### Funcionário

1. Login com e-mail corporativo  
2. Gerente: gerenciamento de itens do cardápio  
3. Cozinha: visualização e tratamento de pedidos  

---

## Menu de Produtos

### Lanches (Hambúrguer)

| Item               | Ingredientes                                                                 |
|--------------------|-------------------------------------------------------------------------------|
| X Bacon            | Alface, Tomate, Hambúrguer, Presunto, Mussarela, Bacon                        |
| Bacon Especial     | Alface, Tomate, Batata Palha, Milho, Hambúrguer, Presunto, Mussarela, Bacon   |
| X-Frango           | Alface, Tomate, Batata Palha, Milho, Filé de Frango, Presunto, Mussarela, Bacon |
| Frango Especial    | Alface, Tomate, Batata Palha, Milho, Filé de Frango, Presunto, Mussarela, Bacon, Salsicha, Ovo |
| Especiais          | Alface, Tomate, Batata Palha, Milho, Hambúrguer, Mussarela, Bacon             |
| Bacon Mega         | Alface, Tomate, Batata Palha, Milho, Hambúrguer, Presunto, Mussarela, Bacon, Salsicha, Ovo |
| X-Frango Turbo     | Alface, Tomate, Batata Palha, Milho, Filé de Frango, Presunto, Mussarela, Bacon |

### Esfihas

| Item         | Ingredientes            |
|--------------|-------------------------|
| Carne Seca   | Carne, Mussarela        |
| Calabresa    | Calabresa, Cebola       |
| Bauru        | Presunto, Queijo, Tomate|
| Pizza        | Queijo, Tomate, Orégano |
| Bacon        | Bacon, Queijo           |
| Atum         | Atum, Cebola            |
| Dois Queijos | Mussarela, Requeijão    |

### Sobremesas (Sorvete)

| Item                  | Ingredientes             |
|-----------------------|--------------------------|
| Ice Cream             | Vanilla                  |
| Chocolate             | Chocolate                |
| Strawberry            | Morango, Calda de Morango|
| Napolitano            | Morango, Chocolate       |
| Baunilha c/ Chocolate | Baunilha, Chocolate      |
| Brigadeiro            | Chocolate, Brigadeiro    |
| Maracujá              | Maracujá                 |

### Bebidas

| Item                       | Observações |
|----------------------------|-------------|
| Água Mineral com Gás       | —           |
| Água Mineral sem Gás       | —           |
| Schweppes Citrus           | —           |
| Coco Verde                 | —           |
| Coca-Cola Lata             | —           |
| Heineken Lata              | —           |
| Skol Lata                  | —           |

---

## Bounded Contexts & Context Map

- Cliente Context: cadastro, busca, pedido  
- Cardápio Context: definição e edição de itens  
- Cozinha Context: aceitação e rejeição de pedidos  
- Atendimento Context: balcão, drive-thru, delivery  

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
  - Prometheus Exporter + Grafana para métricas e dashboards  

- **CI/CD**  
  - GitHub Actions ou Azure DevOps  
  - Build, testes automatizados, security scans e deploy no cluster  

---

## Diagrama da Arquitetura da Solução (DDD)

```mermaid
flowchart LR
  %% Contextos / Databases
  subgraph Autenticação
    AuthMS[Auth MS]
    AuthDB[(auth_db)]
  end

  subgraph Cardápio
    CardMS[Menu MS]
    CardDB[(menu_db)]
  end

  subgraph Pedidos
    OrderMS[Order MS]
    OrderDB[(order_db)]
  end

  subgraph Cozinha
    KitchenMS[Kitchen MS]
    KitchenDB[(kitchen_db)]
  end

  %% Infraestrutura
  subgraph Infraestrutura
    API[API Gateway]
    MQ[(RabbitMQ)]
    Grafana[(Grafana)]
  end

  %% Fluxos REST
  API --> AuthMS
  API --> CardMS
  API --> OrderMS
  API --> KitchenMS

  %% Publicação de eventos (Producer)
  CardMS -->|menu_item.unavailable| MQ
  OrderMS -->|order.created / order.cancelled| MQ
  KitchenMS -->|order.accepted / order.rejected| MQ

  %% Consumo de eventos (Consumer)
  MQ --> CardMS
  MQ --> KitchenMS
  MQ --> OrderMS

  %% Persistência
  AuthMS --> AuthDB
  CardMS --> CardDB
  OrderMS --> OrderDB
  KitchenMS --> KitchenDB

  %% Observabilidade
  AuthMS & CardMS & OrderMS & KitchenMS --> Grafana
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

- Grafana + Prometheus para métricas de performance  
- Dashboards customizáveis e thresholds automáticos  

## 6. CI/CD Automatizado

- Pipeline único: build, testes (unitários, integração e segurança)    

## 7. Persistência por Contexto

- Bancos SQL isolados para consistência transacional  

## 8. Segurança e Compliance

- JWT para autenticação stateless  
- Criptografia em repouso e em trânsito  
- Role-based Access Control (RBAC) no Gateway
---

# Bancos de Dados por Serviço

## Serviços e seus Schemas

| Serviço         | Database   | Tabelas              |
|-----------------|------------|----------------------|
| Auth Service    | `auth_db`  | `users`              |
| Menu Service    | `menu_db`  | `menu_items`         |
| Order Service   | `order_db` | `orders`, `order_items` |
| Kitchen Service | `kitchen_db` | `kitchen_events`     |

---


## Script MySQL – FastTech Foods

```sql
CREATE DATABASE IF NOT EXISTS auth_db
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

CREATE DATABASE IF NOT EXISTS menu_db
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

CREATE DATABASE IF NOT EXISTS order_db
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

CREATE DATABASE IF NOT EXISTS kitchen_db
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;


USE auth_db;

CREATE TABLE `user_roles` (
  `id` TINYINT UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(50) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `name_unique` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

INSERT INTO `user_roles` (`name`) VALUES 
('CLIENTE'),
('ATENDENTE'),
('GERENTE'),
('COZINHEIRO');

CREATE TABLE `users` (
  `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(100) COLLATE utf8mb4_unicode_ci NOT NULL,
  `email` VARCHAR(255) COLLATE utf8mb4_unicode_ci NOT NULL,
  `cpf` CHAR(11) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `password_hash` VARCHAR(255) COLLATE utf8mb4_unicode_ci NOT NULL,
  `role_id` TINYINT UNSIGNED NOT NULL,
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `email` (`email`),
  UNIQUE KEY `cpf` (`cpf`),
  FOREIGN KEY (`role_id`) REFERENCES `user_roles`(`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;


USE menu_db;

-- Tabela de tipos de refeição
CREATE TABLE IF NOT EXISTS meal_types (
  id TINYINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL UNIQUE,    -- Ex: 'LANCHES', 'BEBIDAS'
  name VARCHAR(100) NOT NULL,          -- Ex: 'Lanches', 'Bebidas'
  description TEXT
) ENGINE=InnoDB
  DEFAULT CHARSET = utf8mb4
  COLLATE = utf8mb4_unicode_ci;

INSERT INTO meal_types (code, name, description) VALUES
  ('LANCHES', 'Lanches', 'Sanduíches, salgados e porções rápidas'),
  ('BEBIDAS', 'Bebidas', 'Sucos, refrigerantes, cafés e drinks'),
  ('PRATOS_PRINCIPAIS', 'Pratos Principais', 'Refeições completas como almoços e jantares'),
  ('SOBREMESAS', 'Sobremesas', 'Doces, bolos, sorvetes e guloseimas'),
  ('VEGETARIANO', 'Vegetariano', 'Opções sem carne, com foco em vegetais');

-- Tabela principal do cardápio
CREATE TABLE IF NOT EXISTS menu_items (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(150) NOT NULL,
  description TEXT,
  price DECIMAL(10,2) NOT NULL CHECK (price >= 0),
  meal_type_id TINYINT UNSIGNED NOT NULL,
  available BOOLEAN NOT NULL DEFAULT TRUE,
  image_url VARCHAR(255),
  tags JSON,
  calories INT UNSIGNED,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
                                 ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (meal_type_id) REFERENCES meal_types(id),
  INDEX idx_menu_meal_type (meal_type_id),
  INDEX idx_menu_available (available),
  INDEX idx_menu_price (price)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4
  COLLATE = utf8mb4_unicode_ci;



USE order_db;

CREATE TABLE IF NOT EXISTS orders (
  id               BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  customer_id      BIGINT UNSIGNED NOT NULL,
  delivery_method  ENUM('BALCAO','DRIVE_THRU','DELIVERY') NOT NULL,
  total            DECIMAL(10,2) NOT NULL CHECK (total >= 0),
  status           ENUM('PENDENTE','EM_PREPARO','PRONTO','CANCELADO') NOT NULL DEFAULT 'PENDENTE',
  cancel_reason    TEXT,
  created_at       TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at       TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  INDEX idx_orders_status         (status),
  INDEX idx_orders_created        (created_at),
  INDEX idx_orders_customer_status(customer_id, status)
) ENGINE=InnoDB
  DEFAULT CHARSET = utf8mb4
  COLLATE = utf8mb4_unicode_ci;


CREATE TABLE IF NOT EXISTS order_items (
  id             BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  order_id       BIGINT UNSIGNED NOT NULL,
  menu_item_id   BIGINT UNSIGNED NOT NULL,
  quantity       INT UNSIGNED      NOT NULL CHECK (quantity > 0),
  price_at_order DECIMAL(10,2)     NOT NULL CHECK (price_at_order >= 0),
  total_item     DECIMAL(10,2) GENERATED ALWAYS AS (quantity * price_at_order) STORED,
  created_at     TIMESTAMP         NOT NULL DEFAULT CURRENT_TIMESTAMP,

  FOREIGN KEY (order_id)
    REFERENCES orders(id) ON DELETE CASCADE,

  FOREIGN KEY (menu_item_id)
    REFERENCES menu_db.menu_items(id),

  INDEX idx_order_items_order (order_id),
  INDEX idx_order_items_menu  (menu_item_id)
) ENGINE=InnoDB
  DEFAULT CHARSET = utf8mb4
  COLLATE = utf8mb4_unicode_ci;


USE kitchen_db;

CREATE TABLE IF NOT EXISTS kitchen_events (
  id            BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  order_id      BIGINT UNSIGNED NOT NULL,
  action        ENUM('ACEITO','RECUSADO') NOT NULL,
  justification TEXT,
  created_at    TIMESTAMP          NOT NULL DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (order_id)
    REFERENCES order_db.orders(id) ON DELETE CASCADE,
  INDEX idx_kitchen_events_order (order_id)
) ENGINE=InnoDB
  DEFAULT CHARSET = utf8mb4
  COLLATE = utf8mb4_unicode_ci;


```


## Curls

```
curl --location 'https://apim-fiap.azure-api.net/api/auth/register' \
--header 'Ocp-Apim-Subscription-Key: 942dfc762bb745b6a80d42aefe26b772' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "string",
    "email": "user@example.com",
    "cpf": "99399158053",
    "password": "string",
    "role": "string",
    "position": "string"
}'

curl --location 'https://apim-fiap.azure-api.net/api/auth/login' \
--header 'Ocp-Apim-Subscription-Key: 942dfc762bb745b6a80d42aefe26b772' \
--header 'Content-Type: application/json' \
--data '{
    "email": "string",
    "password": "string"
}'

curl --location --request GET 'https://apim-fiap.azure-api.net//api/auth/validate' \
--header 'Ocp-Apim-Subscription-Key: 942dfc762bb745b6a80d42aefe26b772' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIzZjY1MGY2ZC1mMzIwLTQyMzEtYjM0My03ZWFhYTk0YzY5ZTUiLCJlbWFpbCI6InVzZXJAZXhhbXBsZS5jb20iLCJqdGkiOiJmOTQ5MTVmNS1iMTNiLTQ3ZDYtOThiMi0yM2I0NTQ3YzA3Y2UiLCJleHAiOjE3NTMwMzI1NjEsImlzcyI6IkF1dGhTZXJ2aWNlLkFwaSIsImF1ZCI6IkF1dGhTZXJ2aWNlVXNlcnMifQ.m_cAH48jW-1rf33VXQsXiySRBA_MgChGzjlLkRjwb2U' \
--data '{
    "email": "string",
    "password": "string"
}'
```

