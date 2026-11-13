# 07 — Security Threat Model & Checklist

> Status: Draft

## 1. Assets

1. Tenant business data (sales, inventory)
2. User credentials & sessions
3. LLM API key / spending budget
4. AI audit logs (contain business data)
5. Database & infrastructure

## 2. Threat model (STRIDE-lite)

| # | Threat | Vector | Mitigation | Test |
|---|---|---|---|---|
| T1 | Cross-tenant data access | IDOR (เปลี่ยน UUID), org_id ใน request | Org from session; `WHERE organization_id=$1` ทุก query; composite FKs; 404 for foreign IDs | 06 §3 |
| T2 | Cross-tenant via AI | Prompt บอกให้ใช้ org อื่น | Tool args ไม่มี org field; Principal-only scope | Golden #15–16 |
| T3 | Prompt injection → unauthorized action | User text / product names | Read-only tools, allow-list, role filter, schema validation | 06 §5, golden #17–18 |
| T4 | Hallucinated numbers | LLM | Evidence built by backend, fidelity check, UI warning | 06 §5 |
| T5 | Credential stuffing / brute force | Login | Rate limit per IP+email, bcrypt cost 12, generic error | 06 §4 |
| T6 | Session theft | XSS, cookie leak | httpOnly+Secure+SameSite cookie, hashed token at rest, CSP, React escaping, no `dangerouslySetInnerHTML` for AI text | Manual + E2E |
| T7 | CSRF | Cross-site POST | SameSite=Lax + Origin check + custom header | 06 §4 |
| T8 | SQL injection | Filters, search | pgx parameterized queries only; sort allow-list | Code review + tests |
| T9 | LLM cost abuse / DoS | Spam questions, long input | Rate limit per user/org, message length, max tokens/iterations, timeout, daily org cap | 06 §5 |
| T10 | Secret leakage | Logs, errors, repo | Env secrets, `.gitignore`, log redaction, safe error mapping, secret scan | CI gitleaks |
| T11 | System prompt / internal info leak | Prompt | No secrets in prompt; error mapping hides internals | Golden #19 |
| T12 | Privilege escalation | Role change API | Owner-only, last-owner guard, role from DB not session cache | 06 §4 |
| T13 | Audit log tampering / over-collection | — | App DB role no DELETE on audit tables (V1.1), sanitize args, no raw prompts with secrets | Review |
| T14 | Exposed database | Docker network | Postgres not published in prod compose; internal network only | Deploy review |
| T15 | Vulnerable dependencies | Supply chain | Pin versions, `govulncheck`, `npm audit`, Dependabot | CI |

## 3. Checklist (MVP)

### Authentication & session
- [ ] bcrypt cost ≥ 12; password min length 10
- [ ] Session token 32 bytes crypto/rand; stored as SHA-256
- [ ] Cookie: `HttpOnly; Secure; SameSite=Lax; Path=/`
- [ ] Session expiry + revoke on logout
- [ ] Login rate limit

### Authorization & tenancy
- [ ] RBAC middleware per route (deny by default)
- [ ] Every tenant repository function takes orgID; lint/review rule
- [ ] Composite FKs prevent cross-org references
- [ ] Route coverage test for tenant isolation

### Input & output
- [ ] Body size limit 16 KB
- [ ] Validation at handler; invariants in domain
- [ ] Sort/filter allow-lists
- [ ] Error mapper never returns stack/SQL text
- [ ] AI text rendered as plain text / sanitized markdown

### AI-specific
- [ ] Tool allow-list per role
- [ ] Strict JSON decode (`DisallowUnknownFields`)
- [ ] Iteration/tool-call/time/token limits
- [ ] Rate limit per user + daily cap per org
- [ ] Fidelity check + audit status
- [ ] No API keys / cookies / system prompt in audit or logs

### Transport & headers
- [ ] HTTPS via Nginx Proxy Manager
- [ ] CORS restricted to frontend origin, credentials allowed only for it
- [ ] Security headers: CSP, X-Content-Type-Options, Referrer-Policy, frame-ancestors

### Secrets & infra
- [ ] `.env` in `.gitignore`; `.env.example` only
- [ ] Secrets via Portainer stack env
- [ ] Postgres not exposed publicly; separate DB user with least privilege
- [ ] Backups encrypted/stored off-host; restore tested
- [ ] gitleaks + govulncheck + npm audit in CI

## 4. Out of scope for MVP (documented risk)

- MFA, SSO
- Row-Level Security in PostgreSQL (consider V1.1 as second layer)
- WAF / bot protection for public demo (mitigated by rate limits + daily cap)
