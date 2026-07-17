# ieee-fetch tool

Installed at `~/.local/bin/ieee-fetch` (a copy lives in this repo under `scripts/`). Pure curl, no browser.

## Usage

```bash
ieee-fetch 10374220                                          # by arnumber
ieee-fetch "https://ieeexplore.ieee.org/document/10374220"   # by document URL
ieee-fetch "https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=10374220"  # by stamp URL
ieee-fetch 10374220 paper.pdf                                # custom output filename
```

Default output: `<skill-root>/downloads/ieee-<arnumber>.pdf`, resolved from the script's own location so it lands beside the skill wherever it is installed (override with `$IEEE_FETCH_DIR`, or pass an explicit path as the second argument).

## How it works

1. GET `/document/<arnumber>` — IEEE issues CloudFront-signed cookies based on your IP
2. GET `/stampPDF/getPDF.jsp?arnumber=<arnumber>` — the cookies unlock the PDF

Institutional identity comes entirely from the IP. **No IEEE account, no login, no persistent cookies stored.**

## Hard constraint

⚠️ **Your IP must be inside your institution's IEEE Xplore-registered range** — in practice that means being on the campus network or the institutional VPN. Which journals you can fetch is decided by your institution's own IEEE subscription.

Check whether the VPN is active:

```bash
curl -sS https://ifconfig.me
# the IP shown must be in your institution's range
```

## Failure modes

| Symptom | Cause | Fix |
|---|---|---|
| `HTTP 502` + short HTML | VPN off / IP not in the institutional range | check the VPN connection |
| `HTTP 200` but content-type `text/html` | institution does not subscribe to that journal | find another access route |
| Timeout | IEEE temporarily refusing service | retry with a sleep |
| File is 4-5 KB | got an HTML error page, not a PDF | the content-type check reports this |

## Not supported

- Elsevier (`10.1016`), Springer (`10.1007`), Wiley (`10.1002`) — different access mechanics per publisher
- Content requiring a personal (not institutional) subscription
- IEEE Standards / some very old papers (>30 years)

## Batch download template

```bash
mkdir -p /tmp/survey-<topic>
cd /tmp/survey-<topic>
for AR in 10374220 9876543 ...; do
  ieee-fetch "$AR" "$(printf '%s.pdf' "$AR")" || echo "FAIL $AR"
done
ls -la
```

## After the PDF lands

Pull the full text for summaries / method extraction / comparison, using whatever the host agent offers — a native PDF reader (Claude Code: `Read <path>.pdf`), an installed `pdf` skill, or `pdftotext <path>.pdf -` on the command line.
