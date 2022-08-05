# Github Action Node project

### Example

```
name: Main
on: [push]
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: mobileaction/github-actions/node/build@v1
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: mobileaction/github-actions/node/lint@v1
        with:
          yarn-command: lint:all-report
```
