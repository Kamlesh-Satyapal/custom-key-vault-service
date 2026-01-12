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
        │     Front Layer Service     │
        │  - AuthN / AuthZ            │
        │  - Tenant validation        │
        │  - Cache                    │
        └──────────────┬──────────────┘
                       │ mTLS
        ┌──────────────▼──────────────┐
        │     Secret CRUD Service     │
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
        │    Audit Consumer Service   │
        │  - OpenSearch (Hot)         │
        │  - Object Storage (Cold)    │
        └─────────────────────────────┘


## 🏗️ High-Level Architecture

API Gateway  
→ Front Layer Service (JWT + mTLS + Cache)  
→ Secret CRUD Service (Encryption + Versioning)  
→ CockroachDB (Strong Consistency)  
→ Kafka → Audit Consumer Service  

---

## 🗄️ Data Storage Strategy

### Secrets
- CockroachDB
- Strong consistency (Raft)
- Read/write replicas

### Cache
- Redis
- TTL-based

### Audit
- Kafka
- OpenSearch (hot)
- Object storage (cold)

---

## 🔐 Security Model

- OAuth2 + JWT authentication
- RBAC authorization
- Merchant isolation using:
  JWT.merchantId == mTLS.cert.merchantId == request.merchantId

---

## 🔑 Encryption Model

- Envelope encryption
- Master keys stored in KMS / HSM
- Encryption handled only in CRUD Service
- No plaintext persistence

---

## 🔁 Secret Rotation

- Time-based or policy-based
- New version created on rotation
- Backward-compatible reads

---

## 📜 Auditing

All operations are asynchronously audited:

- CREATE
- READ
- UPDATE
- ROTATE
- DELETE

Audit logs are immutable and append-only.

---

## 🧠 Design Principles

- Interface segregation
- Stateless services
- In-memory defaults for development
- Pluggable production implementations

---

## 🚀 Deployment

- Kubernetes
- Stateless microservices
- Horizontal scaling
- Multi-region support
- Zero-trust networking

---

## 🔮 Future Enhancements

- Policy-based access
- Dynamic secrets
- Secret leasing
- Disaster recovery automation

---

## 🏁 Summary

An enterprise-grade secret management platform designed with **security-first**, **scalability**, and **clean architecture** principles.

