# Skirmish Arena

A plain-Java engine that simulates automated card duels between two AI-controlled champions: no UI, no web, no database. Built during Switchfully's Vibecoding Day, entirely by prompting an AI coding agent.

> Status: under construction. See [PLAN.md](PLAN.md) for progress.

## Requirements

- JDK 25
- No build tool. Our organization blocks Maven builds, so the project is built by IntelliJ or plain `javac`. The only library, JUnit 5, is the jar in `lib/`.

## Open in IntelliJ

Open the repository folder. The module files in the repository already mark `src/main/java` as sources, `src/test/java` as tests, and the JUnit jar as a test library. If IntelliJ asks for an SDK, pick your JDK 25.

## Test

- **IntelliJ**: right-click `src/test/java`, then *Run 'All Tests'*.
- **Command line**: `./test.sh`, or `test.cmd` on Windows. Both compile everything into `build/` and run every test.

## Run

- **IntelliJ**: run `com.skirmisharena.Main`, with program arguments set in the run configuration if needed.
- **Command line**: no compile step needed. Java 22+ compiles the source files on the fly:

```bash
java src/main/java/com/skirmisharena/Main.java --bot-a aggressive --bot-b defensive --matches 1000 --seed 42
```

| Option | Default | Meaning |
|---|---|---|
| `--bot-a` | `aggressive` | Bot for side A: `aggressive`, `defensive` or `balanced` |
| `--bot-b` | `defensive` | Bot for side B |
| `--matches` | `1000` | Number of matches to simulate (N) |
| `--seed` | `42` | Seed of the run: same seed, same results |
| `--log` | `sample-match.log` | File that receives the full log of match 1 |

## Output

- **Console**: win rate per side, average match length, average damage per match, and how matches ended (KO vs turn limit).
- **`sample-match.log`**: the turn-by-turn log of match 1.

Reference results will be added here at the end of the build (PLAN.md, step 10).

## Documents

- [DESIGN.md](DESIGN.md): game rules and the decisions we made where the brief was silent.
- [PLAN.md](PLAN.md): the steps and prompts we followed.
- [CLAUDE.md](CLAUDE.md): the instructions given to the coding agent.
