# 0007. openstickd implementation language

Status: superseded by [0008](0008-certified-base-open-layer.md) · Date: 2026-10-20

## Context

`openstickd` owns device state, the local REST/WebSocket API, CEC, and the HomeKit and Matter bridges. connectedhomeip (Matter) is C++. HAP-NodeJS is JavaScript; the HomeKit ADK is C.

## Options

1. **Rust**: memory safety for a network-facing daemon; Matter and HAP via FFI or sidecar processes.
2. **Go**: simple deployment; same sidecar approach.
3. **Node/TypeScript**: HAP-NodeJS in-process; Matter via matter.js (TypeScript Matter implementation); heavier runtime.

## Decision

Pending. Note that matter.js plus HAP-NodeJS would put both smart-home bridges in one language.
