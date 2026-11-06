# WI Arch Security Adoption Checklist

Use this checklist for every repository in the WI Arch ecosystem. A link to the central baseline is not evidence of implementation.

Reference: [WI-ARCH-SECURITY-BASELINE.md](WI-ARCH-SECURITY-BASELINE.md).

## Repository record

For each repository, record:
- Product owner and technical maintainer.
- Stack, runtime, hosting and deployment environments.
- Data categories handled (public, internal, confidential, financial, health, credentials).
- Authentication provider and authorization model.
- Database, object storage, integrations and external processors.
- Production secrets owner and rotation process.
- Backup location, retention and restore owner.
- CI workflows, required checks and release approver.
- Last security review date and unresolved risks.

## Universal release checklist

- [ ] No committed credentials, private keys, production data or backup files.
- [ ] Production configuration has no development fallback secrets.
- [ ] Authentication and recovery flows are rate-limited and use secure session/token lifecycle.
- [ ] Authorization is enforced server-side for every sensitive operation.
- [ ] User/tenant/resource ownership is tested against IDOR and cross-tenant access.
- [ ] Input validation, output encoding, CSRF and CORS are configured appropriately.
- [ ] Uploads and imports have size/type/content validation and private storage.
- [ ] HTTPS, secure cookies and security headers are enabled in production.
- [ ] Logs redact secrets and sensitive user data.
- [ ] Dependencies and lockfiles are maintained; alerts and updates are reviewed.
- [ ] CI runs tests, lint/type checks, build and applicable security scans.
- [ ] Default branch requires PR/review and passing status checks.
- [ ] Backups are encrypted, access-controlled and restore-tested.
- [ ] Incident response and private vulnerability reporting path are documented.
- [ ] README/security docs accurately describe current controls and limitations.

## Stack-specific minimums

### Laravel / PHP applications

- [ ] Use Laravel password hashing; never persist plaintext passwords or temporary plaintext copies.
- [ ] Protect web routes with authentication and policies/gates or equivalent authorization.
- [ ] Validate ownership of route-model bindings, nested resources, uploads, exports and background jobs.
- [ ] Keep CSRF protection enabled for session-authenticated web routes.
- [ ] Rate-limit authentication, password reset, OTP, invitation and sensitive endpoints.
- [ ] Use secure, HttpOnly, SameSite cookies and HTTPS in production.
- [ ] Use Eloquent bindings/parameterized SQL; review raw expressions and dynamic query construction.
- [ ] Protect mass assignment with explicit fillable/guarded rules and Form Requests.
- [ ] Encrypt secrets at rest with managed keys; do not store provider secrets as plaintext model fields.
- [ ] Review queues, scheduler, logs, storage/app, public storage links and backups for data leakage.
- [ ] Test policies, IDOR, tenant scopes, password reset, session revocation and data exports.

### React / Next.js / Node.js applications

- [ ] Keep secrets out of NEXT_PUBLIC_ and VITE_ variables and browser bundles.
- [ ] Avoid storing long-lived bearer tokens in localStorage; prefer secure server-managed sessions where possible.
- [ ] Configure CSP, frame protection, MIME sniffing, referrer and permissions policies.
- [ ] Validate all API input server-side and use schema validation for external payloads.
- [ ] Apply rate limits, body/time limits and safe error handling to API routes.
- [ ] Avoid dangerouslySetInnerHTML; sanitize unavoidable rich HTML with a maintained sanitizer.
- [ ] Restrict outbound fetch destinations to prevent SSRF.
- [ ] Validate file uploads and generated downloads.
- [ ] Audit service workers and caches so sensitive responses are never cached offline.
- [ ] Test authentication, authorization, XSS, CSRF, SSRF and route-level data access.

### Flutter / Dart mobile applications

- [ ] Store tokens and secrets in platform-backed secure storage (Keychain/Keystore), not shared preferences/plain files.
- [ ] Use TLS and validate API authorization on the server; certificate pinning only with a tested rotation plan.
- [ ] Do not embed service-role keys, private API secrets or signing keys in the app binary.
- [ ] Protect deep links, exported Android components, intent filters and universal links.
- [ ] Review screenshots, clipboard, notifications, app switcher previews and local caches for sensitive data.
- [ ] Enforce biometric checks as an additional local gate, never as a replacement for server authorization.
- [ ] Obfuscate/release-build appropriately and protect signing keys.
- [ ] Test token expiry/revocation, device loss, local data deletion and backup behavior.

### Tauri / Rust desktop applications

- [ ] Keep Tauri capabilities and permissions minimal; deny by default.
- [ ] Restrict IPC commands and validate every argument in Rust.
- [ ] Never expose arbitrary shell execution or unrestricted filesystem access to the webview.
- [ ] Use OS keychain/credential storage for secrets.
- [ ] Apply a restrictive CSP and avoid remote untrusted content in privileged webviews.
- [ ] Sign releases and protect signing certificates/keys.
- [ ] Validate update signatures and secure update channels.
- [ ] Test path traversal, IPC authorization, plugin permissions and local file access.

### Supabase applications

- [ ] Enable RLS on every table exposed through client APIs.
- [ ] Write explicit SELECT/INSERT/UPDATE/DELETE policies and test cross-user/cross-tenant isolation.
- [ ] Keep service-role keys exclusively server-side.
- [ ] Validate Edge Function inputs and authorize the caller for each action.
- [ ] Restrict storage bucket policies and signed URL lifetimes.
- [ ] Review Auth redirect URLs, MFA settings, session behavior and account recovery.
- [ ] Audit SQL functions, SECURITY DEFINER use, grants and search_path settings.
- [ ] Test RLS with multiple users and privileged roles, not only an admin account.

## Security evidence

For each checked item, link to one or more of:
- Source file or configuration.
- Automated test.
- CI workflow/status check.
- Deployment configuration or operational runbook.
- Security review or accepted risk record.

Do not mark a control complete based only on a README statement.

## Initial rollout order

1. Inventory all WI Arch repositories and classify data sensitivity.
2. Fix urgent, confirmed exposures (plaintext credentials, broken tenant boundaries, leaked keys).
3. Enable branch protection and required CI on production repositories.
4. Add stack-specific security tests and automated scanning.
5. Review backups, logs, secrets and third-party integrations.
6. Schedule periodic security reviews and incident-response exercises.
