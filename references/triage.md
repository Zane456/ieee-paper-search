# Paper triage criteria

Judge whether a paper deserves the user's time in this priority order.

## 1. Citation count

| Range | Meaning |
|---|---|
| >100 | foundational / classic, must-read |
| 50-100 | anchor paper, representative of the direction |
| 10-50 | active, discussed in the community |
| 1-10 | judge by year; new papers may not have spread yet |
| 0 | judge by year and venue; <1 year is normal, >2 years is a warning sign |

**Important correction**: low citations on a paper younger than 2 years is normal — do not drop it for that alone.

## 2. Venue tier

The DOI prefix identifies the publisher. Common high-quality venues by field:

### Power electronics
- **Top journals**: IEEE TPEL (Trans. Power Electronics), IEEE JESTPE
- **First-tier journals**: IEEE TIE (Trans. Industrial Electronics), IEEE TIA (Trans. Industry Applications), IEEE TTE (Trans. Transportation Electrification)
- **Top conferences**: ECCE, APEC, IECON, ICPE / ECCE-Asia, PCIM
- **Preprints**: arxiv eess.SY

### Signal processing
- Top journals: IEEE TSP, IEEE SPL (Signal Processing Letters), IEEE TIP
- Top conferences: ICASSP, CVPR / ICCV, NeurIPS

### Computer vision / machine learning
- Top conferences > journals: CVPR / ICCV / ECCV / NeurIPS / ICLR / ICML
- Top journals: IEEE TPAMI, IJCV

### Control
- Top journals: IEEE TAC (Trans. Automatic Control), Automatica
- Top conferences: CDC, ACC

Unfamiliar field? DOI prefix `10.1109/` is IEEE, `10.1016/` Elsevier, `10.1007/` Springer, `10.1002/` Wiley, `10.48550/arXiv` arxiv, `10.2139/ssrn` SSRN.

## 3. Year

- **<3 years**: SOTA, current mainstream methods
- **3-5 years**: mature methods, widely used as baselines
- **5-10 years**: foundational, where the method was laid down
- **>10 years**: classic, original theory

For a survey, **mix old and new**: 1-2 foundational (>5 years, high citations) + 3-5 recent SOTA (<3 years).

## 4. Authors / affiliation

Fields have anchor groups. Recognition signals:
- The same first author appears in multiple hits
- Affiliation is a strong group in the field (e.g. ETH Kolar group, TU Delft, Tsinghua, SJTU, HUST for power electronics)
- The corresponding author is a leading figure

Do not force this for unfamiliar fields — citations + venue already cover most of the judgment.

## 5. Title keyword match

Query keywords should appear in the title. If they only appear in a corner of the abstract, relevance is likely weak.

## Verdict template

Keep the verdict to one or two words:

| Verdict | Condition |
|---|---|
| **must-read** | high citations + top journal + keywords in title |
| **representative** | mid-high citations + top journal/conference, representative of the direction |
| **new-method** | low citations but <2 years + top journal + novel angle |
| **background** | foundational paper, for building concepts |
| **optional** | related but off the main line |
| **weakly-related** | abstract shows the direction is off |
| **drop** | false hit / same-name different field |

## Anti-patterns

❌ Deciding the direction from the first hit alone (result ordering is not necessarily by relevance)
❌ Dropping 0-citation papers outright (they may be new)
❌ Picking only high-citation papers (misses the last year's SOTA)
❌ Ignoring venue (predatory journals waste your time)
