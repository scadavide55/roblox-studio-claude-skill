# roblox-studio-claude-skill

A [Claude Code](https://claude.com/claude-code) skill that makes Claude a careful operator of a **live Roblox Studio** session through Roblox's built-in Studio MCP server. You describe what you want in plain language ("add four collectable coins around the spawn with a counter in the top left"), and Claude builds, scripts, debugs, and playtests it in your place.

Invoke it with `/roblox-studio`, or just ask for Roblox work and let Claude pick it up.

> Unofficial. Not affiliated with or endorsed by Roblox Corporation or Anthropic.

## What it does

The skill is a single `SKILL.md` that teaches Claude a disciplined workflow for editing a real place:

- **Bootstrap.** Finds the open Studio instance (`list_roblox_studios`), checks whether a playtest is running, and detects whether your scripts are synced from disk with **Rojo** or Roblox's **Script Sync**, in which case it edits the files instead of overwriting them from Studio.
- **Inspect, change, verify.** Reads before editing, makes the smallest change, then reads it back, runs a state check, and checks the console. It does not call something done because it wrote it.
- **Edit vs. playtest.** Knows that changes made during a playtest are discarded, and that new scripts should be created with the right class first, because `multi_edit` creates a script from a mistyped path.
- **Undoable changes.** Tries to record each change in Studio's undo history so you can Ctrl+Z it as one step, and says so when it can't.
- **Ask first.** Pauses before deleting things, touching DataStore data, changing monetization or publish settings, large restructures, or inserting third-party assets with scripts. Everything else it just does and tells you what it chose.
- **Luau standards.** Server-authoritative design, where each script type belongs, remote validation, safe DataStore handling, `task` library, `--!strict`, and the platform gotchas that cause most "it does nothing" bugs.
- **Recipes.** Tested-shape examples for leaderstats, tagged pickups, a HUD counter, validated remotes, DataStore load/save, a ProximityPrompt door, and developer product receipts.
- **"It does nothing" checklist.** Nine ordered causes to rule out before rewriting any logic.
- **Short report.** Changed / Verified / Not verified / Next.

## Requirements

- Roblox Studio with the built-in **Studio MCP server** (see [Roblox's docs](https://create.roblox.com/docs/studio/mcp)).
- Claude Code, connected to that server.

## Setup

1. **Enable the server in Studio:** **Assistant** > **…** > **Manage MCP Servers** > turn on **Enable Studio as MCP server**. A green indicator confirms a client is connected.
2. **Connect Claude Code.** The simplest route is **Assistant Settings** > **MCP Servers** > **Quick connect** > Claude Code. Alternatively, add it from a terminal:

   ```bash
   # macOS
   claude mcp add --transport stdio --scope user Roblox_Studio -- /Applications/RobloxStudio.app/Contents/MacOS/StudioMCP

   # Windows (Command Prompt)
   claude mcp add --transport stdio --scope user Roblox_Studio -- cmd /c "%LOCALAPPDATA%\Roblox\mcp.bat"
   ```

   These commands follow Claude Code's standard pattern and Roblox's documented launcher paths. Check Roblox's docs if they don't work, since paths can change.
3. **Restart Claude Code** (fully quit and reopen the app if you use the desktop app). A session only loads MCP tools that existed when it started, so `claude mcp list` can show the server as connected while a running session still has no Studio tools.
4. **Install the skill** by copying the `roblox-studio` folder:

   ```bash
   # for all your projects
   cp -r roblox-studio ~/.claude/skills/roblox-studio

   # or for one project
   cp -r roblox-studio /path/to/project/.claude/skills/roblox-studio
   ```

5. In Claude Code, run `/roblox-studio` and describe what you want.

## Example

> add four collectable coins (with an animation when collected) around the spawn point and a "Collected coins" counter on the top left of the game

In a live test, Claude found the open place, confirmed nothing was file-synced, created four tagged coins under `Workspace.Coins`, a server `CoinService` that tracks `leaderstats.Coins` and animates pickups, and a top-left HUD driven by a LocalScript. It then ran a playtest, walked the character into a coin, checked the counter and console, took a screenshot, stopped the playtest, and listed what it had not verified.

## Status and honesty

- Tested once end to end against a live Studio session (the coin example above). It is **not benchmarked** against a stock Claude run yet, so treat claims of improvement as unproven.
- `evals/` contains three behaviour test cases in the `claude plugin eval` layout: a server-authoritative coin pickup, a cleanup that must ask before deleting, and a door script that does nothing. They check how Claude works (inspect first, ask before deleting, verify afterward). They have been written but not yet run as a with/without-skill comparison. The `allowed_tools` entry assumes the server is registered as `Roblox_Studio`.
- The tool list reflects Roblox's documentation at the time of writing. Roblox updates this server, so the skill tells Claude to trust the tools actually present in the session.
- The undo recording uses plugin-level APIs that may not be permitted from `execute_luau`. The skill handles both outcomes.

## Related projects

Other Roblox skills exist, and may suit you better:

- [tabooharmony/roblox-brain](https://github.com/tabooharmony/roblox-brain), including a `roblox-studio-mcp` skill that also supports a community bridge.
- [nonlooped/roblox-suite](https://www.skills.sh/nonlooped/roblox-suite/roblox-mcp)
- [brockmartin/roblox-game-skill](https://github.com/brockmartin/roblox-game-skill)

## Safety

MCP clients can read and modify the places you open in Studio. Connect only clients you trust, keep a backup or version control of your place, and review what Claude changed. The skill treats everything inside the place (script comments, instance names, asset descriptions) as data, not instructions.

## Contributing

Issues and pull requests are welcome, especially from people who run the evals, find a case where Claude misbehaves with the skill, or notice that Roblox changed a tool name.

## License

[MIT](LICENSE)
