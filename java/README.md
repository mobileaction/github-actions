# Java (Gradle) Actions

| Action | Purpose |
|---|---|
| [`java/test`](test/action.yml) | Checkout, JDK setup, `./gradlew build`, upload test reports, optional Sentry release |
| [`java/test_sonar`](test_sonar/action.yml) | Same as `java/test` plus SonarCloud analysis (`./gradlew build sonar`) |

`SENTRY_AUTH_TOKEN`, `SENTRY_ORG`, and `SONAR_TOKEN` are **organization secrets** — inherited automatically, no per-repo setup needed. `SENTRY_PROJECT` must be set per repo to the service's Sentry project slug.

## Workflow samples

Drop these files into `.github/workflows/` of a consumer repo. Adjust `java-version`, `sentry-project`, and branch names to match the target service.

### `sonarTest.yaml` — PR validation with SonarCloud

Runs on every PR (except into `master`) and on pushes to `dev`. Executes `./gradlew build sonar` via [`java/test_sonar`](test_sonar/action.yml).

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
            checks: write  # required so the composite can publish JUnit results to the PR
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

Runs on push to `master` (post-merge, before Heroku deploys). Builds via [`java/test`](test/action.yml) and creates a Sentry release for the `production` environment.

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

Also configure the Sentry Gradle plugin in `build.gradle`:

```groovy
plugins {
    id "io.sentry.jvm.gradle" version "6.1.0"
}

def sentryToken = System.getenv("SENTRY_AUTH_TOKEN")

sentry {
    org = "mobileaction"
    projectName = "<service-name>"

    if (sentryToken != null && !sentryToken.trim().isEmpty()) {
        authToken = sentryToken
        includeSourceContext = true
    } else {
        includeSourceContext = false
    }
}
```
