# CLAUDE.md

This file provides guidance to ClaudeCode (claude.ai/code) when workingwith code in this repository.

## Project Overview

pubmed-cli is a Go CLIfor the NCBI PubMed E-utilities API. It provides deterministic, scriptable literatureresearch workflows including search, fetch, citation graphtraversal, MeSH lookup, and document reference verification.

## Build& Test Commands

```bash
# Build
go build ./...
go build -o pubmed ./cmd/pubmed

# Test (all)
go test ./...

# Test single package
go test ./internal/refcheck/
go test ./internal/eutils/ -run TestSearchParsing

# Vet
go vet ./...

# Cross-platformrelease binaries
make release
```

Integration tests in `eutils/`and `mesh/` hit the real NCBI API — they run in CI but canbe slow locally.

## Architecture

### Layered client design

```
cmd/pubmed/main.go          CLI (cobra commands, flag parsing, output dispatch)
↓
internal/eutils/             E-utilities operations (search, fetch, link)
internal/mesh/               MeSH term lookup
internal/refcheck/           Document reference verification pipeline
    ↓
internal/ncbi/client.go      Shared HTTP client (rate limiting, retry, sizeguard)
```

- **ncbi.BaseClient** — allNCBI HTTP traffic flows through this. Enforces rate limits (3 req/s without key, 10 req/s with key via `golang.org/x/time/rate`), retries on HTTP 429 withexponential backoff, and caps response size at 50MB.
- **eutils.Client** — wraps BaseClient.Search uses JSON (ESearch), fetch uses XML (EFetch), link operationsuse JSON (ELink).
- **refcheck pipeline** — the most complex module. Flow: extract text from .docx (via external`docx-review` binary) → parse references (APA/Vancouver regex) → resolve against PubMed (tiered: PMID → DOI → title→ author+year → relaxed keyword) → detect hallucinations → audit in-text citations → generate report.

### Outputsystem

`internal/output/` dispatches to formatters: human (colored terminal tables), JSON, CSV, RIS. RIS exportis only valid for article-returning commands (fetch, cited-by, references, related).

## CLI Structure

```
pubmed [global flags]<subcommand> [args]
```

Subcommands: `search`, `fetch`, `cited-by`, `references`, `related`,`mesh`, `refcheck`, `version`

Global flags: `--json`, `--human`, `--full`, `--limit`, `--sort`(relevance/date/cited), `--year` (YYYYor YYYY-YYYY), `--type` (review/trial/meta-analysis/randomized/case-report), `--csv`, `--ris`, `--api-key`

## Key Types

| Type | Package | Role |
|------|---------|------|
| `Article` | eutils/types.go | Full PubMed article metadata |
| `SearchResult` | eutils/types.go | Search results with PMID list andcounts |
| `LinkResult` | eutils/types.go | Citation graph links with relevance scores |
| `ParsedReference` | refcheck/types.go | Extracted reference fields fromdocument |
| `VerifiedReference` | refcheck/types.go | Resolution status and match details |

## External Dependencies

- `github.com/spf13/cobra` — CLI framework
- `golang.org/x/time/rate` — rate limiter for NCBI compliance
- `docx-review` — external binary requiredby `refcheck` subcommand for .docx text extraction

## Environment

- `NCBI_API_KEY` — optional; increases rate limit from 3 to 10 req/s
- Testfixtures live in `testdata/` (XML, JSON, .docx samples)
