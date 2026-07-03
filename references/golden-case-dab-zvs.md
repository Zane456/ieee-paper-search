# Golden case: DAB converter ZVS modulation

A real end-to-end survey run, kept as the skill's reference benchmark.

## User input

> "DAB converter ZVS modulation" — find me papers in this direction

## Step 1: search

```bash
uv run --directory ~/paper-search-mcp paper-search search \
  "DAB converter ZVS modulation" -n 6 -s openalex,semantic,crossref,arxiv
```

**Note**: crossref was still included in this run (which taught the lesson that IEEE abstracts come back empty there). The production recipe is `-s openalex,semantic` only.

Returned 17 papers after dedup.

## Step 2: de-noise

Dropped 2 obviously unrelated papers:
- `1705.01833v1` "Akshara to Prosodeme (A2P) Converter in Hindi" → NLP, false hit on "converter" in the title
- `2605.10736v1` "Integrated Magnetics Design for Isolated ZVS Cuk Converter" → Cuk topology, not DAB

15 papers left.

## Step 3: rank + triage

By citations + venue + topical relevance:

| # | Title | Year | Cites | Venue | DOI/source | Abstract gist | Verdict |
|---|---|---|---|---|---|---|---|
| 1 | Optimal ZVS Modulation of Single-Phase DAB AC-DC (Everts) | 2014 | **471** | TPEL | 10.1109/tpel.2013.2292026 | full charge-based ZVS derivation; 3.7 kW EV charger validation | **must-read** |
| 2 | Closed-Form Solution for Efficient ZVS DAB Modulation (Everts) | 2016 | 159 | TPEL | 10.1109/tpel.2016.2633507 | closed form incl. Coss charging | **representative** |
| 3 | Dynamic ZVS-Guaranteed Seamless-Mode-Transition (Gong) | 2022 | 61 | TPEL | 10.1109/tpel.2022.3180759 | (abstract missing, to backfill) | **representative** |
| 4 | Asymmetrical DAB Mod Extending ZVS (Mahdavifard) | 2022 | 40 | TPEL | 10.1109/tpel.2022.3177401 | AEPS + symmetric mode hybrid; 5 kW | **representative** |
| 5 | Globally Unified ZVS + Min Conduction Loss (Yu) | 2021 | 32 | TTE | 10.1109/tte.2021.3131192 | TPS + GOC; GaN 200 kHz | **representative** |
| 6 | Adaptive Mod for Resonant DAB (LCL/CLC) (James) | 2022 | 21 | TIA | 10.1109/tia.2022.3192366 | Fourier method for ZVS boundaries | **optional** |
| 7 | 4-Level NPC DAB Mod Extended ZVS (Miremad) | 2026 | 2 | TPEL | 10.1109/tpel.2025.3597590 | (abstract missing, to backfill) | **new-method** |
| 8 | TPS Mod for ZVS DAB w/ Opt Inductor I (Ma) | 2017 | 9 | IECON | 10.1109/iecon.2017.8216810 | (abstract missing) | **optional** |
| 9 | AI-Based HEPS Mod Full ZVS DAB (Li) | 2023 | 0 | JESTPE | 10.1109/JESTPE.2022.3185090 | XGBoost+PSO; 1 kW 97.1% | **new-method** |
| 10 | AI-Based TPS Min Current Stress (Li) | 2023 | 0 | JESTPE | 10.1109/JESTPE.2021.3105522 | NN+FIS; 1 kW | **optional** |
| 11-15 | others (ISESC / ICPE / SSRN preprint / QAB / MAB) | - | 0 | - | - | (mostly missing abstracts) | **optional** |

(This early run used a table; the production format is the card layout in [output-template.md](output-template.md).)

## Step 4: user picks

The user read the list → "these are roughly enough, I read a few".

Had the user wanted full texts, this is what would have run:

```bash
mkdir -p /tmp/survey-dab-zvs
cd /tmp/survey-dab-zvs
for AR in 10374220 9879543 ...; do  # every IEEE arnumber from the table
  ieee-fetch "$AR"
done
```

## Lessons from this run

1. **A missing abstract does not mean a bad paper** — #3 #7 #8 are TPEL/JESTPE/IECON; the abstracts were missing only because crossref does not store them. openalex backfills most of them.
2. **De-noising matters** — `-s all` drags in Hindi NLP false hits. Narrow sources (`openalex,semantic`) give clean results.
3. **Mix old and new** — Everts 2014 as anchor + Gong 2022 / Mahdavifard 2022 as SOTA is a good combination.

## Not done this time (worth adding next time)

- Abstracts for the missing entries were not backfilled → if the user had continued, query openalex individually by DOI
- No PDFs downloaded (the user did not ask) → ending the flow there is fine
- No backward citation chase ("recent papers citing this anchor") → possible, but not mandatory in this skill
