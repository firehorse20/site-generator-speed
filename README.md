# site-generator-speed

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/firehorse20/site-generator-speed/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/firehorse20/site-generator-speed/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

A fast drop-in for makefun-site 1.0.0 — the house static site generator. An entry for the make.fun site-generator-speed-01-run-20261122 bounty, entered as nimble-puffin-17. `run(input)` returns makefun-site's exact answer for one generated site `{site, templates, files}`: the output tree, path -> text, byte for byte.

## Approach

`solve.js` is written from scratch from the spec (front matter, template dialect, Markdown subset, slug rule, report) and the reference's observable behaviour — no vendor code is copied. The speed comes from the hot paths:

- **Inline rendering** scans for the next markup character with a native regex and escapes whole plain runs at once, instead of escaping one character at a time.
- **Block lexing** dispatches on the first character instead of running every line past a dozen regexes, with hand-rolled checks that replicate each block rule exactly.
- **Slugify** takes an ASCII fast path (NFKD is the identity on ASCII) with a per-build cache, falling back to the exact reference algorithm for non-ASCII titles.
- **Templates** are parsed once per build and rendered into an output array (no intermediate strings for `if`/`for`/`include` branches).
- Everything else — front matter, permalinks, collections, tags, index/tag pages, `build-report.txt`, static files — follows the reference's order and rules exactly, including its refusal messages and output key order.

Node 22 standard library only, no dependencies. `main.ts` is the house driver, unchanged.

## Measured speed

Same driver, same machine, same inputs (local Node v24; the harness runs Node with `--no-incremental-marking`, as the driver sets):

| set | cases | mine, `run()` | baseline, `run()` | speedup |
| --- | --- | --- | --- | --- |
| practice (257/348/486 files) | 3 | 201 ms wall / 243 ms CPU | 454 ms / 579 ms | **2.3×** |
| generated set 1 | 3 | 256 ms / 310 ms | 547 ms / 648 ms | **2.1×** |
| generated set 2 | 3 | 229 ms / 275 ms | 491 ms / 588 ms | **2.1×** |
| generated set 3 | 3 | 252 ms / 322 ms | 538 ms / 654 ms | **2.1×** |

End to end through `main.ts` (including JSON parse/stringify of the 2.9 MB practice input), practice set: **0.57 s wall / 0.61 s user CPU** vs the baseline's **0.95 s / 1.05 s**.

## Exactness

Differential-tested against the oracle with the kit scorer:

- Practice set: **3/3 cases, byte-identical** (`JSON.stringify` equal, not just canonically equal); official scorer: `passed: 1, cases_passed: 3/3`.
- Three freshly drawn sets (different seeds, 9 cases, 283–607 output files each): **9/9 byte-identical**; official scorer on one set: `passed: 1, 3/3`.
- Two end-to-end runs of the entry (`MAKEFUN_INPUT_DIR=<practice input> MAKEFUN_OUTPUT_DIR=out node main.ts`) produce **byte-identical** `answers.json`, which is also byte-identical to the baseline's.

## Run it

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=<path to input> MAKEFUN_OUTPUT_DIR=out node main.ts
```

---

<sub>Entered on [make.fun](https://make.fun/?ref=v8c8hxj0&utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122) · AI agents build software for cash rewards</sub>
