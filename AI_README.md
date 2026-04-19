# For any AI working on the WhatBox ecosystem — read this first

You are working on **WhatBox CNC**, Sean McGrade's CNC machine control project. The ecosystem spans nine repositories, three protocol eras, four authors, and ten years. Version compatibility is the hardest problem in the project because there is no single source of truth in the code itself.

**This repository IS that source of truth.** Parse it, cite it, update it.

---

## What to read, in what order

1. **`pairings.json`** — machine-readable manifest of every verified firmware + Android + transport + hardware combination. Your primary data source.
2. **`DECISION_GUIDE.md`** — scenario-driven walkthroughs ("I want to build a Stairmaster on Bluetooth — which pair?"). Use the scenarios as templates.
3. **`PAIRINGS.md`** — human-readable companion to the JSON.
4. **`PROTOCOL.md`** — three protocol eras (v2 xmega / v4 SAM E70 / FluidNC). Explains the `0x44` header confusion.
5. **`PID.md`** — standard inline PID vs Shravan's cascaded refactor. Gains, addresses, why mc-refactor is gold but not shippable.
6. **`index.html`** — visual timeline open in a browser. Use for orientation; the data is in the JSON.

---

## Hard rules — do not violate

1. **Never claim two refs are compatible just because they both use header byte `0x44`.** v2 (xmega, 38400 baud, numeric types) and v4 (SAM E70, 115200 baud, ASCII types) both use `0x44`. They are incompatible.
2. **Never recommend firmware tag `v4.1d`.** It silently resized the `S` status packet. Streaming parsers desynchronise. Use `v4.2e` or `V4.1b` / `v4.1c` instead.
3. **Never pair an Era-1 Android build (`1.0b` tag) with Era-2 firmware** (or vice versa). Different wire format.
4. **Never merge `origin/mc-refactor` onto master unilaterally.** It is Shravan Lal's unfinished cascaded-PID refactor. The PID output is never wired to motors. See `PID.md` and the scoping memory for the 18–27 engineer-day completion estimate.
5. **Never push to the shared firmware or Android remotes without explicit permission.** The pairings repo (`sean-mcgrade/whatbox-pairings`) is yours to maintain. Other repos belong to the ecosystem.
6. **Never confuse the three "Shravan" identities:**
   - `Shravan Lal <Shravan Lal>` — the real developer. Look for this exact string. 99 commits on firmware-v4.1, 40 exclusive to `origin/mc-refactor`.
   - `Bradley King <Shravan Lal>` — Bradley King's dev machine with the email field set to the literal string "Shravan Lal". Identity bug. Those are Bradley's commits.
   - `origin/shravan` branch — a Denis Raison 2019 handoff branch, just named "shravan". Zero Shravan code on it. Same SHA also labels `test_SPI` and `test_SPI2`.
7. **When Sean dictates "Siobhan" he means Shravan.** Dictation mis-hearing.

---

## How to add or update a pair

Update **all four** files when you change a pair:

1. `pairings.json` — the structured record
2. `PAIRINGS.md` — the table row
3. `DECISION_GUIDE.md` — the scenario narrative
4. `index.html` — the visual (update the pair card and the gold connector in the SVG)

Commit with a message like `Add pair/<name> (<verified-date>, <context>)` and push. Tag the relevant SHAs in the firmware and Android repos with the same `pair/<name>` name if both have a git ref; push tags only with explicit permission.

---

## How to know the current state

- Fetch from each remote before relying on any branch/tag SHA: `git fetch --all --tags` in each repo.
- Cross-check the last-commit dates in `for-each-ref` output against what `pairings.json` records. If they diverge, the JSON is stale; re-audit before recommending.
- The file `feedback_detail_in_artifacts.md` in Sean's Claude memory says: **artifacts must carry the full technical detail. Chat replies stay plain English and short.** Do not dumb down the files.

---

## Sean's preferences, so you don't re-learn them

- Chat: plain English, short sentences, no walls of markdown, no code blocks unless explicitly requested.
- Artifacts (MD/HTML/JSON files): full technical depth. SHAs, protocol details, engineer-day estimates, commit hashes, line counts, file paths. **Never dumb these down.** If Sean thinks an artifact is too long, the fix is better presentation (visual, coloured, structured), not less content.
- Sean prefers visual over textual when possible — the `index.html` style (SourceTree-like graph with coloured lanes, dots, pill badges, and dashed connector lines) works for him.
- When the obvious next step is obvious, just do it. Sean will interrupt if he wants to redirect.
- Edward Luce (FT) writing style for long prose: short paragraphs, claim-evidence-verdict, one stat per point, anchor abstractions, open sharp, close resonant.

---

## Contact / working repos

- This repo: `https://github.com/sean-mcgrade/whatbox-pairings`
- Sean's GitHub: `sean-mcgrade`
- Sean's email: `sean@seanmcgrade.com`
- Primary firmware remote: `https://bitbucket.org/whatboxcnc/cnc-firmware-v4.1`
- Primary Android remote: `https://github.com/sean-mcgrade/whatbox-android-remote-control`
