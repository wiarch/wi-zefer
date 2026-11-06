# WI Arch Security Baseline

**Status:** initial engineering baseline, not a certification or independent audit.  
**Owner:** WI Arch maintainers.  
**Reference implementation:** [WI Zefer](https://github.com/wiarch/wi-zefer), derived from [carrilloapps/zefer](https://github.com/carrilloapps/zefer).

## 1. Purpose and scope

WI Arch contains products built with different stacks: Laravel/PHP, React/Node, Flutter/Dart, Rust/Tauri and Supabase. No single library can secure all of them. This document defines the minimum security requirements every WI Arch repository must adopt, with stack-specific guidance in [WI-ARCH-SECURITY-ADOPTION.md](WI-ARCH-SECURITY-ADOPTION.md).

WI Zefer is the reference project for client-side file/text encryption, secure random generation, password tooling and security engineering practices. It is **not** a universal authentication provider, server-side secret manager, KMS, identity platform, WAF, endpoint protection tool or substitute for application-level authorization.

A repository is not considered compliant merely because it links to this document or uses Zefer.

## 2. Security principles

1. **Deny by default:** every protected operation requires explicit authorization.
2. **Verify at the server boundary:** frontend visibility and client-side checks are never authorization.
3. **Least privilege:** users, services, CI jobs, tokens and database accounts receive only required access.
4. **Minimize sensitive data:** collect, transmit, retain and log only what is necessary.
5. **Secure defaults:** production must fail closed when secrets or security configuration are missing.
6. **Defense in depth:** authentication, authorization, validation, encryption, monitoring and recovery complement one another.
7. **Cryptographic humility:** use reviewed platform primitives and established libraries; do not invent protocols or claim guarantees beyond the threat model.
8. **Traceability:** sensitive actions and security decisions must be auditable without logging secret values.
9. **Recoverability:** backups and restore procedures are part of security, not optional operations.
10. **Honest status:** distinguish implemented, tested, deployed and independently reviewed controls.

## 3. Mandatory controls for every production repository

### Identity and authentication

- Hash passwords with a current, framework-supported password hashing algorithm and safe parameters. Never encrypt or store plaintext passwords.
- Use MFA for privileged/admin accounts wherever supported.
- Enforce secure password reset and account recovery flows; use single-use, expiring tokens.
- Set session/token expiry, rotation and revocation semantics. Logout must invalidate the server-side session or refresh token, not only delete a browser value.
- Store browser sessions in secure, HttpOnly, SameSite cookies where the architecture supports them. Avoid long-lived bearer tokens in localStorage.
- Rate-limit login, registration, password reset, MFA verification and sensitive endpoints.
- Never put passwords, access tokens, API keys, reset tokens or encryption passphrases in URLs.

### Authorization and tenant isolation

- Enforce ownership, role and tenant checks on the server for every read, write, download, export, import and background job.
- Apply deny-by-default authorization and explicit policy checks to sensitive operations.
- Do not trust user IDs, tenant IDs, shop IDs, role names or resource IDs supplied by clients.
- Test IDOR/BOLA, horizontal and vertical privilege escalation, cross-tenant access and mass assignment.
- Background jobs and scheduled commands must scope records explicitly; they cannot depend on a logged-in user context.
- Sharing must be explicit, revocable, scoped and auditable.

### Secrets and configuration

- Never commit .env files, private keys, service-role keys, OAuth secrets, database credentials, signing keys, production dumps or real user data.
- Keep secrets in the deployment platform's secret manager or an approved vault. Separate development, staging and production credentials.
- Production must fail closed if a required secret is missing or has a known development fallback.
- Rotate credentials after exposure. Removing a secret from the latest commit does not remove it from Git history, forks, caches or backups.
- Keep private credentials out of frontend bundles and variables with public prefixes such as VITE_ or NEXT_PUBLIC_.
- Use least-privilege service accounts and short-lived CI credentials where possible.

### Data protection and cryptography

- Classify data before choosing controls: public, internal, confidential, regulated/highly sensitive.
- Enforce HTTPS/TLS for all production traffic and secure cookie settings.
- Encrypt databases, object storage and backups at rest using managed platform capabilities where appropriate.
- Use authenticated encryption (AEAD) and cryptographically secure random generation from platform APIs. Never reuse an AES-GCM nonce with the same key.
- Use a vetted KDF for password-based encryption; store salts and versioned parameters with ciphertext. Validate KDF parameters and input sizes before expensive work.
- Keep encryption keys separate from the encrypted data and backups. Define rotation, recovery and revocation procedures.
- Do not implement custom cryptographic algorithms or protocols without expert review.
- Client-side encryption protects data only while the delivered client code and runtime are trusted. A compromised origin, XSS, malicious dependency, browser extension or endpoint can capture plaintext or keys.
- Document exactly which fields are encrypted. Do not claim full E2EE/zero-knowledge if metadata, logs, backups or other fields remain server-readable.
- Do not log plaintext, decrypted payloads, passwords, tokens, private keys, financial/medical records or encryption material.

### Input, output and API security

- Validate all input server-side with allowlists, strict types, bounds and domain-specific constraints.
- Use parameterized queries/ORM bindings. Never concatenate untrusted input into SQL or shell commands.
- Encode output for its context. Avoid unsafe HTML injection and unsafe deserialization.
- Apply CSRF protection to cookie-authenticated state-changing requests.
- Configure CORS with explicit trusted origins; do not use wildcard origins with credentials.
- Apply rate limits and request/body/time limits to public endpoints.
- Defend against SSRF: validate outbound destinations, block private/link-local/metadata IP ranges and avoid arbitrary user-controlled fetch URLs.
- Uploads require size limits, content/signature checks, safe filenames, private storage and controlled parsing. Do not trust extensions or client MIME types.
- Add a restrictive Content Security Policy and security headers appropriate to the framework; test before rollout to avoid breaking legitimate app behavior.

### Dependencies and software supply chain

- Commit lockfiles and use deterministic CI installs (npm ci, composer install from lockfile, pub get --enforce-lockfile where supported).
- Enable Dependabot or an equivalent automated dependency alert/update process.
- Review direct and transitive dependency advisories; prioritize reachable critical/high vulnerabilities.
- Pin GitHub Actions to full commit SHAs. Give workflows minimum permissions and avoid running untrusted PR code with secrets.
- Scan source, dependencies, secrets, containers and infrastructure configuration.
- Review generated artifacts and release checksums/signatures where applicable.
- Keep forked projects synchronized with upstream security fixes and document local patches.

### CI/CD and repository governance

- Protect the default branch. Require pull requests, at least one review for production code, and required CI checks.
- Prevent force-push and deletion of the default branch.
- Require tests, type checks/lint, build and relevant security gates before merge.
- Do not let a report-only scanner be mistaken for a blocking security gate. Triage findings and define which severities block releases.
- Use CODEOWNERS for security-sensitive paths when maintainers/teams are defined.
- Restrict GitHub Actions permissions to read by default; grant write permissions only to the specific job that needs them.
- Protect production deployment environments with approvals and scoped secrets.
- Never expose secrets to workflows triggered by untrusted fork pull requests.

### Logging, monitoring and incident response

- Log security-relevant events: authentication failures, privilege changes, sensitive exports, billing/security setting changes and administrative actions.
- Use structured logs, but redact tokens, secrets, personal/medical/financial values and request bodies.
- Alert on unusual login activity, privilege escalation, repeated failures, failed backups and unexpected deployment changes.
- Maintain a private vulnerability disclosure path and a documented incident-response owner.
- Preserve evidence securely, rotate compromised credentials, assess affected users and document remediation.

### Backups and continuity

- Backups must be access-controlled, encrypted at rest and protected from public web access.
- Define retention and deletion policies. Apply the same protection to exports, snapshots and replicas.
- Test restore procedures regularly and document recovery objectives.
- Do not send production backups through ordinary chat, email or issue trackers.

## 4. Zefer-specific role and limitations

Zefer provides browser-side encryption of text/files into password-protected .zefer containers using Web Crypto (AES-256-GCM and PBKDF2-SHA-256), plus password generation/analysis and a CLI/MCP companion in the upstream ecosystem.

Appropriate WI Arch uses:
- Encrypting sensitive files before transfer or storage in a location not trusted to hold plaintext.
- Protecting offline exports and selected configuration artifacts.
- Generating high-entropy passphrases locally.
- Inspecting and decrypting .zefer files in controlled workflows.

Do not use Zefer as:
- A replacement for authentication, authorization, sessions, tenant isolation or TLS.
- A drop-in database field encryption system without a separately reviewed integration and key lifecycle.
- A server-side KMS, secrets manager or password manager.
- A way to make a compromised web origin safe.
- Proof that an application is secure or compliant.

Never pass secrets as shell command-line arguments in automation if they can appear in shell history or process listings. Prefer an approved secret store and a supported secure input mechanism.

## 5. Current Zefer review findings to track

These are source-review findings and hardening priorities, not a completed penetration test:

| Priority | Finding / risk | Required action |
|---|---|---|
| Critical for governance | WI Zefer main was unprotected when reviewed | Enable default-branch protection and require CI checks/review |
| High | Public file header controls PBKDF2 iterations | Validate type and enforce a strict supported range before KDF work |
| High | Binary header and chunk lengths are attacker-controlled | Bound header size, block sizes, chunk count and chunk lengths; reject trailing/truncated data |
| High | Decrypt URL parameters accept passphrases/answers | Remove/deprecate secret-in-URL support; clearing history after load is not sufficient |
| Medium | IP restriction contacts multiple third-party IP services | Disclose the data transfer and make it opt-in; do not treat client-side IP checks as access control |
| Medium | Security scanners run monthly and use report-only continuation | Add timely scanning and define release-blocking thresholds after tuning |
| Medium | No restrictive CSP was visible in Next.js headers | Design and test a CSP compatible with the application; do not ship a guessed policy |
| Medium | Browser-delivered code can be modified by a compromised origin/dependency | State the trusted-client assumption; consider signed/reproducible releases and deployment controls |

## 6. Severity and release policy

- **Critical:** active compromise, remote code execution, unauthenticated access to sensitive data, exposed production secrets, cross-tenant bulk data exposure, or cryptographic failure affecting confidentiality. Stop release/deployment and contain immediately.
- **High:** exploitable authorization bypass, sensitive data exposure, account takeover, injection with meaningful impact, or high-severity reachable dependency vulnerability. Block release until fixed or formally risk-accepted by the security owner.
- **Medium:** meaningful defense-in-depth weakness with limited exploitability or impact. Assign an owner and remediation date; document any exception.
- **Low:** hardening or low-impact issue. Track and resolve through normal maintenance.

A scanner's severity alone is not a substitute for exploitability and impact assessment. Exceptions must include owner, rationale, compensating control and expiry date.

## 7. Adoption and compliance

Use [WI-ARCH-SECURITY-ADOPTION.md](WI-ARCH-SECURITY-ADOPTION.md) for the per-repository checklist. Each repository must maintain its own security notes and demonstrate controls with tests, configuration or deployment evidence.

This baseline is a living engineering standard. Revisit it after incidents, major architecture changes, new threat intelligence or an independent security review.
