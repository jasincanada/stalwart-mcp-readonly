# 09 — Stalwart API capabilities for an email MCP

**Research checked:** 2026-10-07  
**Scope:** Stalwart Mail Server and protocol capabilities relevant to searching, reading, exporting, and (optionally) changing or sending email. This is a source audit, not a deployment test.

## Executive summary

The Stalwart project README identifies JMAP for Mail, IMAP4rev1/IMAP4rev2, ManageSieve, and an SMTP server as supported product surfaces. JMAP standards define mail objects, query, submission, and blob-download mechanisms; IMAP standards define mailbox search and retrieval, including retrieval of the complete message. Those protocol facts do not establish that a particular Stalwart release enables every extension, exposes raw message blobs as expected, preserves every stored header byte-for-byte, or has a given operational limit. Those implementation and deployment facts remain **UNVERIFIED** until checked against the deployed version and its official version-matched documentation/configuration. [Stalwart project README](https://github.com/stalwartlabs/stalwart#readme); [RFC 8620](https://www.rfc-editor.org/rfc/rfc8620); [RFC 8621](https://www.rfc-editor.org/rfc/rfc8621); [RFC 9051](https://www.rfc-editor.org/rfc/rfc9051).

**Version caution:** This checkout contains no Stalwart version declaration, and the upstream README is a moving branch, not a pinned release. A specific documented/deployed version and a list of recent breaking changes could not be established here. **UNVERIFIED.** Verify the exact server build/release and read the matching release notes before relying on any result below.

## Findings (verified)

### Surface inventory

“Standard capability” below describes what the named protocol/specification defines, not a guarantee that every Stalwart build or configuration enables it. The Stalwart README establishes product-level support only for the surfaces it names. Version-specific endpoint names, authentication configuration, quotas, and behavior are intentionally not inferred.

| Surface | What the primary source establishes | Access, limits, raw message, and header fidelity |
|---|---|---|
| **JMAP Core** | RFC 8620 defines the JMAP session, API requests, and blob download/upload mechanisms. Stalwart's README lists JMAP for Mail. [RFC 8620](https://www.rfc-editor.org/rfc/rfc8620); [Stalwart README](https://github.com/stalwartlabs/stalwart#readme) | JMAP is an HTTP-based protocol, but Stalwart’s accepted authentication schemes, per-user authorization, rate limits, request-size limits, and enabled capabilities are **UNVERIFIED**. A session's download URL template can address a blob; whether a specific deployed email blob yields the complete original RFC 5322 octets, with `Received` and `Message-ID` preserved untouched, is **UNVERIFIED** for Stalwart. |
| **JMAP Mail** | RFC 8621 defines mailboxes, email objects, querying, and email retrieval. Stalwart lists JMAP for Mail. [RFC 8621](https://www.rfc-editor.org/rfc/rfc8621); [Stalwart README](https://github.com/stalwartlabs/stalwart#readme) | Standard mail query/get operations expose structured properties. The RFC’s `Email` object includes a `blobId`; raw-byte fidelity and the exact server-side mapping from that blob to stored source must be tested against the deployed version. Stalwart authentication, message/attachment limits, and rate limits: **UNVERIFIED**. |
| **JMAP Blob Management** | The Stalwart README advertises Blob Management and links RFC 9404. RFC 9404 defines a JMAP blob-management extension. [Stalwart README](https://github.com/stalwartlabs/stalwart#readme); [RFC 9404](https://www.rfc-editor.org/rfc/rfc9404.html) | Exact Stalwart support for the extension and its authorization/retention semantics is **UNVERIFIED**. Blob access is not itself proof of an untouched RFC 5322 export. |
| **JMAP Submission** | RFC 8621 defines `EmailSubmission` for submitting email. The README says Stalwart has an SMTP server, but does not establish JMAP submission support. [RFC 8621](https://www.rfc-editor.org/rfc/rfc8621); [Stalwart README](https://github.com/stalwartlabs/stalwart#readme) | Stalwart JMAP Submission enablement, authentication, limits, and configuration are **UNVERIFIED**. SMTP submission is a separate surface below. |
| **JMAP Sieve** | Stalwart advertises a ManageSieve server. That does **not** establish support for the separate JMAP Sieve extension. [Stalwart README](https://github.com/stalwartlabs/stalwart#readme); [RFC 5804](https://www.rfc-editor.org/rfc/rfc5804.html) | JMAP Sieve capability and its permissions are **UNVERIFIED**. |
| **IMAP** | Stalwart advertises IMAP4rev1 and IMAP4rev2. The IMAP protocol defines mailbox listing, search, fetch, flags, and mailbox operations. [Stalwart README](https://github.com/stalwartlabs/stalwart#readme); [RFC 9051](https://www.rfc-editor.org/rfc/rfc9051); [RFC 3501](https://www.rfc-editor.org/rfc/rfc3501) | IMAP uses authenticated protocol sessions; which SASL mechanisms Stalwart enables, its limits, and account-level restrictions are **UNVERIFIED**. IMAP `FETCH` can request the complete message (`BODY[]` in rev1 terminology); whether Stalwart returns original stored bytes and preserves `Received`/`Message-ID` untouched is **UNVERIFIED**. |
| **ManageSieve** | Stalwart's README advertises a ManageSieve server; RFC 5804 specifies the protocol for managing Sieve scripts. [Stalwart README](https://github.com/stalwartlabs/stalwart#readme); [RFC 5804](https://www.rfc-editor.org/rfc/rfc5804.html) | Script-management capability is not required for email reading. Authentication methods, script-size limits, and authorization controls in Stalwart are **UNVERIFIED**. |
| **SMTP submission** | The README identifies an SMTP server. RFC 6409 specifies message submission, distinct from mailbox-reading protocols. [Stalwart README](https://github.com/stalwartlabs/stalwart#readme); [RFC 6409](https://www.rfc-editor.org/rfc/rfc6409) | SMTP submission sends messages; it is not a standard mailbox search/read API. Stalwart’s submission authentication, message-size and rate limits, and whether it accepts credentials different from mailbox credentials are **UNVERIFIED**. SMTP does not provide an API for fetching an existing message's raw source. |
| **Management/admin API** | The sources reviewed do not verify the deployed version’s management API surface, methods, auth scheme, or access-control model. **UNVERIFIED.** | Do not assume an admin API is read-only, has narrowly scoped API keys, or can retrieve user mail. Verify the version-matched official admin/API documentation and exercise each intended request with a non-admin test identity. Limits, raw-source behavior, and header fidelity: **UNVERIFIED**. |
| **REST/OpenAPI definitions** | No version-pinned REST or OpenAPI definition was located in the sources reviewed. **UNVERIFIED.** | Do not generate an MCP integration from guessed routes or schemas. Confirm whether a definition exists in the official repository/release corresponding to the deployed version. |
| **Webhooks / event hooks** | No verified Stalwart webhook/event-hook API for mailbox changes was established from the sources reviewed. **UNVERIFIED.** | Verify event types, delivery/retry guarantees, payload contents, auth, and limits in version-matched official docs before depending on them. |
| **CLI** | No verified CLI command set, user-mail access capability, or credential model was established from the sources reviewed. **UNVERIFIED.** | Confirm exact binary/version and official CLI reference. Do not use administrative CLI credentials as an MCP substitute without validating its authorization boundary. |
| **Filesystem / blob-store access** | Protocol RFCs do not define Stalwart’s internal storage layout. No storage-path or direct-blob access guarantee is established here. **UNVERIFIED.** | Direct storage access may bypass protocol authorization and is not a supported MCP API unless the deployed-version documentation explicitly says so. Do not infer that stored files are one-message-per-file or RFC 5322 source. |

### Capability matrix

“Supported” means the protocol’s standard defines the capability and the Stalwart README identifies that product surface; it does not certify the deployed configuration or byte-level behavior. **UNVERIFIED** means implementation-specific confirmation is required. “Not supported” means the named protocol does not define the operation as a normal operation of that surface; it does not rule out a separate Stalwart extension.

| Capability needed by the MCP | JMAP | IMAP | Admin API | Filesystem / blob store |
|---|---|---|---|---|
| Search email | Supported | Supported | UNVERIFIED | UNVERIFIED |
| List mailboxes | Supported | Supported | UNVERIFIED | UNVERIFIED |
| Get structured headers/properties | Supported | Supported | UNVERIFIED | UNVERIFIED |
| Get raw RFC 5322 source | UNVERIFIED | Supported (protocol fetch; Stalwart byte fidelity UNVERIFIED) | UNVERIFIED | UNVERIFIED |
| Download an email blob / attachment | Supported (standard blob mechanism; Stalwart fidelity UNVERIFIED) | Supported (message-part fetch; implementation details UNVERIFIED) | UNVERIFIED | UNVERIFIED |
| List attachments | Supported (structured email/body properties) | Supported (message body structure) | UNVERIFIED | UNVERIFIED |
| Flag / move / delete | Supported (standard mail mutations) | Supported (standard mailbox/message operations) | UNVERIFIED | UNVERIFIED |
| Send email | Supported by JMAP only if JMAP Submission is enabled; Stalwart enablement UNVERIFIED | Not supported | UNVERIFIED | Not supported |
| Manage Sieve rules | UNVERIFIED (JMAP Sieve not established) | Not supported | UNVERIFIED | UNVERIFIED |

Standards basis: [RFC 8620](https://www.rfc-editor.org/rfc/rfc8620), [RFC 8621](https://www.rfc-editor.org/rfc/rfc8621), [RFC 9051](https://www.rfc-editor.org/rfc/rfc9051), [RFC 3501](https://www.rfc-editor.org/rfc/rfc3501), [RFC 6409](https://www.rfc-editor.org/rfc/rfc6409), [RFC 5804](https://www.rfc-editor.org/rfc/rfc5804.html), [RFC 9404](https://www.rfc-editor.org/rfc/rfc9404.html).

## Recommendations (opinion)

1. Prefer a protocol-level integration over direct storage access. For evidence exports, compare the exported bytes with an independently retrieved server-side reference and record the exact method, server version, and digest. Do not label an export “original” until byte preservation is verified.
2. Start with only the required JMAP or IMAP methods. Do not expose SMTP submission, Sieve management, admin functions, or direct storage to a read-only MCP.
3. Before implementation, create a deployment verification record: exact server version/build; advertised JMAP capabilities; enabled IMAP/SASL mechanisms; mailbox/message/attachment size limits; configured rate limits; identity authorization; and raw-source tests covering `Received`, `Message-ID`, folded headers, duplicate headers, and MIME parts.
4. Treat every item marked **UNVERIFIED** as a release blocker for a feature that depends on it. Capture official documentation or a reproducible integration test for the deployed release rather than filling gaps with assumptions.

## Sources and verification notes

- Stalwart Mail Server, upstream [README](https://github.com/stalwartlabs/stalwart#readme) (moving branch; checked 2026-10-07). Product-level protocol claims only.
- JMAP standards: [RFC 8620](https://www.rfc-editor.org/rfc/rfc8620) and [RFC 8621](https://www.rfc-editor.org/rfc/rfc8621).
- IMAP standards: [RFC 9051](https://www.rfc-editor.org/rfc/rfc9051) and [RFC 3501](https://www.rfc-editor.org/rfc/rfc3501).
- Submission and Sieve standards: [RFC 6409](https://www.rfc-editor.org/rfc/rfc6409) and [RFC 5804](https://www.rfc-editor.org/rfc/rfc5804.html).
- Blob Management: [RFC 9404](https://www.rfc-editor.org/rfc/rfc9404.html).
- **Not established:** server release/version, recent breaking changes, current management API/CLI/webhook definitions, deployed limits, auth settings, and raw-byte/header fidelity. Verify against the matching official release documentation and controlled deployment tests.
