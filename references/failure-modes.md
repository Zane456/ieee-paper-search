# Common pitfalls (do this / don't do that)

## 1. Source selection

❌ `paper-search search "..." -s all`
→ noise explosion: a DAB converter search pulled in a Hindi phonetics paper (title also contained "converter")

✅ `paper-search search "..." -s openalex,semantic`
→ academic results, IEEE papers come with abstracts

---

❌ `paper-search search "..." -s crossref` (IEEE topic)
→ 6/6 papers with empty abstracts, impossible to judge content

✅ `paper-search search "..." -s openalex,semantic`
→ abstracts complete

---

## 2. Search keywords

❌ `"DAB converter"` alone
→ hundreds of candidates, direction far too broad

✅ `"DAB converter ZVS modulation"` with qualifiers
→ 17 precise candidates

---

❌ Searching with non-English keywords (most sources do not index them)
→ low hit rate

✅ English terms + domain acronyms ("DAB", "ZVS", "TPS modulation")
→ high hit rate

---

## 3. Triage ranking

❌ Sorting by date only, newest first
→ misses foundational high-citation papers (Everts 2014, 471 citations, but old)

✅ Citations + venue tier + year **combined**
→ mix old and new: 1-2 foundational + 3-5 recent SOTA

---

❌ Dropping anything with 0 citations
→ misses methods younger than 1 year

✅ At 0 citations check year and venue: <1 year + a top journal like TPEL = still worth reading

---

## 4. Presenting to the user

❌ Pasting every 200-word abstract in full
→ heavy reading load, user gives up

✅ Compress each abstract to 2-4 sentences (method / result / validation)
→ user scans 10 papers in 30 seconds

---

❌ No verdict per paper, leaving the user to judge
→ user has to re-read everything

✅ Add a verdict: must-read / representative / new-method / background / optional / drop
→ user picks directly

---

## 5. Fetching full text

❌ Bulk-downloading PDFs right after presenting the list
→ user may not want all of them; wastes time and disk

✅ Wait for the user's picks → then download
→ on demand

---

❌ Running `ieee-fetch` without the institutional VPN
→ HTTP 502, a 4 KB HTML error page

✅ Verify the IP first with `curl ifconfig.me`, or check file size afterwards (a real PDF is 100 KB+)

---

❌ Using ieee-fetch on Elsevier / Springer papers
→ ieee-fetch only works on the IEEE domain

✅ Decide the download path by DOI prefix (10.1109 → ieee-fetch; anything else → tell the user)

---

## 6. After output

❌ Pretending to have read the PDF (inventing full-text content from the abstract)
→ hallucinated citations and claims

✅ Actually download the PDF → open it with the host agent's PDF reader (or `pdftotext`) → answer from the full text
→ accurate

---

## 7. Non-IEEE papers leaking into the list

❌ Listing everything openalex / semantic returned (IEEE + Elsevier + Springer + arxiv + ...)
→ the user picks the Elsevier one and you cannot fetch it (ScienceDirect sits behind Cloudflare)

✅ Filter by DOI prefix immediately after searching: keep only `10.1109/` and `10.23919/`, drop the rest
→ every listed paper is fetchable via `ieee-fetch`

Special case: keep arxiv preprints whose DOI is `10.1109/...` (preprints later published by IEEE) — two paths exist: the arxiv pdf_url or ieee-fetch.

If more than 50% of candidates got filtered out, tell the user: "IEEE coverage of this direction is thin — for Elsevier/Springer switch to the generic `paper-search` skill and triage manually." Never silently accept a starved candidate pool.

---

## 8. Repeated work

❌ Re-searching from scratch every time the user says "find me a few more on X"
→ overlapping candidates, wasted tokens

✅ Cache the previous results at `/tmp/survey-<topic>/results.json` and check it first
(not enforced by this skill, but remember: do not re-search what you already searched)
