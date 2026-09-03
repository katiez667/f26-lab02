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

## Why `AvailabilityCalculatorTest` stayed green over a real bug

`AvailabilityCalculator.freeSlots` had a bug: it never emitted the free interval
*after* the last booking (or the whole day, when there were no bookings at all) up
to `dayEnd`. Re-running `AvailabilityCalculatorTest` alone against that buggy code
gets **100% line, instruction, and branch coverage** on `AvailabilityCalculator`
(0 missed instructions, 0 missed branches, 0 missed lines per `jacoco.csv`) and all
6 tests pass. High coverage did not save it because coverage only measures whether
code *ran*, not whether the right *input* reached the buggy state or whether the
*assertions* could see a wrong result. Three concrete weaknesses:

1. **Controllability gap — no booking set ever leaves the tail of the day open.**
   Every test's bookings include one that runs flush to `DAY_END` (540–1020, or
   ..–1020), so `cursor` always already equals `dayEnd` when the merge loop
   finishes. The suite never drives the calculator into the one state that needs a
   trailing free slot appended, so no assertion — however good — could have caught
   this from any of those five tests.

2. **Controllability gap — the empty-bookings case is never tried.** No test calls
   `freeSlots(DAY_START, DAY_END, List.of())`. That's the simplest input there is,
   it should return one slot covering the whole day, and it's exactly the smallest
   failing case jqwik found (`dayStart=0, dayEnd=1, bookings=[]`). A boundary this
   basic should be an example-based test on its own, but the suite skips straight
   from "one full-day booking" to "gaps between multiple bookings."

3. **Observability gap — `returnedSlotsNeverOverlapABooking` runs the buggy code
   and still can't see it.** Its booking, 600–660, does *not* reach `DAY_END`, so
   this test does put the calculator in the buggy state and the tail slot
   660–1020 really is dropped. But the assertion only checks that each *returned*
   slot doesn't overlap the booking — it never compares the returned list against
   the full expected set of free slots, so a completely missing slot is invisible
   to it. The bug fires during the test; nothing in the test looks at the right
   place to notice.

The pattern: coverage tools answer "did we execute this code," not "did we feed it
the value that breaks it" (controllability) or "did we check the part of the output
that would show it broke" (observability). This bug is also an *omission* — a
missing statement, not a wrong one — so there was never a line to leave uncovered
in the first place; the loop that does exist gets fully exercised regardless of
whether it appends the missing trailing interval.

## Where things are

- Component: `src/main/java/edu/cmu/cs214/availability/`
- Example-based tests: `src/test/java/edu/cmu/cs214/availability/AvailabilityCalculatorTest.java`
- Property-based tests: `src/test/java/edu/cmu/cs214/availability/AvailabilityProperties.java`
- Setup: `SETUP.md`

See the Lab 2 handout on the course page for the three milestones you show a TA.
