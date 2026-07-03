[English](README.md) | [简体中文](README.zh-CN.md)

<div align="center">

# ieee-paper-search

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Platform: Claude Code](https://img.shields.io/badge/Platform-Claude%20Code-blueviolet.svg)](https://claude.com/claude-code)
![Type: Agent Skill](https://img.shields.io/badge/Type-Agent%20Skill-blue.svg)
![Scope: IEEE only](https://img.shields.io/badge/Scope-IEEE%20only-brightgreen.svg)

<br>

**An agent skill that turns "find me papers on X" into a triaged IEEE reading list — abstracts translated into your language, PDFs one confirmation away.**

<br>

[Prerequisites](#-prerequisites-read-first) · [What it does](#what-it-does) · [Install](#install) · [Usage](#usage) · [How it works](#how-it-works) · [FAQ](#faq)

</div>

---

## ⚠️ Prerequisites (read first)

This skill is useless without all three of these. Check them before installing.

1. **Your network must have IEEE Xplore access.**
   PDF download works by institutional IP recognition — your machine's IP must be inside an IP range your university/company has registered with its IEEE Xplore subscription. In practice that means being on the **campus network or the institutional VPN**. No subscription → search and triage still work, but full-text download returns HTTP 502.
   Quick check: `curl -sS https://ifconfig.me` must show an IP in your institution's range, and opening a paywalled paper on `ieeexplore.ieee.org` in a browser should show the PDF button unlocked.

2. **[Claude Code](https://claude.com/claude-code) is the recommended platform.**
   This is an Agent Skill (`SKILL.md` + references), designed and daily-driven in Claude Code. Other harnesses that read SKILL.md may work but are untested.

3. **[paper-search-mcp](https://github.com/openags/paper-search-mcp) + [uv](https://docs.astral.sh/uv/)** provide the search backend (OpenAlex + Semantic Scholar). Install steps below.

---

## What it does

You say:

> "Find me papers on DAB converter ZVS modulation"

The skill runs: **search → IEEE-only filter → de-noise → rank → cards → you pick → fetch PDFs**, and answers with cards like this (abstract translated into your conversation language):

```
### [1] Optimal ZVS Modulation of Single-Phase Single-Stage Bidirectional DAB AC–DC Converters

🔗 https://doi.org/10.1109/tpel.2013.2292026
📅 2014 · 📍 IEEE TPEL · 📊 citations 471
👤 Jordi Everts; ... ; Johann W. Kolar (ETH/KU Leuven)

**Abstract (translated)**: <2-4 sentences in your language — method, key novelty,
experimental results, validation scale>

💡 Verdict: must-read (field anchor, 471 citations, founding paper of charge-based ZVS analysis)
```

Then you reply "1 and 4" and it downloads exactly those PDFs via `ieee-fetch` — a session-warmed curl that authenticates purely by your institutional IP (~6 s per paper, no IEEE account, no stored cookies).

### Why IEEE-only

Everything listed is guaranteed fetchable. Papers whose DOI prefix is not `10.1109/` or `10.23919/` are dropped up front, because listing an Elsevier paper you cannot download wastes your time. If the filter kills >50% of candidates, the skill tells you and suggests a generic search instead.

### Design choices baked in

- **OpenAlex + Semantic Scholar only** as search sources — Crossref returns empty abstracts for IEEE papers (measured: 0/6 vs 6/6), and `-s all` floods you with false hits
- **Ranking** = citations + venue tier + year + author group, with explicit anti-patterns (a 0-citation paper <1 year old on TPEL is kept, not dropped)
- **Cards, not tables** — each paper gets a translated 2-4 sentence abstract in an engineering register and a one-word verdict (must-read / representative / new-method / background / optional)
- **Never bulk-downloads** without your confirmation, and dropped papers are always listed with reasons

## Install

```bash
# 1. The skill itself
git clone https://github.com/Zane456/ieee-paper-search.git ~/.claude/skills/ieee-paper-search

# 2. The ieee-fetch downloader
mkdir -p ~/.local/bin
cp ~/.claude/skills/ieee-paper-search/scripts/ieee-fetch ~/.local/bin/
chmod +x ~/.local/bin/ieee-fetch
# make sure ~/.local/bin is on your PATH

# 3. The search backend
git clone https://github.com/openags/paper-search-mcp.git ~/paper-search-mcp
cd ~/paper-search-mcp && uv sync
```

If you clone paper-search-mcp somewhere other than `~/paper-search-mcp`, adjust the path in `SKILL.md` (one `uv run --directory` line).

## Usage

In Claude Code, just ask in natural language:

- "查一下 DAB converter ZVS modulation 方面的论文" (any language works)
- "find papers on model predictive control for three-level inverters"
- "survey papers about GaN gate driver design, last 3 years"

The skill auto-triggers on paper-survey phrasing. Downloads land in `downloads/` inside the skill folder (gitignored), with an auto-maintained `INDEX.md`. Override the location with `$IEEE_FETCH_DIR`.

You can also use the downloader standalone:

```bash
ieee-fetch 10374220
ieee-fetch "https://ieeexplore.ieee.org/document/10374220"
ieee-fetch 10374220 /tmp/paper.pdf
```

## How it works

| Stage | Tool | Detail |
|---|---|---|
| Search | paper-search-mcp CLI | `-s openalex,semantic`, 8 hits per source, optional year filter |
| Filter | DOI prefix | keep `10.1109/` and `10.23919/` only, dedup by DOI |
| Triage | [references/triage.md](references/triage.md) | citations → venue tier → year → author group |
| Present | [references/output-template.md](references/output-template.md) | cards with translated abstracts + verdicts |
| Fetch | [scripts/ieee-fetch](scripts/ieee-fetch) | warm session on `/document/<id>`, then `stampPDF` — IP-based auth |
| Read | Claude Code `Read` | PDF straight into context for summaries and method extraction |

The `references/` folder also documents [source-selection rationale](references/sources.md), [8 field-tested failure modes](references/failure-modes.md), and a [full golden-case run](references/golden-case-dab-zvs.md).

## FAQ

**It says HTTP 502 / I get a 4 KB file.**
Your IP is not inside your institution's IEEE range. Connect the campus VPN and re-check with `curl ifconfig.me`.

**HTTP 200 but content-type is text/html.**
Your institution's subscription does not cover that journal.

**Can it fetch Elsevier / Springer / Wiley?**
No, by design. Those publishers use different access mechanics. Non-IEEE papers are filtered out before you ever see them.

**Is this bypassing the paywall?**
No. It uses exactly the access your institution already pays for — the same IP-based mechanism your browser uses on campus. No account sharing, no credential storage, no circumvention. You are responsible for complying with your institution's and IEEE's terms of use; downloaded PDFs are for your own research use and must not be redistributed.

## License

[MIT](LICENSE) © [Zane456](https://github.com/Zane456)
