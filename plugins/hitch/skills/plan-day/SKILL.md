---
name: plan-day
description: Fill or tidy one day of a hitch itinerary — works from the trip's saved places, orders stops so the day doesn't zigzag across town, respects bookings already fixed to that day, checks the forecast, and leaves the rest of the trip alone.
argument-hint: "[trip and day, e.g. Kyoto trip day 3, or the 14th]"
---

# Plan a day

$ARGUMENTS

Read `hitch-conventions` first if it isn't already loaded.

## 1. Read the day in context

`list_trips` if needed, then `get_trip`. Identify the target `day_id` — the user may
name a date, a weekday, or a position ("the second day"). If it's ambiguous, ask;
scheduling onto the wrong day is annoying to undo.

Before placing anything, know four things:

- **What's already on this day**, and in what order. Someone may have planned it.
- **What's fixed.** Reservations on this date are anchors — a 19:30 dinner booking
  sets where the evening has to be, and a morning flight eats the morning.
- **Where they're sleeping** the night before and the night of. The day starts and
  ends at those addresses, and if they differ, this is a moving day and has less
  slack than it looks.
- **What's on the neighbouring days.** Don't put them across town twice in a row, and
  don't schedule the same place on two days.

## 2. Check the weather before choosing

`get_weather` at the trip's destination coordinates for that date. Rain moves gardens,
viewpoints, and long walks off the day and pulls museums and covered markets onto it.
If the date is past the forecast horizon you'll get `no_forecast` — say so and plan
without it rather than inventing a forecast.

## 3. Build the day from the shortlist first

Prefer places already saved on the trip. Somebody put them there deliberately, and
scheduling them is the whole point of the shortlist.

Only search for something new when the day has a real gap — a lunch stop between two
sights, an evening with nothing near where they're staying. Then `search_places`
biased toward the trip's coordinates, `add_place` with the real coordinates and
provider ids, and schedule it.

Order the stops so the day works on the ground:

- Group by area. A day that crosses the city three times is a day mostly spent in
  transit, and the itinerary won't show you that — the coordinates will.
- Respect what opens when. Note anything closed on that weekday in the day's notes;
  Monday museum closures ruin more days than weather does.
- Leave the day breathable. Three or four substantial stops is a full day. Six is a
  route march, and the traveller will silently drop half of it.
- Put the fixed bookings where they actually fall and build around them.

`schedule_item` with `place_id`, positioned with `before_item_id`/`after_item_id`, or
no anchor to append. Use `schedule_item` with `note` for the things that aren't
places: "walk along the river to the next stop", "leave time for lunch nearby",
"last train back is 23:10".

## 4. Reordering someone else's day

If the day already has stops in a poor order, don't quietly rewrite it. Say what you
want to move and why, then move it with `move_item` — carrying the version you read
for that item. If the write is rejected, someone edited it while you worked:
`get_trip` again and redo it against the new state.

`remove_item` only when the user asked for something to come off. The place stays
saved on the trip, so it's reversible — say that when you do it.

## 5. Set the day's notes

`update_day` with the one or two things a traveller needs to know before they set off:
the shape of the day, a closure, a booking time to be somewhere for, the weather call
you made. `notes` replaces what's there — read the existing notes and fold them in
rather than overwriting a collaborator's line.

## 6. Report back

Walk the finished day in order, with the reasoning where it isn't obvious — why this
grouping, why this got moved to another day, what you left on the shortlist and why.
Then say what's still loose: a meal with nowhere booked, an evening with nothing, a
place the user wanted that didn't fit.
