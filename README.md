# 🛡️ Application Security Knowledge Base

A **one-stop playbook** for enterprises to **build and scale Application Security (AppSec) capabilities**.
This repository helps **Developers, DevOps, and Security teams** understand **who owns what** in the DevSecOps lifecycle and provides **practical tools, checklists, and references**.

## DevSecOps Ownership Model

See `01-devsecops-visual/devsecops-ownership.svg` for a simple ownership visual (open in a browser or preview).

- **Developers** → Write secure code, use pre-commit hooks, avoid vulnerable libraries.
- **DevOps** → Secure pipelines, enforce policies, manage secrets, automate SBOMs.
- **Security Team** → Define standards, threat modeling, risk assessments, red/blue team validation.

## Sections
1. Secure Coding Practices & Checklists (`02-secure-coding-practices/`)
2. Trusted Libraries (`03-trusted-libraries/`)
3. Standard Architectures (AWS, Microsoft, patterns/anti-patterns) (`04-architecture/`)
4. Mistakes to Avoid (`05-mistakes-to-avoid/`)
5. SBOM & Supply Chain Security (`06-sbom/`)
6. Pre-Commit Hooks (25+ curated list) (`07-precommit-hooks/`)
7. Other AppSec Practices (`08-other-appsec-practices/`)
8. References (`references.md`)

## Quick start
1. Unzip this bundle
2. Browse the `docs/` and `templates/` folders and copy into your project repos
3. Adopt the pre-commit hooks and CI examples as pipeline steps
4. Use SBOM steps at build time to track dependencies and supply chain

## Links (quick)
- OWASP Top 10: https://owasp.org/www-project-top-ten/
- OWASP ASVS: https://owasp.org/www-project-application-security-verification-standard/
- NIST SSDF: https://csrc.nist.gov/publications/detail/sp/800-218/final
- CycloneDX: https://cyclonedx.org/
- SLSA: https://slsa.dev/
- AWS Well-Architected Security Pillar: https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html
