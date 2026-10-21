# 2026-10-19

## Steps
- Installed: git, curl, build-essential, VS Code (+ Python and Ruff extensions).
- uv was already installed (standalone installer, `~/.local/bin/uv`); updated to 0.13.1 with `uv self update`.
- Set up: Git identity, SSH key for GitHub (`ssh -T git@github.com` works), VS Code AI features turned off.
- Created the GitHub repo `ai-stack` and cloned it over SSH into `~/code/ai-stack`.

## Time
~1.5 h (estimated the next day, not tracked on the day)

## Decisions
- One repo for the whole stack, with the gateway as its first service: all services will share one Docker Compose file, one CI pipeline and one LOG.md, and a change that touches several services fits in one PR.
- The repo lives on the Linux SSD (`~/code`), not on the shared NTFS HDD: NTFS on Linux handles symlinks and file permissions badly (uv's `.venv` relies on symlinks, Postgres needs strict permissions), Windows Fast Startup can leave the drive locked or risk corrupting it, and an HDD is slow with thousands of small files.

## Snags
- `yes` command: the SSH prompt had already exited, so typing "yes" ran the `yes` program, which prints "y" forever. Fix: Ctrl+C. Lesson: before typing, check whether the shell prompt or a program is waiting for input.
- Host-key prompt: on the first connection, SSH asks me to confirm GitHub's identity. I compared the fingerprint with GitHub's published list (pasting the fingerprint at the prompt makes SSH compare it). Host key (proves the server is GitHub) ≠ my key (proves I am me).
- VS Code `.deb` install printed an `N:` line about `_apt` and permission denied: it's a harmless notice (`N:`), not an error (`E:`).
- Microsoft's apt repository for VS Code was not added (`apt policy code` showed only `/var/lib/dpkg/status`), so VS Code would not update through apt. Fix tried: pre-answer the debconf question and reinstall the `.deb`. Result: fixed; `apt policy code` now shows `packages.microsoft.com`
- `cd downloads` failed: Linux paths are case-sensitive (`~/Downloads`).

---

# 2026-10-20

## Steps
- Step 1: Created branch `task1`, wrote LOG.md (Monday's entry), committed.
- Step 2: Switched the project from Mint's Python (3.12.3, `/usr/bin`) to a uv-managed Python 3.12.15 (`uv python install 3.12`, then recreated `.venv`). Reason: the project no longer depends on the OS's Python, which a system update or a Mint upgrade could change or remove.
- Step 3: Added dependencies. Normal: fastapi, pydantic, uvicorn. Dev group: httpx, openai, pytest, ruff.
- Step 4: Committed `uv.lock`: it records the exact version of every package, including indirect ones, so a fresh clone, CI and Docker all get the same environment.
- Step 5: Rewrote README.md.
- Step 6: Reviewed files and commands, updated LOG.md.

## Time
- Step 1: 25 min
- Step 2: 10 min
- Step 3: 10 min
- Step 4: 10 min
- Step 5: 20 min
- Step 6: 10 min

Total: 85 min (planned: 60)

## Decisions
- Package layout (`src/ai_stack/`) instead of app layout: the project is installed into `.venv`, so `import ai_stack` works from anywhere, including `tests/`, without depending on the current folder. A loose `main.py` is only importable if the repo root happens to be on Python’s import path.
- `.python-version` vs `requires-python`: `.python-version` is the Python I develop on (uv builds `.venv` with it; "3.12" means any 3.12.x). `requires-python` (in `pyproject.toml`) is the range the project promises to support: uv picks dependency versions that work across that whole range, and installers refuse older Pythons. The first must sit inside the second. Note: `>=3.12` also promises 3.13+, which I am not testing.
- Separate dev group: the production image installs only normal dependencies. Test and lint tools (pytest, httpx, ruff, openai) are needed for developing and CI, not for running the gateway; shipping them would make the image bigger and add packages that could carry vulnerabilities.
- Package name `ai_stack`: the repo will hold more than the gateway, so the top-level package is named after the stack; the gateway will live in `ai_stack/gateway/`.
- `uv.lock` is committed (see Step 4).

## Snags
- uv used Mint's Python (`/usr/bin/python3.12`) instead of downloading its own: uv uses a matching Python already on the system if it finds one. Fixed in Step 2.
- `uv init` did not create a `.gitignore`: it only does that when it creates the Git repo itself, and mine already existed. Wrote one by hand: `__pycache__/`, `.venv/`, `.env`.
- Read "untracked" in `git status` as "don't commit". Untracked means Git has never seen the file; only `.gitignore` means "never commit".
- The README claimed uv always downloads Python 3.12. That was wrong (see the first snag). I caught it by checking the claim against what happened on my machine.
- `.venv` (this project's packages) ≠ `.env` (secrets, never committed). The managed Python lives in `~/.local/share/uv/python/`; `.venv` only links to it.
