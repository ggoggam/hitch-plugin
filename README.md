# hitch-plugin

A Claude Code plugin marketplace for [hitch](https://hitch.ggoggam.dev), the
collaborative trip planner.

## Install

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

If you already installed from GitHub, remove that copy first with
`/plugin marketplace remove hitch-plugin`. A marketplace name can only be
registered once per user, so the local one replaces the published one silently,
and it's easy to spend an afternoon testing the copy you aren't editing.

Validate before pushing:

```bash
claude plugin validate ./plugins/hitch --strict
claude plugin validate . --strict
```

The first checks the plugin and every skill's frontmatter, the second checks the
marketplace catalog. `--strict` turns unrecognized-field warnings into errors,
which is what catches a misspelled manifest key before anyone installs it.

Editing a `SKILL.md` takes effect immediately in the running session. Changes to
`.mcp.json`, `agents/`, or either manifest need `/reload-plugins` or a restart.

## Layout

```
.claude-plugin/marketplace.json     catalog; source points at ./plugins/hitch
plugins/hitch/
├── .claude-plugin/plugin.json      manifest, incl. the server_url userConfig option
├── .mcp.json                       the hitch HTTP MCP server
├── skills/
│   ├── hitch-conventions/          background rules, not in the / menu
│   ├── plan-trip/                  /hitch:plan-trip
│   ├── trip-brief/                 /hitch:trip-brief
│   ├── plan-day/                   /hitch:plan-day
│   ├── log-booking/                /hitch:log-booking
│   └── trip-prep/                  /hitch:trip-prep
└── agents/place-scout.md           hitch:place-scout
```

## License

MIT
