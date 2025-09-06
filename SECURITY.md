# Security Policy

## Supported Versions

We provide security updates for the latest minor release line.

|        Version |       Supported       |
| -------------: | :-------------------: |
|         `main` |          ✅           |
| older branches | ❌ (best-effort only) |

---

## Reporting a Vulnerability

If you believe you’ve found a security issue:

- Email: **security@yourdomain.com**
- Subject: `[skyNest] Vulnerability Report`
- Include: affected endpoints, reproduction steps, logs, impact, and PoC if safe.
- Please **do not** create a public issue.

**Response targets**

- Acknowledge: **48 hours**
- Initial triage: **5 business days**
- Fix ETA: varies by severity; critical issues prioritized.

If you need encryption, request our PGP key via email.

---

## Scope

- **API**: NestJS service (App Router `/api/v1`, Swagger at `/api/v1/docs`)
- **Auth**: Access JWT + refresh JWT (cookies in production; Authorization header in dev)
- **Storage**: MongoDB Atlas cluster; object storage via AWS S3 / GCP Cloud Storage via PlatformModule
- **Cache**: Redis/Valkey (AWS ElastiCache) when enabled; disabled/no-op on other platforms
- **Edge**: Nginx reverse proxy + TLS
- **Front-end**: Next.js 15 (separate repository)
- **Infra**: Docker (multi-stage), EC2 deploy, GitHub Actions CI/CD
- **Integrations**: Twilio (OTP), SendGrid (email), Excel import/export

---

## Out of Scope (Examples)

- Social engineering
- Findings requiring privileged console access unavailable to attackers
- DoS via volumetric traffic (report rate-limit bypasses though)

---

## Coordinated Disclosure

Please allow us time to remediate before public disclosure. We’ll credit reporters who follow this policy and consent to being named.

---

## Data Protection & Privacy

- Treat all **student/PII** as sensitive: names, emails, phone numbers, IDs, admit cards, QR codes, remarks.
- Only log what’s necessary; **never log OTPs, tokens, or full payloads**.
- **Redact PII** in application logs by default; enable extended logging only in secure sandboxes.
- Exports (CSV/XLSX) must be **role-gated** and **watermarked** when feasible.

---

## Application Security Standards

### 1) Authentication & Sessions

- **JWT**
  - Access token: short TTL (e.g., `5–15m`)
  - Refresh token: longer TTL, **HTTP-only**, `SameSite=None; Secure` in production
  - Refresh tokens are **hashed** in DB; rotate on use and on login
- **Dev mode**: tokens may be sent via `Authorization: Bearer` header; **never** in localStorage in prod.
- **OTP**
  - Store only salted hash + expiry; throttle requests; cooldown on resend; max attempts.

### 2) Authorization

- **RBAC/ABAC via CASL**
  - Centralize abilities; enforce **per-route guards** + **service-level checks**
  - Deny-by-default; allow least privilege per role (ADMIN/STUDENT/COLLEGE)
  - Sensitive endpoints require explicit abilities (read/write/export).

### 3) Input Validation & Sanitization

- Use **DTOs with `class-validator`/`class-transformer`** on all inbound payloads.
- Protect against **NoSQL injection**: never pass raw strings into Mongo queries; whitelist fields.
- Strip/escape HTML in rich-text (remarks) to prevent stored XSS.
- Validate file uploads (MIME, size, extension) and scan if available.

### 4) Headers, TLS, and CORS

- Enable **Helmet** with a strict baseline. Example:
  - `X-Content-Type-Options: nosniff`
  - `X-Frame-Options: DENY`
  - `Referrer-Policy: no-referrer`
  - `Cross-Origin-Opener-Policy: same-origin`
  - `Cross-Origin-Resource-Policy: same-site`
- **CSP**: default-src 'self'; allow only required CDNs; lock down `img-src`, `connect-src` to API & storage domains.
- **CORS**: restrict to known origins via env (`ALLOWED_ORIGINS`), allow credentials only when needed.
- **TLS**: Enforce HTTPS at Nginx; redirect HTTP → HTTPS.

### 5) Rate Limiting & Abuse Controls

- Global + per-route rate limits (stricter for `/auth/*`, `/otp/*`).
- Brute-force protection with IP + user key.
- Soft lockouts / CAPTCHA when thresholds are exceeded.

### 6) Secrets & Config

- Store secrets in **environment variables** (never in Git).
- Rotate credentials for JWT, DB, Redis, Twilio, SendGrid on compromise.
- Use separate **dev/stage/prod** envs; different keys per environment.
- Limit GitHub PAT scopes (e.g., only the public README mirror repo).

### 7) File Handling (Admit Cards, Excel, QR)

- Uploads to S3/GCS via signed URLs; never trust client MIME.
- Keep uploads **private by default**; generate time-bound signed URLs for access.
- Parse Excel robustly; reject unrecognized columns; quarantine invalid rows.
- Strip metadata and verify QR content before storing.

### 8) Caching (Redis/Valkey)

- **Do not** cache authenticated responses unless explicitly allowed via decorator.
- Separate cache namespaces per environment.
- Use TLS/auth for Redis/Valkey; restrict network access to app subnets only.
- Graceful degrade when cache is unavailable.

### 9) Database (MongoDB Atlas)

- IP allow-list/VPC peering; SRV connection strings with TLS.
- **Principle of least privilege** DB user; separate read/write users if feasible.
- Unique indexes for natural keys; validate ObjectIds server-side.

### 10) Observability & Logs

- Nginx + app logs → Promtail → Loki → Grafana.
- Redaction middleware to remove tokens/PII from logs.
- **Security alerts** for: repeated auth failures, role-escalation attempts, suspicious exports.
- Keep logs for the minimum retention required; secure Loki with auth.

---

## Secure Coding Conventions (NestJS)

- Use **guards** (`JwtAuthGuard`, `BasicAuthGuard`) + **interceptors** for consistent behaviors.
- All controllers must declare **DTOs**; never accept `any`.
- **Swagger**: mark auth-required routes with security schemes; **hide** internal/admin routes from public docs when necessary.
- Centralized **exception filter**; no raw error messages to clients.
- Prefer parameterized queries and repository methods; **never** concatenate untrusted input.
- Add unit/integration tests for authz and negative cases.

---

## Dependency & Build Hygiene

- Automated updates via **Dependabot** (npm + GitHub Actions).
- CI runs: `npm audit --audit-level=moderate`, `npm run test`, `eslint`, `tsc --noEmit`.
- Docker: multi-stage builds, non-root user where possible, minimal base images.
- SBOM (optional): generate with `cyclonedx-npm`.

---

## Incident Response (IR)

1. **Detect**: alert from monitoring or report.
2. **Triage**: assign severity (Critical/High/Medium/Low).
3. **Contain**: revoke keys, block indicators, disable affected features if needed.
4. **Eradicate**: patch & deploy; rotate credentials.
5. **Recover**: validate fix; monitor closely.
6. **Postmortem**: document timeline, impact, lessons; update runbooks/tests.

---

## Hardening Checklists

### API (quick)

- [ ] Helmet + CSP enabled
- [ ] DTO validation everywhere
- [ ] Strict CORS origins
- [ ] Rate limit on auth/OTP
- [ ] Tokens in HTTP-only cookies (prod)
- [ ] CASL checks at controller + service layer
- [ ] Swagger secured (no sensitive routes exposed)

### Infra (quick)

- [ ] TLS enforced at Nginx
- [ ] Private networks for DB/Redis, no public ports
- [ ] Secrets from env/manager, not in repo
- [ ] CI secrets minimal scope, branch protection on `main`
- [ ] Loki/Grafana behind auth, PII redacted

---

## Example Configuration Snippets

**Helmet + CSP (NestJS)**

```ts
// main.ts
import helmet from 'helmet';
app.use(
  helmet({
    referrerPolicy: { policy: 'no-referrer' },
    crossOriginOpenerPolicy: { policy: 'same-origin' },
    crossOriginResourcePolicy: { policy: 'same-site' },
    contentSecurityPolicy: {
      useDefaults: true,
      directives: {
        'default-src': ["'self'"],
        'img-src': ["'self'", 'data:', 'https:'],
        'connect-src': ["'self'", process.env.ALLOWED_API_ORIGIN],
        'script-src': ["'self'"],
        'style-src': ["'self'", "'unsafe-inline'"],
      },
    },
  }),
);
```

**CORS (restrict by env)**

```ts
app.enableCors({
  origin: (origin, cb) => {
    const allowed = (process.env.ALLOWED_ORIGINS ?? '')
      .split(',')
      .map((s) => s.trim())
      .filter(Boolean);
    if (!origin || allowed.includes(origin)) cb(null, true);
    else cb(new Error('Not allowed by CORS'));
  },
  credentials: true,
});
```

**Rate Limit (example with @nestjs/throttler)**

```ts
ThrottlerModule.forRoot([
  {
    ttl: 60, // seconds
    limit: 20,
  },
]);
// Apply stricter guards to /auth,/otp routes
```

---

## Contact & Acknowledgements

Security contact: **security@yourdomain.com**  
We appreciate responsible disclosure and will credit reporters upon request.
