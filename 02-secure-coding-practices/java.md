# Java Secure Coding Quick Guide

- Use parameterized prepared statements (no string concatenation for SQL).
- Use Spring Security for authn/authz; avoid rolling your own.
- Use the Java Cryptography Architecture (JCA) with providers like BouncyCastle only when needed.
- Validate inputs using javax.validation / Hibernate Validator.
- Use OWASP Java Encoder for output encoding.
- Keep dependencies updated; use Maven/Gradle lockfiles and Dependabot/renovate.
