# 01 — Problem & Threat Model

This document establishes *why* SOHO environments are difficult to defend, *who*
attacks them, and *why* existing solutions fall short. It motivates the
requirements that the µSOC (document [02](./02-usoc-concept-and-definition.md))
is designed to satisfy.

## 1. SOHO at the edge of the enterprise

Small Office / Home Office (SOHO) networks have evolved from marginal setups used
by a few freelancers into **critical nodes in modern supply chains and
remote-work ecosystems** (concept paper §1). The COVID-19 pandemic and the
consolidation of hybrid-work models extended corporate perimeters into
residential spaces, effectively turning home networks into **micro-branches** that
connect employees, contractors and IoT devices directly to corporate resources
and cloud services.

The consequence is that the compromise of a single SOHO network can have impact
far beyond the individual user — reaching corporate assets and, in some cases,
critical infrastructure. The SOHO network is no longer a private concern; it is
an exposed segment of a larger trust chain.

## 2. Typical SOHO network architecture and why it resists monitoring

A typical SOHO network is built around a consumer or small-business router,
frequently supplied by the Internet Service Provider (ISP). A common architecture
consists of (concept paper §2.1):

- an **ISP customer-premises equipment (CPE)** combining modem and routing
  functions;
- a **consumer Wi-Fi router** performing NAT, basic firewalling and switching;
- ad-hoc additions — wireless access points, mesh nodes, powerline extenders —
  bolted on to improve coverage.

From a monitoring standpoint, this topology is hostile to security visibility:

| Property | Consequence for monitoring |
|----------|----------------------------|
| **Ubiquitous NAT, private addressing** | Host identities are masked behind a single public IP; external services cannot attribute malicious activity to a specific device. |
| **Flat broadcast domain** | Business laptops, personal phones, smart TVs, IP cameras, printers and IoT appliances typically share one segment; no segmentation. |
| **No natural tap point** | Standard topologies offer no span/mirror port or inline location where traffic can be observed. |
| **Limited router CPU / RAM** | Consumer routers are optimized for throughput and ease of use, not deep packet inspection or extensive logging. |
| **End-of-life (EoL) devices** | Many devices have reached EoL, no longer receive updates, yet remain Internet-connected for years. |
| **Attack-surface expanders** | UPnP, port forwarding, dynamic DNS and vendor cloud services widen the external attack surface. |
| **Pervasive encryption** | HTTPS, TLS-based application protocols and DNS-over-HTTPS reduce payload visible to edge inspection, forcing reliance on metadata. |

The combined effect: any security monitoring or response capability for SOHO must
operate under tight resource constraints and with limited control over the
underlying infrastructure.

## 3. Threat model: adversary classes

Drawing on threat-model analyses of home and SOHO routers, the underlying
academic work (concept paper §2.2) distinguishes four adversary classes relevant
to SOHO:

- **WAN adversaries (Adv_WAN)** — located on the Internet, they scan services
  exposed on the router's WAN interface (remote management, VPN daemons,
  file-sharing gateways) and exploit vulnerabilities, default credentials or
  outdated firmware to gain remote access.
- **LAN adversaries (Adv_LAN)** — compromised internal devices or malicious
  guests that attack the router and other devices from inside, bypassing
  perimeter protections.
- **Supply-chain and botnet operators** — actors who systematically compromise
  large numbers of SOHO routers and IoT devices, incorporating them into botnets
  used for DDoS, residential-proxy services, password-spraying, or as operational
  relay boxes in advanced campaigns.
- **Advanced Persistent Threat (APT) actors** — adversaries who compromise SOHO
  routers used by employees of critical-infrastructure operators or high-value
  organizations, in order to pivot into corporate networks or to disguise their
  activity as benign consumer traffic.

### Impact

Successful attacks span a spectrum:

- **Confidentiality** — traffic interception (weak Wi-Fi), unauthorized access to
  NAS/file shares with default credentials, behavioral inference from traffic
  metadata.
- **Integrity** — router/IoT compromise and configuration tampering (DNS
  hijacking, traffic redirection), malware installation on local hosts.
- **Availability** — exploitation that crashes the router, local DoS, repeated
  resets under brute-force attempts.
- **Systemic risk** — compromised SOHO devices become part of botnets, proxy or
  relay infrastructure used against third-party victims. For a SOHO connected by
  VPN to a head office, router or endpoint compromise enables **pivoting into the
  corporate network** and use of the link as a legitimate-looking tunnel for
  exfiltration or persistent access.

### Documented campaigns (threat context)

The literature and public advisories document this abuse concretely: the
**Volt Typhoon / KV botnet** proxied command-and-control traffic through
vulnerable SOHO routers into critical-infrastructure networks; **TheMoon** malware
targeted end-of-life home routers to build large proxy infrastructures; and
2024–2025 campaigns exploited **TP-Link** flaws (e.g. the **Quad7** botnet) for
password-spraying against Microsoft 365 accounts, forcing emergency firmware
releases even for officially end-of-life devices.

## 4. Insecure by design and by default

Empirical studies (concept paper §4.1, §5.4) confirm that many commercial devices
remain *insecure by design* and *insecure by default*. Recurring problems
include:

- weak or predictable default Wi-Fi/admin passwords;
- support for legacy Wi-Fi protocols (WEP, WPA/WPA2-TKIP);
- WPS enabled or supported with hard-coded PINs and no attempt-limiting;
- unencrypted HTTP management interfaces without rate-limiting or CAPTCHA;
- firmware-update mechanisms lacking proper TLS or integrity verification;
- additional exposed services (Telnet, FTP, UPnP) that are hard or impossible to
  disable;
- guest networks without real isolation, easily enabled remote management, and
  IPv6 without a default firewall.

A distinctive category is **"deep defaults"** — apparently neutral features
(guest Wi-Fi, remote access, cloud account, UPnP, USB storage) that activate
*secondary* settings introducing new attack surfaces once enabled. Compounding
this, national-scale studies report widespread use of **EoL routers** running
outdated firmware that is rarely patched. Even recent, still-supported devices
exhibit a weak out-of-the-box security profile; adding EoL hardware makes the
technical risk **systematic, not exceptional**.

## 5. Gap analysis: why existing solutions fall short

Research and practice have produced capable measures, but each is structurally
misaligned with SOHO realities (concept paper §4, §5).

### Network monitoring / IDS-IPS stacks

Practitioners deploy Suricata or Snort on small single-board computers
(Raspberry Pi) or low-power mini-PCs, or use purpose-built NSM distributions like
**Security Onion** and **SELKS** (Suricata + Zeek + Elasticsearch + dashboards).
These are technically capable and license-cost-effective, but in their original
form they:

- are **resource-intensive** (multiple CPU cores, substantial RAM, storage for
  packet capture and indices);
- assume familiarity with concepts such as WAN/LAN role assignment, span ports
  and IDS policy tuning;
- place the **entire burden of rule curation and false-positive triage** on the
  local operator;
- remain closer to a *collection of tools* than a cohesive incident-management
  function.

For power users and security professionals these trade-offs are acceptable; for
the average SOHO owner they constitute a high barrier to adoption.

### SOCaaS and MDR

SOC-as-a-Service (SOCaaS) and Managed Detection and Response (MDR) demonstrate
that SOC capabilities can be delivered as a service, but they target SMBs and
mid-market organizations, not individual SOHO sites (concept paper §4.3):

- **Cost** scales with endpoints and data volume — MDR commonly bills per
  endpoint (≈ $15–30 / endpoint / month), and SOCaaS with SIEM ingest can be more
  expensive still, reaching tens of thousands of USD annually for modest fleets.
- **Onboarding architecture** assumes identity providers, centralized directory
  services and fleet-management tooling where agents can be deployed.
- **Expected data sources** — endpoint telemetry, Active Directory logs, cloud
  platform logs, VPN concentrators, line-of-business apps — may simply not exist
  in a SOHO.
- The **SOHO router**, the primary network-edge artifact, is rarely a first-class
  citizen; these architectures focus on server and endpoint telemetry.

### Three categories of limitation

| Category | Core limitations in SOHO |
|----------|--------------------------|
| **Technical** | Limited device visibility and capacity; NAT masking host identity; pervasive encryption reducing payload visibility; device heterogeneity so no single control class covers all vectors; hardware ceilings on IDS rule-set size and capture. |
| **Economic / operational** | Vendor focus on cost and simplicity over security; no dedicated IT staff; non-trivial install, tuning and rule maintenance; manual alert triage and **alert fatigue**; persistent user difficulty hardening networks. |
| **Privacy / regulatory** | Residential traffic carries rich behavioral data (presence, habits, sometimes health-related); even coarse metadata can infer sensitive activity; centralizing detailed telemetry raises data-protection/GDPR concerns; tension between visibility and **data minimization**. |

### Summary: solution categories vs. SOHO constraints

| Solution category | Typical constraint in SOHO |
|-------------------|----------------------------|
| SOHO router security analyses | Insecure defaults, EoL firmware, exposed management, weak update mechanisms, limited user expertise |
| Local firewall / IDS on commodity hardware | Requires dedicated hardware, non-trivial reconfiguration, rule tuning, manual triage |
| Full NSM distributions (Security Onion, SELKS) | High resource usage, steep learning curve, designed for analyst teams |
| SOCaaS / MDR offerings | Cost scales with endpoints/data; assumes structured IT environment; limited router/IoT focus |
| Collaborative / blockchain / federated IDS | Mostly prototype-stage, evaluated on synthetic datasets; integration with commodity SOHO hardware largely unexplored |

## 6. Resulting requirements

The gap analysis points to a consistent conclusion: effective SOHO defense
requires **network-centric visibility, intrusion detection and basic response at
or near the network edge**, but packaged in a form compatible with:

- **limited hardware** (consumer mini-PC / low-power appliance);
- **minimal on-site expertise** (near-zero-touch, automated rule management,
  guided response);
- **strict privacy expectations** (local-first processing, explicit data
  minimization, opt-in and filtered external export).

No existing model satisfies all three simultaneously. This is precisely the niche
addressed by the **micro-SOC (µSOC)**, defined in the next document.

---

*Prev: [00 — Overview](./00-overview.md) · Next: [02 — The µSOC concept & formal definition](./02-usoc-concept-and-definition.md)*
