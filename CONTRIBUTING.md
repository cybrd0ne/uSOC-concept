# Contributing to uSOC-concept

This repository documents the **µSOC** concept and maps it to its open-source
reference implementation. It is **documentation** — prose, diagrams, and short
illustrative snippets — not deployable code. Contributions that improve clarity,
accuracy, structure, or coverage are welcome.

## Ground rules

- **Accuracy first.** The documents describe a specific architecture. Changes must
  stay faithful to the concept and to the reference implementation at
  [github.com/cybrd0ne/suru-foss](https://github.com/cybrd0ne/suru-foss). When the
  two would diverge, say so explicitly rather than papering over it.
- **No full code.** Use short, clearly-labelled illustrative snippets (≤ ~15
  lines). For anything larger, link to the path in the reference implementation.
- **No secrets, no private hostnames.** This is a public concept repo. Do not add
  credentials, internal hostnames, real IPs, or private infrastructure detail.
- **Keep the structure.** Documents are numbered and cross-linked (`docs/en/NN-*.md`)
  and map onto the conceptual levels and the implementation tiers. New material
  goes *inside* the document it belongs to, or as the next sequential document.

## Bilingual documentation (EN + RO)

The concept is maintained in **English** (`docs/en/`) and **Romanian**
(`docs/ro/`).

- **English is the canonical source.** Make substantive changes in `docs/en/`
  first.
- **Romanian mirrors English.** Each `docs/ro/NN-*.md` corresponds to the same
  English file and should track its meaning, not its wording literally.
- **The Romanian edition may be held back from publication.** When it is, the
  `docs/ro/` tree is excluded from version control via `.gitignore`; see that file
  for the one-line unblock procedure. Do not force-add `docs/ro/` while it is held
  back.

## Diagrams

Diagrams are authored as [Mermaid](https://mermaid.js.org/) sources under
`diagrams/*.mmd` and embedded inline in the documents (GitHub renders them).
Edit the `.mmd` source and the inline block together; do not commit rendered
`*.svg`/`*.png` (they are git-ignored).

## Proposing a change

1. Open an issue describing the gap or inaccuracy, or
2. Submit a pull request that edits the relevant `docs/en/` file(s) (and, when the
   Romanian edition is published, the matching `docs/ro/` file).

Keep each pull request scoped to one topic so it is easy to review against the
source concept and the reference implementation.

## License

By contributing you agree your contribution is licensed under
**[CC BY 4.0](./LICENSE)**, the license of this repository.
