---
name: windows-dev
description: Survival rules for agents working on Windows and PowerShell — shell syntax differences, paths, line endings, process file locks, exit codes, and spawn quirks that break Linux-minded commands. Use when running any shell command on this machine, when writing scripts that will execute on win32, when configuring tools that spawn child processes (MCP servers, npx, npm), or when a Windows-specific error appears.
---

# Windows / PowerShell Dev Rules

Battle-tested on win32. Append-only: each new verified quirk becomes a new bullet.

## Shell syntax (PowerShell, not bash)
- No `&&` / `||` chains. Use separate statements, or `;` plus an explicit check.
- After EVERY native exe, check `$LASTEXITCODE` — a zero exit code is not the same as "no error printed", and `$?` is not a reliable native-exit check.
- No `head`/`tail`/`grep`/`sed`/`awk`. Use `Get-Content -TotalCount N`, `-Tail N`, `Select-String`, or `Select-Object -First/-Last N`.
- `curl` is an alias for `Invoke-WebRequest` in Windows PowerShell — real curl is `curl.exe`.
- `rm -rf` doesn't exist: `Remove-Item -Recurse -Force`; add `-ErrorAction SilentlyContinue` when absence is acceptable.
- Pipes expect objects, not text — string formatting before a pipe can silently reshape output.

## Paths, files, line endings
- Backslash paths: `C:\Users\...` — quote any path with spaces; prefer `Join-Path` in scripts.
- Never hardcode `/tmp`. The approved temp dir here is `%LOCALAPPDATA%\Temp` (`C:\Users\Thinkbook\AppData\Local\Temp`).
- CRLF vs LF: files often have CRLF. When editing, preserve exactly; when generating scripts, remember PS reads CRLF fine, bash does not.
- Case-insensitive filesystem: `README.md` and `readme.md` collide — globbing with the wrong case still matches, which can mask typos.
- Windows locks open files: a rebuild or delete fails while a dev server/test/process holds a handle. "File in use" = find and stop the holder first; never just retry in a loop.

## Environment, spawns, exit codes
- Env vars: read `$env:X` (config files may use `{env:X}` interpolation); `export` doesn't exist — `$env:X = "value"` affects only the current process tree.
- Spawning `.cmd`/`.ps1` shims (npx, npm-installed bins) from a non-shell parent process on Windows requires a wrapper: `cmd /c <tool> ...` — plain exec of the shim name fails. This applies to MCP server configs and any tool config that launches npm binaries.
- Long-running servers: run them as background tasks; foreground timeouts kill them mid-flight.
- Execution policy can block scripts: run via `pwsh -File` / `powershell -ExecutionPolicy Bypass -File` rather than editing policy.
- Symlinks need admin or Developer Mode; shebang lines don't work — call interpreters explicitly (`node script.js`, never `./script`).
- npm globals live in `%APPDATA%\npm` (`C:\Users\Thinkbook\AppData\Roaming\npm`) — `.ps1`/`.cmd` shims there; that dir is on PATH.

## Judgments
- Verify assumptions by running the smallest probe command first; translate bash idioms mentally before pasting them.
- Prefer the dedicated tools (read/glob/grep/edit) over shell one-liners — they're cross-platform and don't inherit these quirks.
