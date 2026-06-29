# Output contract (shared, single source of truth)

This file defines the EXACT shape every final answer must take, no matter which
runtime executes the workflow (Claude skill or GitHub Copilot). The
`song-finder` agent produces it; the orchestrator returns it verbatim. For the run
directory, file names, and resume rules, see [run-io.md](run-io.md).

## Hard rules

1. **Exclude every input song and every input artist's input track.** None of the
   songs from the user's seed playlist may appear in the output. (A seed artist
   may reappear with a *different* song only if it genuinely fits; prefer
   broadening the roster.)
2. **Honor the requested length.** Default to 100 songs if the user did not say a
   number. If they asked for N, output exactly N.
3. **No duplicates.** Every `song - artist` line is unique.
4. **Real, findable songs only.** No invented titles. If unsure a track exists,
   drop it and pick another.

## The two blocks (order matters)

The final message contains exactly two fenced code blocks, in this order, with no
prose before, between, or after them other than a single optional one-line label
above each.

### Block 1 — the playlist (the only thing the user copies)

A plain ```text``` fenced block. Each line is:

```
Song Title - Artist
```

- One song per line, `Song Title - Artist`, separated by ` - ` (space hyphen space).
- **No** numbering, **no** section headers, **no** blank lines, **no** commentary,
  **no** emojis, nothing but `Song - Artist` lines.
- This block must be cleanly copy-pasteable into Spotify/Apple Music importers.

### Block 2 — the post-amble (explanation)

A separate ```markdown``` fenced block placed AFTER block 1. It explains the
choices so the user understands how the new list matches and expands the seed.

- It is general and may be sectioned (e.g. "Core vibe", "Tempo arc", "Genre mix",
  "Bridges to the event"). It does **not** need to explain every song.
- Reference concrete signals: how it matches the seed's vibe, specific anchor
  songs, BPM ranges, keys, genres (in order of frequency), era, energy arc, and
  how it expands the seed for the requested event/vibe.
- No em dashes or en dashes anywhere (use commas, periods, or rewrite).

## The spreadsheet (third artifact)

Also write a CSV file with one row per output track and the reasoning. Save it to
the run's output location (see orchestrator). Columns, in this exact order:

```
position,song,artist,genre,bpm,key,energy,role,reason
```

- `energy`: a 1 to 5 score (1 = slow/intimate, 5 = peak hype).
- `role`: what the track does for the set (e.g. "opener", "slow dance",
  "floor-filler", "singalong", "cooldown", "bridge to country", "nostalgia pop").
- `reason`: 1 to 3 sentences on why this track was chosen and how it ties to the
  seed playlist or the requested event.
- Quote any field containing a comma per RFC 4180.

If the runtime cannot write files (e.g. Claude mobile), output the CSV as a THIRD
fenced ```csv``` code block instead, after the post-amble, and say so in one line.
