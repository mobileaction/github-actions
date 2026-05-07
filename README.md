# github-actions

Reusable composite GitHub Actions used by `mobileaction` service repos.

## Available actions

### Java (Gradle)

| Action | Purpose |
|---|---|
| [`java/test`](java/test/action.yml) | Checkout, JDK setup, `./gradlew build`, upload test reports, optional Sentry release |
| [`java/test_sonar`](java/test_sonar/action.yml) | Same as `java/test` plus SonarCloud analysis (`./gradlew build sonar`); requires `SONAR_TOKEN` |
| [`java/coverage`](java/coverage/action.yml) | _Deprecated — no longer used_ |

See [java/README.md](java/README.md) for usage examples.

### Node (Yarn)

| Action | Purpose |
|---|---|
| [`node/lint`](node/lint/action.yml) | `yarn install`, `yarn lint` (or any `yarn-command`) |
| [`node/test`](node/test/action.yml) | `yarn install`, `yarn test` |
| [`node/build`](node/build/action.yml) | `yarn install`, `yarn build` |

## Usage

Reference an action from any consumer repo, pinning to a major tag:

```yaml
- uses: mobileaction/github-actions/java/test_sonar@v5
```

## Versioning

```bash
git tag -a -m "Description of this release" v6
git push --follow-tags
```

## CI

Every PR is validated by [`.github/workflows/validate.yml`](.github/workflows/validate.yml):

- **actionlint** — schema check + shellcheck on inline `shell: bash` steps.
- **yamllint** — YAML format check across all `action.yml` files.
