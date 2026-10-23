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

---

# 2026-10-21

## Steps
- Removed the uv init demo code (`[projects.scripts]` + `main()`), set the pyproject description.
- Split the work into two commits (code vs log) with imperative messages.
- Fresh-clone check passed: `uv sync` + `import ai_stack`, exit code 0.
- Opened the PR "Set up project scaffolding with uv" (description + test line), squash-merged with a cleaned message, deleted the branch.

## Time
~60 min

## Learned
- package = what a thing is; dependency = role it plays for my project.
- Editable install: code changes in `src/` apply immediately; `pyproject.toml` changes need a reinstall (`uv sync` showed `~ ai-stack`).
- `echo $?` shows the last command's exit code; 0 = success.

## Snags
- `/tmp` is a temporary folder which is deleted on reboot

---

# 2026-10-22

## Steps
- Created the Task 2 branch from updated `main`; committed the 2026-10-21 entry.
- Installed Ollama (Linux instructions from ollama.com).
- Pulled `gemma4:e4b` (~6.6 GB total: 5.5 GB main part + ~1 GB second part + small files).
- First answer via `ollama run gemma4:e4b` (pi to 20 decimals: correct).
- `ollama ps`: 100% GPU, CONTEXT 4096, runner llamacpp. The 6700 XT worked without any workaround.
- VRAM check: card1 (6700 XT) 6.1 GB used with the model loaded; card2 (integrated graphics) 25 MB.
- Reading session 1: AI Engineering, Ch. 10 intro + Steps 1–3.

## Time
- Ollama setup and tests: 60 min
- Reading: 30 min
- Notes: 10 min

Total: 100 min (planned: 60)

## Decisions
- Model: `gemma4:e4b`. It's official, recent (April 2026), its ~6 GB footprint leaves headroom on 12 GB of VRAM, and it would stay usable on CPU if the GPU failed. Candidate for M2: `gemma4:12b-it-qat`.
- Always pin the exact model tag (`gemma4:e4b`, not `gemma4`): `latest` can change underneath me. Same reason as pinning Python and committing `uv.lock`.

## Reading: AI Engineering, Ch. 10 (intro, Steps 1–3)
- Router vs gateway: the router decides which model gets a request (e.g., a prompt too long for one model goes to one with a bigger context window). The gateway gives one interface to all models, so apps don't change when models change; it also handles API keys and access, cost tracking and rate limits, fallback when a provider fails, and logging.
- Input guardrails can mask sensitive data with placeholders and keep the mapping, so nothing confidential reaches third-party APIs and the response can be unmasked afterwards.
- Guardrails can block unsafe or policy-violating prompts before any model call, which also saves a paid API call.

Question: If two models can both handle a prompt but give different results, how does the system decide where to send it? Or does it send to both and pick the better answer? E.g., an 8B vs a 32B model, especially for code or artistic design.
Answer: Route on measured quality. Options from cheap to expensive: rules → a classifier router → a cascade (small model first, escalate if a check fails; code has objective checks like tests, design doesn't) → send to both and judge (mostly offline, to build evaluation data). M1: the client picks the model; the logs build the evidence; content-based routing is a later milestone.

## Snags
- Ollama's search mixes community uploads (`user/model`) with official models (no username). Red flags: "abliterated", distills from random uploaders with few downloads, MLX tags (Apple-only).
- `ollama ps` SIZE showed 364 MB while VRAM use was 6.1 GB, so that column doesn't reflect VRAM use. Measured directly via `/sys/class/drm/card*/device/mem_info_vram_used`.
- Ollama's default context is 4096 tokens, much smaller than the model supports. Matters in M2.

## Next (2026-10-23)
- Unloaded VRAM reading (`ollama stop gemma4:e4b`, rerun the sysfs loop) → model footprint = 6.1 GB minus that.
- `ollama run gemma4:e4b --verbose` → record the eval rate (tokens/sec).
- curl to `/v1/chat/completions` → save the raw response to `tests/fixtures/ollama_chat_response.json`.
- Reading session 2: Monitoring and Observability.

---

# 2026-10-23

## Steps
- VRAM footprint: card1 1 GB unloaded vs 6.1 GB loaded → `gemma4:e4b` ≈ (6.1 − X) GB.
- Speed baseline (`ollama run gemma4:e4b --verbose`, 2 runs):
  - Generation: ~122–127 tokens/s, stable across runs.
  - Prompt processing: 221 tokens/s at a 24-token prompt, ~1,127 tokens/s at 1,105 tokens.
  - Load duration < 1 ms both times: `ollama run` loads the model when the session starts.
- Saved a real response with curl to `tests/fixtures/ollama_chat_response.json`; `json.tool` confirms valid JSON.
- Reading session 2: Ch. 10, Monitoring and Observability (plus the sections before it).

## Time
- Reading: 35 min
- VRAM, speed, fixture, notes: 50 min

Total: 85 min (planned: 60)

## Learned
- A chat re-sends the whole conversation with every message: the second prompt was 1,105 tokens for a short question. The model is stateless; the client sends the history.
- Two different speeds: the prompt is processed in parallel (fast), the answer is generated one token at a time (slower). Tiny prompts give misleading prompt speeds.
- Fixture: standard OpenAI format, no non-standard fields. Beyond the minimal model: `system_fingerprint`, `usage.prompt_tokens_details.cached_tokens`. Task 3's response model must read this file unchanged, so the extra-fields decision applies to responses too.
- "Say hello in one sentence." = 22 prompt tokens: each message carries formatting tokens, and they're billed.

## Reading: Ch. 10, Monitoring and Observability
- MTTD (mean time to detection), MTTR (mean time to respond), CFR (change failure rate)
- Metrics depend on the app (e.g. how often do users stop a generation halfway?; What's the average number of tokens per input? What's the average number of tokens per output?)
- Monitoring tells me *that* something went wrong; observability means the system records enough to work out *why* from its outputs. ("Observability" became popular in the mid-2010s.)
- Logs are essential for debugging (model, tokens, finish_reason, latency…). But logging full prompts/responses has privacy costs: M1 logs metadata, not content.

Question: Should observability be planned before building? Could an AI model later watch the observability data and fix problems (with my approval), so the app maintains or grows itself?
Answer: Decide up front what's needed to debug one failed request, and record that from day one (e.g., a request ID in every log line); improve it as failures teach me. An AI that maintains the system is possible later: it depends on good observability, should earn trust in stages (diagnose → propose fixes as PRs → low-risk actions), and never ships unreviewed changes.

## Snags
- `ollama run`: the model loads when the session starts, not with the first question.
- `uv run` rebuilt ai-stack after a branch switch: Git rewrote `pyproject.toml`, and uv reinstalled the project. Harmless.


## Later ideas
- An AI agent that reads observability data and proposes fixes as PRs (needs M1 logging first).
