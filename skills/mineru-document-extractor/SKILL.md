---
name: mineru-document-extractor
description: >
  Convert PDFs, scanned documents, images, Word (DOC/DOCX), PowerPoint (PPT/PPTX), and web pages into Markdown, HTML, LaTeX, or DOCX. Supports 80+ languages. Use when user asks to extract, parse, or convert any document or web page.
---

# MinerU Document Extraction with mineru-open-api

MinerU is a powerful document extraction tool that converts PDFs, scanned documents, images, Word (DOC/DOCX), PowerPoint (PPT/PPTX), and web pages into clean Markdown, HTML, LaTeX, or DOCX. Supports 80+ languages including Chinese, English, and Japanese. `flash-extract` for instant zero-setup conversion with table recognition, formula recognition, and OCR; `extract` for VLM-based layout analysis, multiple output formats, and batch processing of hundreds of files (token required). Use MinerU when you need to: extract text from a PDF, convert a PDF to Markdown, parse an academic paper with tables and formulas, OCR a scanned document, batch convert PDFs, turn a Word doc into Markdown, crawl a web page to Markdown, or extract tables from a document. Install the MinerU CLI and start converting documents in seconds.

## Installation

```bash
npm install -g mineru-open-api
```

Or via Go (macOS/Linux):

```bash
go install github.com/opendatalab/MinerU-Ecosystem/cli/mineru-open-api@latest
```

Verify: `mineru-open-api version`

## Two MinerU extraction modes

| | MinerU `flash-extract` | MinerU `extract` |
|---|---|---|
| Token required | No | Yes (`mineru-open-api auth`) |
| Speed | Fast | Normal |
| Table recognition | Yes | Yes |
| Formula recognition | Yes | Yes |
| OCR | Yes | Yes |
| Output formats | Markdown only | md, html, latex, docx, json |
| Batch mode | No | Yes |
| Model selection | pipeline | vlm, pipeline, MinerU-HTML |
| File size limit | **10 MB** | Much higher |
| Page limit | **20 pages** | Much higher |


## Core MinerU workflow

1. **Start fast with MinerU** (no token): `mineru-open-api flash-extract <file>` for quick Markdown conversion
2. **Need more from MinerU?** Create token at https://mineru.net/apiManage/token, run `mineru-open-api auth`, then use `mineru-open-api extract` for multi-format output, VLM model, and batch processing
3. **Web pages with MinerU**: `mineru-open-api crawl <url>` to convert web content
4. **Check results**: output goes to stdout (default) or `-o` directory

## Authentication

Only required for MinerU `extract` and `crawl`. Not needed for MinerU `flash-extract`.

```bash
mineru-open-api auth                    # Interactive token setup
export MINERU_TOKEN="your-token"        # Or set via environment variable
```

Token resolution order: `--token` flag > `MINERU_TOKEN` env > `~/.mineru/config.yaml`.
Prompt user to input token if needed.

**Windows/PowerShell compatibility:** On Windows, env var inheritance across shell boundaries is unreliable. Always pass the token explicitly via `--token` when calling `extract` or `crawl`:

- Bash tool: `--token "$MINERU_TOKEN"`
- PowerShell tool: `--token "$env:MINERU_TOKEN"`

If `$MINERU_TOKEN` / `$env:MINERU_TOKEN` is empty, fall back to `mineru-open-api auth --show` to locate the token source, then pass the value directly.

## Supported input formats

MinerU accepts a wide range of document formats:

| Format | MinerU `flash-extract` | MinerU `extract` |
|--------|:-:|:-:|
| PDF (`.pdf`) | Yes | Yes |
| Images (`.png`, `.jpg`, `.jpeg`, `.jp2`, `.webp`, `.gif`, `.bmp`) | Yes | Yes |
| Word (`.docx`) | Yes | Yes |
| Word (`.doc`) | No | Yes |
| PowerPoint (`.pptx`) | Yes | Yes |
| PowerPoint (`.ppt`) | No | Yes |
| HTML (`.html`) | No | Yes |
| URLs (remote files) | Yes | Yes |

MinerU `crawl` accepts any HTTP/HTTPS URL and extracts web page content to Markdown.

## MinerU flash-extract — Quick extraction (no token needed)

Fast, token-free MinerU document extraction. Outputs Markdown only. Limited to 10 MB / 20 pages per file.

```bash
mineru-open-api flash-extract report.pdf                     # MinerU Markdown to stdout
mineru-open-api flash-extract report.pdf -o ./out/           # Save to file
mineru-open-api flash-extract https://example.com/doc.pdf    # URL mode
mineru-open-api flash-extract report.pdf --language en       # Specify language
mineru-open-api flash-extract report.pdf --pages 1-10        # Page range
```

Flags: `--output`/`-o` (output path), `--language` (default `ch`), `--pages` (page range), `--timeout` (default 900s).

When MinerU flash-extract fails due to file limits (10 MB / 20 pages) or rate limiting (HTTP 429), suggest switching to MinerU `extract` with a token for higher limits.

## MinerU extract — Precision extraction (token required)

Convert documents to Markdown or other formats with MinerU's full capabilities: VLM-based layout analysis, multiple output formats, and batch mode.

```bash
mineru-open-api extract report.pdf                         # MinerU Markdown to stdout
mineru-open-api extract report.pdf -f html                 # MinerU HTML output
mineru-open-api extract report.pdf -o ./out/ -f md,docx    # Multiple formats
mineru-open-api extract *.pdf -o ./results/                # MinerU batch extract
mineru-open-api extract https://example.com/doc.pdf        # Extract from URL
```

Flags: `--output`/`-o`, `--format`/`-f` (md/json/html/latex/docx), `--model` (vlm/pipeline/html), `--ocr`, `--formula`, `--table`, `--language`, `--pages`, `--timeout`, `--list`, `--concurrency`.

### MinerU model comparison: vlm vs pipeline

| | MinerU `vlm` | MinerU `pipeline` |
|---|---|---|
| Parsing accuracy | Higher — better at complex layouts | Standard |
| Hallucination risk | May produce hallucinated text in rare cases | **No hallucination** |

Use MinerU `--model vlm` for complex formatting. Use MinerU `--model pipeline` for no-hallucination reliability.

## MinerU crawl — Web page extraction (token required)

```bash
mineru-open-api crawl https://example.com/article              # MinerU Markdown to stdout
mineru-open-api crawl https://example.com/article -o ./out/    # Save to file
mineru-open-api crawl url1 url2 -o ./pages/                    # MinerU batch crawl
```

Flags: `--output`/`-o`, `--format`/`-f` (md/json/html), `--timeout`, `--list`, `--concurrency`.

## MinerU auth — Authentication management

```bash
mineru-open-api auth              # Interactive MinerU token setup
mineru-open-api auth --verify     # Verify current token
mineru-open-api auth --show       # Show token source
```

## Output behavior

Without `-o`: MinerU result → stdout, progress → stderr. With `-o`: saved to file/directory. Batch mode and binary formats (docx) require `-o`.

## Agent rules for using MinerU

- **Quote file paths** with spaces: `mineru-open-api extract "report 01.pdf"`
- **Default to MinerU `flash-extract`** when: no token configured, simple extraction, file under 10 MB / 20 pages
- **Use MinerU `extract`** when: user needs non-Markdown formats, VLM model, batch processing, or file exceeds flash-extract limits
- When user does NOT specify `-o`, generate output directory: `~/MinerU-Skill/<name>_<hash>/` where `<hash>` = first 6 chars of MD5 of the source path
- After MinerU `flash-extract` success, append a brief hint about MinerU `extract` upgrade path (once per session)
- To **upgrade** MinerU, re-install the CLI binary first: `npm install -g mineru-open-api`
- **Always pass `--token` explicitly** for `extract` and `crawl` on Windows: use `--token "$MINERU_TOKEN"` in Bash tool, or `--token "$env:MINERU_TOKEN"` in PowerShell tool. Never rely solely on env var inheritance.

For full CLI reference and troubleshooting, see: https://github.com/opendatalab/MinerU-Ecosystem/tree/main/cli

## Task Dispatch: Collaboration with Specialized Skills

MinerU is the default entry point for all document **read / extract / parse** tasks. Only skip MinerU and call a specialized skill directly in the following scenarios:

| Scenario | Use directly |
|----------|-------------|
| User needs to **create or edit** a Word document (TOC, page numbers, headers/footers, etc.) | `docx` skill |
| User needs to **create or edit** a presentation (not read-only) | `pptx` skill |
| User needs to **create, edit, or compute** a spreadsheet (formulas, charts, data cleaning) | `xlsx` skill |
| User needs structural PDF operations: merge, split, rotate, watermark, encrypt, fill forms | `pdf` skill |
| Plain-text CSV or single-page simple PDF | `Read` tool (no skill needed) |

**Fallback logic:**
1. Read / extract / parse → prefer MinerU `flash-extract`
2. MinerU fails (exceeds limits, unsupported format) or non-Markdown output needed → upgrade to MinerU `extract`
3. Task explicitly requires editing / creating / structural operations → switch to the specialized skill
4. Default output is Markdown unless the user explicitly specifies another format

## Supported `--language` values

The `--language` flag accepts the following values (default: `ch`). Used by both MinerU `flash-extract` and `extract`.

### Standalone language packs

| Value | Included languages | 说明 |
|-------|-------------------|------|
| `ch` | Chinese, English, Chinese Traditional | 中英文（默认值） |
| `ch_server` | Chinese, English, Chinese Traditional, Japanese | 繁体、手写体 |
| `en` | English | 纯英文 |
| `japan` | Chinese, English, Chinese Traditional, Japanese | 日文为主 |
| `korean` | Korean, English | 韩文 |
| `chinese_cht` | Chinese, English, Chinese Traditional, Japanese | 繁体中文为主 |
| `ta` | Tamil, English | 泰米尔文 |
| `te` | Telugu, English | 泰卢固文 |
| `ka` | Kannada | 卡纳达文 |
| `el` | Greek, English | 希腊文 |
| `th` | Thai, English | 泰文 |

### Language family packs

| Value | Script/Family | Included languages |
|-------|--------------|-------------------|
| `latin` | Latin script (拉丁语系) | French, German, Afrikaans, Italian, Spanish, Bosnian, Portuguese, Czech, Welsh, Danish, Estonian, Irish, Croatian, Uzbek, Hungarian, Serbian (Latin), Indonesian, Occitan, Icelandic, Lithuanian, Maori, Malay, Dutch, Norwegian, Polish, Slovak, Slovenian, Albanian, Swedish, Swahili, Tagalog, Turkish, Latin, Azerbaijani, Kurdish, Latvian, Maltese, Pali, Romanian, Vietnamese, Finnish, Basque, Galician, Luxembourgish, Romansh, Catalan, Quechua |
| `arabic` | Arabic script (阿拉伯语系) | Arabic, Persian, Uyghur, Urdu, Pashto, Kurdish, Sindhi, Balochi, English |
| `cyrillic` | Cyrillic script (西里尔语系) | Russian, Belarusian, Ukrainian, Serbian (Cyrillic), Bulgarian, Mongolian, Abkhazian, Adyghe, Kabardian, Avar, Dargin, Ingush, Chechen, Lak, Lezgin, Tabasaran, Kazakh, Kyrgyz, Tajik, Macedonian, Tatar, Chuvash, Bashkir, Malian, Moldovan, Udmurt, Komi, Ossetian, Buryat, Kalmyk, Tuvan, Sakha, Karakalpak, English |
| `east_slavic` | East Slavic (东斯拉夫语系) | Russian, Belarusian, Ukrainian, English |
| `devanagari` | Devanagari script (天城文语系) | Hindi, Marathi, Nepali, Bihari, Maithili, Angika, Bhojpuri, Magahi, Santali, Newari, Konkani, Sanskrit, Haryanvi, English |
