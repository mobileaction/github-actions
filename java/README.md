# Github Action Java project

### Example

```
name: Main
on: [push]

env:
  DEEPSOURCE_DSN: "<enter here project dsn>"

jobs:
  build_test:
    name: Build and Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: mobileaction/github-actions/java/test@v2
  coverage:
    name: Upload Coverage Report
    needs: build_test
    if: github.ref != 'refs/heads/master'
    runs-on: ubuntu-latest
    steps:
      - uses: mobileaction/github-actions/java/coverage@v2
```
