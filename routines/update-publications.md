---
name: update-publications
purpose: Keep the publications list on www.labourlaw.at current by gathering new entries from public sources, diffing them against what is live, and drafting an update for human review before publishing.
inputs:
  - Author name(s) to track (default: Andreas Tinhofer; optionally Eva Krichmayr)
  - Path to the publications master file (default: data/publications.yaml)
  - Target page on the site (default: the author page on www.labourlaw.at)
  - Publish mode (default: draft-for-review)
when-to-use: When the publications page on the firm website is outdated, on a recurring cadence (e.g. monthly), or after a new article/book/talk is released.
---

# Update Publications (labourlaw.at)

Maintain the publications list with as much automation as possible while
keeping a human in the loop. Nothing goes live unreviewed, and **no citation
is ever invented or altered** — every entry must be traceable to a source URL.

The flow has three stages: **Gather → Reconcile → Publish.** A master file
(`data/publications.yaml`) is the single source of truth; the website is a
rendered view of it.

## Sources to gather from

Pull candidate publications from these authoritative public sources. Record a
`source_url` for every entry.

- LexisNexis author page — journal articles (ecolex, ZAS, DRdA, ASoK, ARD,
  RdW, ...): https://lesen.lexisnexis.at/autor/Tinhofer/Andreas
- Linde Verlag author page — books / contributions:
  https://shop.lindeverlag.at/person/tinhofer-andreas-6697
- MANZ shop — books (e.g. *KI und Arbeitsrecht*, ISBN 978-3-214-25670-8):
  https://shop.manz.at
- RDB Keywords "Arbeitsrecht" (rdb.at) — editorial role
- Firm news/blog on www.labourlaw.at — talks, Fachtagungen (e.g. the
  "Automatisierung der Arbeit" symposium at WU Wien)
- Optional, if they exist: ORCID, Google Scholar, LinkedIn "Publications"

> Network note: direct page fetches may be blocked (publisher WAFs return 403;
> some sites refuse the fetch tool). Run this routine in an environment whose
> network policy allows these domains. If direct fetch is unavailable, fall
> back to web search to locate entries, but treat search hits as *candidates*
> to verify, never as final citations.

## Steps

1. **Read** the current master file (`data/publications.yaml`). If it does not
   exist, create it by extracting the list currently shown on the site (this
   is the one-time bootstrap).
2. **Gather** candidates from every source above. For each candidate normalize
   to the schema in *Master file format* below. Capture the verbatim citation
   string from the source — do not reformat numbers or reword titles.
3. **Reconcile** candidates against the master file:
   - `NEW` — found in sources, absent from master.
   - `CHANGED` — present but citation/title/year differs (show old vs. new).
   - `STALE` — in master but not found in any source. **Flag only**; never
     auto-delete (a publisher page dropping an old article does not mean it
     should leave the CV).
   - De-duplicate across sources (same work listed by two publishers).
4. **Draft** the proposed master file and a change summary (see *Output
   format*). Stop here and present it for review.
5. **On approval**, write the updated `data/publications.yaml`, then render and
   publish (see *Publishing*).

## Master file format

`data/publications.yaml` — one entry per publication:

```yaml
- type: article        # article | book | commentary | chapter | talk | editorial
  authors: [Tinhofer, A.]
  title: "…"
  venue: "ecolex"      # journal, publisher, conference, or commentary work
  year: 2024
  citation: "ecolex 2024/123, 456"   # verbatim from source; the canonical form
  lang: de             # de | en
  source_url: "https://…"
  status: live         # live | draft | stale-review
```

## Output format (the review draft)

```
## Publications update — <date>

### New (<n>)
- <type> | <citation> | <source_url>

### Changed (<n>)
- <title>
  - was: <old citation>
  - now: <new citation>  (<source_url>)

### Stale — needs your call (<n>)
- <citation>  (no longer found at <last source>; keep?)

### Proposed diff to data/publications.yaml
<unified diff>
```

Keep it scannable. Group by type within each section if the list is long.

## Publishing

After approval, render the master file to the site. Choose the path that
matches the platform (the site appears to run WordPress):

- **WordPress REST API (preferred, highest automation):** PATCH the target
  page via `POST /wp-json/wp/v2/pages/{id}` using an Application Password
  (store as an env var, never in the repo). Render entries grouped by year
  (newest first) as a Gutenberg/HTML block. Verify the response, then confirm
  the live page.
- **Manual fallback:** emit a ready-to-paste HTML block grouped by year and
  hand it to whoever edits the site. Use this when no API credential is
  available or the site is on a builder (Wix/Squarespace/etc.).

## Recurring run

To keep it hands-off, schedule the **Gather → Reconcile → Draft** stages
(e.g. monthly) and only notify when there is a non-empty diff. The Publish
stage always waits for approval. Schedule via a Claude Code on the web
scheduled session, or run `/loop 720h /update-publications` locally.

## Guardrails

- Never invent, complete, or "tidy up" a legal citation. Copy it verbatim from
  the source; if you cannot get the exact citation, mark the entry as
  unverified and leave it out of the published list.
- Never auto-delete entries. STALE items are flagged for the user's decision.
- Never publish without explicit approval of the draft.
- Never commit API credentials or Application Passwords — use env vars.
- If a source page can't be reached, report which one and stop rather than
  guessing its contents.
