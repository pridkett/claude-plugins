# CLAUDE.md — Project Notes for influx_weather_cowork_plugin

## Lessons Learned

### MCP Server Configuration Schema

The `.mcp.json` schema for remote HTTP servers requires `"type": "http"` and supports
`"oauth": { "clientId": "..." }` as a valid key. The `clientSecret` is omitted for public
clients using PKCE or when the server handles it server-side.

**Working format:**
```json
{
  "mcpServers": {
    "server-name": {
      "type": "http",
      "url": "https://...",
      "oauth": {
        "clientId": "your-client-id"
      }
    }
  }
}
```

Do **not** add fields outside the schema (e.g., bare `"oauth"` without `"type": "http"` caused
a parse failure). Validate early with `claude --plugin-dir . --debug`.

### Claude Code OAuth Handling

Claude Code performs the full OAuth 2.0 flow automatically when the MCP server returns
`401 WWW-Authenticate`. It uses `.well-known` discovery to find the token endpoint. Tokens are
cached by Claude Code — no manual credential export or env vars needed once the `clientId` is
set in `.mcp.json`.

**Discovery endpoint**: Claude Code queries `/.well-known/oauth-authorization-server`, NOT
`/.well-known/openid-configuration`. If your IdP only serves the latter (e.g. Pocket ID),
serve the former as a static file via a reverse proxy (Caddy, nginx, etc.).

**Scopes**: Claude Code will not send scopes unless the MCP server's
`/.well-known/oauth-protected-resource` response includes `"scopes_supported"`. Add that field
or tokens will arrive without the necessary claims.

**Public clients required**: Claude Code plugins use public OAuth clients (no `clientSecret`).
The IdP client registration must have:
- Public Client enabled
- PKCE (S256) enabled
- `http://localhost:*/callback` in allowed redirect URIs (Claude opens a local browser redirect)

### Plugin Structure

The minimal required layout:
```
plugin-dir/
├── .claude-plugin/
│   └── plugin.json      ← name, version, description, author
├── .mcp.json            ← mcpServers config
└── skills/
    └── <skill-name>/
        └── SKILL.md     ← YAML frontmatter (name, description) + instruction body
```

Skills are invoked as `/<plugin-name>:<skill-name>` in Claude Code.

### InfluxQL Notes (v1, this schema)

- Measurement: `weather`, tag: `host=edgewater`
- `hourlyrainin` is a **rate** field — a point-in-time estimate of rain extrapolated to
  inches/hour. Use `max(hourlyrainin)` to find peak rain rate. It is NOT a running accumulator.
- For true accumulation use `dailyrainin`, `weeklyrainin`, `monthlyrainin`, `totalrainin`
  (pre-aggregated by the station). Read with `last()` over a short recent window.
- `GROUP BY time(1h)` is standard for most trend queries; use `time(30m)` for pressure.
- Absolute date ranges: `WHERE time >= '2024-01-01T00:00:00Z' AND time <= '2024-01-31T23:59:59Z'`

---

## Git Commit Message Convention

All commits in this project use **Conventional Commits** format:

```
<type>(<scope>): <short summary>

[optional body]

[optional footer]
```

**Types:**
- `feat` — new skill, new MCP connection, new capability
- `fix` — bug fix (broken query, schema error, config parse failure)
- `docs` — README, CLAUDE.md, SKILL.md content only
- `chore` — config files, marketplace.json, dependency updates
- `refactor` — restructuring without behavior change

**Scope** (optional, use plugin component name): `mcp`, `skill`, `config`, `marketplace`

**Examples:**
```
feat(skill): add search-summary skill with multi-query weather report
fix(mcp): remove invalid oauth block causing schema parse error
docs: document hourlyrainin as rate field not accumulator
chore(config): set type=http and clientId in .mcp.json
```

Rules:
- Summary line ≤ 72 characters, lowercase, no trailing period
- Use imperative mood ("add", "fix", "remove" — not "added", "fixes")
- Reference the specific skill or file in scope when change is narrow
