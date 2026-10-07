# 01 — Existing-server survey

Status: source-reviewed snapshot, 2026-10-07; no servers executed.

## Findings

The table records inspected primary source, not endorsements, an exhaustive
catalogue, or live interoperability tests. “Latest” means the latest default-
branch commit returned during this research, not a guarantee of continuing
maintenance. Commit-pinned links make the comparison reproducible.

| Project / inspected revision | Tools exposed | Authentication / transport | Read versus write separation | Licence | Observed maintenance | Gap against our needs |
| --- | --- | --- | --- | --- | --- | --- |
| [codeChap/mcp-server-stalwart](https://github.com/codeChap/mcp-server-stalwart/tree/7dab8387bcb36809269b61bcaf04a86e0a663b70) | Mail: `get_mailboxes`, `search_emails`, `get_emails`, `delete_emails`, `create_mailbox`, `download_attachments`, `send_email`. Admin/domain/account/alias/permission/password tools; outbound diagnostics. [S1–S3] | stdio; JMAP Basic username/password; optional separate admin credentials. [S4–S5] | Mail/admin/outbound/send routers combined unconditionally; absent admin configuration rejects admin calls, not removal of mail writes. [S6] | **UNVERIFIED**: inspected root had no LICENSE; package manifest declares no licence. [S7] | Commit returned dated 2026-08-26; CI file contains format/lint/build/test steps, but execution **UNVERIFIED**. [S8] | Not a fixed mail-read-only tool set; admin/secret-returning functions outside our scope. Raw-thread manifest/hash workflow **UNVERIFIED**, not demonstrated by inspected tool inventory |
| [T-0-co/stalwart-mcp](https://github.com/T-0-co/stalwart-mcp) | **UNVERIFIED** | **UNVERIFIED** | **UNVERIFIED** | **UNVERIFIED** | Exact repository page and [API](https://api.github.com/repos/T-0-co/stalwart-mcp) returned 404 during research | Need owner-supplied accessible URL/ref or authorized primary files. Do not infer private, deleted, renamed, or related to similarly named repositories |
| [wyattjoh/jmap-mcp](https://github.com/wyattjoh/jmap-mcp/tree/55eed8536e1c334f703938e5b6ca5335d1e5bda1), generic JMAP | Reads: `get_mailboxes`, `search_emails`, `get_emails`, `get_threads`, `get_email_changes`, `get_search_updates`; mutation: mark/move/patch mailbox/delete; submission: send/reply. [J1] | stdio entrypoint configured with session URL and bearer token; optional account ID. [J2] | Mutation registration gated by selected JMAP account `isReadOnly`; submission also gated by submission capability. This is upstream metadata, not independently tested credential confinement. [J2–J3] | MIT. [J4] | Commit returned dated 2026-09-23; selected tests inspect state/missing IDs using mocks. [J5] | Useful paging/state concepts; mail mutation exists for writable accounts. Byte-exact export/CSV/hash/custody guarantees **UNVERIFIED** |
| [jgalea/mailbox-mcp](https://github.com/jgalea/mailbox-mcp/tree/70b5ff9dea381e31aeb411dc88e9c64088342c7c), generic email/JMAP comparator | Broad account/read/thread/send/reply/forward/draft/bulk/mailbox/attachment/export/settings registry; full-profile tests expect 49 tools. [M1] | JMAP Basic over validated HTTPS, redirects refused; encrypted stored credentials (AES-256-GCM), separate Gmail OAuth path. [M2] | `read` profile filters listings and enforces dispatcher refusal; account `readOnly` checked. Export/download classified local writes, excluded by read profile. Draft profile still permits trash/bulk-trash. [M1, M3] | MIT. [M4] | Commit returned dated 2026-10-05, save-path hardening; no test suite executed in this study. [M5] | Useful call-time enforcement and local-write distinction; read profile does not equal our read-mail-plus-evidence-export profile. Evidence custody guarantees **UNVERIFIED** |

### Primary-source register

**Stalwart-specific comparator**

- **S1:** [mail handlers](https://github.com/codeChap/mcp-server-stalwart/blob/7dab8387bcb36809269b61bcaf04a86e0a663b70/src/server/mail.rs#L13-L175).
- **S2:** [admin handlers](https://github.com/codeChap/mcp-server-stalwart/blob/7dab8387bcb36809269b61bcaf04a86e0a663b70/src/server/admin_tools.rs#L13-L255).
- **S3:** [outbound handlers](https://github.com/codeChap/mcp-server-stalwart/blob/7dab8387bcb36809269b61bcaf04a86e0a663b70/src/server/outbound.rs#L14-L115).
- **S4:** [startup and configuration](https://github.com/codeChap/mcp-server-stalwart/blob/7dab8387bcb36809269b61bcaf04a86e0a663b70/src/main.rs#L18-L60).
- **S5:** [JMAP authentication](https://github.com/codeChap/mcp-server-stalwart/blob/7dab8387bcb36809269b61bcaf04a86e0a663b70/src/jmap.rs#L38-L108).
- **S6:** [router assembly and admin guard](https://github.com/codeChap/mcp-server-stalwart/blob/7dab8387bcb36809269b61bcaf04a86e0a663b70/src/server/mod.rs#L31-L60).
- **S7:** [root at inspected ref](https://api.github.com/repos/codeChap/mcp-server-stalwart/contents/?ref=7dab8387bcb36809269b61bcaf04a86e0a663b70);
  [manifest](https://github.com/codeChap/mcp-server-stalwart/blob/7dab8387bcb36809269b61bcaf04a86e0a663b70/Cargo.toml#L1-L19).
- **S8:** [commit](https://github.com/codeChap/mcp-server-stalwart/commit/7dab8387bcb36809269b61bcaf04a86e0a663b70);
  [CI definition](https://github.com/codeChap/mcp-server-stalwart/blob/7dab8387bcb36809269b61bcaf04a86e0a663b70/.github/workflows/ci.yml#L13-L45).

**Generic JMAP comparator**

- **J1:** [documented tool inventory](https://github.com/wyattjoh/jmap-mcp/blob/55eed8536e1c334f703938e5b6ca5335d1e5bda1/README.md#L14-L47).
- **J2:** [entrypoint and capability checks](https://github.com/wyattjoh/jmap-mcp/blob/55eed8536e1c334f703938e5b6ca5335d1e5bda1/packages/jmap-mcp/src/mod.ts#L19-L69).
- **J3:** [mutation registration](https://github.com/wyattjoh/jmap-mcp/blob/55eed8536e1c334f703938e5b6ca5335d1e5bda1/packages/jmap-mcp/src/tools/email.ts#L390-L470);
  [account metadata](https://github.com/wyattjoh/jmap-mcp/blob/55eed8536e1c334f703938e5b6ca5335d1e5bda1/packages/jmap/src/client.ts#L267-L304).
- **J4:** [MIT licence](https://github.com/wyattjoh/jmap-mcp/blob/55eed8536e1c334f703938e5b6ca5335d1e5bda1/LICENSE#L1-L21).
- **J5:** [commit](https://github.com/wyattjoh/jmap-mcp/commit/55eed8536e1c334f703938e5b6ca5335d1e5bda1);
  [mock tests](https://github.com/wyattjoh/jmap-mcp/blob/55eed8536e1c334f703938e5b6ca5335d1e5bda1/packages/jmap-mcp/tests/tools/email_test.ts#L94-L134).

**Generic email comparator**

- **M1:** [tool registry / local writes](https://github.com/jgalea/mailbox-mcp/blob/70b5ff9dea381e31aeb411dc88e9c64088342c7c/src/tools/registry.ts#L34-L66);
  [profile tests](https://github.com/jgalea/mailbox-mcp/blob/70b5ff9dea381e31aeb411dc88e9c64088342c7c/tests/tools/profile.test.ts#L24-L82).
- **M2:** [JMAP authentication](https://github.com/jgalea/mailbox-mcp/blob/70b5ff9dea381e31aeb411dc88e9c64088342c7c/src/providers/jmap.ts#L63-L140);
  [provider credentials](https://github.com/jgalea/mailbox-mcp/blob/70b5ff9dea381e31aeb411dc88e9c64088342c7c/src/provider-factory.ts#L14-L75);
  [encryption](https://github.com/jgalea/mailbox-mcp/blob/70b5ff9dea381e31aeb411dc88e9c64088342c7c/src/auth/credentials.ts#L11-L48).
- **M3:** [call-time enforcement](https://github.com/jgalea/mailbox-mcp/blob/70b5ff9dea381e31aeb411dc88e9c64088342c7c/src/tools/registry.ts#L119-L212).
- **M4:** [MIT licence](https://github.com/jgalea/mailbox-mcp/blob/70b5ff9dea381e31aeb411dc88e9c64088342c7c/LICENSE#L1-L21).
- **M5:** [commit](https://github.com/jgalea/mailbox-mcp/commit/70b5ff9dea381e31aeb411dc88e9c64088342c7c).

### Limits of this survey

No live Stalwart compatibility, passing CI, comprehensive dependency licensing,
or runtime no-write behavior was verified. Source inspection of selected files
does not prove the absence of an export feature elsewhere. For each
**UNVERIFIED** evidence workflow, verification requires pinned implementation
and format documentation plus a synthetic raw-byte/hash/completeness test.
No external source code has been copied into this repository.

## Recommendations

1. Build a static six-tool allowlist and enforce it again at dispatch, independent
   of advertised annotations or upstream `isReadOnly`.
2. Keep mail credentials separate from administrator credentials; do not accept
   admin secrets, account-switching or password-returning tools.
3. Treat export as a constrained local write, not a mailbox mutation; a generic
   “read profile” is insufficient to describe that distinction.
4. Use conceptual lessons about state, paging, missing IDs and call-time checks,
   not copied source. Respect MIT notices if reuse is ever separately proposed.
   With unverified licence, do not reuse code without explicit permission.
5. Resolve the requested inaccessible repository before calling the survey
   complete; do not let it block drafting our independent design.
