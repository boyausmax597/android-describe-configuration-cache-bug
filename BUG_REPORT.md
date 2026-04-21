`android describe` fails on any project with `org.gradle.configuration-cache=true` in `gradle.properties`.

The CLI injects `.gradle/init.gradle.kts` and registers a `DumpModelTask`. Its `@TaskAction` calls `Task.project` at execution time — both to access `project.logger` and to pass `project` to `ToolingModelBuilder.buildAll(...)`. Gradle 9 forbids this under configuration cache and fails with:

```
Invocation of 'Task.project' by task ':app:dumpAndroidProjectModel' at execution time is unsupported with the configuration cache.
```

Notably, the init script itself contains a comment acknowledging the design flaw:

```kotlin
// This task is problematic because it uses the project instance. We would need a better solution
// longer term by probably embedding the capability in AGP or requesting an API extension with Gradle.
```

Since configuration cache is Gradle's own recommended default and is increasingly enabled out of the box, this blocks `android describe` for a growing share of projects.

## Steps to reproduce

1. `android create empty-activity --name="CC Repro" --output=./repro`
2. Add `org.gradle.configuration-cache=true` to `repro/gradle.properties`
3. `android describe --project_dir=./repro`

## Minimal reproducer

https://github.com/Thomas-Boutin/android-describe-configuration-cache-bug

Includes `repro-output.txt` (CLI failure), `repro-gradle-stacktrace.txt` (full Gradle CC violation), and `workaround-output.txt` (passing run).

## Workaround

```bash
./gradlew --no-configuration-cache \
  --init-script .gradle/init.gradle.kts \
  :app:dumpAndroidProjectModel
```

## Environment

| Field | Value |
|---|---|
| Android CLI | `0.7.15232955` |
| Gradle | `9.1.0` |
| AGP | `9.1.1` |
| OS | macOS Darwin 25.3.0 arm64 |
