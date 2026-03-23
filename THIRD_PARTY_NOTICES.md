# Third-Party Notices

This file documents third-party software, content, and related notices relevant to the CryptoKey Donations repository.

## Purpose

The purpose of this file is to:

- identify third-party code or materials included in the repository,
- preserve required attribution and license information,
- distinguish inspiration from direct inclusion,
- help maintainers stay compliant with upstream licenses.

---

## Current Status

At the current repository stage, the project may reference external tools, libraries, contracts, or repositories for inspiration and compatibility planning.

**Unless explicitly noted below, reference to a third-party project does not mean that code from that project is included in this repository.**

If code is later copied, modified, vendored, or redistributed from any third-party source, maintainers should add an entry here with:

- project name,
- upstream URL,
- copyright holder,
- applicable license,
- paths in this repository where the material appears,
- notes on modifications.

---

## Expected Dependencies

The following third-party projects are expected to be used as dependencies (imported, not vendored) as development progresses:

### OpenZeppelin Contracts

**Project:** OpenZeppelin Contracts
**Upstream URL:** https://github.com/OpenZeppelin/openzeppelin-contracts
**License:** MIT
**Usage:** Smart contract base classes (e.g. `ReentrancyGuard`, `IERC20`).

### Foundry / Hardhat

**Project:** Foundry / Hardhat (TBD)
**Upstream URL:** https://github.com/foundry-rs/foundry or https://github.com/NomicFoundation/hardhat
**License:** MIT / MIT
**Usage:** Smart contract development, testing, and deployment framework.

### Stripe

**Project:** Stripe SDK
**Upstream URL:** https://github.com/stripe/stripe-node
**License:** MIT
**Usage:** Fiat on-ramp credit card processing.

---

## Entry Format for Future Inclusions

If third-party code is later added directly to this repository, use a section like this:

```text
Project: <project name>
Upstream URL: <url>
License: <license identifier>
Repository paths:
- path/to/file1
- path/to/file2
Modification status:
- modified / unmodified
Notes:
- <any important compliance notes>
```
