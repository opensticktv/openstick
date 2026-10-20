# 0001. Repository layout

Status: accepted · Date: 2026-10-20

## Context

One person is building hardware, an OS, a daemon, a UI, a website and a Home Assistant integration. These have different licences, toolchains and release cadences.

## Decision

Separate repos under the `opensticktv` org: `openstick` (planning), `os`, `openstickd`, `launcher`, `hardware`, `website`, `ha-openstick`, plus `.github` for org defaults. `ha-openstick` must be its own repo because HACS installs integrations per repository.

## Consequences

Licences stay clean per repo (Apache-2.0 code, CERN-OHL-S-2.0 hardware, CC BY 4.0 docs). Cross-repo changes need linked issues. Revisit a monorepo for `os`, `openstickd` and `launcher` if they start changing in lockstep.
