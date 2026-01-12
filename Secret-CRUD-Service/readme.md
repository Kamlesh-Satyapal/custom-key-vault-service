# 🔐 Secret CRUD Service

The **Secret CRUD Service** is the core service responsible for **secure storage, retrieval, encryption, versioning, and consistency** of secrets in the Secret Vault Platform.

This service is the **only component** allowed to:
- Encrypt secrets
- Decrypt secrets
- Persist secret data
- Interact with the database

---

## 🎯 Responsibilities

- Create secrets for a merchant
- Update secrets with versioning
- Fetch latest or specific secret versions
- Delete (soft-delete) secrets
- Encrypt & decrypt secret values
- Enforce strong consistency
- Publish audit events

---

## 🧩 Service Boundaries

| Allowed | Not Allowed |
|------|------------|
| Encrypt / decrypt secrets | Authentication |
| Database access | Authorization decisions |
| Version management | Caching |
| Audit publishing | Plaintext persistence |

---

## 🏗️ Architecture Overview

```
Front Layer Service
        |
        | mTLS
        v
Secret CRUD Service
        |
        | JDBC
        v
PostgreSQL (Strong Consistency)
        |
        v
Kafka (Audit Events)
```

---

## 🗄️ Database Choice

### Current
- **PostgreSQL**
- ACID compliant
- Strong consistency
- Widely supported
- Easy local setup

### Future (Drop-in Replacement)
- CockroachDB
- YugabyteDB

> The service uses **repository interfaces** so database migration requires no business logic changes.

---

## 🧱 Data Model

### `merchant_secret`

| Column | Type | Description |
|------|------|-------------|
| id | UUID | Primary key |
| merchant_id | VARCHAR | Tenant identifier |
| secret_name | VARCHAR | Logical secret name |
| current_version | INT | Latest version |
| status | VARCHAR | ACTIVE / DELETED |
| created_at | TIMESTAMP | Creation time |
| updated_at | TIMESTAMP | Last update |

---

### `merchant_secret_version`

| Column | Type | Description |
|------|------|-------------|
| id | UUID | Primary key |
| secret_id | UUID | FK to merchant_secret |
| version | INT | Version number |
| encrypted_value | BYTEA | Encrypted secret |
| key_id | VARCHAR | KMS key reference |
| created_at | TIMESTAMP | Version creation time |

---

## 🔑 Encryption Model

- Envelope encryption
- Master keys stored in KMS / HSM
- Data key generated per secret
- Encryption happens **before persistence**
- Decryption happens **only on read**

> Plaintext secrets never touch disk or logs.

---

## 🔁 Versioning Strategy

- Every update creates a new version
- Latest version marked via `current_version`
- Older versions retained for rollback
- Rotation service uses versioning transparently

---

## 📜 Audit Events

Each operation publishes an audit event:

```json
{
  "merchantId": "merchant-123",
  "secretName": "db-password",
  "operation": "CREATE | READ | UPDATE | DELETE | ROTATE",
  "timestamp": "ISO-8601"
}
```

Publishing is **asynchronous** via Kafka.

---

## 🧠 Interface-First Design

### Secret Repository

```java
public interface SecretRepository {
    void saveSecret(Secret secret);
    void saveVersion(SecretVersion version);
    Optional<SecretVersion> findLatest(String merchantId, String secretName);
}
```

### Encryption Service

```java
public interface EncryptionService {
    byte[] encrypt(byte[] plaintext);
    byte[] decrypt(byte[] ciphertext);
}
```

### Audit Publisher

```java
public interface AuditPublisher {
    void publish(AuditEvent event);
}
```

---

## 🧪 Default Implementations

| Interface | Implementation |
|--------|---------------|
| SecretRepository | JPA (Postgres) |
| EncryptionService | Local AES (dev) |
| AuditPublisher | Kafka Producer |

---

## ⚙️ Consistency Guarantees

- Single write path
- Serializable transactions
- Read-after-write consistency
- No eventual consistency

---

## 🚀 Running Locally

### Prerequisites
- Java 17+
- PostgreSQL
- Kafka (optional for audit)

### Local DB Setup

```sql
CREATE DATABASE secret_vault;
```

Configure `application.yml` accordingly.

---

## 🔮 Future Enhancements

- CockroachDB migration
- Transparent multi-region reads
- HSM-backed encryption
- Soft delete with retention policies

---

## 🏁 Summary

The Secret CRUD Service is the **security backbone** of the platform.

It guarantees:
- Strong consistency
- Secure encryption
- Versioned secrets
- Auditable operations
