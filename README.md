# EXE Toolbox

A PyQt6 desktop GUI for packaging Python apps into Windows `.exe` files,
with the fixes for the things PyInstaller silently gets wrong.

Also includes a PNG -> multi-size Windows `.ico` converter for app icons.

> *Screenshot coming.*

## What makes it different

- **Flask / web EXEs that actually work.** Bundles `templates/`, `static/`,
  `.env` and icons by default: no more HTTP 500 on a packaged Flask app.
- **Tells you which source built it.** Every EXE carries a JSON stamp with
  source sha256 and build time. Stale-binary debugging dies.
- **Lean builds on demand.** One click builds in a throwaway venv from
  `requirements.txt`, so a fat dev environment doesn't bloat the EXE.
- **PyQt taskbar icons that don't go blank** on unpinned EXEs (a Windows
  gotcha, fixed automatically).

Engineering log with the bug stories: [CHANGELOG.md](CHANGELOG.md).

## Tech

PyQt6, Pillow, PyInstaller, Python 3.12+. Windows-first.

## Install & run

```powershell
git clone https://github.com/CanGitArchive/Exe-Toolbox.git
cd Exe-Toolbox
py -3 -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt
.venv\Scripts\python.exe exe_toolbox.py
```

The app can package itself: drop `exe_toolbox.py` onto the target field,
click **⚡ Build EXE**.

## License

MIT: see [LICENSE](LICENSE).
