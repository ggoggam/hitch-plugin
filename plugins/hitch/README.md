# hitch

Plan trips collaboratively with [hitch](https://hitch.ggoggam.dev) from Claude Code or
OpenAI Codex.

This plugin connects the hitch MCP server and adds the workflows around it: starting a
trip, filling in days, logging bookings, briefing where a trip stands, and working the
pre-departure list.

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

If you self-host hitch, Claude Code prompts for your endpoint here. Accept the
default (`https://hitch.ggoggam.dev/mcp`) otherwise, and change it later with
`/plugin` → `hitch` → configure if you need to.

**3. Authenticate.** The hitch server is an OAuth-protected resource and requests the
`profile`, `trips:read`, and `trips:write` scopes. Until you sign in, every hitch tool
returns 401:

```
/mcp
```

Pick `hitch`, choose Authenticate, and approve the scopes in the browser tab that
opens.

**4. Check it worked.** `/mcp` should list `hitch` as connected, and typing `/hitch`
should offer five commands. `/hitch:trip-brief` is the safe one to try first — it
only reads.

## Install in Codex

**1. Add the marketplace.**

```bash
codex plugin marketplace add ggoggam/hitch-plugin
```

**2. Install `hitch`** from the plugin browser:

```
/plugins
```

hitch is an OAuth-protected resource, and the marketplace entry is
`authentication: ON_INSTALL`, so Codex takes you through the consent screen while
installing. To sign in again later — or if you skipped it — run `codex mcp login hitch`.

**3. Start a new session.** Codex loads a plugin's skills and MCP server at session
start, not in the session you installed from.

**4. Check it worked.** `codex mcp list` shows `hitch`, and the skills are offered by
name. `trip-brief` is the safe one to try first — it only reads.

Self-hosting: the Codex manifest hardcodes `https://hitch.ggoggam.dev/mcp`, because
Codex has no equivalent of Claude Code's `userConfig` prompt. Override it in
`~/.codex/config.toml`:

```toml
[mcp_servers.hitch]
url = "https://your-host/mcp"
```

### Troubleshooting

| Symptom | Cause |
| :--- | :--- |
| Tools return 401 | Not authenticated yet — `/mcp` in Claude Code, `codex mcp login hitch` in Codex |
| `/hitch` commands missing after install (Claude Code) | Run `/reload-plugins`, or restart Claude Code |
| Skills or tools missing after install (Codex) | Start a new session — Codex reads plugin components at session start |
| Two of every hitch tool | You also have hitch as a user or project MCP server — see below |
| Connection fails on a self-hosted instance | Endpoint must be the full MCP path, e.g. `https://your-host/mcp`, not the site root |
| Self-hosted endpoint ignored in Codex | Codex reads no `userConfig`; override `[mcp_servers.hitch]` in `~/.codex/config.toml` |

## Skills

| Command | What it does |
| :--- | :--- |
| `/hitch:plan-trip` | Starts a trip: pins the destination coordinates so the map and weather work, fills the itinerary with one day per date, seeds candidate places and dated to-dos |
| `/hitch:trip-brief` | Read-only. Lays out a trip day by day — stops, bookings in their own local times, where you're sleeping, the unscheduled shortlist, open to-dos, forecast where there is one |
| `/hitch:plan-day` | Fills or tidies one day — works from the saved shortlist, groups stops by area, respects fixed bookings, checks the forecast |
| `/hitch:log-booking` | Turns a pasted confirmation into reservations and accommodations, with wall-clock times and IANA zones off the ticket |
| `/hitch:trip-prep` | Finds what still needs booking or renewing, adds it as dated to-dos in deadline order, ticks off what's done |

The commands are named `/hitch:plan-trip` and so on in Claude Code; in Codex the same
skills are offered by bare name (`plan-trip`, `trip-brief`, …).

`hitch-conventions` is a sixth skill that isn't in the `/` menu. It loads automatically
whenever the model touches a trip. It's the operating manual: read-before-write version
numbers, coordinates that must come from `search_places`, the difference between saving
and scheduling a place, and why reservation times are never converted to UTC.

## Place scout

`place-scout` researches candidates — "find us six good dinner options near the hotel" —
and saves them to the trip's shortlist with real coordinates. It saves places and never
schedules them.

It ships twice, because the two harnesses model this differently:

- **Claude Code** runs it as the `hitch:place-scout` subagent (`agents/place-scout.md`),
  in its own context, so a dozen searches don't fill up yours.
- **Codex** runs it as the `place-scout` skill, promoted to an agent by
  `skills/place-scout/agents/openai.yaml`.

The skill is marked `user-invocable: false` so it stays out of Claude Code's `/` menu and
the subagent remains the single entry point there. The two copies carry the same
instructions — edit both.

## Why the skills exist

The hitch MCP server ships its own instructions, so a model can use the tools without
this plugin. What the plugin adds is the discipline around them, in the places where
getting it wrong is quiet rather than loud:

- **Destination coordinates can only be set once.** `/hitch:plan-trip` looks them up
  *before* calling `create_trip`, because a trip pinned to the wrong city can't be
  repinned.
- **Writes carry a version.** Trips are shared and edited concurrently. A rejected
  write means re-read and retry — not retry harder. `update_trip` also requires `name`
  even when you're only changing dates, which silently renames trips if you miss it.
- **Saving isn't scheduling.** `add_place` builds a shortlist; `schedule_item` puts
  something on a day. Conflating them produces trips that look planned and aren't.
- **Times are wall-clock plus a zone.** 09:15 `Asia/Tokyo`, exactly as printed on the
  ticket. Converting to UTC or to the traveller's home zone produces bookings that
  look right and are hours off.
- **Notes replace, they don't append.** On a shared trip, sending new `notes` without
  reading the old ones deletes what a collaborator wrote.

## Permission rules (Claude Code)

Tools from a plugin-bundled server carry both the plugin name and the server key. To
pre-approve the read-only tools in `settings.json`:

```json
{
  "permissions": {
    "allow": [
      "mcp__plugin_hitch_hitch__list_trips",
      "mcp__plugin_hitch_hitch__get_trip",
      "mcp__plugin_hitch_hitch__search_places",
      "mcp__plugin_hitch_hitch__get_weather"
    ]
  }
}
```

Leaving the write tools to prompt is the right default on a shared trip.

## Already running hitch as a user or project MCP server?

You'll get two connections to the same server and two copies of every tool. In Claude
Code, remove your own entry — `claude mcp remove hitch` — or leave the plugin's server
toggled off in `/mcp` and use the skills against your existing connection.

In Codex the plugin's server and a `[mcp_servers.hitch]` block in `~/.codex/config.toml`
share the same name, so your entry overrides the plugin's rather than duplicating it —
which is also how you point the plugin at a self-hosted instance.

Either way the skills refer to tools by bare name (`get_trip`, `add_place`), so they work
against whichever connection is live.

## License

MIT
