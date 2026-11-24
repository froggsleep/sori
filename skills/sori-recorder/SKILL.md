---
name: sori-recorder
description: Operate the installed sori macOS recorder and hand a verified recording to a user-approved follow-up workflow. Use when the user asks to start or stop recording, check status or audio levels, choose a microphone, list or verify recordings, or continue work from a sori recording. This skill does not transcribe audio or change sori source code.
---

# sori recorder

Use sori's local CLI and request JSON output when reading results programmatically.

## Boundaries

- A direct request to record is authorization to start. Do not start from calendar context, ambient conversation, or an inferred meeting.
- The user may authorize recording and a named follow-up action in the same request. Do not infer an action they did not name.
- Uploading, sharing, attaching, or transcribing audio requires an explicit request for that action.
- If a follow-up tool needs to upload audio, name the service and get approval before sending the files unless the user already approved that service.
- Do not change macOS privacy permissions, switch microphones, or delete recordings unless the user asks.
- Keep exact error messages. Do not report success before the CLI returns `ok: true`.
- Never create or edit `~/sori/consent-notice.json`. If start reports that the recording notice is not accepted, ask the user to open sori and accept “Ask before you record” themselves, in the setup window (… → Permissions…) or by clicking Record. Acceptance does not replace permission to record each conversation.

## Start and monitor

Check state before starting:

```sh
sori status --json
```

If sori is already recording, report the active folder instead of starting another recording. Otherwise run:

```sh
sori start --json
```

Keep the returned folder path. While recording, use `sori status --json` for elapsed time, selected devices, and audio levels. If the microphone is silent, report the selected device without changing it.

If start times out, run `sori status --json` before retrying. A late start must not create a second recording.

## Stop and verify

Run:

```sh
sori stop --json
```

Wait for `ok: true`. The stop response means both WAV files and `mixed.m4a` were finalized. Read `meta.json` in the returned folder and verify that `status` is `done`, then report the folder and duration.

Do not open or process `mic.wav`, `system.wav`, or `mixed.m4a` unless the user requested work on the audio.

## Continue from a completed recording

If the user already requested a follow-up action, hand the verified recording folder to the relevant available skill or tool. Keep the work local when the user requested local processing. sori does not provide transcription itself, so do not claim that this skill can transcribe or summarize audio.

If the user did not specify what happens next, offer these options once:

- Leave the recording untouched.
- Transcribe it.
- Summarize it or extract decisions and action items.
- Export or share it with a destination the user names.

Wait for the user's choice before opening either WAV file. Apply the choice only to the recording folder you just reported or one the user explicitly selects. If transcription is requested without a method, use an available local tool. If only an external service is available, name it and ask before uploading.

## Find recordings and choose a microphone

Use:

```sh
sori list --json
sori devices --json
```

Read preferences with `sori settings --json`. Only when the user asks, use `sori set-mixed-file on --json` or `sori set-mixed-file off --json`. This applies to the next recording and keeps both original WAV files. MCP equivalents are `sori_settings` and `sori_set_mixed_file` with a required boolean `enabled`.

Use `sori set-mic <NAME> --json` only after the user chooses that device. Use `sori set-mic auto --json` when the user wants sori to follow the default physical microphone.

Recordings are under `~/sori/recordings`. Each folder contains `mic.wav`, `system.wav`, `mixed.m4a` (both tracks in one file; `meta.json` lists it under `tracks.mixed` when present), and `meta.json`. If a command fails, check `sori status --json` before another state-changing command.

### Saving and combined audio

`stopping` means capture is being stopped and files are being finalized. Wait for completion; do not start another recording or report it saved yet. Combined audio creation can take up to five minutes. `sori stop --json` and `sori list --json` include `tracks`, `mixed_status`, `dropped`, and `warnings`. `dropped` gives the milliseconds of audio lost per track (`mic_ms`, `system_ms`); the warning `audio_dropped` means one second or more was lost, so tell the user. `mixed_status: failed` with `status: done` means the original WAV files were saved but the combined file failed. Report both facts. `not_created` covers disabled mixing and older recordings. If a client times out, check status and the matching recording folder in the list before retrying.
