# 00 — Overview

## What the µSOC is

The **µSOC (micro Security Operations Center)** is a constrained, integrated form
of a Security Operations Center designed specifically for **Small Office / Home
Office (SOHO)** environments. Where a traditional SOC combines people, processes
and technology to deliver continuous monitoring, detection, investigation and
response across an enterprise, the µSOC delivers a **minimal but coherent subset**
of those functions — telemetry collection, detection, prioritized alerting, basic
triage, and guided response — sized for:

- a **narrow scope**: a single SOHO network with a handful of users and devices;
- **heterogeneous consumer hardware** and non-expert operators;
- **strict usability and privacy constraints**: near-zero-touch deployment,
  simplified interfaces, and explicit data-minimization.

The µSOC is not "a smaller enterprise SOC". The combination of scope, function
set, deployment model and constraints makes it a distinct type of SOC.

## Why it exists

SOHO networks sit uncomfortably between two extremes:

- They are **increasingly targeted** — compromised SOHO routers and end-of-life
  IoT devices are routinely conscripted into botnets, residential-proxy networks,
  and state-sponsored relay infrastructure.
- The solutions that could protect them are **misaligned**: full NSM stacks
  (Security Onion, SELKS) demand dedicated hardware and expert operation, while
  SOCaaS/MDR offerings are priced and architected for SMBs and enterprises with
  identity providers, endpoint fleets and dedicated IT staff.

The µSOC closes this gap by packaging network-centric visibility, detection and
basic response into a form compatible with limited hardware, minimal on-site
expertise, and strong privacy expectations.

## Five design principles

The architecture is dimensioned around five principles (see
[document 03](./03-architecture-overview.md)):

1. **Security by default** — restrictive initial configuration following
   established best practice (CIS baselines), default-deny firewalling with
   explicit exceptions.
2. **Data minimization & local-first privacy** — security telemetry is processed
   and stored locally; there is no implicit flow of raw data to the cloud.
3. **Near-automatic operation** — configuration, updates and detection-rule
   management are automated, removing the dependence on local expertise.
4. **Minimal hardware** — every component runs on consumer-grade hardware
   (mini-PC, low-power appliance) or as resource-bounded Docker containers / VMs.
5. **Modularity & flexibility** — a minimal deployment is possible, and the
   architecture scales up as the security function matures.

## How to read this repository

This repository documents the µSOC at **two layers**:

- **The concept** — documents [01](./01-problem-and-threat-model.md) through
  [06](./06-user-interaction-and-playbooks.md), plus
  [09](./09-scenarios-and-limitations.md): the problem, the formal model, the
  architecture, the telemetry and normalization model, the detection-and-response
  engine, the non-expert interaction model, and the limitations.
- **The implementation mapping** — document
  [07](./07-implementation-mapping.md) and
  [08](./08-deployment-modes.md): how each conceptual element corresponds to a
  concrete artifact in the open-source reference implementation, and how the
  perimeter is physically deployed.

Throughout, code is shown only as **short illustrative snippets**. The full,
runnable implementation is referenced, not reproduced — see
[document 07](./07-implementation-mapping.md) and the repository README for the
link to the open-source reference implementation.

## Scope of this documentation

This is **documentation, not a deployable artifact**. It does not ship secrets,
private hostnames, AI/agent tooling, or red-team tooling. Its purpose is to make
the µSOC concept understandable, citable, and reproducible from the public
reference implementation.

---

*Next: [01 — Problem & threat model](./01-problem-and-threat-model.md)*
