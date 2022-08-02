# Github Action Java project

### Example

```
name: Main
on: [push]
jobs:
  build_test:
    name: Build and Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: mobileaction/github-actions/java/test@v1
  coverage:
    name: Upload Coverage Report
    needs: build_test
    if: github.ref != 'refs/heads/master'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: mobileaction/github-actions/java/coverage@v1
        with:
          codacy-project-token: ${{ secrets.CODACY_PROJECT_TOKEN }}
```
