# Game Lab

A landing page for game prototypes: [poeticmatter.github.io/game-lab](https://poeticmatter.github.io/game-lab/).

Each game is its own repository and deploys to its own GitHub Pages path. This repo is only a list of links, and games never depend on it. To take a game full-scale, remove its entry here and keep developing it in its own repo.

## Adding a game

1. Create the game from [poeticmatter/game-template](https://github.com/poeticmatter/game-template) (see its README).
2. Add an entry to `games.json`:
   ```json
   { "title": "…", "description": "…", "url": "https://poeticmatter.github.io/<Repo>/", "repo": "https://github.com/poeticmatter/<Repo>" }
   ```
3. Commit and push. GitHub Pages serves this repo straight from `main`, with no build step.

## Shared Supabase project

All games share one Supabase project, `game-lab` (`zavdrnttsfcupopdidda`). The free plan pauses idle projects, so `.github/workflows/supabase-keepalive.yml` sends it a cheap read every three days. Each game owns its own table, `<game-id>_games`, created by the migration in that game's repo:

| Game | Table |
|---|---|
| Hex Tag | `hextag_games` |
| Flow Fighter | `flow_games` |
| Delivery Van | `van_games` |
| Game Template | `template_games` |
