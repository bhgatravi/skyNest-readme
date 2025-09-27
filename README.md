# SkyNest API 🚀

SkyNest API is a production-grade, scalable, and platform-agnostic backend built with **NestJS**, **MongoDB**, and **Docker**, designed for extensible deployments across multiple cloud providers (AWS, GCP, Azure). It supports user authentication, OTP verification, platform-specific dependency injection, dynamic environment handling, and secure refresh-token-based session management.

---

# API — Developer Guide

A NestJS + TypeScript API with a strong focus on **code quality**, **consistency**, and **DX**.  
This project ships with **ESLint (flat config)**, **Prettier**, **Husky + lint-staged** pre-commit hooks, a **centralized exception handler**, and **Swagger** docs.

---

## Table of Contents

- [Tech Stack](#tech-stack)
- [Quick Start](#quick-start)
- [Project Scripts](#project-scripts)
- [Code Style & Linting](#code-style--linting)
- [Pre-commit Hooks](#pre-commit-hooks)
- [Error Handling (Centralized)](#error-handling-centralized)
- [API Docs (Swagger)](#api-docs-swagger)
- [Environment Variables](#environment-variables)
- [Storage Platform Selection](#storage-platform-selection)
- [CORS](#cors)
- [Conventional Patterns](#conventional-patterns)
- [Troubleshooting](#troubleshooting)

---

## Tech Stack

- **Runtime**: Node.js (LTS recommended)
- **Framework**: NestJS (Express)
- **Language**: TypeScript
- **DB**: MongoDB (Mongoose)
- **Auth**: JWT (access & refresh)
- **Mail**: Nodemailer and/or SendGrid
- **File Storage**: AWS S3 / Azure Blob / GCP (pluggable via `PlatformModule`)
- **Docs**: Swagger (OpenAPI)
- **Quality**: ESLint (flat config), Prettier, Husky, lint-staged

---

## Quick Start

```bash
# 1) Install dependencies
npm i

# 2) Set up environment (copy and edit as needed)
cp .env.example .env

# 3) Enable Git hooks (Husky)
npm run prepare

# 4) Start dev server
npm run start:dev

# 5) Lint & format (optional)
npm run lint
npm run lint:fix
npm run format
```

Swagger UI will be available at: `http://localhost:<PORT>/<API_PREFIX>/docs`  
(Defaults: `PORT=4000`, `API_PREFIX=v1`)

---

## Project Scripts

```jsonc
// package.json (relevant)
{
  "scripts": {
    "start": "nest start",
    "start:dev": "nest start --watch",
    "build": "nest build",

    "lint": "eslint .",
    "lint:fix": "eslint . --fix",

    "format": "prettier --write .",
    "format:check": "prettier --check .",

    "prepare": "husky install",
  },
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix"],
    "*.{js,json,md,css,scss,html,yml,yaml}": ["prettier --write"],
  },
}
```

> **Note:** With ESLint **flat config** (`eslint.config.js`), flags like `--ext` are not needed. Use `eslint .`.

---

## Code Style & Linting

- **ESLint (flat config)** enforces:
  - import ordering (`import/order`)
  - unused import removal (`unused-imports/no-unused-imports`)
  - sensible TS rules for Nest/Mongoose patterns
- **Prettier** handles code formatting (aligned with GitHub UI)

**Prettier config** (`.prettierrc`):

```json
{
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "semi": true,
  "arrowParens": "always"
}
```

**Common lint tips**

- Remove `async` if there is no `await` (`require-await`).
- Convert `ObjectId` to string before concatenation: `String(id)` or `id.toHexString()`.
- Narrow `unknown` with type guards before using values.
- Avoid `any` where feasible; prefer `unknown` + refinement.

---

## Pre-commit Hooks

We use **Husky** + **lint-staged** to ensure only clean code is committed.

```bash
npm run prepare  # installs Husky Git hooks
```

Create `.husky/pre-commit`:

```sh
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"
npx lint-staged
```

To bypass checks temporarily: `git commit -m "msg" --no-verify` (avoid unless necessary).

---

## Error Handling (Centralized)

Global setup in `main.ts`:

- `ValidationPipe` (whitelist, forbid non-whitelisted, transform)
- `TransformInterceptor` (unified success envelope)
- `HttpExceptionFilter` (unified error envelope)
- `MulterExceptionFilter` (friendly upload errors)

**Pattern:** Throw Nest exceptions in controllers/services (`BadRequestException`, etc.) and let the **global filters** format the response.

---

## API Docs (Swagger)

- UI: `/<API_PREFIX>/docs` (e.g. `/v1/docs`)
- JSON: `/<API_PREFIX>/docs-json`

`createDocument(app)` is called **after** the global prefix is set, so docs inherit the prefix.

---

## Environment Variables

Create `.env` (and optionally `.env.<env>`, `.env.aws`, `.env.azure`, etc.). Common keys:

```ini
# Core
PORT=4000
API_PREFIX=v1
NODE_ENV=development
PRIMARY_DOMAIN=localhost

# JWT
JWT_TOKEN_EXPIRESIN=1h
JWT_SECRET=dev-secret
REFRESH_TOKEN_EXPIRESIN=7d
REFRESH_TOKEN_SECRET=dev-refresh-secret

# OTP
OTP_TTL_SECONDS=900

# CORS (example app origin)
API_URL=http://localhost:4000

# Mail (Nodemailer)
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=user
SMTP_PASS=pass
SMTP_FROM_EMAIL=no-reply@example.com

# SendGrid (optional)
USE_SENDGRID=false
SENDGRID_API_KEY=
SENDGRID_FROM_EMAIL=no-reply@example.com

# Twilio (optional)
TWILIO_SID=
TWILIO_AUTH_TOKEN=
TWILIO_FROM_NUMBER=

# Platform selection
PLATFORM=AWS   # or AZURE
```

---

## Storage Platform Selection

`PlatformModule.register()` resolves one of the `StorageService` implementations at runtime:

- **AWS**: S3 (via `@aws-sdk/client-s3`)
  - `AWS_BUCKET_NAME`, `AWS_REGION`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_KEY`
- **Azure**: Blob Storage (via `@azure/storage-blob`)
  - `AZURE_STORAGE_ACCOUNT_NAME`, `AZURE_STORAGE_ACCOUNT_KEY`,  
    `AZURE_STORAGE_CONTAINER_NAME`, `AZURE_CONTAINER_URL`

Set `PLATFORM` to `AWS` or `AZURE` to pick the implementation.

---

## CORS

- Dev allowlist includes `http://localhost:3000`, `http://127.0.0.1:3000`.
- In production, update allowlist and **PRIMARY_DOMAIN** as required.
- `main.ts` trusts proxy (`app.set('trust proxy', 1)`) for correct HTTPS detection behind Nginx.

---

## Conventional Patterns

- **DTOs**: Use `class-validator` & `class-transformer`.
- **Services**: Keep transactional logic in services; controllers remain thin.
- **Mongoose**: Use `lean()` for read-only queries. Guard `unknown` types from `lean()` results.
- **Enums**: Prefer enum _keys_ for @ApiProperty/@IsIn and normalize values at boundaries.

---

## Troubleshooting

**ESLint flat config complains about `--ext`**  
Use `eslint .` or the provided scripts (`npm run lint`, `npm run lint:fix`).

**CORS errors in dev**  
Confirm the frontend origin is in the allowlist (see `main.ts`).

**ObjectId concatenation warnings**  
Use `String(id)` or `id.toHexString()` before concatenation or template literals.

**Missing model/provider errors**  
Ensure the module that **declares and exports** the provider (e.g., `MongooseModule.forFeature([...])`) is imported by the consumer module.

---

## ✨ Features

- 🔐 **Authentication**
  - Local login (email/mobile + password)
  - Google OAuth login
  - JWT access & refresh token support
  - OTP-based mobile/email verification
  - Secure cookie handling

- 🧠 **Platform-Agnostic Architecture**
  - Dynamic dependency loading from `package.{PLATFORM}.json`
  - Separate `.env.{PLATFORM}` for each cloud provider
  - Flexible integration with AWS S3, GCP, Azure, etc.

- ⚙️ **Production-Ready Dockerized Setup**
  - Multi-stage Dockerfile with optimized build
  - Automatic platform-specific dependency installation via `postinstall.js`
  - Docker Compose based environment provisioning

- 📝 **Robust API Layer**
  - Swagger auto-generated documentation
  - Guards, interceptors, and centralized response formatting
  - Modular and scalable project structure

---

## 📦 Project Structure

```
.
├── Dockerfile
├── docker-compose.yml
├── .env.production
├── .env.aws / .env.gcp / .env.azure
├── package.json
├── package.aws.json
├── postinstall.js
├── src/
└── ...
```

---

## 🚀 Setup & Run

### 1. Prerequisites

- Node.js 20+
- Docker & Docker Compose
- MongoDB (local or hosted)
- Cloud provider credentials (e.g., AWS keys for S3)

### 2. Environment Setup

Create a `.env.production` file with shared secrets and app configs.

Create a `.env.{PLATFORM}` for cloud-specific variables (e.g., `AWS_ACCESS_KEY`, etc.).

Example: `.env.aws`

```env
PLATFORM=AWS
AWS_REGION=ap-south-1
AWS_BUCKET=my-bucket
```

### 3. Build & Run (Docker)

```bash
docker compose build \
  --build-arg NODE_ENV=production \
  --build-arg PLATFORM=AWS \
  --build-arg LOWER_PLATFORM=aws \
  nestjs-app

docker compose up -d
```

---

## 🧪 Development

To run locally:

```bash
npm install
# PLATFORM is required to install specific deps
PLATFORM=AWS npm run postinstall:dev
npm run start:dev
```

> `postinstall:dev` will dynamically install `package.aws.json` and set up `.env.aws`.

---

## 📘 API Documentation

Available at:

```
http://localhost:4000/api/v1/docs
https://api.ethreeindia.com/v1/docs
```

Swagger is auto-configured and includes all routes, request/response types, guards, and error messages.

---

## 🛠 Deployment Notes

- Uses multi-stage Docker builds to keep production image minimal.
- `.env.production` is always loaded first, followed by `.env.{PLATFORM}`.
- All platform dependencies are merged before `npm install` via `postinstall.js`.

---

### 🔐 Hybrid Authentication: Web + Mobile Support

This application implements a **unified authentication strategy** that securely supports **both web and mobile platforms**.

---

## 🔐 Authentication Strategy (Web, Mobile, and Dev Support)

This NestJS backend is designed to support both secure production clients (web browsers) and development/mobile clients using JWT authentication.

### 🌐 1. Web Clients (Production)

- **Access and Refresh Tokens** are set as `HttpOnly` cookies:
  - `accessJwt`
  - `refreshJwt`
  - `loggedIn` (non-HttpOnly flag for client checks)
- Cookies are configured using a dynamic helper based on request origin and protocol:
  - Uses `SameSite=None` and `Secure=true` for HTTPS cross-origin
  - Uses `SameSite=Lax` and `Secure=false` for local development
- Cookie flags and expiry are set using a centralized utility:
  - `src/common/utils/cookie-options.util.ts`

> ✅ This setup protects against CSRF and ensures security for browser clients

---

### 📱 2. Mobile Apps or Dev Clients (Localhost Testing)

- **Tokens are returned in the response body** for clients to store (e.g., `sessionStorage`)
- Client manually attaches `Authorization: Bearer <token>` header
- The backend reads the token using `@UseGuards(JwtAuthGuard)`
- `withCredentials` is optional in this mode

> ✅ Ideal for mobile apps, Postman, or frontend development on localhost

---

### ⚙️ 3. Cookie Configuration Logic

- Automatically chooses `SameSite`, `Secure`, and `domain` flags
- Logic is based on:
  - `req.headers.origin`
  - `req.protocol` or `req.secure`
  - `NODE_ENV`
- Cookie settings are computed using `getCookieOptions()` helper:

````ts
// Example
res.cookie('accessJwt', token, getCookieOptions(req, {
  tokenType: 'accessJwt',
  configService,
}));


### ✅ Web Clients (Browser-based)

- Uses `res.cookie()` to set:
  - `accessJwt` and `refreshJwt` as `httpOnly` cookies
  - `loggedIn` as a frontend-visible cookie
- Cookies are automatically attached to requests by the browser
- Secure cookie flags (`SameSite`, `Secure`, `HttpOnly`) are applied dynamically based on environment
- Access is controlled via server-side `middleware.ts` in Next.js

---

| Feature                        | Web Browser Clients    | Mobile App Clients     |
| ------------------------------ | ---------------------- | ---------------------- |
| Token Transport Method         | `httpOnly` cookies     | `Authorization` header |
| Session Storage                | Secure browser cookies | Secure mobile storage  |
| Server-side Token Validation   | Cookie-based auth      | Header-based auth      |
| Middleware/Guard Compatibility | ✅ Supported            | ✅ Supported            |


## 🍪 Cookie Management Strategy

This application implements a **secure, environment-aware, cross-origin compatible cookie-based authentication system** using `accessJwt`, `refreshJwt`, and `loggedIn` cookies.

---

### ✅ Key Features

- **Environment-aware behavior**: Cookie flags like `SameSite` and `secure` adapt dynamically based on:
  - `NODE_ENV` (`development`, `staging`, `production`)
  - Request `origin` (`localhost`, subdomain, etc.)
- **Cross-origin support**:
  - Local frontend → remote backend (e.g., EC2 API)
  - Fully local dev (localhost:3000 → localhost:4000)
  - Production on same domain or subdomains
- **JWT-based security**:
  - Tokens are signed and stored securely in `httpOnly` cookies
  - Cookie expiration matches JWT expiration via `.env`

---

### 🔐 Cookie Overview

| Cookie Name | Purpose               | HTTP-Only | Secure | SameSite | Expiration Source             |
|-------------|------------------------|-----------|--------|----------|-------------------------------|
| `accessJwt` | Short-lived JWT        | ✅        | ✅     | `None` / `Strict` / `Lax` | `JWT_TOKEN_EXPIRESIN`         |
| `refreshJwt`| Long-lived refresh JWT | ✅        | ✅     | `None` / `Strict` / `Lax` | `JWT_REFRESH_TOKEN_EXPIRESIN` |
| `loggedIn`  | Frontend-only login flag | ❌        | ✅     | `None` / `Strict` / `Lax` | Fixed                         |

---

### 🧠 Smart Detection Logic

The system includes a **reusable cookie configuration helper** that dynamically determines:

- ✅ Whether the frontend is running **locally or remotely**
- ✅ Whether cookies should be sent **cross-site or same-site**
- ✅ What flags to apply:
  - `SameSite`: `'lax'`, `'strict'`, or `'none'`
  - `Secure`: `true` or `false`
  - `Domain`: conditionally set in production for subdomain support

This logic is implemented in a centralized utility:

> 📁 [`src/common/utils/cookie-options.util.ts`](./src/common/utils/cookie-options.util.ts)

---

### 💡 Benefits

- ✅ **Works reliably across environments**
  Supports full local development, staging, and production setups.

- ✅ **Enhances security by design**
  Prevents CSRF, XSS, and MITM attacks using proper cookie flags.

- ✅ **Keeps configuration centralized**
  All cookie behavior is determined from a single helper, reducing duplicated logic.

- ✅ **Ensures consistency with JWT**
  Cookie expiration is auto-aligned with JWT expiration using `ms()` + `.env` values.

---

### 🔧 Environment Detection Logic

The utility evaluates:


const origin = req.headers.origin || '';
const isSecureRequest = req.protocol === 'https' || req.secure;
const isLocalFrontend = origin.includes('localhost');
const isSameOriginFrontend = origin.endsWith('yourdomain.com');

| Env         | Secure | SameSite | Domain           |
| ----------- | ------ | -------- | ---------------- |
| Local dev   | ❌      | `lax`    | (not set)        |
| Local → EC2 | ✅      | `none`   | (not set)        |
| Production  | ✅      | `strict` | `yourdomain.com` |




# HTTP Caching (Valkey/Redis) for NestJS

# HTTP Response Cache (NestJS) — Valkey/Redis/ioredis + ElastiCache

This project implements a **platform‑aware**, production‑friendly HTTP response cache for NestJS using **Valkey/Redis + ioredis**. The cache is activated automatically on the **AWS** platform (via `PlatformModule`) and is a **no‑op elsewhere**. The app also runs fine with **no Redis configured**.

> This README combines **framework integration** (decorators, interceptor, admin API/UI) **and** complete **backend setup scenarios** (Redis/Valkey containers, host service, and AWS ElastiCache with/without cluster, plus localhost tunneling).

---

## Table of contents
1. [What you get](#-what-you-get)
2. [Architecture overview](#-architecture-overview)
3. [Environment quickstarts](#-environment-quickstarts)
4. [Setup steps (already wired in)](#-setup-steps-already-wired-in)
5. [Using the cache in controllers](#-using-the-cache-in-controllers)
6. [Invalidation after writes](#-invalidation-after-writes)
7. [Admin cache API & Web UI](#-admin-cache-api--web-ui)
8. [Observability](#-observability)
9. [Safety notes](#-safety-notes)
10. [Troubleshooting](#-troubleshooting)
11. [Backends & deployment scenarios](#-backends--deployment-scenarios)
12. [Health checks & one‑liners](#-health-checks--one-liners)
13. [Dockerfile / image tips](#-dockerfile--image-tips)
14. [Appendix A — Env reference](#appendix-a--env-reference)
15. [Appendix B — URL‑encoding cheatsheet](#appendix-b--url-encoding-cheatsheet)

---

## ✨ What you get

- **Global cache interceptor** that:
  - Caches **GET** responses only
  - Skips caching when `Authorization` header is present **unless you allow it**
  - Supports **per‑route TTL** and **custom cache keys**
  - Uses **normalized URL** (`path + sorted query`) when you don’t set a custom key
  - Serializes values as **plain JSON** (no Mongoose internals like `$__` / `_doc`)
  - Adds: `X-Cache`, `X-Cache-TTL`, `X-Cache-Key` response headers

- **Decorators** to control caching:
  - `@HttpCacheTTL(ms)`
  - `@HttpCacheKey('my:key')`
  - `@AllowAuthCache()` (cache even with `Authorization` header)
  - `@NoHttpCache()` (opt‑out when needed)

- **Simple invalidation helpers** to call after writes

- **Admin cache API** + a lightweight **web UI** to inspect/purge keys

---

## 🧩 Architecture Overview

- `PlatformModule` (marked **@Global**) provides:
  - `REDIS` client (on AWS only) via **ioredis**
  - `CacheService` wrapper for get/set/del/PTTL/key‑building
  - `NoopCacheService` on non‑AWS platforms

- `HttpCacheInterceptor` (global `APP_INTERCEPTOR`):
  - Reads route metadata (decorators) via `Reflector`
  - Gets/sets cache entries via `CacheService`

- `StartupStatusService`:
  - Logs Valkey and Mongo connectivity on app boot

---

## ⚙️ Environment Quickstarts

App runs even with **no cache**; enable a backend by environment variables.

### Local EC2 / same‑box Valkey (dev‑friendly)
```env
PLATFORM=AWS
CACHE_PREFIX=e3-admin
HTTP_CACHE_TTL_MS=60000

VALKEY_HOST=127.0.0.1
VALKEY_PORT=6379
VALKEY_USERNAME=app
VALKEY_PASSWORD="your-strong-password"
REDIS_STARTUP_TIMEOUT_MS=5000
```

### ElastiCache with TLS (cluster or single node)
```env
PLATFORM=AWS
VALKEY_HOST=mycache.xxxxxx.ap-south-1.cache.amazonaws.com
VALKEY_PORT=6379
VALKEY_USERNAME=app          # if ACL enabled
VALKEY_PASSWORD="your-strong-password"
VALKEY_TLS=true
```

> Prefer split vars (`HOST/PORT/USERNAME/PASSWORD/TLS`) rather than a single URL string. If you **must** use a URL, encode the password and use `rediss://` for TLS.

**Full scenario matrix and docker/tunnel recipes** live below in [Backends & deployment scenarios](#-backends--deployment-scenarios).

---

## 🚀 Setup Steps (already wired in)

1) **Global platform module** (exports `CacheService`, `REDIS` on AWS):
```ts
// app.module.ts
@Module({
  imports: [
    PlatformModule.register(),
    // ...
  ],
})
export class AppModule {}
```

2) **Global cache interceptor**:
```ts
providers: [
  { provide: APP_INTERCEPTOR, useClass: HttpCacheInterceptor },
]
```

3) **CORS headers** (to read cache headers in browsers):
```ts
app.enableCors({
  origin: true,
  credentials: true,
  exposedHeaders: ['X-Cache', 'X-Cache-Key', 'X-Cache-TTL'],
});
```

---

## 🧪 Using the Cache in Controllers

```ts
// events.controller.ts
@HttpCacheTTL(120_000)                  // 2 minutes
@HttpCacheKey('events:list:v2')         // stable key for easy invalidation
@AllowAuthCache()                       // safe because response is not user-specific
@Get('/events')
async findAll() {
  return this.eventsService.findAll();  // return plain JSON (use .lean() on reads)
}

@HttpCacheTTL(60_000)                   // vary by query automatically
@AllowAuthCache()
@Get('/events/eventwithqr')
async findAllEventsWithQrCode(@Query('limit') limit?: number, @Query('offset') offset?: number) {
  return this.eventsService.findAllEventsWithQrCode(limit ?? 50, offset ?? 0);
}

@HttpCacheTTL(300_000)                  // vary by URL param automatically
@AllowAuthCache()
@Get('/events/:id')
async findOne(@Param('id') id: string) {
  return this.eventsService.findOne(new Types.ObjectId(id));
}
```
> For POST/PUT/PATCH/DELETE, caching is not applied by the interceptor.

---

## 🧼 Invalidation after Writes

```ts
@Injectable()
export class EventsService {
  constructor(private readonly cache: CacheService /* ... */) {}

  private listKey() {
    return this.cache.key('http', 'GET', 'events:list:v2'); // match controller
  }
  private byIdKey(id: string | Types.ObjectId) {
    return this.cache.key('http', 'GET', `/events/${String(id)}`);
  }
  private async invalidateAfterWrite(id?: string | Types.ObjectId) {
    try { await this.cache.del(this.listKey()); } catch {}
    if (id) { try { await this.cache.del(this.byIdKey(id)); } catch {} }
    // eventwithqr relies on short TTL; add prefix purge if ever needed.
  }

  async create(dto: RequestCreateEventDto) {
    const saved = await this.eventModel.create(dto);
    await this.invalidateAfterWrite(saved.id);
    return saved.toObject();
  }

  async update(id: string, dto: RequestUpdateEventDto) {
    const updated = await this.eventModel.findByIdAndUpdate(id, dto, { new: true }).lean<Events>().exec();
    if (!updated) throw new NotFoundException('Event not found');
    await this.invalidateAfterWrite(id);
    return updated;
  }
}
```

> **Reads** should use `.lean()` for plain JSON. **Writes** should return `.toObject()` for hydrated docs.

---

## 🧰 Admin Cache API & Web UI

**BasicAuth‑protected endpoints:**
- `GET /admin/cache/info` — ping + dbsize + prefix
- `GET /admin/cache/keys?prefix=...&cursor=0&count=100` — list keys via SCAN
- `GET /admin/cache/get?key=...` — value + type + TTL
- `POST /admin/cache/purge` — delete by prefix (supports `dryRun`, `batch`, `limit`)

Example purge (dry‑run):
```bash
curl -u admin:secret123 -H "Content-Type: application/json" \
  -d '{"prefix":"e3-admin:development:http:GET:*","dryRun":true}' \
  http://localhost:4000/admin/cache/purge
```

**Admin Cache Web UI** (zero‑dependency HTML) served from `public/admin-cache.html`:
- Open: `http://localhost:4000/static/admin-cache.html`
- Enter BasicAuth credentials and optional API base (e.g., `/api`)
- Browse keys, inspect values, and purge by prefix (with dry‑run)

> Keep it admin‑only (BasicAuth + network ACL/VPN).

---

## 🔎 Observability

- **Response headers**:
  - `X-Cache: HIT | MISS`
  - `X-Cache-Key: <key>`
  - `X-Cache-TTL: <ms>` (remaining TTL for HITs; configured TTL for MISS)

- **Startup logs**:
  - `Valkey: CONNECTED (pong=PONG)` or `Valkey: DOWN (...)`
  - `MongoDB: CONNECTED (db=...)`

- **CLI checks**:
```bash
valkey-cli -h 127.0.0.1 -p 6379 --user app --askpass ping
valkey-cli -h 127.0.0.1 -p 6379 --user app --askpass --scan --pattern 'e3-admin:development:*' | head
```

---

## 🛡️ Safety Notes

- Prefer **SSH/SSM tunnel** over exposing Redis publicly.
- Never use `KEYS` in production—use `SCAN` (the Admin API does).
- Do **not** cache user‑specific responses unless you explicitly mark the route with `@AllowAuthCache()` and you are sure it’s safe.
- If changing serialization (hydrated → plain objects), **purge old cache** or bump key version (e.g., `events:list:v1` → `v2`).

---

## 🧯 Troubleshooting

- **`getaddrinfo ENOTFOUND mycache.xxxxxx...`** — You left a placeholder in `VALKEY_HOST/URL`. Use the real host or localhost + NAT map for tunnels.\n
- **TLS errors to `127.0.0.1`** — Don’t set `VALKEY_TLS=true` for localhost unless you also set `SNI` and tunnel.\n
- **Password in URL fails** — URL‑encode it, or prefer split env vars (easier).\n
- **`$__` / `_doc` in responses** — Purge old cache; interceptor now serializes to plain JSON.\n
- **`UnknownDependenciesException` for `CacheService`** — Ensure `PlatformModule` is `@Global()` and imported once in `AppModule`.\n
- **ioredis cluster: `Failed to refresh slots cache.`** — Ensure `CLUSTER SLOTS` works from the box; in code use `dnsLookup: (addr, cb) => cb(null, addr)` and `tls: { servername: <clustercfg host> }`; pass creds only when present.\n
- **`ERR_TLS_CERT_ALTNAME_INVALID`** — You connected to IP (e.g., 127.0.0.1) but cert is for `*.cache.amazonaws.com`. Set `SNI` (see scenarios below).\n
- **`bind [127.0.0.1]:6379: Address already in use`** — Another process holds 6379. Free it or use alternate ports.\n

---

## 🧱 Backends & Deployment Scenarios

### Quick matrix
| Scenario | Minimal env | How it connects |
|---|---|---|
| **No cache (noop)** | `REDIS_BACKEND=none` | App works without Redis |
| **Redis container (local/EC2)** | `REDIS_BACKEND=standalone` + `REDIS_URL=redis://:URLENCODED_PASS@HOST:PORT/DB` | Single node |
| **Valkey container (local/EC2)** | Same as above, or `redis://user:URLENCODED_PASS@...` | Single node |
| **ElastiCache (Cluster Enabled)** | `REDIS_BACKEND=elasticache` + `VALKEY_CLUSTER=true` + `VALKEY_HOST=clustercfg.*` + `VALKEY_PORT=6379` + `VALKEY_TLS=true` [+ ACL] | **Cluster + TLS** |
| **ElastiCache (Cluster Disabled)** | `REDIS_BACKEND=elasticache` + `VALKEY_URL=rediss://...` **or** split host/port + `VALKEY_TLS=true` | Single node + TLS |
| **Local tunnel to ElastiCache** | Cluster vars + `VALKEY_HOST=127.0.0.1` + `VALKEY_PORT=<local>` + `VALKEY_NAT_MAP=remote=127.0.0.1:<local>` + `VALKEY_SNI=<clustercfg>` | Cluster over SSH/SSM tunnel |

> URL‑encode password in URLs (`/`→`%2F`, `=`→`%3D`, `@`→`%40`).

### Redis container — local or EC2
See **docker-compose.redis.yml** and `.env.redis` example in the quickstarts above.

### Valkey container (with ACL)
Mount `users.acl`, start server with `--aclfile`, and connect via `redis://user:URLENCODED_PASS@...`.

### Existing Valkey on host (not container)
Ensure it binds to an interface your app (possibly in Docker) can reach. From a Linux container to host service, use `172.17.0.1`.

### ElastiCache — Cluster Mode Enabled
- Env: `VALKEY_CLUSTER=true`, `VALKEY_HOST=clustercfg.*`, `VALKEY_TLS=true`, optional `VALKEY_USERNAME/PASSWORD` (if ACL enabled).
- **Client code must** keep hostnames and send SNI:
  `dnsLookup: (addr, cb) => cb(null, addr)` and `tls: { servername: process.env.VALKEY_SNI || process.env.VALKEY_HOST }`.
- Install CA certs in the runtime image.

### ElastiCache — Cluster Mode Disabled
Use `VALKEY_URL=rediss://...` (URL‑encoded pass) **or** split fields (`VALKEY_HOST/PORT/TLS` + optional `USERNAME/PASSWORD`).

### Localhost tunneling to ElastiCache (dev)
Create local forwards for the config endpoint (seed) and all node endpoints, then either:
- **Use NAT map**: `VALKEY_HOST=127.0.0.1`, `VALKEY_PORT=<seed>`, `VALKEY_SNI=<clustercfg>`, `VALKEY_NAT_MAP='node-1:6379=127.0.0.1:<p1>,node-2:6379=127.0.0.1:<p2>'`
- **/etc/hosts approach**: map each AWS hostname to 127.0.0.1 and forward 6379 per hostname (more setup).

---

## 🧪 Health checks & one‑liners

```bash
# Who holds 6379 locally
sudo ss -ltnp | grep ':6379' || lsof -iTCP:6379 -sTCP:LISTEN

# Test ElastiCache from EC2
redis-cli --tls -h clustercfg.<cluster>.<az>.cache.amazonaws.com -p 6379 -c CLUSTER INFO

# Inside container, quick redis-cli install & ping (Alpine)
apk add --no-cache redis >/dev/null 2>&1 || true; \
redis-cli --tls -h "$VALKEY_HOST" -p "$VALKEY_PORT" -c ping
```

## 🐳 Dockerfile / image tips

- **Do not copy `.env.*` files into the image**. Provide env at runtime via Compose/CI.
- Install CA certs in runtime image for TLS:
  ```dockerfile
  RUN apk add --no-cache ca-certificates && update-ca-certificates
  ```
- Bind local dev containers to `127.0.0.1` only; don’t expose on public interfaces.

---

## Appendix A — Env reference

**Backend selector**
```env
# none | standalone | elasticache | auto
REDIS_BACKEND=elasticache
```

**Standalone (single node)**
```env
REDIS_URL=redis://[:user@]URLENCODED_PASS@HOST:PORT/DB
REDIS_KEY_PREFIX=e3:prod:
REDIS_STARTUP_TIMEOUT_MS=15000
```

**ElastiCache (cluster)**
```env
VALKEY_CLUSTER=true
VALKEY_HOST=clustercfg.<cluster>.<az>.cache.amazonaws.com
VALKEY_PORT=6379
VALKEY_TLS=true
# ACL (if enabled)
VALKEY_USERNAME=app
VALKEY_PASSWORD=PlainPassword   # RAW (not URL-encoded)
# Dev tunneling helpers
VALKEY_SNI=clustercfg.<cluster>.<az>.cache.amazonaws.com
VALKEY_NAT_MAP=node-0001.<cluster>.<az>.cache.amazonaws.com:6379=127.0.0.1:16380,node-0002...=127.0.0.1:16381
```

**ElastiCache (single node)**
```env
# Option A: URL
VALKEY_URL=rediss://app:URLENCODED_PASS@PRIMARY-ENDPOINT.cache.amazonaws.com:6379/0
VALKEY_TLS=true
# Option B: split
VALKEY_HOST=PRIMARY-ENDPOINT.cache.amazonaws.com
VALKEY_PORT=6379
VALKEY_TLS=true
VALKEY_USERNAME=app        # if ACL
VALKEY_PASSWORD=PlainPassword
```

---

## Appendix B — URL‑encoding cheatsheet

| Char | Encoded |
|---|---|
| `/` | `%2F` |
| `@` | `%40` |
| `:` | `%3A` |
| `?` | `%3F` |
| `#` | `%23` |
| `=` | `%3D` |
| `%` | `%25` |

---

**That’s it!** Choose your backend, wire the env, and use the decorators to control cache behavior. The admin API/UI and troubleshooting section should cover the rest. Happy caching 🚀



# Secrets & Configuration Guide

This project supports **provider‑agnostic secrets preloading** (AWS / GCP / Azure) plus sane local fallbacks. Secrets are fetched **before Nest boots** so every module can safely read `process.env.*`.

> TL;DR: In dev, set `SECRET_PROVIDER=local` and use `.env` files. In cloud, set `SECRET_PROVIDER=aws|gcp|azure` and store your real values in the provider’s secret manager. Keys from **common** secret are loaded first, then **platform** secret overrides.

---

## 1) How it works

### Boot sequence
1. **dotenv** loads base files (`.env.<platform>`, `.env.<env>`, then `.env`).
2. **Secret preloader** (`src/bootstrap/preload-secrets.ts`) runs in `main.ts` **before** `NestFactory.create()`:
   - Decides provider via `SECRET_PROVIDER` (or `PLATFORM` fallback).
   - Resolves which secret names to fetch (common **then** platform).
   - Fetches secrets via provider service (`AwsSecretsService` / `GcpSecretsService` / `AzureSecretsService`).
   - Merges keys into `process.env` **without** overwriting values that are already set.

3. **Nest** starts. Modules like `MongooseModule.forRootAsync()` can now depend on variables like `MONGO_URI`.

### Common vs Platform secrets
- **Common**: `<base>/<stage>` (e.g., `e3-admin/dev`) – shared across all platforms.
- **Platform**: `<base>/<stage>/<provider>` (e.g., `e3-admin/dev/aws`) – keys that override or add provider‑specific settings.

**Order matters**: common → platform (platform wins on key collision).

---

## 2) Environment variables (control plane)

| Variable | Purpose | Typical values |
|---|---|---|
| `SECRET_PROVIDER` | Selects how to fetch secrets | `local`, `aws`, `gcp`, `azure` |
| `SECRETS_REQUIRED` | Fail fast if fetching secrets fails | `true`/`false` |
| `SECRET_BASE` | Base secret path slug | e.g. `e3-admin` |
| `SECRET_STAGE` | Stage used in default naming | `dev`, `staging`, `prod` |
| `SECRET_NAME` | Explicit common secret name (overrides defaults) | e.g. `e3-admin/dev` |
| `SECRET_NAME_PLATFORM` | Explicit platform secret name | e.g. `e3-admin/dev/aws` |
| `SECRET_NAMES` | Comma‑separated list (generic override) | `name1,name2` |
| `AWS_SECRET_NAMES` | AWS-only list, wins over generic | `name1,name2` |
| `GCP_SECRET_NAMES` / `GCP_SECRET_IDS` | GCP-only names or full resource IDs |  |
| `AZURE_SECRET_NAMES` | Azure-only list |  |
| `PLATFORM` | Enables platform wiring (S3/Redis etc.) | `AWS`, `GCP`, `AZURE` |
| `REDIS_URL` | Enable Redis wiring in `PlatformModule` | e.g. `redis://127.0.0.1:6379/0` |
| `MONGO_URI` | MongoDB connection string | `mongodb+srv://...` |
| `SECRETS_ENV_DUMP` | (Debug) dump all env keys after merge | `true`/`false` |
| `SECRETS_LOG_VALUES` | (Debug) show values instead of masking | `true`/`false` |

> **Note:** `SECRETS_REQUIRED` only controls _what happens if fetching fails_. It does **not** decide whether we try to fetch. Use `SECRET_PROVIDER=local` to skip remote calls.

---

## 3) Secret naming & resolution

If you **don’t** pass any overrides (`SECRET_NAME*`, `*SECRET_NAMES*`), we build names from:
```
<base>/<stage>              # common   (defaultCommonName)
<base>/<stage>/<provider>   # platform (defaultPlatformName)
```
Where:
- `<base>` is from `SECRET_BASE`, else from `package.json` name, else folder name.
- `<stage>` is from `SECRET_STAGE` (maps `production` → `prod`, `staging` aliases → `staging`, else `dev`).
- `<provider>` is `aws` / `gcp` / `azure`.

Overrides (highest to lowest priority):
1. Provider‑specific lists: `AWS_SECRET_NAMES`, `GCP_SECRET_IDS|NAMES`, `AZURE_SECRET_NAMES`
2. Generic list: `SECRET_NAMES`
3. Pair: `SECRET_NAME` (+ `SECRET_NAME_PLATFORM` or default platform name)
4. Defaults (as above)

---

## 4) AWS setup (example provider)

### 4.1 Create secrets
Create **two** Secrets Manager entries (Regions must match your app’s `AWS_REGION`):
- Common: `e3-admin/dev`
- Platform: `e3-admin/dev/aws`

> For production, use `e3-admin/prod` and `e3-admin/prod/aws` (or your chosen names).

### 4.2 Payload formats
The loader accepts either:
- **JSON** (no comments – standard JSON):
  ```json
  {
    "MONGO_URI": "mongodb+srv://user:pass@cluster/db",
    "JWT_TOKEN_SECRET": "a_very_long_random_string_at_least_32_chars",
    "REDIS_URL": "redis://:password@host:6379/0",
    "SENDGRID_API_KEY": "SG.xxxxx"
  }
  ```
- **KEY=VALUE lines** (dotenv style):
  ```
  MONGO_URI=mongodb+srv://user:pass@cluster/db
  JWT_TOKEN_SECRET=a_very_long_random_string_at_least_32_chars
  REDIS_URL=redis://127.0.0.1:6379/0
  ```

> If you want to include inline documentation, use the description field in Secrets Manager, or maintain a checked‑in `secrets.schema.md`. The loader purposefully ignores keys that start with `_` or end with `__comment` if you decide to store metadata alongside values (only applicable for JSON payloads you control programmatically).

### 4.3 IAM policy (minimum to read those secrets)
Attach a policy to the IAM user/role the app runs as:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadProjectSecrets",
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue",
        "secretsmanager:DescribeSecret"
      ],
      "Resource": [
        "arn:aws:secretsmanager:*:*:secret:e3-admin/*"
      ]
    }
  ]
}
```

### 4.4 Credentials
The AWS SDK resolves creds in this order (simplified):
- `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY` (+ `AWS_SESSION_TOKEN` if STS)
- Shared profile `~/.aws/credentials` (via `AWS_PROFILE`)
- EC2/ECS role

Set **`AWS_REGION`** explicitly (e.g., `ap-south-1`).

---

## 5) Local vs Cloud recipes

### Local development (skip remote)
`.env`:
```ini
NODE_ENV=development
PLATFORM=AWS              # optional: to keep AWS-specific wiring like S3
SECRET_PROVIDER=local     # <— do NOT call Secrets Manager
SECRETS_REQUIRED=false    # irrelevant in local mode

# your local values
MONGO_URI=mongodb://127.0.0.1:27017/mydb
REDIS_URL=redis://127.0.0.1:6379/0
JWT_TOKEN_SECRET=dev_only_change_me
```

### Cloud (use AWS secrets & fail fast)
`.env` (or environment variables in your runner):
```ini
NODE_ENV=production
PLATFORM=AWS
SECRET_PROVIDER=aws       # <— call AWS Secrets Manager
SECRETS_REQUIRED=true     # crash if secrets cannot be loaded
SECRET_BASE=e3-admin
SECRET_STAGE=prod
AWS_REGION=ap-south-1
# (no MONGO_URI here; it will come from the secrets)
```

Optionally pin names explicitly:
```ini
AWS_SECRET_NAMES=e3-admin/prod,e3-admin/prod/aws
```

---

## 6) Redis behavior (PlatformModule)

- Redis is enabled when **`REDIS_URL`** exists.
- In **AWS dev** without a `REDIS_URL`, the module falls back to `redis://127.0.0.1:6379/0` so your local Valkey/Redis works out‑of‑the‑box.
- On startup, `StartupStatusService` logs:
  - `Valkey: CONNECTED (pong=PONG)` when OK
  - `Valkey: DISABLED (no REDIS_URL)` or `(platform != AWS)` when disabled
  - `Valkey: DOWN (…err…)` if cannot connect

---

## 7) Key files

- `src/bootstrap/preload-secrets.ts` — provider‑agnostic preloader used in **`main.ts`**.
- `src/platform/platform.module.ts` — loads dotenv files, provides `StorageService`, optional Redis wiring, and exposes the `SECRETS` service for runtime use.
- `src/platform/aws/aws-secrets.service.ts` (and GCP/Azure equivalents) — fetch & parse.
- `src/platform/secrets-parse.ts` — tolerant parser for JSON / KEY=VALUE.
- `src/infra/health/startup-status.service.ts` — logs Redis + Mongo status on boot.

---

## 8) Verification checklist

1. Set `SECRET_PROVIDER=local` and run — app should boot using only `.env` values.
2. Switch to cloud mode:
   - `SECRET_PROVIDER=aws`
   - Ensure AWS creds + `AWS_REGION` are available.
   - Create `e3-admin/dev` and `e3-admin/dev/aws` with required keys, especially `MONGO_URI` and `JWT_TOKEN_SECRET`.
3. Start the app — you should see:
   - “Fetched/merged N keys” (if you enable the optional logging helper) or
   - MongoDB connected and, if applicable, Valkey connected.
4. Remove local `MONGO_URI` from `.env` and confirm it still works (now sourced from secrets).

---

## 9) Common errors & fixes

- **`Could not load credentials from any providers`**
  AWS creds not found. Export `AWS_ACCESS_KEY_ID` / `AWS_SECRET_KEY` or set a profile. Set `AWS_REGION`.

- **`AccessDeniedException: not authorized to perform GetSecretValue`**
  IAM policy missing `secretsmanager:GetSecretValue` for your secret ARNs. See policy above.

- **`ResourceNotFoundException: Secrets Manager can't find the specified secret`**
  Name/Region mismatch. Confirm secret names (`AWS_SECRET_NAMES` or defaults from `SECRET_BASE`/`SECRET_STAGE`) and region.

- **`MONGO_URI is missing` or Mongoose says URI is `undefined`**
  The value didn’t make it into `process.env`. Check that the secret payload contains `MONGO_URI` *and* that secrets were fetched (provider not `local`, credentials OK, names correct).

- **Valkey/Redis keeps reconnecting**
  Set `REDIS_URL` or run a local Redis in dev. The module will log the status line.

---

## 10) Security tips

- Never commit real secrets into Git.
- In prod, set `SECRETS_REQUIRED=true` so misconfigurations fail fast.
- Rotate `JWT_TOKEN_SECRET` regularly and use strong random values (≥ 32 chars).
- Limit IAM permissions to the precise secrets your app needs.

---

## 11) Quick start (commands)

```bash
# install deps
npm i

# local dev using only .env
export SECRET_PROVIDER=local
npm run start:dev

# cloud-mode locally (reads from AWS)
export SECRET_PROVIDER=aws
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_KEY=...
export AWS_REGION=ap-south-1
npm run start:dev
```

---

# Deployment: Blue/Green CI/CD to EC2 (NestJS)

This repository is configured for **zero‑downtime Blue/Green deployments** to an **EC2** host using **GitHub Actions**, **Docker Compose**, and **Nginx**.

---

## Quick Facts

- **Workflow**: `.github/workflows/deploy-bluegreen.yml`
- **Compose**: `docker-compose-bg.yml` (no `container_name`, exposes container port 4000 via color‑specific host port)
- **Service name**: `nestjs-app`
- **Domain**: `api.ethreeindia.com`
- **Health endpoint**: `GET /v1/health`
- **Colors / Ports**: **BLUE → 4001**, **GREEN → 4002**
- **Active color file (server)**: `/srv/app/.active_color_api`
- **Nginx upstreams (server)**:
  - `/etc/nginx/upstreams/api-blue.conf`  → `upstream api_upstream { server 127.0.0.1:4001; }`
  - `/etc/nginx/upstreams/api-green.conf` → `upstream api_upstream { server 127.0.0.1:4002; }`
  - `/etc/nginx/upstreams/api-active.conf` (symlink to one of the above)
  - `/etc/nginx/conf.d/bg-config.conf` includes the **active** upstream

---

## 1) One‑time Server Setup (EC2)

> Run these **on the EC2 host** (Ubuntu) once.

### 1.1 Create upstreams and active symlink
```bash
sudo mkdir -p /etc/nginx/upstreams

# BLUE → 4001
sudo tee /etc/nginx/upstreams/api-blue.conf >/dev/null <<'NGX'
upstream api_upstream {
  server 127.0.0.1:4001;
  keepalive 64;
}
NGX

# GREEN → 4002
sudo tee /etc/nginx/upstreams/api-green.conf >/dev/null <<'NGX'
upstream api_upstream {
  server 127.0.0.1:4002;
  keepalive 64;
}
NGX

# Set active = BLUE initially
sudo ln -sfn /etc/nginx/upstreams/api-blue.conf /etc/nginx/upstreams/api-active.conf
````

### 1.2 Load the active upstream at the **http** level

Create a tiny include file that nginx.conf will pick up (Ubuntu loads `/etc/nginx/conf.d/*.conf` from inside the `http {}` block by default):

```bash
sudo tee /etc/nginx/conf.d/bg-config.conf >/dev/null <<'NGX'
include /etc/nginx/upstreams/api-active.conf;
NGX
```

### 1.3 Point your API server block to the upstream

In the `server {}` for `api.ethreeindia.com`, ensure `proxy_pass` uses the upstream name:

```nginx
location / {
  proxy_pass http://api_upstream;   # <— not a fixed port
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
  proxy_cache_bypass $http_upgrade;
}
```

Then test & reload:

```bash
sudo nginx -t && sudo nginx -s reload
```

---

## 2) Compose (color‑ready)

`docker-compose-bg.yml` must **not** set `container_name` and must bind host port by color:

```yaml
services:
  nestjs-app:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        NODE_ENV: ${NODE_ENV:-production}
        PLATFORM: ${PLATFORM:-AWS}
        LOWER_PLATFORM: ${LOWER_PLATFORM:-aws}
    env_file:
      - .env.production
    ports:
      - '127.0.0.1:${HOST_PORT:-4000}:4000' # BLUE=4001, GREEN=4002
    environment:
      - NODE_ENV=production
      - PORT=4000
      - HOST=0.0.0.0
    healthcheck:
      test: ['CMD-SHELL', 'curl -fsS http://127.0.0.1:4000/v1/health || exit 1']
      interval: 30s
      timeout: 10s
      retries: 3
    restart: unless-stopped
```

---

## 3) GitHub Actions Workflow

**File**: `.github/workflows/deploy-bluegreen.yml`  
**Triggers**: push to `main` + manual dispatch

What it does:

1. SSH to EC2, clone/pull repo (`bhgatravi/nestjs-proj`).
2. Loads `.env.production` on the server for build args and env.
3. Builds the **inactive** color with BuildKit, sets:
   - `COMPOSE_PROJECT_NAME=nestjs-<color>`
   - `HOST_PORT=<4001|4002>`
4. Starts the target color and health‑checks it directly on `127.0.0.1:<port>/v1/health`.
5. Flips Nginx symlink `/etc/nginx/upstreams/api-active.conf` to the target color, reloads Nginx.
6. Verifies **public** health at `http://api.ethreeindia.com/v1/health`.
7. Stops the old color stack.

> The workflow includes post‑flip verifications so a misconfiguration (wrong symlink, wrong `proxy_pass`, failed reload) fails the job.

---

## 4) GitHub Secrets

Add these in **Repository → Settings → Secrets and variables → Actions**:

- `EC2_HOST` → public IP / DNS of your EC2
- `EC2_USER` → SSH user (e.g., `ubuntu`)
- `EC2_SSH_KEY` → **raw PEM** contents (multi‑line, begin/end lines included)
- `GH_PAT` → fine‑grained token with **Contents: Read** for this private repo

---

## 5) Environment File on Server

Create `/srv/app/nestjs-proj/.env.production` (used by Compose & as build args). Example:

```
NODE_ENV=production
PLATFORM=AWS
LOWER_PLATFORM=aws
# add your app config/secrets here (do NOT commit this file)
```

---

## 6) How to Deploy

- **Automatic**: push to `main` → workflow runs and flips colors if healthy.
- **Manual**: Actions tab → **Deploy NestJS to EC2 (Blue/Green + GH_PAT + Health + BuildKit)** → **Run workflow**.

On success, traffic at `http://api.ethreeindia.com` is served by the **new** color.

---

## 7) Rollback (instant)

Flip the symlink back and reload Nginx:

```bash
# To BLUE (4001)
sudo ln -sfn /etc/nginx/upstreams/api-blue.conf  /etc/nginx/upstreams/api-active.conf
sudo nginx -t && sudo nginx -s reload

# To GREEN (4002)
sudo ln -sfn /etc/nginx/upstreams/api-green.conf /etc/nginx/upstreams/api-active.conf
sudo nginx -t && sudo nginx -s reload
```

---

## 8) Troubleshooting

- **502** at `/v1/health` or `/v1/docs`:

  ```bash
  # Which color is active?
  readlink -f /etc/nginx/upstreams/api-active.conf

  # Direct health on each color
  curl -i http://127.0.0.1:4001/v1/health
  curl -i http://127.0.0.1:4002/v1/health

  # Confirm server block uses the upstream name (not fixed port)
  sudo nginx -T | sed -n '/server_name api.ethreeindia.com/,/}/p' | grep proxy_pass

  # Logs
  docker compose -p nestjs-blue  -f docker-compose-bg.yml logs --tail=200 nestjs-app
  docker compose -p nestjs-green -f docker-compose-bg.yml logs --tail=200 nestjs-app
  tail -n 200 /var/log/nginx/api-error.log
  ```

- **Build ok, new color unhealthy** → fix app, re‑run deploy; old color stays serving until successful flip.
- **Port conflict** → make sure no `container_name` is set and that only one stack per color runs.

---

## 9) Notes & Safety

- Do **not** commit `.env.production` or any PEM keys.
- Build uses **BuildKit** (`DOCKER_BUILDKIT=1`) for speed and features.
- The workflow prunes dangling images to save disk.
- If you run stateful services (DB/Redis), keep them **outside** the blue/green app stack or only in one color.

---

# Observability Stack — Grafana + Loki + Promtail

This repo contains a **production-ready logging stack** for your apps running on the same EC2 host:

- **Next.js (web)** at `https://app.ethreeindia.com/`
- **NestJS (api)** at `https://api.ethreeindia.com/`
- **Grafana (logs UI)** at `https://logs.ethreeindia.com/` (proxied via Nginx to a localhost-only Grafana)

Everything is Dockerized with **docker compose**. Logs from your containers are tailed by **Promtail**, stored in **Loki**, and explored in **Grafana**.

---

## 🧭 TL;DR — Quick Start (on the EC2 host)

```bash
# 1) Clone & enter
git clone <your-repo-url> /srv/observability
cd /srv/observability

# 2) Prepare env
cp .env.example .env
# Edit .env and set at least:
#   GRAFANA_ADMIN_USER=admin
#   GRAFANA_ADMIN_PASS=<strong-password>
#   GF_SERVER_ROOT_URL=https://logs.ethreeindia.com/

# (Optional) 2b) SMTP for password reset emails
nano .env.smtp.local
# Put SES or Gmail settings (see "📧 SMTP" section), then:
chmod 600 .env.smtp.local

# 3) One-time filesystem permissions (host)
sudo mkdir -p /var/lib/promtail
sudo chmod 755 /var/lib/promtail
sudo chmod o+rx /var/lib/docker /var/lib/docker/containers

# 4) Start the stack
docker compose --env-file .env --env-file .env.smtp.local up -d

# 5) Nginx vhost points logs.ethreeindia.com → 127.0.0.1:3300 (see config below)
sudo nginx -t && sudo systemctl reload nginx
```

Now open **https://logs.ethreeindia.com/** and log in with your admin credentials.  
Go to **Explore → Data source: Loki** and try queries like `{}` or `{container=~".*nestjs.*"}`.

---

## 🏗️ Architecture

```
[Next.js container] --\
                        \                +------------+         +---------+
[NestJS container] ----- > Docker logs → |  Promtail  |  push → |  Loki   |
                        /                +------------+         +---------+
[any other container] -/                      |
                                              v
                                         +---------+
                                         | Grafana |  ← reverse proxy (Nginx) ← Internet
                                         +---------+
```

- **Promtail** uses Docker service discovery and tails `/var/lib/docker/containers/<id>/<id>-json.log`.
- **Loki** stores logs with labels such as `container`, `compose_service`, `env`, etc.
- **Grafana** binds to localhost `127.0.0.1:3300` and is **only** exposed publicly via Nginx over HTTPS.

---

## 📂 Directory Layout

```
/srv/observability
├─ docker-compose.yml
├─ .env.example
├─ grafana/
│  ├─ provisioning/
│  │  ├─ datasources/   (optional: auto-provision Loki datasource)
│  │  └─ dashboards/    (optional: providers for JSON dashboards)
├─ dashboards/           (optional: JSON dashboards)
├─ loki/
│  └─ (using image built-in config; data kept in a named volume)
├─ promtail/
│  └─ config.yml
└─ nginx/
   └─ logs.ethreeindia.conf (example reverse-proxy server block)
```

---

## ⚙️ docker-compose.yml (summary)

Key points already wired for your setup:

- **Grafana**: `127.0.0.1:3300 -> 3000` (Nginx will proxy to this)
- **Promtail**: `127.0.0.1:9080 -> 9080` (host-only status endpoints `/ready`, `/targets`)
- **Loki**: no public port (Grafana talks to Loki on the Docker network)

```yaml
version: '3.9'
name: observability

services:
  loki:
    image: grafana/loki:3.0.0
    command: ['-config.file=/etc/loki/local-config.yaml']
    volumes:
      - loki-data:/loki
    restart: unless-stopped
    networks: [observability]

  promtail:
    image: grafana/promtail:3.0.0
    command:
      - '-config.file=/etc/promtail/config.yml'
      - '-config.expand-env=true'
    environment:
      ENV_LABEL: 'prod'
    ports:
      - '127.0.0.1:9080:9080'
    volumes:
      - ./promtail/config.yml:/etc/promtail/config.yml:ro
      - /var/lib/promtail:/var/lib/promtail
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
    depends_on:
      - loki
    restart: unless-stopped
    networks: [observability]

  grafana:
    image: grafana/grafana:11.2.0
    ports:
      - '127.0.0.1:3300:3000'
    environment:
      - GF_SECURITY_ADMIN_USER=${GRAFANA_ADMIN_USER}
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_ADMIN_PASS}
      - GF_USERS_ALLOW_SIGN_UP=false
      - GF_ANALYTICS_REPORTING_ENABLED=false
      - GF_SERVER_ROOT_URL=${GF_SERVER_ROOT_URL}
      # SMTP (if provided via .env.smtp.local)
      - GF_SMTP_ENABLED=${GF_SMTP_ENABLED}
      - GF_SMTP_HOST=${GF_SMTP_HOST}
      - GF_SMTP_USER=${GF_SMTP_USER}
      - GF_SMTP_PASSWORD=${GF_SMTP_PASSWORD}
      - GF_SMTP_FROM_ADDRESS=${GF_SMTP_FROM_ADDRESS}
      - GF_SMTP_FROM_NAME=${GF_SMTP_FROM_NAME}
      - GF_SMTP_STARTTLS_POLICY=${GF_SMTP_STARTTLS_POLICY}
      - GF_SMTP_EHLO_ID=${GF_SMTP_EHLO_ID}
      - GF_SMTP_SKIP_VERIFY=${GF_SMTP_SKIP_VERIFY:-false}
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
      - ./dashboards:/var/lib/grafana/dashboards:ro
    depends_on:
      - loki
    restart: unless-stopped
    networks: [observability]

networks:
  observability:
    driver: bridge

volumes:
  grafana-data:
  loki-data:
```

---

## 📝 Promtail config (promtail/config.yml)

Already tuned for Docker logs + useful labels + basic JSON parsing.

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /var/lib/promtail/positions.yml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: docker
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
        refresh_interval: 5s

    relabel_configs:
      # where to read the Docker JSON file from
      - source_labels: ['__meta_docker_container_id']
        target_label: __path__
        replacement: /var/lib/docker/containers/$1/$1-json.log

      # helpful labels
      - source_labels: ['__meta_docker_container_name']
        target_label: container
        regex: '/(.*)'
      - source_labels:
          ['__meta_docker_container_label_com_docker_compose_service']
        target_label: compose_service
      - source_labels:
          ['__meta_docker_container_label_com_docker_compose_project']
        target_label: compose_project
      - source_labels: ['__meta_docker_container_log_stream']
        target_label: stream
      - target_label: env
        replacement: ${ENV_LABEL}

    pipeline_stages:
      - docker: {}
      - json:
          expressions:
            level: level
            message: message
          source: log
          on_error: 'skip'
      - regex:
          # named capture (?P<level>) works with promtail's RE2
          expression: '^(?i)(?P<level>error|warn|info|debug|trace)'
          source: log
          on_error: 'skip'
      - labels:
          level:
```

---

## 🔐 Environment

`/srv/observability/.env.example`

```ini
# Grafana admin
GRAFANA_ADMIN_USER=admin
GRAFANA_ADMIN_PASS=ChangeMe!123

# Public URL used in links (emails, redirects)
GF_SERVER_ROOT_URL=https://logs.ethreeindia.com/

# Optional: set environment label for Promtail (e.g., prod / dev)
ENV_LABEL=prod
```

> **Do not commit secrets.** Keep production secrets (passwords, SMTP) in a local-only file.

---

## 📧 SMTP (password reset, alert emails)

Create **`.env.smtp.local`** (don’t commit this file) and load it when starting Compose.

**Amazon SES (ap-south-1 example)**

```ini
GF_SMTP_ENABLED=true
GF_SMTP_FROM_ADDRESS=no-reply@ethreeindia.com
GF_SMTP_FROM_NAME=Grafana (Logs)
GF_SMTP_HOST=email-smtp.ap-south-1.amazonaws.com:587
GF_SMTP_USER=<SES SMTP username>
GF_SMTP_PASSWORD=<SES SMTP password>
GF_SMTP_STARTTLS_POLICY=MandatoryStartTLS
GF_SMTP_EHLO_ID=logs.ethreeindia.com
```

**Gmail (App Password)**

```ini
GF_SMTP_ENABLED=true
GF_SMTP_FROM_ADDRESS=you@gmail.com
GF_SMTP_FROM_NAME=Grafana (Logs)
GF_SMTP_HOST=smtp.gmail.com:587
GF_SMTP_USER=you@gmail.com
GF_SMTP_PASSWORD=<16-char app password>
GF_SMTP_STARTTLS_POLICY=MandatoryStartTLS
GF_SMTP_EHLO_ID=logs.ethreeindia.com
```

Start with SMTP loaded:

```bash
docker compose --env-file .env --env-file .env.smtp.local up -d grafana
```

### Set user emails

Password reset sends to the **email on the user account**.  
If the built-in `admin` user has no email, set one (in Grafana UI or via sqlite if locked out).

---

## 🌐 Nginx site for logs.ethreeindia.com

Reverse-proxy to Grafana on localhost:3300. TLS via Certbot.

```nginx
# HTTP → HTTPS redirect
server {
    listen 80;
    server_name logs.ethreeindia.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name logs.ethreeindia.com;

    ssl_certificate     /etc/letsencrypt/live/app.ethreeindia.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/app.ethreeindia.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # Security headers (adjust to your policy)
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options DENY;
    add_header X-XSS-Protection "1; mode=block";
    add_header Referrer-Policy "strict-origin-when-cross-origin";

    location / {
        proxy_http_version 1.1;
        proxy_read_timeout 180s;

        proxy_set_header Host               $host;
        proxy_set_header X-Real-IP          $remote_addr;
        proxy_set_header X-Forwarded-For    $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Host   $host;
        proxy_set_header X-Forwarded-Proto  https;

        proxy_redirect off;
        proxy_pass http://127.0.0.1:3300/;
    }

    access_log /var/log/nginx/logs-access.log;
    error_log  /var/log/nginx/logs-error.log;
}
```

Certbot example (multi-domain):

```bash
sudo certbot --nginx \
  -d app.ethreeindia.com \
  -d api.ethreeindia.com \
  -d logs.ethreeindia.com \
  --redirect
```

---

## 🧪 Health & Debug

Promtail (host):

```bash
curl -sS http://127.0.0.1:9080/ready        # expect: ready
curl -sS http://127.0.0.1:9080/targets | jq .  # requires jq; otherwise just curl it
```

Grafana (host):

```bash
curl -I http://127.0.0.1:3300/login  # expect HTTP/1.1 301 → https://logs.ethreeindia.com/login
```

Loki (from same Docker network; optional):

```bash
NET=$(docker network ls | awk '/observability/ {print $2}' | head -n1)
docker run --rm --network "$NET" curlimages/curl:8.8.0 -sS http://loki:3100/ready
```

---

## 🔐 Reset the Grafana admin password (safe way)

If the `.env` values don’t work (they seed only on first DB creation), reset against the **data volume**:

```bash
cd /srv/observability
docker compose stop grafana

# Identify the volume name
VOL=$(docker volume ls --format '{{.Name}}' | grep -E '^observability_grafana-data|grafana-data' | head -n1)

# Fix ownership to UID/GID 472
docker run --rm -v "$VOL":/var/lib/grafana alpine:3.20 \
  sh -lc 'chown -R 472:472 /var/lib/grafana'

# Reset using grafana-cli (override entrypoint)
docker run --rm --entrypoint grafana-cli -v "$VOL":/var/lib/grafana \
  grafana/grafana:11.2.0 admin reset-admin-password 'NewStrongPass!234'

docker compose start grafana
```

Open in **incognito** and log in with the new password.

---

## 🔎 Common Loki queries

- All logs:
  ```
  {}
  ```
- Only NestJS containers (by name or compose service):
  ```
  {container=~".*nestjs.*"}        # or {compose_service=~".*nestjs.*"}
  ```
- Errors only (using parsed labels):
  ```
  {level="error"}
  ```
- Filter + parse JSON + pick fields:
  ```
  {compose_service="nestjs-blue-nestjs-app-1"} | json | userId="123"
  ```

---

## 🛡️ Security Notes

- Grafana is **not** publicly bound; only Nginx publishes it over HTTPS.
- Don’t commit secrets (`.env.smtp.local`, real admin passwords).
- Rotate admin credentials and consider creating named admin users per person.
- Optionally disable the default `admin` account after creating your own admin user.

---

## 🧰 Day-2 Operations

```bash
# Restart services
docker compose up -d
docker compose restart grafana loki promtail

# Tail logs
docker compose logs -f grafana
docker compose logs -f promtail
docker compose logs -f loki

# Update images
docker compose pull
docker compose up -d
```

---

## ❓FAQ

**Q: I get “Too many redirects” when opening Grafana.**  
A: Ensure Nginx sets `X-Forwarded-Proto https` and your `GF_SERVER_ROOT_URL` is the public URL (`https://logs.ethreeindia.com/`).

**Q: Password reset says “Failed to send email”.**  
A: Check Grafana logs immediately after triggering the email. Common fixes: correct SMTP creds, use port 587 + `GF_SMTP_STARTTLS_POLICY=MandatoryStartTLS`, verify SES sender/recipient identities if account is in sandbox, or allow outbound on 587.

**Q: Promtail is restarting.**  
A: YAML/permissions/mounts are the usual causes. Confirm mounts, ensure `/var/lib/docker/containers` and `/var/lib/promtail` are readable, and that the `__path__` relabel points to `$id-json.log`.

ChatGPT Auto-Fix for NestJS

This repository includes a GitHub Actions workflow that uses **ChatGPT (gpt-5)** to automatically fix lint, typecheck, and build issues in a **NestJS + TypeScript** codebase and opens a pull request with the changes.

- **Autofix workflow name:** `ChatGPT Autofix (Nest API root)`
- **Default trigger:** on push to `dev` (also supports manual `workflow_dispatch`)
- **Edits are minimal & safe:** public API routes, env var names, and provider contracts are preserved
- **PRs include rationale:** the bot leaves short notes to reviewers (e.g. `.ai-notes.md`)

> If you also enabled the optional **Typo Scan** workflow, see the appendix at the end.

---

## How it works

1. **Trigger**  
   When you push to `dev` (or run manually), the workflow starts on `ubuntu-latest`.

2. **Environment & install**
   - Detects package manager (`pnpm`, `yarn`, `npm`)
   - Sets up Node 20 and caches dependencies
   - Installs project dependencies

3. **Signals (non-fatal)**  
   Runs available scripts (`lint`, `typecheck`, optional `sanity`) to collect diagnostics the model can use as hints.

4. **Changed files only**  
   Determines which files changed in the push and **scopes the model’s edits to those files**, chunked to respect token limits.

5. **ChatGPT pass**  
   For each chunk, the workflow sends a compact prompt + the file contents and collected diagnostics to the **Responses API**  
   (`POST /v1/responses`, model `gpt-5`). The model replies with a **strict JSON** containing full-file rewrites; the runner applies them and stages the edits.

6. **Notes & sentinel**  
   If the model included notes, they’re appended to `.ai-notes.md`. A sentinel file `.ai-edits-applied` is created when any edit is applied.

7. **PR creation**  
   If edits were staged, a PR is opened to `dev` via `peter-evans/create-pull-request@v7` with commit message  
   `chore(chatgpt-api-root): auto-fix lint/types/tests`.

---

## Repository secrets

Create the following **repository secrets** (Settings → Secrets and variables → Actions → New repository secret):

| Secret           | Description                                                                                  |
| ---------------- | -------------------------------------------------------------------------------------------- |
| `OPENAI_API_KEY` | Your OpenAI API key with access to `gpt-5`                                                   |
| `GH_PAT`         | A GitHub Personal Access Token with `repo` scope so the workflow can push a branch & open PR |

> The workflow gracefully logs and exits when `OPENAI_API_KEY` is missing.

---

## Files & paths behavior

- **Ignored for triggers:**  
  The `on.push.paths-ignore` excludes docs/images/yml/etc. so cosmetic changes don’t trigger the run.
- **Edit scope:**  
  Only files changed in the pushed commit range are considered; the model is asked to **touch imports minimally** if needed.
- **Outputs:**
  - `.ai-notes.md` — short rationale / follow-ups from the model
  - `.ai-edits-applied` — sentinel indicating at least one edit was applied (used to detect “real” changes)

---

## Safety rails in the prompt

The workflow instructs the model to:

- Fix **ESLint/Prettier/TypeScript/build** issues with **minimal, safe edits**
- **Do not** change public **HTTP routes**, **env var names**, or **provider contracts**
- Keep config changes **narrow** and document them in notes
- Prefer **local fixes** over sweeping refactors

---

## Running it

### 1) Automatically on push

Push to the `dev` branch:

```bash
git checkout dev
git commit -m "wip"
git push origin dev
```

### 2) Manually from Actions

- Go to **Actions → ChatGPT Autofix (Nest API root) → Run workflow**
- Choose the branch (e.g. `dev`) and run

---

## Customization

- **Change the trigger branch**  
  In the workflow YAML:

  ```yaml
  on:
    push:
      branches: ['dev'] # change to 'main' or another branch if preferred
  ```

- **Include/exclude paths**  
  Update `paths-ignore` to tune what pushes trigger a run. You can also add a `paths:` list to only react to certain folders.

- **Node / package manager**  
  Bump Node in `actions/setup-node@v4` or pin your preferred `pnpm` via `corepack prepare`.

- **Token budgets / chunking**  
  The script chunks changed files to ~40k characters per batch (~10k tokens). Adjust inside the runner script if needed.

---

## Troubleshooting

**“No changes detected.”**  
The model chose not to edit anything or couldn’t confidently fix issues in the diff. Check the job log for applied edits; if none, push a failing change or add a `sanity` script to surface errors.

**No PR created.**

- Verify `GH_PAT` is set and valid.
- The “Detect changes” step prints staged files; if empty, there’s nothing to PR.

**429 rate limits.**  
The workflow includes lightweight backoff and chunking. If you routinely hit 429s, reduce chunk size or run less frequently.

**OpenAI 400: “Unsupported parameter: temperature.”**  
The workflow uses the **Responses API** for `gpt-5` and does **not** send `temperature`. If you run any local scripts, remove `temperature` when calling `gpt-5`.

**“Context access might be invalid: typos”**  
This was from referencing a step’s own outputs in the same step. The current workflows avoid that pattern.

---

## Local dry-run (optional)

You can simulate an autofix run against a single file locally using your API key:

```bash
export OPENAI_API_KEY=sk-...
export CHANGED_FILES="src/some/file.ts"
node scripts/chatgpt-autofix.mjs
```

> In CI we write the script **ephemerally** to the runner; if you want local runs, keep a copy under `scripts/`.

---

## Security & privacy

- Secrets are read via standard GitHub Actions env (`OPENAI_API_KEY`, `GH_PAT`).
- The model only sees **changed files** plus high-level diagnostics (lint/typecheck output).
- The helper script is created in the runner’s temp folder and **is not committed** to the repo.

---

## FAQ

**Will this rewrite my app?**  
No. The prompt demands **minimal** edits. Large refactors aren’t attempted.

**Does it rely on unit tests to fix bugs?**  
No. It targets **lint/TS/build** issues primarily. For functional bugs, add failing unit/integration tests to guide fixes.

**Can I run it on every branch?**  
Yes—add branches to `on.push.branches` or duplicate the job with different settings.

---

## Appendix: Optional Typo Scan (on-demand, full repo)

If you also added the on-demand typo scan workflow:

- **Workflow name:** `ChatGPT Typo Scan (On-Demand Full Repo)`
- **Trigger:** manual `workflow_dispatch`
- **Scope:** scans **entire repo** (code & docs) for **obvious** English misspellings  
  (conservative on identifiers; avoids API-breaking renames)
- **Output:** `.ai-typos.md` (table of suggestions) and a sentinel `.ai-typos-found`
- **PR behavior:** opens a PR **only** when suggestions exist

Run it from **Actions → ChatGPT Typo Scan (On-Demand Full Repo) → Run workflow**.

# AI-powered SEO Generation & Prompt Management

This feature automatically generates high‑quality SEO metadata for Events using OpenAI, while giving you a **Prompt Management** system to control the tone/instructions centrally. It also provides **deterministic fallbacks**, **caching**, **metrics**, and **unique storage** per event.

---

## Table of Contents

- [What it does](#what-it-does)
- [Architecture](#architecture)
- [Data Models](#data-models)
- [Normalization & JSON‑LD](#normalization--json-ld)
- [SEO Generation Flow](#seo-generation-flow)
- [Prompt Management](#prompt-management)
- [HTTP APIs](#http-apis)
- [Caching](#caching)
- [Metrics](#metrics)
- [Environment Variables](#environment-variables)
- [Next.js Integration](#nextjs-integration)
- [Seeding a Prompt](#seeding-a-prompt)
- [Troubleshooting](#troubleshooting)
- [Security Notes](#security-notes)

---

## What it does

- **Generates SEO** (title, description, keywords, OG text, slug) for an event via OpenAI.
- **Never returns partial SEO**: deterministic hydration fills any missing fields.
- **Builds JSON‑LD (`schema.org/Event`)** with reliable start/end dates and organizer info.
- **Stores one SEO doc per event** (enforced by a unique index).
- **Caches reads** and **invalidates** when SEO is (re)generated.
- **Tracks metrics**: cache hits/misses, token usage, and latency histograms.
- **Centralizes prompts** in a `PromptsService` with DB overrides and defaults.

---

## Architecture

```
[Next.js] ───────────(Axios)──────────▶ [/v1/seo/:eventId]
   │                                           │
   │ fetchSeoByEventId                         │  SeoController
   ▼                                           ▼
generateMetadata()                   SeoService.generate(event,eventId,force)
                                              │
                                              ├─ normalizeEvent(raw)
                                              ├─ build prompt (PromptsService: ns=seo,key=system)
                                              ├─ call OpenAI (gpt‑5|o4‑mini|4o-dated)
                                              ├─ hydrate with deterministic defaults
                                              ├─ buildEventJsonLd(norm, hydrated)
                                              └─ upsert {eventId, contentHash, seo} in Mongo
```

---

## Data Model

**Collection:** `seodocs`

```ts
{
  _id: ObjectId,
  eventId: ObjectId | string,        // unique with contentHash
  contentHash: string,               // sha1(SEO-relevant seed)
  seo: SeoPack,                      // AI+deterministic pack
  createdAt: Date,
  updatedAt: Date
}
```

### SeoPack (response shape)

```ts
type SeoPack = {
  seoTitle: string;
  seoDescription: string;
  keywords: string[];
  ogTitle: string;
  ogDescription: string;
  ogImageText?: string;
  canonicalPath: string;
  robots: { index: boolean; follow: boolean };
  slug: string;
  jsonLd: Record<string, any>; // schema.org/Event
};
```

### PromptDoc

```ts
{
  _id: ObjectId,
  ns: string,               // e.g. "seo"
  key: string,              // e.g. "system"
  locale?: string | null,   // e.g. "en-IN" (nullable)
  text: string,             // the prompt body (templated or static)
  version?: number,
  meta?: Record<string, any>,
  createdAt: Date,
  updatedAt: Date
}
```

---

## Normalization & JSON‑LD

1. **normalizeEvent(raw)** → `EventNorm` ensures date/times become ISO strings (`dateFromISO`, `dateToISO`) and pulls core fields (title, city, venue, highlights, exams, courses…).
2. **buildEventJsonLd(norm, seoPack)** → Generates a schema.org/Event object:
   - Uses `seoPack.seoDescription` for `description`
   - Converts `dateFromISO` and `dateToISO` to `startDate`/`endDate`
   - Includes organizer details if available
   - Adds keywords (exams/courses) safely

> If a field is missing from OpenAI, hydration ensures `seoPack` still contains valid values to drive JSON‑LD.

---

## SEO Generation Flow

1. **Normalize** incoming event → `norm`
2. **Hash** SEO‑relevant fields → `contentHash`
3. **Check stored SEO** by `(eventId, contentHash)` (short‑circuit on hit unless `force`)
4. **Fetch system prompt** via `PromptsService.get('seo','system')` with DB, env, or defaults
5. **Call OpenAI**:
   - Prefer `json_schema` response format (for dated 4o models)
   - Otherwise use **tool‑calling strict** mode (works with `gpt-5`, `o4-mini`)
6. **Hydrate** model output with deterministic fallbacks (always complete)
7. **Build JSON‑LD** using hydrated values
8. **Upsert** a single record per event (unique constraint)
9. **Invalidate** cache keys for this event
10. **Emit metrics** for calls/latency/tokens

---

## Prompt Management

The system prompt is not hardcoded. It is resolved in this order:

1. **DB override**: a `PromptDoc` with `ns="seo"`, `key="system"`, and matching `locale` (if provided)
2. **Env var**: `PROMPT_SEO__SYSTEM`
3. **Built‑in default** (`PROMPT_DEFAULTS.seo.system`)

You can manage prompts via admin APIs (below). Prompts are cached with TTL and can interpolate variables.

---

## HTTP APIs

### SEO (public)

#### `GET /v1/seo/:eventId` _(BasicAuth)_

Fetch the cached SEO pack for an event.

- **Auth**: `Basic` (consistent with Event GET; configurable)
- **Caching**: enabled via decorators; key varies by `:eventId`
- **Response**: `SeoPack | null`

**Example:**

```bash
curl -u "$BASIC_USER:$BASIC_PASS" \
  http://localhost:4000/v1/seo/68a385080db39ab6dcb9c4d7
```

#### `POST /v1/seo/generate?force=1` _(JWT)_

Generate (or refresh) SEO for an event. You can pass the event payload or only the `eventId` (the service will fetch the event server‑side).

**Body:**

```json
{
  "eventId": "68a385080db39ab6dcb9c4d7",
  "event": {
    /* optional: raw event json; if omitted, server loads by eventId */
  },
  "force": "1"
}
```

**Response:** `SeoPack`

**Notes:**

- If `force` is `1|true`, the content hash short‑circuit is bypassed.
- On success, route invalidates related cache keys.

---

### Prompts (admin)

#### `GET /v1/prompts?ns=seo&key=system&locale=en-IN&limit=50&offset=0` _(BasicAuth)_

List prompts filtered by namespace/key/locale with pagination.

#### `GET /v1/prompts/lookup?ns=seo&key=system&locale=en-IN` _(BasicAuth)_

Lookup a single prompt by its unique triplet.

#### `GET /v1/prompts/:id` _(BasicAuth)_

Fetch prompt by id.

#### `POST /v1/prompts` _(JWT)_

Create a new prompt (fails if the (ns,key,locale) exists).

#### `PUT /v1/prompts/upsert` _(JWT)_

Create or update a prompt by (ns,key,locale).

#### `PATCH /v1/prompts/:id` _(JWT)_

Update by id (also invalidates caches for both old and new keys).

#### `DELETE /v1/prompts/:id` _(JWT)_

Delete by id (invalidates cache for that key).

---

## Caching

- **HTTP caching** via decorators:
  - `@HttpCacheTTL(ms)` and `@HttpCacheKeyFromParam('eventId', 'seo:')`
  - `@AllowAuthCache()` allows caching even when guards are present
- **SeoService** mirrors `EventsService` invalidation:
  - Busts the list key (`seo:list:v1`) and both URL/alias keys for a single event
- **PromptsService** caches prompt lookups with TTL; invalidated on writes

---

## Metrics

Emitted via `MetricsService`:

- **Counters**
  - `seo_cache_events_total{type=hit|miss}`
  - `seo_generate_calls_total{model,mode,status}`
  - `seo_openai_tokens_total{model,mode,kind=input|output|total}`
- **Histograms**
  - `seo_generate_latency_ms{model,mode,status}` with buckets `[50,100,200,500,1000,2000,5000,10000]`

Use these to build dashboards/alerts.

---

## Environment Variables

| Variable                      | Example                    | Purpose                                                         |
| ----------------------------- | -------------------------- | --------------------------------------------------------------- |
| `OPENAI_API_KEY`              | `sk-...`                   | OpenAI access token                                             |
| `OPENAI_SEO_MODEL`            | `gpt-5`                    | Preferred model (`gpt-5`, `o4-mini`, `gpt-4o-2024-08-06`, etc.) |
| `PROMPT_SEO__SYSTEM`          | _long text_                | Env‑level system prompt (fallback)                              |
| `NEXT_PUBLIC_USE_BASIC_AUTH`  | `true`                     | Adds Basic auth to API calls from Next                          |
| `NEXT_PUBLIC_BASIC_AUTH_USER` | `admin`                    | Basic user (Next only)                                          |
| `NEXT_PUBLIC_BASIC_AUTH_PASS` | `secret`                   | Basic pass (Next only)                                          |
| `API_URL`                     | `http://localhost:4000/v1` | SSR base URL                                                    |
| `NEXT_PUBLIC_API_URL`         | `http://localhost:4000/v1` | Browser base URL                                                |
| `NEXT_PUBLIC_USE_PROXY`       | `true`                     | Use `/api` Next proxy in dev                                    |
| Mongo/Redis/etc.              | —                          | Standard Mongoose/Cache configuration                           |

> **Temperature**: Only set for the dated `gpt-4o-*` models in the Responses API. For `gpt-5`/`o4-mini`, we omit it to avoid API errors.

---

## Next.js Integration

1. **Axios client** (`lib/api.ts`) handles Basic/JWT and SSR cookie forwarding.
2. **SEO service** (`services/seoService.ts`):
   ```ts
   export const fetchSeoByEventId = async (eventId: string) => {
     return api.get<SeoPack | null>(`/seo/${eventId}`, { cache: 'no-store' });
   };
   ```
3. **Page SEO** (`app/event/[id]/page.tsx`) using your generic builder:

   ```ts
   export async function generateMetadata({
     params,
   }: {
     params: Promise<{ id: string }>;
   }): Promise<Metadata> {
     const { id } = await params;
     let seo: SeoPack | null = null;

     try {
       const res = await fetchSeoByEventId(id);
       seo = res.data ?? null;
     } catch {}

     if (!seo) {
       const ev = await loadEvent(id);
       if (!ev) return { title: 'Event not found' };
       seo = await generateSeo(id, ev);
     }

     return buildMetadataFromResource<SeoPack>({
       fetcher: async () => seo,
       pick: (s) => ({
         title: s.seoTitle,
         description: s.seoDescription,
         path: s.canonicalPath,
         keywords: s.keywords,
       }),
       fallbackTitle: 'Event Details',
       fallbackDescription: 'Event information and registration.',
       overrides: {
         robots: { index: seo.robots.index, follow: seo.robots.follow },
       },
     });
   }
   ```

---

## Seeding a Prompt

Use **admin API** to upsert the system prompt:

```bash
curl -X PUT "http://localhost:4000/v1/prompts/upsert" \
  -H "Authorization: Bearer $JWT" \
  -H "Content-Type: application/json" \
  -d '{
    "ns": "seo",
    "key": "system",
    "locale": "en-IN",
    "text": [
      "You are an SEO writer for a university/career events website in India.",
      "Only use facts from the provided event JSON. Do NOT invent details.",
      "Keep titles <=60 chars, descriptions <=155 chars. Avoid clickbait.",
      "Return a short, clean slug (kebab-case).",
      "Use IST (Asia/Kolkata) context for dates/times."
    ].join(" ")
  }'
```

---

## Troubleshooting

- **“Unknown parameter: tool_choice.\*”**  
  You’re calling Responses API with unsupported `tool_choice`. Remove it.
- **“Unsupported parameter: temperature”**  
  Don’t set `temperature` for `gpt-5`/`o4-mini`. Only the dated `gpt-4o-*` Responses models accept it.
- **OpenAI returns text, not structured output**  
  We parse `tool_call` arguments first; if missing, we try `output_text`. Hydration then guarantees a full pack.
- **Null `startDate`/`endDate`**  
  Ensure `normalizeEvent()` sets `dateFromISO`/`dateToISO`. The JSON‑LD builder uses those ISO strings to set dates.
- **GET /seo/:id returns null**  
  If your collection stores `ObjectId` for `eventId`, the service now queries by `ObjectId` **or** string to match both.
- **Multiple docs for the same event**  
  A unique index on `{ eventId: 1 }` prevents duplicates. On `11000`, the service falls back to an `updateOne`.

---

## Security Notes

- Reads can be protected via **BasicAuth**; writes (generate, prompts CRUD) require **JWT**.
- Axios SSR forwards cookies, and your API refreshes tokens when 401 (except auth routes).
- Avoid logging secrets; request/response interceptors redact `Authorization` headers.
- Robots defaults to `{ index: true, follow: true }`. Adjust per route if needed.

---

# Bug Tracking & Error Reporting (NestJS) — Quick README

A drop‑in module to **capture, dedupe, assign, and track** backend (NestJS) errors — plus accept **frontend error intake** (from Next.js or any client) — and store them in MongoDB. Supports **Slack alerts** and can be **enabled/disabled via env**.

---

## Features

- **Global capture** via `HttpExceptionFilter` (registered app‑wide)
  - Classifies **5xx** as `severity=BUG`
  - Classifies **408/429** as `severity=WARNING` (optional triage)
  - Fingerprints errors (stable SHA‑256) to dedupe recurring issues
  - Records request context, sanitized payload samples, and increments an `occurrences` counter
- **Frontend intake** endpoint (`POST /bug-intake`) to ingest errors reported by your web app
- **Admin APIs** to list, assign, and update bug status (Swagger documented)
- **Slack alerts** (optional)
- **Feature toggles** via environment variables

---

## TL;DR — Setup

1. **Enable module**
   - In Nest: set `BUG_TRACKING_ENABLED=true`
   - (Optional) Slack: `BUG_SLACK_ENABLED=true` + `BUG_SLACK_WEBHOOK=...`

2. **Import module (conditional)**
   - In `app.module.ts`: import `BugTrackerModule` only when `BUG_TRACKING_ENABLED=true`

3. **Register global filter**
   - In `main.ts`: register `HttpExceptionFilter` (DI-injected with `BugTrackerService`)

4. **Expose admin APIs**
   - `GET /bug-trackers` (filters: `status`, `severity`, `route`, `q`)
   - `PATCH /bug-trackers/:fingerprint/assign`
   - `PATCH /bug-trackers/:fingerprint/status`
   - These are already documented in Swagger

5. **Create frontend intake endpoint**
   - `POST /bug-intake` (protected by `x-bug-intake-token`)
   - Compute fingerprint and call `BugTrackerService.recordOrUpdate(...)`

6. **Next.js integration (client)**
   - Global error screen: re-export a central `ErrorScreen`
   - Axios interceptor: report `network/timeout` + `5xx`
   - Next proxy route `/bug-intake` forwards to Nest `BUG_INTAKE_URL` with server-only `BUG_INTAKE_TOKEN`

---

## Environment (minimal)

**NestJS**

```
BUG_TRACKING_ENABLED=true
BUG_SLACK_ENABLED=false           # true to enable Slack
BUG_SLACK_WEBHOOK=...             # if using webhook
```

**Next.js**

```
ERROR_REPORTING_ENABLED=true
BUG_INTAKE_URL=https://api.yourdomain.com/v1/bug-intake
BUG_INTAKE_TOKEN=super-long-random
```

> Use a **Next proxy route** (`/bug-intake`) so tokens stay **server-only**.

---

## Data model (at a glance)

- `fingerprint` (unique), `route`, `statusCode`
- `status` (`NEW`, `TRIAGED`, `ASSIGNED`, `IN_PROGRESS`, `RESOLVED`, `WONT_FIX`)
- `severity` (`BUG` | `WARNING`), `occurrences`, `lastSeen`, `resolvedAt?`
- `message`, `name`, `stack`
- `assigneeId`, `assigneeName`
- `tags[]`, `context`, `samplePayload`, `sentryEventId`, `history[]`

**Indexes**: `fingerprint (unique)`, `lastSeen`, `(status,lastSeen)`, `(severity,lastSeen)`, `(route,lastSeen)`

---

## How it classifies

- **5xx** → `severity=BUG` (tracked)
- **408 / 429** → `severity=WARNING` (tracked)
- **4xx** → not tracked as bugs (normal responses/logs)

---

## Admin flow

1. List & filter by `severity`/`status`
2. Assign to a developer
3. Update status (adds audit history, stamps `resolvedAt`)
4. Optional Slack notifications

---

## Security & privacy

- Frontend intake requires `x-bug-intake-token` (server-held secret)
- Request payloads are sanitized (redacts common secret keys)
- Prefer proxy route to avoid CORS and keep secrets off the client

---

## File map (where to look)

- **Filter**: `src/filters/http-exception/http-exception.filter.ts`
- **Module**: `src/bug-tracker/bug-tracker.module.ts`
- **Service**: `src/bug-tracker/bug-tracker.service.ts`
- **Schema**: `src/bug-tracker/schemas/bug-report.schema.ts`
- **Controllers**: `src/bug-tracker/bug-intake.controller.ts`, `src/bug-tracker/bug-reports.controller.ts`
- **Next proxy**: `src/app/bug-intake/route.ts`
- **Client**: `src/app/_errors/ErrorScreen.tsx`, `src/lib/api.ts`, `src/lib/bugReporter.ts`

---

## FAQ

- **Can I disable anytime?** Yes — flip the env flags; no code edits required.
- **Does it loop with my Axios interceptor?** Add and respect a skip header (e.g., `x-skip-bug-report: 1`).
- **Can I add more severities?** Yes — extend schema + classifier.
- **Sentry?** You can store the `sentryEventId` alongside reports for cross-links.

```

## Feature Guides

- [AI SEO Generation (Events)](docs/seo-generation.md)

## Feature Guides

- [Prompts Registry](docs/prompts.md)
```
