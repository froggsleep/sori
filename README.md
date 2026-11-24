# sori

[![skills.sh](https://skills.sh/b/froggsleep/sori)](https://skills.sh/froggsleep/sori/sori-recorder)

sori is a meeting recorder for the Mac menu bar. It records your microphone and the other side of the call into separate files and also saves a mixed file. Recordings stay on your Mac. There is no account, cloud service, or telemetry.

Download it from [reasonar.ai/sori](https://reasonar.ai/sori). It's free, needs macOS 14.4 or later on Apple silicon, and updates itself. Release notes are at [reasonar.ai/sori/releases](https://reasonar.ai/sori/releases).

This repository holds the agent skill and is the place to report problems. The app's source code is not published here.

## Command line

Install the `sori` command from the menu bar settings, then use:

```sh
sori start
sori stop
sori status
sori devices
sori list
```

Add `--json` to any command for machine-readable output. `sori stop` returns after both WAV files, `mixed.m4a`, and `meta.json` are written.

Each recording gets its own folder:

```text
~/sori/recordings/2026-09-02-1423/
  mic.wav
  system.wav
  mixed.m4a
  meta.json
```

If `~/sori/on-finish` exists and is executable, sori runs it with the finished recording folder as its first argument.

## Use sori with an AI agent

### MCP

`sori mcp` runs an MCP server. Open sori's setup window and pick your app under **Add to** on the MCP row: Claude Desktop, Claude Code, Codex (the ChatGPT app uses it through Codex), or Cursor. Each app asks you to confirm, and sori never edits its settings files. For any other app, Copy Config for Other Apps copies this block:

```json
{
  "mcpServers": {
    "sori": {
      "args": ["mcp"],
      "command": "/Applications/sori.app/Contents/MacOS/sori-cli"
    }
  }
}
```

`sori mcp config` prints the same steps with the path on your Mac.

### Skill

Agents that can run shell commands can use the `sori` command through the recorder skill. After installing the app and the command, add the skill:

```sh
npx skills add froggsleep/sori --skill sori-recorder
```

The skill is in [`skills/sori-recorder/SKILL.md`](skills/sori-recorder/SKILL.md). An agent can use it to start and stop recording, check audio levels, verify finished files, and ask what to do next. sori does not transcribe. That has to come from another tool your agent has.

An agent should start recording only when you ask. For example:

> Record this meeting. When I ask you to stop, keep the files local and ask whether I want a transcript or action items.

## Problems

Open an issue here or email froggsleep@gmail.com. The log at `~/sori/sori.log` (Open Log in the menu) helps. It contains no audio, but it does list device names and paths in your home folder, so read it before you attach it.

## License

The skill and the documents in this repository are MIT licensed. The sori app is free to use.
