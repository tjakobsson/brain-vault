---
type: literature
status: developing
tags:
  - software-supply-chain
  - sbom
  - vex
  - security-tools
  - kubernetes
created: 2026-08-30T09:24:04.900221987+02:00
updated: 2026-08-30T09:24:04.900341862+02:00
---

[BOMHort](https://docs.bomhort.dev/) is an Apache-2.0, Kubernetes-native platform for collecting and governing [[Software Bill of Materials|SBOMs]]. It was previously called SeeBOM.

==BOMHort is the operational layer around SBOMs, not an SBOM generator.== It turns stored inventories into searchable vulnerability, license, dependency, and cross-project views, then uses [[Vulnerability Exploitability Exchange|VEX]] to add product-specific exploitability decisions.

> [!summary] In one line
> Ingest SPDX or CycloneDX -> identify packages by PURL -> query OSV -> apply OpenVEX -> check license policy -> explore the results in an API or dashboard.

## What it combines

- SPDX 2.3, CycloneDX 1.0 through 1.7, and SPDX wrapped in in-toto envelopes
- OpenVEX statements with `not_affected`, `affected`, `fixed`, and `under_investigation` states
- OSV vulnerability lookups and a daily refresh for newly disclosed CVEs
- License classification, policy exceptions, and GitHub-based resolution of missing license data
- Dependency search, version-skew detection, affected-project views, and cluster-level summaries
- S3 or local-file ingestion, a REST API, and an Angular dashboard

## How the data moves

```text
SBOM and VEX files
        |
        v
ingestion watcher -> ClickHouse queue -> parsing workers
                                            |  parse SBOM or VEX
                                            |  resolve licenses
                                            |  query OSV
                                            v
                                      ClickHouse tables
                                            |
                          daily CVE refresh | REST API
                                            v
                                      Angular dashboard
```

The backend is written in Go and split into an ingestion watcher, parsing workers, an API gateway, and a CVE refresher. ClickHouse stores the inventories and derived findings. The components run locally with Docker Compose or on Kubernetes through Helm.

## Why VEX matters here

A raw vulnerability count is not a useful risk count. BOMHort matches package URLs against OSV, then applies OpenVEX statements at query time. Its dashboard defines effective vulnerabilities as total findings minus findings suppressed by VEX.

==Keeping VEX evaluation at query time lets a revised statement change the result without rebuilding every SBOM.== The original package match remains visible while the product-specific assessment changes.

> [!warning]
> "Suppressed by VEX" does not mean "the CVE disappeared." The trustworthiness of the result depends on who authored the VEX statement, why they chose the status, and whether the statement matches the exact product represented by the SBOM.

## Is it a software supply-chain platform?

Yes, with a narrower description: it is currently an **SBOM visualization and governance platform**. It covers inventory, known-vulnerability correlation, VEX, and license policy across projects and clusters.

It is not yet a complete supply-chain assurance system. The July 2026 roadmap still lists in-toto Witness integration and attestation verification as planned work. It also lists dependency health, EPSS, SBOM comparison, blast-radius search, and Cyber Resilience Act reporting for later phases.

> [!note] Current maturity
> The project is pre-1.0, with v1.0 targeted for October 2026 in the published roadmap. Its API, ClickHouse schema, and Helm values may still change. Authentication is available but disabled by default, so an exposed deployment needs deliberate configuration.

## Where it fits

| Layer | Question | BOMHort role |
| --- | --- | --- |
| Inventory | What is in the product? | Ingests and indexes SBOMs |
| Detection | Which components match known vulnerabilities? | Queries OSV by PURL and refreshes matches daily |
| Context | Does the vulnerability affect this product? | Applies OpenVEX statements |
| Governance | Is this acceptable under our policy? | Reports license violations and exceptions |
| Assurance | Can we prove artifact origin and integrity? | Partial today; verification is roadmap work |

The interesting part is the join. Any one feed is incomplete on its own. ==An SBOM platform becomes useful when it preserves the inventory evidence while continuously adding vulnerability, exploitability, and policy context.==

## Sources

- [BOMHort documentation](https://docs.bomhort.dev/)
- [Architecture](https://docs.bomhort.dev/docs/architecture/)
- [Getting started](https://docs.bomhort.dev/docs/getting-started/)
- [Roadmap](https://docs.bomhort.dev/docs/roadmap/), last updated 2026-07-15
- [BOMHort repository](https://github.com/seebom-labs/BOMHort)
