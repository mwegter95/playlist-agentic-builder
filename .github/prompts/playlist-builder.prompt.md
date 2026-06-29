---
description: "Build a new playlist that matches and expands the vibe of a pasted seed playlist, excluding the seed's songs. Runs an agentic researcher -> analyzer -> song-finder flow and returns a copy-paste playlist block, an explanation post-amble, and a reasoning CSV."
name: Playlist Builder
argument-hint: "Paste the seed playlist plus the target (event/vibe and song count)"
agent: agent
tools: [read, edit, search, web, agent]
---

# Playlist Builder (agentic workflow)

Take the user's seed playlist and target from the chat input (and any attached
files) and produce a new playlist that captures the same vibe **without reusing
any seed song**, tailored to the requested event/vibe.

The shared, canonical logic lives in
[.claude/skills/playlist-builder](../../.claude/skills/playlist-builder/SKILL.md).
This prompt is the GitHub Copilot orchestrator for that same workflow.

## Inputs to gather
- The **seed playlist**: lines of `Song - Artist`.
- The **target**: event/vibe and song count `N`. If `N` is missing, default to
  100. If the event/vibe is missing, ask one short question, then proceed.

## Run the agentic flow

Dispatch each phase as a subagent (#tool:agent) in order. Each subagent **writes
its files to `runs/<slug>/` and reads the prior phase's files from disk** (see
[run-io.md](../../.claude/skills/playlist-builder/reference/run-io.md)); pass only
paths plus a short summary forward, never raw transcripts. Before dispatching a
phase, skip it if its outputs already exist (resume from the middle):

1. **playlist-researcher** — profile the seed; writes `research.md` +
   `research.csv` (per-song rows + a SUMMARY cell with the vibe post-amble).
2. **playlist-analyzer** — convert that into a quantitative build spec; writes
   `analysis.md` (genre quotas summing to N, energy arc, BPM/era targets,
   guardrails, expansion strategy, exclusions).
3. **playlist-song-finder** — select N real songs in batches of ~25 appended to
   disk (avoids token limits), exclude every seed song, write `playlist.txt`,
   `postamble.md`, and `playlist.csv`, and produce the final two-block output.

If subagents are unavailable, run the three phases sequentially yourself using the
specs in [.claude/skills/playlist-builder/agents](../../.claude/skills/playlist-builder/agents/).

## Final output (strict)

Return exactly what
[output-contract.md](../../.claude/skills/playlist-builder/reference/output-contract.md)
defines:

1. A ```text``` block of `Song - Artist` lines only (exactly N, no headers, no
   numbering, no other text) so the user can copy just this block.
2. A separate ```markdown``` post-amble explaining how the list matches and
   expands the seed (sectioned; reference BPM, genre frequency, anchors, energy
   arc; need not cover every song).
3. The reasoning spreadsheet at `runs/<slug>/playlist.csv`
   (`position,song,artist,genre,bpm,key,energy,role,reason`).

Hard rules: exclude every seed song, no duplicates, real songs only, hit N
exactly, no em dashes or en dashes.
