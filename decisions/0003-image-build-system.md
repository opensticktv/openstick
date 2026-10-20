# 0003. Image build system

Status: superseded by [0008](0008-certified-base-open-layer.md) · Date: 2026-10-20

## Context

The PRD originally chose Buildroot. Google Chrome is a Debian package with Debian library dependencies (0002). Flatpak is also simpler on a mainstream distro.

## Options

1. **debos** (Debian, recipe-based, used by Collabora and Mobian): Debian userland, simple YAML recipes.
2. **mkosi** (systemd project): reproducible, Debian or other distros, good A/B and verity support.
3. **Buildroot**: smallest, most reproducible, but Chrome and Flatpak become a maintenance burden.

## Decision

Pending. Lean: mkosi or debos on Debian. Decide in the v0 phase after building one image that installs Chrome and boots to cage.
