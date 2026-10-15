# CLAUDE.md

This is the hub of the user's **game lab**: a collection of multiplayer web game prototypes for playtesting with friends. This repo is only a static landing page (`index.html` renders `games.json`), deployed by GitHub Pages from `main` to https://poeticmatter.github.io/game-lab/.

## How the lab is organized

- **One repo per game**, cloned next to this one in `C:\Code\<Repo>`. Each game is standalone and deploys to `https://poeticmatter.github.io/<Repo>/`, so it can later be taken full-scale on its own.
- **Templates:** every game is created from one of two templates, each with a replaceable `src/game/` and an example game. Their READMEs and CLAUDE.mds describe the game contract.
  - `poeticmatter/game-template` (local copy: `C:\Code\game-template`): two players choose secretly at the same time, then the turn resolves. Example: rock-paper-scissors.
  - `poeticmatter/game-template-turns` (local copy: `C:\Code\game-template-turns`): 2–5 players take turns in order, with names and seats. Example: Pig.
- **GameKit** (`poeticmatter/GameKit`, local copy: `C:\Code\GameKit`): the shared multiplayer platform both templates use (lobby, Supabase async play, PeerJS live play, hot seat), with one entry point per kind of game (`game-kit/simultaneous`, `game-kit/turns`). Games pin it to a release tag in `package.json` and bump it deliberately. Older games (Hex Tag, Flow Fighter, Delivery Van) still carry their own copy in `src/platform/`.
- **Shared Supabase project:** `game-lab`, project ref `zavdrnttsfcupopdidda` (reachable through the Supabase MCP). Each game owns its own table, `<game-id>_games`. The ids in use are listed in the README table.
- **Keep-alive:** `.github/workflows/supabase-keepalive.yml` pings the project every 3 days so the free plan doesn't pause it.
- Games never import from this hub or from each other.

## Creating a new game

When the user asks to create or start a new game:

1. **Settle the names.** Ask if they weren't given:
   - `<Repo>`: GitHub repo name in PascalCase, e.g. `SpiralDuel`.
   - `<game-id>`: short, lowercase letters, digits or underscores, e.g. `spiral`. It must not already be in the README table.
   - `<Title>`: display name, e.g. `Spiral Duel`.
   - A one-line description for the hub.
   - **Which template:** do players choose at the same time (two players, `game-template`) or take turns in order (2–5 players, `game-template-turns`)?
2. **Create the repo from the chosen template**, from `C:\Code`:
   ```bash
   gh repo create poeticmatter/<Repo> --template poeticmatter/<template> --public --clone
   ```
   GitHub copies the template in the background, so the clone can come out empty. If it does, wait a few seconds and run `git pull origin main`.
3. **Initialize it**, in `C:\Code\<Repo>`:
   ```bash
   npm install
   npm run init-game -- <Repo> <game-id> "<Title>"
   cp ../game-template/.env.local .env.local
   ```
   Update the tagline in `src/game.config.ts` and replace the README with a short one for the new game.
4. **Create its table:** apply `supabase/migrations/0001_create_<game-id>_games.sql` to project `zavdrnttsfcupopdidda` with the Supabase MCP `apply_migration` tool. Name the migration `<game-id>_0001_create_<game-id>_games`. (For the turn-based template this also creates the `<game-id>_claim_seat` function.)
5. **Commit and push**, then deploy and turn on Pages:
   ```bash
   npm run deploy
   gh api -X POST repos/poeticmatter/<Repo>/pages -f "source[branch]=gh-pages" -f "source[path]=/"
   gh repo edit poeticmatter/<Repo> --homepage "https://poeticmatter.github.io/<Repo>/"
   ```
6. **Register it here:** add an entry to `games.json` and a row to the README's table list, then commit and push this repo.
7. **Hand off:** the new repo still contains the template's example game (rock-paper-scissors or Pig). Tell the user to open a new Claude Code session in `C:\Code\<Repo>` to design and build the actual game in `src/game/`.

## Retiring or graduating a game

Remove its entry from `games.json` and from the README table. Its repo, deployment and table stay as they are. If the game graduates to its own Supabase project, move its migrations over and drop its table here.

## Not set up yet

- **Email turn notifications for async play** (on hold): the plan is a `match_subscribers` table with insert-only RLS, a trigger on each `<game-id>_games` table, and one shared Edge Function. The open question is the sender: Resend needs a verified domain; the alternatives are Gmail SMTP or a Discord webhook.
