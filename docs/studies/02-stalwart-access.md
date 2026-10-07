# 02 — Safe Stalwart mail access

Status: protocol and official-docs study; deployment recipe **UNVERIFIED**.
Reviewed 2026-10-07.

## Findings

### Source/version boundary

Stalwart findings below were read in its official documentation repository at
[`8623f64887bb34bd57e22491c5782955069177e5`](https://github.com/stalwartlabs/website/tree/8623f64887bb34bd57e22491c5782955069177e5/src/content/docs/docs).
Direct published-site access failed in this environment. Published-site parity
and applicability to the owner's installed version are **UNVERIFIED**.
Stalwart lists RFC 8620, RFC 8621 and IMAP ACL RFC 4314 among supported standards
([official RFC list](https://stalw.art/docs/development/rfcs/);
[inspected source](https://github.com/stalwartlabs/website/blob/8623f64887bb34bd57e22491c5782955069177e5/src/content/docs/docs/development/rfcs.md)).
Protocol definitions below are verified standards, not deployed-server tests.

### JMAP reading sequence

| Step | Verified interface / interpretation | Primary source |
| --- | --- | --- |
| Discover | Stalwart documents `/.well-known/jmap` for endpoint/session details and `/jmap` for mail operations, on an HTTP listener | [Stalwart JMAP](https://stalw.art/docs/http/jmap/), [pinned source](https://github.com/stalwartlabs/website/blob/8623f64887bb34bd57e22491c5782955069177e5/src/content/docs/docs/http/jmap/index.md) |
| Select account | Session contains `apiUrl`, `downloadUrl`, `accounts`, `primaryAccounts` and capabilities. Default mail account is not necessarily the intended shared account; `isReadOnly` is account metadata | [RFC 8620 §2](https://www.rfc-editor.org/rfc/rfc8620.html#section-2) |
| Enumerate mailboxes | `Mailbox/get`, requesting IDs, names, parent/role and `myRights`; rights describe the caller's access | [RFC 8621 §§2, 2.1](https://www.rfc-editor.org/rfc/rfc8621.html#section-2.1) |
| Search | `Email/query` returns Email IDs; `inMailbox`, `text`, date filters and sorting are specified. Position/limit paging and query state are not a transactionally frozen snapshot | [RFC 8621 §4.4](https://www.rfc-editor.org/rfc/rfc8621.html#section-4.4), [RFC 8620 §§3.9, 5.5](https://www.rfc-editor.org/rfc/rfc8620.html#section-5.5) |
| Retrieve metadata | `Email/get` retrieves selected properties including whole-message `blobId`, `threadId`, headers, received/sent time and body structure; honor `notFound` rather than omitting missing objects | [RFC 8621 §§4.1–4.2](https://www.rfc-editor.org/rfc/rfc8621.html#section-4.2), [RFC 8620 §5.1](https://www.rfc-editor.org/rfc/rfc8620.html#section-5.1) |
| Raw RFC 5322 | Email's `blobId` represents raw message octets. Expand Session `downloadUrl` with `accountId`, `blobId`, `type` and `name`; authenticated GET. `type=message/rfc822` is representation metadata, not a MIME reconstruction request | [RFC 8621 §4.1.1](https://www.rfc-editor.org/rfc/rfc8621.html#section-4.1.1), [RFC 8620 §6.2](https://www.rfc-editor.org/rfc/rfc8620.html#section-6.2) |
| Thread membership | `Thread/get` returns Email IDs; thread grouping algorithm is not standardized and a thread can span mailboxes. Export must recheck authorization on every member | [RFC 8621 §3](https://www.rfc-editor.org/rfc/rfc8621.html#section-3) |

Read these standards through the primary publication sources if RFC-site access
is unavailable: [core XML](https://github.com/jmapio/jmap/blob/master/rfc/src/rfc8620.xml),
[mail XML](https://github.com/jmapio/jmap/blob/master/rfc/src/rfc8621.xml).

The whole-message blob is different from an attachment-part blob. JMAP parsed
headers and `bodyValues` can transform or replace bytes; they are not substitutes
for raw evidence ([RFC 8621 §§4.1.2.1, 4.1.4](https://www.rfc-editor.org/rfc/rfc8621.html#section-4.1.4)).
`receivedAt` is store receipt/internal time; `sentAt` is parsed sender Date,
not verified sending time ([RFC 8621 §§4.1.1, 4.1.3](https://www.rfc-editor.org/rfc/rfc8621.html#section-4.1.3)).
Byte-for-byte fidelity of the actual deployment is **UNVERIFIED** until the
synthetic comparison in [study 05](05-evidence-export.md).

### Authentication options

| Option | Verified Stalwart documentation | Read-only qualification |
| --- | --- | --- |
| Account password | Mail access uses account passwords, app passwords or OAuth, not management API keys. [A1] | Full user's credential is not inherently least privilege |
| Application password | Named/revocable credentials; current docs describe expiry, source restrictions and `Inherit`, `Disable`, `Replace` permission modes. Owners create them; administrators cannot create on another user's behalf. [A2] | `Inherit` does not mean read-only. Version-specific effective restrictive permissions **UNVERIFIED** |
| OAuth | JMAP authentication; authorization-code/device flows; metadata at `/.well-known/oauth-authorization-server`, token issuance/refresh at `/auth/token`; IMAP `OAUTHBEARER` / `XOAUTH2` subject to client support. [A3] | Exact read-only mail scope and enforcement **UNVERIFIED**; do not guess scope strings |
| Management API key | Explicitly cannot log into JMAP mail, IMAP, POP3, SMTP submission or DAV services. It is a management credential. [A1] | **Not** a candidate mail-reader token, even if management permissions are restricted |

- **A1:** [API keys](https://stalw.art/docs/auth/authentication/api-key/);
  [pinned primary file](https://github.com/stalwartlabs/website/blob/8623f64887bb34bd57e22491c5782955069177e5/src/content/docs/docs/auth/authentication/api-key.md).
- **A2:** [app passwords](https://stalw.art/docs/auth/authentication/app-password/);
  [pinned primary file](https://github.com/stalwartlabs/website/blob/8623f64887bb34bd57e22491c5782955069177e5/src/content/docs/docs/auth/authentication/app-password.md);
  [credential permissions reference](https://stalw.art/docs/ref/object/app-password/#credentialpermissions),
  [pinned reference](https://github.com/stalwartlabs/website/blob/8623f64887bb34bd57e22491c5782955069177e5/src/content/docs/docs/ref/object/app-password.md).
- **A3:** [interoperability](https://stalw.art/docs/auth/oauth/interoperability/),
  [flows](https://stalw.art/docs/auth/oauth/flows/),
  [endpoints](https://stalw.art/docs/auth/oauth/endpoints/);
  [pinned interoperability](https://github.com/stalwartlabs/website/blob/8623f64887bb34bd57e22491c5782955069177e5/src/content/docs/docs/auth/oauth/interoperability.md),
  [pinned flows](https://github.com/stalwartlabs/website/blob/8623f64887bb34bd57e22491c5782955069177e5/src/content/docs/docs/auth/oauth/flows.md),
  [pinned endpoints](https://github.com/stalwartlabs/website/blob/8623f64887bb34bd57e22491c5782955069177e5/src/content/docs/docs/auth/oauth/endpoints.md).

### Permissions versus mailbox access

Stalwart distinguishes server-wide permissions from per-resource ACLs; permissions
alone are not a demonstrated grant to another user's mailbox
([permissions](https://stalw.art/docs/auth/authorization/permissions/);
[pinned source](https://github.com/stalwartlabs/website/blob/8623f64887bb34bd57e22491c5782955069177e5/src/content/docs/docs/auth/authorization/permissions.md)).
Its built-in `user` role includes sending; `admin` includes every permission.
Custom roles/denials are documented
([roles](https://stalw.art/docs/auth/authorization/roles/);
[pinned source](https://github.com/stalwartlabs/website/blob/8623f64887bb34bd57e22491c5782955069177e5/src/content/docs/docs/auth/authorization/roles.md)).
`impersonate` is explicitly broad mailbox-wide access, not documented
read-only delegation
([administrator/impersonation](https://stalw.art/docs/auth/authorization/administrator/#impersonation);
[pinned source](https://github.com/stalwartlabs/website/blob/8623f64887bb34bd57e22491c5782955069177e5/src/content/docs/docs/auth/authorization/administrator.md)).

The generated [permission reference](https://stalw.art/docs/ref/permissions/)
([pinned source](https://github.com/stalwartlabs/website/blob/8623f64887bb34bd57e22491c5782955069177e5/src/content/docs/docs/ref/permissions.md))
lists `authenticate`, `jmapMailboxGet`, `jmapEmailQuery`, `jmapEmailGet`,
`jmapBlobGet`. These are **candidate** read permissions, not a verified complete
allowlist. Session/discovery, `Thread/get`, raw-download and effective credential
permission interactions remain **UNVERIFIED**. Do not grant `fetchAnyBlob`,
which the reference describes as fetching arbitrary blobs including unowned ones.
Explanatory docs and generated reference use differing identifier styles
(kebab-case versus camelCase); do not turn this list into untested configuration.

### IMAP alternative

Stalwart's reference distinguishes `imapExamine` (read-only opening),
`imapSelect` (read-write opening), and `imapFetch`
([permission reference](https://stalw.art/docs/ref/permissions/)).
Standards-based non-mutating choices are `EXAMINE`, `UID SEARCH` and
`UID FETCH ... BODY.PEEK[]`; `BODY.PEEK` avoids setting Seen
([RFC 9051 §§6.3.2, 6.4.4–6.4.5](https://www.rfc-editor.org/rfc/rfc9051.html#section-6.4.5)).

IMAP ACL `l` enables lookup and `r` enables reading/searching; avoid `s`
(Seen changes), `w`, `i`, `p`, `k`, `x`, `t`, `e`, `a` for shared-source
read-only access ([RFC 4314 §2.1](https://www.rfc-editor.org/rfc/rfc4314.html#section-2.1)).
`r` can supply source content for a COPY; destination rights and other protocol
permissions still matter. `EXAMINE` alone does not confine a credential.
IMAP identifiers are UID/UIDVALIDITY-bound, not JMAP Email IDs
([RFC 9051 §2.3.1.1](https://www.rfc-editor.org/rfc/rfc9051.html#section-2.3.1.1)).

## Recommendations

### Least-privilege provisioning procedure — validate before relying on it

This is a proposed operator workflow, **not** a verified click-by-click or
API-payload recipe:

1. Pin Stalwart version/directory backend; consult its matching permission and
   sharing docs. Create a dedicated reader principal (synthetic naming example:
   `user@example.test`), with no admin/user-role inheritance that restores writes.
2. Grant only lookup/read access to selected source mailboxes, not full-owner
   access or impersonation. IMAP `lr` is a candidate way to express source ACLs;
   exact JMAP sharing payload and Stalwart ACL mapping are **UNVERIFIED**.
   Confirm visible shared account and `Mailbox.myRights` with synthetic mail.
3. Restrict the principal/custom role to tested authentication and needed read
   methods. Discover the actual permission for `Thread/get` and download on this
   version. Do not add broad privileges simply to fix a failed read.
4. Have the reader owner issue a dedicated app password with explicit restrictive
   permission policy, expiry/revocation, and source restrictions if supported
   by that deployment. Alternatively use OAuth only after verifying effective
   least privilege. Do not use the owner's ordinary password or a management key.
5. Store credential outside repo/client prompts, with restricted OS access.
   Allow only operator-configured TLS endpoints, approved account/mailbox IDs,
   discovery/download origins; do not expose secret-bearing URLs in responses.
6. Test allowed reads AND forbidden writes with the same credential, independently
   of MCP: Email set/create/update/destroy, import/copy, mailbox mutations,
   submission/upload/admin and alternate-protocol writes. Disable unused
   protocol permissions, including SMTP/IMAP if JMAP-only. Ensure reads/export
   leave keywords, membership and bytes unchanged; test token revocation too.

Exact commands, role syntax, sharing payload and minimum permission recipe
remain **UNVERIFIED** until this procedure passes on the owner's version.
If it cannot pass, stop: a server-method allowlist is defense in depth, not
a substitute for upstream confinement.

Prefer JMAP-only V1; IMAP is a separately reviewed alternative, never an automatic
failure fallback. Its read choices and ACLs need the same negative tests and raw-
byte verification. Resolve these blockers in [study 08](08-open-questions.md).
