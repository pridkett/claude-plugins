# Weather InfluxDB Claude Code Plugin

A Claude Code plugin that connects to the Ambient Weather station MCP server and provides
pre-formatted InfluxQL query skills for each weather metric category.

## Prerequisites

- Claude Code CLI
- Access to `https://wagstrom2022mba.pirate-walleye.ts.net/mcp` (OAuth 2.0 credentials required)

## Setup

### 1. Load the plugin

```sh
claude --plugin-dir . --debug
```

On first use, the MCP server will return a `401 WWW-Authenticate` challenge and Claude Code will
perform the full OAuth 2.0 flow automatically via `.well-known` discovery — no credentials to
configure in advance.

### 3. Verify

```
/help
```

You should see all 6 skills listed under the `weather-influx` namespace.

## Available Skills

| Skill | Trigger | Description |
|---|---|---|
| `search-temperature` | `/weather-influx:search-temperature` | Outdoor/indoor temps, highs/lows |
| `search-precipitation` | `/weather-influx:search-precipitation` | Rainfall totals by period |
| `search-humidity` | `/weather-influx:search-humidity` | Outdoor/indoor humidity |
| `search-wind` | `/weather-influx:search-wind` | Wind speed, gusts, direction |
| `search-pressure` | `/weather-influx:search-pressure` | Barometric pressure trends |
| `search-summary` | `/weather-influx:search-summary` | Full weather report |

## Example Usage

```
/weather-influx:search-temperature last 7 days
/weather-influx:search-precipitation how much did it rain this month?
/weather-influx:search-summary what was the weather like yesterday?
```

## InfluxDB Schema

- **Measurement**: `weather`
- **Tag**: `host=edgewater`
- **Fields**: `tempf`, `tempinf`, `humidity`, `humidityin`, `baromabsin`, `baromrelin`,
  `hourlyrainin`, `dailyrainin`, `eventrainin`, `weeklyrainin`, `monthlyrainin`, `totalrainin`,
  `windspeedmph`, `windgustmph`, `maxdailygust`, `winddir`, `solarradiation`, `uv`, `batt_co2`

## Team Marketplace Installation

If this repo is published to GitHub, teammates can install via:

```
/plugin marketplace add <your-org>/influx_weather_cowork_plugin
/plugin install weather-influx@wagstrom-weather
```

## Pocket ID Configuration Notes

Getting Claude Code's OAuth flow to work end-to-end with [Pocket ID](https://github.com/stonith404/pocket-id)
required three non-obvious fixes. Documented here for anyone replicating this setup.

### 1. Serve `/.well-known/oauth-authorization-server` via Caddy

Claude Code discovers the authorization server via `/.well-known/oauth-authorization-server`,
**not** `/.well-known/openid-configuration`. Pocket ID does not serve the former natively, so
front it with Caddy and serve a static file at that path.

`/etc/caddy/oauth-authorization-server.json`:
```json
{
  "issuer": "https://pocketid-external.pirate-walleye.ts.net",
  "authorization_endpoint": "https://pocketid-external.pirate-walleye.ts.net/authorize",
  "token_endpoint": "https://pocketid-external.pirate-walleye.ts.net/api/oidc/token",
  "jwks_uri": "https://pocketid-external.pirate-walleye.ts.net/.well-known/jwks.json",
  "response_types_supported": ["code"],
  "grant_types_supported": ["authorization_code", "refresh_token"],
  "code_challenge_methods_supported": ["S256"],
  "token_endpoint_auth_methods_supported": ["client_secret_post", "client_secret_basic"]
}
```

Add a route in your Caddyfile to serve this file statically at
`/.well-known/oauth-authorization-server` on the Pocket ID domain.

### 2. Include `scopes_supported` in the MCP server's protected resource metadata

Claude Code did not send scopes in its token requests until the MCP server's
`/.well-known/oauth-protected-resource` response explicitly listed them. The working response
for this server at `https://wagstrom2022mba.pirate-walleye.ts.net/mcp`:

```json
{
  "resource": "https://wagstrom2022mba.pirate-walleye.ts.net/mcp",
  "authorization_servers": ["https://pocketid-external.pirate-walleye.ts.net"],
  "scopes_supported": ["openid", "profile", "email", "groups"],
  "bearer_methods_supported": ["header"]
}
```

### 3. Configure the Pocket ID OIDC client correctly

In the Pocket ID admin UI for this MCP server's client application:

- **Public Client**: must be checked — Claude Code plugins/CoWork require public clients
- **PKCE**: must be enabled — provides security for public clients without a client secret
- **Callback URLs**: add `http://localhost:*/callback` — Claude Code opens a local browser
  redirect to complete the authorization code exchange

The `clientId` in `.mcp.json` comes from this Pocket ID client registration. No `clientSecret`
is needed (or possible) for public clients.

---

## Notes

- All queries use InfluxQL (v1), not Flux.
- The MCP tool name is `query_weather` — accepts a single InfluxQL query string.
- OAuth is handled automatically by Claude Code via `401 WWW-Authenticate` / `.well-known`
  discovery. No credentials are stored in `.mcp.json` — Claude Code manages token storage.
- Run with `--debug` on first launch to confirm OAuth handshake and tool discovery.
