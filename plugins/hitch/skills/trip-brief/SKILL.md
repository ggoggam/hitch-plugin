---
name: trip-brief
description: Lay out one hitch trip in full — day by day itinerary, bookings in their own local times, where you're staying each night, saved places nobody has scheduled, open to-dos with deadlines, and the forecast where there is one. Read-only.
argument-hint: "[trip name or id]"
---

# Brief a trip

$ARGUMENTS

Read `hitch-conventions` first if it isn't already loaded.

**This skill does not write.** No adding, scheduling, moving, or tidying — even if
you spot something obviously wrong. Note it at the end and let the user decide.

## Gather

`list_trips` if the user named the trip in words, then `get_trip`. One read has
everything: days, items, places, reservations, accommodations, to-dos.

Then, only where it earns its place: `get_weather` at the trip's destination
coordinates for the next few days of the trip. Beyond about 16 days out it returns
`no_forecast` — say "too far out to forecast" and move on rather than guessing.

## Lay it out

Lead with the shape of the trip in one or two lines: where, when, how many days, who
else is on it if the trip is shared.

Then the itinerary, day by day, in date order. For each day:

- The date and weekday. Travellers plan around weekdays — "Mon 13 Apr", not "Day 2".
- The stops in their scheduled order, with the day's notes if it has any.
- Any reservation that falls on that day, at its local wall-clock time, in the zone
  it happens in. A 09:15 flight from Haneda is 09:15 Asia/Tokyo — never converted.
- Where they're sleeping that night, from the accommodation date ranges.
- The forecast, if there is one for that date.

Say "nothing planned" for an empty day plainly. Empty days are information, not a gap
to paper over — and on a trip that's still taking shape they're most of the trip.

After the itinerary:

- **Saved but not scheduled.** The shortlist nobody has placed on a day yet. This is
  usually the most useful part of the brief.
- **Open to-dos**, soonest deadline first, flagging anything overdue or due within a
  week. Skip the ticked ones unless the user asked for everything.
- **Bookings not tied to a day**, if any — a car rental spanning the trip, a return
  flight past the end date.

## Close with what you noticed

A short list, stated as observations rather than fixes: a day with six stops next to
one with nothing, a restaurant booking on a day the itinerary is across town, a to-do
whose deadline has passed, an outdoor day forecast to rain, a booked flight that
arrives after the first night's check-in time.

Offer the next step — a specific one. Not "let me know what you'd like to do".
