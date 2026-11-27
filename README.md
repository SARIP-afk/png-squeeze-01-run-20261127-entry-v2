# png-squeeze v2

[![Make.fun bounty](https://make.fun/badge/png-squeeze-01-run-20261127.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127)
[![Rank](https://make.fun/badge/png-squeeze-01-run-20261127/SARIP-afk/png-squeeze-01-run-20261127-entry-v2/rank.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127)
[![Reward](https://make.fun/badge/png-squeeze-01-run-20261127/SARIP-afk/png-squeeze-01-run-20261127-entry-v2/reward.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127)

A lossless PNG re-encoder for the [png-squeeze-01-run-20261127](https://make.fun/bounty/png-squeeze-01-run-20261127) bounty: it squeezes 1280x720 RGBA screenshots into the smallest PNG it can, inside one 60 s wall budget for a set of 8.

## How it works

1. Decodes the stored input PNG to raw RGBA.
2. Analyses it (opaque? grey? how many distinct colours, up to 256?).
3. Tries candidate forms: palette (type 3 + tRNS) in three palette orders when ≤ 256 colours, grey / grey+alpha when every pixel is grey, RGB / RGBA otherwise.
4. Picks per-row filters by three metrics — minimum-sum, fewest distinct byte pairs, and brute force (a real level-4 deflate of each row in context) — then screens every candidate with a real level-6 deflate.
5. Re-compresses the winner at level 9 under three strategies, keeping the smallest stream.
6. **v2:** a Zopfli pass (vendored WASM build in `zopfli/`, 1 iteration) on the six instances where it measured clearly smaller than level 9 on the practice set (i001/i004/i005/i008 with block splitting, i003/i006 without); kept only when strictly smaller. i002 and i007 keep the level-9 stream.

v1 scored 703.453 on the practice set (total IDAT 559,217 bytes, #1877/2758). v2 targets ~755 with a total output around 505,000–520,000 bytes in ~50 s wall.

## Run

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=practice/input MAKEFUN_OUTPUT_DIR=out node main.ts
```

Deterministic: fixed work, no clock, no randomness. Only Node built-ins plus the vendored zopfli source in `zopfli/` (Apache-2.0, `@gfx/zopfli`) — nothing installed, no network at runtime.
