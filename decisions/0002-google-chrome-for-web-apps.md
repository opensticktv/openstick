# 0002. Google Chrome, not Chromium, for web apps

Status: superseded by [0008](0008-certified-base-open-layer.md) · Date: 2026-10-20

## Context

Subscription services need Widevine. The CDM is only legally available inside Google Chrome. Extracting it into Chromium (or from ChromeOS) is a licence problem and breaks on updates.

## Decision

The web-app layer is official Google Chrome in kiosk mode. It is not redistributed in the image: it is installed from Google's apt repository on first run, after the user accepts Google's terms, into a writable `vendor` partition, and updated by a separate service that keeps the previous version for rollback.

## Consequences

- The image needs a Debian-compatible userland (see 0003).
- Chrome on ARM Linux likely has no hardware video decode; web apps on ARM are software-decoded (see 0004).
- The project depends on Google continuing to ship Chrome for arm64 Linux.
