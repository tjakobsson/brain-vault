---
type: permanent
status: developing
tags:
  - software-supply-chain
  - sbom
  - security
created: 2026-08-30T09:24:04.899625736+02:00
updated: 2026-08-30T09:24:04.899676402+02:00
---

A Software Bill of Materials, or SBOM, is a machine-readable inventory of the components in a piece of software. It records names, versions, suppliers, package identifiers, licenses, and dependency relationships. SPDX and CycloneDX are the main exchange formats.

==An SBOM tells us what software contains. It does not tell us whether the product is exploitable.== That requires vulnerability data plus product-specific context from [[Vulnerability Exploitability Exchange]].

> [!example] The useful chain
> `product -> SBOM component and version -> vulnerability database match -> VEX applicability -> action`

## What it enables

- Find every product containing a newly disclosed vulnerable component.
- Detect version skew and stale dependencies across projects.
- Review open-source licenses against organizational policy.
- Compare product releases and trace dependency changes.
- Give customers and auditors evidence about software composition.

## What it does not solve

An inventory goes stale unless it is generated for each release, attached to an identifiable artifact, and kept available to its consumers. A package match can also overstate risk. The vulnerable code may be unreachable, disabled, absent from the final artifact, or already patched.

> [!warning]
> Treating every SBOM-to-CVE match as an exploitable vulnerability creates alert noise. Suppressing matches without signed, reviewable justification creates the opposite problem.

The practical value appears when a system joins SBOMs with current vulnerability intelligence, VEX statements, provenance, and policy. [[BOMHort]] is one open-source example of such a governance layer.

## Sources

- [CISA: Software Bill of Materials](https://www.cisa.gov/sbom)
- [SPDX](https://spdx.dev/)
- [CycloneDX](https://cyclonedx.org/)
