# memwatch

A `btop`-style memory-pressure monitor for macOS, written in pure zsh. No
dependencies beyond what ships with macOS — no Homebrew, no Python, no ncurses.

Designed to run in a tmux window on a headless box and be watched over SSH from
a phone terminal (tested with [Blink Shell](https://blink.sh) on iOS).

```
 ◆ MEMORY ──────────────────────────────────────────────── Tue 20 Oct 13:21:55

 PRESSURE  23% free    [████████████░░░░] 23-23 avg 23
                                                   ⣀⡀                ⢀⣀
                                                   ⣤⣤ ⠀⠀ ⢀⣠⣤⣶⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿
 SWAP      1.5/3.0G    [████████░░░░░░░░] 50-50%
                                                   ⣤⣤ ⢀⣀⣤⣶⣿⣿⣿⣿
 last 4s · 2s/sample · load 7.21 4.24 4.19

 TOP RSS
  1  14.4G  Python
  2   1.9G  vmware-vmx
  ...
```

## What it shows

- **PRESSURE** — macOS kernel pressure percentage, the same metric Activity
  Monitor plots as "Memory Pressure". The gauge shows **% free** (higher =
  safer); the chart and bar plot **pressure** (100 − free) and grow as memory
  pressure grows, btop-meter style.
  - colour follows pressure **growth**, Activity-Monitor style, with smooth
    interpolation (no discrete states). Slope over a responsive 6s window:
    green ≤15%/min, amber 15–250, red onset ~250 (25 pressure-points in 6s),
    full red at ~500 ("50% in 3s") — red means explosive growth only
  - half-window lag compensation: the colour series is shifted 2 samples
    forward and the right edge additionally reads a fast 4s slope, so colour
    centres on the shape it describes instead of lagging ~6s behind it
  - floor overrides: free <10% forces amber, free <5% forces red
- **SWAP** — swap usage as % of total swap size, with used/total in GB
  (colour = swap used as % of **total RAM**, since swap capacity is dynamic:
  3G on a 128G machine is ~2% and calm; amber ~6% of RAM, red ~13%).
- **TOP RSS** — the five largest processes by resident memory.

History is sampled every 2 s and rendered as multi-row [braille](https://en.wikipedia.org/wiki/Braille_Patterns)
charts (4 rows for pressure, 2 for swap), the same dot technique `btop` uses —
4×2 resolution per terminal cell. Charts auto-shrink on short terminals and the
layout re-wraps live on resize (SIGWINCH-safe: width is re-read every frame
from `stty`, because `tput` lies inside tmux `screen`-family TERMs).

## Install

```sh
curl -o ~/.local/bin/memwatch https://raw.githubusercontent.com/oliverlavery/memwatch/main/memwatch
chmod +x ~/.local/bin/memwatch
```

Requirements: macOS, zsh 5.x. Runs anywhere tmux runs; only the metrics are
macOS-specific (`memory_pressure`, `vm.swapusage`, `vm.loadavg`).

## Usage

```sh
memwatch            # run it
MW_COLS=56 memwatch # force width (useful for testing phone layouts)
MW_ASCII=1 memwatch # plain-ASCII sparklines, for terminals whose font lacks braille
MW_SMOOTH=0 memwatch# disable the display smoothing (EMA) on chart geometry
```

### tmux

```sh
tmux new-window -n mem '/full/path/to/memwatch'
```

Use the absolute path — tmux panes don't inherit your interactive PATH.

## Why raw bytes matter

A subtle portability lesson baked into this script: zsh's `printf "\uXXXX"`
only works when the runtime locale is UTF-8 — under `LC_CTYPE=C` it fails with
`character not in range` and your precious glyphs are silently mangled or
dropped. memwatch generates braille as raw UTF-8 bytes (`\xe2\xa0..`), so it
renders identically under any locale. If a chart looks empty, it's locale
corruption, not your font — and `MW_ASCII=1` is the escape hatch.

## License

MIT — see [LICENSE](LICENSE).
