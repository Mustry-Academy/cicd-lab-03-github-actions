# Block B — GitHub Actions: workflows, jobs, steps

**Duration:** ~90 minutes
* 15 min demo
* 20 min we-do
* 35 min you-do
* 15 min debrief
* ~5 min buffer

## Goal

You should leave this block able to:

- Read a GitHub Actions workflow and explain what every line does
- Write a workflow from scratch with sensible defaults: PR trigger, least-privilege permissions, path filters
- Wire the Block A linters into CI so the safety net runs on every PR
- Use secrets and the auto-provisioned `GITHUB_TOKEN` correctly

## Pre-flight

```bash
git fetch --tags
git checkout block-b-start
```

You should have a clean repo (all Block A fixes applied, `.yamllint.yml` in place).

You'll need: a GitHub repo of your own. If you haven't already, push this lab to your own GitHub org / personal account so you can open PRs against it. (`gh repo create --source=. --push --private`.)

## I do (15 min)

The instructor draws the mental model on the board:

```
workflow  ──contains──▶  jobs  ──contains──▶  steps  ──run──▶  actions or shell commands
   │                       │                      │
   │                       │                      └── permissions, env, working-directory…
   │                       └── runs-on, needs, strategy, env, permissions
   └── on (triggers), permissions, env, concurrency
```

Then live-create `.github/workflows/ci.yml`:

```yaml
name: CI
on:
  pull_request:
permissions:
  contents: read
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r sample-app/requirements.txt
      - run: pytest sample-app/tests -q
```

Open a PR, watch it run, read the logs together. Each step is a separate collapsible block in the GitHub UI — that's intentional.

## We do (20 min)

Together, add a second job that runs the linters from Block A:

```yaml
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install yamllint==1.35.1 ruff==0.6.9
      - run: yamllint -c .yamllint.yml .
      - run: ruff check .
      - uses: hadolint/hadolint-action@v3.1.0
        with:
          dockerfile: sample-app/Dockerfile
      - uses: raven-actions/actionlint@v2
      - run: sudo apt-get update && sudo apt-get install -y --no-install-recommends shellcheck
      - run: shellcheck scripts/*.sh
```

Discuss as you go:

- **`permissions: contents: read`** at the workflow level — why least-privilege matters even for innocuous-looking jobs.
- **`GITHUB_TOKEN`** — automatically provided per-job, expires when the job ends, scoped to the repo.
- **Secrets vs variables** — secrets are encrypted and masked in logs; variables are plain text. Live-add one secret (`EXAMPLE_SECRET`) in repo settings and reference it as `${{ secrets.EXAMPLE_SECRET }}` in a debug step. Confirm it shows as `***` in the log.

## You do (35 min)

Make the workflow yours.

### Part 1 — Path filters (10 min)

Right now, the workflow runs on every PR — even docs-only changes. Add a `paths:` filter to `on.pull_request` so the workflow only triggers when files that actually affect tests/lint change:

```yaml
on:
  pull_request:
    paths:
      - "sample-app/**"
      - "scripts/**"
      - "docker-compose.yml"
      - ".github/workflows/**"
      - ".yamllint.yml"
```

Open a tiny PR that touches **only** `README.md`. Confirm the workflow is **skipped**, not just passed (look for the "Skipped" label).

### Part 2 — Compose validation (5 min)

Add a final step to the lint job:

```yaml
      - run: docker compose config -q
```

This catches Compose-level issues that `yamllint` can't see — references to undefined services, port conflicts, malformed environment.

### Part 3 — Status badge (5 min)

Add a CI status badge to the top of `README.md`:

```markdown
[![CI](https://github.com/<you>/cicd-lab-03-github-actions/actions/workflows/ci.yml/badge.svg)](https://github.com/<you>/cicd-lab-03-github-actions/actions/workflows/ci.yml)
```

Push, refresh, confirm the badge renders.

### Part 4 — Required check (10 min)

In your repo settings, configure branch protection on `main`:

- Require a pull request before merging
- Require status checks to pass before merging — select `lint` and `test`

Open a fresh PR that breaks one lint rule. Confirm GitHub blocks the merge. Then fix and re-push.

### Part 5 — Sanity check (5 min)

Commit any remaining changes. Match the reference state in [`instructor-notes/block-b-key.md`](../instructor-notes/block-b-key.md). End state: `block-b-end`.

## Stretch challenge `[OPTIONAL]`

**Matrix the tests over multiple Python versions:**

```yaml
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python: ["3.11", "3.12"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python }}
          cache: "pip"
          cache-dependency-path: sample-app/requirements.txt
      - run: pip install -r sample-app/requirements.txt
      - run: pytest sample-app/tests -q
```

You should now see two parallel test jobs. Note the `cache: "pip"` line — it speeds up the second run substantially.

**Read but don't implement:** the difference between `on: pull_request` and `on: pull_request_target`. Skim the [GitHub Security Lab post on `pull_request_target`](https://securitylab.github.com/resources/github-actions-preventing-pwn-requests/) — short version, `pull_request_target` runs the workflow file from `main` *with secrets* against the PR's code, which is a well-known privilege-escalation footgun. Default to `pull_request` unless you have a specific reason.

## Debrief (15 min)

- Where does the workflow actually run? (Hint: ephemeral runner spun up per job.)
- What happens if a step fails midway? What about a job? Read [`jobs.<id>.if`](https://docs.github.com/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idif) and discuss `continue-on-error`.
- How do you make CI a required check? You just did it — talk through the *people* implications. (Who decides what's blocking?)
- For your Ignition projects: what's the *first* check you'd add to CI? Probably not the last; probably the cheapest one that catches a real bug you've seen.
