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

## Workflow samples

Drop these files into `.github/workflows/` of a consumer repo. Adjust `java-version`, `sentry-project`, and branch names to match the target service. Secrets (`SONAR_TOKEN`, `SENTRY_AUTH_TOKEN`) must be configured in the repo's GitHub settings.

### `sonarTest.yaml` — PR validation with SonarCloud

Runs on every PR (except into `master`) and on pushes to `dev`. Executes `./gradlew build sonar` via [`java/test_sonar`](java/test_sonar/action.yml).

```yaml
name: 'Sonar Test'
on:
    pull_request:
        types: [opened, synchronize, reopened]
        branches-ignore:
            - master
    push:
        branches:
            - dev

env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

jobs:
    build_test:
        name: Build and Test
        runs-on: ubuntu-latest
        permissions:
            contents: write
        steps:
            - uses: mobileaction/github-actions/java/test_sonar@v5
              with:
                java-version: 21
```

Also configure SonarCloud in `build.gradle`:

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

### `deployTest.yml` — production build + Sentry release

Runs on push to `master` (post-merge, before Heroku deploys). Builds via [`java/test`](java/test/action.yml) and creates a Sentry release for the `production` environment.

```yaml
name: 'Deploy Test'
on:
  push:
    branches:
      - master

env:
  SENTRY_AUTH_TOKEN: ${{ secrets.SENTRY_AUTH_TOKEN }}
  SENTRY_ORG: 'mobileaction'
  SENTRY_PROJECT: '<service-name>'

jobs:
  build_test:
    name: Build and Test
    runs-on: ubuntu-latest
    steps:
      - uses: mobileaction/github-actions/java/test@v5
        with:
          java-version: 21
          create-sentry-release: 'true'
          sentry-environment: 'production'
```

Also configure the Sentry Gradle plugin in `build.gradle` so the CI build can upload source context and create the release:

```groovy
plugins {
    id "io.sentry.jvm.gradle" version "6.1.0"
}

// Safely attempt to read the token from the environment
def sentryToken = System.getenv("SENTRY_AUTH_TOKEN")

sentry {
    org = "mobileaction"
    projectName = "<service-name>"

    // Only enable the advanced Sentry features if the token actually exists
    if (sentryToken != null && !sentryToken.trim().isEmpty()) {
        authToken = sentryToken
        includeSourceContext = true // Uploads inline code snippets in CI
    } else {
        includeSourceContext = false // Skips Sentry upload tasks for local developer builds
    }
}
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
