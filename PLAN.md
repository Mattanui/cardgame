# PLAN — Skirmish Arena, step by step

A docs commit, then eleven small steps, each ending in one commit. Every step has a ready-to-paste prompt, the checks to do before committing, and a commit message. Rules come from [DESIGN.md](DESIGN.md), agent instructions from [CLAUDE.md](CLAUDE.md).

## How we work (4 people, 1 agent)

**Roles rotate at every step**

| Role | Job |
|---|---|
| Driver | Pastes (and adapts) the prompt, then pastes the delivered files into IntelliJ at the given paths |
| Reviewer | Reads the full diff in IntelliJ's commit dialog |
| Rule checker | Compares the behaviour and the tests with DESIGN.md |
| Tester | Runs `mvn test` (and the main when it exists) and ticks the checklist |

**Loop for each step**

1. The Driver sends the step's prompt.
2. The Driver pastes every delivered file.
3. The Tester runs `mvn test`.
4. The Reviewer and the Rule checker go through the checklists below.
5. Commit with the proposed message, then push.

**When something is wrong**, never fix it by hand. Roll back the files (IntelliJ: *Rollback*) or re-prompt with the exact error: "Test X fails with: `<output>`. Fix only that." One step = one commit; a small fix-up commit is fine, but never mix two steps.

**New conversation?** Paste CLAUDE.md and DESIGN.md first, then the files the step will touch.

**Checklist for every step**

- [ ] `mvn test` is green, and the number of tests went up (steps that change code)
- [ ] Only the files of this step changed
- [ ] No new dependency in `pom.xml`; no `new Random()`, `Math.random()` or system clock in game code
- [ ] Tests assert exact values taken from DESIGN.md, not just "no exception"
- [ ] Each of us can explain every new class. If not, ask the agent to explain it before committing

---

## Step 0 — Docs (no prompt)

Create an empty folder, run `git init`, add DESIGN.md, PLAN.md, CLAUDE.md and README.md, then commit and push.

Commit: `docs: design, plan, agent instructions and readme`

## Step 1 — Maven skeleton

> Step 1 of PLAN.md: Maven skeleton. Create pom.xml (Java 25 through `maven.compiler.release`, UTF-8, JUnit Jupiter 5 in test scope as the only dependency, a Surefire version that runs JUnit 5, `finalName` `skirmish-arena`, and a jar manifest with Main-Class `com.skirmisharena.Main`), a .gitignore for Maven and IntelliJ, a `Main` class that prints "Skirmish Arena", and one smoke test. No game code yet.

Checks:
- [ ] `mvn package`, then `java -jar target/skirmish-arena.jar` prints "Skirmish Arena"
- [ ] In IntelliJ, open `pom.xml` as a project and set the project SDK to 25

Commit: `chore: maven project skeleton`

## Step 2 — Cards and pool

> Step 2 of PLAN.md: cards. In package `card`, create the sealed interface `Card` (`name()`, `cost()`, `category()`), one record per effect and the enums `CardCategory` and `DefenseKind`, as in DESIGN.md §10. `CardPool.standard()` returns the 27 cards of DESIGN.md §4 in table order, with the exact names, costs, values, durations and copies. Create `engine.GameRules` with the game constants of DESIGN.md (30 HP, starting hand 3, hand limit 7, deck 20, 50 turns, mana cap 10, defensive threshold 15). Records reject invalid values (negative cost, value ≤ 0). Tests: 27 cards, 16 distinct names, 6/6/5/10 per category, and one parameterized test checking every row of the §4 table. No Amplified effects and no engine yet.

Checks:
- [ ] Read `CardPool` line by line against the §4 table: names, costs, values, durations, copies
- [ ] `AmplifyCard` carries no value

Commit: `feat(card): card types and the 27-card pool`

## Step 3 — Champion and match setup

> Step 3 of PLAN.md: champion state and match setup (DESIGN.md §2, and the Draw phase of §3). In `engine`, create `Champion` (name, HP, capacity, mana, hand as `List<Card>`, draw pile as `Deque<Card>`, active defense as an optional `ActiveDefense` record, unused for now). Add: building the draw pile (20 random cards from the champion's own copy of `CardPool.standard()`, then shuffled, both with the `Random` passed in); drawing the starting hand of 3; `draw()`, which takes the top card unless the hand holds 7 cards or the pile is empty and says whether a card was drawn; and `putAtBottom(card)`. Tests with fixed seeds and hand-built piles: the pile has 20 cards all taken from the pool; the same seed gives the same pile; drawing is skipped at 7 cards and on an empty pile; a played card goes to the bottom. No turns and no bots yet.

Checks:
- [ ] The `Random` is passed in; nothing creates its own
- [ ] Each champion builds from its own copy of the pool

Commit: `feat(engine): champion state, deck building and draw rules`

## Step 4 — Turn loop, with attacks only

> Step 4 of PLAN.md: the turn loop (DESIGN.md §3 and §6), with Attack cards only. In `bot`, create `Strategy` (`Optional<Card> nextCard(BotView view)`) and `BotView`, a read-only record: own hand (unmodifiable copy), HP, capacity, mana, whether an own defense is active, own pile size, opponent HP, opponent hand size, whether Amplify is pending, cards played this turn. Give `Champion` its `Strategy`. In `engine`, create `Side`, `EndReason`, `MatchResult` (winner or draw, turns played, end reason, damage dealt per side) and `Match` (two champions, first side, `Random`, `MatchLog`) whose `play()` runs the five phases: Draw; Mana (capacity = min(own turn count, 10), mana refilled); Play (ask the strategy, check that the card is in the hand and affordable, pay, apply, put it at the bottom of the pile, stop at once on a KO); Resolve; End (unspent mana lost). `EffectResolver` applies `AttackCard` only; any other card type throws `UnsupportedOperationException` for now. In `log`, create `MatchLog` (`void line(String)`), `TextMatchLog` (keeps lines in memory) and `NoMatchLog`, and log each draw, play and pass in the format of DESIGN.md §9. More than 50 cards played in one turn throws `IllegalStateException`. Tests, with test-only strategies (always pass; play the first playable card): capacity goes 1, 2 … 10 and stays at 10; two passing bots draw after exactly 50 turns; a KO ends the match at once with `EndReason.KO`; with first side B, B plays turn 1; a card not in hand or not affordable throws.

Checks:
- [ ] 50 turns in total, not per champion
- [ ] Capacity follows each champion's own turn count
- [ ] A bot cannot modify its hand through `BotView`

Commit: `feat(engine): turn loop, mana, end conditions and attacks`

## Step 5 — Heal, resource, draw and steal

> Step 5 of PLAN.md: the effects of `HealCard`, `ResourceCard`, `DrawCard` (Insight) and `StealCard` (Pickpocket), as in DESIGN.md §3 and §5, without Amplify. Heal is capped at 30 HP. Resource restores mana up to the current capacity, never above. Insight draws from the own pile and stops at 7 cards or on an empty pile. Pickpocket takes a random card (with the match `Random`) from the opponent's hand into the own hand: nothing if their hand is empty, stops at 7 cards, and the stolen card now belongs to the thief. Log each effect as in DESIGN.md §9. One test per rule with hand-built hands and piles, for example: heal at 29 HP → 30; capacity 5 with 3 mana left, Focus → 4; Surge at full mana → unchanged; Pickpocket on an empty hand does nothing; a stolen card, once played, goes to the thief's pile.

Checks:
- [ ] Resource never raises the capacity
- [ ] Pickpocket uses the match `Random`

Commit: `feat(engine): heal, resource, draw and steal effects`

## Step 6 — Defense and Amplify

> Step 6 of PLAN.md: Defense and Amplify (DESIGN.md §4, "Amplified" column, and §5). Defense: one active defense per champion, and a Defense card is illegal (the engine throws) while its caster has one. It applies to each incoming attack card: REDUCE by its amount (minimum 0), HALVE rounded down, BLOCK to 0. Its duration counts the opponent's turns and goes down at the end of each opponent turn. Amplify: the next card of the same turn uses its Amplified effect; the bonus is lost at the end of the turn; a second Amplify while one is pending adds nothing. Amplified effects exactly as in the §4 table, including Barrier → blocks everything for 2 turns and Aegis → blocks everything for 2 turns. For an attack, double first, then apply the defense. After this step, `EffectResolver` handles every card type in an exhaustive `switch` with no `default` branch. Tests: one per rule and one per row of the Amplified column, for example: Guard against two Strikes → 1 + 1 damage; Barrier against Jab → 0 and against Heavy Blow → 2; Aegis lasts exactly one opponent turn; amplified Meteor against Shield → 14.

Checks:
- [ ] The duration counts down at the end of the opponent's turn, not the caster's
- [ ] Every row of the Amplified column has its own test

Commit: `feat(engine): defenses and Amplify`

## Step 7 — Bots

> Step 7 of PLAN.md: the three bots of DESIGN.md §7, `AggressiveStrategy`, `DefensiveStrategy` and `BalancedStrategy`, plus `BotType` (`aggressive`, `defensive`, `balanced` → strategy). Implement each bot as an ordered list of rules, in exactly the order of §7, using the shared vocabulary (playable, useful, best, Amplify combo). Defensive delegates to Balanced when HP ≥ 15, checked before every card. Bots only read the `BotView`: if it lacks information a rule needs, tell me before adding it. Tests: at least one per rule of each bot, each with a hand-built `BotView` showing that the rule wins over the rules below it. For example: Aggressive with 10 mana holding Meteor and Amplify plays Amplify, then Meteor; with 7 mana it plays Meteor first; Defensive at 14 HP picks a defense over Meteor, and at 15 HP plays like Balanced.

Checks:
- [ ] Rule order in the code matches §7 exactly
- [ ] No randomness in any bot

Commit: `feat(bot): aggressive, defensive and balanced strategies`

## Step 8 — Complete match log

> Step 8 of PLAN.md: complete the human-readable log so that a whole match reads like DESIGN.md §9: a match header (bots, seed, who starts); a turn header (turn number, active side, both HPs, mana); every draw or skipped draw with the reason; every play and its effect; the pass with unused mana; the end-of-turn defense countdown; and a final line (winner or draw, reason, turns, HPs). Bulk runs use `NoMatchLog` and must not pay for building strings (for example with an `enabled()` check). Test: a short hand-built match whose full log is compared line by line with an expected text.

Checks:
- [ ] Read the expected text in the test: does each line follow the rules?

Commit: `feat(log): full turn-by-turn match log`

## Step 9 — Series, stats and command line

> Step 9 of PLAN.md: the runnable deliverable. `SeriesStats` turns a list of `MatchResult` into the stats of DESIGN.md §9 (win rate per side, draws, average length, average damage per side, KO vs turn limit). `Main` parses `--bot-a`, `--bot-b`, `--matches`, `--seed` and `--log`, with the names and defaults of README.md, and rejects unknown options with a usage message. It creates one `Random` from the seed, runs N matches alternating the first player (odd → A, even → B), logs match 1 with `TextMatchLog` into the `--log` file and the others with `NoMatchLog`, then prints the stats. Tests: `SeriesStats` arithmetic on hand-made results; the same options run twice give identical stats and an identical log (determinism); argument errors.

Checks:
- [ ] Run `java -jar target/skirmish-arena.jar` twice: identical output
- [ ] Open `sample-match.log` and check it reads like DESIGN.md §9

Commit: `feat: run N matches, print stats, write sample log`

## Step 10 — Reference runs and review

Run these four commands (1000 matches, seed 42) and keep the outputs: `aggressive` vs `defensive`, `aggressive` vs `balanced`, `defensive` vs `balanced`, `aggressive` vs `aggressive`. Then:

> Step 10 of PLAN.md. Here are the outputs of our four reference runs: `<paste>`. Put them as a table in the Output section of README.md, and update DESIGN.md §11 with what they show about KOs and match length. Then review the code against DESIGN.md, section by section, and list every gap you find. Do not fix anything in this step.

Checks:
- [ ] Pick three turns in `sample-match.log` and verify them by hand against DESIGN.md (mana, damage after defense, heal cap)
- [ ] Turn each gap the agent listed into a fix prompt (one fix-up commit per gap)

Commit: `docs: reference results and design review`

## Step 11 (optional) — Change a rule mid-build

If the stats confirm DESIGN.md §11 (few KOs, matches close to 50 turns), choose one rule change as a team, for example drawing 2 cards per turn, then:

> Step 11 of PLAN.md. Change this rule: `<rule>`. Update DESIGN.md, the code and the tests. First list every file you will touch and why, and wait for my go before delivering.

Checks:
- [ ] Compare the stats before and after the change
- [ ] Note in DESIGN.md §11 how many files and tests the change touched

Commit: `feat!: <the rule change>`
