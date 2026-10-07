# 07 — Plan to December 25, 2026

Status: proposed schedule. Baseline: 2026-10-07; deadline: 2026-12-25.

## Findings

This repository is in the design phase; these studies are not a working server.
The largest unresolved dependencies are upstream write-denial verification,
owner-approved disclosure policy, and real client/version compatibility
([open questions](08-open-questions.md)). Those are study conclusions, not
verified properties of a future implementation.
All future implementation and client acceptance outcomes are **UNVERIFIED**
until the corresponding gates below pass.

## Recommendations

Assume one part-time developer at **8 hours/week**, about 92 hours across the
partial weeks below, including research, development, review and testing.
These are planning estimates, not promises. Do not start implementation until
the owner reviews this documentation-only PR. No real mail is needed for tests.
If approval slips, reduce scope rather than removing security gates.

### Week-by-week milestones

Each row has one deliverable and a go/no-go check; all later code work belongs in
future PRs, not this one.

| Week (inclusive dates, 2026) | Hours | Milestone | Acceptance / gate |
| --- | ---: | --- | --- |
| Oct 7–11 | 4 | Review eight studies, settle local-first scope and client/provider policy | Owner signs decisions on mail disclosure, evidence semantics, and README exception |
| Oct 12–18 | 8 | Pin Stalwart version; build isolated synthetic mailbox; validate principal/ACL recipe | Query/get/download succeed; set/import/copy/submission/mailbox mutation and IMAP writes fail with same credential. Stop release planning if not enforceable |
| Oct 19–25 | 8 | Select implementation language/SDK in a future PR; pin MCP revision and primary client | Client launches stdio, uses selected revision's lifecycle/version rules, lists exactly six fixed tools; schemas reject unknown fields; no debug text on stdout |
| Oct 26–Nov 1 | 8 | JMAP discovery, account allowlist, TLS/endpoint controls, credential handling | No cross-account reads; unsafe discovery/redirect/DNS targets rejected; revoked credentials fail safely |
| Nov 2–8 | 8 | Search and mailbox listing, bound cursors | Multi-page synthetic set has no silent omissions; stale cursor forces restart; quotas and unsupported filters tested |
| Nov 9–15 | 8 | Headers, raw read, attachment metadata | Duplicate/malformed headers retained as data; raw bytes hash to fixture; oversize raw rejected, not truncated; no HTML/attachment execution |
| Nov 16–22 | 8 | Thread enumeration and core export | Frozen membership, fresh directory, deterministic filenames, exact .eml bytes, manifest rows and independent SHA-256 match |
| Nov 23–29 | 8 | Partial/race/encoding and filesystem hardening | Missing Date, duplicate messages, concurrent thread change, disk-full, cancellation, symlink/traversal and CSV cases pass; no false complete export |
| Nov 30–Dec 6 | 8 | Injection/exfiltration and privacy review | Hostile body/header/filename cannot alter server tools, destinations, paths or credentials; outbound and audit logs inspected with synthetic canaries |
| Dec 7–13 | 8 | Primary-client end-to-end test; independent review | Owner-approved client displays untrusted results and approvals; read → raw → export → offline verify works; critical findings resolved |
| Dec 14–20 | 8 | Release candidate, operator instructions and incident/revocation rehearsal | New operator reproduces least-privilege setup and verification; no-write tests repeat; restore/custody limitations documented |
| Dec 21–25 | 8 | Buffer, final owner acceptance and tagged release (future PR) | All mandatory gates green; otherwise ship design/preview labelled non-production, not “evidence-grade” release |

### Minimal viable release

- One operator, one explicitly allowlisted JMAP account, selected mailboxes,
  dedicated upstream principal whose mail-write denial is independently tested.
- Local stdio and **one verified owner-chosen client**. Do not advertise Grok or
  Perplexity compatibility based on inference.
- All six tools in [study 04](04-tool-design.md), bounded search and reads,
  no mail mutations, raw thread .eml plus both CSVs and bundle inventory.
- Attachment metadata and raw message retention mandatory; separate attachment
  extraction optional, explicitly recorded as disabled if cut.
- Fail-closed quotas, safe filesystem writes, synthetic security fixtures,
  reproducible offline integrity verification and operator custody instructions.

### Mandatory acceptance tests

| Area | Release requirement |
| --- | --- |
| Mail immutability | Same upstream credential denies all applicable write methods/protocols; search/get/export leave mailbox membership, keywords and message bytes unchanged |
| Schema and authorization | Unknown tool/field, bad date, oversized query, wrong account, guessed IDs, expired grant/cursor rejected |
| Pagination and completeness | Search state changes are not hidden; export enumerates all accessible thread IDs without `collapseThreads`; permissions/missing items cannot produce `complete` |
| Preservation | Non-ASCII headers, folded/duplicate headers, mixed line endings, binary MIME, missing/invalid Date, and duplicate bytes round-trip exactly from fixture blob to .eml |
| Integrity and custody | Independent verifier catches changed/deleted/extra files and CSV changes using inventory and externally retained inventory digest |
| Hostile input | All channels in study 03 exercised; no attachment/link execution, arbitrary egress, tool changes, root escape, or credentials in errors/logs |
| Limits and recovery | Streaming limits, timeouts, concurrency, disk-full and cancellation leave identifiable incomplete exports; retries never overwrite evidence |
| Client and privacy | Selected client/version supports the negotiated protocol and result schema; operator understands model-provider disclosure; actual approval control tested |

### Cut order if time runs short

1. Remote Streamable HTTP and remote artifact retrieval: **not in local MVP**.
2. Additional clients, legacy SSE compatibility and convenience UI.
3. Separate attachment extraction (retain original attachments within .eml and
   `ATTACHMENTS.csv` metadata; mark extraction disabled, never “missing” silently).
4. Advanced filter combinations, body previews, performance tuning and caching.
5. Signing/timestamp-service integration and cross-account exports.

Never cut: upstream write denial, account binding, raw-byte fidelity, hashes,
manifest status, safe paths, taint labels, bounded operations, secret redaction,
or disclosure review. If any fails, move the release date or ship a clearly
labelled design/experimental preview. December 25 is a planning target, not a
reason to weaken the evidence claim.
