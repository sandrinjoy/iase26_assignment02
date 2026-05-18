# Flaky Test Report

**Name:** Pudukkattukaran Joy, Sandrine

## Flaky Test 1

**Test name:** `de.seuhd.worldcup.WorldCupTest#standings are stable when multiple teams tie on all criteria()`

**Root cause:**
When teams tied on all sorting criteria (points, goal difference, goals), the sort order was random because `IdentityHashMap` iteration is non-deterministic.

**Fix:**
Added team ID as a final tiebreaker in the sort order (`thenBy { it.team.id }`) so ties always sort the same way.

## Flaky Test 2

**Test name:** `de.seuhd.worldcup.FileBettingServiceTest#test file betting with threads()`

**Root cause:**
Two threads were reading, modifying, and writing the bet file at the same time. Their operations interleaved, causing some bets to be lost. Expected 100 bets but only got 30 because writes were overwriting each other.

**Fix:**
Wrapped the read-modify-write operation in `synchronized(lock)` so only one thread can modify the file at a time.

## Flaky Test 3

**Test name:** `de.seuhd.worldcup.WorldCupTest#evaluate returns zero when no bets are placed()`

**Root cause:**
`BettingService` caches evaluation results. When `clear()` was called to reset bets, it cleared the bets but not the cache. So `evaluate()` returned stale cached results from the previous test instead of calculating from the empty bet store.

**Fix:**
Added `cachedResult = null` to the `clear()` method so the cache is invalidated when bets are cleared.

## Flaky Test 4

**Test name:** `de.seuhd.worldcup.FileBettingServiceTest#fresh service has no bets()`

**Root cause:**
Tests ran in random order and shared the same temp file. If the "save bets" test ran before "fresh service", leftover bets would still be in the file, causing the fresh service test to fail.

**Fix:**
Added `@AfterEach cleanup()` to delete the shared file after each test so no bets persist between tests.

## Flaky Test 5

**Test name:** `de.seuhd.worldcup.WorldCupTest#load json from network()`

**Root cause:**
The test had a 300ms timeout, but the code tries multiple URLs with retry logic where each attempt can take up to 8 seconds (3s connect + 5s read). When URLs were slow or down, the test exceeded the timeout waiting for retries.

**Fix:**
Increased timeout from 300ms to 20 seconds so the retry logic has enough time to work.
