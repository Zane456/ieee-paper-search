---
name: ieee-paper-search
description: IEEE literature survey (first-choice paper-finding skill in IEEE-heavy domains). Given a topic or keywords, search IEEE-only papers (DOI prefix 10.1109 / 10.23919) via OpenAlex + Semantic Scholar for abstracts, triage by recommendation, present as cards with abstracts translated into the user's language, then after user confirmation download PDFs with ieee-fetch and parse with the pdf skill. Priority — in power electronics / power supply / DAB / converter / Simulink / MATLAB / signal processing / IEEE journals (TPEL, TIE, TIA, SPL, etc.) this skill outranks the generic paper-search skill (same tool class, this one is narrower and smarter). deep-research is out of scope for comparison — that is a survey/meta-analysis pipeline, not a paper lookup tool; only use it when the user explicitly says "write a survey / lit-review / meta-analysis". Triggers - 查论文、找论文、文献调查、文献调研、找 X 方面的工作、找 IEEE 上的论文、找 paper、survey papers about X、find papers on X、literature survey、look up papers.
---

# IEEE Paper Survey Skill

## What it does

When asked "what papers are out there on X", run this pipeline: search → de-noise → rank → present cards → user picks → fetch full text.

## What it does NOT do

- Does not write survey articles (that is `deep-research`, if installed)
- No citation graphs, no meta-analysis
- Never bulk-downloads dozens of PDFs without user confirmation

## Main flow

### 1. Tighten the query
If the user's keywords are too broad ("DAB converter"), add one qualifier ("DAB converter ZVS modulation"). A more specific query means less noise.

### 2. Search (**hard constraint**)

```bash
uv run --directory ~/paper-search-mcp paper-search search "<query>" \
  -n 8 -s openalex,semantic [-y <year>]
```

(Adjust `--directory` if paper-search-mcp is cloned elsewhere — see README.)

Parameters:
- `-n 8`: 8 hits per source (after dedup usually 8-15 papers, enough to triage)
- `-s openalex,semantic`: **hard constraint** — do not use `-s all`, do not run crossref alone (rationale in [references/sources.md](references/sources.md))
- `-y <year>`: year filter (Semantic Scholar side). Add proactively when the user says "last 3 years" / "latest" / "since 2022". Format `2022` or `2020-2024`

Special case: add `arxiv` when preprints are wanted (`-s openalex,semantic,arxiv`).

### 3. De-noise + dedup + **IEEE-only filter**

**Hard filter**: keep only papers whose DOI prefix ∈ {`10.1109/`, `10.23919/`}, drop everything else. Rationale — the only reliable full-text path in this skill is `ieee-fetch`; a non-IEEE paper survives triage but its PDF is unreachable, wasting the user's time.

Concretely, filter the `papers[*]` array of the `paper-search` JSON output:
```
keep iff paper.doi startswith "10.1109/" or "10.23919/"
```

Then on the survivors:
- Drop off-domain hits (a DAB search must not include Cuk converters or NLP "converter" papers)
- Dedup by DOI (the same IEEE paper may appear once each in openalex + semantic + arxiv; keep any one)

**Tell the user**: if too much got filtered (>50% of candidates dropped), proactively say "IEEE coverage of this direction is thin — switch to the generic `paper-search` skill for Elsevier/Springer?"

### 4. Rank

Priority order:
1. **Citation count** (>50 anchor, >10 active, <10 judge by year)
2. **Venue tier** (top journal > top conference > regular journal > preprint)
3. **Year** (<3 years for SOTA, >5 years for foundational)
4. **Authors/affiliation** (bonus for anchor groups in the field)

Detailed criteria: [references/triage.md](references/triage.md)

### 5. Card-style presentation (**critical — never use tables**)

Card format, one per recommended paper:

```
### [N] Original paper title (keep English)

🔗 https://doi.org/10.1109/xxx
📅 <year> · 📍 <venue short name, e.g. TPEL / IECON> · 📊 citations <N>
👤 <first author; corresponding author (et al.)>

**Abstract (translated)**: <translate the English abstract into the user's
conversation language, 2-4 sentences, keeping method / key novelty /
experimental results / validation scale>

💡 **Verdict**: <must-read / representative / new-method / background / ...>
(one-sentence reason)

---
```

Requirements:
- **List only the recommended ones** (already triaged); dropped papers get a one-line dismissal at the end
- **Always translate** the abstract into the user's conversation language — never paste the raw English abstract
- Link as `https://doi.org/<doi>` (directly clickable)

Full template and examples: [references/output-template.md](references/output-template.md)

### 6. Wait for the user to pick
**No proactive bulk download.** After the cards, ask "which ones do you want in full text / or change keywords?"

### 7. Fetch full text

**Single path**: `ieee-fetch <arnumber|url>`

Downloads to `~/.claude/skills/ieee-paper-search/downloads/ieee-<arnumber>.pdf` by default and auto-appends a line (time / filename / size) to `INDEX.md` in the same directory. To see what has been downloaded, Read that INDEX.md — **never `ls` the whole PDF pile into context**.

Your machine's IP must be inside your institution's IEEE Xplore-registered range (campus network or VPN). Detailed constraints: [references/ieee-fetch.md](references/ieee-fetch.md).

> Why not `paper-search download`: IEEE does not expose PDFs to paper-search's built-in downloader, even behind an institutional VPN. `ieee-fetch` is a session-warmed curl written specifically for IEEE, ~6 s per paper.
>
> For IEEE preprints **also hosted on arxiv** (search result shows source `arxiv` + DOI `10.1109/...`), the arxiv `pdf_url` is a fallback via plain curl (no VPN needed), but the preferred path is still `ieee-fetch` for the IEEE version of record.

### 8. (Optional) Read the PDF

`Read <path>.pdf` — renders the PDF into context (figures, summaries, method extraction).

### 9. Chain into other skills (**recommend proactively**)

After the PDFs land, depending on what the user wants next, **ask** whether to chain (only offer skills that are actually installed):

| User intent | Recommended skill |
|---|---|
| Extract tables / figures / annotations / references from PDFs | `pdf` skill |
| Turn several papers into a survey | `deep-research` pipeline |
| Write a paper on top of these references | `academic-paper` skill |

Ask format: "Downloaded. Continue with — A. pdf skill table extraction / B. switch to deep-research for a survey / C. something else?" Never act silently.

## Common pitfalls

See [references/failure-modes.md](references/failure-modes.md). The three that bite most:

1. **Searching IEEE topics with crossref alone** → all abstracts empty, triage impossible
2. **Searching with `-s all`** → noise explosion (Hindi NLP, Cuk converter, other false hits)
3. **Running ieee-fetch without the institutional VPN** → HTTP 502 "Temporarily Unavailable"

## Golden case

A full end-to-end run kept as the reference benchmark: [references/golden-case-dab-zvs.md](references/golden-case-dab-zvs.md) ("DAB converter ZVS modulation" → 17 candidates → 12 IEEE papers).
