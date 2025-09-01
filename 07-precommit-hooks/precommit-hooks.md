# Pre-commit Hooks — 25+ Curated Hooks (examples and tools)

## Basic hygiene
1. check-case-conflict
2. end-of-file-fixer
3. trailing-whitespace
4. check-yaml
5. check-json

## Security & secrets
6. detect-secrets (Yelp detect-secrets)
7. git-secrets
8. truffleHog (or trufflehog3)

## Dependency / SCA (local)
9. run-pip-audit (python)
10. npm-audit (node)
11. go vet / govulncheck (go)

## Linters / formatters / static checks
12. eslint (javascript)
13. flake8 / black (python)
14. go fmt / govet (go)
15. mvn -DskipTests=false -Dspotbugs (java/spotbugs)

## IaC checks
16. checkov / terrascan
17. tflint

## Tests & coverage
18. run-unit-tests
19. coverage-report-check

## Misc security checks
20. sops-validate (for encrypted secrets)
21. sbom-generate (generate SBOM for change)
22. license-check (detect forbidden licenses)
23. containerfile-lint / hadolint
24. compose-lint / dockerfilelint
25. vulnerable-deps-check (lightweight scan)

## Notes
- Use `pre-commit` framework (pre-commit.com) to orchestrate these hooks.
- Keep hooks fast; heavy scans can be gated to CI rather than pre-commit.
