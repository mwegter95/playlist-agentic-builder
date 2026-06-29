# Playlist Agentic Builder

An **agentic workflow** that takes a small seed playlist and builds a new, larger
playlist that captures the same vibe and sentiment, **without reusing any of the
seed songs**, tailored to a requested event or vibe (wedding, party, workout, road
trip, and so on).

It runs three agent phases:

```
Researcher  ->  Analyzer  ->  Song-Finder
 profile        build spec     final playlist + post-amble + CSV
 the seed       (quotas,       (N real songs, none from the seed)
                energy arc)
```

The same workflow runs in **two runtimes** from **one shared source of truth**:

- **Claude** (desktop app, web, or mobile) as a Skill.
- **GitHub Copilot** (VS Code) as an agentic prompt with three subagents.

## What you get

Every run returns three artifacts:

1. **The playlist** — one clean code block of `Song - Artist` lines (and nothing
   else), so you can copy it straight into Spotify or Apple Music.
2. **A post-amble** — a separate block explaining how the new list matches and
   expands the seed (vibe, BPM, genres by frequency, anchor songs, energy arc).
3. **Two reasoning spreadsheets** — `research.csv` (per seed song: genre, BPM,
   key, energy, era, role, reception, plus a SUMMARY cell with the overall-vibe
   post-amble) and `playlist.csv` (one row per new track with 1 to 3 sentences on
   why it was chosen).

Each phase writes its outputs to `runs/<slug>/` and reads the previous phase's
files from disk, so a run is **debuggable and resumable from the middle** if a
phase errors or stops.

## Repository layout

```
playlist-agentic-builder/
  README.md
  .claude/skills/playlist-builder/        <- SINGLE SOURCE OF TRUTH (also the Claude skill)
    SKILL.md                              <- Claude entry point + orchestration
    agents/
      researcher.md                       <- phase 1 spec  (shared by both runtimes)
      analyzer.md                         <- phase 2 spec  (shared)
      song-finder.md                      <- phase 3 spec  (shared)
    reference/
      output-contract.md                  <- the strict output format (shared)
      run-io.md                           <- run directory, artifacts, resume + token rules (shared)
      example-wedding.md                  <- worked example for tone/scope
  .github/                                <- GitHub Copilot wiring (thin wrappers)
    prompts/
      playlist-builder.prompt.md          <- Copilot orchestrator (/playlist-builder)
    agents/
      playlist-researcher.agent.md        <- subagent -> reads agents/researcher.md
      playlist-analyzer.agent.md          <- subagent -> reads agents/analyzer.md
      playlist-song-finder.agent.md       <- subagent -> reads agents/song-finder.md
```

**Reuse model:** the agent specs and the output contract live once, under
`.claude/skills/playlist-builder/`. The Copilot agents and prompt are thin
wrappers that point at those same files, so there is one place to edit the logic.
The Claude skill folder is self-contained (everything it needs is inside it), so
it can be zipped and uploaded on its own.

---

## Setup: GitHub Copilot (VS Code)

The wiring is already in `.github/`. With this folder open as a workspace (or the
repo present in your workspace):

1. Open Copilot Chat in Agent mode.
2. Type `/` and pick **playlist-builder** (or run `Chat: Run Prompt...`).
3. Paste your seed playlist and target, for example:

   > create a 100 song playlist for a couple's wedding basing it on the vibes of
   > this input playlist
   >
   > Sweet Caroline - Neil Diamond
   > Wagon Wheel - Darius Rucker
   > ... (rest of seed)

The orchestrator dispatches the three subagents (`playlist-researcher`,
`playlist-analyzer`, `playlist-song-finder`) in order, writes `runs/<slug>/`
artifacts, and returns the two code blocks.

Notes:
- All three subagents have web search and fetch tools, write their outputs to
  `runs/<slug>/`, and read the prior phase's files from disk. If a run stops, the
  next invocation skips completed phases (whose files already exist) and resumes
  from the first incomplete one.
- The song-finder builds large lists in batches of about 25 appended to disk, which
  avoids the token-limit failure that one-shot generation of 100 songs can cause.
- The three `*.agent.md` files are `user-invocable: false`, so they run only as
  subagents of the orchestrator, not as standalone chat agents.
- To make these available in **any** workspace, copy `.github/prompts/*` and
  `.github/agents/*` into your VS Code user prompts folder, or keep this repo in
  a multi-root workspace. (User-scoped skills are not supported, so for the Claude
  skill files use the workspace.)

---

## Setup: Claude (desktop, web, or mobile) as a Skill

The skill lives at `.claude/skills/playlist-builder/` and is self-contained.

### Claude.ai (web and mobile)
1. Go to **Settings -> Capabilities -> Skills** (or the Skills area of your
   account) and choose to add a custom skill.
2. Zip the **`playlist-builder/` folder** (the one containing `SKILL.md`,
   `agents/`, and `reference/`) and upload it.
3. In any chat (including the mobile app), paste your seed playlist and ask, e.g.
   "create a 100 song playlist for a wedding based on the vibes of this playlist".
   Claude loads the skill from its description and runs the flow. On mobile, the
   CSV comes back as a third code block (no file system), per the output contract.

### Claude Desktop / Claude Code
1. Place (or symlink) the `playlist-builder/` folder into your Claude skills
   directory:
   - Claude Code: `~/.claude/skills/playlist-builder/` (or a project-level
     `.claude/skills/playlist-builder/`, which this repo already provides).
2. Invoke it by describing the task in chat, or in Claude Code via the skill name.
   Where a file system is available, it writes `runs/<slug>/research.md`,
   `analysis.md`, and `playlist.csv`.

---

## Usage contract (applies to both runtimes)

- **Input:** a seed playlist (`Song - Artist` lines) plus a target (event/vibe and
  optional count). Default count is 100.
- **Output block 1:** only `Song - Artist` lines, exactly N, no headers, no
  numbering, copy-paste ready.
- **Output block 2:** the post-amble explanation (separate block).
- **Output artifacts:** `research.csv` and `playlist.csv` (or third `csv` blocks
  on mobile).
- **Resumable:** each phase writes to `runs/<slug>/`; a stopped run resumes from
  the first phase whose output files are missing.
- **Hard rules:** never reuse a seed song, no duplicates, real songs only, hit N
  exactly, and no em dashes or en dashes.

## Editing the workflow

Change behavior in one place: the specs under
`.claude/skills/playlist-builder/agents/` and the format/IO in
`.claude/skills/playlist-builder/reference/` (`output-contract.md`, `run-io.md`).
Both runtimes pick up the change because the Copilot wrappers reference these same
files.
