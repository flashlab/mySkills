---
name: pubmed
description: Search PubMed/PMC medical literature via MCP server. Use for literature search, retrieving abstracts, full-text articles, and citation analysis. Supports MeSH terms, batch fetching, PDF downloads, and RIS export.
---

# PubMed Search Skill

Direct PubMed/PMC API access via `web_fetch`. No dependencies required.

## API Key (Required Step)

Before making any request, read the NCBI API key from the environment variable
`PUBMED_TOKEN` (configured in `~/.claude/settings.json` → `env.PUBMED_TOKEN`).

- PowerShell: `$env:PUBMED_TOKEN`
- Bash / POSIX: `$PUBMED_TOKEN`
- Inside `web_fetch` / `web-router`: substitute the literal value into the URL.

Append `&api_key=<PUBMED_TOKEN>` to **every** `eutils.ncbi.nlm.nih.gov` URL
(esearch, esummary, efetch, elink). This raises the rate limit from 3 → 10 req/s.

If `PUBMED_TOKEN` is unset or empty, fall back to anonymous (3 req/s) and warn
the user once. Do **not** append `api_key=` to non-eutils hosts (`icite.od.nih.gov`,
`www.ncbi.nlm.nih.gov/pmc/...`) — they ignore it.

## API Endpoints

| Function | Endpoint |
|----------|----------|
| Search | `eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi` |
| Summary | `eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi` |
| Fetch | `eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi` |
| Similar | `eutils.ncbi.nlm.nih.gov/entrez/eutils/elink.fcgi` |
| Citations | `icite.od.nih.gov/api/pubs` |
| PMC Full Text | `www.ncbi.nlm.nih.gov/pmc/utils/oa/oa.fcgi` |

> `${PUBMED_TOKEN}` below is the literal value of `env.PUBMED_TOKEN`.
> Always append `&api_key=${PUBMED_TOKEN}` to eutils URLs (omit if unset).

## 1. Search (with Sort)

```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pubmed&term=QUERY&retmax=20&retmode=json&sort=SORT&api_key=${PUBMED_TOKEN}
```

**Sort options:**
- `sort=relevance` — Best Match (ML-based, default)
- `sort=pub_date` — Most Recent

**Example:**
```
?db=pubmed&term=gastric+cancer[mh]+AND+artificial+intelligence[mh]&retmax=10&retmode=json&sort=pub_date&api_key=${PUBMED_TOKEN}
```

Returns: `idlist` (PMIDs), `count`, `querytranslation`

## 2. Get Summaries (titles, authors, PMCID)

```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi?db=pubmed&id=PMID1,PMID2&retmode=json&api_key=${PUBMED_TOKEN}
```

Returns: title, authors, journal, pubdate, pmcid (if available)

## 3. Fetch Abstracts

```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&id=PMID1,PMID2&rettype=abstract&retmode=xml&api_key=${PUBMED_TOKEN}
```

Returns: Full abstract XML with all metadata

## 4. Count Only (fast)

```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pubmed&term=QUERY&retmax=0&retmode=json&api_key=${PUBMED_TOKEN}
```

Use `retmax=0` for fast count without retrieving IDs.

## 5. Find Similar Articles

```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/elink.fcgi?dbfrom=pubmed&db=pubmed&id=PMID&cmd=neighbor_score&retmode=json&api_key=${PUBMED_TOKEN}
```

Returns related articles ranked by similarity score.

## 6. Citation Counts (iCite)

```
https://icite.od.nih.gov/api/pubs?pmids=PMID1,PMID2,PMID3
```

Returns: citation_count, relative_citation_ratio, nih_percentile, etc.

## 7. PMC Full Text / PDF URLs

```
https://www.ncbi.nlm.nih.gov/pmc/utils/oa/oa.fcgi?id=PMCID
```

Returns OA article links including PDF URLs.

## 8. RIS Export

```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&id=PMID1,PMID2&rettype=medline&retmode=text&api_key=${PUBMED_TOKEN}
```

Returns MEDLINE format (convertible to RIS).

## Query Syntax

See [references/query-syntax.md](references/query-syntax.md) for MeSH terms and search fields.

## Workflow Example

```python
import os
key = os.environ.get("PUBMED_TOKEN", "")
suffix = f"&api_key={key}" if key else ""

# 1. Search with relevance sort
search = web_fetch(f"...esearch.fcgi?term=gastric+cancer+AI&sort=relevance&retmax=20{suffix}")
pmids = search["idlist"]

# 2. Get summaries
summaries = web_fetch(f"...esummary.fcgi?id={','.join(pmids)}{suffix}")

# 3. Fetch abstracts for top hits
abstracts = web_fetch(f"...efetch.fcgi?id={pmids[0]},{pmids[1]}&rettype=abstract{suffix}")

# 4. Get citation counts (iCite — no api_key)
citations = web_fetch(f"https://icite.od.nih.gov/api/pubs?pmids={','.join(pmids)}")
```

## Rate Limits

- Without API key: 3 req/s
- With API key (`env.PUBMED_TOKEN`): 10 req/s — automatically applied
- Respect ≥0.1 s between calls when authenticated; ≥0.34 s otherwise

Get a new key: https://www.ncbi.nlm.nih.gov/account/settings/ (then set
`env.PUBMED_TOKEN` in `~/.claude/settings.json`).
