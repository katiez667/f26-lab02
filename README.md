# Lab 2 Starter: Availability Calculator

A small reservation component. Given a room's bookings and the day's business hours,
`AvailabilityCalculator.freeSlots` computes when the room is free. It is the code you
work in for Lab 2.

It ships with a generated test suite that passes, and a property-based test harness
(jqwik) with one example property. Everything is green. Your job in Lab 2 is to decide
whether green actually means correct.

**Read `ARCHITECTURE.md` before the code.**

## Build and test

```
mvn test
```

`mvn test` runs both files, the ordinary example-based tests (`AvailabilityCalculatorTest`)
and the property-based tests (`AvailabilityProperties`). A code-coverage report is written
to `target/site/jacoco/index.html`.

## Continuous integration

This repository has CI configured in `.github/workflows/ci.yml`. GitHub disables workflows on a
fresh fork, so enable them once on your fork (the handout shows where). After that, every
push runs `mvn test`. You will watch the gate go red when your new property finds the bug, then
green once you fix it.

## Why `AvailabilityCalculatorTest` missed the bug

That suite gets 100% line/branch coverage on the buggy `AvailabilityCalculator`
and all 6 tests pass. Coverage looked perfect; the bug was still there.

1. Controllability gap: every test's bookings run right up to `DAY_END`, so the
   loop never has to emit a trailing free slot. No input ever reached the broken
   state, so no assertion could have caught it.
2. Controllability gap: nobody calls `freeSlots` with an empty bookings list,
   the simplest input there is - and the exact minimal case jqwik found.
3. Observability gap: `returnedSlotsNeverOverlapABooking` uses a booking
   (600-660) that does stop short of `DAY_END`, so it does hit the bug - the
   660-1020 tail gets dropped. But it only checks that returned slots don't
   overlap a booking, never that the right slots came back, so it can't tell
   anything's missing.

Coverage tells you code ran, not whether it ran on an input that could break it,
and not whether anything actually looked at the part of the output that broke.
This bug is also just a missing line rather than a wrong one, so there was
never anything for coverage to flag as unexercised.

## Where things are

- Component: `src/main/java/edu/cmu/cs214/availability/`
- Example-based tests: `src/test/java/edu/cmu/cs214/availability/AvailabilityCalculatorTest.java`
- Property-based tests: `src/test/java/edu/cmu/cs214/availability/AvailabilityProperties.java`
- Setup: `SETUP.md`

See the Lab 2 handout on the course page for the three milestones you show a TA.
