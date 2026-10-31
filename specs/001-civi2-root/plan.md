# Implementation Plan: Civi 2 temporary Root

**Date**: 2026-11-01 | **Spec**: [spec.md](spec.md) | **Status**: retrospective

## Summary

The observed path retained Android and locked BL, installed GhostLock plus a KernelSU-derived manager, then used Shizuku after a failed direct attempt. Evidence, sources, hashes, actual sequence and recovery limits are maintained in [the operation log](../../docs/civi2-root-log.md).

## Technical context

Civi 2 `ziyi`; Android 15 / `OS3.0.5.0.VLLCNXM`; kernel `5.10.236-android12-9-00003-gfb24cf99ad97-ab14313284`; ADB over USB. Temporary runtime privilege and device-dependent exploit behavior are not firmware installation or proof of OpenWrt routing.

## Evidence and constraints

- Independent `su -c id` showed uid 0 for one boot; locked BL and enforcing SELinux were checked after success.
- The first attempt rebooted the phone; application diagnosis led to Shizuku for the second attempt. See the log for full details.
- Reboot restoration and data backup were not verified. Recheck device identity and current state; use a known recovery path before any destructive step.
- Project constitution is an unfilled template; no substantive gates can be inferred from it.

## Documentation structure

`AGENTS.md` → `.agents/skills/civi2-root/SKILL.md` → this spec or the operation log, selected by task. The operation log is the source of truth for observed commands and tool versions; [tasks.md](tasks.md) separates completed work from unverified work.
