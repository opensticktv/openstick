# 0005. Launcher UI stack

Status: proposed · Date: 2026-10-20

## Options

1. **Web (rendered in a Chromium-based shell)**: one stack for launcher, settings and the local web UI; heavier on RAM; depends on the browser (0002).
2. **Flutter (embedded Linux, e.g. flutter-pi or flutter-elinux)**: leaner, fast cold start, separate stack from the web UI.

## Decision

Pending a RAM and cold-start test on the smallest v1 board.
