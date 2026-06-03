# Changelog

## v1.9 — UI overhaul

The Python → EXE tab was crowded (five group boxes stacked vertically). Split
it into four focused sub-tabs (**Build / Options / Data & hidden imports /
Advanced**) and put a prominent `⚡ Build EXE` button (52 px, bold) directly
under the target `.py` field, so the common case is "pick `.py`, click Build."

Polish: AppUserModelID input now greys out with its checkbox; all path fields
gained clear (×) buttons; trailing info paragraph removed (sub-tab labels
carry that load).

## v1.8 — Build provenance, isolated venv, fat-venv warning, stale-source check

Four diagnosability / lean-build features triggered by a real misdiagnosis
where a 4-hour debug turned out to be a stale binary, not a packager bug.

- **Build stamp.** Every EXE now carries `EXE_TOOLBOX_BUILD.json` at the
  bundle root: source sha256, mtime, size, `built_at`, toolbox version,
  PyInstaller version, host platform. The same fingerprint is logged before
  every build. "Is this EXE from the current source?" becomes a one-second
  sha256 check.
- **Fat-venv warning.** Heuristic scan of the build interpreter for heavy
  libraries (numpy / pandas / scipy / torch / lxml / sqlalchemy / pyarrow /
  …) that are *not* declared in `requirements.txt`. Transitive optional
  imports can drag those into the EXE; a real build came back at 115 MB
  from this pattern. The warning surfaces the suspects in the build log.
- **Isolated build mode.** Optional checkbox that creates a throwaway venv
  inside the build's temp dir, installs only `requirements.txt + pyinstaller`,
  builds with that interpreter, and discards it. Lean by construction.
- **Stale-source soft-confirm.** Flags target paths that sit inside snapshot
  folders (`Version_Control/`, `OldMains/`, `Archive/`, …) or have a newer
  same-base-name sibling in the parent. Soft Yes/No dialog before the build
  runs.

Smaller fix: the *Detect web/Flask data* button no longer blanket-adds the
`pystray._win32` hidden import. It scans the target source first via
`target_imports_module` and only wires it when `pystray` is actually used —
otherwise PyInstaller emits a misleading "Hidden import not found" error
for Flask apps that have no tray code.

## v1.7 — Data-file bundling for Flask / web apps

PyInstaller only auto-collects imported Python modules. Anything else
(Jinja templates, static assets, `.env`, icons) had to be declared
explicitly via `--add-data`, and the toolbox previously didn't expose that.
The symptom: a packaged Flask app builds cleanly, then returns HTTP 500 on
the index route because Jinja can't find its templates inside the EXE.

The fix:

- New **Data files & hidden imports** UI section. One entry per line:
  `SRC` or `SRC | DEST`. Folders default DEST to their name; files default
  to bundle root.
- `resolve_data_entry()` emits `--add-data` with `os.pathsep` (`;` on
  Windows, `:` on POSIX). A `:` on Windows is the classic reason
  `--add-data` silently does nothing — exactly the original bug.
- A listed-but-missing path raises `FileNotFoundError` rather than being
  silently skipped. Silent-skip *is* the original bug; staying loud is the
  fix.
- **Detect web/Flask data** button auto-wires `templates/`, `static/`,
  `icon.*`, `.env` from beside the target `.py`, plus
  `--collect-data certifi` for outbound HTTPS/TLS.
- Hidden-imports field for backends PyInstaller can't statically discover
  (e.g. `pystray._win32`, the dynamically imported Windows tray backend).

Also fixed a latent `NameError`: `open_folder()` was called by three
"Open output folder" buttons but never defined.

## v1.6.x — Initial git history

Moved from hand-versioned filename suffixes (`exe_toolbox_V1_6_6.py`) to
git history.

Carried forward from earlier iterations:

- PyInstaller front-end: icon (`.ico` or `.png`, PNG auto-converted to
  multi-size `.ico` first), optional `requirements.txt` install,
  windowed / onefile / clean controls, raw extra-args field for advanced
  cases.
- AppUserModelID runtime hook + forced PyQt window/taskbar icon hook —
  fixes blank/generic taskbar icons on unpinned PyQt EXEs.
- Quoted `py.EXE` paths in the Python command are stripped before
  subprocess invocation. Passing `['"C:\\Windows\\py.EXE"', '-3']` to
  `subprocess.Popen` triggers `WinError 2` because Windows tries to open
  a literal file named with quote marks.
- PNG → multi-size Windows ICO converter (16 / 24 / 32 / 48 / 64 / 128 /
  256 px) with optional light/dark border-background removal and
  square-canvas centering. Writes via temp file + atomic replace to avoid
  a Pillow direct-save corruption issue.
- PaddleOCR official preset: shells out to the selected Python to compute
  `--collect-data paddlex --collect-binaries paddle --copy-metadata …`,
  and unchecks one-file (onedir is safer for Paddle).
