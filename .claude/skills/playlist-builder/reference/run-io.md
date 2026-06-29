# Run I/O contract (shared, single source of truth)

Every phase **writes its output to disk** before returning, and **reads the prior
phase's files from disk** instead of carrying them in the chat context. This is
what lets a run be debugged, and resumed from the middle if a phase errors or
stops (the song-finder phase in particular has hit token limits when everything
was held in context).

## Run directory

Create one directory per request and put every artifact in it:

```
runs/<slug>/
  research.md      <- researcher: handoff summary (vibe, genres by frequency, anchors, constraints)
  research.csv     <- researcher: per-song details + a final SUMMARY row whose notes cell holds the vibe post-amble
  analysis.md      <- analyzer: the quantitative build spec (quotas, energy arc, targets, expansion)
  playlist.txt     <- song-finder: final list, `Song - Artist` lines only (the copy-paste block)
  postamble.md     <- song-finder: the explanation post-amble
  playlist.csv     <- song-finder: per-track reasoning
```

`<slug>` is a short kebab-case summary of the request, e.g. `wedding-100`.

When no file system is available (Claude.ai web/mobile), skip files and use the
code-block fallbacks in [output-contract.md](output-contract.md); the per-phase
handoff is then carried as a compact summary in context instead.

## Resume / skip rule (run from the middle)

Before running a phase, check whether its output files already exist in
`runs/<slug>/` and are non-empty:

- researcher done  -> `research.md` and `research.csv` exist
- analyzer done    -> `analysis.md` exists
- song-finder done -> `playlist.txt`, `postamble.md`, and `playlist.csv` exist

If a phase's outputs already exist, SKIP it and pass the paths forward. Only re-run
a phase whose output is missing, empty, or was explicitly flagged for redo. This
makes a resumed run cost only the unfinished work.

## Token discipline (prevents the song-finder token error)

1. **Pass paths, not contents.** Never paste a prior phase's full file back into
   the next phase or into chat. Read the file narrowly (the handoff summary first,
   the full file only if needed).
2. **Write incrementally.** The song-finder builds the playlist in batches of about
   25 songs, appending each batch to `playlist.txt` and `playlist.csv` as it goes,
   so it never holds the entire list plus all reasoning in one response.
3. **Keep the big artifact on disk.** Do NOT echo `playlist.csv` in chat (it is the
   largest piece). The final chat message is only the two blocks from the output
   contract: the playlist text and the post-amble. The CSV is referenced by path.
   (Mobile/no-FS is the only exception, where the CSV is a third code block.)
4. **No transcripts.** Subagents return a 2 to 4 line summary plus the paths they
   wrote, never their working notes.

## research.csv shape

One row per seed song, then one final summary row.

Header:

```
position,song,artist,genre,subgenre,bpm,key,energy,era,role,reception,notes
```

- Per-song rows: `position` is 1..M (M = number of seed songs). `energy` is 1 to 5.
  `role` is what the song does in a set. `reception` is recognizability / public
  reaction. `notes` is a short per-song observation.
- Final row: `position` = `SUMMARY`, `song` = `OVERALL VIBE`, leave the musical
  columns blank, and put the **vibe post-amble in the `notes` cell**: 3 to 6
  sentences on the seed's overall vibe, the genres by frequency, and exactly what
  the new playlist will match against and how it will expand the seed for the
  event. Quote the cell (it contains commas) per RFC 4180.
