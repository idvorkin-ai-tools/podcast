# Igor's Podcast

AI-narrated audio of [Igor's essays](https://idvork.in). Charon voice via Gemini 3.1 Flash TTS.

## Subscribe

Paste this into Overcast / Apple Podcasts / Pocket Casts / Castro / Castamatic:

```
https://idvorkin-ai-tools.github.io/podcast/feed.xml
```

Or use the one-tap subscribe link: [`podcast://idvorkin-ai-tools.github.io/podcast/feed.xml`](podcast://idvorkin-ai-tools.github.io/podcast/feed.xml)

## Repo layout

```
podcast/
├── feed.xml                                       # RSS 2.0 + iTunes + Podcasting 2.0 namespaces
├── episodes/
│   ├── 001-seven-habits.mp3                       # The audio (with embedded ID3 CHAP frames)
│   └── 001-seven-habits.chapters.json             # Podcasting 2.0 chapter sidecar
├── covers/                                        # Episode + show cover art
└── .nojekyll                                      # Serve files as-is via GH Pages (no Jekyll processing)
```

## Episodes

| # | Title | Duration | Source post |
|---|---|---|---|
| 001 | The 7 Habits of Highly Effective People — Igor's Take | 2:13:03 | [idvork.in/7h-concepts](https://idvork.in/7h-concepts) |

## Production pipeline

1. **Compose** — concatenate the blog-post chapter files, strip Jekyll frontmatter + HTML, add `[short pause]` / `[long pause]` prosody tags between sections.
2. **Chunk** — split the script into ~400-word chunks for Gemini TTS (per-call latency budget).
3. **Generate** — `~/gits/chop-conventions/skills/gen-tts/generate-tts.py batch` with Charon voice, 4-6 parallel workers.
4. **Paraphrase failures** — Gemini's RECITATION filter trips on verbatim quoted passages. Rewrite the failing chunks in your own words and retry.
5. **Concat + encode** — ffmpeg concat demuxer → libmp3lame, mono 64 kbps, 24 kHz. Insert 200 ms silence between chunks.
6. **Chapter markers** — write ID3 CHAP + CTOC frames via mutagen. Write companion `chapters.json` sidecar (Podcasting 2.0 spec).
7. **Ship** — drop into `episodes/`, add a new `<item>` to `feed.xml`, push.

## Why a separate repo

The audio + chapter sidecars + feed XML want to live together. Was originally split across `idvorkin/blob` (audio) and `idvorkin/idvorkin.github.io` (feed) but that fragmented the change history and made episode adds touch two PRs. One repo = one merge.
