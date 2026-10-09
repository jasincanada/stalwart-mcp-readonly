# 10 — Stalwart app passwords and authentication

**Research checked:** 2026-10-07  
**Scope:** Credentials for a Stalwart-backed email MCP, with emphasis on restricting the consequences of prompt injection and credential theft.

**Version reference:** Upstream changelog latest entry checked: **v0.16.25, 2026-10-05**. A deployed server may use another version.

## Executive summary

The current upstream changelog documents app passwords and API keys with limited access, labels, IP restrictions, and expiration dates. It does **not** establish the exact scope/permission model or prove read-only, protocol-only, or mailbox-only restrictions. Stalwart’s official documentation also has specific references for app passwords, API keys, roles, permissions, OAuth/OIDC, 2FA, quotas, and tracing; those documentation pages could not be fetched and their detailed behavior is **UNVERIFIED in this check**. Do not configure an MCP using guessed permission names, endpoints, or configuration keys. Verify against the version-matched pages and test the restriction boundary before deployment. [Stalwart changelog, v0.16.0](https://github.com/stalwartlabs/stalwart/blob/main/CHANGELOG.md#0160---2026-04-20).

The consequence is important: least privilege can be specified as a desired security property, but the precise Stalwart credential that achieves it cannot be named from the verified evidence in this study. Until protocol and identity scopes are verified, a separate account and network-level containment are safer assumptions than “read-only app password.”

## Findings (verified)

### Stalwart-specific credential facts

| Question | Finding |
|---|---|
| Can Stalwart create app-specific passwords? | **Supported in upstream v0.16:** the changelog lists app passwords with limited access, labels, IP address restrictions, and expiration dates. Exact deployment availability is **UNVERIFIED**. [Changelog, v0.16.0](https://github.com/stalwartlabs/stalwart/blob/main/CHANGELOG.md#0160---2026-04-20) |
| How are app passwords created, stored, and shown? | **UNVERIFIED in this check.** The official [App Passwords documentation](https://stalw.art/docs/auth/authentication/app-password/) describes a self-service path at Account → Credentials → App Passwords and secret display at creation; it reportedly stores the secret hashed and does not reveal it later. Confirm the UI, one-time display, and storage claims against the deployed version. |
| What is their format? | **Verified format change:** since v0.16.3 (2026-04-30), app passwords begin with the literal prefix `app_` instead of `app ` (space). This is a prefix, not a complete password or a credential to reuse. [Changelog, v0.16.3](https://github.com/stalwartlabs/stalwart/blob/main/CHANGELOG.md#0163---2026-04-30) |
| Can they be revoked individually? | **UNVERIFIED in this check.** The [App Passwords](https://stalw.art/docs/auth/authentication/app-password/) and [AppPassword object](https://stalw.art/docs/ref/object/app-password/) references describe credential management, but immediate revocation behavior and the effect on existing authenticated sessions must be tested. |
| Which scopes are available? | **Partially verified:** the changelog says “limited access” is supported, and confirms IP restrictions and expiry dates. Whether this means read-only, protocol-specific, mailbox-specific, or method-specific scope is **UNVERIFIED**. Do not infer scope from the phrase “app password.” Test forbidden operations. [Changelog, v0.16.0](https://github.com/stalwartlabs/stalwart/blob/main/CHANGELOG.md#0160---2026-04-20) |
| How do app passwords interact with roles, permissions, quotas, and mailbox ACLs? | **UNVERIFIED in this check.** Official references exist for [roles](https://stalw.art/docs/auth/authorization/roles/), [permissions](https://stalw.art/docs/auth/authorization/permissions/), [administrators](https://stalw.art/docs/auth/authorization/administrator/), and [quotas](https://stalw.art/docs/auth/authorization/quotas/); the exact names, `enabledPermissions`/`disabledPermissions` behavior, `maxAppPasswords` default, and cross-protocol/mailbox ACL enforcement were not verified from page text. See [RFC 4314](https://www.rfc-editor.org/rfc/rfc4314.html) and [RFC 9564](https://www.rfc-editor.org/rfc/rfc9564.html) for ACL standards, not Stalwart implementation guarantees. |
| OAuth 2.0 / OIDC / bearer-token alternatives for JMAP, IMAP, or SMTP? | Stalwart has official [OAuth](https://stalw.art/docs/auth/oauth/) and [OpenID Connect](https://stalw.art/docs/auth/openid/) documentation, but which flow/mechanism is supported for each of JMAP, IMAP, and SMTP is **UNVERIFIED in this check**. Protocol standards include [RFC 8620](https://www.rfc-editor.org/rfc/rfc8620), [RFC 9051](https://www.rfc-editor.org/rfc/rfc9051), [RFC 6409](https://www.rfc-editor.org/rfc/rfc6409), [RFC 6749](https://www.rfc-editor.org/rfc/rfc6749), [RFC 6750](https://www.rfc-editor.org/rfc/rfc6750), and [RFC 7628](https://www.rfc-editor.org/rfc/rfc7628); they do not establish Stalwart deployment support. |
| Management API keys: existence and restrictions? | **Partially verified:** v0.16 changelog lists limited-access API keys, labels, IP restrictions, and expiry. The official [API Keys](https://stalw.art/docs/auth/authentication/api-key/) and [ApiKey object](https://stalw.art/docs/ref/object/api-key/) pages could not be fetched, so creation steps, scopes, storage, revocation, and suitability for mail access are **UNVERIFIED in this check**. |
| Two-factor behavior with app passwords, protocol auth, and API keys? | **UNVERIFIED in this check.** Stalwart documents [two-factor authentication](https://stalw.art/docs/auth/authentication/2fa/) and TOTP ([RFC 6238](https://www.rfc-editor.org/rfc/rfc6238.html)). Confirm whether app passwords bypass interactive 2FA, and whether second factors protect credential creation/recovery, against the deployed version. |
| Audit logging of credential use and changes? | **UNVERIFIED in this check.** Stalwart documents [tracing](https://stalw.art/docs/telemetry/tracing/); authentication event names (`auth.success`, `auth.failed`) appear in documentation search results but were not confirmed from page text. Verify emitted events, fields, retention, integrity, and credential-revocation coverage. |

**Version/security note:** The v0.16.15 changelog reports fixing a scoped-credential privilege-escalation issue involving `SysApiKeyCreate` or `SysApiKeyUpdate` permissions. Use a release containing that fix and do not grant credential-management permissions to an MCP identity without a demonstrated need. This does not establish that all scoped-credential configurations are safe. [Changelog, v0.16.15](https://github.com/stalwartlabs/stalwart/blob/main/CHANGELOG.md#01615---2026-07-26).

Stalwart’s v0.16.3 changelog changed the app-password prefix to `app_`. A wrong, expired, or unknown app password/API key is also mentioned in v0.16.25 IMAP-authentication fixes; this does not document revocation/session semantics. [Changelog](https://github.com/stalwartlabs/stalwart/blob/main/CHANGELOG.md).

**Evidence boundary:** A protocol’s support for authentication is not proof that Stalwart enables a given mechanism or applies the same restrictions to every protocol. The RFCs above are protocol/security standards, not Stalwart feature documentation. For the protocol inventory and raw-message uncertainty, see [09-stalwart-api-capabilities.md](09-stalwart-api-capabilities.md).

## Least-privilege designs (recommendations)

The following are security targets and deployment blueprints, not verified Stalwart procedures. Each place that depends on Stalwart-specific controls is explicitly marked **UNVERIFIED**. “Limited access” app passwords/API keys, IP restrictions, and expiration dates are documented for v0.16, but “read-only” and exact permission names/method mappings have not been verified; none are invented here.

### (a) Read-only MCP

**Target:** A dedicated principal can search/list/read only the intended mailbox data. It cannot change flags, move/delete mail, send, administer accounts, or manage Sieve.

**Least privilege achievable in Stalwart:** **Partially verified; read-only scope remains UNVERIFIED.** Stalwart v0.16 documents limited-access app passwords, labels, IP restrictions, and expiry, but the sources reviewed do not establish a read-only scope or exact read permission. If credentials simply inherit the full account’s rights, a credential labeled “app password” does not reduce the MCP’s authority.

**Placeholder setup blueprint (confirm every marked item before use):**

1. Create a separate mailbox/service principal such as `mcp-read@example.test`; do not reuse a human administrator or the mailbox owner’s everyday credential. **UNVERIFIED:** exact supported account-creation procedure.
2. Grant only read/search access to the required mailbox using the documented account permission or mailbox ACL, if Stalwart supports that distinction. **UNVERIFIED:** permission/ACL names, inheritance, and whether mailbox ACLs cover JMAP and IMAP equally.
3. In the documented account credential area (official docs say Account → Credentials → App Passwords), create an app-specific credential with a label such as `mcp-read`; select the narrowest available scope, an expiry, and an allowed source IP if available. **UNVERIFIED in this check:** deployed UI path and option semantics. If it cannot be constrained to read-only, do not represent it as read-only.
4. Configure the MCP for only the selected read protocol and mailbox. Keep SMTP submission, mailbox mutations, Sieve, and management access unavailable at the process/network level.
5. Test allowed and denied cases with synthetic mail: search, read, raw export; then attempt flag, move, delete, send, and admin operations. Require denials at the server boundary, not merely a hidden MCP tool.
6. Store the generated value securely; official docs reportedly show the secret only at creation and store it hashed. **UNVERIFIED in this check:** one-time display/storage behavior. For rotation, create a replacement, verify it, revoke the old credential, and test that the old value no longer authenticates; revocation and session invalidation semantics are **UNVERIFIED**.

**Stolen credential impact:** The thief can perform every operation the authenticated principal can perform through every reachable enabled protocol until the credential is revoked or otherwise expires. If read-only enforcement is not server-side, this may include changing/deleting mail or sending as the account. This is a threat-model consequence, not a Stalwart-specific claim about current permissions.

### (b) Draft-only write MCP

**Target:** The MCP may prepare drafts but cannot send, alter unrelated mail, or administer the server.

**Least privilege achievable in Stalwart:** **UNVERIFIED.** Limited-access credentials exist, but the reviewed sources do not prove that draft creation can be separated from sending or that a draft-only permission exists.

**Placeholder setup blueprint:**

1. Use a dedicated principal such as `mcp-draft@example.test`, separate from both the mailbox owner and read-only MCP principal. **UNVERIFIED:** exact provisioning procedure.
2. Verify in primary docs and with a test account that draft creation is permitted while submission, send/submit, delete, move, and unrelated mailbox writes are denied. **UNVERIFIED:** exact permission names and whether “draft” is a separately enforceable capability.
3. If the service cannot enforce draft-only rights, place a policy-enforcing proxy in front of the MCP that allows only documented JMAP methods/properties needed to create or update drafts and denies all send/submission and unrelated mutations. The required method names and behavior must come from the deployed-version protocol docs; do not guess them.
4. Keep SMTP submission unreachable from this MCP. Validate that a malicious draft’s recipient/body cannot trigger a separate send path.
5. Rotate/revoke as for the read-only setup; test that the retired credential cannot create or submit drafts. **UNVERIFIED:** exact Stalwart revocation behavior.

**Stolen credential impact:** At minimum, the thief can create or alter content within the principal’s documented rights. If draft/send separation is not enforced server-side, the thief may also send as the account; test that boundary explicitly.

### (c) Full read/write MCP

**Target:** Read/search and explicitly authorized message mutations, optionally including send. This is the broadest and highest-risk profile.

**Least privilege achievable in Stalwart:** **Partially verified.** Limited-access API keys/app passwords exist; exact method-level permission support, their suitability for mail access, and separation of send from read/write were not established.

**Placeholder setup blueprint:**

1. Create a separate, non-administrator MCP principal such as `mcp-rw@example.test`; scope it to a dedicated or shared mailbox where feasible. **UNVERIFIED:** exact user and shared-mailbox ACL procedures.
2. Enumerate the minimum required operations (for example, search/read plus a specifically allowed set of flags or moves). Request exact corresponding permissions only after confirming names and enforcement in official docs; there is no verified permission list to reproduce here.
3. Keep sending disabled unless an explicit requirement exists. If sending is required, treat it as a separate capability with user confirmation and bounded recipients/content policy.
4. Restrict network reachability to the MCP host, apply an allowlist at a trusted proxy/firewall if available, and keep admin/CLI credentials out of the MCP process.
5. Exercise all allowed and denied cases. Audit the exact principal and operation. Rotate and revoke via documented release-specific steps. **UNVERIFIED:** APIs, audit event coverage, and session invalidation.

**Stolen credential impact:** The thief can search/read and perform every enabled mutation within the principal’s reach; if submission is allowed, they may send messages as the account. A broad mailbox ACL can expose or affect shared-mailbox contents. Actual Stalwart authorization outcomes remain **UNVERIFIED** until tested.

### Explicit least-privilege boundary

**Not established / treat as unavailable until proven:** The sources verified here do not demonstrate that Stalwart can issue a credential restricted to read-only mail, draft-only writes, a selected protocol, or selected mailboxes. This is not a claim that Stalwart can never provide those controls; it means none of the three MCP profiles may be deployed on the assumption that it does. If the deployed release cannot enforce the required boundary at the server, Stalwart alone cannot provide that least-privilege profile. Do not enable the affected write/read integration until a test proves the restriction.

## Controls if Stalwart cannot express the required least privilege

Whether each control is available in a deployment is **UNVERIFIED** and must be checked. These are compensating-control recommendations:

- Use separate service accounts and credentials for read-only, draft, and send functions. A separate password for the same unrestricted principal is not sufficient separation.
- Use a dedicated/shared mailbox with a narrowly scoped mailbox ACL only if that ACL is enforced for the selected protocol and every relevant operation.
- Restrict network access to the MCP host or a controlled proxy; do not expose management interfaces or SMTP submission to a read-only process.
- Use a policy proxy that allowlists documented JMAP methods and validates arguments, and denies sending/mutation by default. The proxy must be tested against batch requests, alternate protocol paths, and direct server access.
- Apply external rate/size limits and monitor access if server-native limits or logs are insufficient.
- Prefer a server-side denial over tool descriptions, prompt instructions, per-session consent, or UI hiding alone.

## Sources and verification notes

- Stalwart upstream [CHANGELOG.md](https://github.com/stalwartlabs/stalwart/blob/main/CHANGELOG.md), checked 2026-10-07: v0.16.0 documents limited-access app passwords and API keys with labels, IP restrictions, and expiry; v0.16.3 changes the app-password prefix; v0.16.15 fixes the cited scoped-credential issue.
- Official Stalwart documentation pages: [App Passwords](https://stalw.art/docs/auth/authentication/app-password/), [API Keys](https://stalw.art/docs/auth/authentication/api-key/), [Roles](https://stalw.art/docs/auth/authorization/roles/), [Permissions](https://stalw.art/docs/auth/authorization/permissions/), [OAuth](https://stalw.art/docs/auth/oauth/), [OpenID Connect](https://stalw.art/docs/auth/openid/), [2FA](https://stalw.art/docs/auth/authentication/2fa/), [Quotas](https://stalw.art/docs/auth/authorization/quotas/), [Tracing](https://stalw.art/docs/telemetry/tracing/). Their content could not be directly fetched in this check; page-specific claims are marked **UNVERIFIED in this check**.
- [RFC 8620](https://www.rfc-editor.org/rfc/rfc8620) (JMAP Core), [RFC 9051](https://www.rfc-editor.org/rfc/rfc9051) (IMAP4rev2), and [RFC 6409](https://www.rfc-editor.org/rfc/rfc6409) (message submission) define protocol contexts, not Stalwart feature support.
- [RFC 6749](https://www.rfc-editor.org/rfc/rfc6749) (OAuth 2.0) and [RFC 6750](https://www.rfc-editor.org/rfc/rfc6750) (bearer token standard) define standards, not Stalwart’s provider/configuration.
- **Not verified:** exact app-password workflow/storage/revocation behavior, API key scopes, permission names, OAuth/OIDC support by protocol, MFA behavior, audit events, quota/ACL interactions, and revoke/rotation/session invalidation. Resolve against the linked official documentation and source for the precise deployed release, then test with a non-production principal.
