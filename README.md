# Nankoyo Team API – Public Documentation

> **Note**  
> The main PHP codebase (team portal + management system) is **closed-source** while we finalize commercial licensing.  
> This single document hosts all public-facing specs, guidelines and examples.  
> Live system: [https://nankoyo.com](https://nankoyo.com)

## 1. Document Map
1. Authentication flow  
2. Endpoint catalogue  
3. Error & rate-limit model  
4. Webhook event catalogue  
5. Changelog & release cycle  
6. Support & contribution channel  

Reading time ≈ 6 min; everything is searchable in-page.

## 2. Authentication
- Bearer access token + refresh token (JWT, RS256, 2 h TTL)  
- Login endpoint: `POST /api/login`  
- Refresh endpoint: `POST /api/refresh`  
- All scoped endpoints require `Authorization: Bearer <token>` header  
- Role matrix: `public` (read-only team list) → `user` (dashboard) → `admin` (write)  

## 3. Endpoints Overview
Base URL: `https://nankoyo.com/api`  

| Method | Path | Scope | Purpose |
|--------|------|-------|---------|
| GET | /team | public | Paginated team roster |
| POST | /team | admin | Add new member |
| PUT | /team/{id} | admin | Update profile |
| DELETE | /team/{id} | admin | Remove member |
| GET | /dashboard/stats | user | KPI counters |
| POST | /webhooks | admin | Register callback URL |
| GET | /health | public | Service health & build tag |

Full request/response schema is Swagger-compatible; ask for the OpenAPI file if you need auto-generated clients.

## 4. Error & Rate-Limit Model
HTTP status + machine-readable code inside JSON `"error"` object.  
Global limit: 100 req/min per IP, burst 20. Retry-After header supplied.

| HTTP | Code | Message | Notes |
|------|------|---------|-------|
| 401 | 1001 | Unauthorized | Missing/invalid token |
| 403 | 1003 | Forbidden | Role too low |
| 422 | 2001 | Validation Error | Details in `errors[]` |
| 429 | 3001 | Too Many Requests | Back-off required |
| 500 | 9001 | Internal Error | Log ID returned for support |

## 5. Webhooks
Events are signed with HMAC-SHA256 (`whsec_*` secret).  
Payload version: 1 (semver).  
Available topics: `team.member_added`, `team.member_removed`, `dashboard.kpi_alert`.  
Delivery timeout 5 s, 3 retries with exponential back-off.

## 6. Changelog & Release Cycle
We follow SemVer and calendar-based cadence (minor release every 4 weeks, patches anytime).  
Public changelog starts at v1.0.0 (2024-05-01). Breaking changes are announced 30 days ahead via email and dedicated `/changelog` RSS.

## 7. Security & Compliance
- All traffic TLS 1.3, cert by Cloudflare (strict mode)  
- Passwords hashed with Argon2id, 12-bit cost  
- OWASP Top-10 review before each minor release  
- GDPR data-processing register available under NDA  

## 8. Support Channel
- Technical: open an issue in this documentation repo  
- Security: security@nankoyo.com (PGP key on website)  
- Commercial: hello@nankoyo.com  

## 9. Roadmap (public commitments)
- Q3 2024 – WebSocket real-time dashboard  
- Q4 2024 – SCIM 2.0 provisioning  
- Q1 2025 – Open-source Node & Python SDK (license MIT)  

## 10. Contribution Note
Pull-requests are welcome **only for docs, examples and SDKs**; core platform remains closed until further notice.

------------------------------------------------
Thank you for integrating with Nankoyo. We strive to keep response times < 150 ms (p99) and publish quarterly uptime reports on our blog.
