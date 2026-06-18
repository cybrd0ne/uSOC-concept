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

| Tool | Role | Reference tier | Docs |
|------|------|----------------|------|
| **pfSense / OPNsense** | Open-source firewall OS hosting the Router SOHO | `tier1-perimeter/` | [pfSense](https://docs.netgate.com/pfsense/) · [OPNsense](https://docs.opnsense.org/) |
| **pfBlockerNG** | IP/DNS reputation filtering + DNSBL | `tier1-perimeter/` + `tier2-telemetry/` | [docs](https://docs.netgate.com/pfsense/en/latest/packages/pfblocker.html) |
| **Suricata** | Inline IPS / signature detection (preferred, multi-threaded) | `tier1-perimeter/` + `tier2-telemetry/` | [docs](https://docs.suricata.io/en/latest/) |
| **Snort / Talos** | Alternative IDS engine; Talos community rules | `tier2-telemetry/` | [docs](https://docs.snort.org/) |
| **Zeek** | Passive network telemetry (conn/dns/ssl/http/files) | `tier1-perimeter/` | [docs](https://docs.zeek.org/en/master/) |
| **syslog-ng** | Edge log collection + JSON normalization + TLS forwarding | `tier1-perimeter/` | [docs](https://www.syslog-ng.com/technical-documents/list/syslog-ng-open-source-edition) |
| **Logstash** | SIEM-side residual transform + enrichment to ECS | `tier3-core/` | [docs](https://www.elastic.co/guide/en/logstash/current/index.html) |
| **OpenSearch + Dashboards** | SIEM data store + visualization | `tier3-core/` | [docs](https://opensearch.org/docs/latest/) |
| **Filebeat** | Alternative/complementary log shipper | `tier3-core/` | [docs](https://www.elastic.co/guide/en/beats/filebeat/current/index.html) |
| **Ansible** | Agentless (SSH) configuration & remediation automation | `tier4-operations/` | [docs](https://docs.ansible.com/) |

---

## Academic & technical sources

The µSOC concept is grounded in the published literature on SOHO cybersecurity —
large-scale router-security analyses, post-pandemic SOHO measurement studies,
NSM/IDS performance evaluations, SOC/SOCaaS architecture studies, and threat
advisories. The works below are the substantive sources for the problem framing,
threat model, and design rationale used across documents 01–06.

### Peer-reviewed publications

1. J. Ye, X. de Carné de Carnavalet, L. Zhao, M. Zhang, L. Wu, and W. Zhang,
   "Exposed by Default: A Security Analysis of Home Router Default Settings,"
   in *Proc. 19th ACM Asia Conf. on Computer and Communications Security
   (AsiaCCS '24)*, Singapore, 2024, pp. 63–79.
   doi: [10.1145/3634737.3637671](https://doi.org/10.1145/3634737.3637671)
2. J. Ye, X. de Carné de Carnavalet, L. Zhao, M. Zhang, L. Wu, and W. Zhang,
   "Exposed by Default: A Security Analysis of Home Router Default Settings and
   Beyond," *IEEE Internet of Things Journal*, vol. 12, no. 2, pp. 1182–1199,
   2025. doi: [10.1109/JIOT.2024.3502405](https://doi.org/10.1109/JIOT.2024.3502405)
3. O. Freitas, F. Taffarel, A. L. dos Santos, and L. A. P. Júnior, "Uncovering
   Hidden Risks in IoT Devices: A Post-Pandemic National Study of SOHO Wi-Fi
   Router Security," *Journal of Internet Services and Applications*, vol. 15,
   no. 1, pp. 485–495, 2024.
   doi: [10.5753/jisa.2024.3834](https://doi.org/10.5753/jisa.2024.3834)
4. F. Alsakran, G. Bendiab, S. Shiaeles, and N. Kolokotronis, "Intrusion
   Detection Systems for Smart Home IoT Devices: Experimental Comparison Study,"
   in *Security in Computing and Communications (SSCC 2019)*, Communications in
   Computer and Information Science, vol. 1208, Springer, Singapore, 2020,
   pp. 89–104.
   doi: [10.1007/978-981-15-4825-3_7](https://doi.org/10.1007/978-981-15-4825-3_7)
5. E. Gyamfi and A. Jurcut, "Intrusion Detection in Internet of Things Systems:
   A Review on Design Approaches Leveraging Multi-Access Edge Computing, Machine
   Learning, and Datasets," *Sensors*, vol. 22, no. 10, art. 3744, 2022.
   doi: [10.3390/s22103744](https://doi.org/10.3390/s22103744)
6. D. Wang, M. Jiang, R. Chang, Y. Zhou, H. Wang, B. Hou, L. Wu, and X. Luo,
   "An Empirical Study on the Insecurity of End-of-Life (EoL) IoT Devices,"
   *IEEE Transactions on Dependable and Secure Computing*, vol. 21, no. 4,
   pp. 3501–3514, 2024.
   doi: [10.1109/TDSC.2023.3334017](https://doi.org/10.1109/TDSC.2023.3334017)
7. M. Vielberth, F. Böhm, I. Fichtinger, and G. Pernul, "Security Operations
   Center: A Systematic Study and Open Challenges," *IEEE Access*, vol. 8,
   pp. 227756–227779, 2020.
   doi: [10.1109/ACCESS.2020.3045514](https://doi.org/10.1109/ACCESS.2020.3045514)

### Reports & gray literature

8. G. Souchay et al., "Literature Review of Small Office–Home Office (SOHO)
   Networks Security from 2017–2022," Ingénieurs de l'Industrie, 2024.
   *(Engineering-school report; no canonical DOI.)*

### Threat advisories & incident reporting

9. U.S. Department of Justice, Office of Public Affairs, "U.S. Government
   Disrupts Botnet People's Republic of China Used to Conceal Hacking of
   Critical Infrastructure" (Volt Typhoon / KV Botnet), Jan. 2024.
   <https://www.justice.gov/archives/opa/pr/us-government-disrupts-botnet-peoples-republic-china-used-conceal-hacking-critical>
10. Lumen / Black Lotus Labs, "The Dark Side of TheMoon."
    <https://blog.lumen.com/the-darkside-of-themoon/>
11. Malwarebytes, "TP-Link Warns of Botnet Infecting Routers and Targeting
    Microsoft 365 Accounts" (Quad7 / 7777), Sep. 2025.
    <https://www.malwarebytes.com/blog/news/2025/09/tp-link-warns-of-botnet-infecting-routers-and-targeting-microsoft-365-accounts>

---

*Prev: [09 — Scenarios & Limitations](./09-scenarios-and-limitations.md) · Back to [00 — Overview](./00-overview.md)*
