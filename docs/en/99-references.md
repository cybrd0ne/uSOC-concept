# 99 — References

This document collects the standards, frameworks, threat-intelligence sources and
open-source tools referenced by the µSOC concept, together with a note on the
academic and technical sources from which the concept derives.

---

## Standards & frameworks

| Standard / framework | Role in the µSOC | Reference |
|----------------------|------------------|-----------|
| **ECS — Elastic Common Schema** | The unified normalization schema adopted by the µSOC; defines consistent JSON field names for security, network, host and application events | <https://www.elastic.co/docs/reference/ecs> |
| **OCSF — Open Cybersecurity Schema Framework** | Alternative open schema considered during normalization design; broader cloud focus, limited support in the open-source SIEM stack | <https://schema.ocsf.io/> |
| **MITRE ATT&CK** | Adversary tactics/techniques taxonomy used to annotate and map detection and response coverage | <https://attack.mitre.org/> |
| **Sigma** | Platform-independent detection-rule standard enabling portable, documented detection logic (with embedded user-facing response context) | <https://github.com/SigmaHQ/sigma> |
| **STIX 2.1** | OASIS standard for representing and sharing indicators of compromise; ingested as threat-intel feeds | <https://docs.oasis-open.org/cti/stix/v2.1/os/stix-v2.1-os.html> |
| **CIS Benchmarks** | Configuration-hardening baselines referenced by the "security by default" principle | <https://www.cisecurity.org/cis-benchmarks> |
| **ACME / Let's Encrypt** | Automated TLS certificate issuance/renewal for the Router SOHO's exposed interfaces | <https://letsencrypt.org/> |
| **CA/Browser Forum** | Governs certificate transparency; basis for the caveat to use public CAs only for publicly exposed services | <https://cabforum.org/> |
| **GDPR / NIS2** | Regulatory context driving data-minimization, local-first processing, and (for NIS2) local threat-intel sharing considerations | — |
| **MCP — Model Context Protocol** | Open protocol standardizing how AI agents access context and tools; used by the *optional* Level 2 response orchestration | <https://modelcontextprotocol.io/> |

---

## Threat-intelligence sources

The µSOC ingests threat intelligence **inbound-only**, into the Router SOHO
(reputation feeds) and the SIEM device (STIX2 / local aggregator). Representative
sources referenced by the concept:

| Source | Indicator type | Notes |
|--------|----------------|-------|
| **Emerging Threats** (open blocklists) | IP / domain | Aggregated from multiple sources (DShield, Abuse.ch), updated daily |
| **Spamhaus DROP / EDROP** | IP blocks | Address blocks owned/leased by criminal organizations; strong C2 / residential-proxy coverage |
| **Abuse.ch** — Feodo Tracker, URLhaus, MalwareBazaar | IP / URL / file hash | Active malware infrastructure (Emotet, TrickBot, Cobalt Strike, QakBot) |
| **AlienVault OTX** | IP / domain / hash / URL | Community Open Threat Exchange, accessible via REST API |
| **Q-Feeds (Community Edition)** | IP / domain | European OSINT feeds (phishing, botnets, malicious infrastructure) |
| **MaxMind GeoLite2** | Geo / ASN | GeoIP and ASN enrichment + selective geo-blocking |
| **MISP** | Aggregated IOCs | Optional local IOC aggregator exposing a REST API for SIEM enrichment; keeps TI context local (relevant to NIS2) |

---

## Tools referenced

The named open-source tools and their role in the µSOC. All are realized in the
open-source reference implementation at
[github.com/cybrd0ne/suru-foss](https://github.com/cybrd0ne/suru-foss).

| Tool | Role | Reference tier |
|------|------|----------------|
| **pfSense / OPNsense** | Open-source firewall OS hosting the Router SOHO | `tier1-perimeter/` |
| **pfBlockerNG** | IP/DNS reputation filtering + DNSBL | `tier1-perimeter/` + `tier2-telemetry/` |
| **Suricata** | Inline IPS / signature detection (preferred, multi-threaded) | `tier1-perimeter/` + `tier2-telemetry/` |
| **Snort / Talos** | Alternative IDS engine; Talos community rules | `tier2-telemetry/` |
| **Zeek** | Passive network telemetry (conn/dns/ssl/http/files) | `tier1-perimeter/` |
| **syslog-ng** | Edge log collection + JSON normalization + TLS forwarding | `tier1-perimeter/` |
| **Logstash** | SIEM-side residual transform + enrichment to ECS | `tier3-core/` |
| **OpenSearch + Dashboards** | SIEM data store + visualization | `tier3-core/` |
| **Filebeat** | Alternative/complementary log shipper | `tier3-core/` |
| **Ansible** | Agentless (SSH) configuration & remediation automation | `tier4-operations/` |

---

## Academic & technical sources

The µSOC concept derives from an underlying **systematic review** and
**architecture study** of SOHO cybersecurity — the two source works that this
repository documents. The full bibliography (large-scale router-security analyses,
post-pandemic SOHO measurement studies, NSM/IDS performance evaluations,
SOCaaS/MDR market and architecture analyses, collaborative/federated IDS research,
and threat advisories) is contained in those works.

Representative references drawn from the source works:

- Ye et al., *"Exposed by Default: A Security Analysis of Home Router Default
  Settings (and Beyond)"* — router default-settings security analysis.
- Freitas et al., *"Uncovering Hidden Risks in IoT Devices: A Post-Pandemic
  National Study of SOHO Wi-Fi Router Security"* — national SOHO measurement study.
- Souchay et al., *"Literature Review of Small Office–Home Office (SOHO) Networks
  Security from 2017–2022"* — SOHO security mapping study.
- Alsakran et al., *"Intrusion Detection Systems for Smart Home IoT Devices:
  Experimental Comparison Study"* — Snort/Suricata/Zeek resource comparison.
- Gyamfi & Jurcut, *"Intrusion Detection in Internet of Things Systems: A
  Review…"* — IoT NIDS / MEC / ML review.
- Wang et al., *"An Empirical Study on the Insecurity of End-of-Life (EoL) IoT
  Devices"* — EoL device-population vulnerability study.
- Vielberth et al., *"Security Operations Center: A Systematic Study and Open
  Challenges"* — SOC building blocks and open challenges.
- Threat advisories and reporting on **Volt Typhoon / KV botnet**, **TheMoon**,
  and **TP-Link / Quad7** campaigns targeting SOHO routers.

> For complete citation details, consult the underlying systematic-review and
> architecture works that this repository describes.

---

*Prev: [09 — Scenarios & Limitations](./09-scenarios-and-limitations.md) · Back to [00 — Overview](./00-overview.md)*
