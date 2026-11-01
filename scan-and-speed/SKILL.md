---
name: "scan-and-speed"
description: "Audit and fix a web/mobile app for legal compliance, security, speed and admin security monitoring. Use on 'scan and speed', 'speed and security', or before launching or hardening any app."
---

# Scan and Speed (Legal + Security + Speed + Monitoring audit)

Run this on every app — finished apps and new ones. Goal: the app does not get the owner in legal trouble, is not easy to hack, does not feel slow, and the owner can see and control who does what.

## How to run

1. **Map the app first.** Identify stack (framework, hosting, DB, auth, payments, email, analytics, AI providers), routes/pages, forms, data collected, third-party scripts/SDKs. Note what does NOT apply (e.g. no payments → skip refund/subscription items).
2. **Scan all four sections below.** For each item mark: PASS / FIXED / NEEDS-HUMAN / N/A, with file:line evidence.
3. **Fix what code can fix**, smallest safe change first, on a branch (`scan-and-speed/<date>`). Never commit secrets. Run the build and tests after fixes. Lessons from real runs: check for login-free token endpoints, data endpoints that trust a `companyId`/`userId` query param, plaintext passwords sent to the browser, upserts that can move rows between tenants (`ON DUPLICATE KEY UPDATE company_id=...`), server secrets injected into the client bundle (Vite `define`, `NEXT_PUBLIC_`), and schema/seed files in `public/`.
4. **Flag what code cannot fix** as NEEDS-HUMAN (legal text review, DPAs signed with vendors, DMCA agent registration, license permissions, customer logo permission). Generate drafts/templates for these, clearly labelled "DRAFT — have a lawyer review".
5. **Report** as a table: Section | Item | Status | Evidence | What changed / what the owner must do. Put critical security issues at the top.

Rules: Never invent compliance claims (SOC 2, HIPAA, GDPR-certified, uptime %) — remove or soften unsupported ones. Never fabricate reviews, testimonials or logos. Ask before destructive actions (rewriting git history, deleting data, rotating production keys).

---

## 1. LEGAL & COMPLIANCE

**Policies & pages**
- [ ] Privacy Policy page (what data, why, who it's shared with, retention, user rights, contact)
- [ ] Terms of Service page
- [ ] Refund policy (if selling anything)
- [ ] Business contact details (legal name, address/email) in footer or contact page
- [ ] Published sub-processors list (hosting, DB, email, analytics, AI, payments)
- [ ] Realistic uptime commitments only (no unbacked "99.99%")

**Consent & acceptance**
- [ ] Unchecked checkbox at signup to accept Privacy Policy + ToS (not pre-ticked)
- [ ] Record acceptance: user id, timestamp, policy version, IP/user agent
- [ ] Separate, optional marketing-consent checkbox (not bundled with ToS)
- [ ] Cookie banner with Accept / Reject / Customize, equal prominence
- [ ] Block non-essential trackers/analytics until consent is given
- [ ] Unsubscribe link in every marketing email, honored promptly
- [ ] Parental consent before collecting children's data (COPPA <13 US, GDPR-K up to 16) — or age-gate and block

**Data rights & handling**
- [ ] Data access/export request flow (self-serve or documented email)
- [ ] Account & data deletion flow (including backups/third parties within stated time)
- [ ] Data retention limits defined and enforced (scheduled cleanup jobs)
- [ ] Stop collecting unnecessary data — remove unused form fields/columns
- [ ] Prevent unauthorized AI training on user data (check AI provider settings, opt-outs, state it in policy)
- [ ] Data Processing Agreements with vendors (NEEDS-HUMAN; list which vendors)
- [ ] Audit third-party SDKs, analytics and tracking — list each, purpose, data sent; remove unused

**Honest marketing / no dark patterns**
- [ ] Remove fake countdown timers and fake scarcity ("only 2 left")
- [ ] Remove fake reviews/testimonials
- [ ] Remove unsupported claims ("#1", "guaranteed", "10x") unless evidenced
- [ ] Remove false SOC 2 / ISO / HIPAA claims or badges
- [ ] Remove dark patterns: confirmshaming, hidden opt-outs, pre-checked boxes, roach-motel flows
- [ ] No hidden fees — show total price (taxes/fees) before checkout
- [ ] Disclose auto-renewal clearly (price, interval, how to cancel) before purchase
- [ ] Subscriptions as easy to cancel as to start (in-app cancel button)
- [ ] Label AI-generated content and AI chat/agents as AI; never show canned/fake "AI insights" when the AI is unavailable
- [ ] Don't use customer/partner logos without written permission

**Accessibility (also a legal risk — ADA / EAA)**
- [ ] Color contrast meets WCAG AA (4.5:1 text, 3:1 large text/UI)
- [ ] Alt text on meaningful images; empty alt on decorative ones
- [ ] Full keyboard navigation, visible focus states, no keyboard traps, skip-link
- [ ] Form inputs have labels; buttons have accessible names

**Licenses & IP**
- [ ] Verify image, font and icon licenses (commercial use allowed)
- [ ] Check open-source license obligations (attribution file, GPL/AGPL copyleft risk) — run a license scan of dependencies
- [ ] DMCA designated agent registration if users upload content (US; NEEDS-HUMAN)

---

## 2. SECURITY

**Auth**
- [ ] Proper authentication on every protected route and API (server-side checks, not just UI)
- [ ] Passwords hashed with Argon2id (or bcrypt/scrypt if Argon2 unavailable) — never plain/MD5/SHA; migrate legacy plaintext on next login
- [ ] Password reset links single-use and expire (≤ 1 hour)
- [ ] Admin routes locked down (role check server-side, ideally MFA)
- [ ] Authorization checks per resource (no IDOR — users can't read others' records by changing an id); for Supabase, RLS enabled on every table
- [ ] No default/demo accounts with known passwords in production

**Secrets**
- [ ] No API keys in client code or repo; secrets only in env vars on the server
- [ ] Check which env vars are exposed to the browser (`NEXT_PUBLIC_`, `VITE_`, `EXPO_PUBLIC_`, Vite `define`)
- [ ] Scan git history for leaked secrets (gitleaks / trufflehog)
- [ ] Revoke and rotate any exposed keys (NEEDS-HUMAN confirmation)
- [ ] Check for exposed files: `.env`, `.git/`, backups, SQL dumps in `public/`, source maps, `/debug`, admin panels
- [ ] No example secrets that work as real fallbacks (e.g. default JWT secret)

**Input & data**
- [ ] Parameterized queries / ORM everywhere — no string-built SQL (SQL injection)
- [ ] Validate all form and API input server-side (schema validation e.g. zod)
- [ ] Escape output / no unsafe `dangerouslySetInnerHTML` / `innerHTML` (XSS)
- [ ] CSRF protection on cookie-authenticated state-changing requests
- [ ] Restrict file uploads: type allow-list, size limit, random names, no execution, private storage
- [ ] Verify prices and totals on the server — never trust client-sent prices
- [ ] Multi-tenant apps: tenant id always taken from the session, never from the request body/query

**Transport & headers**
- [ ] Enforce HTTPS (redirect + HSTS)
- [ ] Secure cookies: `HttpOnly`, `Secure`, `SameSite=Lax/Strict`
- [ ] Restrict CORS (HTTP and WebSocket) to known origins (no `*` with credentials)
- [ ] Security headers: CSP, X-Content-Type-Options, X-Frame-Options/frame-ancestors, Referrer-Policy, Permissions-Policy
- [ ] Rate limiting on login, signup, reset, OTP, AI and expensive endpoints

**Ops**
- [ ] Debug mode / verbose errors / stack traces off in production
- [ ] Update dependencies; run `npm audit` / `pip-audit`; fix high/critical
- [ ] Monitoring and logging of auth events and errors (no secrets/PII in logs) — see section 4
- [ ] Final security review pass of all changes

---

## 3. SPEED / PERFORMANCE

**Perceived speed**
- [ ] Skeleton loading screens instead of blank pages/spinners
- [ ] Optimistic UI — update before the server responds, roll back on error
- [ ] Stream AI responses token by token
- [ ] Debounce search and autocomplete (≈250–300 ms)

**Frontend**
- [ ] Images in WebP/AVIF, correctly sized, `width/height` set
- [ ] Lazy-load images below the fold
- [ ] Server-render or statically generate pages where possible (SSR/SSG/ISR)
- [ ] Split JavaScript by route; dynamic-import heavy components
- [ ] Load third-party scripts after the page (`defer`/`async`/after-interactive); no dev-only CDNs (e.g. Tailwind Play CDN) in production
- [ ] Virtualize long lists
- [ ] Prevent unnecessary re-renders (memo, stable keys/callbacks, state placement)
- [ ] Self-host fonts, limit weights, `font-display: swap`, preload the main one
- [ ] Remove unused dependencies; check bundle size

**Network & API**
- [ ] Prevent duplicate API calls (dedupe/cache with React Query/SWR, abort stale requests); no polling that re-downloads full datasets
- [ ] Run independent requests in parallel (`Promise.all`)
- [ ] Paginate API results
- [ ] Timeouts (and retries with backoff) on all external API calls
- [ ] Brotli or Gzip compression enabled
- [ ] Serve static files through a CDN with long-lived browser caching (hashed filenames)

**Backend & database**
- [ ] Fix N+1 queries (joins / batch loading)
- [ ] Add indexes for common filters, joins and sorts
- [ ] Select only needed columns (no `SELECT *`)
- [ ] Database connection pooling (e.g. Supabase pooler / PgBouncer for serverless)
- [ ] Cache frequently read data (Redis / edge cache) with sensible TTLs
- [ ] Move slow tasks (emails, image processing, AI batch, exports) to a background job queue
- [ ] Place server/functions in the same region as the database
- [ ] Load balance across multiple instances when traffic needs it

**Measure**
- [ ] Trace slow backend requests (OpenTelemetry or host tracing)
- [ ] Real-user performance monitoring (Core Web Vitals: LCP, INP, CLS — e.g. Vercel Speed Insights)
- [ ] Test on slow phones and throttled networks (Lighthouse mobile, 4G throttling)
- [ ] Record before/after numbers in the report

---

## 4. SECURITY MONITORING & ADMIN ACCESS CONTROL

**Security logs (backend + frontend)**
- [ ] Backend audit log table (append-only): timestamp, tenant, user id, role, IP, user agent, event type, outcome, details (no passwords/tokens/PII beyond what is needed)
- [ ] Log at minimum: login success/failure, logout, token refresh, password change/reset, role/permission change requests + approvals/rejections, user create/disable/delete, admin actions, data export, rate-limit hits (429), authorization denials (401/403), cross-tenant attempts, passkey/SSO events
- [ ] Frontend security/error log endpoint (authenticated, rate-limited, size-capped): JS errors, CSP violations (`report-to`/`report-uri`), failed auth responses, suspicious client events
- [ ] Retention limit on logs (e.g. 90–180 days) with scheduled cleanup; logs mentioned in the privacy policy

**Security Logs screen in the app admin interface**
- [ ] A "Security" page inside the app's admin panel: filter by date, user, event type, outcome; refresh; export CSV
- [ ] Visible to the app owner/admin only. Other users (even other admins) see it only after the owner grants access, and granting requires re-entering the owner's admin password + passkey
- [ ] Server-side permission check on the logs API (never UI-only); every view/export of the logs is itself logged

**Strong admin authentication**
- [ ] Passkeys (WebAuthn) mandatory for app admins / owner on every app; recovery path defined (backup passkey or recovery codes)
- [ ] Step-up auth (admin password + passkey) required for sensitive actions: grant/revoke log access, role changes, deleting users/data, exports
- [ ] SSO (OpenID Connect / SAML: Google, Microsoft Entra ID, etc.) available for business tenants; map IdP groups to roles; still require passkey or IdP MFA for admins

**Role changes need approval**
- [ ] Adding or removing a role/permission creates a pending request; it takes effect only after an admin approves with password + passkey (requester cannot self-approve)
- [ ] Approval/rejection, approver, and timestamps recorded in the audit log; requester and owner notified

If passkey/SSO packages cannot be installed in the current session (blocked registry), build the logs + approval flow now and mark passkey/SSO NEEDS-HUMAN with the exact packages and config steps (e.g. `@simplewebauthn/server` + `@simplewebauthn/browser`; `openid-client` or the host's auth provider).

---

## Report template

```
# Scan and Speed report — <app> — <date>
Stack: ...
Summary: X fixed, Y need you, Z N/A

## Critical (fix now)
...
## Legal | Security | Speed | Monitoring & Access tables
| Item | Status | Evidence | Change / Owner action |
## Owner to-do (can't be done in code)
- Sign DPA with <vendor> ...
- Lawyer review of Privacy Policy / ToS drafts ...
## Performance before → after
```

Not legal advice: generated policies are starting drafts; the owner should have them reviewed for their jurisdiction (e.g. UAE PDPL, GDPR, CCPA).