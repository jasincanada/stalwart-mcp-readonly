# Design studies

Research snapshot: **2026-10-07**. Target release: **December 25, 2026**.
Status: **design phase**; documentation only, no implemented server guarantees.

## Study index

| Study | Status | Verified basis / unresolved work |
| --- | --- | --- |
| [01 — Existing servers](01-existing-servers.md) | Source-reviewed; partial survey | Pinned tool/auth/licence observations; requested T-0-co repository and codeChap licence UNVERIFIED |
| [02 — Stalwart access](02-stalwart-access.md) | Protocol/docs reviewed; deployment pending | JMAP and documented permissions; effective read-only credential recipe UNVERIFIED on owner's deployment |
| [03 — Threat model](03-threat-model.md) | Proposed controls | Official MCP security baseline; enforcement and client prompt-injection resilience UNVERIFIED |
| [04 — Tool design](04-tool-design.md) | Proposed v1 schemas | Six tools, strict inputs/outputs, bounds and paging; implementation/client grants UNVERIFIED |
| [05 — Evidence export](05-evidence-export.md) | Proposed acquisition specification | RFC raw-blob/encoding semantics; .eml/CSV/hash/checklist design; deployed byte fidelity and legal requirements UNVERIFIED |
| [06 — Transports and clients](06-transport-and-clients.md) | Docs-reviewed compatibility matrix | MCP revisions and official Grok Build/client support; owner's client/version choice and end-to-end compatibility UNVERIFIED |
| [07 — Christmas plan](07-plan-to-christmas.md) | Proposed schedule | Weekly gates through Dec 25; one part-time developer, owner approval and unresolved blockers |
| [08 — Open questions](08-open-questions.md) | Ranked decision register | P0 release/design blockers, P1 verification, P2 deferrals; owner decisions pending |

## Reading and status conventions

Each study separates **Findings** (cited primary-source observations, including
limits) from **Recommendations** (our proposals, not product capabilities).
**UNVERIFIED** is explicit missing evidence with a verification path. A documented
protocol feature is not proof of a particular deployment or client.

Primary websites were not all reachable in this research environment; primary
repository files were used where available. Sources are linked inline, with
commit-pinned community comparisons and dated MCP specification references.
Recheck rolling official documentation before implementing or releasing.

Start with access, threat model and open questions; approve the six-tool and
evidence contracts before implementation. All examples are synthetic; no mail
data or secrets belong in this repository. Generated evidence belongs outside it.

This PR adds only these studies and the explicitly requested short top-level
README update (the sole exception to the `docs/`-only rule). No source code,
dependencies, CI changes or new test infrastructure are included.

**Documentation only; no code. Do not merge until the owner reviews.**
