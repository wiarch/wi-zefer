# WI Zefer — WI Arch security review

**Review date:** 2026-11-06  
**Scope:** repository metadata, upstream lineage, selected source files, security documentation and GitHub workflow configuration.  
**Limitations:** static source review only. No local install, runtime test, fuzzing, penetration test, dependency audit execution or independent cryptographic review was performed.

## Repository lineage and synchronization

- WI Arch repository: [wiarch/wi-zefer](https://github.com/wiarch/wi-zefer), public, default branch `main`.
- Upstream: [carrilloapps/zefer](https://github.com/carrilloapps/zefer).
- The fork had no commits ahead of upstream and was 10 commits behind.
- PR [#1](https://github.com/wiarch/wi-zefer/pull/1) synchronized the upstream commits. It was merged on 2026-11-06 as `47f518566d80b17dd21d89481fcdf44d15bac7b7`.
- The synchronization included upstream v0.12.0 security fixes, CI, security scanning, Dependabot and documentation updates.
- The upstream security release reports fixes for 14 vulnerable dependencies (including a critical Next.js advisory), a ReDoS issue, shell execution risk and other static-analysis findings. This is the upstream maintainer's release statement, not an independent verification by WI Arch.

## What the project does

Zefer is a browser-side encryption application. It encrypts text and files into versioned `.zefer` containers using Web Crypto API primitives (AES-256-GCM and PBKDF2-SHA-256). It also provides password generation/analysis, optional sharing controls and an upstream CLI/MCP companion.

It is useful as a local encryption and secure-file-sharing tool. It does not provide application identity, backend authorization, session management, tenant isolation, database key management, WAF or infrastructure security.

## Strengths observed

- Uses Web Crypto API for AES-GCM and PBKDF2-SHA-256.
- Generates random salts and IVs with `crypto.getRandomValues`.
- Uses a per-file derived AES key and per-chunk IV derivation for large files.
- Uses authenticated encryption, so ciphertext tampering is detected.
- Has backward-compatibility tests for file formats and chunk handling.
- Current upstream adds CI on Node 20 and 22 for lint, typecheck, tests with 100% configured coverage thresholds and production build.
- Current upstream adds Semgrep, Trivy, Gitleaks and optional SonarQube reporting; GitHub Actions are pinned to full commit SHAs.
- Dependabot updates npm and GitHub Actions dependencies.

## Risks and follow-up

| Priority | Observation | Why it matters | Recommended remediation |
|---|---|---|---|
| High | Public header's `iterations` value is parsed as untrusted input and passed to PBKDF2 during decryption | A malicious file may request an excessive KDF workload and freeze or exhaust the browser | Validate header schema and enforce a documented allowlist/range before calling Web Crypto |
| High | Binary header length, reveal-block size and chunk length fields are parsed from the file | Malformed sizes can trigger excessive reads/allocations or inconsistent parsing | Enforce maximum header size, validate all offsets and block boundaries, require chunk lengths within remaining bytes, reject trailing/truncated data |
| High | Decryption accepts passphrase/answer query parameters | Secrets can reach the origin in the initial HTTP request, server logs, browser history/sync and monitoring before client-side URL cleanup | Remove/deprecate secret-in-URL support; use a separate secure input/channel |
| Medium | Optional IP restriction calls multiple third-party IP services | The user's public IP is disclosed to external providers; this is not a private local-only feature | Explain disclosure, make opt-in, minimize providers and do not use it as authorization |
| Medium | IP/VPN detection is heuristic and client-side | A modified client can bypass it; IP heuristics are not reliable VPN detection | Treat as convenience only; avoid presenting it as a security boundary |
| Medium | Security scan runs monthly and scan commands continue on error to upload SARIF | Findings are reported but do not automatically block merges/releases | Add timely scans and define release-blocking thresholds after tuning |
| High | GitHub API reports default branch protection disabled | CI may run but a direct push can bypass review and required checks | Enable default-branch protection and require PR, review and CI checks |
| Medium | No restrictive CSP was present in the reviewed Next.js response-header configuration | CSP can reduce the impact of injection and compromised script sources | Design and test a compatible CSP; do not ship a guessed policy |
| Medium | Browser code is delivered by a web origin | A compromised origin or dependency can ship code that captures passphrases/plaintext before encryption | Document trusted-client assumption; protect deployment and supply chain; consider reproducible/signed releases |

## Interpretation of security claims

- AES-256-GCM and PBKDF2 are established primitives, but their use does not by itself prove the entire application is secure.
- Client-side encryption means the server is not intended to receive plaintext or keys during normal operation. It does not protect against malicious JavaScript delivered by a compromised origin, XSS, malicious extensions or a compromised endpoint.
- Expiration, IP restrictions and max-attempt counters enforced only by the client are convenience controls, not strong access-control guarantees.
- A 100% code coverage threshold is useful evidence of exercised code paths, not proof of absence of vulnerabilities.
- The upstream security scan release notes are not a third-party audit or cryptographic certification.

## WI Arch decision

Adopt WI Zefer as the **central reference repository for security engineering standards, encrypted-file tooling and security review practices**. Adopt [WI-ARCH-SECURITY-BASELINE.md](WI-ARCH-SECURITY-BASELINE.md) and [WI-ARCH-SECURITY-ADOPTION.md](WI-ARCH-SECURITY-ADOPTION.md) across the ecosystem.

Do not integrate the .zefer encryption format into application databases or authentication flows without a separate threat model, key-management design, compatibility plan and security review.

## Next actions

1. Enable branch protection/rulesets on `main`.
2. Add strict KDF and file-format parser bounds with adversarial tests.
3. Remove secret-bearing URL parameter support.
4. Document IP-provider disclosure and downgrade IP checks to non-security convenience.
5. Design CSP and release/deployment hardening.
6. Apply the WI Arch baseline to the remaining repositories, beginning with authentication, tenant isolation, plaintext secrets and backups.
