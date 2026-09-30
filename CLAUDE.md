# CLAUDE.md — instructions for the coding agent

Read this file and DESIGN.md before any change. In a new conversation, ask for both files (and the current code of the classes you need) before writing anything.

## Project

Skirmish Arena is a plain-Java engine that simulates automated card duels between two bots. There is no human player, UI, web layer or database. The game rules live in DESIGN.md, and they win over anything you know from other card games (Hearthstone included).

## How this team works

- 100 % vibecoding: the team never types or edits code by hand. They paste what you produce into IntelliJ, run the tests, read the diff in the commit dialog and commit.
- One prompt = one step of PLAN.md. Do that step only: do not start the next one, and do not "improve" code outside its scope.
- If the prompt, DESIGN.md and the existing code disagree, or a rule is missing, stop and ask. Never invent a game rule.
- A rule change always updates DESIGN.md in the same step.

## Hard constraints

- Java 25, root package `com.skirmisharena`.
- **No build tool.** Our organization blocks Maven builds, so there is no `pom.xml` and no `build.gradle`; never add one. The project is built by IntelliJ or by plain `javac`. Layout: `src/main/java`, `src/test/java`, and `lib/` for the one jar we use.
- No frameworks and no runtime dependencies. The only library is JUnit 5, as `lib/junit-platform-console-standalone-1.9.1.jar`, used by tests only. That is JUnit Jupiter 5.9: do not use APIs added in later versions. No Mockito, Lombok or logging framework: write small stubs by hand, print with `System.out` or a `Writer`.
- Deterministic: every random choice goes through the single `java.util.Random` created from the run's seed and passed in. Never use `Math.random()`, `new Random()` without that seed, `ThreadLocalRandom`, `Collections.shuffle(list)` without the `Random`, the system clock, `HashMap`/`HashSet` iteration order in game logic, or parallel streams.
- Bots never change the game state. They read a `BotView` and return a card from their hand, or pass. The engine validates every play and throws `IllegalStateException` on an illegal one.
- English everywhere: code, comments, log lines, docs.

## Code conventions

- Packages and class names as in DESIGN.md §10. Ask before adding a new package or renaming a class listed there.
- Records for immutable data. `Card` is a sealed interface with one record per effect; effects are applied with an exhaustive `switch` over the card type, with no `default` branch.
- Game numbers (30 HP, hand limit 7, 50 turns, mana cap 10, threshold 15…) are constants in `GameRules`, never magic numbers.
- Small classes and methods, no static mutable state, no dead code, no TODO left behind.
- Comments explain *why*, and cite the DESIGN.md section when code implements a rule.

## Tests

- Every rule in DESIGN.md has at least one JUnit test, named after the rule (`halveRoundsDown`, `defenseCannotBePlayedWhileOneIsActive`).
- Tests build their situation by hand (fixed hand, fixed pile, fixed seed) and assert exact values. Never assert on a random outcome you did not fix; never write a test that only checks "no exception".
- All tests must pass before you deliver: `./test.sh` (or `test.cmd` on Windows), which compiles everything with `javac` and runs every test with the JUnit jar.

## Delivering a step

Answer in this order:

1. **Summary**: what changed and why, in 3 to 5 lines, and any rule you had to interpret.
2. **Files**: every new or changed file, with its path and its full content (never a partial snippet). List deleted files.
3. **Verify**: the command to run and the expected result (number of tests, sample output).
4. **Commit message**, Conventional Commits style, e.g. `feat(engine): apply defense per attack card`.

Before delivering, compile and run the whole test suite yourself when your environment allows it, and say so plainly when it does not.
