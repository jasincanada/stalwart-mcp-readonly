# 05 — Evidence export specification

Status: proposed acquisition format v1, not a legal-admissibility claim.
Reviewed 2026-10-07.

## Findings

- JMAP distinguishes an Email's raw-message `blobId`, parsed headers/body
  structure, received time, sent time and thread identity
  ([RFC 8621 §4.1.1](https://www.rfc-editor.org/rfc/rfc8621.html#section-4.1.1)).
  Download is defined separately from JSON object retrieval
  ([RFC 8620 §6.2](https://www.rfc-editor.org/rfc/rfc8620.html#section-6.2)).
- RFC 5322 specifies message syntax, including the Date field; do not treat
  parsed JSON as the original message
  ([RFC 5322 §§2, 3.6.1](https://www.rfc-editor.org/rfc/rfc5322.html#section-3.6.1)).
- SHA-256 is defined by NIST's Secure Hash Standard
  ([FIPS 180-4](https://doi.org/10.6028/NIST.FIPS.180-4)).
  CSV field quoting and record conventions are described in
  [RFC 4180 §2](https://www.rfc-editor.org/rfc/rfc4180.html#section-2).

**UNVERIFIED:** byte fidelity of the owner's particular Stalwart version,
storage pipeline and JMAP download path. Verify with a synthetic fixture:
compare known retained blob octets with authenticated download and export.
Even passing that test cannot establish that the retained blob is the original
SMTP wire stream or that the message was never modified before acquisition.

Research access note: RFC text was read through the primary
[JMAP core](https://github.com/jmapio/jmap/blob/master/rfc/src/rfc8620.xml) and
[mail](https://github.com/jmapio/jmap/blob/master/rfc/src/rfc8621.xml) publication
sources because direct RFC-site access was blocked. NIST's
[bibliographic record](https://github.com/usnistgov/NIST-Tech-Pubs/blob/master/bib/NIST.FIPS.180-4.ris)
confirms the FIPS citation; independent inspection of its algorithm text is
**UNVERIFIED** because the PDF fetch was blocked. Verify against the linked
primary publication and published SHA-256 test vectors before implementation.

## Recommendations

### Claim and boundaries

“Evidence-grade” here means reproducible acquisition of the **server-retained
raw message bytes**, with documented scope, failures and verifiable integrity.
It does not establish sender identity, truth of the contents, delivery history,
trusted capture time, a complete conversation, or admissibility in a jurisdiction.
Owner/legal review determines whether signing, trusted timestamps or a formal
custody workflow are necessary.

Export root is operator-controlled and **outside this repository**. Never place
mail, addresses, credentials, or operational endpoint details into the repository.
Use synthetic examples such as `user@example.test` in documentation/tests.

### Acquisition and completion

1. Authenticate as the constrained principal; record configured scope using a
   non-network `source_label` (for example `synthetic-source`), software version,
   UTC acquisition start and thread ID. Record distinct Email and Thread state
   tokens from `Email/get` and `Thread/get`, respectively, for that account
   ([RFC 8620 §5.1](https://www.rfc-editor.org/rfc/rfc8620.html#section-5.1)).
   Session `state` and search `queryState` are not substitutes for these tokens.
2. Use `Thread/get` to freeze the accessible thread's Email ID set, record missing
   IDs/errors, and check mailbox/account authorization per Email. Do not group by
   subject or assume matching Message-ID means the same Email object.
3. Get each Email's whole-message `blobId`, metadata and body structure.
   Download raw blobs into exclusively created temporary files. Request an
   uncompressed representation; any HTTP transfer/content decoding belongs to
   the HTTP layer, not a MIME transformation. Hash and store the **blob octets**,
   not HTTP framing or compressed transfer bytes. Fixture-test this boundary.
   HTTP Content-Encoding and MIME Content-Transfer-Encoding are different
   layers ([RFC 9110 §8.4](https://www.rfc-editor.org/rfc/rfc9110.html#section-8.4)).
4. No newline conversion, UTF-8 conversion, header unfolding, MIME decoding,
   mbox escaping, reserialization, BOM insertion, antivirus rewriting or
   attachment stripping of `.eml` files. Stream SHA-256 over the same bytes
   written; compare independent disk rehash before finalizing.
5. Optionally store separately decoded attachment payloads, without changing
   `.eml`. Write both CSVs, then recheck accessible thread membership and message
   blob mappings with fresh `Thread/get` and `Email/get`. Require consistent
   per-type state throughout acquisition and compare before/after tokens within
   the same type/account; fail conservatively on changes. Record both types'
   tokens, not a single ambiguous “account state.”
6. Close and sync files; create `ARTIFACTS.csv` inventory; rehash inventory;
   publish directory by a same-filesystem no-overwrite atomic operation. Only
   then return `status: complete` and its inventory SHA-256.

No JMAP transaction spanning reads is assumed. Before/after checks reduce races,
not prove absence of an intermediate change. `complete` means every object in
the recorded, authorized, accessible membership set was acquired; hidden mail,
deleted historical messages and other accounts are not covered. A permissions
failure or known inaccessible ID prevents complete status. Never silently shrink
membership when a read fails.

Failure, cancellation, changed membership/blob, malformed metadata preventing
inventory, or limit breach leaves an `incomplete-<exportId>` directory, with CSV
failure rows where possible and no completed publication. Incomplete exports
are inspectable by the operator but must not be presented as successful evidence.
Preserve acquired bytes; operator retention policy controls later deletion.
A retry creates a fresh export ID, never overwrites or “repairs” prior evidence.

### Layout and filenames

- Directory: `export-<exportId>/`, with 32 lowercase hexadecimal random characters
  generated by the server; incomplete staging is not a completed export.
- Raw files: `messages/000001_20261007T120000000Z_<full-sha256>.eml`.
  This is a fake timestamp/placeholder naming example, not a literal final name.
- Assign ordinal after sorting frozen metadata by valid `receivedAt` ascending,
  then opaque Email ID bytewise; null received times last. Timestamp is received
  time converted to UTC milliseconds, **not** the sender's Date header.
  Use `undated` when received time is unavailable; retain full precision in CSV.
- Optional attachment files: `attachments/000001_0001_<full-sha256>.bin`.
  Ordinals bind to the message and MIME part traversal order.
- Generated relative paths use only ASCII and known separators. Sender filenames,
  subject, Message-ID, thread/account IDs and caller input never become paths.
  No symlink traversal, absolute paths, dot segments, overwrites or shell use.

### CSV representation (normative proposal)

All CSVs: UTF-8 without BOM, CRLF records, a header row, all fields double-quoted,
embedded quotes doubled, decimal non-negative numbers, UTC RFC 3339 timestamps,
lowercase hexadecimal SHA-256. Null = empty field; zero is not null.
Schema is versioned as `1`. Reject unexpected columns when verifying v1.

To keep evidence CSVs from becoming spreadsheet instructions, **all untrusted
free text and opaque JMAP identifiers** use standard base64 of UTF-8 in columns
ending `_b64`. Raw header bytes use base64 of bytes, not decoded UTF-8.
Decoding is a viewer operation and must not overwrite CSV/evidence. This policy
also removes ambiguity from newlines/control characters. CSV quoting alone is
not our safety control. Server enum/count/hash/path columns have strict
character allowlists. Do not store subject or address summaries just for convenience.

#### `MANIFEST.csv` — one row per frozen Email ID, including failures

| Columns | Meaning |
| --- | --- |
| `schema_version`, `export_id`, `export_status` | `1`, generated ID, `complete` or `incomplete` repeated per row |
| `source_label`, `software_version`, `acquisition_started_at`, `acquisition_finished_at` | Operator non-sensitive label, release identifier and local UTC times; not trusted timestamps |
| `account_id_b64`, `thread_id_b64`, `email_id_b64`, `blob_id_b64`, `mailbox_ids_b64` | Opaque identifiers; mailbox IDs = base64 of a UTF-8 JSON array of strings, sorted bytewise |
| `email_state_before_b64`, `email_state_after_b64`, `thread_state_before_b64`, `thread_state_after_b64`, `membership_sha256` | Per-type `Email/get` and `Thread/get` state tokens and membership hash defined below; empty after-state on failure |
| `ordinal`, `relative_path`, `byte_length`, `sha256` | Actual .eml artifact identity; empty path/length/hash if not successfully acquired and rehashed |
| `received_at`, `sent_at`, `date_status`, `date_headers_b64` | JMAP metadata times; `date_status` enum `valid`, `missing`, `invalid`, `multiple`; base64 of all raw Date header fields, including folding and field order |
| `message_id_headers_b64`, `duplicate_of_ordinal` | Original Message-ID fields as raw bytes; first earlier ordinal with identical .eml hash, or empty |
| `attachment_policy`, `row_status`, `error_code` | `metadata_only` or `extract`; `ok` or `failed`; fixed error enum from study 04, empty for success |

`membership_sha256` hashes UTF-8 JSON of an array of `[emailId, blobId]` pairs,
sorted by the UTF-8 bytes of Email ID. Canonical serialization is exactly:
ASCII array punctuation and commas; no whitespace, BOM or final newline;
double-quoted strings; escape quotation mark as `\"`, backslash as `\\`, and
every U+0000–U+001F as lowercase `\u00xx` (no short escapes); emit all other
Unicode scalar values as literal UTF-8, without slash escaping or Unicode
normalization. Reject invalid Unicode scalar sequences. This fixes escaping as
well as whitespace so independent implementations hash identical bytes.
This is our chosen canonical representation, not a JMAP standard.
Keep the CSV ID fields for independent reconstruction. If blob IDs
cannot all be obtained, leave membership hash empty and status incomplete.

`date_status` comes from raw headers: exactly one parseable Date = `valid`, none =
`missing`, malformed single = `invalid`, more than one = `multiple`. Do not invent,
insert or repair a Date. Sender Date / JMAP `sentAt` is not proof of delivery or
chronology; use received time only as an ordering convenience. Capture raw header
fields without normalizing their octets; parse failures never alter .eml.
Message-ID absence/duplicates are retained, not treated as acquisition identity.

#### `ATTACHMENTS.csv` — one row per attachment-like MIME part

Include explicit attachments and named/inline non-body parts per the pinned
parser policy; document that policy/version. Zero parts still produces a header-only
CSV. If body structure cannot be inventoried, mark export incomplete rather than
claiming there were no attachments.

| Columns | Meaning |
| --- | --- |
| `schema_version`, `export_id`, `email_id_b64`, `message_ordinal`, `part_ordinal`, `part_id_b64`, `blob_id_b64` | Parent binding and stable MIME part identity |
| `filename_b64`, `media_type_b64`, `disposition_b64`, `transfer_encoding_b64` | Untrusted MIME metadata; empty when absent |
| `declared_size`, `decoded_byte_length`, `relative_path`, `sha256` | Metadata size vs actual decoded artifact size/hash; actual fields empty unless extracted |
| `extraction_status`, `error_code` | `not_requested`, `extracted`, `failed`, or `unsupported`; fixed error code or empty |

In metadata-only mode, parts remain preserved within .eml; extraction is explicitly
`not_requested`, not a failure. Extraction mode requires every selected part to
complete, otherwise the export is incomplete. Do not execute, render or unpack
attachments. Bound part depth/count and decoded bytes as in study 04.

Decoded attachment hashes cover **decoded MIME payload octets**, not the encoded
base64/quoted-printable section within .eml. Preserve original MIME encoding in
.eml; do not transcode attachment charsets. Unknown/broken transfer encodings
may be retained in raw mail with `unsupported` extraction, never silently
“fixed.” Verify downloaded attachment blob semantics against a synthetic fixture
before claiming decoded equivalence on the selected Stalwart version.
JMAP part blobs already decode known MIME transfer encodings, and multipart
parts can have null blob IDs; do not decode a downloaded part blob again
([RFC 8621 §4.1.4](https://www.rfc-editor.org/rfc/rfc8621.html#section-4.1.4)).
Record null part blob IDs explicitly as non-extractable; extraction mode cannot
claim success when a selected part cannot be acquired.

### Duplicates, inventory and custody

Distinct Email IDs with identical raw octets get separate rows **and files**;
mark later rows' `duplicate_of_ordinal`. Do not deduplicate by subject,
Message-ID, parsed body or attachment name. Hash equality is an integrity
comparison, not proof the two acquisitions have the same provenance.

`ARTIFACTS.csv` contains `relative_path`, `byte_length`, `sha256`, with one row
for every .eml, extracted .bin, `MANIFEST.csv` and `ATTACHMENTS.csv`, sorted by
relative path bytewise. It excludes itself to avoid a circular hash. It uses the
same CSV encoding rules and contains generated paths only.

Return the inventory's SHA-256 to the operator, who retains it **outside the
export directory** in their custody record. The record should identify collector,
authorized scope, source label, acquisition environment, local clock status,
software version, handoffs and storage controls. Do not put personal custody
records in this repository. Editing both files and in-directory hashes defeats
self-contained tamper detection; an independently retained inventory digest
(or later signature) is essential. An exported digest alone is not authenticity.

### Verification procedure and checklist

Use an independent offline verifier and binary-mode SHA-256 implementation
(for example the operator's existing `sha256sum` utility); do not open .eml
or decoded attachments while verifying. CSVs need a real CSV parser, not
whitespace splitting. The verifier is future work, not added in this PR.

- [ ] Obtain the independently retained inventory digest; hash `ARTIFACTS.csv`
  before trusting its paths or rows.
- [ ] Reject absolute/traversal/duplicate paths and symlinks; enumerate directory
  files and reject unlisted artifacts (inventory itself is the only exception).
- [ ] Check each artifact's binary byte count and SHA-256 against inventory.
- [ ] Parse CSV header/version/quoting/enums and decode `_b64` columns strictly.
- [ ] Require all manifest rows `ok`, all export statuses `complete`, all frozen
  member IDs present exactly once; reconstruct `membership_sha256`.
- [ ] Compare .eml row count, generated ordinals, filenames, hashes and sizes;
  verify duplicate references point backward to equal hashes.
- [ ] Check attachment parent/part references; extracted files' sizes and hashes;
  explicit `not_requested` policy where extraction is disabled.
- [ ] Inspect recorded Date status, time-source limitations, authorization scope,
  state-change/race notes and failures without altering evidence.
- [ ] Record verification time, verifier/version, result and custody handoff in
  an external protected record; preserve original export read-only.

Acceptance fixtures include missing/invalid/multiple Date, duplicate IDs versus
duplicate bytes, folded/encoded headers, mixed line endings, malformed MIME,
binary payloads, partial download, mutation during capture and a one-byte
post-export alteration. Integrity success must never be described as proof of
sender identity or universal legal admissibility.
