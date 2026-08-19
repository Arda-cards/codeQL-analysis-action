# CodeQL Analysis Action

[![ci](https://github.com/Arda-cards/codeQL-analysis-action/actions/workflows/ci.yaml/badge.svg)](https://github.com/Arda-cards/codeQL-analysis-action/actions/workflows/ci.yaml)
[CHANGELOG.md](CHANGELOG.md)

This replaces GitHub's default setup entirely. The documented way to adopt an
advanced configuration is to "Switch to advanced" and disable CodeQL default
setup, which is repository-wide rather than per-language — so a consumer that
adopts this must also disable default setup, and this workflow has to cover
every language that setup was covering, not only the compiled ones.

Three constraints shaped the compiled job, and together they rule out the
simpler designs.

`org.gradle.caching=true` is set in `operations` and `common-module`, so a
cache hit makes `compileKotlin` UP-TO-DATE and the compiler never runs.
CodeQL would then extract nothing and report zero alerts — indistinguishable
from clean code. `--no-build-cache` is what stops that, and it is why this
build cannot share the test build's cache even though it shares the
dependency cache.

## Arguments

See [action.yaml](action.yaml).

## Usage

```yaml
name: "CodeQL"

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

permissions: { }

jobs:
  analyze:
    name: Analyze (${{ matrix.language }})
    runs-on: 'ubuntu-latest'
    permissions:
      security-events: write
      packages: read
      actions: read
      contents: read

    strategy:
      fail-fast: false
      matrix:
        include:
          - language: actions
            build_mode: none
          - language: java-kotlin
            build_mode: manual
    steps:
      - uses: actions/checkout@v7
      - uses: Arda-cards/codeQL-analysis-action@dna/PDEV-1414
        with:
          language: ${{ matrix.language }}
          build_mode: ${{ matrix.build_mode }}
          token: ${{ secrets.GITHUB_TOKEN }}
          gpr_key: ${{ secrets.GPR_READ_KEY }}
          gpr_user: ${{ secrets.GPR_READ_USER }}
```

## Permission Required

```yaml
permissions:
  security-events: write
  packages: read
  actions: read
  contents: read
```
