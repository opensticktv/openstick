# 0006. OTA update system

Status: superseded by [0008](0008-certified-base-open-layer.md) · Date: 2026-10-20

## Options

1. **RAUC**: open source, A/B slots, signed bundles, no server required (any HTTPS host), widely used in embedded Linux.
2. **Mender**: open-source client, richer server; hosted server is a paid service.

## Decision

Pending. Lean: RAUC with a static HTTPS update host, which fits the continuity plan (nothing depends on a private server).
