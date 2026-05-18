# AGENTS.md

## Project at a glance
- Kotlin/JVM console app for the 2026 FIFA World Cup assignment.
- Main entry point: `src/main/kotlin/de/seuhd/worldcup/Main.kt`.
- Core domain model and JSON schema live in `Data.kt`; group standings logic is in `StandingsService.kt`.

## How the app is wired
- `JsonLoader.loadJson()` reads `/world_cup_2026_full_data.json` from the classpath; the app should run from any working directory.
- `Main.kt` builds `teamsById` from `data.groups.flatMap { it.teams }.associateBy { it.id }` and routes the menu to standings, match listing, betting, and scoring.
- Match display and betting use `Match.scoreOrNull()` and `Prediction.fromCode(...)` / `Prediction.outcomeOf(...)`.

## Domain rules to preserve
- Standings use 3/1/0 scoring, then goal difference, then goals scored; unplayed matches (`null` scores) are ignored.
- `StandingsService.calculate(...)` errors on matches that reference unknown team IDs.
- `BettingService` stores one bet per `matchId`; `placeBet(...)` replaces any existing bet.
- `BettingService.evaluate(...)` counts only matches with both a stored bet and a final score.
- `Prediction` codes are assignment-defined: `0 = DRAW`, `1 = HOME_WIN`, `2 = AWAY_WIN`.

## Betting service gotcha
- `BettingService` caches the last `evaluate(...)` result in `cachedResult`; any mutation of bets must invalidate that cache.
- The unimplemented methods in `BettingService.kt` are `evaluateBonus(matches)`, `removeBet(matchId)`, and `changeBet(bet)`.

## Testing conventions
- Unit tests use `kotlin.test` with JUnit 5 on the Gradle JVM test runner.
- `WorldCupTest.kt` holds the pure logic coverage for standings and betting.
- `FileBettingServiceTest.kt` is intentionally state-sensitive: it uses a shared temp file and `@TestMethodOrder(MethodOrderer.Random::class)` to expose leakage.
- `BettingServiceTest.kt` contains empty test bodies that should assert the KDoc-described behavior.

## Workflow
- Use the Gradle wrapper from the project root: `./gradlew build`, `./gradlew test`, `./gradlew run`.
- Use `./gradlew repeatTests` to run the suite 20 times when checking for flakiness.
- JDK 25 is requested via Gradle toolchains; no manual `JAVA_HOME` setup is expected.

## Editing guidance
- Prefer small, local changes; preserve the current package layout under `de.seuhd.worldcup`.
- Keep resource loading classpath-based rather than filesystem-based.
- When adding tests, follow the existing helper style (`match(...)`, `assertEquals(...)`) in `WorldCupTest.kt` and `BettingServiceTest.kt`.
