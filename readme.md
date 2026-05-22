Let me map out the data flow first, then write the modules:

```
URL ──► yt-clean-url ──► yt-info ──► yt-select (playlist only) ──► yt-download ──► yt-post
       (sanitize)      (fetch meta)  (interactive pick, stdin)    (core logic)   (open/browse)
```

Each script: stdin/stdout JSON or plain text, `stderr` for interactive prompts so pipes stay clean.All 6 scripts, each doing exactly one thing. Here's the data-flow architecture:

```
URL (argv)
  │
  ▼
yt-clean-url          strips ?list= & tracking params → clean URL (stdout)
  │
  ▼
yt-info               fetches metadata via yt-dlp     → JSON (stdout)
  │
  ├─ playlist? ──► yt-select   interactive picker on stderr/tty → "1,3,5-8" (stdout)
  │                                                                    │
  └─ video? ──────────────────────────-────────────────────────────────┤
                                                                       │
                                                                       ▼
                                                              yt-download        → JSON {output_dir, files} (stdout)
                                                                       │
                                                                       ▼
                                                              yt-post            interactive open/browse menu
```

**Key design decisions:**

- **stderr for prompts, stdout for data** — interactive text goes to stderr so `|` pipes stay clean. This means `yt-select` can be used in a pipeline without contaminating the indices output.

- **Each module is independently useful** — you can run `yt-info <url> | jq .title` or `yt-clean-url <messy-url>` in a shell script without invoking the rest.

- **`ytdl` knows no business logic** — it only wires the pipes, handles the playlist/video branch, and passes data through. All logic lives in the leaf tools.

- **`yt-download` avoids double-fetching** — if `--title` is passed (playlist branch), it uses that for the output dir. Only for bare single-video calls does it re-fetch to get the channel name.

**Install:**
```bash
cp ytdl/yt-* ytdl/ytdl ~/.local/bin/
```
