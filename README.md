# android describe — Configuration Cache Bug Reproducer

Minimal reproduction project for a bug in the [Android CLI](https://developer.android.com/tools/agents/android-cli):
`android describe` fails on any project with Gradle configuration cache enabled.

Bug report filed at: https://issuetracker.google.com/issues/new?component=2091212

## Prerequisites

- [Android CLI](https://developer.android.com/tools/agents/android-cli) installed (`android --version`)
- JDK 17+
- A connected Android emulator or device (not required for this repro)

## Steps to reproduce

### 1. Clone this repository

```bash
git clone https://github.com/Thomas-Boutin/android-describe-configuration-cache-bug.git
cd android-describe-configuration-cache-bug
```

### 2. Verify configuration cache is enabled

```bash
grep "configuration-cache" gradle.properties
# Expected: org.gradle.configuration-cache=true
```

This is the **only** deviation from a stock `android create empty-activity` project.

### 3. Run `android describe`

```bash
android describe --project_dir=.
```

**Expected:** JSON model files produced under `app/build/AndroidProject.json`, exit 0.

**Actual:** build fails with exit 1:

```
Error: Critical tasks failed: [:app:dumpAndroidProjectModel]
Error: gradlew failed with exit code 1
```

Full output captured in [`repro-output.txt`](repro-output.txt).
Full Gradle stacktrace in [`repro-gradle-stacktrace.txt`](repro-gradle-stacktrace.txt).

### 4. Confirm the workaround works

```bash
./gradlew --no-configuration-cache \
  --init-script .gradle/init.gradle.kts \
  :app:dumpAndroidProjectModel
```

**Expected:** `BUILD SUCCESSFUL`, `app/build/AndroidProject.json` created.

Output captured in [`workaround-output.txt`](workaround-output.txt).

## Root cause

The CLI injects `.gradle/init.gradle.kts` which registers `DumpModelTask`.
Its `@TaskAction` accesses `Task.project` at execution time (line 32):

```kotlin
@TaskAction
fun dumpModel() {
    project.logger.lifecycle(...)                             // ← banned under CC
    val model = basicBuilder.buildAll(modelName.get(), project) // ← banned under CC
    ...
}
```

Gradle 9 forbids `Task.project` access at execution time when configuration cache is enabled.
The init script itself acknowledges this with a TODO comment.

## Environment

| Field | Value |
|---|---|
| Android CLI | `0.7.15232955` |
| Gradle | `9.1.0` |
| AGP | `9.1.1` |
| OS | macOS Darwin 25.3.0 arm64 |

See [`android-info.txt`](android-info.txt) for full CLI environment details.