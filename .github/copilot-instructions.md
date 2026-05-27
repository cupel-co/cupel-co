# Copilot instructions

These notes are for agents new to this repository. Trust this file and only search further if details here are missing or wrong.

## Repository summary
- This repository (`cupel-co/cupel-co`) is the top-level project repo for the Cupel organisation.
- At present it contains no application source code. It holds:
  - `LICENSE` and a placeholder `README.md`.
  - `.github/` configuration: workflows, CODEOWNERS, Dependabot config, and the PR template.
- All CI behaviour is delegated to reusable workflows hosted in the sibling repository `cupel-co/workflows` (versioning, release, PR notifications, PR updates). Local workflows are thin wrappers that `uses:` those reusable workflows pinned by commit SHA.
- When application code is eventually added, it should live under the `src/` folder.

## Security and standardisation rules
- **Always pin third-party (and cross-repo first-party) `uses:` references to a full-length 40-char commit SHA, never a tag or branch.** Tags/branches are mutable and a supply-chain risk.
  - Correct: `uses: owner/repo@de0fac2e4500dabe0009e67214ff5f5447ce83dd`
  - Correct: `uses: cupel-co/workflows/.github/workflows/release.create.yml@90206ef2c0f1c41fc9166fff3dba678e66277667`
  - Incorrect: `uses: actions/checkout@v4`, `uses: owner/repo@main`
- A trailing `# vX.Y.Z` comment after the SHA is allowed (and encouraged) to document the intended version, as seen in `dependency.yml`.
- All cross-repo reusable workflow calls in this repo currently pin to the same `cupel-co/workflows` SHA (`90206ef2c0f1c41fc9166fff3dba678e66277667`). When bumping, update every reference consistently.
- Workflows must declare least-privilege permissions:
  - Default `permissions: {}` at the workflow level when possible (see `codeql.yml`, `dependency.yml`).
  - Grant per-job `permissions:` only for what the job needs (e.g. `contents: write` for release, `pull-requests: write` for PR updates).
- `actions/checkout` should use `persist-credentials: false` unless the job specifically needs the default token to push.
- Dependabot manages GitHub Actions updates weekly (`.github/dependabot.yml`); accept its SHA bumps rather than hand-editing pins to tags.

## Conventions for workflows in `.github/workflows/`
- File naming is lowercase, hyphen-separated, and groups related events (e.g. `pull-request-notify.yml`, `pull-request-update.yml`).
- Each workflow sets a `concurrency` group of `'${{ github.workflow }}-${{ github.ref_name }}'` with `cancel-in-progress: false` when it must not interrupt a running release/version run.
- Reusable workflow secrets are passed explicitly under `secrets:` (do not use `secrets: inherit`). Expected org/repo secrets referenced today:
  - `GOOGLE_CHAT_WEBHOOK_URL`
  - `GH_GPG_PRIVATE_KEY`, `GH_GPG_PRIVATE_KEY_PASSWORD`
  - `GH_RELEASE_PAT`
- CodeQL is configured for the `actions` language only (this repo currently has no other code). If application code is added, extend `codeql.yml` `languages:` accordingly.
- Dependency Review (`dependency.yml`) runs on PRs with `fail-on-severity: high` and `comment-summary-in-pr: on-failure` — preserve this behaviour.

## Stack and runtime versions
- No application stack is defined yet.
- Workflow jobs run on `ubuntu-latest` runners.
- Do not introduce a language/runtime assumption until app code lands; if you add code, also add the appropriate `.gitignore` entries (the current `.gitignore` is the Visual Studio / .NET template plus `.idea/`).

## Setup, build, run, and test
- There is no build, run, or test step for this repo today. Do not invent one.
- If you add code, also add: a build/test workflow that follows the pinning rules above, an entry in the root `README.md`, and CODEOWNERS coverage if needed.

## Project layout and key files
- `README.md` — project landing page (currently only a CI badge for `integrate.yml`).
- `LICENSE` — project license.
- `.gitignore` — Visual Studio / .NET template plus `.idea/`.
- `.github/CODEOWNERS` — `@cupel-co/platform` owns everything by default.
- `.github/dependabot.yml` — weekly grouped updates for `github-actions` in `/.github`.
- `.github/pull_request_template.md` — PR template with issue link, preview badge, description, screenshots, and notes sections. Preserve the `{{...}}` placeholders; they are filled in by tooling.
- `.github/workflows/`:
  - `integrate.yml` — push-to-`main`: calls `version.generate.yml` then `release.create.yml` (needs `contents: write`).
  - `preview.yml` — PR event: calls `version.generate.yml` only.
  - `pull-request-notify.yml` — PR opened/reopened: posts Google Chat notification.
  - `pull-request-update.yml` — PR opened/reopened: updates PR metadata; targets environment `Preview`.
  - `codeql.yml` — push/PR/weekly cron CodeQL scan for GitHub Actions.
  - `dependency.yml` — PR dependency review.

## Known gaps and red flags
- The repository is intentionally minimal right now. If a task asks you to "edit the action under `.github/actions/...`", that path does **not** exist here — those live in other Cupel repos. Ask the user which repo they mean before creating new action folders here.
- If you see a `uses:` reference pinned to a tag, branch, or short SHA, treat it as a security issue and propose pinning to a full-length commit SHA (Dependabot will then keep it current).
- If you bump the `cupel-co/workflows` SHA, update **all** workflow files in lockstep so they stay on the same version.

