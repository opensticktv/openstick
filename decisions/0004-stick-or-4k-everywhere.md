# 0004. Stick form factor or 4K YouTube everywhere

Status: superseded by [0008](0008-certified-base-open-layer.md) · Date: 2026-10-20

## Context

No v1 candidate gives both a true stick and 4K YouTube. The CM5 stick has HEVC-only hardware decode (no VP9, AV1 or H.264). RK3588 boards decode VP9 and AV1 but Chrome on ARM likely cannot use the decoder. Intel N100-class boxes decode everything and Chrome uses it through VA-API, but are boxes with a cable.

## Options

1. **CM5 stick**: true stick; no 4K YouTube anywhere.
2. **Full RK3588 board**: 4K in Kodi and native players; browser probably software-decoded; not a stick.
3. **Intel N100-class**: 4K in native apps and Chrome; most mature Linux support; not a stick; CEC needs a board with CEC wired or a USB-CEC adapter.
4. Support two: one reference (RK3588 or Intel) plus the CM5 as a compact variant without the 4K YouTube claim.

## Decision

Pending the board spike's eight exit criteria (PRD, Hardware).
