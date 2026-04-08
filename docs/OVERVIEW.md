# literal-identifiers-test-case: Overview

## Purpose

This repository is a minimal Scala test case that reproduces a bug in the Scala 2.12 compiler (scalac) related to literal identifiers (backtick-quoted field names) in case classes. When two case class fields both use backtick-quoted names that share the same prefix (e.g., `` `CMMC Capability Number` `` and `` `CMMC Capability` ``), the compiler incorrectly maps field accessors, causing the second field to return the value of the first. Tracked as [scala/bug#12014](https://github.com/scala/bug/issues/12014).

## Architecture

```
literal-identifiers-test-case/
├── build.sbt                          # Project build definition (Scala 2.12.10, ScalaTest 3.1.1)
├── project/
│   └── build.properties              # sbt version (1.2.8)
└── src/test/scala/net/bstjohn/
    └── CaseClassesSpec.scala         # All test cases and case class definitions
```

The entire codebase lives in a single test file. There is no production source code — this project exists solely to reproduce and document the compiler bug.

## Key Patterns

### The Bug: `BadRecord`

```scala
case class BadRecord(
  `CMMC Capability Number`: String,
  `CMMC Capability`: String
)
```

When field names are backtick-quoted and one is a prefix of another (both contain `"CMMC Capability"`), the scalac compiler generates incorrect accessor code. Accessing `record.\`CMMC Capability\`` returns the value stored in `\`CMMC Capability Number\`` instead.

**Root cause**: scalac's case class desugaring appears to match accessor methods by a prefix/substring of the mangled identifier name, causing collisions when one literal identifier is a prefix of another.

### Workarounds: `GoodRecord1` and `GoodRecord2`

Two workarounds are demonstrated:

| Approach | Example | Notes |
|----------|---------|-------|
| Replace spaces with underscores | `CMMC_Capability` | No backticks needed |
| Add a distinguishing character | `` `CMMC xCapability` `` | Breaks the prefix match |

Both workarounds prevent the name collision and produce correct accessor behavior.

### Test Structure

Tests use ScalaTest `AnyFunSpec` with `should` matchers. There are three `it` blocks:
1. **BadRecord** — expected to **fail** (demonstrates the bug)
2. **GoodRecord1** — passes (underscore workaround)
3. **GoodRecord2** — passes (extra character workaround)

Tests are forked (`fork in Test := true`) to ensure a clean JVM per run.

## Configuration

| Setting | Value |
|---------|-------|
| Scala version | 2.12.10 |
| sbt version | 1.2.8 |
| ScalaTest | 3.1.1 |
| Test scope | `src/test/` only — no main sources |
| Fork tests | `true` |

## Running

```sh
sbt test
```

The `BadRecord` test will fail by design, demonstrating the bug. `GoodRecord1` and `GoodRecord2` tests pass.
