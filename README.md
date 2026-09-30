# Skirmish Arena

A plain-Java engine that simulates automated card duels between two AI-controlled champions: no UI, no web, no database. Built during Switchfully's Vibecoding Day, entirely by prompting an AI coding agent.

> Status: under construction. See [PLAN.md](PLAN.md) for progress.

## Requirements

- JDK 25
- Maven 3.9 or later

## Build and test

```bash
mvn test
```

## Run

From IntelliJ: run `com.skirmisharena.Main`, adding program arguments in the run configuration if needed.

From the command line:

```bash
mvn package
java -jar target/skirmish-arena.jar --bot-a aggressive --bot-b defensive --matches 1000 --seed 42
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
