---
name: matrix-game-builder
description: Turns a matrix wargame (a PDF or rules text) into a playable web artifact with AI players, an AI umpire, an advisor and a debrief. Use when someone shares a matrix game and wants to play it solo or online.
---

# Matrix game builder

This skill takes a matrix game (usually a PDF with a scenario, actor briefs, metric tracks and an event table) and turns it into a single playable web page. The person picks a seat. AI players take the other seats, and a separate AI umpire rates every argument before the dice decide.

The person using this skill is often a facilitator, trainer or analyst, not a developer. Do the technical work yourself, keep questions few and plain, and explain the result in terms of play, not code.

The reference build this skill comes from is Narrow Waters, a U.S.-Iran matrix game by Shadowbox Training. Everything below generalizes what worked there.

## What you will produce

One self-contained HTML page, published as an artifact, that:

- lets the player choose a seat and include or leave out optional actors
- runs the turn loop: random event, sealed arguments, umpire ruling, dice, results
- keeps a situation board (metric tracks), an established-facts ledger and a turn history
- offers an advisor for players new to the scenario or to matrix games
- writes an after-action debrief on request
- saves the game in the browser so it can be resumed

The page calls Claude through the artifact `sample` capability, so the person needs no API key or server.

## Step 1: Read the game and write a game spec

Read the whole source. Matrix game PDFs often split rules, actor briefs and tracks across sections, and some pages (like maps) may be empty placeholders.

Write a game spec as a JavaScript object before writing any page code. It should hold:

- **Meta:** title, designer or publisher, source link, one-paragraph scenario summary for prompts.
- **Actors:** id, name, required or optional, core problem, objectives, and their own metrics. Give each a distinct badge color.
- **Shared metrics:** key, name, starting value, range (often -3 to +3), crisis end if any, and a one-sentence description.
- **Actor metrics:** same fields as shared metrics, plus owner.
- **Extra tracks:** for example a supply or munitions track with named levels.
- **Turn rules:** what a turn represents in time, how many arguments per actor, argument parts (usually action, measurable result, reasons).
- **Adjudication:** success thresholds by strength (commonly strong 5+, average 7+, weak 9+ on 2D6), modifier range, rules for ambitious arguments (for example three metrics raises the bar).
- **Random events:** the table, how it is rolled, whether events repeat, and the rule for used numbers.
- **Special rules:** counter-arguments, elections, secret actions, who decides what when an actor is not played.
- **End conditions:** many matrix games have none. Say so rather than inventing one.

Write a description for every metric, event, track and strength level. The page shows these when a player clicks a label. Where the source gives no description, write one from the name and the actors' objectives and keep a list of which ones you wrote.

If the game has no numeric metrics at all, build a narrative version: no tracks, the facts ledger becomes the board, and arguments claim outcomes in words.

## Step 2: Check gaps with the person

Send one short message before building. Cover only what applies:

- **Gaps in the source:** undefined crisis thresholds, missing maps, unclear counter-argument rules. Say what default you will use.
- **Inferred descriptions:** mention that you wrote some metric descriptions and they should check them.
- **Permission:** the game belongs to its designer. A private prototype is usually fine. Anything shared widely should have the designer's blessing. The page always credits the original and says it is not affiliated.
- **Real-world framing:** if actors are real states or armed groups, the AI players voice them analytically and the umpire stays neutral. Say so on the page.

If the person is not there to answer, use the defaults, build, and list the decisions at the top of your reply.

## Step 3: Build the page

Read the artifact server's `pages` and `design` guides and the `sample` capability description before writing code.

### Keep the engine separate from the AI

Plain code handles everything rule-bound: dice, the event table, thresholds, modifiers, clamping metrics to their range, the facts ledger, saving and loading. AI handles judgment and prose only. This keeps the rules exact and makes the page easy to port to a full app later.

Keep all game state in one object (seat, actors in play, turn, metric values, starting values, tracks, used events, facts, history, current turn) and save it to localStorage after every change, wrapped in try/catch.

### AI roles

Use separate calls for separate roles. Never let one call both play an actor and judge it.

- **Event narrator (umpire voice):** given the rolled event and the current state, writes what concretely happens in at most 70 words and may move 0 to 2 metrics by 1. If an actor who would normally decide the event is not in play, the umpire decides.
- **AI players:** one call per actor, run in parallel while the human writes. Each sees its own brief, the public state, established facts and the event, never the other arguments. Ask for JSON with action, results (metric key and +1 or -1, usually 1 or 2, at most 3) and reasons.
- **Umpire ruling:** one call that sees all revealed arguments, the state and the facts ledger. It rates each strong, average or weak, sets a modifier, may drop disproportionate claimed effects (never add), writes one sentence for success and one for failure, and proposes a short binding fact for success. Tell it to judge substance not eloquence, to treat average as the norm, to rate three-metric claims one level lower, and to treat contradicting an established fact as weak unless the argument explicitly argues for reversal.
- **Contest:** the player may challenge the ruling on their own argument once per turn. The umpire changes the rating only for a substantive point.
- **Advisor:** acts as the player's chief of staff. It sees only what the player sees. It can suggest three distinct options (for example diplomatic, coercive, economic) each with likely rating and main risk, and a "Use as draft" button fills the argument form. It also answers free-text questions about the scenario or about matrix games.
- **Debrief:** an after-action review with sections for where things stand, how each actor fared, turning points and lessons for the human player.

Validate every AI response in code: keep only known metric keys, force deltas to +1 or -1, cap the count, and fall back sensibly (an AI actor whose call fails passes the turn). Handle `not_granted` and `rate_limited` errors with a plain message, never a loop.

### Turn loop

1. **Begin turn:** roll the event, show the event reveal (below), apply its effects.
2. **Argue:** the human writes a three-part argument while AI players write theirs in the background.
3. **Seal:** once the human submits, wait for all AI players, then call the umpire.
4. **Rule:** show every argument with its rating, threshold, modifier and rationale. Offer the contest.
5. **Resolve:** roll for each argument, apply effects, add facts, update tracks, record history.

### UX requirements

These came from real play-testing. Build all of them.

- **Block the page while the game is working.** Use a full-screen overlay and set `inert` on the page behind it. Show what is happening as a short checklist with the current step highlighted, plus a simple animation. For the umpire's single call, advance the checklist on a timer through what it weighs (capabilities, proportion, established facts, how moves collide, thresholds) and hold on the last step until it returns.
- **Make random events impossible to miss.** Show two dice tumbling, then landing on the real values, then the event name stamped in large type with its description, then the umpire's account and any metric changes. Wait for the player to click Continue. In the event table, highlight this turn's event and strike through past ones.
- **Reveal dice one argument at a time** in the overlay (actor, roll, threshold, success or failure), then let results slide in.
- **Pulse the metrics that just changed** on the board after an event and after resolution.
- **Color changes by meaning, not sign.** Red means a change moves a metric toward its crisis zone, green away from it, neutral when no crisis is defined. Military escalation going up is red even though it is +1.
- **Every label and pill explains itself on click.** Metric names, metric chips, actor badges, event pills, strength ratings and section labels open a small popover next to the item (a bottom sheet on phones). No info icons. Use a faint dotted underline on text labels. Close on outside click, Escape or scroll. Make them keyboard reachable.
- **Use tabs to save space:** Current turn (board, facts, turn panel), History (full turn record with the umpire's reasoning), Debrief (generates itself when opened, caches until another turn is played, with a Rewrite button). Keep New game alone in the upper right of the header.
- **Scroll back to the top of the game** after the event card closes, after submitting an argument, after rulings and after resolution, so the player starts reading from the top.
- **Add a footer** crediting the original game with links to its PDF and publisher, stating it is a prototype and not affiliated, and noting that AI players voice actors analytically.
- **Respect reduced motion:** skip tumbling, stamping and pulsing.
- **Phones:** put the turn panel above the board, stack everything in one column under 900px, keep tap targets at least 44px.

### Visual design

Take the picture from the scenario's own world. For a maritime game, a schematic chart strip of the key waterway with a ship that moves along the lane as the navigation metric changes worked well. For a land or political game, consider a stylized map of the contested region or a single track that matters most. Keep it to one header graphic. The board and turn panel are the real interface.

If the header graphic is a full-bleed SVG that crops with `preserveAspectRatio="xMidYMid slice"`, its edges are cut off at some widths. Scale its height with the viewport width, compute the visible part of the viewBox in script, and move edge labels and moving markers inside it on resize.

## Step 4: Test before publishing

You cannot call `sample` outside the artifact, so test with a mock. In a headless browser (Playwright if available), inject a fake `window.claude.use` that returns plausible JSON for each prompt type, with short delays. Play at least one full turn: start, event reveal, advisor options and "Use as draft", a question to the advisor, submit, contest, resolve, history tab, debrief tab. Check for page errors and take screenshots at phone, laptop and wide desktop widths. Look at them.

Things that broke in the reference build and are worth checking every time:

- An element with the `hidden` attribute still showing because a CSS `display` rule overrides it. Add `[hidden]{display:none!important}`.
- Header graphic labels cut off at certain widths (see Visual design).
- The overlay never closing after an AI error. Every path must turn it off.
- Form text lost when the turn panel re-renders. Save drafts on input and only re-render the AI status list while the player is typing.
- A reload mid-turn leaving the game stuck. Roll a half-started turn back, and restart missing AI players when loading into the argue phase.

## Step 5: Publish

Write the page to a file and publish it with the artifact tool, with a short title and an icon. Then declare the capability: `declare_capabilities` with `{"sample": {}}`. A new page is published first and declared second.

Later updates: republish the file. If the person has shared the artifact publicly, an update goes live to everyone with the link, so confirm with them before republishing.

## Handing it over

Tell the person, in plain language:

- which seats and options exist and how a turn flows
- what you defaulted or wrote yourself (crisis zones, metric descriptions, missing rules)
- that the first AI call will ask their permission
- that you tested with stand-in AI answers, so the first real game is the real test, and the main thing to judge is whether the umpire's ratings feel fair
- the natural next steps: tune the umpire, then add multiplayer (the artifact's `room` and `db` capabilities support shared play within an organization), then consider a full app if it needs public sign-ups
