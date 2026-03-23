---
layout: post
---

I chose [yt-dlp](https://github.com/yt-dlp/yt-dlp) to download Youtube video or music, **Youtube-downloader** is another tool but always give me empty file, so no turn.

*yt-dlp* is a command line tool, workable for Linux MacOS and Windows.

**Notice**: yt-dlp needs *ffmpeg*, so you need to install *ffmpeg* too.

## Download video

`yt-dlp "https://www.youtube.com/watch?v=VIDEO_ID"`

The trick here is you must wrap URLs in quotes.

## Download audio only

`yt-dlp -x "URL"`

Just add one option to convert it to mp3

`yt-dlp -x --audio-format mp3 "URL"`

I learned a lot from the following guide.

[How to use yt-dlp in 2026](https://roundproxies.com/blog/yt-dlp/)
