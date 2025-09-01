# Secure Coding Practices — Universal Checklist (Developer + Reviewer + CI)

## Developer (pre-commit)
- Input validation and output encoding (framework-based).
- Use parameterized queries / ORM to prevent SQL injection.
- Sanitize or escape user-controlled HTML/JS (XSS protection).
- Use secure authentication libraries (OIDC, OAuth2, established SDKs).
- Do NOT write custom crypto; use vetted libraries.
- No hard-coded secrets; use env vars + vault.
- Run local SCA (pip-audit, npm audit, go list -m -u) before commit.
- Add unit tests for authentication, input validation, and error cases.
- Add logging but mask PII and secrets.
- Add feature flags for risky features behind safe defaults.

## Reviewer (PR)
- Verify security-related changes have tests.
- Verify no secrets in diffs.
- Validate new dependencies and transitive deps.
- Validate added config is secure-by-default (CORS, CSP, cookie flags).
- Confirm authz checks for new endpoints/resources.

## CI / Pipeline gates
- SAST (block on high severity).
- SCA (block on critical/defined threshold CVEs).
- Secrets scanning (block if leaked).
- SBOM generation and attach to artifact.
- DAST on pre-prod (optional for each release).
- IaC scanning for infra changes.
