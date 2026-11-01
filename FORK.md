# Mufanc TeaVM fork

This fork starts from the upstream `0.16.0` tag (`e87d452e4`) and uses the
version `0.16.0-mufanc.1`. It contains general Java compatibility fixes only.

## Changes

- Backport upstream PRs [#1300](https://github.com/konsoletyper/teavm/pull/1300),
  [#1301](https://github.com/konsoletyper/teavm/pull/1301), and
  [#1302](https://github.com/konsoletyper/teavm/pull/1302): monitor ownership,
  expired wait listeners, interrupt state, and stale coroutine exceptions.
  Their tests are adapted to the JUnit 4 API used in this release.
- Backport upstream compiler fix
  [`ec91cf0e5`](https://github.com/konsoletyper/teavm/commit/ec91cf0e5c73ad17bdfd998357fdd46af607bd08):
  preserve nonempty Wasm operand stacks across optimized coroutine blocks.
- Grow the coroutine object stack without reflective array allocation.
- Preserve seeded `Random` sequences for bounded integers, booleans and bytes;
  clear cached Gaussian values on reseeding.
- Treat zlib buffer exhaustion as a no-progress result; allow empty inflater
  input without hiding corrupt streams.
- Round JavaScript float arithmetic and casts in the existing strict mode,
  including long-to-float conversion without double rounding. The default
  non-strict mode is unchanged.
- Render optional decimal digits around zero, preserve negative zero, and use
  decimal HALF_UP rounding for fixed-point `Formatter` output.

## Local artifacts

Publish into a project-local Maven repository, without remote publication:

```sh
./gradlew publishToMavenLocal -Dmaven.repo.local="$PWD/build/fork-maven"
```

Add that directory as a Maven repository in a consuming build and use
`0.16.0-mufanc.1` for **all** `org.teavm` artifacts and the Gradle plugin.
Enable the existing strict option when JavaScript float compatibility is needed.

## Scope

This is not a complete Java compatibility implementation. DecimalFormat binary
floating-point rounding and CLDR locale aliases/fallbacks still need separate
work. No application-specific reflection metadata or rendering adapters belong
in this repository.

## Validation

- 156 selected E2E test methods passed with JavaScript and Wasm GC enabled,
  including optimized and unoptimized output. These cover Object, Thread,
  Random, deflate/GZIP, DecimalFormat, Formatter, VM/async behavior, and the
  monitor users ArrayBlockingQueue, ConcurrentHashMap and field updaters.
- 141 selected E2E test methods passed with the C backend enabled, using the
  system clang on macOS and both optimization settings.
- All 213 `:core:test` unit tests passed.
- `:core:checkstyleMain`, `:classlib:checkstyleMain` and `:tests:checkstyleTest`
  passed. Repository-wide `checkstyleMain` also passed with only
  `:tools:idea:instrumentCode` excluded: that unrelated IntelliJ task expects a
  nonexistent `Packages` directory in the cached JDK 25.
- `publishToMavenLocal -Dmaven.repo.local=.../build/fork-maven` succeeded. An
  independent Java consumer compiled and ran against those artifacts on both
  JS and Wasm GC; its monitor, interrupt, random, compression, optional-decimal,
  and float probes matched the JVM results. Node 24 required the per-process
  `--experimental-wasm-exnref` flag; the browser E2E tests used Chrome.

The complete upstream suite and application integration were not run. Local
reports and command logs are under `build/reports/fork-regressions` and
`build/logs`, respectively. Upstream backports preserve their original authors, author dates,
commit messages and coauthor trailers via `git cherry-pick -x`; local adaptations
and additional fixes are separate commits.
