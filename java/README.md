# Github Action Java project Flows

## Sonar Cloud Test and Analyse Operations

### Github Actions

- Define SONAR_TOKEN on Github Repository Secrets, value is on Sonar

main.yml
```
name: Main
on:
  push:
    branches:
      - dev
  pull_request:
    types: [opened, synchronize, reopened]

env:
  SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

jobs:
  build_test:
    name: Build and Test
    runs-on: ubuntu-latest
    steps:
      - uses: mobileaction/github-actions/java/test_sonar@v3
```

### Gradle File Update

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

## Basic Test Operations

### Github Actions

pre_deploy.yml
```
name: PreDeploy
on:
  push:
    branches:
      - master

jobs:
  build_test:
    name: Build and Test
    runs-on: ubuntu-latest
    steps:
      - uses: mobileaction/github-actions/java/test@v3
```