# AGENTS.md — Contributor Guide for AI Agents

This repo (`mobileaction/github-actions`) contains reusable **composite GitHub Actions** consumed by other `mobileaction` service repos via `uses: mobileaction/github-actions/<path>@<tag>`.

---

## Repo layout

```
java/
  test/action.yml          # Gradle build + test + optional Sentry release
  test_sonar/action.yml    # Same + SonarCloud analysis + dependency submission
node/
  lint/action.yml          # yarn install + yarn lint (or custom command)
  test/action.yml          # yarn install + yarn test
  build/action.yml         # yarn install + yarn build
.github/workflows/
  validate.yml             # CI: actionlint + yamllint on every PR
README.md                  # Overview + action tables + versioning/CI notes
java/README.md             # Java workflow samples and Gradle configuration (authoritative)
node/README.md             # Node usage example
```

---

## Action contracts

### `java/test`

| Input | Required | Default | Notes |
|---|---|---|---|
| `java-version` | yes | `17` | Whole or semver string |
| `java-distribution` | yes | `temurin` | Passed to `actions/setup-java` |
| `create-sentry-release` | no | `false` | Set `'true'` on master push |
| `sentry-environment` | no | `development` | e.g. `production` |

**What it does:** checkout → JDK setup (Gradle cache) → `./gradlew build` → upload JUnit XML artifacts → publish results via `dorny/test-reporter@v3` → (if enabled) fetch full history → create Sentry release.

**Consumer permissions required:**
```yaml
permissions:
  checks: write   # for dorny/test-reporter to post PR check
```

**Sentry env vars:**
```yaml
env:
  SENTRY_AUTH_TOKEN: ${{ secrets.SENTRY_AUTH_TOKEN }}  # org-level secret, inherited automatically
  SENTRY_ORG: 'mobileaction'                           # org-level, same for all repos
  SENTRY_PROJECT: '<service-name>'                     # per-repo, set to the service's Sentry project slug
```

---

### `java/test_sonar`

| Input | Required | Default |
|---|---|---|
| `java-version` | yes | `17` |
| `java-distribution` | yes | `temurin` |

**What it does:** checkout (full depth) → JDK setup → SonarCloud cache → `./gradlew build sonar --info` → upload JUnit XML → publish via `dorny/test-reporter@v3` → Gradle dependency submission.

**Consumer permissions required:**
```yaml
permissions:
  contents: write  # Gradle dependency submission
  checks: write    # dorny/test-reporter
```

**Consumer secrets required:**
- `SONAR_TOKEN` — org-level secret, inherited automatically, no per-repo setup needed.

**Consumer `build.gradle` must include:**
```groovy
plugins {
    id "org.sonarqube" version "4.4.1.3373"
}

sonar {
    properties {
        property "sonar.projectKey", "mobileaction_<service-name>"
        property "sonar.organization", "mobileaction"
        property "sonar.host.url", "https://sonarcloud.io"
    }
}
```

---

### `node/lint`, `node/test`, `node/build`

All three share the same shape:

| Input | Required | Default |
|---|---|---|
| `node-version` | yes | `14` |
| `yarn-command` | yes | `lint` / `test:unit` / `build` |

**What they do:** checkout → cache `node_modules` by `yarn.lock` hash → `actions/setup-node` with yarn cache → `yarn install` → `yarn <yarn-command>`.

No special permissions needed. No secrets needed.

---

## Versioning

Tags are **major-version floating tags** (e.g. `v5`, `v6`). All consumers pin to a major tag.

To cut a new release after merging changes to `main`:

```bash
git tag -a -m "Description of this release" v6
git push --follow-tags
```

Current latest tag: **v5** (see git tags). Update the version number in README samples when bumping.

---

## CI (this repo's own validation)

`.github/workflows/validate.yml` runs on every PR:

- **actionlint** (`raven-actions/actionlint@v2`) — validates action YAML schema and inline `shell: bash` steps via shellcheck.
- **yamllint** (`ibiqlik/action-yamllint@v3`) — enforces YAML formatting (max line length 200, document-start disabled).

**Before opening a PR:** ensure any new or modified `action.yml` passes both checks. The validate workflow is the gate.

---

## Conventions

- All actions use `using: "composite"`.
- Pinned third-party actions use explicit major tags (`@v5`, `@v3`, etc.), never `@main` for third-party deps (the node actions currently use `@main` for `actions/*` — treat this as legacy debt, do not replicate the pattern in new actions).
- JUnit results are always uploaded as artifacts **and** published as a PR check via `dorny/test-reporter@v3`. Both steps use `if: always()` so failures are still reported.
- Do not add inputs without defaults unless they are truly required from the consumer. Composite actions cannot use `if: inputs.foo` — use string comparison (`inputs.foo == 'true'`).

---

## Secrets reference

| Secret | Scope | Who configures |
|---|---|---|
| `SENTRY_AUTH_TOKEN` | Org-level | Inherited automatically — no per-repo setup |
| `SENTRY_ORG` | Org-level | Hardcoded `mobileaction` — no per-repo setup |
| `SENTRY_PROJECT` | Per-repo | Set to the service's Sentry project slug in each consumer workflow |
| `SONAR_TOKEN` | Org-level | Inherited automatically — no per-repo setup |
