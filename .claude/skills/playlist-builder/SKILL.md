---
name: playlist-builder
description: "Build a new playlist that matches and expands the vibe of a seed playlist, excluding the seed's songs. Use when the user pastes a small playlist (lines of `Song - Artist`) and asks to create a larger playlist for an event or vibe (wedding, party, workout, road trip, etc.), e.g. 'create a 100 song playlist for <vibe> based on the vibes of this playlist'. Runs an agentic researcher -> analyzer -> song-finder flow and returns a copy-paste-ready playlist block, an explanation post-amble, and a reasoning spreadsheet. Trigger phrases: build a playlist, make a playlist, playlist from this playlist, expand this playlist, playlist for a wedding/party based on these songs."
argument-hint: "Paste the seed playlist plus the target (event/vibe and song count)"
---

# Playlist Builder

An agentic workflow that takes a small seed playlist and produces a new, larger
playlist that captures the same vibe and sentiment **without reusing any of the
seed songs**, tailored to a requested event or vibe.

It runs three agent phases in sequence. Each phase has a dedicated spec file in
this skill folder. Read the spec for a phase, do that phase's work fully, hold its
output, then move to the next phase. The phases share one goal: match the seed's
DNA, then expand it tastefully for the event.

## When to use
The user pastes a seed playlist (lines of `Song - Artist`) and asks for a bigger
playlist for some event or vibe. If the user did not give a song count, default to
100. If they did not name an event/vibe, ask one short clarifying question, then
proceed.

## The agentic flow

Run these in order. Do not skip ahead, each phase consumes the previous one's
output.

1. **Researcher** — Profile every seed song (genre, BPM, key, energy, era, role,
   reception) and summarize the seed's overall style, vibe, and genres in order of
   frequency. Spec: [agents/researcher.md](agents/researcher.md).
2. **Analyzer** — Turn that profile into a quantitative build spec: genre quotas
   summing to N, an energy arc for the event, BPM/era targets, lyrical guardrails,
   and an expansion strategy (adjacent artists/scenes to mine). Spec:
   [agents/analyzer.md](agents/analyzer.md).
3. **Song-Finder** — Select N real songs that fill the quotas and ride the arc,
   exclude every seed song, track per-track reasoning inline, and emit the final
   output. Spec: [agents/song-finder.md](agents/song-finder.md).

If you have the ability to spawn subagents (e.g. Claude Code, GitHub Copilot),
run each phase as an isolated subagent and pass only its summary forward. If you
do not (Claude.ai web/mobile), run the phases sequentially in one context.

## Output contract (strict)

The final answer MUST follow [reference/output-contract.md](reference/output-contract.md)
exactly. In short:

- **Block 1**: a single ```text``` code block of `Song - Artist` lines only,
  exactly N lines, no headers, no numbering, no commentary, nothing else, so the
  user can copy just this block into a music app.
- **Block 2**: a separate ```markdown``` code block, the post-amble, explaining
  how the new list matches and expands the seed (sectioned, references BPM, genre
  frequency, anchors, energy arc). It need not explain every song.
- **Spreadsheet**: `playlist.csv` written to disk when possible
  (`position,song,artist,genre,bpm,key,energy,role,reason`), otherwise emitted as
  a third ```csv``` block.

Hard rules: exclude every seed song, no duplicates, real songs only, hit N
exactly, and use no em dashes or en dashes anywhere.

A worked reference (couple's wedding) is in
[reference/example-wedding.md](reference/example-wedding.md). Use it for tone and
scope only, never copy its songs.

## Where to write files (when a file system is available)
Create `runs/<short-slug>/` next to where you are working and write
`research.md`, `analysis.md`, and `playlist.csv` there. The slug is a kebab-case
summary of the request (e.g. `wedding-100`). On Claude web/mobile, skip files and
use the code-block fallbacks described in the output contract.
