# android describe — Configuration Cache Bug Reproducer

Minimal reproduction project for a bug in the [Android CLI](https://developer.android.com/tools/agents/android-cli):
`android describe` fails on any project with Gradle configuration cache enabled.

Bug report filed at: https://issuetracker.google.com/issues/new?component=2091212

## How to reproduce

See [REPRODUCING.md](REPRODUCING.md) for the full step-by-step guide, root cause analysis, and workaround.

## Environment

| Field | Value |
|---|---|
| Android CLI | `0.7.15232955` |
| Gradle | `9.1.0` |
| AGP | `9.1.1` |
| OS | macOS Darwin 25.3.0 arm64 |