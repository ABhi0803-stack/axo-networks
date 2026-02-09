# AXO Networks — Security + Modern UI Execution Plan (One file at a time)

This plan is designed to execute **safely in small steps**.  
Goal: increase production security posture first, then modernize UI/animation system without breaking flows.

---

## 0) Guiding principles

- Security changes first (auth, secrets, endpoint protection, validation).
- Small PRs: one file per PR whenever possible.
- Every change must include at least one check/test command.
- Keep UX smooth: accessibility + performance over flashy animation.

---

## Phase 1 — Security hardening (priority)

### File 1: `services/AuthService.js`
**What to change**
- Remove static admin login (`admin@axonetworks.com` / `admin123`).
- Remove fallback JWT secret (`"axo-secret"`), require `process.env.JWT_SECRET`.
- Replace local `new Pool(...)` with shared DB pool import from `src/config/db`.

**Why**
- Eliminates hardcoded credential backdoor.
- Prevents weak-secret token risk.
- Reduces DB connection inconsistency.

---

### File 2: `services/PasswordResetService.js`
**What to change**
- Remove fallback JWT secret (`"axo-secret"`).
- Replace local `new Pool(...)` with shared DB pool import from `src/config/db`.
- Add password policy validation (length + complexity) before hash.

**Why**
- Standardized token security.
- Consistent DB behavior.
- Better account security hygiene.

---

### File 3: `services/userProvisioningService.js`
**What to change**
- Stop logging temporary passwords in plaintext.
- Return a one-time setup token (or set `must_reset_password=true`) instead of printing secrets.
- Normalize role values (`BUYER` / `SUPPLIER`) to match middleware expectations.

**Why**
- Prevents credential leakage via logs.
- Enables secure first-login flow.
- Avoids authz bugs due to role mismatch.

---

### File 4: `server.js`
**What to change**
- Protect `PUT /api/network-request/:id/status` with auth + admin role middleware.
- Add rate limit middleware for auth/login + sensitive write endpoints.
- Add secure HTTP headers middleware (helmet-style policy).

**Why**
- Blocks unauthorized approval/rejection.
- Mitigates brute-force and abuse.
- Improves baseline web security posture.

---

### File 5: `routes/rfqFiles.routes.js`
**What to change**
- Require authentication for upload endpoint.
- Add file size limits and MIME allowlist.
- Store uploads outside public static tree and generate safe file names.

**Why**
- Reduces malicious upload risk.
- Prevents oversized upload abuse.
- Limits accidental exposure of uploaded files.

---

### File 6: `src/config/envCheck.js`
**What to change**
- Add required env vars for refresh secret, SMTP creds (if used), and base URL.
- Validate secure defaults (`NODE_ENV=production` constraints for prod).

**Why**
- Fails fast on insecure deployment configuration.

---

## Phase 2 — API quality + validation

### File 7: `middleware/validate.middleware.js`
- Centralize Zod validation helpers for body/query/params.
- Return consistent error envelopes.

### File 8: `routes/auth.routes.js`
- Add schema validation for login/refresh.
- Add login rate-limiting and account lockout hooks.
- Remove verbose auth logs that expose internals.

### File 9: `middleware/errorHandler.middleware.js`
- Add environment-aware responses (no stack/details in prod).
- Normalize error codes/messages for frontend UX.

---

## Phase 3 — Modern UI + animations (safe + performant)

### File 10: `frontend/styles/index.css`
- Introduce design tokens (color, spacing, radius, shadow, motion).
- Add dark mode ready variables.
- Add reduced-motion media query.

### File 11: `frontend/index.html`
- Improve semantic structure and accessibility (landmarks, aria labels).
- Keep hero but simplify hierarchy and improve CTA clarity.

### File 12: `frontend/js/*.js` (start with `login.js`)
- Replace noisy console logs with user-focused feedback.
- Add loading states, disabled buttons, inline validation.
- Add smooth transitions for success/error states.

### File 13: `frontend/styles/login.css`
- Build modern, responsive auth screen.
- Strong focus states, visible error/success messaging.

---

## Phase 4 — User-friendliness and trust

### File 14: `frontend/js/admin-dashboard.js`
- Add searchable/filterable request table.
- Add status badges + optimistic UI update pattern.

### File 15: `frontend/styles/admin-dashboard.css`
- Improve spacing, visual hierarchy, and responsive table/card behavior.

### File 16: `frontend/js/network-access.js`
- Add stepper-form UX, autosave draft, and completion indicators.

---

## Execution order for next 5 PRs

1. `services/AuthService.js`
2. `services/PasswordResetService.js`
3. `services/userProvisioningService.js`
4. `server.js`
5. `routes/rfqFiles.routes.js`

If you agree, we start now with **PR #1: `services/AuthService.js` only**.
