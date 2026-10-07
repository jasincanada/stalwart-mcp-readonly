# 10 — Stalwart app passwords and authentication

**Research checked:** 2026-10-07  
**Scope:** Credentials for a Stalwart-backed email MCP, with emphasis on restricting the consequences of prompt injection and credential theft.

## Executive summary

The Stalwart project sources reviewed establish protocol support, but do **not** verify the current app-password feature, its format/storage/revocation/scopes, admin API key behavior, exact permission names, OAuth/OIDC support for each protocol, two-factor interactions, or credential-use audit events. These are **UNVERIFIED**, not assumptions that the features are absent. Do not configure an MCP using guessed permission names, endpoints, or configuration keys. Obtain the version-matched official Stalwart documentation and test the restriction boundary before deployment. [Stalwart project README](https://github.com/stalwartlabs/stalwart#readme).

The consequence is important: least privilege can be specified as a desired security property, but the precise Stalwart credential that achieves it cannot be named from the verified evidence in this study. Until protocol and identity scopes are verified, a separate account and network-level containment are safer assumptions than “read-only app password.”

## Findings (verified)

### Stalwart-specific credential facts

| Question | Finding |
|---|---|
| Can Stalwart create app-specific passwords? | **UNVERIFIED.** The reviewed upstream README does not document this feature. Verify in current official account/admin documentation for the exact deployed release. |
| How are app passwords created, formatted, stored, and shown? | **UNVERIFIED.** No format, display/recovery policy, hash/storage scheme, or creation API is asserted here. Verify the official version-matched documentation and implementation. |
| Can they be revoked individually? | **UNVERIFIED.** Test revocation and confirm outstanding sessions/tokens are invalidated or expire as documented. |
| Can an app password be read-only, protocol-specific, mailbox-specific, IP-restricted, or expiring? | **UNVERIFIED** for every scope. Do not infer scope from the term “app password.” Verify actual authorization behavior using attempted forbidden operations. |
| How do app passwords interact with account roles, permissions, quotas, and mailbox ACLs? | **UNVERIFIED.** The exact role/permission names and whether an alternate password inherits the account’s full rights must be checked in primary Stalwart documentation for the installed version. |
| OAuth 2.0 / OIDC / bearer-token alternatives for JMAP, IMAP, or SMTP? | **UNVERIFIED** per protocol and deployment. RFCs define protocol-level authorization mechanisms and token formats, but do not establish Stalwart support or configuration. See [RFC 8620](https://www.rfc-editor.org/rfc/rfc8620), [RFC 9051](https://www.rfc-editor.org/rfc/rfc9051), [RFC 6409](https://www.rfc-editor.org/rfc/rfc6409), [RFC 6749](https://www.rfc-editor.org/rfc/rfc6749), and [RFC 6750](https://www.rfc-editor.org/rfc/rfc6750). |
| Management API keys: creation, scope, storage, and revocation? | **UNVERIFIED.** Do not treat any administrative credential as a narrow API key until its documented scope and enforcement are verified. |
| Two-factor behavior with app passwords, protocol auth, and API keys? | **UNVERIFIED.** Confirm whether second factors protect only interactive login or also credential creation/recovery and whether app passwords bypass interactive MFA by design. |
| Audit logging of credential use and changes? | **UNVERIFIED.** Confirm which events are logged, fields retained, log integrity/retention, and whether failed authentication and app-password revocation are audited. |

**Evidence boundary:** A protocol’s support for authentication is not proof that Stalwart enables a given mechanism, exposes a specific app-password workflow, or applies the same restrictions to every protocol. The RFCs above are protocol/security standards, not Stalwart feature documentation. For the protocol inventory and raw-message uncertainty, see [09-stalwart-api-capabilities.md](09-stalwart-api-capabilities.md).

## Least-privilege designs (recommendations)

The following are security targets and deployment blueprints, not verified Stalwart procedures. Each place that depends on Stalwart-specific controls is explicitly marked **UNVERIFIED**. “Exact permissions required” cannot be supplied safely until official permission names and method-to-permission mapping are verified; none are invented here.

### (a) Read-only MCP

**Target:** A dedicated principal can search/list/read only the intended mailbox data. It cannot change flags, move/delete mail, send, administer accounts, or manage Sieve.

**Least privilege achievable in Stalwart:** **UNVERIFIED.** No verified read-only app-password scope or exact read permission is established here. If credentials simply inherit the full account’s rights, a credential labeled “app password” does not reduce the MCP’s authority.

**Placeholder setup blueprint (confirm every marked item before use):**

1. Create a separate mailbox/service principal such as `mcp-read@example.test`; do not reuse a human administrator or the mailbox owner’s everyday credential. **UNVERIFIED:** exact supported account-creation procedure.
2. Grant only read/search access to the required mailbox using the documented account permission or mailbox ACL, if Stalwart supports that distinction. **UNVERIFIED:** permission/ACL names, inheritance, and whether mailbox ACLs cover JMAP and IMAP equally.
3. Create an app-specific credential only if official documentation confirms its scope. If it cannot be constrained to read-only, do not represent it as read-only. **UNVERIFIED:** creation UI/API, protocol binding, expiry, and secret format.
4. Configure the MCP for only the selected read protocol and mailbox. Keep SMTP submission, mailbox mutations, Sieve, and management access unavailable at the process/network level.
5. Test allowed and denied cases with synthetic mail: search, read, raw export; then attempt flag, move, delete, send, and admin operations. Require denials at the server boundary, not merely a hidden MCP tool.
6. Store the secret in the deployment’s approved secret store; rotate by issuing a replacement, verifying it, then revoking the old credential. **UNVERIFIED:** Stalwart’s rotation/revocation procedure and session invalidation timing.

**Stolen credential impact:** The thief can perform every operation the authenticated principal can perform through every reachable enabled protocol until the credential is revoked or otherwise expires. If read-only enforcement is not server-side, this may include changing/deleting mail or sending as the account. This is a threat-model consequence, not a Stalwart-specific claim about current permissions.

### (b) Draft-only write MCP

**Target:** The MCP may prepare drafts but cannot send, alter unrelated mail, or administer the server.

**Least privilege achievable in Stalwart:** **UNVERIFIED.** The reviewed sources do not prove that draft creation can be separated from sending or that a permission exists for draft-only writes.

**Placeholder setup blueprint:**

1. Use a dedicated principal such as `mcp-draft@example.test`, separate from both the mailbox owner and read-only MCP principal. **UNVERIFIED:** exact provisioning procedure.
2. Verify in primary docs and with a test account that draft creation is permitted while submission, send/submit, delete, move, and unrelated mailbox writes are denied. **UNVERIFIED:** exact permission names and whether “draft” is a separately enforceable capability.
3. If the service cannot enforce draft-only rights, place a policy-enforcing proxy in front of the MCP that allows only documented JMAP methods/properties needed to create or update drafts and denies all send/submission and unrelated mutations. The required method names and behavior must come from the deployed-version protocol docs; do not guess them.
4. Keep SMTP submission unreachable from this MCP. Validate that a malicious draft’s recipient/body cannot trigger a separate send path.
5. Rotate/revoke as for the read-only setup; test that the retired credential cannot create or submit drafts. **UNVERIFIED:** exact Stalwart revocation behavior.

**Stolen credential impact:** At minimum, the thief can create or alter content within the principal’s documented rights. If draft/send separation is not enforced server-side, the thief may also send as the account; test that boundary explicitly.

### (c) Full read/write MCP

**Target:** Read/search and explicitly authorized message mutations, optionally including send. This is the broadest and highest-risk profile.

**Least privilege achievable in Stalwart:** **UNVERIFIED.** Exact method-level permission support, app-password scoping, and separation of send from read/write were not established.

**Placeholder setup blueprint:**

1. Create a separate, non-administrator MCP principal such as `mcp-rw@example.test`; scope it to a dedicated or shared mailbox where feasible. **UNVERIFIED:** exact user and shared-mailbox ACL procedures.
2. Enumerate the minimum required operations (for example, search/read plus a specifically allowed set of flags or moves). Request exact corresponding permissions only after confirming names and enforcement in official docs; there is no verified permission list to reproduce here.
3. Keep sending disabled unless an explicit requirement exists. If sending is required, treat it as a separate capability with user confirmation and bounded recipients/content policy.
4. Restrict network reachability to the MCP host, apply an allowlist at a trusted proxy/firewall if available, and keep admin/CLI credentials out of the MCP process.
5. Exercise all allowed and denied cases. Audit the exact principal and operation. Rotate and revoke via documented release-specific steps. **UNVERIFIED:** APIs, audit event coverage, and session invalidation.

**Stolen credential impact:** The thief can search/read and perform every enabled mutation within the principal’s reach; if submission is allowed, they may send messages as the account. A broad mailbox ACL can expose or affect shared-mailbox contents. Actual Stalwart authorization outcomes remain **UNVERIFIED** until tested.

## Controls if Stalwart cannot express the required least privilege

Whether each control is available in a deployment is **UNVERIFIED** and must be checked. These are compensating-control recommendations:

- Use separate service accounts and credentials for read-only, draft, and send functions. A separate password for the same unrestricted principal is not sufficient separation.
- Use a dedicated/shared mailbox with a narrowly scoped mailbox ACL only if that ACL is enforced for the selected protocol and every relevant operation.
- Restrict network access to the MCP host or a controlled proxy; do not expose management interfaces or SMTP submission to a read-only process.
- Use a policy proxy that allowlists documented JMAP methods and validates arguments, and denies sending/mutation by default. The proxy must be tested against batch requests, alternate protocol paths, and direct server access.
- Apply external rate/size limits and monitor access if server-native limits or logs are insufficient.
- Prefer a server-side denial over tool descriptions, prompt instructions, per-session consent, or UI hiding alone.

## Sources and verification notes

- Stalwart upstream [README](https://github.com/stalwartlabs/stalwart#readme), checked 2026-10-07; moving branch and not a credential reference.
- [RFC 8620](https://www.rfc-editor.org/rfc/rfc8620) (JMAP Core), [RFC 9051](https://www.rfc-editor.org/rfc/rfc9051) (IMAP4rev2), and [RFC 6409](https://www.rfc-editor.org/rfc/rfc6409) (message submission) define protocol contexts, not Stalwart feature support.
- [RFC 6749](https://www.rfc-editor.org/rfc/rfc6749) (OAuth 2.0) and [RFC 6750](https://www.rfc-editor.org/rfc/rfc6750) (bearer token standard) define standards, not Stalwart’s provider/configuration.
- **Not verified:** app-password workflow/semantics, API key scopes, exact permissions, OAuth/OIDC support by protocol, MFA behavior, audit events, quotas/ACL interaction, and revoke/rotation/session invalidation. Resolve against official documentation and source for the precise deployed release, then test with a non-production principal.
