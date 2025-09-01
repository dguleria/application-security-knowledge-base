# Python Secure Coding Quick Guide

- Use parameterized queries for DB (e.g., psycopg2 placeholders).
- Use 'cryptography' library for crypto needs; avoid hashlib-only solutions for complex uses.
- Use Pydantic for input validation where possible.
- Use secure frameworks (Django/Flask with secure config) and enable built-in protections.
- Run pip-audit and safety in CI for SCA.
