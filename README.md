# 🚀 SaaS Backend Platform - Portfolio Showcase

> 🔒 **Source Code is Private.** Contact me for access or technical deep-dive.

[![Java](https://img.shields.io/badge/Java-21-orange?logo=java)](https://www.java.com)
[![Spring Boot](https://img.shields.io/badge/Spring_Boot-4.0.4-green?logo=springboot)](https://spring.io/projects/spring-boot)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Live Demo](https://img.shields.io/badge/Live-Demo-brightgreen)](https://saas-backend-2sjw.onrender.com)

A production-grade multi-tenant SaaS backend built with Java 21 and Spring Boot 4, featuring enterprise-level security, scalability, and compliance patterns.

---

## 🌐 Live Demo

| Endpoint | URL | Status |
| :--- | :--- | :--- |
| **Health Check** | [GET /api/v1/health](https://saas-backend-2sjw.onrender.com/api/v1/health) | ✅ Live |
| **Root Welcome** | [GET /](https://saas-backend-2sjw.onrender.com) | ✅ Live |
| **Base URL** | `https://saas-backend-2sjw.onrender.com` | ✅ Live |

---

## ✨ Key Features

| Feature | Technology | Status |
| :--- | :--- | :--- |
| **Multi-Tenant Architecture** | Tenant-scoped queries, data isolation | ✅ Live |
| **API Key Authentication** | SHA-256 hashing, one-time display | ✅ Live |
| **Rate Limiting** | Redis sliding window + Lua scripts (50 req/60s) | ✅ Live |
| **Async Email Queue** | Redis queue, retry logic, dead letter queue | ✅ Live |
| **Tamper-Evident Audit** | SHA-256 hash chain, DB immutability triggers | ✅ Live |
| **Production Deployment** | Render + Neon PostgreSQL + Upstash Redis | ✅ Live |

---
## 🛠️ Tech Stack

| Category | Technologies |
| :--- | :--- |
| **Language** | Java 21 |
| **Framework** | Spring Boot 4, Spring Security, Spring Data JPA |
| **Database** | PostgreSQL 15 (Neon), Flyway Migrations |
| **Cache/Queue** | Redis 7 (Upstash), Redisson, Lua Scripts |
| **Email** | Resend API (Async Queue + Retry) |
| **Deployment** | Render.com, Docker, CI/CD |
| **Security** | SHA-256, BCrypt, API Keys, Audit Hash Chain |

---

## 🧪 API Documentation


### 1. Root endpoint

`GET /`

Purpose: quick welcome/status payload.

Example response:

```json
{
  "service": "SaaS Backend API",
  "status": "Running",
  "version": "1.0.0",
  "endpoints": {
    "health": "/api/v1/health",
    "register": "POST /api/v1/auth/register",
    "docs": "See README.md"
  }
}
```

### 2. Health endpoint

`GET /api/v1/health`

Purpose: readiness check for deployment/monitoring.

Example response:

```json
{
  "status": "UP",
  "service": "saas-backend",
  "timestamp": "2026-03-30T21:12:00Z"
}
```

### 3. Register tenant (public)

`POST /api/v1/auth/register`

Headers:

- `Content-Type: application/json`

JSON fields:

- `tenantName` (string, required)
- `tenantSlug` (string, required)
- `ownerEmail` (valid email, required)
- `password` (min 8 chars, required)

Request example:

```json
{
  "tenantName": "Acme Inc",
  "tenantSlug": "acme-inc",
  "ownerEmail": "owner@acme.com",
  "password": "StrongPass123"
}
```

curl example:

```bash
curl -X POST "$BASE_URL/api/v1/auth/register" \
  -H "Content-Type: application/json" \
  -d '{
    "tenantName": "Acme Inc",
    "tenantSlug": "acme-inc",
    "ownerEmail": "owner@acme.com",
    "password": "StrongPass123"
  }'
```

Success response (important parts):

- `apiKey` (raw secret, shown once)
- `tenant` object
- `user` object

Save `apiKey` immediately for all protected calls.

### 4. Create tenant (protected)

`POST /api/v1/tenants`

Headers:

- `Content-Type: application/json`
- `X-API-Key: <api_key_from_register>`

JSON fields:

- `name` (string, required)
- `slug` (string, required)

Request example:

```json
{
  "name": "Second Workspace",
  "slug": "second-workspace"
}
```

curl example:

```bash
curl -X POST "$BASE_URL/api/v1/tenants" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -d '{
    "name": "Second Workspace",
    "slug": "second-workspace"
  }'
```

### 5. List tenants (protected)

`GET /api/v1/tenants`

Headers:

- `X-API-Key: <api_key_from_register>`

curl example:

```bash
curl -H "X-API-Key: $API_KEY" "$BASE_URL/api/v1/tenants"
```

### 6. Audit logs (protected)

`GET /api/v1/audit/logs`

Query params:

- `cursor` (optional, long)
- `limit` (optional, default 50, max 200)
- `action` (optional)
- `resourceType` (optional)
- `startDate` and `endDate` (optional, ISO datetime)

Examples:

```bash
curl -H "X-API-Key: $API_KEY" "$BASE_URL/api/v1/audit/logs"
curl -H "X-API-Key: $API_KEY" "$BASE_URL/api/v1/audit/logs?limit=20&cursor=100"
curl -H "X-API-Key: $API_KEY" "$BASE_URL/api/v1/audit/logs?action=TENANT_CREATED"
```

### 7. Audit chain verification (protected)

`GET /api/v1/audit/verify`

Returns:

- `200` when chain is valid
- `400` when tampering is detected

curl example:

```bash
curl -H "X-API-Key: $API_KEY" "$BASE_URL/api/v1/audit/verify"
```

## Auth and Rate Limiting

### API key header

All protected routes require:

- `X-API-Key: <key>`

### Limit tiers

- Burst: 50 requests / 10 seconds (per API key prefix)
- Tenant: 1000 requests / 60 seconds (per tenant)

On limit breach:

- HTTP `429`
- `Retry-After` header
- JSON error with `RATE_LIMIT_EXCEEDED`

## End-to-End Test Script

```bash
# 1) Set base URL
BASE_URL="https://your-service.onrender.com"

# 2) Health
curl "$BASE_URL/api/v1/health"

# 3) Register
REGISTER_RESPONSE=$(curl -s -X POST "$BASE_URL/api/v1/auth/register" \
  -H "Content-Type: application/json" \
  -d '{
    "tenantName": "Live Test Tenant",
    "tenantSlug": "live-test-tenant",
    "ownerEmail": "owner@example.com",
    "password": "password123"
  }')

echo "$REGISTER_RESPONSE"

# 4) Extract API key manually from response and set it here
API_KEY="paste_api_key_here"

# 5) Call protected API
curl -H "X-API-Key: $API_KEY" "$BASE_URL/api/v1/tenants"
```

## Email Queue

### How It Works
- Registration enqueues a welcome email asynchronously via Redis queue
- Background worker processes jobs with **exponential backoff retry** (3 attempts max)
- Failed jobs after 3 retries move to a **Dead Letter Queue (DLQ)** for inspection
- All delivery attempts are logged in `email_delivery_logs` table for audit/compliance

### Email Provider: Resend
- **Demo/Sandbox**: Uses `onboarding@resend.dev` (no verification required)
- **Production**: Configure a verified domain in [Resend Domains](https://resend.com/domains) and update `RESEND_FROM_EMAIL`

### Environment Variables
| Variable | Purpose | Example |
| :--- | :--- | :--- |
| `RESEND_API_KEY` | Resend API authentication | `re_abc123...` |
| `RESEND_FROM_EMAIL` | Sender address (must be verified or sandbox) | `onboarding@resend.dev` |

### Testing Email
```bash
# Register a tenant (triggers welcome email)
curl -X POST "$BASE_URL/api/v1/auth/register" \
  -H "Content-Type: application/json" \
  -d '{
    "tenantName": "Email Test",
    "tenantSlug": "email-test",
    "ownerEmail": "your-real-email@gmail.com",
    "password": "password123"
  }'

# Check your inbox (including Spam/Promotions) for email from onboarding@resend.dev
```

## Security Notes

- API keys are stored hashed (SHA-256), raw key shown once.
- Passwords are BCrypt-hashed.
- Stateless API authentication.

## Known Production TODO

- Add OpenAPI/Swagger docs.
- Add integration tests for email transport and DLQ behavior.
