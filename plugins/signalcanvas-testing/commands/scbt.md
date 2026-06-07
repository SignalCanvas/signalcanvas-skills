Run the SignalCanvas browser trace integration tests.

Prerequisites: dev server must be running at localhost:5173
  Start it with: ./scripts/dev-web.sh

## Questions

Ask the user ALL FOUR of these questions together in a single AskUserQuestion tool call:
1. "Run headed or headless?" — options: Headless (fast, no browser window), Headed (browser visible during test)
2. "After tests complete, keep the window open?" — options: No (closes automatically), Yes (pauses so you can inspect the final state)
3. "Which protocols to route through?" — options:
   - "Random" — let the test pick from the stock library
   - "Dante → MADI" — Dante source, MADI bus
   - "Dante → AES67" — Dante source, AES67 bus
   - "Other" — type the protocols you want, space-separated in notes (e.g. "AES50 OptoCore")
     (supported: Dante, MADI, AES50, AES67, OptoCore, SoundGrid, TWINLANe, AVB, NDI; last = bus)
4. "Force any specific devices into the chain?" — options:
   - "No" — builder picks freely from the stock library
   - "Other" — type one or more device model names in the notes field, comma-separated
     (e.g. "Quantum 338, Stage Rack 48"); builder determines role (source/bus/converter/sink)

Note: the AskUserQuestion tool always adds an "Other" free-text option — users type into the notes field.

## Deriving opts

**protocols** (question 3):
- "Random": no protocols → random mode
- "Dante → MADI": `['Dante', 'MADI']`
- "Dante → AES67": `['Dante', 'AES67']`
- Other typed input: split on spaces/commas → e.g. `['AES50', 'OptoCore']`

**devices** (question 4):
- "No": omit `devices` key entirely
- Other typed input: split on commas → e.g. `['Quantum 338', 'Stage Rack 48']`

Combine into one opts object: `{ protocols: [...], devices: [...] }` omitting empty arrays.

## Before running

Update e2e/canvasTrace.spec.ts:
- Replace `await buildScenario(page)` with `await buildScenario(page, <opts>)` where <opts> is derived above
  e.g. `await buildScenario(page, { protocols: ['MADI', 'OptoCore'] })`
  or   `await buildScenario(page)` for random mode
- For "keep window open" (Yes): temporarily add `await page.pause()` as the last line before the closing `})`

## Run command

- Headless + No:  npx playwright test e2e/ --reporter=list
- Headless + Yes: PWDEBUG=1 npx playwright test e2e/ --reporter=list
- Headed  + No:   PLAYWRIGHT_SLOW_MO=500 npx playwright test e2e/ --headed --reporter=list
- Headed  + Yes:  PWDEBUG=1 PLAYWRIGHT_SLOW_MO=800 npx playwright test e2e/ --headed --reporter=list

## After the run

Restore canvasTrace.spec.ts:
- Revert to `await buildScenario(page)` (no opts)
- Remove `await page.pause()` if it was added

## Chain structure

The test uses REAL devices from the stock library (no synthetic TC_* templates).
Chain: analogue source → [optional converters] → bus node (internal mix bus) → sink

## Report

- The generated chain printed to stdout (device models, port names, channel label)
- Which of the 5 phases passed: smoke → verified path → reload → smoke (post-reload) → verified path (post-reload)
- On failure: which device model was missing from the trace, expected vs actual trace content
- Phase 4–5 failures with Phase 1–2 passing = reload regression (save/load round-trip broken)
