# Worked example — couple's wedding (100 songs)

This is a reference for tone and scope, not a script to copy. The actual output
must be freshly generated for the user's real seed and prompt.

## Example user input

> I'm looking to create a 100 song playlist based on this one for a couple's
> wedding.
>
> Sweet Caroline - Neil Diamond
> Wagon Wheel - Darius Rucker
> Chicken Fried - Zac Brown Band
> Pink Pony Club - Chappell Roan
> Save a Horse (Ride a Cowboy) - Big & Rich
> (Simply) The Best - Jimmy Barnes, Tina Turner
> Forever and Ever, Amen - Randy Travis
> Dancing Queen - ABBA
> No Body - Blake Shelton
> Don't Stop Believin' - Journey
> Wonderful Tonight - Eric Clapton
> Give It All We Got Tonight - George Strait
> Girls Just Want to Have Fun - Cyndi Lauper
> Friends in Low Places - Garth Brooks Tribute
> Summer Of '69 - Bryan Adams
> Old Time Rock & Roll - Bob Seger
> That's My Kind Of Night - Luke Bryan
> Bye Bye Bye - *NSYNC
> I'm Still Standing - Elton John

## What good looks like for this seed

- **Researcher** identifies the seed as a crowd-pleasing wedding-reception blend:
  modern + classic country (Zac Brown, Randy Travis, George Strait, Luke Bryan),
  classic-rock singalongs (Journey, Bob Seger, Bryan Adams), 80s pop (ABBA, Cyndi
  Lauper, Elton John), one slow dance (Wonderful Tonight), one current pop crossover
  (Pink Pony Club), and a nostalgic boy-band moment (NSYNC). Genres by frequency:
  country first, then classic rock, then 80s/pop. Energy skews mid-to-high with a
  couple of slow-dance anchors. Family-friendly, multi-generational, singalong-heavy.
- **Analyzer** turns that into targets, e.g.: ~40 country (mix modern + classic),
  ~25 classic-rock/singalong, ~20 pop/dance across 80s to today, ~10 slow dances /
  first-dance candidates, ~5 wildcard floor-fillers. Define an energy arc (dinner
  -> first dance -> open floor -> peak -> last call) and guardrails (clean-ish
  lyrics, broad recognizability, avoid breakup songs at a wedding).
- **Song-finder** produces 100 NEW tracks (none from the seed), e.g. more George
  Strait/Randy Travis-adjacent classics, Shania Twain, Brooks & Dunn, Luke Combs,
  Morgan Wallen-style modern country, plus Whitney/Earth Wind & Fire/Bon Jovi/
  Footloose-era floor-fillers and current crossover pop, then explains the arc and
  ships the CSV.

The point: match the seed's DNA, then expand it tastefully for the event.
