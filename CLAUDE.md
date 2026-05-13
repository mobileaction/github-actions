# CLAUDE.md

See [AGENTS.md](AGENTS.md) for the full contributor guide: repo layout, action contracts, input/output tables, versioning, CI, and conventions.

## Quick orientation

- This repo holds **composite GitHub Actions** consumed by other `mobileaction` repos via `uses: mobileaction/github-actions/<path>@<tag>`.
- Changes go to `main` and are released by floating a new major tag (`v6`, `v7`, …).
- Every PR is validated by `.github/workflows/validate.yml` (actionlint + yamllint). Run it mentally before proposing changes.

## Working in this repo

- Edit only files under `java/`, `node/`, `.github/workflows/`, and the root/sub `README.md` files. Do not touch `.idea/` or other IDE files.
- When modifying an action, check whether the README sample needs updating too.
- Do not introduce new `action.yml` files without a corresponding README section and a sample workflow snippet.
- All inline `run:` blocks must use `shell: bash` explicitly — actionlint will fail otherwise.
- Keep third-party action pins at explicit major tags, not `@main`.
