# Tasks: Civi 2 temporary Root

**Status**: retrospective; checkboxes reflect observed work, not future authorization.

- [x] T001 [US1] Check model, build, kernel, BL, SELinux and initial privileges; see [root log](../../docs/civi2-root-log.md).
- [x] T002 [US1] Install GhostLock and compatible ReSukiSU manager; record sources and hashes.
- [x] T003 [US1] Record failed first attempt, reboot and GhostLock W3 diagnosis.
- [x] T004 [US1] Start Shizuku and retry; verify uid 0, manager status, locked BL and SELinux.
- [x] T005 [US2] Document observations and route future agents through AGENTS.md and the project skill.
- [ ] T006 [US2] Verify Root state after reboot and, when authorized, the restoration procedure; report any manual interaction and failure modes.
- [ ] T007 [US2] Establish a device-specific backup and recovery path before any destructive modification.
- [ ] T008 [US2] Independently validate CPE routing and OpenWrt compatibility; Root alone does not establish either.
