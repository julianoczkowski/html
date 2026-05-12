# Security policy

## Supported versions

Security-sensitive fixes apply to the default branch ([`main`](https://github.com/julianoczkowski/html/tree/main)) and are released by tagging and publishing as usual for this project. This repository contains documentation and **skill files** (Markdown plus a handful of `.html` reference files) consumed by local AI tools, not a hosted service or installable package with a separate runtime.

## Reporting a vulnerability

**Please do not** open a public issue for unfixed security problems.

- Prefer **[GitHub private security advisories](https://github.com/julianoczkowski/html/security/advisories/new)** for this repository so the report stays private and we can coordinate a fix and disclosure.
- If you cannot use advisories, contact the maintainers with enough detail to reproduce or understand the issue; we will treat it as confidential and respond as soon as we can.

Include: affected paths or patterns, what you think could go wrong, and steps to reproduce (if any). Screenshots or minimal excerpts are fine when they help; avoid pasting secrets.

## What we consider in scope

- Content in this repo that could plausibly lead an agent to **unsafe or harmful** behavior in realistic use (e.g. instructions that push toward bypassing user consent, exfiltrating data, or abusing the user's environment).
- **Accidental** inclusion of secrets, credentials, or private tokens in the repository (we will rotate/remove and rewrite history if needed).
- Sample HTML in `gallery/` or pattern playbooks that performs network requests, loads remote scripts, or otherwise contradicts the skill's own "single file, no network, no build step" rule.

## Out of scope

- **Generic LLM or agent limitations** (e.g. prompt injection against third-party models) — we cannot "patch" the behavior of external providers or tools; use skills in trusted environments and review them before use in sensitive contexts.
- Typos, subjective design opinions, or bugs that do not have a clear security or safety impact.
- Issues that only affect long-deprecated forks or copies not maintained here.

## License and disclaimer

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file. Software and documentation are provided "as is," without warranties; see the license for full terms.
