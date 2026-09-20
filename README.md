# Ghosts-grave-gg-2.0
Python tool that recovers "ghost code" — code that Google Antigravity CLI (agy) claims to have written but that never actually landed in your workspace.

What it does

When agy says "✅ Done! I've updated src/api/handler.ts" but the file is empty, unchanged, or missing — gg finds out where the code actually went.

It searches six places:

· Git worktrees (git worktree list)
· Sandbox directories (/tmp/agy-*, ~/.antigravity/sandbox/)
· Other folders in the same Antigravity Project (settings.json)
· Editor swap files (.swp, .tmp, file~)
· Git dangling blobs (git fsck --lost-found)
· The session transcript itself — if the agent's tool call still contains the code, gg extracts it from there

Requirements

· Pydroid 3 (Android) — free on the Play Store
· Python 3 — Pydroid includes it
· Antigravity CLI installed and used at least once (so there are session logs to scan)
· Optional: git if you want worktree and dangling-blob recovery

How to install

1. Open Pydroid 3
2. Tap the menu (☰) → New file
3. Name it Gg.py
4. Paste the entire contents of the Gg.py file into the editor
5. Save

That's it. No terminal, no chmod, no symlinks needed.

How to run

Option A — Tap the Run button (▶) in Pydroid

This runs Gg.py with no arguments — a read-only scan that prints a report table.

Option B — Run it from Pydroid's terminal

For more control, open Pydroid's terminal tab and type:

```bash
cd /path/to/your/project
python Gg.py
```

Pointing it at a specific project:

```bash
python Gg.py --workspace /path/to/your/project
```

Options

```
--workspace DIR     Primary workspace directory (default: current folder)
--history-dir DIR   Custom Antigravity session/history directory
--json              Output findings as JSON (machine-readable)
--exhume            Recover ghost files into workspace (prompts before overwriting)
--yes, -y           Auto-confirm overwrite prompts during --exhume
--watch             Live monitor — alerts when a claimed write doesn't land
--interval SEC      Polling interval for --watch (default: 2.0)
--help, -h          Show usage
```

Common commands

```bash
python Gg.py                       # scan for ghost code (read-only)
python Gg.py --json                # machine-readable output
python Gg.py --exhume              # recover ghosts (asks before overwriting)
python Gg.py --exhume -y           # recover without prompts
python Gg.py --watch               # live monitor mode
python Gg.py --workspace ~/proj    # scan a specific folder
```

What the report looks like

```
=================================================================================
                       GHOSTS GRAVE (gg) - RECOVERY REPORT
=================================================================================
Workspace: /storage/emulated/0/my-project
Inspected: 12 claimed writes
---------------------------------------------------------------------------------
CLAIMED PATH                             | STATUS         | FOUND AT
---------------------------------------------------------------------------------
src/api/handler.ts                       | ❌ MISSING     | Git Worktree: .agy/wt-7f2/...
src/db/schema.sql                        | ⚠️ STALE       | None
tests/unit/auth.test.ts                  | ✅ OK          | Workspace
=================================================================================

Found 2 ghost file(s) (1 recoverable in graves).
To restore them to your workspace, run: Gg.py --exhume
```

Status meanings:

· ✅ OK — file exists and was modified when agy claimed it was
· ⚠️ STALE — file exists but hasn't changed since before the claim (agent lied)
· ❌ MISSING — file isn't there at all (ghost code)

Recovery

```bash
python Gg.py --exhume
```

Prompts before overwriting anything. If a ghost was found in a worktree, sandbox, or the session transcript, gg writes or copies it back to where agy said it should be.

Important limitation

gg only finds ghosts when Antigravity logged a tool call claiming a write.

· If agy emitted a write_file call and the file never appeared → gg finds it ✅
· If agy just said "Done!" in plain text without any tool call → there's nothing to find ❌

To check which case you're hitting:

```bash
python Gg.py --json
```

· Lists claims → tool calls happened; gg can help
· Empty list → no tool calls were logged; gg can't help with that case

Troubleshooting

"No Antigravity CLI session directory found."
Antigravity hasn't been used on this device yet, or its logs are in an unusual location. Point it manually:

```bash
python Gg.py --history-dir /path/to/your/antigravity/folder
```

"No ghosts found" but you know code is missing.
Either the tool call wasn't logged (see limitation above), or the missing file was never claimed in a session gg can read. Try --history-dir to point at a specific session folder.

Colors look garbled in Pydroid's output pane.
ANSI colors are detected automatically. If Pydroid shows escape codes literally, run with output redirected or ignore it — the report is still readable.

Git worktree / fsck errors.
These are optional features. If git isn't installed or the project isn't a git repo, those searches silently skip.

License

Do whatever you want with it. No warranty — this is a recovery tool of last resort.

---
