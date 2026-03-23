# Licensing

This repository uses different licenses for different classes of material.

The goal is to keep the smart contracts and software in the commons, while making the design documents and written materials broadly shareable.

## Summary

### Smart contracts, scripts, and software
Licensed under **GNU Affero General Public License v3.0 or later (AGPL-3.0-or-later)**.

This includes, unless otherwise noted:

- Solidity smart contracts (e.g. `DonationVault`)
- deployment and migration scripts
- testing infrastructure and test suites
- fiat on-ramp service code
- integration libraries and SDK code
- CLI tools and utilities
- CI/CD configuration and build scripts

### Documentation, design documents, and written materials
Licensed under **Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)**.

This includes, unless otherwise noted:

- `README.md`
- design documents (e.g. `Docs/DESIGN.md`)
- architecture explanations
- diagrams
- specifications
- governance documents
- website copy stored in this repository
- non-code research writing

### Third-party code and dependencies
Third-party code remains under its original license.

Nothing in this repository relicenses third-party software.
If a file, directory, submodule, vendored component, or dependency includes its own license, that license controls that material.

---

## Why this repo uses split licensing

CryptoKey Donations is intended to be a fully decentralized funding platform where autonomous entities can receive and spend donations without any centralized intermediary.

Because of that, the project wants to prevent silent proprietary capture of the smart contracts and supporting software. For that reason, the software is licensed under AGPL-3.0-or-later.

At the same time, the project wants the design documents, architecture explanations, and public-facing documentation to be widely shareable and remixable, provided attribution is preserved and derivatives remain open. For that reason, documentation is licensed under CC BY-SA 4.0.

---

## File-level default rule

Unless a file or directory explicitly states otherwise:

- **Code** is licensed under **AGPL-3.0-or-later**
- **Documentation and non-code text/media** are licensed under **CC BY-SA 4.0**

If there is any conflict between a file header and this document, the file header or local license notice takes precedence.

---

## Suggested repository layout

The following convention is recommended:

- `contracts/`, `scripts/`, `test/`, `src/`, `server/`, `lib/`
  → **AGPL-3.0-or-later**

- `Docs/`, `docs/`, `spec/`, `governance/`
  → **CC BY-SA 4.0**

- `third_party/`, `vendor/`, `external/`
  → licensed according to their included upstream licenses

---

## Contributor intent

By contributing code to this repository, you agree that your contribution is licensed under **AGPL-3.0-or-later**, unless explicitly stated otherwise in the relevant path or file.

By contributing documentation, writing, diagrams, or other non-code materials to this repository, you agree that your contribution is licensed under **CC BY-SA 4.0**, unless explicitly stated otherwise in the relevant path or file.

---

## Third-party and upstream components

This repository may incorporate ideas from, integrate with, or depend on third-party projects, including Ethereum libraries, OpenZeppelin contracts, and fiat on-ramp providers.

All third-party code, assets, and materials remain subject to their original licenses and notices.

If this repository includes any copied, modified, vendored, or derived third-party material, the maintainers should:

1. preserve original copyright notices
2. preserve original license notices
3. identify the upstream source
4. comply with all applicable attribution and redistribution requirements

Nothing in this repository should be interpreted as replacing or removing any third-party license obligations.

---

## No implicit patent grant beyond the applicable license

This repository does not provide any patent rights beyond those granted, if any, by the applicable license governing the relevant material.

If stronger explicit patent language is desired for some parts of the ecosystem, those parts should be placed under a license that provides it and clearly marked.

---

## Trademark notice

The project name, logos, and branding, if any, are not automatically licensed for unrestricted trademark use by these copyright licenses.

Copyright licenses govern copying, modification, and distribution of creative works and software.
Trademark rights, if any, are separate.

---

## How to apply the licenses in the repo

The repository should include at minimum:

- `LICENSE-AGPL`
- `LICENSE-CC-BY-SA`
- `LICENSES.md`

Optionally also include:

- `NOTICE`
- `THIRD_PARTY_NOTICES.md`

Recommended root-level structure:

```text
LICENSE-AGPL
LICENSE-CC-BY-SA
LICENSES.md
NOTICE
THIRD_PARTY_NOTICES.md
```
