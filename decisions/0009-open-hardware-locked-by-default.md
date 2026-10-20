# 0009. Open hardware, locked by default, unlockable by the owner

Status: accepted · Date: 2026-10-20 · Amends [0008](0008-certified-base-open-layer.md)

## Context

0008 used a partner's closed, certified board. The goal is a device that is open hardware and that anyone can buy with 4K working. 4K certification locks the software and the factory-installed keys, not the board design.

## Decision

- OpenStick designs the mainboard from the chip's certified reference design and publishes it (CERN-OHL-S-2.0), with the enclosure and accessories.
- A design house certifies that board as the OpenStick model. Units are sold finished with factory-provisioned keys and a locked bootloader, so 4K works.
- Owners can unlock their unit from settings (off by default, wipes the device, clear warning) and run any OS. Unlocking drops subscriptions to 720p or less, because the device can no longer prove it's unmodified.
- No spoofing or key extraction, ever.

## Consequences

- Chip choice now depends on the vendor allowing published schematics.
- Certifying a new board costs more and takes longer than a white-label.
- People can build the board from the files; self-built units have no device keys and run at 720p.
