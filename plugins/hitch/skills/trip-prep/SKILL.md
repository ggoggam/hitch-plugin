---
name: trip-prep
description: Work the pre-departure list for a hitch trip — find what still needs booking, renewing, or reserving before the trip works, add it as dated to-dos in deadline order, and tick off what the trip shows is already done.
argument-hint: "[trip name or id]"
---

# Pre-departure prep

$ARGUMENTS

Read `hitch-conventions` first if it isn't already loaded.

## 1. Read the whole trip

`list_trips` if needed, then `get_trip`. You're looking for the gap between what's
planned and what's actually secured — the itinerary tells you the intent, the
reservations tell you what exists.

Work out how long until departure. That's what turns a list into priorities.

## 2. Find what's missing

Go through the plan against what's booked:

- **Getting there and back.** Is there a reservation for the outbound and the return?
  A trip with an arrival flight and no way home is the most common gap there is.
- **Every night covered.** Walk the dates and check the accommodation ranges cover each
  one. Flag any night with no bed, and any gap between a check-out and the next
  check-in.
- **Scheduled places that need booking ahead.** A stop on the itinerary with no
  reservation, at somewhere that requires one — timed-entry museums, popular
  restaurants, tours, anything with a queue. This is the highest-value part of the
  pass.
- **Getting around.** Rail passes that must be bought before arrival, a car with no
  rental booked, airport transfers at hours when transit doesn't run.
- **Documents.** Passport validity (many countries require six months beyond
  departure), visas or travel authorisations, vaccination requirements, an
  international driving permit if there's a car.
- **Practicalities that only bite on arrival.** Travel insurance, a card that works
  abroad, an eSIM or roaming plan, adapters, prescriptions in original packaging.
- **Cancellation deadlines** on what's already booked — a refundable hotel that stops
  being refundable on a date is a to-do, not a footnote.

Don't invent generic advice. Every item should trace to something in this trip: a
place on the itinerary, a night with no stay, a border being crossed, a booking with a
deadline.

## 3. Write the list

`add_todo` for each gap. What makes these useful rather than nagging:

- **The name carries the reason.** "Book Ghibli Museum tickets — they sell out a month
  ahead" beats "Museum tickets". The user reads this weeks later with no context.
- **`due_date` is when it must be *done*,** working back from the deadline — a month
  before for tickets that sell out a month ahead, not the day they sell out.
- **`priority`**: 3 for things that break the trip (flights, a night with no bed,
  documents), 2 for things that lose a planned experience (a sold-out restaurant), 1
  for convenience, 0 for nice-to-have.
- **`description`** for the detail: the booking URL, the phone number, what's needed to
  hand, the cost.

Check the existing to-dos before adding — don't duplicate what's already listed.

## 4. Tick off what's done

If a to-do says to book something and the trip now has that reservation, close it:
`update_todo` with `checked: true` and the version you read. Say which ones you closed
and on what evidence, so the user can disagree.

Adjust a due date that's already passed but is still worth doing, rather than leaving
it silently overdue.

## 5. Report back

The list in deadline order, with days remaining. Lead with anything overdue or due
inside a week, and say plainly which items would actually break the trip if missed
versus which would just be a shame.

If the trip is far enough out that nothing's urgent, say that — it's a useful answer.
