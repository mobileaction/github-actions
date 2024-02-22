# Github Action Java project with Sonar

## Github Action Sample

- Define SONAR_TOKEN on Github Repository Secrets, value is on Sonar

```
name: Main
on:
  push:
    branches:
      - dev
      - master
  pull_request:
    types: [opened, synchronize, reopened]
jobs:
  build_test:
    name: Build and Test
    runs-on: ubuntu-latest
    steps:
      - uses: mobileaction/github-actions/java/test_analyze@main
```

## Gradle File Update

- Apply Sonar plugin on gradle file
```
plugins {
...
    id "org.sonarqube" version "4.4.1.3373"
}
```

- Add Sonar config in gradle file
- Project key should match the sonar project, eg: mobileaction_app_match
```
sonar {
    properties {
        property "sonar.projectKey", "mobileaction_XXX"
        property "sonar.organization", "mobileaction"
        property "sonar.host.url", "https://sonarcloud.io"
    }
}
```