# gradle build

[![ci](https://github.com/Arda-cards/CodeQL-workflow/actions/workflows/ci.yaml/badge.svg)](https://github.com/Arda-cards/CodeQL-workflow/actions/workflows/ci.yaml)
[CHANGELOG.md](CHANGELOG.md)

CodeQL analysis for a Gradle project, as jobs a consumer runs alongside its
build rather than inside it.

This replaces GitHub's default setup entirely. The documented way to adopt an
advanced configuration is to "Switch to advanced" and disable CodeQL default
setup, which is repository-wide rather than per-language — so a consumer that
adopts this must also disable default setup, and this workflow has to cover
every language that setup was covering, not only the compiled ones.

Hence two jobs. They differ in one thing that cannot be expressed in a single
CodeQL invocation: whether the language needs to be built.

- `compiled`    java-kotlin, build-mode manual — CodeQL's extractor observes
                the compiler, so there has to be a compile for it to watch.
- `interpreted` actions, javascript-typescript and the like, build-mode none —
                extracted straight from source, so no JDK, no Gradle, no buf.

Three constraints shaped the compiled job, and together they rule out the
simpler designs.

1. `init`, the build, and `analyze` must live in one job. A composite action
    cannot create a job, which is why this is a reusable workflow.

2. Wrapping the consumer's existing `./gradlew build` would make analysis wait
    for the tests. Compiling separately costs one extra compile and hides the
    whole analysis under the test run instead.

3. `org.gradle.caching=true` is set in `operations` and `common-module`, so a
    cache hit makes `compileKotlin` UP-TO-DATE and the compiler never runs.
    CodeQL would then extract nothing and report zero alerts — indistinguishable
    from clean code. `--no-build-cache` is what stops that, and it is why this
    build cannot share the test build's cache even though it shares the
    dependency cache.

## Arguments

See [codeql.yaml](.github/workflows/codeql.yaml).

## Usage

```yaml
codeql:
  if: "${{ github.event_name != 'push' || github.ref_name != 'main' }}"
  permissions:
    contents: read
    security-events: write
  uses: Arda-cards/CodeQL-workflow/.github/workflows/codeql.yaml@v1
  with:
    no_build_languages: '["actions","javascript-typescript"]'
    buf_version: "1.57.0"
  secrets:
    gpr_user: "${{ secrets.GPR_READ_USER }}"
    gpr_key: "${{ secrets.GPR_READ_KEY }}"
    buf_token: "${{ secrets.BUF_TOKEN }}"
```

## Permission Required

```yaml
permissions:
  contents: read
  security-events: write
```
