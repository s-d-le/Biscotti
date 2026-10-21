---
status: draft
---

# Architecture: Audio Import

Changes are additive and bottom-up through the existing layers.

## DataStore
- `AudioRole`: add `case imported` (stored as raw string; no migration needed).
- `audioPaths(meetingID:)` returns a new `MeetingAudio` value: `.pair(mic:system:)` or
  `.single(URL)`. Gate on `isPresent` as today. Update `TranscriptionService` accordingly.
- `createImportedMeeting(startDate:duration:)`-style helper (or reuse `createMeeting` +
  `attachAudio` + `markAudioPresence`, as `RecordingController` does).

## Transcription package
- `XPCProcessRequest.systemPath` becomes `String?` (Codable: absent key decodes to nil,
  so old/new sides stay compatible).
- `TranscriptionEngine.processAudio(micPath:systemPath:…)`, `Transcriber.processAudio(mic:system:…)`,
  `XPCEngineAdapter`, and `BiscottiTranscriber.xpc` `main.swift` accept an optional system path.
  For a single file the caller passes it as `micPath` with `systemPath == nil`.
- `AudioLoading.loadAndMerge` loads the system file only when present; `AudioMerger`
  already supports a single non-empty stream (`LabeledRange.label == .mic`).
- `SpeakerAnalyzer` (voiceprint backfill) gets the same optionality.

## TranscriptionService
- `Transcribing.processAudio(mic:system:customVocabulary:)` → `system: URL?`. `FakeTranscriber`
  and tests updated.

## AppCore
- New `AppCore+ImportAudio.swift`:
  `func importAudio(urls: [URL]) async -> [UUID]` — validates (AVFoundation), copies,
  creates meetings, then enqueues one `pendingTranscriptionTask` chain per file
  (sequential). Validation/copy run off the main actor.
- Pure helper `AudioImportValidator` (UTType + AVFoundation probe) behind a protocol seam
  so tests don't need real audio decoding for most cases.

## UI
- `DesignSystem`: new `ImportedAudioPill` (modelled on `SourcePill`), used in Meeting Detail and the list row.
- Read model: `isImported` derived from the audio role, exposed to the list and detail view models.
- `AppShellUI`: toolbar `Import Audio` button, `.fileImporter`, `.onDrop` on the list,
  `File > Import Audio…` command in `BiscottiApp`.
- `MeetingDetailUI`: playback already loads `[mic, system].compactMap` — verified to work
  for a single `.imported` ref; add `.imported` handling where roles are switched on.
- Alerts reuse the existing alert pattern from Settings import.

## Testing
- Unit: validator, `audioPaths` single/pair, XPC request back-compat decode,
  `AudioLoading` single-file path, `importAudio` happy path/failure cleanup
  (fake transcriber), role handling in DataStore.
- Manual-test script entry in `ManualTestApp` for a real mp3/m4a end to end.
