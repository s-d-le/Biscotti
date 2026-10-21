---
status: draft
---

# Implementation Plan: Audio Import

- [ ] **Phase 1 — Single-file transcription.** `AudioRole.imported`, `MeetingAudio`,
  optional `systemPath` through XPC/engine/Transcriber/`Transcribing`; fakes + tests.
- [ ] **Phase 2 — AppCore `importAudio`.** Validator, copy, meeting creation, auto
  transcribe + enhance chain, failure cleanup; tests.
- [ ] **Phase 3 — UI.** Toolbar button, File menu command, drag-and-drop, alerts,
  Meeting Detail role handling.
- [ ] **Phase 4 — Verification & docs.** `make lint test`, manual hardware test with
  mp3/m4a/wav, update `CLAUDE.md` stage notes and the README feature list.
