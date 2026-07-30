---
name: plan-trip
description: Start a new hitch trip from a destination and dates — pins the destination so the map and weather work, fills the itinerary with one day per date, and seeds it with candidate places and pre-departure to-dos.
argument-hint: "[destination and dates, e.g. Kyoto, 12-18 April]"
---

# Start a trip

$ARGUMENTS

Read `hitch-conventions` first if it isn't already loaded.

## 1. Settle the basics before creating anything

You need a destination, a start date, and an end date. If the user gave a vague
range ("a week in Kyoto in April"), pick concrete dates and say which you chose —
they can move them later with `update_trip`, and an unpinned trip is harder to plan
against than a slightly wrong one.

Resolve relative dates against today's date, and sanity-check the year: "12 April"
in November means next year.

## 2. Look up the destination's coordinates first

**This matters and cannot be fixed casually later.** `search_places` the city or
region itself — not a venue in it — and copy its `latitude` and `longitude`.

Pass them to `create_trip` as `destination_lat`/`destination_lng`. A trip without
them shows no weather and opens its map nowhere, and **coordinates can only be set
once** — `update_trip` can pin a trip that has none, but is rejected if the trip is
already pinned somewhere else. So get it right on the first call.

## 3. Create it

`create_trip` with `name`, `destination`, `start_date`, `end_date`, and the
coordinates. Giving both dates fills the itinerary with one day per date, ready to
schedule into.

- `name` is what the user will scan a list for. "Kyoto in spring" beats "Kyoto Trip".
- Leave `currency` alone unless the user named one; the default is their own base
  currency, which is the currency they think in.
- Put anything the user said about the shape of the trip — who's coming, the pace they
  want, what they're there for — in `description`. It's the context the next planning
  session starts from.

## 4. Seed it so it's useful immediately

An empty trip isn't worth much. Without being asked, add a small amount of substance
and tell the user what you added:

- **A handful of candidate places.** `search_places` for the obvious anchors of the
  destination, then `add_place` each with its real coordinates and provider ids, and
  a `notes` line saying why it's on the list. Save, don't schedule — the shortlist is
  what the travellers choose from. Six to ten is plenty; more is noise.
- **The to-dos with deadlines.** The things that bite people: tickets that sell out
  weeks ahead, rail passes bought before arrival, a passport with under six months
  validity, visas, seat reservations. `add_todo` with a `due_date` and a `priority`
  (0 none, 1 low, 2 medium, 3 high). Set the due date early enough to act on, not the
  day it becomes impossible.

If the user already told you about booked flights or a hotel, log them now —
`add_reservation` and `add_accommodation`, local wall-clock times plus IANA zones.

Don't schedule anything onto specific days yet unless the user asked for a full
itinerary. Choosing which day gets which place is the interesting part, and it wants
their input — `/hitch:plan-day` handles a day at a time.

## 5. Report back

Give them the trip name and id, the date range, how many days the itinerary now has,
what you saved, and what's on the to-do list. Then say what the obvious next move is:
usually booking the thing with the earliest deadline, or planning the first day.
