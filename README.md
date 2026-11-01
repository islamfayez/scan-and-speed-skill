# Scan and Speed — a Claude skill

A [Claude skill](https://support.claude.com/en/articles/12512180-use-skills-in-claude) that audits and fixes any web or mobile app across four areas, then writes a report with what was fixed and what the owner still needs to do.

1. **Legal & compliance:** privacy policy, terms, consent and acceptance records, cookie and tracker blocking, data access and deletion, retention, dark patterns, honest claims, AI labelling, accessibility, licenses.
2. **Security:** authentication, password hashing (Argon2id / scrypt), secrets and exposed files, SQL injection / XSS / CSRF, tenant isolation, HTTPS and security headers, CORS, rate limiting, dependency audits.
3. **Speed:** skeletons, optimistic UI, AI streaming, images, code splitting, caching, compression, parallel requests, N+1 queries, indexes, pooling, background jobs, real-user monitoring.
4. **Security monitoring & admin access:** backend and frontend security logs, a Security Center in the app's admin area (owner-only, with access granted by owner password), mandatory passkeys, SSO, and approvals for adding, changing or removing roles.

Each item is marked **PASS / FIXED / NEEDS-HUMAN / N/A** with evidence. Fixes go on a `scan-and-speed/<date>` branch, and nothing is merged automatically.

## Install

**Claude (web, desktop, mobile):**
1. Download this repo as a ZIP, or zip the `scan-and-speed/` folder.
2. Go to **Customize > Skills**, click **+**, choose **Create skill**, then **Upload a skill**, and select the zip.

**Claude Code:** copy the `scan-and-speed/` folder into `~/.claude/skills/` (or `.claude/skills/` in a project).

## Use

Ask Claude: *"Run scan and speed on this app"* or *"speed and security check"*.

## Disclaimer

Not legal advice. Generated policies and legal items are starting drafts. Have them reviewed by a qualified lawyer for your jurisdiction (e.g. UAE PDPL, GDPR, CCPA).
