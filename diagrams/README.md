# Diagrams

Architecture diagrams are authored as [Mermaid](https://mermaid.js.org/) source
(`*.mmd`). GitHub renders the same Mermaid blocks inline inside the documents, so
these `.mmd` files are the single, diff-able source of truth — there are no
committed binary images. Locally-rendered `*.svg`/`*.png` files (if you generate
any) are git-ignored.

| File | Describes | Used in |
|------|-----------|---------|
| `01-four-level-architecture.mmd` | The four functional µSOC levels and their mapping to the SURU four-tier implementation | `03-architecture-overview.md`, `07-implementation-mapping.md` |
| `02-telemetry-flow.mmd` | End-to-end telemetry collection, edge normalization, SIEM ingest, indexing | `04-telemetry-and-normalization.md` |
| `03-response-hierarchy.mmd` | The four-level graduated automated-response hierarchy | `05-detection-correlation-response.md` |
| `04-deployment-modes.mmd` | Integrated (active) vs. passive (capture sensor) perimeter deployment | `08-deployment-modes.md` |

## Rendering locally (optional)

```bash
# Requires @mermaid-js/mermaid-cli (npm i -g @mermaid-js/mermaid-cli)
mmdc -i 01-four-level-architecture.mmd -o 01-four-level-architecture.svg
```

The rendered output is ignored by git on purpose; edit the `.mmd` source and let
GitHub render it.
