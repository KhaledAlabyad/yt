# yt

A modular Python YouTube downloader built on top of [yt-dlp](https://github.com/yt-dlp/yt-dlp). Instead of one monolithic script, it's a small pipeline of single-purpose tools wired together by a glue script (`ytdl`).

## How it works

```
URL
 │
 ▼
yt-clean-url      strips tracking params & playlist noise  →  clean URL
 │
 ▼
yt-info           fetches metadata via yt-dlp              →  JSON
 │
 ├─ playlist? ──► yt-select   interactive picker (stderr/tty)  →  "1,3,5-8"
 │                                                                   │
 │                                                                   ▼
 └─ video? ──────────────────────────────────────────────────────────┐
                                                                     ▼
                                                           yt-download   →  JSON {output_dir, files}
                                                                     │
                                                                     ▼
                                                           yt-post       interactive open/browse menu
```

**Design rules:**

- Interactive prompts go to **stderr**; data goes to **stdout** — pipes stay clean.
- Every `yt-*` script is independently useful on the command line.
- `ytdl` is pure glue: no business logic, just wires the pipeline.
- `yt-download` avoids double-fetching metadata when title is already known (playlist branch).

## Prerequisites

- Python 3.10+
- [yt-dlp](https://github.com/yt-dlp/yt-dlp) installed and on your `PATH`

```bash
pip install yt-dlp
```

## Installation

Copy all scripts to somewhere on your `PATH`:

```bash
cp yt-clean-url yt-info yt-select yt-download yt-post ytdl ~/.local/bin/
```

Or clone and symlink:

```bash
git clone https://github.com/KhaledAlabyad/yt.git
ln -s "$PWD/yt/ytdl" ~/.local/bin/ytdl
# repeat for any yt-* tools you want standalone
```

## Usage

```bash
ytdl 'https://www.youtube.com/watch?v=VIDEO_ID'
ytdl 'https://www.youtube.com/playlist?list=PLAYLIST_ID'
```

> **Tip:** always quote the URL so `&` isn't interpreted by the shell as a background operator.

### What happens

1. The URL is sanitized by `yt-clean-url`.
2. Metadata is fetched silently.
3. **Playlist:** an interactive picker lets you choose which videos to download (ranges like `1,3,5-8` are supported). Already-archived videos are marked and skipped by default.
4. **Single video:** if it's already in the archive you're prompted before re-downloading.
5. Downloads are saved under `~/Downloads/yt-dlp/<channel or playlist title>/`.
6. A post-download menu lets you open the file or browse the folder.

## Modules

| Script | Role |
|---|---|
| `ytdl` | Pipeline glue; entry point |
| `yt-clean-url` | Strips tracking params, normalises the URL |
| `yt-info` | Fetches metadata via yt-dlp, outputs JSON |
| `yt-select` | Interactive playlist item picker; reads from tty, writes selection to stdout |
| `yt-download` | Calls yt-dlp to do the actual download; outputs JSON with result paths |
| `yt-post` | Post-download menu (open file, open folder, do nothing) |

## Archive

Downloaded video IDs are recorded in `~/Downloads/yt-dlp/archive.txt` (yt-dlp's standard archive format). This prevents duplicate downloads across sessions.

## License

MIT
