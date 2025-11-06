---
title: Target Architecture - Container Diagram
last_updated: 2025-11-06
platform: OpenShift (Service Mesh enabled)
purpose: AI-Assisted EA Demo Target (final state for demo)
---

# Target Container Diagram

```mermaid
flowchart TB
%% C4-Style Container Diagram (Mermaid近似)
%% 外部境界
User[利用者\n(User)]:::person

subgraph Web["ブラウザ / デバイス"]
  LunchApp[LunchApp\n(Frontend SPA)]:::fe
  UsageCheckApp[UsageCheckApp\n(Frontend SPA)]:::fe
  HealthcareApp[HealthcareManagerApp\n(Frontend SPA)]:::fe
end

User -->|操作| LunchApp
User -->|操作| UsageCheckApp
User -->|操作| HealthcareApp

%% プラットフォーム境界
subgraph OCP["OpenShift クラスタ（Service Mesh）"]
  Keycloak[(Keycloak\nAuthN/AuthZ IdP)]:::infra
  Kafka[(AMQ Streams / Kafka\nEvent Streaming)]:::infra
  Debezium[(Debezium / Connect)]:::infra

  %% バックエンド群
  Bill[Bill API\n(Spring Boot)]:::svc
  Employee[Employee API\n(Spring Boot)]:::svc
  History[History API\n(Spring Boot)]:::svc
  Menu[Menu API\n(Spring Boot)]:::svc
  Payment[Payment API\n(Spring Boot)]:::svc
  Private[Private API\n(Spring Boot)]:::svc
  Transaction[Transaction API\n(Spring Boot)]:::svc
  UserProfile[UserProfile API\n(Spring Boot)]:::svc

  %% データベース（サービス毎のスキーマ/DB想定）
  BillDB[(PostgreSQL\nbilldb)]:::db
  EmpDB[(PostgreSQL\nemployeedb)]:::db
  HistDB[(PostgreSQL\nhistorydb)]:::db
  MenuDB[(PostgreSQL\nmenudb)]:::db
  PayDB[(PostgreSQL\npaymentdb)]:::db
  PrivDB[(PostgreSQL\nprivatedb)]:::db
  TxDB[(PostgreSQL\ntransactiondb)]:::db
  UPDB[(PostgreSQL\nuserprofiledb)]:::db

  %% 認可・通信
  LunchApp -->|OIDC/OAuth2| Keycloak
  UsageCheckApp -->|OIDC/OAuth2| Keycloak
  HealthcareApp -->|OIDC/OAuth2| Keycloak

  %% フロント→バック
  LunchApp -->|REST/JSON| Menu
  LunchApp -->|REST/JSON| Payment
  LunchApp -->|REST/JSON| Transaction
  UsageCheckApp -->|REST/JSON| Private
  HealthcareApp -->|REST/JSON| Employee

  %% サービス間連携（例）
  Payment -->|REST| Transaction
  Bill -->|REST| Transaction

  %% イベント連携
  Transaction <-->|Produce/Consume| Kafka
  Payment <-->|Produce/Consume| Kafka
  Bill <-->|Consume| Kafka
  Menu <-->|Consume| Kafka
  Employee <-->|Consume| Kafka
  History <-->|Consume| Kafka

  %% CDCなど（任意）
  Debezium -->|CDC Events| Kafka

  %% 永続化
  Bill --> BillDB
  Employee --> EmpDB
  History --> HistDB
  Menu --> MenuDB
  Payment --> PayDB
  Private --> PrivDB
  Transaction --> TxDB
  UserProfile --> UPDB
end

%% スタイル
classDef person fill:#fff,stroke:#333,stroke-width:1px,rx:6,ry:6;
classDef fe fill:#e7f3ff,stroke:#1b6dd1,rx:6,ry:6;
classDef svc fill:#eef8ec,stroke:#2e7d32,rx:6,ry:6;
classDef db fill:#fff7e6,stroke:#e69500,rx:6,ry:6;
classDef infra fill:#f3e8fd,stroke:#7e57c2,rx:6,ry:6;