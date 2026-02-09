# AXO Networks quick codebase + website review (2026-02-09)

## Scope reviewed
- Backend Node/Express API, middleware, services, and route wiring.
- Frontend structure and key landing page copy/navigation.
- External website reachability from this environment.

## High-priority issues

1. **Hardcoded admin credentials in application code**
   - `services/AuthService.js` allows login with static credentials (`admin@axonetworks.com` / `admin123`).
   - This is a critical security risk and should be removed immediately in production.

2. **Unsafe fallback JWT secret**
   - `services/AuthService.js` and `services/PasswordResetService.js` fallback to `"axo-secret"` if `JWT_SECRET` is missing.
   - This can make token forging feasible when env configuration drifts.

3. **Sensitive credential disclosure in logs**
   - `services/userProvisioningService.js` logs the generated temporary password in plaintext.
   - Any centralized logs or support exports could leak account credentials.

4. **Unprotected verification endpoint**
   - `server.js` exposes `PUT /api/network-request/:id/status` without authentication/authorization middleware.
   - Anyone with API access could approve/reject requests and trigger account creation.

## Medium-priority issues

1. **File upload endpoint lacks guardrails**
   - `routes/rfqFiles.routes.js` accepts uploads without file type allowlist, size limits, malware scanning, or auth checks.

2. **Inconsistent DB connection strategy**
   - Main app uses shared pool from `src/config/db.js`, but some services instantiate their own pools (`services/AuthService.js`, `services/PasswordResetService.js`).
   - This can cause connection sprawl and inconsistent runtime behavior.

3. **Over-verbose runtime logs in DB bootstrap**
   - `src/config/db.js` logs DB host/user/name each startup and exits process on pool errors.
   - Better to reduce exposure and use controlled restart policies.

## Product/website notes

- The landing page messaging strongly positions AXO as an "operating layer" for EV manufacturing with offerings around network access, part discovery, and finance integration (`frontend/index.html`).
- From this environment, direct curl access to `https://www.axonetworks.com/` returned `403` via connect tunnel, so external production-site runtime checks could not be completed here.

## Recommended immediate actions (order)

1. Remove static admin login path and require DB-backed auth only.
2. Make `JWT_SECRET` mandatory everywhere (no default fallback), fail fast at boot.
3. Stop logging temporary passwords; use one-time setup link flow.
4. Protect `PUT /api/network-request/:id/status` with authentication + strict admin role middleware.
5. Add multer constraints (`fileFilter`, `limits`) and authorization to file upload route.
6. Standardize all services on one shared DB pool module.

