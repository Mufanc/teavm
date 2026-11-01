# Mufanc TeaVM fork

This fork starts from upstream `0.16.0` (`e87d452e4`) and uses version
`0.16.0-mufanc.1`. It keeps a minimal set of general fixes for a JS consumer.

## Changes

- Backport upstream wait timeout/interruption and listener cleanup fixes from
  Ethan Resnick (PRs #1301 and #1302), preserving original authors and coauthors
  with `git cherry-pick -x`. Adapt their assertions to this release's JUnit 4.
- Retain remaining monitor entry waiters when handing the lock to one thread.
- Treat zlib buffer exhaustion as a no-progress result and support empty
  inflater input, while retaining errors for corrupt streams.

## Consumption

Use the source as a Git submodule and substitute TeaVM modules through Gradle
`includeBuild`, or publish project-local Maven artifacts:

```sh
./gradlew publishToMavenLocal -Dmaven.repo.local="$PWD/build/fork-maven"
```

## Scope and validation

Random sequences, JS float precision, and number formatting retain upstream
0.16.0 behavior. Full JVM compatibility and Wasm support are outside this
minimal fork's validation scope. Game-specific adapters belong in the consumer.

The retained patches have targeted end-to-end regression tests. Run tests with
`teavm.tests.js=true`, `teavm.tests.wasm-gc=false`, `teavm.tests.c=false`, and
`teavm.tests.optimized=true` to check optimized and unoptimized JS output.
The complete upstream suite has not been run for this reduced source state.

Validation of the reduced source: 18 Object/DeflateStream test methods passed
on JS with optimized and unoptimized output. Core/classlib main checkstyle and
test checkstyle passed.
