# 08 — Open questions and owner decisions

Status: unresolved items ranked by blocking effect, 2026-10-07.

## Findings

Source review establishes protocol and selected repository behavior, not
deployment guarantees. In particular, account/ACL restrictions, retained-byte
fidelity and end-to-end client approvals need synthetic tests on the selected
versions. See [access](02-stalwart-access.md), [survey](01-existing-servers.md),
[evidence](05-evidence-export.md), and [client matrix](06-transport-and-clients.md)
for primary citations and the distinction between facts and proposals.

**UNVERIFIED** means evidence is missing, not that the feature is absent.
An owner decision is not a factual claim about a product.

## Recommendations

Resolve P0 before implementation commitment, P1 before release, and P2 only if
in scope. No owner should provide secrets or real mail in issue comments, the PR,
test fixtures, or this repository.

| Rank | Type / unresolved question | Why it blocks | Owner / verification or decision needed |
| --- | --- | --- | --- |
| P0.1 | **UNVERIFIED:** effective read-only JMAP principal on the owner's pinned Stalwart version/directory | A fixed read-tool list alone cannot confine a stolen or misused upstream credential | Operator: identify version and backend without publishing endpoints; test shared-account read ACL plus read-method permissions; deny set/import/copy/submission/upload/admin and alternate-protocol writes using the same credential. Save synthetic pass/fail evidence |
| P0.2 | Decision: scope and private-mail disclosure to AI providers | Read-only still exports confidential information to whichever model receives results | Owner: select permitted account/mailbox scope, client/provider, retention policy, approvals and prohibited cross-tool capabilities; decline cloud clients if unacceptable |
| P0.3 | Decision / **UNVERIFIED:** which exact Grok client/version does the owner intend? | Official Grok Build's `grok` CLI is documented in study 06; a similarly named third-party client is not interchangeable and end-to-end compatibility remains unverified | Owner: select official product/package/version (no secrets); developer verifies lifecycle, tools, structured content, byte limits and approval flow. Select a verified alternative if unavailable |
| P0.4 | Decision: local-only MVP versus required remote Perplexity deployment | Remote identity, artifact authorization and public attack surface do not fit the same part-time budget | Owner: approve stdio-first; if remote is mandatory, reduce other scope and schedule explicit OAuth/Origin/session/egress tests |
| P0.5 | Decision: definition of “evidence-grade” | Hashes do not establish authorship, legal admissibility or historical completeness | Owner/legal adviser: approve server-retained-byte claim, accessible-thread scope, custody process and jurisdiction-specific requirements; decide if external signature/timestamp is a release gate |
| P0.6 | **UNVERIFIED:** selected storage/download path preserves retained octets | A parsed/rebuilt message defeats the intended evidence claim | Operator/developer: compare a known synthetic retained blob with JMAP download and exported .eml, including non-UTF-8, malformed and binary fixtures; distinguish pre-storage changes from exporter changes |
| P0.7 | Decision / **UNVERIFIED:** selected MCP protocol revision and client compatibility | Study 04's 2025-11-25 contract and newer 2026-07-28 lifecycle rules cannot be mixed | Developer/owner: pin client and protocol; inspect modern tool/schema rules if choosing 2026-07-28; update contract and test negotiation/lifecycle before implementation |
| P1.1 | **UNVERIFIED:** client-mediated grants and taint display | `approved: true` from the model is not owner consent; clients may hide warnings | Developer: demonstrate host/client-controlled grant bound to principal/account/tool/ID, expiry and denial; verify warnings survive display/serialization on chosen client |
| P1.2 | **UNVERIFIED:** custom role / API-key / OAuth permission confinement on chosen deployment | Authentication mechanisms do not inherently imply read-only scope | Operator: verify documented permissions and live effective grants, expiration/revocation and protocol applicability; use narrow dedicated principal rather than assuming `jmap` OAuth scope means read-only |
| P1.3 | Decision: how to handle live-mailbox races and thread scope | A thread may span mailbox/filter boundaries; before/after checks are not a transactional snapshot | Owner: accept accessible frozen-membership claim and fail-on-change policy; require offline/retained snapshot workflow if stronger historical completeness is needed |
| P1.4 | Decision: attachment extraction versus metadata-only | Adds parser, size, encoding and filesystem risks | Owner: keep inert extraction optional; approve MIME selection policy and unknown-encoding behavior. Raw .eml retention and both CSVs remain mandatory |
| P1.5 | Decision / **UNVERIFIED:** practical limits and export storage | Proposed byte/time/count caps may reject legitimate evidence or exceed client limits | Developer: benchmark synthetic 8/32 MiB messages, 100-message thread, quota/disk-full/concurrency, parser depth and client payload ceilings. Operator chooses protected root, capacity and retention |
| P1.6 | **UNVERIFIED:** deployed TLS/discovery/download endpoint behavior | Unexpected origin/redirect/proxy behavior can cause SSRF or credential leakage | Operator/developer: authorize exact origins privately; test discovery templates, redirects, DNS changes and proxy settings; do not grant arbitrary network access |
| P1.7 | Decision: custody, clock quality, verifier and audit retention | A digest stored only beside editable files cannot detect coordinated tampering | Owner: choose externally retained inventory digest, collector/handoff record and trusted storage; developer supplies independent offline verifier in a later PR |
| P1.8 | **UNVERIFIED:** remote Perplexity plan/version/auth and artifact experience | Documented connection does not guarantee our output schemas, approval controls or evidence handling | Owner/developer: verify current official plan docs and synthetic connection; remote path IDs are not client-local files; do not promise remote export download in V1 |
| P1.9 | Decision: implementation language/SDK and supply-chain review | Future implementation needs a maintained SDK and bounded parsers | Owner/developer: choose only after design approval; review licences/advisories in a future code PR. No dependencies added here |
| P1.10 | **UNVERIFIED:** independent inspection of NIST FIPS algorithm text | Primary PDF access was blocked during research | Developer: read linked FIPS 180-4 and verify standard SHA-256 implementation against published test vectors; use existing standard cryptographic library, not a new hash implementation |
| P2.1 | **UNVERIFIED:** `T-0-co/stalwart-mcp` availability | Requested comparator cannot be surveyed from an inaccessible repository | Owner: supply correct accessible URL or authorized primary files/ref. 404 does not determine private/deleted/renamed status |
| P2.2 | **UNVERIFIED:** codeChap repository licence | No permission to reuse source can be inferred | Maintainer/owner: provide licence at inspected revision; no copying meanwhile. Independent design is not blocked |
| P2.3 | **UNVERIFIED:** live interoperability, CI success and comprehensive guarantees of surveyed servers | Source inventory does not establish runtime correctness | Developer: only if reuse/comparison is later proposed, inspect pinned releases/dependency licences and run synthetic tests; not necessary to adopt their code |
| P2.4 | Decision: legacy SSE, additional clients, multiple accounts and remote downloads | Each expands compatibility and isolation work | Owner: defer to post-MVP; require a separate reviewed threat model and acceptance criteria |

### Closure discipline

For each verified closure record: date, version/ref, primary citation, synthetic
test/procedure, result, limitations and reviewer. Do not replace **UNVERIFIED**
with “verified” from a marketing claim or a successful login alone. Link an
owner decision to the applicable study; do not silently change the contract.

No unresolved P0 may be hidden by the December deadline. If release gates remain
open, use the design/experimental fallback in [study 07](07-plan-to-christmas.md).
