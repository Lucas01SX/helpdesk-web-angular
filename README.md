# Helpdesk Web — Angular

Angular 21 SPA for the helpdesk platform. Consumes the [Helpdesk Platform .NET API](https://github.com/Lucas01SX/helpdesk-platform-dotnet).

**Stack:** Angular 21 · TypeScript 5.9 strict · SCSS · Vitest 4 · ESLint 10 · Prettier 3

---

## Commands

```bash
# Development server (http://localhost:4200, proxies /api to :5000)
npm start

# Run unit tests (watch mode)
npm test

# Run unit tests once (CI)
npm test -- --watch=false

# Lint
npm run lint

# Production build
npm run build
```

---

## Architecture

Standalone components throughout — no NgModule. State via Angular Signals (`signal`, `computed`).

```
src/app/
├── core/
│   ├── auth/          ← AuthStore (signal), AuthInterceptor, authGuard, roleGuard
│   ├── models/        ← TypeScript interfaces (auth, ticket, comment, attachment, SLA)
│   └── services/      ← AuthService, TicketService
├── features/
│   ├── auth/          ← login, register, verify-email, reset-password
│   ├── home/          ← role-aware dashboard
│   ├── tickets/       ← ticket list + detail
│   └── sla/           ← SLA scores (Agent/Manager only)
└── layout/
    ├── shell/         ← sidebar + router-outlet (authenticated)
    └── auth-layout/   ← centered layout (unauthenticated)
```

**Auth flow:** Access token stored in memory only (never `localStorage`). Refresh token in `HttpOnly` cookie. `AuthInterceptor` attaches Bearer header and retries on 401 via silent refresh.

**Guards:** `authGuard` — requires authenticated session. `roleGuard` — requires specific role from `route.data['roles']`.

**Lazy loading:** all feature routes use `loadComponent`.

---

## Environment

| File | Used when |
|---|---|
| `src/environments/environment.ts` | `ng serve` (dev) — `apiUrl: ''`, proxy handles routing |
| `src/environments/environment.production.ts` | `ng build` — `apiUrl` points to production host |

Dev proxy config: `proxy.conf.json` → `/api/*` forwarded to `http://localhost:5000`.

---

## CI Pipeline

Every push to `main` or `develop` and every PR targeting those branches runs:

1. **Lint** — ESLint + Prettier
2. **Test** — Vitest
3. **Build** — Angular production build
