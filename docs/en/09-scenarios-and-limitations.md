# 09 — Scenarios & Limitations

## A guaranteed minimum, not an exhaustive shield

In its standard installation configuration, the µSOC is designed to deliver a
**universal minimum guaranteed level of defensive protection** applicable to any
SOHO environment, without elements specific to the organization's line of
business. This minimum operates on two complementary planes:

- **At the network perimeter**, through the detection and blocking mechanisms of
  the Router SOHO (pfBlockerNG, Suricata/Snort IPS, Zeek — see
  [05 — Detection, Correlation & Response](./05-detection-correlation-response.md)).
- **At the operating-system level** of monitored endpoints, through the optional
  EDR agents.

The standard coverage targets the threats documented as recurrent in SOHO
environments — exploitation of EoL/out-of-date router firmware, command-and-control
traffic, botnets, DNS exfiltration, attacks on exposed services, and phishing —
i.e. the attack vectors with the highest prevalence in the literature (see
[01 — Problem & Threat Model](./01-problem-and-threat-model.md)).

However, **this generalist coverage is a starting point, not an exhaustive level
of protection.** The defensive efficacy of the µSOC is fundamentally conditioned
by the specific operational context of each SOHO organization — the context that
determines the real attack surface, the exposed applications, and the categories
of data processed. The standard configuration cannot anticipate and implicitly
cover the multitude of threat scenarios specific to each activity profile, which
introduces structural limitations that must be explicitly recognized and addressed
through activity-specific personalization.

---

## Activity-specific risk profiles

Different SOHO business areas generate distinct risk profiles, each with specific
software applications, proprietary data flows, and particular threat vectors that
are **not** covered implicitly by the Sigma rules and reputation feeds enabled in
the standard µSOC configuration.

| Activity profile | Specific threat vectors | Personalization required beyond the standard config |
|------------------|-------------------------|------------------------------------------------------|
| **Public web apps & e-commerce** | SQL injection, cross-site scripting (XSS), exploitation of vulnerable web components, credential stuffing against admin panels | Dedicated detection rules, application-specific HTTP/HTTPS inspection policy, and preferably a **Web Application Firewall (WAF)** in front of the exposed service — a component absent from the standard µSOC config |
| **Accounting / ERP / financial management** (Saga, WinMentor, SAP Business One, Odoo) | Exfiltration of financial data, unauthorized ERP-database access, unauthorized modification of accounting records; connections to banking/fiscal/public-institution services | Application-specific behavioural baselines and correlation rules adapted to the organization's workflows |
| **CRM / customer relationship management** | Bulk personal-data (PII) access and exfiltration — internal (malicious employee exporting the customer database) or external | Correlation across CRM application logs, network activity (exported data volume), and user authentication; GDPR-aligned bulk-access detection |
| **PII processing & GDPR** (medical, legal, HR, recruitment, third-party accounting) | Unauthorized access to personal-data files, transfers of personal data outside the perimeter | Prior data-asset classification and storage-location-specific monitoring rules; GDPR compliance reporting |
| **HR & contract management** | Exfiltration of employee personal/bank/medical data and confidential contract documents | File-access monitoring at the agent level, correlated with network activity, with alerting calibrated to the organization's own file types and storage locations |
| **Medical data processing** (individual practices, small clinics) | Targeted ransomware on medical systems, unauthorized access to patient records (highest sensitivity; GDPR + sectoral regulation) | Network segmentation (medical devices in a dedicated VLAN) and record-system-specific monitoring rules — significantly stricter than the standard profile |

---

## Escalation for incidents beyond µSOC capacity

The architecture explicitly acknowledges that some classes of security incident
exceed the response capacity of a system optimized for SOHO environments without
dedicated IT staff. For these scenarios, the user-interaction model includes a
**predefined escalation mechanism**, triggered automatically at configured
severity thresholds — for example, a critical event persisting after automatic
response, or confirmed lateral movement within the network.

The escalation mechanism notifies a **designated technical contact** (external IT
administrator or managed-service provider) and automatically exports an **incident
context package**:

- the correlated events,
- the relevant ECS-normalized logs,
- the history of automatic actions executed,
- the incident recommendations.

This enables a specialist to take over the investigation rapidly **without
requiring direct access to the SIEM device**.

---

## The necessity of post-install personalization

The limitations above are **not architectural deficiencies of the µSOC**; they
reflect a fundamental reality of cybersecurity: *no universal security solution
implicitly covers every risk profile without adaptation to the specific context.*

The standard µSOC configuration must be understood as a **security foundation** —
a guaranteed, verifiable minimum — on top of which each SOHO organization must
build an activity-specific personalization layer. This personalization requires a
structured post-install process:

1. **Inventory of critical assets and applications** — identify all software in
   use, internally and externally exposed services, sensitive-data storage
   locations, and data flows with third parties (suppliers, clients, public
   institutions).
2. **Classification of processed data** — identify the categories of sensitive
   data present (financial, PII, medical, trade secrets) and the systems that
   process them, as a basis for prioritizing monitoring rules.
3. **Definition of organization-specific behavioural norms** — calibrate anomaly
   detection against the normal behaviour of the organization's network and
   specialized applications, reducing false positives generated by legitimate
   activity.
4. **Addition of custom detection rules** — extend the detection set with
   threat scenarios specific to the organization's application profile, using
   **Sigma** for portability and standardized documentation.
5. **Configuration of specific response playbooks** — adapt the predefined
   playbooks to the organization's application topology and internal
   incident-response procedures, including technical contacts and the specific
   escalation chain.

The absence of this personalization step can reduce the µSOC to a generalist
monitoring tool, with limited coverage for threats specific to the organization's
operational context and a high volume of irrelevant alerts that can generate
**alert fatigue** and reduce the user's attention to genuinely significant events.

---

*Prev: [08 — Deployment Modes](./08-deployment-modes.md) · Next: [99 — References](./99-references.md)*
