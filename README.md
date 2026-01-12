# 🔐 Custom Secret Vault Platform

A distributed, highly available, and secure **Secret Management Platform** inspired by HashiCorp Vault and Azure Key Vault.

This system is designed for **multi-tenant (multi-merchant)** environments with **strong consistency**, **hardware-backed encryption**, **automatic rotation**, and **immutable auditing**.

---

## ✨ Key Characteristics

- 🔐 Secure secret storage with envelope encryption
- 🏢 Strong tenant (merchant) isolation
- 🔁 Automatic secret rotation
- 📊 Asynchronous, immutable audit logging
- ⚡ Read-optimized with caching
- 🧠 In-memory first design with interface segregation
- 🌍 Highly available and geo-distributed
- ☁️ Kubernetes-ready

---

## 🧩 Core Services

| Service | Responsibility |
|------|---------------|
| **Front Layer Service** | Authentication, authorization, tenant isolation, caching |
| **Secret CRUD Service** | Core secret lifecycle, encryption, versioning |
| **Rotation Service** | Scheduled and policy-based secret rotation |
| **Audit Consumer Service** | Asynchronous audit processing and storage |

---

## 🏗️ High-Level Architecture

```text
                ┌──────────────┐
                │ API Gateway  │
                └──────┬───────┘
                       │
                OAuth2 (JWT) + mTLS
                       │
        ┌──────────────▼──────────────┐
        │     Front Layer Service      │
        │  - AuthN / AuthZ             │
        │  - Tenant validation         │
        │  - Cache                     │
        └──────────────┬──────────────┘
                       │ mTLS
        ┌──────────────▼──────────────┐
        │     Secret CRUD Service      │
        │  - Encryption               │
        │  - Versioning               │
        │  - Strong consistency       │
        └──────────────┬──────────────┘
                       │
        ┌──────────────▼──────────────┐
        │   CockroachDB (Secrets)     │
        │   KMS / HSM (Keys)          │
        └──────────────┬──────────────┘
                       │
                   Kafka (Audit Events)
                       │
        ┌──────────────▼──────────────┐
        │    Audit Consumer Service    │
        │  - OpenSearch (Hot)          │
        │  - Object Storage (Cold)     │
        └─────────────────────────────┘


