# Block A — Validation and linters as your safety net

**Duration:** ~90 minutes
* 15 min demo
* 15 min we-do
* 30 min you-do
* 15 min debrief
* ~15 min buffer / stretch

## Goal

You should leave this block able to:

- Run **yamllint, hadolint, actionlint, shellcheck, and ruff** locally against a real project
- Read each tool's output and decide whether to fix, configure away, or ignore each finding
- Add a `.yamllint.yml` to make the linter fit *your* project rather than the other way around
- Explain when CI linting helps and when it gets in the way

## Pre-flight

```bash
git fetch --tags
git checkout block-a-start
```

You're now on a deliberately-broken state with **6 planted lint issues** scattered across the repo. The instructor will walk you through finding them; you'll fix the rest solo.

Install the tools you don't have yet. macOS:

```bash
brew install yamllint hadolint shellcheck actionlint
pip install ruff==0.6.9
```

Linux / Codespaces:

```bash
pip install yamllint==1.35.1 ruff==0.6.9
# Then one of: apt install / install from release tarballs
sudo apt-get update && sudo apt-get install -y shellcheck
# hadolint and actionlint: download from GitHub releases
curl -fsSL https://github.com/hadolint/hadolint/releases/latest/download/hadolint-Linux-x86_64 \
  -o /usr/local/bin/hadolint && chmod +x /usr/local/bin/hadolint
bash <(curl -fsSL https://raw.githubusercontent.com/rhysd/actionlint/main/scripts/download-actionlint.bash)
```

If you'd like to read ahead: [`docs/validation-and-linters.md`](../docs/validation-and-linters.md).

## I do (15 min)

The instructor introduces the safety-net metaphor: every linter is a cheap, fast check that catches a class of bug. The point isn't to use them all; the point is to know which one would have caught yesterday's regression.

Live-demo on the broken `block-a-start` state. For each tool, run it, read the output, fix one finding:

1. `yamllint -c .yamllint.yml docker-compose.yml` — finds the planted YAML issue.
2. `docker compose config -q` — schema-level Compose validation. Compose says "yes this parses."
3. `hadolint sample-app/Dockerfile` — best practices for Dockerfiles. Look at the rule codes (DL3008, DL3015…) — every one has a documented justification.
4. `actionlint` — workflow YAML syntax + expression typing. Run it on the seeded `example.yml` workflow.
5. `ruff check .` — fast Python linter. Replaces flake8 / pylint / pycodestyle for ~95% of cases.
6. `shellcheck scripts/*.sh` — catches almost every shell scripting bug ever made.

Each tool is a separate command, but they share a shape: one config file, fast feedback, clear rule codes.

## We do (15 min)

Run each tool yourself on the broken state. Together as a class, fix **one** of the findings — pick the most interesting one. (The instructor will choose live.)

Don't fix the rest yet — that's the solo exercise.

## You do (30 min)

Fix the remaining planted issues, then make the linter config your own.

1. Run each of `yamllint`, `ruff`, `hadolint`, `shellcheck`, and `actionlint` and capture the findings in `NOTES.local.md` (gitignored). Make a list.
2. Fix every finding. Some take 30 seconds, some need a small refactor. For each fix, in your notes record: *what the tool flagged*, *why the fix is correct*, and *what category of bug this would catch in production*.
3. Re-run every linter until each one is silent.
4. Open `.yamllint.yml`. Notice we already disabled `line-length` for the project — but leave a comment in the file explaining *why*. (Hint: long REDIS_URL strings.)
5. Run `docker compose config -q` one more time. It should still say nothing.
6. Commit. Your end-state should match `block-a-end`.

> If you're stuck on a specific finding, [`instructor-notes/block-a-key.md`](../instructor-notes/block-a-key.md) has the seeded-error walkthrough. **Don't peek before you've spent at least 5 minutes on the finding yourself** — the diagnostic skill is most of the lesson.

## Stretch challenge `[OPTIONAL]`

Wire the linters into a pre-commit hook so bad commits get blocked locally, before they ever reach CI:

```bash
pip install pre-commit
pre-commit install
pre-commit run --all-files
```

The repo ships a starter [`.pre-commit-config.yaml`](../.pre-commit-config.yaml). Read it. Notice it pins versions — pre-commit doesn't *use* whatever's on your `PATH`; it manages its own toolchain. That's good for reproducibility, slightly opaque on first encounter.

Then deliberately break one file (drop a trailing whitespace, an unused import — whatever) and try to commit. Confirm the hook blocks you.

## Debrief (15 min)

- Which of the linters would have caught the most recent real bug your team shipped?
- When does linting hurt instead of help? (Hint: when it flags style preferences as errors, when it's slower than the dev cycle, when its config drifts from the codebase reality.)
- Is `# noqa` ever the right answer? When?
- For your Ignition projects: which of these tools transfer? Which don't? (We come back to this when we wrestle with `project.json` and tag exports in Lab 04.)
