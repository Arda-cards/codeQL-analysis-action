# changelog

[![Keep a Changelog](https://img.shields.io/badge/Keep%20a%20Changelog-1.0.0-informational)](https://keepachangelog.com/en/1.0.0/)
[![Semantic Versioning](https://img.shields.io/badge/Semantic%20Versioning-2.0.0-informational)](https://semver.org/spec/v2.0.0.html)
![clq validated](https://img.shields.io/badge/clq-validated-success)

Keep the newest entry at top, format date according to ISO 8601: `YYYY-MM-DD`.

Categories, defined in [changemap.json](.github/clq/changemap.json):

- *major* release trigger:
  - `Changed` for changes in existing functionality.
  - `Removed` for now removed features.
- *minor* release trigger:
  - `Added` for new features.
  - `Deprecated` for soon-to-be removed features.
- *bugfix* release trigger:
  - `Fixed` for any bugfixes.
  - `Security` in case of vulnerabilities.

## [1.0.0] - 2026-08-18

### Added

- A reusable CodeQL workflow, giving any Gradle repository CodeQL analysis of its Kotlin and Java sources.
  GitHub's own default setup cannot build these projects — it has neither the package credentials nor buf —
  so the workflow compiles with the same toolchain the build uses. It runs beside the caller's build rather
  than inside it, so analysis does not lengthen the path to a merge.

  It replaces default setup rather than supplementing it, because switching to an advanced configuration
  disables CodeQL for the whole repository and not just for the compiled language. The workflow therefore also
  analyses the languages default setup was covering, named through no_build_languages.

  A CodeQL job reports the combined result under one name, so a ruleset has something stable to require:
  the per-language checks are named after the matrix and change whenever the language list does.
