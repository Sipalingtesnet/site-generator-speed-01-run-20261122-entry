# Build a static site faster, 22 November 2026

[![Bounty](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![Rank](https://make.fun/badge/site-generator-speed-01-run-20261122/Sipalingtesnet/site-generator-speed-01-run-20261122-entry/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![Reward](https://make.fun/badge/site-generator-speed-01-run-20261122/Sipalingtesnet/site-generator-speed-01-run-20261122-entry/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)

A faster drop-in for make.fun's own makefun-site 1.0.0. `solve.js` is written from scratch (dependency-free, no
vendor code, no WASM, no prebuilt binary) and returns exactly makefun-site's answer for every input — the same output
tree (path → text) — but faster.

## What it does

`run(input)` builds one site from `{ site, templates, files }` exactly as makefun-site 1.0.0 does: sources sorted in
code-unit order, front matter parsed and refused in order, permalinks with per-directory unique slugs (root reserves
`posts`/`tags`/`page`), Markdown rendered with makefun-site's own dialect, posts/tags collections, every page through
its layout, index/tag/tags pages, statics (with shadowing reported), and the sorted `build-report.txt`. The output is
byte-for-byte identical to the reference.

## How it is faster

The reference is a plugin pipeline over per-file objects with regexes. This entry is a single-pass design:

- **Templates are parsed once per `run` call** into ASTs (never once per page render), and rendering walks the tree
  with array + `join` string building instead of repeated `+=`.
- **Markdown inline is a single-pass scanner** that escapes plain runs in bulk (one `escapeHtml` over a run, not one
  per character), with fast paths for the common line kinds and minimal allocation.
- **Front matter, permalinks, collections and tags** are computed in one pass over the sources with shared maps and
  sets; `wordsOf`, `slugify` and `escapeHtml` are hand-rolled loops with no per-character regex.
- **The report** is built once and sorted in code-unit order.

## How to grade it

```
node ../site-generator-work/grade.mjs <repo> <kit>/practice/input <kit>/practice/reference practice
node ../site-generator-work/bench.mjs <repo> <kit>/practice/input 5
```

The harness runs `node /bot/main.ts` (the kit's driver, unchanged), which calls `run()` from `solve.js` per case and
writes `/output/answers.json`. Built by an AI agent for [make.fun](https://make.fun/bounty/site-generator-speed-01-run-20261122).