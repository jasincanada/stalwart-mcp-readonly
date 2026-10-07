# 06 — Transports and clients

Status: official-doc/source snapshot, not a tested compatibility certification.
Reviewed 2026-10-07. Product names and versions matter.

## Findings

### MCP revision boundary

The official documentation navigation identifies **2026-07-28** as latest at
research time ([navigation source](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/docs.json)).
This study distinguishes that revision from the **2025-11-25** tools contract
used as the provisional schema baseline in [study 04](04-tool-design.md).
Do not assume all clients accept either revision.

| Transport | Verified behavior | Authentication boundary |
| --- | --- | --- |
| stdio | Client launches local subprocess; newline-delimited JSON-RPC through stdin/stdout; diagnostics on stderr, never stdout. [T1] | No HTTP OAuth handshake; official authorization guidance says credentials should come from environment. Local process trust/isolation is operator responsibility. [T3, S1] |
| Streamable HTTP, 2025-11-25 | POST requests; JSON or SSE responses; optional GET stream and `MCP-Session-Id` sessions. [T2] | HTTP authentication plus per-request access controls; session IDs are not bearer authorization. Origin validation/local binding guidance applies. [T2, S2] |
| Streamable HTTP, 2026-07-28 | POST-based; JSON or request-scoped SSE; revision removes protocol sessions and standalone GET stream, replacing long-lived notification handling with `subscriptions/listen`. Legacy initialization/session assumptions must not be carried over. [T1, T4] | OAuth-based MCP resource server model if HTTP authorization is supported; tokens must be intended for this server, not forwarded upstream. [T3, S2] |
| Legacy HTTP+SSE | Separate older transport from 2024-11-05, replaced by Streamable HTTP in 2025-03-26. SSE encoding within Streamable HTTP is **not** proof of legacy compatibility. [T1] | Legacy client support/auth must be verified separately |

- **T1:** [2026-07-28 transports](https://modelcontextprotocol.io/specification/2026-07-28/basic/transports),
  [stdio source](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2026-07-28/basic/transports/stdio.mdx),
  [HTTP source](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2026-07-28/basic/transports/streamable-http.mdx).
- **T2:** [2025-11-25 transport source](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2025-11-25/basic/transports.mdx).
- **T3:** [2026-07-28 authorization](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization),
  [source](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2026-07-28/basic/authorization/index.mdx).
- **T4:** [2026-07-28 changelog](https://modelcontextprotocol.io/specification/2026-07-28/changelog),
  [source](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2026-07-28/changelog.mdx),
  [modern/legacy versioning](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2026-07-28/basic/versioning.mdx).
- **S1:** [official local-server security](https://modelcontextprotocol.io/docs/2026-07-28/tutorials/security/local-server-security),
  [source](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/docs/2026-07-28/tutorials/security/local-server-security.mdx).
- **S2:** [authorization security considerations](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/security-considerations),
  [source](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2026-07-28/basic/authorization/security-considerations.mdx).

### Client matrix — do not confuse MCP clients with servers

“Documented” means inspected official source, not a successful handshake.
Every row's exact protocol revision, maximum payload and sensitive-operation
approval behavior with **our** server is **UNVERIFIED** until tested.

| Client/product | stdio | Streamable HTTP / legacy SSE | Authentication evidence | Primary sources / limits |
| --- | --- | --- | --- | --- |
| **Official Grok Build**, command `grok` | Documented; default | HTTP documented and `StreamableHttpClientTransport` present; guide also has SSE option, but complete legacy fallback **UNVERIFIED** | Remote headers and browser OAuth implementation; other OAuth/client-auth modes **UNVERIFIED** | [G1–G3]. A separate official “Grok CLI” product beyond this executable **UNVERIFIED** |
| **Perplexity consumer / Computer** as client | **UNVERIFIED** | Custom connector transport and legacy support **UNVERIFIED** | Custom connector OAuth/header choices and plan restrictions **UNVERIFIED** | [P1]. Vendor pages discovered but not readable; do not infer local stdio from their API MCP server |
| **Perplexity Agent API** remote MCP tool/connector client | **UNVERIFIED**; do not assume a cloud API can launch local processes | Exact transport matrix **UNVERIFIED** | Connector auth choices **UNVERIFIED** | [P2]. Need readable official docs and authorized synthetic test |
| **Claude Desktop** | Local subprocess setup documented | Remote custom connectors documented; exact remote HTTP/SSE revision matrix **UNVERIFIED** | Remote OAuth/header/plan matrix **UNVERIFIED** in this research; local mail secret belongs in server config | [C1]. Desktop is not product-wide “stdio only”; local setup and cloud connector are different paths |
| **Claude Code** | Documented | Official repository documents HTTP and SSE; precise current remote wire/revision compatibility **UNVERIFIED** | Browser OAuth and headers documented; complete client-auth matrix **UNVERIFIED** | [C2]. Changelog documents 2026-07-28 local stdio negotiation with legacy opt-out; do not assume same remote behavior |
| **VS Code / GitHub Copilot MCP integration** | Command/args/env configuration documented | “HTTP Stream” first, SSE fallback documented | Headers, browser OAuth/client-ID configuration; enterprise-managed auth labelled Preview | [V1]. Exact revision and confidential-client/mTLS support **UNVERIFIED** |

**Grok primary sources**

- **G1:** [official repository README](https://github.com/xai-org/grok-build/blob/main/README.md),
  [MCP user guide](https://github.com/xai-org/grok-build/blob/main/crates/codegen/xai-grok-pager/docs/user-guide/07-mcp-servers.md).
- **G2:** [HTTP transport implementation](https://github.com/xai-org/grok-build/blob/main/crates/codegen/xai-grok-mcp/src/servers.rs).
- **G3:** [OAuth implementation](https://github.com/xai-org/grok-build/blob/main/crates/codegen/xai-grok-mcp/src/oauth.rs),
  [vendor verification page](https://docs.x.ai/build/features/mcp-servers).

**Perplexity primary sources and verification paths**

- **P1:** [Computer MCP page](https://docs.perplexity.ai/docs/getting-started/integrations/computer-mcp-server).
- **P2:** [Agent API MCP](https://docs.perplexity.ai/docs/agent-api/tools/mcp),
  [connectors](https://docs.perplexity.ai/docs/agent-api/tools/connectors).

Separate product: Perplexity's **API MCP server** is not proof that Perplexity
can consume this mail server. Its official repository documents local stdio,
self-hosted Streamable HTTP and hosted remote Streamable HTTP/API-key integration
([README](https://github.com/perplexityai/modelcontextprotocol/blob/main/README.md),
[stdio entrypoint](https://github.com/perplexityai/modelcontextprotocol/blob/main/src/index.ts),
[HTTP entrypoint](https://github.com/perplexityai/modelcontextprotocol/blob/main/src/http.ts)).
Its self-hosted HTTP security notice says inbound callers are **not authenticated**;
the configured API key authorizes outbound API use
([official SECURITY.md](https://github.com/perplexityai/modelcontextprotocol/blob/main/SECURITY.md)).
This is a useful auth-boundary warning, not a pattern to adopt. Hosted OAuth
behavior remains **UNVERIFIED**; check
[official integration docs](https://docs.perplexity.ai/docs/getting-started/integrations/mcp-server).

**Other-client sources**

- **C1:** official MCP walkthroughs:
  [local Desktop setup](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/docs/2026-07-28/develop/connect-local-servers.mdx),
  [remote connectors](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/docs/2026-07-28/develop/connect-remote-servers.mdx);
  vendor verification:
  [Desktop local servers](https://support.claude.com/en/articles/10949351-getting-started-with-local-mcp-servers-on-claude-desktop),
  [remote connectors](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp),
  [authentication](https://claude.com/docs/connectors/building/authentication).
- **C2:** Anthropic repository
  [transport configuration reference](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/mcp-integration/SKILL.md),
  [auth reference](https://github.com/anthropics/claude-code/blob/main/plugins/plugin-dev/skills/mcp-integration/references/authentication.md),
  [changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md);
  verify exact current behavior against [Claude Code MCP docs](https://code.claude.com/docs/en/mcp).
- **V1:** [Microsoft configuration source](https://github.com/microsoft/vscode-docs/blob/main/docs/agents/reference/mcp-configuration.md),
  [published reference](https://code.visualstudio.com/docs/agents/reference/mcp-configuration).

### Research limitations

Direct vendor/spec website fetches often failed DNS resolution. Official repository
files above were readable; search snippets were not accepted as proof.
No credentials, real mail, endpoint probes or client installations were used.
Rolling `main` docs may change: pin selected client release and spec/source refs
when testing. Transport support alone does not prove schema, file handling,
authorization or revision compatibility.

## Recommendations

### Local-first V1

Use stdio, a restricted OS identity and an operator-controlled export root outside
the repository. Upstream JMAP access still uses TLS and constrained mail
credentials. stdio avoids an inbound HTTP listener; it does not sandbox the
process or prevent AI-provider disclosure.

Select one owner-approved client and **pin the protocol revision before coding**.
Study 04 uses 2025-11-25 tool semantics provisionally, not a claim that it is latest
or supported by every current client. If 2026-07-28 is required, review the modern
tool/lifecycle/schema contract and update this design; never mix legacy
initialization/session handling with modern requests.

### If remote is required

Treat it as additional scope with a separate review:

- TLS, explicit inbound authentication, per-user account binding, audience
  validation, PKCE and current resource/authorization discovery requirements.
- Separate MCP bearer credentials from upstream JMAP secrets; no token
  passthrough. A user's mail password must not be a tool argument.
- Origin/Host controls, rate limits, egress restrictions, discovery/redirect/DNS
  SSRF defenses, proxy timeouts and redacted logs
  ([official security best practices](https://modelcontextprotocol.io/docs/2026-07-28/tutorials/security/security_best_practices)).
- Follow selected revision's request/session rules. Legacy session IDs never
  replace authentication; modern HTTP must not invent the removed protocol session.
- Decide principal isolation, retention, artifact ownership and authorization.
  Server paths are **not client-local paths**. V1 returns an export ID to the
  operator, not a public download URL. A remote authenticated artifact retrieval
  service would need its own design; do not add one implicitly.

Remote callers, cross-user leakage, Internet-exposed parsers, OAuth discovery,
reverse proxies, cloud-model disclosure and remote artifact retention extend
the [threat model](03-threat-model.md). Keep remote off the Christmas MVP unless
the owner accepts the cost and corresponding cuts.

### Compatibility acceptance procedure

For each exact client/version: record official docs/release ref, selected revision,
transport handshake/version selection, six-tool listing, strict schema behavior,
untrusted-label display, host-mediated read/export grants, payload-limit failure,
safe errors/cancellation, offline evidence verification and secret-redacted logs.
Use synthetic mail only. If remote: also test expired/wrong-audience tokens,
cross-user IDs, invalid Origin, malicious discovery/redirects, and artifact
authorization. “Connects successfully” is not enough.
