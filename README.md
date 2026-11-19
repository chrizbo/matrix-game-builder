# Matrix game builder

A Claude skill that turns a matrix wargame into a playable web page.

Give Claude a matrix game (usually a PDF with a scenario, actor briefs, metric tracks and a random event table) and it builds a single page where you pick a seat, AI players take the other seats, and a separate AI umpire rates every argument before the dice decide.

It is written for facilitators, trainers and analysts who know matrix games but have never built an interactive experience. Claude does the technical work and explains the result in terms of play.

## What you get

- Seat selection, including optional actors
- The full turn loop: random event, sealed arguments, umpire ruling, dice, results
- A situation board with metric tracks, an established-facts ledger and a turn history
- An advisor that suggests options and answers questions for players new to the scenario or to matrix games
- An after-action debrief that writes itself
- Click-to-explain descriptions on every metric, actor, event and rating
- Saved games that resume where you left off

The page runs as a Claude artifact and calls Claude directly, so there is no API key or server to set up.

## Install

The skill is the `matrix-game-builder` folder in this repo.

- **Claude app:** download this repo, zip the `matrix-game-builder` folder, and add it as a custom skill in Claude's settings.
- **Claude Code:** copy the `matrix-game-builder` folder into `~/.claude/skills/`.

## Use

Attach or link your matrix game and ask something like:

> Turn this matrix game into a playable artifact I can play solo.

Claude reads the game, writes a game spec, checks any gaps with you (for example undefined crisis thresholds or missing maps), builds the page, tests it, and publishes it as an artifact.

## Before you share a game

Matrix games belong to their designers. A private prototype for your own learning is usually fine. If you plan to share a playable version widely, get the designer's permission. Pages built with this skill always credit the original game and say they are not affiliated with it.

If a scenario involves real states or armed groups, the AI players voice them analytically and the umpire stays neutral. The page says so.

## Background

This skill generalizes a prototype built for [Narrow Waters: The U.S.-Iran Conflict](https://www.shadowboxtraining.com/wp-content/uploads/2026/11/Narrow-Waters-Matrix-.pdf), a matrix game from [Shadowbox Training](https://www.shadowboxtraining.com/). The UX and gameplay lessons from play-testing that prototype are written into the skill. This project is not affiliated with or endorsed by Shadowbox Training.

## Status

Early. Issues and pull requests with lessons from other games are welcome.

## License

MIT
