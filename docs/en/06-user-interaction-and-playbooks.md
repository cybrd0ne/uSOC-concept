# 06 — User Interaction & Playbooks

A defining constraint of the µSOC is that it must be **operable by users with no
cybersecurity training**. A traditional SOC relies on tier 1–3 analysts to
interpret raw alerts and decide on response actions; the µSOC instead has to
deliver **pre-processed context, explicit instructions, and automation with
minimal intervention** — turning technical events into clear explanations and
concrete action steps for the end user.

The interaction model is structured in **two levels**, each of which can be
activated independently depending on the hardware available and the maturity of
the deployment:

- **Level 1** — a dashboard- and rule-based interface (works on minimal
  hardware).
- **Level 2** — an *optional* local AI assistant for contextualization and
  guidance (requires additional hardware; fully optional).

---

## Level 1 — Dashboards and predefined rules

The primary interface for the non-expert user is a **unified web UI on the SIEM
device (A_SIEM)** that aggregates and visualizes the normalized telemetry from
*all* µSOC components — the Router SOHO (firewall, Suricata, pfBlockerNG, Zeek)
and the optional endpoint agents (E_EDR).

While the Router SOHO exposes its own native per-component statistics
(pfBlockerNG, Suricata, Zeek), those views are **fragmented**, specific to each
subsystem, and provide no correlated picture of the network's security state. The
aggregated visualization on the SIEM device is therefore the preferred point of
interaction.

The dashboards are organized hierarchically by visibility type, aligned to the
ECS v8 schema (see [04 — Telemetry & Normalization](./04-telemetry-and-normalization.md)).

### The six predefined dashboards

| # | Dashboard | Purpose & key visualizations |
|---|-----------|------------------------------|
| 1 | **Security Overview** | Daily-use entry point answering *"Is my network safe right now?"* — active alerts by severity (critical/high/medium/low) with 24h trend, pfBlockerNG blocks per threat category, a µSOC component health indicator (Router SOHO, Suricata, Zeek, SIEM), and a geographic map of blocked WAN traffic origins. |
| 2 | **Firewall & IP Block Activity** | Perimeter filtering detail: top blocked IPs by frequency, block distribution per reputation feed (Spamhaus, Emerging Threats, Abuse.ch…), DNSBL block timeline per domain, and a filterable/sortable table of the last 100 blocked events. Colour coding (red = critical, orange = suspicious, yellow = informational) reduces cognitive load. |
| 3 | **IDS/IPS Alerts** | Suricata/Snort alerts: distribution across Emerging Threats categories (malware, exploit, trojan, scan, policy), MITRE ATT&CK technique frequency, WAN-vs-LAN alert timeline, and full details of recent alerts with normalized ECS fields (`rule.name`, `rule.category`, `source.ip`, `destination.ip`, `network.protocol`). |
| 4 | **Network Traffic Analysis** | Zeek-aggregated network statistics: top protocols by volume, geographic distribution of external destinations, DNS anomalies (NXDOMAIN rates, high-entropy domains), weak/expired TLS certificates, and abnormally large-volume sessions. |
| 5 | **Endpoint Activity** *(optional)* | Active only when EDR agents are installed: newly created processes per host, network connections per process, critical system-file changes, failed authentication events, and a per-endpoint status indicator (normal/suspect/isolated). |
| 6 | **Active Incidents** | Correlated, multi-source incidents only: active incidents with priority, the correlated context of each (how many µSOC components contributed to detection), the result of any automatic response executed, and the recommended manual actions for the user. |

### The differentiator: detection rules carry response context

A key element distinguishing the µSOC from a generic SIEM deployment is that the
**Sigma detection rules carry pre-processed response context**, stored at
configuration time and displayed to the non-expert user *alongside* the alert
they trigger. Beyond the standard detection fields, the µSOC's Sigma rules include
dedicated user-facing fields written in plain language.

The following is a compact, **illustrative** excerpt (not a full rule file):

```yaml
title: Suricata C2 Communication Detected - SOHO Router
id: a1b2c3e4-e5e6-7810-abcd-ef1234567890
status: stable
description: >
  Command-and-control communication detected between a local device
  and an IP classified as active C2 infrastructure.
references:
  - https://abuse.ch/feodotracker/
tags:
  - attack.command_and_control      # TA0011
  - attack.t1071                     # Application Layer Protocol
level: high
custom:
  user_explanation: >
    A device on your network is trying to reach a known malicious
    control server. This may indicate an active malware infection.
  recommended_actions:
    - "Identify the device with the displayed source IP"
    - "Disconnect that device from the network until checked"
    - "Run a full antivirus scan on the device"
  auto_response_triggered: "IP block applied automatically on the Router SOHO"
  severity_user_label: "URGENT — action required"
```

The content of these instructional fields is pre-defined and rendered in common
language to the user, **removing the dependence on technical security knowledge**
for interpreting and responding to incidents. The full, runnable rule set lives
in the open-source reference implementation
([github.com/cybrd0ne/suru-foss](https://github.com/cybrd0ne/suru-foss),
`tier2-telemetry/sigma/`).

---

## Level 2 — Optional local AI assistant

> **Optional and non-essential.** The base µSOC functions fully *without* any AI
> component. The AI layer only *augments* the existing dashboards and rules; it
> replaces nothing. There is **no deployable AI code in the open-source reference
> implementation** — this section documents the concept as described in the
> source architecture, not a shipped feature.

The second interaction level adds **local artificial-intelligence** capabilities
for contextualizing security events and instructing the user, **without any
dependency on external cloud services** in its default configuration. It requires
hardware beyond the SIEM minimum but operates entirely offline, preserving the
data-minimization principle.

At its core is a **local RAG (Retrieval-Augmented Generation)** system that
combines a pre-indexed security knowledge base (response playbooks, MITRE ATT&CK
descriptions, SOHO-specific remediation guides, component manuals) with the live
context of active SIEM events.

### The three-step RAG flow

1. **Offline knowledge-base indexing** — at initial setup, the µSOC reference
   documents are converted into vector embeddings by a local model and stored in
   a vector store on the SIEM device. Performed once; no internet connection
   required.
2. **Semantic query on alert generation** — when a high-severity event is raised,
   the alert's technical context (event type, involved IPs, triggered Sigma rule,
   relevant ECS fields) is embedded and used for a semantic search across the
   local knowledge base, retrieving the documents most relevant to the incident
   type.
3. **Contextualized response generation** — the retrieved context plus the active
   alert details are passed to a lightweight local language model, which generates
   a natural-language explanation and incident-specific action steps. The result
   is shown in **Dashboard 6 (Active Incidents)** as a contextual-assistance
   panel, complementary to the standardized technical information.

This transforms the µSOC from a passive alerting system into an **active incident
-response assistant** able to answer questions such as *"What does this alert mean
for my network?"*, *"What should I do in the next 30 minutes?"*, or *"Is this
related to yesterday's alert?"* — without transmitting sensitive data outside the
local perimeter.

### Example local components (from the source architecture)

- A **compact local embeddings model** (~300M parameters, under ~200 MB RAM,
  multilingual including Romanian) generating vectors entirely on-device.
- A **small local LLM** (e.g. a 1B/3B-parameter quantized model) for the
  natural-language generation step, running on the SIEM CPU with interactive
  (seconds-scale) latency.

### The three AI configurations

| Criterion | Config A — Local | Config B — Hybrid | Config C — Cloud |
|-----------|------------------|-------------------|------------------|
| Operational-data privacy | Maximum | High (with pre-export filtering) | Medium |
| Analysis / instruction quality | Medium (compact model) | High | High |
| SIEM hardware requirement | ≥ 16 GB RAM | ≥ 8 GB RAM | ≥ 4 GB RAM |
| Internet dependency | Zero | Partial (LLM API) | Complete |
| Operational cost | Zero | API pay-per-use | API pay-per-use |
| User response latency | 5–30 s | 2–5 s | 1–3 s |
| Data-minimization compliance | Complete | Partial | Conditional |

In **Config B (hybrid)**, embeddings remain local while final generation is
delegated to a cloud LLM via secure API — but a **mandatory pre-export filtering
stage** strips identifiable fields (internal IPs, hostnames, user data) so that
only generalized event descriptions and already-public document fragments leave
the perimeter. **Config C** is suitable only for hardware-constrained SIEM devices
processing exclusively public security documents — not operational network logs —
and is not recommended for prototype or high-privacy deployments.

### Documented limitations

- A local embeddings model does not *reason*; it transforms text into vectors to
  find correlations. Natural-language explanation of an attack requires pairing it
  with a generative model.
- The locally-created context is bounded (a small context window). Very long logs
  or long-horizon event analysis require appropriate data chunking.

---

## Playbooks

Incident-response playbooks in the µSOC are **pre-defined at install time**, in
close correlation with the activated Sigma detection rules, and cover the threat
scenarios documented as relevant to the SOHO profile. Each playbook is structured
in three parts:

- **Initiator** — a finding or incident of configured severity.
- **Automatic actions** — executed without user intervention.
- **Instructions** — displayed to the user for actions requiring human decision.

As with the interaction model, playbook implementation operates at two automation
levels.

### Level 1 — Predefined automation

Playbooks are implemented as predetermined action sequences, triggered
automatically when initiation conditions are met, without dynamic contextual
evaluation. Available execution mechanisms:

- **API response** — predefined scripts on the Router SOHO and endpoint agents:
  IP block on the Router SOHO firewall (via `pfctl`), terminate a malicious
  process by PID, quarantine a suspect file, local IP block on the endpoint.
- **OpenEDR isolation** — full network containment for endpoints with an OpenEDR
  agent, triggered automatically on critical-severity events while keeping the
  communication channel to the SIEM open.
- **Configuration & remediation automation** — integration with agentless tooling
  (e.g. **Ansible** over SSH) for remediation beyond the basic mechanisms
  (re-applying firewall policy, updating reputation feeds, restoring a secure
  configuration). Agentless operation avoids installing extra components on the
  Router SOHO and is the **recommended** approach for a minimal stack.

Representative Level 1 playbook — *"C2 communication detected from an internal
endpoint"*:

| Step | Action | Mechanism | User intervention |
|------|--------|-----------|-------------------|
| 1 | Block destination IP on Router SOHO | API response → `pfctl` | No |
| 2 | Isolate source endpoint from network | OpenEDR isolation | No |
| 3 | Collect active-process snapshot | Automation | No |
| 4 | Notify user with instructions | Alerting → email/webhook | — |
| 5 | Full antivirus scan on endpoint | Directed manual action | Yes |
| 6 | Confirm remediation & reintegrate | User decision in Dashboard 6 | Yes |

### Level 2 — Optional contextual orchestration (MCP)

> Optional; depends on the Level 2 AI assistant above.

The second playbook level replaces predetermined logic with contextual AI
reasoning, able to adapt the response to the specific event, network history, and
µSOC configuration. It uses the **Model Context Protocol (MCP)** — an open
protocol that standardizes how applications expose context and tools to AI
agents — to expose µSOC component capabilities as tools (SIEM query for historical
incident context, response execution, threat-intel lookups, firewall-rule updates
on the Router SOHO). Advantages over Level 1:

- **Proportional response** — the agent weighs the real severity against the
  network's historical behaviour before deciding whether full isolation is
  warranted or a perimeter block suffices.
- **False-positive reduction** — the agent can query the source IP's behavioural
  history, the frequency of similar past alerts, and current TI reputation before
  triggering an invasive automatic response.
- **Autonomous investigation** — the agent can formulate and execute SIEM query
  sequences to reconstruct the incident timeline and present a complete narrative
  before requesting confirmation for remediation.
- **Adaptability to heterogeneous SOHO** — unlike static playbooks assuming a
  fixed topology, the agent can consult the current µSOC configuration and adapt
  the response to the specific installation (endpoint count, presence of EDR
  agents, Router SOHO active vs. passive).

**Operational-security requirement:** all AI-agent interactions with µSOC systems
(tool calls, responses, agent reasoning, session data) must be **fully logged and
correlated in the SIEM**, maintaining traceability of response decisions —
including those generated autonomously without human intervention.

---

*Prev: [05 — Detection, Correlation & Response](./05-detection-correlation-response.md) · Next: [07 — Implementation Mapping](./07-implementation-mapping.md)*
