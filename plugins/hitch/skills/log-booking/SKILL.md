---
name: log-booking
description: Turn a pasted booking confirmation — flight, train, hotel, restaurant, car, tour — into hitch reservations and accommodations, with the wall-clock times and IANA timezones off the ticket rather than converted.
argument-hint: "[trip name] — then paste the confirmation"
---

# Log a booking

$ARGUMENTS

Read `hitch-conventions` first if it isn't already loaded.

If the user hasn't pasted the confirmation yet, ask for it. Forwarded email,
screenshot text, or a few lines typed from memory all work.

## 1. Find the trip

`list_trips`, then `get_trip`. If the dates in the confirmation don't fall inside any
trip's range, say so before writing anything — it's usually either the wrong trip or
a trip whose dates need extending, and both are the user's call.

## 2. Read the times off the ticket exactly

This is where bookings get logged wrong, so be deliberate:

- `start_local` is `YYYY-MM-DDTHH:MM` **with no offset** — the wall-clock time printed
  on the ticket. Don't convert to UTC. Don't convert to the user's home zone.
- `start_tz` is the IANA zone of the place it happens: `Asia/Tokyo`, `Europe/Paris`,
  `America/New_York`.
- For a flight or a train that crosses zones, `end_tz` is the **arrival** zone and
  differs from `start_tz`. An overnight flight arrives on a later date — get the
  arrival date right rather than assuming same-day.
- Confirmations often print a 12-hour clock. 7:45 PM is 19:45. A ticket that says
  "arrives 06:20 +1" arrives at 06:20 the next day.

If a zone is genuinely unclear — a station name you can't place, an unfamiliar airport
code — resolve it with `search_places` rather than guessing, or ask. A booking in the
wrong zone is worse than one with a gap, because it looks correct.

## 3. Write it

**Reservations** — anything at a fixed time. `add_reservation` with `kind` from:
`flight`, `train`, `bus`, `ferry`, `car_rental`, `restaurant`, `activity`, `other`.

- `title` should be identifiable at a glance: "NH 862 Haneda to Itami", not "Flight".
- `confirmation_code` is what they'll be asked for at the desk — always carry it.
- `url` if the confirmation has a manage-booking link.
- `notes` for what the fields don't hold: seat, baggage allowance, terminal, platform,
  cancellation deadline, "ask for the counter seats".

**Accommodations** — `add_accommodation` with `check_in`/`check_out` as dates, not
times. Put the check-in and check-out *times* in `notes`, since the fields are dates;
a 15:00 check-in and an 11:00 check-out shape the first and last day.

**Link it to a place.** If the venue is somewhere the traveller goes — restaurant,
hotel, tour meeting point — `search_places` for it, `add_place` with real coordinates
and provider ids, and pass that `place_id` to the reservation or accommodation. That's
what puts the booking and the map pin together. Skip this for flights and trains
unless the user wants the station or airport pinned.

**Multi-leg bookings are multiple reservations.** One confirmation code covering
outbound and return, or a connection with a change, becomes one reservation per leg,
sharing the code. Don't collapse them into one row spanning the gap.

## 4. Updating rather than adding

If the trip already has this booking — a changed flight time, a moved dinner — use
`update_reservation` or `update_accommodation` with the version you read, and send
only the fields that changed. Don't add a second copy. If the write is rejected,
`get_trip` again and retry against the new state.

## 5. Report back, and flag the conflicts

Say what you logged, at what local time and zone, and what you linked it to.

Then check it against the plan and say what doesn't fit:

- A flight landing after the hotel's check-in window, or before the day's first stop.
- A restaurant booking on a day scheduled across town.
- Two things booked at the same hour.
- A stay that doesn't cover a night the trip is away, or an arrival day with no bed.
- A cancellation deadline worth putting on the to-do list with a due date.

Flag them; don't fix the itinerary unprompted.
