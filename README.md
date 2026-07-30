# hitch-plugin

A Claude Code plugin marketplace for [hitch](https://hitch.ggoggam.dev), the
collaborative trip planner.

## Install

```
/plugin marketplace add ggoggam/hitch-plugin
/plugin install hitch@hitch-plugin
```

Then run `/mcp`, pick `hitch`, and authenticate — the server is OAuth-protected.

## Plugins

| Plugin | Description |
| :--- | :--- |
| [`hitch`](plugins/hitch) | Connects the hitch MCP server and adds skills for starting a trip, planning days, logging bookings, briefing a trip, and pre-departure prep |

## Develop locally

Add this directory as a marketplace without publishing anything:

```
/plugin marketplace add /Users/ggoggam/dev/hitch-plugin
/plugin install hitch@hitch-plugin
```

Validate before pushing:

```
claude plugin validate ./plugins/hitch --strict
```

Editing a `SKILL.md` takes effect immediately in the running session. Changes to
`.mcp.json`, `agents/`, or the manifests need `/reload-plugins` or a restart.

## Layout

```
.claude-plugin/marketplace.json     catalog; metadata.pluginRoot points at ./plugins
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
