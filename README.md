# hitch-plugin

A plugin marketplace for [hitch](https://hitch.ggoggam.dev), the collaborative trip
planner. The same plugin installs in **Claude Code** and in **OpenAI Codex** — the two
harnesses share the skills and the MCP server, and each reads its own manifest.

## Install in Claude Code

Run these in Claude Code:

**1. Add the marketplace.** This is the GitHub repo, in `owner/repo` shorthand:

```
/plugin marketplace add ggoggam/hitch-plugin
```

Append `@ref` to pin to a branch or tag, e.g. `ggoggam/hitch-plugin@main`.

**2. Install the plugin.**

```
/plugin install hitch@hitch-plugin
```

If you self-host hitch, Claude Code prompts for the endpoint here. Accept the
default (`https://hitch.ggoggam.dev/mcp`) otherwise. You can change it later with
`/plugin` → `hitch` → configure.

**3. Authenticate.** The hitch server is an OAuth-protected resource, and every
tool returns 401 until you sign in:

```
/mcp
```

Pick `hitch`, choose Authenticate, and approve `profile`, `trips:read`, and
`trips:write` in the browser tab that opens.

**4. Check it worked.** `/mcp` should now show `hitch` as connected, and typing
`/hitch` should list five commands. Try:

```
/hitch:trip-brief
```

### Already running hitch as a user or project MCP server?

You'll get two connections to the same server and two copies of every tool. Either
remove your own entry with `claude mcp remove hitch`, or leave the plugin's server
toggled off in `/mcp` and use the skills against your existing connection — they
refer to tools by bare name, so they work either way.

## Install in Codex

**1. Add the marketplace.** From a terminal:

```bash
codex plugin marketplace add ggoggam/hitch-plugin
```

`--ref <branch-or-tag>` pins it. A local path works too — see below.

**2. Install the plugin.** In Codex, open the plugin browser and install `hitch`:

```
/plugins
```

The marketplace entry is `authentication: ON_INSTALL`, so Codex walks you through the
hitch OAuth consent screen as part of installing. If you skip it, or need to sign in
again later:

```bash
codex mcp login hitch
```

**3. Start a new session.** Codex picks up a plugin's skills and MCP server on session
start, not in the session you installed from.

**4. Check it worked.** `codex mcp list` should show `hitch`, and the skills should be
offered by name — `trip-brief` is the safe one to try first, since it only reads.

### Self-hosting

The Codex manifest hardcodes `https://hitch.ggoggam.dev/mcp`, because Codex has no
equivalent of Claude Code's `userConfig` prompt. Point it at your own instance by
overriding the server in `~/.codex/config.toml`:

```toml
[mcp_servers.hitch]
url = "https://your-host/mcp"
```

It has to be the full MCP path, not the site root.

## Plugins

| Plugin | Description |
| :--- | :--- |
| [`hitch`](plugins/hitch) | Connects the hitch MCP server and adds skills for starting a trip, planning days, logging bookings, briefing a trip, and pre-departure prep |

## Develop locally

Clone the repo, then add your working copy as a marketplace by path — nothing has
to be published for this to work:

```bash
git clone https://github.com/ggoggam/hitch-plugin
```

Then, with Claude Code running in the directory you cloned into:

```
/plugin marketplace add ./hitch-plugin
/plugin install hitch@hitch-plugin
```

The source must be a `./`-prefixed relative path or an absolute one — a bare name
is read as GitHub `owner/repo` shorthand, not a local directory.

In Codex the equivalent is:

```bash
codex plugin marketplace add ./hitch-plugin
```

If you already installed from GitHub, remove that copy first — in Claude Code with
`/plugin marketplace remove hitch-plugin`. A marketplace name can only be
registered once per user, so the local one replaces the published one silently,
and it's easy to spend an afternoon testing the copy you aren't editing.

Validate before pushing:

```bash
claude plugin validate ./plugins/hitch --strict
claude plugin validate . --strict
python3 scripts/validate_plugin.py ./plugins/hitch   # from Codex's plugin-creator skill
```

The first checks the Claude manifest and every skill's frontmatter, the second the
Claude marketplace catalog. `--strict` turns unrecognized-field warnings into errors,
which is what catches a misspelled manifest key before anyone installs it. The third
is Codex's own validator, which is stricter still: it rejects any manifest key outside
its allowed set, so a Claude-only field copied into `.codex-plugin/plugin.json` fails
there rather than at install time.

Editing a `SKILL.md` takes effect immediately in a running Claude Code session. Changes
to `.mcp.json`, `agents/`, or any manifest need `/reload-plugins` or a restart. Codex
reads plugin components at session start, so it needs a new session either way.

## Layout

Two manifests over one set of skills. Neither harness reads the other's files.

```
.claude-plugin/marketplace.json     Claude Code catalog
.agents/plugins/marketplace.json    Codex catalog
plugins/hitch/
├── .claude-plugin/plugin.json      Claude manifest, incl. the server_url userConfig option
├── .codex-plugin/plugin.json       Codex manifest; declares skills + MCP server inline
├── .mcp.json                       the hitch HTTP MCP server (Claude Code only)
├── skills/
│   ├── hitch-conventions/          background rules, not in the / menu
│   ├── plan-trip/                  /hitch:plan-trip
│   ├── trip-brief/                 /hitch:trip-brief
│   ├── plan-day/                   /hitch:plan-day
│   ├── log-booking/                /hitch:log-booking
│   ├── trip-prep/                  /hitch:trip-prep
│   └── place-scout/                Codex agent (agents/openai.yaml); hidden from Claude's / menu
└── agents/place-scout.md           hitch:place-scout (Claude Code only)
```

Two details are worth knowing before editing any of it:

- **Codex auto-discovers nothing that isn't declared.** `.codex-plugin/plugin.json` has
  to name `skills` and `mcpServers` explicitly; Claude Code finds `skills/`, `agents/`
  and `.mcp.json` by convention. A skill added to `skills/` is picked up by both, but a
  new component *type* only exists where its manifest says so.
- **The MCP server is declared twice, on purpose.** Claude Code's `.mcp.json` uses
  `${user_config.server_url}`, which Codex has no equivalent for and would pass through
  as a literal string. Codex's copy lives inline in its manifest with the URL hardcoded,
  under the same `hitch` server key. Change the endpoint in both, or self-hosters break
  on one harness.

## License

MIT
