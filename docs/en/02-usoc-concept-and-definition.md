# 02 — The µSOC Concept & Formal Definition

This document defines the **micro-SOC (µSOC)**: how it differs from a traditional
SOC and from outsourced SOCaaS/MDR models, and its formal specification. It builds
directly on the problem and threat model of document
[01](./01-problem-and-threat-model.md) and is elaborated architecturally in
document [03](./03-architecture-overview.md).

## 1. What a traditional SOC is

A **Security Operations Center (SOC)** is an organizational unit that combines
**people, processes and technology** to deliver continuous monitoring, detection,
investigation and response for security events across an enterprise (concept paper
§2.3). Typical SOC functions include:

- log collection and normalization;
- correlation and alerting, usually through a **Security Information and Event
  Management (SIEM)** platform;
- threat-intelligence integration;
- incident-response coordination;
- reporting for compliance and risk management.

These functions assume scale: an analyst team, an enterprise event volume, and an
infrastructure rich in identity and endpoint telemetry.

## 2. SOCaaS and MDR — and why they are misaligned with SOHO

Over the last decade, **SOC-as-a-Service (SOCaaS)** and **Managed Detection and
Response (MDR)** emerged so organizations could outsource some or all SOC
functions (concept paper §2.3, §4.3):

- **SOCaaS** generally denotes a fully managed SOC delivered through a
  cloud-based subscription — monitoring, detection, incident handling, often
  compliance reporting.
- **MDR** focuses more specifically on advanced threat detection and hands-on
  incident response, typically leveraging endpoint telemetry and threat hunting.

Both have been adopted successfully by SMBs, but they are **poorly aligned with
SOHO realities**:

- **Cost** — services commonly start at several thousand USD per month even for
  relatively small environments; MDR often bills per endpoint (≈ $15–30 /
  endpoint / month), and SOCaaS emphasizing log ingest and SIEM can be costlier
  still.
- **Onboarding assumptions** — identity providers, centralized directory
  services, and fleet-management tooling for agent deployment.
- **Expected data sources** — endpoint telemetry, Active Directory and cloud
  platform logs, VPN concentrators, line-of-business applications — frequently
  absent or minimal in a SOHO.
- **Router not first-class** — the consumer router, the primary SOHO network-edge
  artifact, is rarely central to these architectures, which focus on server and
  endpoint telemetry.

A SOHO site typically has a small number of endpoints, a minimal IT budget, and no
staff to engage in complex onboarding or to interpret detailed SOC reports. The
data-collection models presumed by SOCaaS/MDR often exceed what is both
technically feasible and privacy-acceptable in a SOHO context.

## 3. The µSOC as a distinct type of SOC

The **µSOC** is introduced (concept paper §2.3; architecture §6.1.1) as a
specialized SOC type for SOHO environments, characterized by four properties:

- **Narrow scope** — a single SOHO network with a small number of users and
  devices.
- **Minimal-but-coherent function set** — edge telemetry collection;
  signature- and anomaly-based detection for SOHO-relevant threats; prioritized
  alerting; and a small library of **guided response playbooks**.
- **Hybrid deployment model** — lightweight sensors and agents operate in the
  SOHO environment, while correlation, analytics and longer-term storage are
  provided locally (and, optionally, through a cloud platform).
- **Strict usability and privacy constraints** — near-zero-touch deployment,
  simplified user interfaces, and explicit data-minimization strategies.

The µSOC can be seen as a SOHO-optimized specialization of SOCaaS, but the
combination of scope, function set, deployment model and constraints justifies
treating it as a **distinct type of SOC** rather than a scaled-down enterprise
SOC.

### Design principles

Architecturally, the µSOC is dimensioned around five design principles
(architecture §6.1.1), detailed in document [03](./03-architecture-overview.md):
**security by default**, **data minimization & local-first privacy**,
**near-automatic operation**, **minimal hardware**, and **modularity &
flexibility**.

## 4. Formal definition

Integrating the components described above, the µSOC is defined formally
(architecture §6.1.5) as an integrated cybersecurity system for SOHO
environments, composed of three principal elements:

> ### µSOC = { R_SOHO, A_SIEM, E_EDR<sup>optional</sup> }

where:

- **R_SOHO** — the **Router SOHO**: an open-source firewall appliance providing
  firewall, **IDS/IPS** (Suricata / Snort), **IP/DNS reputation filtering**
  (pfBlockerNG + DNSBL), and **passive telemetry aggregation** (Zeek).
- **A_SIEM** — the **local SIEM device** (Docker / VM) providing collection,
  normalization, correlation and alerting.
- **E_EDR<sup>optional</sup>** — **optional EDR agents** installed on managed
  endpoints, contributing host-level telemetry and, where supported, active
  response.

Two optional layers **compose** with the core without changing its internal
architecture:

- the optional **cloud-protection** component (e.g. Cloudflare Free) **overlaps
  R_SOHO**, extending the filtering perimeter outside the local network;
- the optional **AI-augmentation** component **overlaps A_SIEM**, adding
  contextualization and guided-response generation.

Their absence does not reduce internal visibility or core detection capability —
the µSOC is designed to function fully in its base configuration.

## 5. Key components

The following table summarizes the key components, their functional role, and the
SOHO constraint each addresses (architecture §6.1.5, Table 2):

| Component | Functional role | SOHO constraint addressed |
|-----------|-----------------|---------------------------|
| **Router SOHO (firewall)** | Perimeter access control, NAT, VPN | Insecure firmware, permissive default settings |
| **Suricata / Snort (IDS/IPS)** | Real-time threat detection and blocking | Lack of traffic visibility, absent intrusion detection |
| **pfBlockerNG + DNSBL** | IP/DNS reputation filtering, C2 & ad-tracking blocking | Exposure to botnets and known C2 infrastructure |
| **Zeek** | Telemetry aggregation, minimal statistics (sessions, DNS, TLS) | Low data volume, local storage |
| **ACME / Let's Encrypt + Cloudflare** | Automated TLS certificate management | Incorrect TLS implementations, self-signed certificates |
| **SIEM device (Docker / VM)** | Collection, normalization, correlation, alerting | Absent SOC capability; data kept local |
| **EDR agents (optional)** | Endpoint visibility, active response, compliance | Partial coverage from network-only monitoring |
| **Cloudflare Free (optional)** | DDoS protection, external DNS filtering, TLS proxy | WAN exposure, volumetric attack |

This component set directly addresses the gaps identified in the SOHO security
literature: the absence of a natural monitoring point in standard topologies, the
dependence on local expertise, the prohibitive cost of SOCaaS for environments
without dedicated IT staff, and the structural tension between the visibility
required for security and the privacy of user data.

## 6. Where this leads

- The **architecture** of the four functional levels and the optional overlays is
  detailed in document [03](./03-architecture-overview.md).
- The mapping from each conceptual element to a concrete artifact in the
  open-source reference implementation (tiers `tier1-perimeter/`,
  `tier2-telemetry/`, `tier3-core/`, `tier4-operations/`; full runnable code at
  `github.com/cybrd0ne/suru-foss`) is given in document
  [07](./07-implementation-mapping.md).

---

*Prev: [01 — Problem & threat model](./01-problem-and-threat-model.md) · Next: [03 — Architecture overview](./03-architecture-overview.md)*
