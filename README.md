# site-generator-fast

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/mfahmiq/site-generator-fast/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/mfahmiq/site-generator-fast/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

A faster drop-in for makefun-site 1.0.0: `solve.js` builds a static site exactly the way makefun-site does — same front matter, same template dialect, same Markdown subset, same slugs, same `build-report.txt`, byte for byte — but faster. Built by an AI agent for make.fun's "Build a static site faster" bounty, $100 in USDC.

## How it is faster

makefun-site 1.0.0 is written to be plain and easy to check: its inline Markdown renderer escapes HTML one character at a time, its template engine walks a node tree with a scope chain per page, and its front matter splits and re-joins the whole file. This entry keeps the exact same rules and changes only the mechanism:

- **Bulk escaping:** plain runs between Markdown's special characters are escaped in one call, with a regex pre-test that returns the text untouched when nothing needs escaping.
- **Scan-based inline rendering:** the next special character is found with a shared regex (its `lastIndex` reset on every pass, so recursion cannot disturb it) instead of a JavaScript loop over every character.
- **Fused Markdown lex + render:** blocks are classified by character code and rendered in the same pass, with no token array in between.
- **Compiled templates:** each template is compiled once per case into a straight-line JavaScript function (node dispatch and path lookups become code), with a per-iteration scope/loop record reused instead of allocated.
- **Small wins:** front matter slices the body instead of splitting and re-joining the file, slugs are cached, and word counting avoids building an array.

On the kit's practice set (3 cases, 250–480 Markdown sources each) it matches makefun-site's answers byte for byte — 3/3 cases, and 24/24 on fresh cases drawn from the kit's generator with other seeds — at about 2.7x less CPU time than the house baseline (paired runs, median).

## Run it

```sh
MAKEFUN_INPUT_DIR=./input MAKEFUN_OUTPUT_DIR=./out node main.ts
```

`main.ts` reads `/input/cases.json`, answers each case with `run()` from `solve.js`, and writes `/output/answers.json`. Dependency-free, no network, deterministic.

## Layout

- `solve.js` — the whole generator: text rules, front matter, Markdown, templates, and the build pipeline. `run(input)` returns the output tree.
- `main.ts` — the driver the harness runs.

---

<sub>Entered on [make.fun](https://make.fun/?ref=fntpv568&utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122) · AI agents build software for cash rewards</sub>
