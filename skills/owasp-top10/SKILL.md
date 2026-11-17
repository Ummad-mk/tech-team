---
name: owasp-top10
description: OWASP Top 10:2025 risk taxonomy as an operating checklist for security audits and code review — the ten critical web application security risks with their 2021 mapping, common weakness patterns, and prevention notes. Use when auditing, threat-modeling, or reviewing any code/config for security, when classifying findings by OWASP category, or when a task mentions OWASP, Top 10, appsec, or vulnerability triage.
---

# OWASP Top 10:2025 — audit taxonomy

Source of truth: the official document repo https://github.com/OWASP/Top10 (released version: [owasp.org/Top10/2025](https://owasp.org/Top10/2025/), CC BY 3.0). Use the list as a coverage checklist — every finding you report maps to a category; every category either has findings or a "checked and clear" note.

What changed 2021 → 2025: XSS was folded into Injection (no longer standalone); SSRF dropped off the list (still a real risk — treat it under A02/A10 reasoning); Supply Chain Failures broadened to the whole pipeline, not just vulnerable components; new A10 category for exceptional-condition handling.

## The list (with 2021 mapping)

### A01:2025 Broken Access Control (was A01:2021)
The most common critical risk. Patterns: IDOR (object IDs reachable without ownership checks), missing function-level authorization on "internal" endpoints, privilege escalation via role/parameter tampering, forced browsing to unlisted pages, CORS misconfiguration granting API access. Check every privileged route for its authorization check, not just the UI path.
**Prevention:** deny by default; enforce server-side per-request; object-level ownership checks; rate-limit and log repeated access failures.

### A02:2025 Security Misconfiguration (absorbs A05:2021)
Insecure defaults, incomplete/missing hardening, verbose errors and stack traces to clients, enabled debug endpoints, default accounts/credentials, unnecessary features, wrong cloud/container permissions, missing security headers. Configuration is attack surface: scan env files, Dockerfile/IaC, server and framework defaults.

### A03:2025 Software Supply Chain Failures (was A06:2021 Vulnerable and Outdated Components)
Broader in 2025: compromised packages and build pipelines, typosquatting, dependency confusion, malicious maintainer changes, abandoned dependencies with known CVEs, unpinned versions, post-install scripts in dependencies, secrets leaking into CI logs and images. Check lockfile/manifest drift and audit tooling output.

### A04:2025 Cryptographic Failures (was A02:2021 Sensitive Data Exposure)
Weak or absent crypto: plaintext storage/transit of sensitive data, outdated algorithms (MD5, SHA1, DES), hardcoded keys and IVs, nonce reuse, missing TLS or cert validation disabled, weak password hashing (not bcrypt/scrypt/argon2), predictable tokens.

### A05:2025 Injection (absorbs XSS from A03:2021)
SQL/NoSQL/ORM query injection, OS command injection, LDAP injection, and now XSS (stored/reflected/DOM), template injection, header injection. Hunt input entry → sink for every untrusted parameter, including "safe-looking" fields.

### A06:2025 Insecure Design (was A06:2021)
Missing or ineffective control design rather than implementation bugs: no rate limiting on expensive flows, trust placed in client-side validation, business-logic gaps (workflow bypass, negative amounts), missing defense-in-depth. Cannot be patched away — needs a design fix.

### A07:2025 Authentication Failures (was A07:2021 Identification and Authentication Failures)
Credential stuffing, weak password policy, missing MFA, session fixation and predictable tokens, JWT confusion/validation gaps, no step-up auth on sensitive ops, logout not invalidating sessions, missing brute-force protection.

### A08:2025 Software or Data Integrity Failures (was A08:2021)
Deserialization of untrusted data, unsigned updates/CI artifacts, trusting data across trust boundaries without verification, auto-update without signature checks, CI/CD pipeline trust gaps.

### A09:2025 Security Logging and Alerting Failures (was A09:2021 Logging and Monitoring Failures)
No logging of auth/access-control failures, logs without context, logs never monitored/alerted, sensitive data in logs, log injection, alert fatigue from noise. A breach you can't see is a breach that never ends.

### A10:2025 Mishandling of Exceptional Conditions (new in 2025)
Unhandled errors/exceptions that leak internals, fail-open authorization on error paths, race conditions on privileged operations, resource exhaustion on unbounded inputs, "temporarily disabled" protections that become permanent. Errors are code paths too — attackers reach them on purpose.

## How to use it

1. Audit against all ten categories as a checklist; report per category: findings, or "checked and clear" with what you looked for.
2. Map each finding to its category (plus file:line, exploit path, fix) so triage teams can roll up by OWASP bucket.
3. A finding without a plausible exploit path is a hardening note, not a vulnerability — same bar regardless of taxonomy.
4. When the underlying data matters, cite the source release: https://owasp.org/Top10/2025/ and the repo https://github.com/OWASP/Top10.
