# SBOM Best Practices (CycloneDX / SPDX)

- Generate at build time for each artifact (container, wheel, jar).
- Use tools: Syft (Anchore), CycloneDX CLI, Syft+Grype for scanning.
- Store SBOMs alongside artifacts; sign them.
- Use SBOM to map CVEs -> affected artifacts for rapid response.
