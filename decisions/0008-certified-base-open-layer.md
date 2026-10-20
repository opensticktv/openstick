# 0008. Certified Android TV base with an open layer

Status: accepted · Date: 2026-10-20

## Context

OpenStick must play Netflix, HBO Max, Disney+ and Prime in 4K. Those services only send 4K to devices with Widevine L1 and their own certification. L1 requires a locked bootloader and factory-provisioned keys, so no open or user-flashable device can qualify.

## Options considered

1. Certified base from a manufacturing partner, with OpenStick's open layer on top.
2. Two editions: open Linux device (720p) plus a certified edition.
3. Stay fully open, no 4K.
4. Apps only, for existing certified devices.

## Decision

Option 1. A manufacturing partner supplies a certified Android TV / Google TV platform. OpenStick supplies the launcher, services, integrations, casting receivers, enclosure and accessories, and publishes all of those. The bootloader, OS core, DRM and Google services are closed.

## Consequences

- Supersedes 0002 (Chrome for web apps), 0003 (image build system), 0004 (stick or 4K), 0006 (OTA system) and 0007 (daemon language). 0005 (launcher stack) is resolved as Kotlin with Compose for TV.
- No build-it-yourself kits; devices are sold finished.
- Google Cast becomes available.
- Privacy claims must be limited to OpenStick's own software.
- The fully open design is kept in `docs/PRD.open-linux-edition.md` in case a separate open edition is wanted later.
