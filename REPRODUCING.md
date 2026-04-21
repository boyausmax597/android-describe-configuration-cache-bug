# Bug Repro: `android describe` fails with Gradle Configuration Cache

## Bug summary

`android describe` fails on any project with `org.gradle.configuration-cache=true`.
The CLI's injected `DumpModelTask` calls `Task.project` at execution time, which Gradle 9 forbids under configuration cache.

Bug tracker: https://issuetracker.google.com/issues/new?component=2091212

## Environment

| Field | Value |
|---|---|
| Android CLI | 0.7.15232955 (see `android-info.txt`) |
| Gradle | 9.1.0 (wrapper, see `gradle/wrapper/gradle-wrapper.properties`) |
| AGP | see `gradle/libs.versions.toml` |
| OS | macOS Darwin 25.3.0 arm64 |

## How this repro was created

1. `android create empty-activity --name="Android Describe CC Repro" --output=.`
2. Added exactly **one line** to `gradle.properties`: `org.gradle.configuration-cache=true`

That is the only deviation from the generated template.

## How to reproduce the failure

```bash
android describe --project_dir=.
```

Expected: JSON model files produced under `app/build/AndroidProject.json`, exit 0.
Actual: build fails with exit 1. Full CLI output: `repro-output.txt`.

## Root cause (code pointer)

The CLI writes `.gradle/init.gradle.kts` and registers `DumpModelTask`.
At line 32 of that file:

```kotlin
@TaskAction
fun dumpModel() {
    project.logger.lifecycle(...)                          // ← Task.project at execution time
    val model = basicBuilder.buildAll(modelName.get(), project) // ← Task.project at execution time
    ...
}
```

Both `project` accesses are banned under CC. Gradle stacktrace: `repro-gradle-stacktrace.txt`.

The init script even includes a TODO acknowledging the problem:
```
// This task is problematic because it uses the project instance. We would need a better solution
// longer term by probably embedding the capability in AGP or requesting an API extension with
// Gradle.
```

## Workaround

```bash
./gradlew --no-configuration-cache \
  --init-script .gradle/init.gradle.kts \
  :app:dumpAndroidProjectModel
```

Succeeds and produces `app/build/AndroidProject.json`. Output: `workaround-output.txt`.

## Attached files

| File | Contents |
|---|---|
| `repro-output.txt` | `android describe` failure output |
| `repro-gradle-stacktrace.txt` | Full Gradle stacktrace showing the CC violation |
| `workaround-output.txt` | Passing run with `--no-configuration-cache` |
| `android-info.txt` | `android info` output |
| `.gradle/init.gradle.kts` | The init script injected by the CLI |
