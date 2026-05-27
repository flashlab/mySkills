---
name: frontend-dedup
description: Audit and refactor a single-file userscript or small frontend codebase (one JS file with embedded CSS, or a tiny HTML+JS+CSS bundle) to remove duplication, dead hooks, and redundant rules. Use this whenever the user asks to "optimize", "clean up", "dedupe", "simplify", "tidy", "shrink", or "review for redundancy" a Tampermonkey / Greasemonkey / Violentmonkey userscript, browser extension content script, or single-page frontend file. Also use when the user explicitly points at duplicated logic between two functions ("the extraction logic in A and B has common parts — fold it into A"), at a CSS block ("audit class names", "merge similar rules", "find dead classes"), or at single-use wrappers that should be inlined. Do NOT use for large multi-file SPAs, build-system migrations, or framework refactors — this skill assumes one file (or a small handful) and no build tooling.
---

# Frontend Dedup

Mechanical, low-risk cleanup pass on a small frontend file. Two ordered phases — **JS/logic** first, then **CSS audit** — because removing dead JS class hooks first lets the CSS pass see the actual surface.

## Operating constraints

- **Don't change exception-catching boundaries.** When folding code into a helper, every caller's existing `try/catch` must still wrap the same set of throw sites. Verify by tracing what throws (network calls, parsers, JSON, DOM coercions) and confirming the catch still sees them.
- **Don't add abstractions that aren't paying for themselves.** Three similar lines beats a premature helper. Only extract when the duplication is real (≥2 sites with the same shape) and the helper has a clear name.
- **Don't introduce utility CSS classes that require touching the HTML/JS just to deduplicate two rules.** Comma-joined selectors solve it without a markup edit.
- **Keep defensive `!important` and other host-page guards.** They are usually load-bearing against page-CSS conflicts; removing them silently breaks things only on certain pages.
- **Keep semantic / debug-friendly class markers** (e.g. `Scholarscope_Appendix_Year`) even if no selector currently targets them. They aid DOM inspection and are nearly free. The drop bar is high: drop only when the class has no CSS rule **and** no JS consumer **and** no semantic value.
- **Verify with `node --check` (or the project's tsc / eslint) after each phase**, not just at the end. A green check after each phase isolates regressions.
- **A passing syntax/type check is not a behaviour test.** State this explicitly to the user; runtime smoke testing in a browser remains required.

## Phase 1 — JS / logic dedup

Walk the file top-to-bottom. For each candidate below, apply the edit if the rule fits — never speculatively.

### 1. Push post-fetch parsing into the fetch helper

If a `fetch`-style helper returns raw text/XML/JSON and **every** caller does the same parse step on it, move the parse into the helper. Return a structured object. Each caller destructures only what it needs.

- Trigger phrase from the user: "the extraction logic after `fooFetch` is duplicated in two places — merge it into `fooFetch`".
- Verify: each caller's `try/catch` still wraps the await; the throw sites (network errors, parse errors) remain inside the same try.
- Counterexample: don't do this if one caller needs the raw text for something else (e.g. logging, hashing, raw passthrough) — keep the helper raw and extract a separate parser.

### 2. Unify two near-identical batch / loop functions

Look for two functions with the same control flow and one or two parameter-shaped differences. Replace with one function taking those differences as parameters; original names can become 2-line wrappers if call sites read better that way.

- Typical shape: `processFooBatch(nodes)` and `processBarBatch(nodes)` both iterate, push entries, await Promise.all, then run the same post-step.
- If the only difference is a boolean (one calls an extra batch step), pass a flag — `processBatch(nodes, injectFn, withCitation)`.

### 3. Unify factory functions that only differ in flags

Two `make*Badge` / `make*Element` functions whose only difference is "add this class + this title + this click handler" → one factory taking an options object.

### 4. Inline single-use trivial wrappers

A function called from exactly one site whose body is one statement → inline at the call site and delete the function. Don't inline functions called from ≥ 2 sites, even if trivial — naming has value.

### 5. Hoist repeated DOM selector strings

If a CSS selector string (or any string literal) is duplicated in ≥ 3 sites within the same file, hoist it to a module-level `const`. Pick a name that reflects intent, not contents.

- Typical case: `".search-results-chunk .full-docsum, #gs_res_ccl_mid .full-docsum"` repeated in `applyFilter`, `applySorting`, `selectShownDocsums` → `DOCSUM_SELECTOR`.

### 6. Tiny helpers for repeated 2-line patterns

A repeated `el.textContent = X; el.style.backgroundColor = Y;` across multiple branches → `setBadge(el, text, color)`. Helper bodies should fit on 2–3 lines; if more, the abstraction is probably wrong.

### 7. Replace literal-id reads with a loop

A block that reads four (or more) elements by hard-coded numeric id (`Quartile1Input`, `Quartile2Input`, …) → `forEach((k, i) => …)` over the index array. Often pairs with a similar build-side loop already present in the file.

### 8. Combine same-source extractors

Two extractor functions that both query the **same** selector and return different fields → one extractor returning an object. Saves one `querySelector` per call and one selector-literal duplication.

### 9. Cache repeated DOM lookups within a scope

If a function calls `docsum.querySelector(".docsum-pmid")` once for the text content and again for the parent-node insertion site, query once at the top and pass the element through.

## Phase 2 — CSS audit

Run the JS pass first; the CSS surface only stabilizes after dead JS class hooks are gone.

### 1. Map every class to its consumer

Grep the file for `Scholarscope_` (or whatever the project prefix is). Build a mental table: for each class, mark whether it has a **CSS rule**, a **JS classList consumer** (querySelector, contains, JS state branching), or just a **debug/semantic marker**.

### 2. Drop dead class hooks

A class that exists only as a string in a `class:` attribute, with **no CSS rule** and **no JS consumer**, is dead. Remove it from the class list. Don't touch classes that exist only to be visible in DevTools — those have value as a debugging aid (see operating constraint above).

### 3. Drop dead id hooks

Same logic as classes: an `id="..."` attribute that is never queried by JS and never appears in CSS is dead. Remove the `id` attribute.

### 4. Collapse always-paired classes

If two classes are always added together and always removed together — one as a JS state marker, the other as the CSS rule hook — collapse to one class. Move the CSS rule onto the surviving name. Update every `classList.add` / `classList.remove` site, including any `STATE_CLASSES` constant.

### 5. ID-vs-class selector audit

ID selectors should target either a singleton element you control (e.g. a modal root) or an external page anchor (e.g. `#gs_ab` from Scholar). Don't use ID selectors for elements that could become non-singleton, and don't use class selectors where ID matches the singleton intent. Most existing files are fine; flag deviations.

### 6. Merge rules with identical declarations

Two rules with the same declaration block → comma-joined selector with one block. Don't split selectors across lines unless the line gets very long.

```css
/* before */
.A { background: #0071BC }
.B { background: #0071BC }
.A:hover { background: #20558A }
.B:hover { background: #20558A }

/* after */
.A, .B { background: #0071BC }
.A:hover, .B:hover { background: #20558A }
```

### 7. Extract a shared declaration when only one or two properties differ

If two rules share most of their declarations and differ in one (e.g. `width`), extract the common bit into a comma-joined rule, then per-element overrides for the differing property.

```css
/* before */
.Input { width: 50px; padding: 4px; border: 1px solid #bbb }
.Control { width: 100%; padding: 4px; border: 1px solid #bbb; box-sizing: border-box }

/* after */
.Input, .Control { padding: 4px; border: 1px solid #bbb }
.Input { width: 50px }
.Control { width: 100%; box-sizing: border-box }
```

Don't do this when the differing properties dominate — readability wins.

### 8. Merge `:before` icon dimension rules

Pseudo-element rules often share `width/height/vertical-align/margin-right` and only differ in `content`. Comma-join the layout part, leave per-element `content`.

### 9. Skip merges that hurt readability

If two rules look similar but represent different concerns (e.g. `.FilterValueInputFrame` vs `.FilterQuartileInputFrame` differ in `width`, `border-left`, `padding-left`, `margin-top` — only `width` is "shared" and the merging would obscure the design intent), leave them.

### 10. Don't introduce utility classes for one or two reuses

Adding `.bg-primary` and applying it across HTML/JS to dedup 4 lines of CSS isn't worth the markup churn. Prefer comma-joined selectors. Utility classes only pay off at higher reuse counts (≥ 5 sites in a file with no build pipeline).

## Verification protocol

After each phase:

1. **Syntax check**: `node --check <file>` for plain JS userscripts. For TypeScript/TSX projects use the project's existing `tsc` / lint commands. Silence = pass.
2. **Grep for orphans**: search for any function name, class name, or constant name you removed; confirm zero remaining references.
3. **Line-count delta**: report before/after to the user as a sanity-check that work happened, but don't optimize for line count specifically.
4. **Tell the user runtime testing is still required.** A clean syntax check means the file parses; it doesn't mean the badges still render or the fetch still resolves. Enumerate the smoke-test paths (e.g. "open a PubMed search page, confirm badges appear; expand an abstract; click a NotFound badge; submit the manual lookup form") so the user can step through them.

## What this skill is NOT for

- Multi-file refactors with cross-module impact.
- Build-tool, bundler, or framework migrations (Webpack→Vite, JS→TS, React class→hooks).
- Backend or API behavior changes.
- Renames that touch external consumers (the file is the boundary; changes inside must not require coordinated updates elsewhere).
- Adding tests, lint configs, or CI workflows.
- Any task where the user wants new features rather than cleanup.

If the user asks for any of the above, decline this skill and either propose a different approach or hand back to the general planning workflow.

## Workflow when invoked

1. **Read the file end-to-end.** Don't dedup what you haven't seen. For files >2000 lines use ranged reads but cover the full surface before editing.
2. **Plan, then confirm.** List the candidate edits as a numbered checklist (Phase 1 + Phase 2 candidates) and confirm scope with the user before touching code, especially if the file ships to users (userscript) or is on a shared branch.
3. **Edit in small, named chunks.** One edit per todo. Mark each todo complete immediately after applying it. Don't batch.
4. **Run `node --check` after Phase 1 and again after Phase 2.** Don't wait until the end.
5. **Final report**: list each change with old/new line references and a one-line "why". Note unverified runtime behavior explicitly.
