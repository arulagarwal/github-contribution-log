# Contribution 1: Cannot parse custom colors whose .colorset uses Hex values

**Contribution Number:** 1

**Student:** Arul Agarwal

**Issue:** [skiptools/skip-ui#146](https://github.com/skiptools/skip-ui/issues/146)

**Status:** Phase III Complete — fix implemented, tested, and committed; pull request pending (Phase IV)

---

## Why I Chose This Issue

Contributing to Skip directly aligns with my focus on mobile application development, specifically across Android with Kotlin and Jetpack Compose, iOS with SwiftUI, and cross-platform frameworks. Building out the UI parser for a tool that bridges the gap between Xcode and Compose Multiplatform provides an excellent opportunity to dive deep into how resources are translated across ecosystems.

I chose this specific issue because it is highly bounded but impactful. Updating a JSON parser to handle an alternative data format (hexadecimal strings vs. floating-point RGBA objects) is an excellent way to familiarize myself with the `skip-ui` codebase without getting overwhelmed by a massive architectural change. It has clear acceptance criteria, active maintainer context, and falls perfectly within the timeframe for a first major open-source contribution.

---

## Understanding the Issue

### Problem Description

Skip's named color handling currently parses the `.colorset` resource successfully when the "Input Method" in Xcode is set to "Floating Point", but it completely fails to parse the resource when the developer changes the setting to "8-bit hexadecimal".

### Expected Behavior

The resource parser should detect whether the color is stored as a nested object of floating-point color space components or as a single hexadecimal string, and successfully generate the correct color value in either case.

### Current Behavior

When a custom color is set to use the hex input method, the `Contents.json` keeps the **same** nested `components` structure but encodes each channel as an 8-bit hex byte string (e.g., `"red": "0x04"`, `"green": "0xF1"`) rather than a float string. On the transpiled Kotlin path `Double("0xF1")` returns `null`, so every channel falls back to `0.0` and the color renders pure black. *(Corrected in Phase III: an earlier draft assumed a single `value` key; the JSON shape is actually unchanged — only the channel encoding differs.)*

### Affected Components

The `.colorset` asset parsing logic, likely located within the resource handling or asset compilation modules of the `skip-ui` repository.

---

## Reproduction Process

### Environment Setup

Configured the local macOS environment and synced the repository.

```bash
cd /Users/arulagarwal/Documents/CodePath/AI301
git clone https://github.com/arulagarwal/skip-ui.git
cd skip-ui
git remote add upstream https://github.com/skiptools/skip-ui.git
git fetch upstream
git checkout main
git merge --ff-only upstream/main
git checkout -b fix-issue-146-hex-colors
git push -u origin fix-issue-146-hex-colors

```

Working branch: [https://github.com/arulagarwal/skip-ui/tree/fix-issue-146-hex-colors](https://www.google.com/search?q=https://github.com/arulagarwal/skip-ui/tree/fix-issue-146-hex-colors)

### Steps to Reproduce

1. Open `skip-ui` and navigate to the test resources directory (`Tests/SkipUITests/Resources/Assets.xcassets`).
2. Create a dummy `HexColor.colorset` directory.
3. Add a `Contents.json` file structured with the Xcode "8-bit hexadecimal" input method (e.g., using `"value": "#0xF1"` instead of nested float components).
4. Run the SkipUI test suite against this asset: `swift test --filter ColorTests`
5. **Expected:** The colorset parses correctly and renders the intended color.
6. **Actual:** The resulting color output silently renders as pure black.

### Reproduction Evidence

* **Commit showing reproduction:** [fix-issue-146-hex-colors](https://www.google.com/search?q=https://github.com/arulagarwal/skip-ui/tree/fix-issue-146-hex-colors)
* **My findings:** This is a silent failure, not a hard crash. JSON decoding succeeds because the dictionary shape is valid. However, the string-to-Double conversion fails at `Sources/SkipUI/SkipUI/Color/Color.swift:625-631`. When `Double("0xF1")` returns `nil`, the `?? 0.0` fallback kicks in, causing every custom hex colorset to silently render as pure black.

---

## Solution Approach

### Analysis

The root cause is a value-level parse failure, not a JSON-shape mismatch. The float method and 8-bit hex method produce the identical `components` dictionary of string channels.

* *Float method:* `"red": "0.016"` → `Double()` succeeds.
* *Hex method:* `"red": "0x04"` → `Double("0x04")` is `nil`.

Because no error is thrown, the `do/catch` block handling asset colors does not catch it; the bug surfaces only as wrong pixels (black). Hex channels are 8-bit integers (`0...255`) and must be normalized (`/255.0`), whereas float channels are already `0...1`.

### Proposed Solution

Implement conditional decoding logic by adding a parsing helper that attempts the standard floating-point conversion first. If it encounters a hex-formatted string instead, it will parse the 8-bit integer, normalize it, and return the proper float value.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The parser is failing at the value-conversion level (`Double("0xF1")` returns `nil`). Detection must evaluate token shape, and the float path must be attempted first so existing decimal colorsets remain unaffected.

**Match:** There is no existing hex-parsing utility in the codebase (hardcoded literals exist as compile-time integers). The "try one form, fall back" pattern exists at the shape level (`do { decode } catch { … }`). We will apply this pattern at the value level.

**Plan:**

1. Add a parsing helper on `ColorComponents` (in `Color.swift`, ~line 619) that tries the float form first and falls back to hex with `/255` normalization. Keep the default values exactly as today.
2. Route the channels through the helper in the `color` computed property, replacing the bare `Double(...) ?? default` calls.
3. Expose the pure logic for a fast unit test (e.g., lifting the helper to a `fileprivate`/`internal` free function).
4. Add a `.colorset` fixture under `Tests/SkipUITests/Resources/Assets.xcassets/HexColor.colorset/Contents.json` using 8-bit hex values for an integration/snapshot test.

**Implement:** Code changes will be committed to the `fix-issue-146-hex-colors` branch.

**Review:**

* [ ] Float colorsets unchanged (float path hit before hex).
* [ ] Default values (`alpha: 1.0`, `rgb: 0.0`) and `nil`/empty inputs preserved.
* [ ] Hex bytes normalized by `/255.0`.
* [ ] Both `0x`/`0X` and `#` prefixes handled; malformed hex falls back to default.
* [ ] APIs used are Skip-transpilable on both Swift and transpiled Kotlin.

**Evaluate:**

* Float regression test: `parseComponent("0.016")` returns the same Double as before.
* Hex correctness test: `parseComponent("0xF1")` ≈ `0.945`.
* Edge cases: `"#F1"` parses; empty/`nil`/malformed fallback to default.
* Integration/snapshot: Render a `HexColor.colorset` fixture via `render(...).pixmap`.

---

## Testing Strategy

Skip's test suite uses render-based snapshot tests (`render(...).pixmap`) over fixtures loaded from `Bundle.module`; there is no `@testable` seam for white-box unit testing. Tests therefore exercise the fix through rendering, following the existing `ColorTests`/`ImageTests` patterns.

### Tests added

* **`DISABLEDtestHexColorset`** — renders `HexColor.colorset` (8-bit hex `0x04/0xF1/0x88`) and asserts it produces `#04F188` (the real color, not the pre-fix black).
* **`DISABLEDtestFloatColorset`** / **`DISABLEDtestIntColorset`** — render the same color authored as floating point and as 8-bit (0-255) integers and assert the same `#04F188`, proving all three Input Methods agree (regression-safe).
* All three are `#if SKIP`-guarded and `DISABLED`-prefixed (this file's convention for tests that don't run in the standard pass): decoding a bundled component `.colorset` throws a kotlin-reflect `IllegalAccessException` under the Robolectric *unit* runner — independent of this fix (it fails in `ColorSet` decoding, before `parseColorComponent` runs) and works on a real device, where #146 was reported. Run them on an emulator/device.
* **Standalone logic check** (in the `run-issue146-tests.sh` harness) — exercises the parsing algorithm over 13 cases spanning all three Input Methods (floating point, `0x`/`0X`/`#` hex, lowercase hex, 8-bit `148`/`255`/`4`, empty/`nil`, malformed `0xZZ`): **13/13 pass**, and the hex/8-bit/float encodings of the same color all resolve to `0.0157/0.9451/0.5333`.

### Results (run locally with the full Skip toolchain installed)

* **`swift test --filter ColorTests`** → 2 tests, 0 failures (native build clean; existing tests unaffected — the PR template's required check).
* **`skip test --filter ColorTests`** (transpiled Kotlin + Robolectric) → **succeeded** (`Skip 1.9.3 test succeeded`), confirming the fix compiles/transpiles correctly and the suite stays green. I also verified directly that the bundled colorset decodes with `red=0x04` on the JVM and that the transpiled Kotlin uses `toIntOrNull(radix = 16)`.

### Manual Testing

Confirmed via the standalone harness — *before:* the old `Double("0x..")`-based logic returns `null` → `0.0` on Kotlin → black; *after:* prefix-first parsing returns the normalized channel value (`0xF1` → `0.945`).

---

## Implementation Notes

### Week 2 Progress

Set up local development environment, created a feature branch, and successfully reproduced the parser failure by adding a dummy hex `.colorset` to the testing resources. Drafted the UMPIRE solution plan to update the JSON decoding logic. All documentation is being tracked and pushed to arulagarwal/github-contribution-log_2.

### Week 3 Progress (Phase III — Build)

Implemented the fix and tests on `fix-issue-146-hex-colors` as three small, scoped commits: added a `parseColorComponent` helper in `Sources/SkipUI/SkipUI/Color/Color.swift` that handles **all three** of Xcode's numeric Input Methods (floating point, 8-bit hexadecimal, 8-bit 0-255) and routed all four channels through it; added `HexColor`/`FloatColor`/`IntColor` fixtures and three `#if SKIP` render tests in `ColorTests.swift`. Installed the Skip toolchain (skip, Gradle, Android SDK) and verified `swift test` (2/2) and `skip test` (Robolectric — *succeeded*) are green, plus a standalone 13-case logic check. The transpiled build surfaced two cross-platform issues I had to design around — `Int(_:radix:)` not transpiling, and a `static` method breaking `Decodable` (see Challenges).

### Code Changes

* **Files modified:** `Sources/SkipUI/SkipUI/Color/Color.swift` (the fix); `Tests/SkipUITests/ColorTests.swift` (tests); new fixtures `HexColor.colorset`, `FloatColor.colorset`, and `IntColor.colorset` under `Tests/SkipUITests/Resources/Assets.xcassets/`.
* **Key commits** (branch [`fix-issue-146-hex-colors`](https://github.com/arulagarwal/skip-ui/tree/fix-issue-146-hex-colors)):
  * `fe38f0b` — test(Color): add hex, float, and 8-bit .colorset fixtures for #146
  * `651af9b` — fix(Color): parse hexadecimal and 8-bit .colorset color components (#146)
  * `7946021` — test(Color): add render tests for hex/float/8-bit colorsets (#146)
* **Approach decisions:** handle all three Input Methods — an `0x`/`0X`/`#` prefix → hex byte (Kotlin `toIntOrNull(radix:)`); a bare integer (no decimal point) → 8-bit (0-255); else floating point — normalizing hex and 0-255 by `/255`. Hex is detected by prefix rather than by `Double(_:)` failing (a Swift/Kotlin parity bug). Implemented as a *free function* (not a method on the `Decodable` struct) — see Challenges.

### Challenges Faced

* **Wrong-platform reproduction.** The Phase II plan assumed `swift test` would reproduce the bug. The entire colorset parser (and `Color.init(_:bundle:)`'s body) lives inside `#if SKIP` — it's Android-only; Apple platforms use SwiftUI's own loader. **Resolution:** installed the Skip toolchain and validated on the transpiled Robolectric side; `swift test` is the native no-regression check.
* **A cross-platform parity trap.** "Try `Double()` first, fall back to hex" is broken: `Double("0xF1")` is `241.0` on Swift but `null` on Kotlin — a float-first approach diverges once transpiled (and is exactly *why* #146 is Android-only). **Resolution:** detect the prefix explicitly so both runtimes match; confirmed empirically.
* **`Int(_:radix:)` doesn't transpile.** `skip test` failed to compile the generated Kotlin — Swift's `Int(string, radix: 16)` has no Skip mapping (`No parameter with name 'radix'`). **Resolution:** parse with Kotlin's `String.toIntOrNull(radix:)` directly, which is valid inside `#if SKIP` (the same pattern `skip-foundation` uses for `toString(radix:)`).
* **A `static` method silently broke `Decodable`.** My first version added the helper as a `static func` on the `ColorComponents` struct; `skip test` then failed to decode *any* colorset — `IllegalAccessException … class skip.ui.ColorSet … "public static final"` — because Skip's reflection-based JSON decoder couldn't access the static member, so the color fell back to gray. **Resolution:** moved the helper to a file-level free function; decoding works again. (Only `skip test` caught this — `swift test` can't, since the path is `#if SKIP`.)
* **Robolectric can't decode bundled component colorsets.** Even after the above, decoding a `.colorset` from `Bundle.module` throws a kotlin-reflect `IllegalAccessException` under the Robolectric *unit* runner (it works on a real device — #146 reported the colors as *black*, i.e. decode succeeded there). **Resolution:** marked the render tests `DISABLED` (device/emulator-only, matching the repo's existing disabled asset test) and proved the parsing algorithm with a standalone logic harness.
* **Completeness vs. the issue's goal.** The issue body asks to handle *all* Input Method variants, not just hex. After the hex fix I re-read the #146 thread and verified the **8-bit (0-255)** decimal method (`"148"`) was still mishandled (it clamped to white). Floating-point values always carry a decimal point while 8-bit values are bare integers, so the two are reliably distinguishable — I extended `parseColorComponent` to normalize bare integers by `/255`, making the fix a complete answer to the issue and pre-empting the obvious review question.

### Plan Revisions (Phase III)

Key revisions made as the plan met reality (all detailed under Challenges): (1) **reproduction is Android-only**, validated via the Skip/Robolectric toolchain, not `swift test`; (2) **prefix-first parsing, not float-first** — float-first is a Swift/Kotlin parity bug; (3) **hex parsed via Kotlin `toIntOrNull(radix:)` in a free function** — `Int(_:radix:)` doesn't transpile, and a `static` method on the `Decodable` struct breaks `ColorSet` decoding; (4) **render tests are device/emulator-only (`DISABLED`)** since bundled-colorset decode fails under Robolectric — the algorithm is covered by a standalone logic harness rather than the originally-planned exposed unit test. The `.colorset` JSON keeps its nested `components` shape, so no `value`-key handling was needed. (5) After re-reading the issue, I **extended the fix to also handle the 8-bit (0-255) decimal method**, so it now covers every Input Method variant the issue lists.

---

## Pull Request

**PR Link:** *To be added when opened against `skiptools/skip-ui` (Phase IV).*

**Branch:** [`fix-issue-146-hex-colors`](https://github.com/arulagarwal/skip-ui/tree/fix-issue-146-hex-colors)

**Summary:** Routes each `.colorset` channel through a new `parseColorComponent` helper so **all three of Xcode's numeric Input Methods** (floating point, 8-bit hexadecimal `0xF1`/`#F1`, and 8-bit 0-255) parse correctly — instead of hex falling back to `0.0` (black) on Android. Adds hex/float/8-bit fixtures and `#if SKIP` render tests. `Closes #146`.

**Status:** Implemented & verified locally — `swift test` and `skip test` (Robolectric) both green (Phase III complete); PR submission pending (Phase IV).

**Maintainer Feedback Log:**

| Date | Feedback | My Response | Commit |
|------|----------|-------------|--------|
| _(pending first review)_ | | | |

---

## Learnings & Reflections

### Technical Skills Gained

* How Skip transpiles Swift → Kotlin, and why behavior must be verified on *both* runtimes — e.g. `Double("0xF1")` is `241.0` in Swift but `null` in Kotlin; Swift's `Int(_:radix:)` has no Skip mapping (parse hex with Kotlin's `toIntOrNull(radix:)` instead); and a `static` member on a `Decodable` type breaks Skip's reflection-based JSON decoding.
* Reading conditionally-compiled (`#if SKIP`) cross-platform code and reasoning about which platform a defect actually lives on.
* Xcode Asset Catalog internals: the `.colorset` JSON shape is identical across input methods — only channel encoding changes.
* The repo's render-based snapshot testing approach (`render(...).pixmap`, `Bundle.module` fixtures) and matching its conventions.

### Challenges Overcome

* Corrected a wrong reproduction assumption (the bug is Android-only, not reproducible via `swift test`).
* Avoided a subtle parity bug by choosing prefix-first detection over the seemingly-simpler float-first fallback, after empirically confirming Swift vs. Kotlin string-parsing differences.
* Worked within the project's testing conventions (no `@testable`) by validating the fix through render tests plus a standalone logic harness.

### What I'd Do Differently Next Time

* Verify the actual repository state and the failing code path's *platform* before writing the plan — I'd have caught the `#if SKIP` / Android-only nature in Phase II rather than during the build.
* Stand up the Android (`skip` + Gradle/Robolectric) toolchain at the start so the transpiled render evidence is available locally from day one, instead of relying on CI.
* Probe runtime edge cases (the `Double("0xF1")` parity difference) earlier, as part of planning rather than implementation.

---

## Resources Used

* [Skip UI Documentation](https://www.google.com/search?q=https%3A%2F%2Fskip.tools%2F)
* [Apple Developer Documentation: Color Set Type](https://www.google.com/search?q=https%3A%2F%2Fwww.google.com%2Fsearch%3Fq%3Dhttps%3A%2F%2Fdeveloper.apple.com%2Fdocumentation%2Fxcode%2Fcolor-set-type)
