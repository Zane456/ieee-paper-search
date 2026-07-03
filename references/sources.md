# Source selection rules

## Why hard-limit to OpenAlex + Semantic Scholar

The abstract is the lifeline of triage. Crossref cannot give you abstracts for IEEE / Elsevier / Springer / Wiley papers — these publishers **deliberately withhold abstracts from Crossref** to keep traffic on their own platforms.

Measured on a real DAB ZVS search:
- crossref returned 6 IEEE papers → **0/6 had an abstract**
- openalex returned 6 IEEE papers → **6/6 had an abstract**
- semantic (empty that time, but usually has abstracts)

## Abstract reliability by source

| Source | Abstract | Usage |
|---|---|---|
| **openalex** | ✅ mostly present (incl. IEEE) | primary source |
| **semantic** (Semantic Scholar) | ✅ mostly present | primary source, fills gaps |
| **arxiv** | ✅ always | add when hunting preprints / SOTA |
| **pubmed / pmc / europepmc** | ✅ always | biomedical searches only |
| **biorxiv / medrxiv / iacr** | ✅ present | preprint platforms |
| **doaj** | ✅ present | OA journals |
| crossref | ❌ empty for IEEE and other big publishers | **never use alone** |
| dblp | ❌ metadata only | publication-record verification only |
| unpaywall | ❌ OA status only | for finding open versions |
| google_scholar | ⚠️ incomplete snippets | unreliable |
| ssrn / zenodo / hal | ⚠️ depends on uploader | unstable |
| core / openaire / base / citeseerx | ⚠️ aggregators | uncontrollable |

## Recommended combinations

| Scenario | Sources |
|---|---|
| General paper survey | `-s openalex,semantic` |
| Want SOTA / preprints | `-s openalex,semantic,arxiv` |
| Biomedical | `-s openalex,semantic,pubmed,pmc` |
| Engineering / power electronics / signal processing | `-s openalex,semantic` (openalex has the IEEE papers) |
| Need IEEE full text | `-s openalex,semantic` for candidates → `ieee-fetch` to download |

## Don'ts

❌ `-s all` — too noisy; one DAB run pulled in a Hindi phonetics paper
❌ `-s crossref` alone on IEEE topics — abstracts all empty, triage impossible
❌ Trusting a single source — one source misses papers; openalex + semantic run in parallel as mutual backup
