# Video Ingestion (/watch) — operating reference

<!-- Filed by triage 2026-07-03 from Eli's video-skill file dump (inbox item 7, file).
     Each section carries atom provenance. This is a LIVE engine capability. -->

## How it works — frames + a transcript
<!-- atom 19 · item 7 · 2026-07-03 -->
Claude has no video model, so /watch treats a video as two things: frames (extracted
images) + a timestamped transcript. Claude reads them together like a flipbook with a
script — timestamps line up what's on screen with what's said. This captures on-screen
content that transcript-only tools miss, without an expensive video model. (Source: Brad's
"My Claude Code Can INSTANTLY Watch Any Video", ingested 2026-07-03.)

## Pipeline mechanics
<!-- atom 20 · item 7 · 2026-07-03 -->
`/watch <URL>` → download, extract frames, get transcript, analyze together. Two local CLI
tools do the heavy lifting — yt-dlp (downloader, works on basically the whole internet) and
ffmpeg (screenshots every few seconds + clean audio for transcription). No MCP, no
third-party middle service; the skill's setup script auto-installs missing dependencies and
authenticates the transcription API. All of this is installed on the mini as of 2026-07-03.

## Transcription strategy
<!-- atom 21 · item 7 · 2026-07-03 -->
First choice: the platform's free captions (no API call). Fallback for caption-less sources
(raw MP4, Loom, Instagram Reels): Whisper hosted on Groq (our configured provider — key
installed 2026-07-03) or OpenAI. Groq is fast with a generous free tier (~2 hours of
transcription per hour of wall time); the skill's creator ran it daily for 2 weeks without
leaving the free tier.

## Cost profile
<!-- atom 22 · item 7 · 2026-07-03 -->
Frame count scales with length but caps at 100 frames for anything over 30 min — a 30-min
and a 1-hour video cost about the same, roughly ~$1 per run. Platform captions are free (no
Whisper call). Reference points: creator ran 3x in parallel and burned <10% of a session for
5+ hours of video; a 45-min lecture analyzed in under 2 minutes. Fits the engine's
cost-discipline constraint: video ingestion is cheap enough to use freely.

## Capabilities and use cases
<!-- atom 23 · item 7 · 2026-07-03 -->
Works on 1,000+ sites (anything yt-dlp supports) plus local files. `--start`/`--end` plus a
zoom flag allow frame-by-frame extraction on a small window (e.g. 10s of a 2-hour video)
without burning full context. Proven uses: content research (breaking down a video's hook —
visual setup, exact words, pattern interrupt); debugging screen recordings (find the exact
frame a bug starts — Eli can send a Loom instead of describing a bug); auto-watching
competitor videos into structured second-brain notes.

## Install status on the mini
<!-- atom 24 · item 7 · 2026-07-03 · CORRECTED filing (see atom reasoning) -->
Installed on the mini 2026-07-03 with Eli's approval: yt-dlp, ffmpeg, the /watch skill, and
the Groq API key. Provenance note: the source transcript (from Eli's MacBook session)
claimed the skill was "already installed in MOMO's setup" — that was true on the MacBook
but false on the mini at capture time (verified absent 2026-07-03); the mini install
happened afterwards, guard-gated and Eli-approved. Video ingestion is now a live input
channel for the ingestion loop.

## Caveats
<!-- atom 25 · item 7 · 2026-07-03 -->
"Instant" is marketing gloss: expect a couple of minutes of local download + frame
extraction + transcription before frames can be read. Fast and genuinely cheap, but not
immediate — set expectations accordingly when a watch is requested live.
