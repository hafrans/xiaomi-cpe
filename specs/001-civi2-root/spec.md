# Feature Specification: Civi 2 temporary Root

**Created**: 2026-11-01

**Status**: Retrospective; temporary Root verified for one boot

**Input**: User requested Root for a Xiaomi Civi 2 without an additional router; Bootloader unlocking is difficult, and the user authorized a Root attempt and compatible management tools.

## User scenarios and acceptance

### US1 — Obtain temporary Root with locked Bootloader (P1)

Given Civi 2 `ziyi` on `OS3.0.5.0.VLLCNXM` and locked BL, when a compatible runtime method succeeds, then an independent `su -c id` returns uid 0 and the BL state is recorded. **Observed:** passed during one boot; see [operation log](../../docs/civi2-root-log.md).

### US2 — Manage and recover the device (P2)

Given a successful Root session, a compatible manager reports the active mode and the agent records installation and failures. **Observed:** ReSukiSU reported working/LKM; first attempt restarted the phone. Root after a later reboot and a recovery procedure have not been tested.

## Requirements

- **FR-001**: Preserve user data and locked BL during the documented attempt; no partitions were flashed.
- **FR-002**: Verify Root independently and record software sources, exact versions, hashes and actual results.
- **FR-003**: Treat Root as temporary and current device state as unknown until rechecked.
- **FR-004**: Record failure and uncertainty rather than claiming device or OpenWrt support from a matching kernel string.

## Success criteria

- **SC-001**: On 2026-11-01 `su -c id` returned uid 0; BL reported locked and SELinux Enforcing.
- **SC-002**: A future agent can reach the exact operation log via AGENTS.md and the project Root skill without loading unrelated research.
- **SC-003**: Reboot persistence, unattended recovery, and CPE routing remain unverified.

## Scope and assumptions

This is a retrospective record, not a claim that a Spec Kit workflow preceded the operation. The user's current device state, backups, and later system updates are unknown; recheck before acting.
