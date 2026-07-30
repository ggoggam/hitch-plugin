---
name: hitch-conventions
description: How to read and write hitch trips without breaking them — read-before-write version numbers, coordinates that must come from search_places, the difference between saving and scheduling a place, and local wall-clock reservation times. Load this whenever working with hitch trips, itineraries, saved places, reservations, accommodations, or trip to-dos.
when_to_use: Any request touching a hitch trip — "add X to my Kyoto trip", "what's on day 3", "log this flight", "move dinner to Thursday", "find somewhere for lunch near the hotel".
user-invocable: false
---

# Working with hitch

hitch is a collaborative trip planner. You act on behalf of one signed-in user, on
trips they may share with other people. Someone else may be editing the same trip
while you work.

## Find the trip before you touch it

`list_trips` when the user names a trip in words. It returns every trip they can see
plus their role on each. Match on name, destination, and dates — and if two trips
could plausibly match, ask instead of guessing, because writing to the wrong trip is
invisible to the user until they open it.

`get_trip` before every change. It returns the full trip — details, day-by-day
itinerary with each stop resolved, saved places, reservations, accommodations, to-dos
— and, critically, **the version numbers the write tools require**.

## Versions: read, write, and retry

Writes carry the version you last read for that specific record. If someone edited
it in between, the write is rejected. That is the system working. Re-read with
`get_trip` and retry against the new state — never work around a rejection, and
never retry with a guessed version number.

| Needs a `version` | No version needed |
| :--- | :--- |
| `update_trip`, `update_day`, `update_place`, `update_reservation`, `update_accommodation`, `update_todo`, `move_item` | `create_trip`, `add_place`, `add_reservation`, `add_accommodation`, `add_todo`, `schedule_item`, `remove_item` |

The version belongs to the record, not the trip: `update_place` wants the version of
that place, `update_day` the version of that day.

Two gotchas worth remembering:

- **`update_trip` requires `name`.** To change only the dates, pass the current name
  back unchanged. Omitting it is not "leave it alone".
- **On every other update tool, only the fields you send change.** Omit a field to
  leave it; send an empty string to clear it. `notes` and `description` replace
  wholesale — to append to existing notes, read them first and send the combined
  text, or you will silently delete what someone else wrote.

## Never invent a location

Coordinates and addresses come from `search_places`, never from memory. A wrong
coordinate sends the traveller to the wrong place.

Search, then copy `name`, `address`, `latitude`, `longitude`, `provider`, and
`provider_place_id` from the result you picked into `add_place`. The provider ids are
what let hitch re-resolve the place later, so carry them even when they feel like
noise. Bias searches toward the trip with the destination's `latitude`/`longitude`.

If a search returns nothing plausible, say so. Do not save a place with no
coordinates and hope it resolves later.

## Saving is not scheduling

`add_place` puts a place on the trip's list of candidates. That is all it does.
`schedule_item` is what puts something on a day. A trip can carry a long list of
saved places with an empty itinerary, and that is a normal, useful state — it is the
shortlist everyone is choosing from.

- `schedule_item` takes either `place_id` or `note`, never both. Position it with
  `before_item_id` or `after_item_id`; omit both anchors to append to the end of the day.
- `move_item` moves a scheduled item to another day, or reorders it within its day.
- `remove_item` takes it off the itinerary. **The place stays saved on the trip** and
  can be scheduled again — so removing a stop is cheap and reversible, and worth
  saying so when you do it.

## Reservation times are wall-clock plus a zone

`start_local` is `YYYY-MM-DDTHH:MM` with no offset — the time a traveller reads off
the ticket — and `start_tz` is the IANA zone it is read in. Do not convert to UTC. Do
not shift a time into the user's home zone.

A flight lands in a different zone than it left, so `end_tz` is the arrival zone and
often differs from `start_tz`. An overnight flight arrives on a later date; get the
date right rather than assuming same-day.

`kind` is one of: `flight`, `train`, `bus`, `ferry`, `car_rental`, `restaurant`,
`activity`, `other`.

Accommodations are date ranges (`check_in`/`check_out`, `YYYY-MM-DD`), not times.

Link a reservation or stay to a saved place with `place_id` when the venue is on the
trip — that is what puts the restaurant booking and the restaurant pin together.

## Weather is a forecast, not a promise

`get_weather` takes coordinates, optionally a date. Beyond roughly 16 days out it
returns `no_forecast`. Report that as "too far out to forecast" — never fill the gap
with a seasonal guess, and never present typical April weather as a forecast.

## Money

A trip's `currency` is the currency its budget is totted up in, and it defaults to
the user's own base currency. That is usually right: it is the currency they think
in, not the one they will spend at the destination. Don't set it to the destination's
currency because that seems more logical.

## You are not the only editor

Prefer adding to a plan over rearranging one you did not create. If the itinerary
needs reordering to make sense, say what you are about to move and why before moving
it — someone chose that order for a reason you cannot see.

When you are done, say plainly what changed: what you added, what you scheduled,
what you moved. On a shared trip, the other travellers will see your edits without
having seen your reasoning.
