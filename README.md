# Gamemaster

An always-on game bot for a Politics & War alliance Discord. Every game has a lobby open at all
times, each game gets its own channel, eliminated players are muted, and the social deduction games
run at a slow pace so members in every time zone can take part.

## One-time setup (about 5 minutes)

1. **Create the bot.** Go to <https://discord.com/developers/applications> → **New Application** →
   name it **Gamemaster** → open the **Bot** tab → **Reset Token** → copy the token.
2. **Turn on one switch.** Still on the **Bot** tab, under *Privileged Gateway Intents*, enable
   **Message Content Intent** (needed for typing games like Hangman) and save.
3. **Give the bot its token.** Copy `.env.example` to `.env` and paste the token after
   `DISCORD_TOKEN=`. Never share this file.
4. **Start it.** Double-click `start.bat` (or run `npm start`). The window prints an **invite link**.
5. **Invite it.** Open the invite link in your browser and add the bot to your server.
6. **Switch it on.** In your server, run `/admin setup`. The bot creates three categories —
   *Gamemaster* (the board, results, voice and every lobby), *Gamemaster live games* and
   *Gamemaster secret rooms* — and opens a lobby for every game.

The bot only runs while that window is open. For 24/7 play, run it on an always-on machine
(`npm install` then `npm start`, ideally under a process manager such as pm2).

## The games (25)

**Social deduction (14)** — Mafia · Werewolf · One Night Coup · Shadow Cabinet · Black Ops ·
Ghost Council · Undercover · Safehouse · Decoy · The Leak · Faker · Assassin · Sabotage · Defuse

**Strategy and party (11)** — Missile Standoff · Orbis Royale · Pass the Nuke · Number Bomb ·
War Games · Trivia Blitz · Hangman · Word Scramble · Type Racer · Quick Draw · Most Likely To

## How it runs

- **Every game is always open.** Each game keeps one lobby open permanently; when a lobby starts, a
  new one for that game opens straight away. Players can be in several games at once.
- **Slow pace for social deduction.** In every social deduction game except Assassin, each turn
  (a night, a vote, a decision) stays open **18–20 hours** and ends early once everyone has acted.
  Players are pinged when a turn opens and reminded before it closes. A slow lobby waits up to
  12 hours for more players once it has enough; the first player or an admin can press
  **Start now**. Change the length with `/admin pace` (0 makes everything live).
- **Party games are live.** They start a few seconds after enough players join and take minutes.
- **Restarts are safe.** Lobbies keep the players who joined. Slow games in progress are saved turn
  by turn and continue from the start of the turn that was open (anything already submitted in that
  turn has to be done again). Live party games in progress are dropped, since they only take minutes.

## For players

| Command | What it does |
| --- | --- |
| *(board menu and buttons)* | Join any game, set a nickname, see your profile, read rules |
| `/games` | Private copy of the board with the join menu |
| `/leave` | Leave a game (run it inside that game's channel if you are in several) |
| `/nick set` · `/nick clear` | Nickname used only inside games |
| `/profile` · `/leaderboard` | Stats, coins, level, achievements |
| `/daily` · `/shop` | Daily coins; spend them on a badge shown next to your name |
| `/howto` | Rules for any game |

## For admins

Anyone with **Manage Server**, the role set by `/admin adminrole`, or added with `/admin grant`.

| Command | What it does |
| --- | --- |
| `/admin setup` | Create or repair the Gamemaster channels |
| `/admin grant` · `/admin revoke` | Give or remove a member's admin powers over the bot |
| `/admin start` | Open an extra lobby for any game immediately |
| `/admin forcestart` | Start a lobby without waiting for the countdown |
| `/admin end` | Stop a game or close a lobby |
| `/admin kick` | Remove a player from their game |
| `/admin players` | Every player in a game: nickname → real account |
| `/admin whois` | Who is behind a nickname (or what nickname a member uses) |
| `/admin ban` · `/admin unban` | Block or allow a member in games |
| `/admin pace` | Hours each slow turn stays open (18 = 18–20 hours, 0 = everything live) |
| `/admin slots` | 0 = every game always open (default); N = rotate N games, picked by `/vote` |
| `/admin memberrole` | Hide the arcade from everyone except one role |
| `/admin coins` · `/admin status` | Adjust coins; view config and the last error |

## How muting works

- Game channels are read-only for everyone except the current players.
- When a player is eliminated the bot removes their permission to type, and server-mutes them if
  they are sitting in the arcade voice channel. Everything is restored when the game ends.
- In Mafia and Werewolf the channel locks at night, the killers get a private den, and the dead get
  a private afterlife chat. In Ghost Council the dead are deliberately *not* muted.
- Server admins bypass channel permissions in Discord, so for them the bot deletes out-of-turn
  messages instead.

## Development

`npm test` needs no token. It plays every game to completion with simulated players, crashes slow
games mid-turn and checks the rebuilt game matches the original exactly, and soak-tests the whole
bot — including a restart — against a fake Discord server.

To add a game, copy a file in `src/games/` and list it in `src/games/index.js` (Discord menus hold
25 entries, so that is the cap). For a game to survive restarts, its `run()` must learn about player
input only through `poll` / `ask` / `askAll` / `voteOut` / `step`, and take randomness from
`src/util.js` rather than `Math.random` — see the note at the top of `src/game.js`.
