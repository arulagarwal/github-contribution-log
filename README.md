# Contribution 1: Cannot parse custom colors whose .colorset uses Hex values

**Contribution Number:** 1

**Student:** Arul Agarwal

**Issue:** [skiptools/skip-ui#146](https://github.com/skiptools/skip-ui/issues/146)

**Status:** Phase II Complete

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

When a custom color in the `.xcassets` directory is set to use hex encoding, the resulting `Contents.json` file changes structure, replacing the nested RGB floats with a single `value` key containing the hex string (e.g., `#04F188`). The current parser does not account for this structural change and fails to load the color.

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

### Unit Tests

* [ ] Test case 1: Parse standard floating-point `.colorset` to ensure regressions are avoided.
* [ ] Test case 2: Parse an 8-bit hexadecimal `.colorset` successfully.
* [ ] Test case 3: Handle malformed or missing hex values gracefully.

### Integration Tests

* [ ] Integration scenario 1: Verify the translated Compose Multiplatform code reflects the correct hex color.
* [ ] Integration scenario 2: Render a mock UI component utilizing a hex-based colorset.

### Manual Testing

*[To be completed in Phase III]*

---

## Implementation Notes

### Week 2 Progress

Set up local development environment, created a feature branch, and successfully reproduced the parser failure by adding a dummy hex `.colorset` to the testing resources. Drafted the UMPIRE solution plan to update the JSON decoding logic. All documentation is being tracked and pushed to arulagarwal/github-contribution-log_2.

### Week 3 Progress

*[To be filled]*

### Code Changes

* **Files modified:** [List]
* **Key commits:** [Links to important commits]
* **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**

* [Date]: [Summary of feedback received]
* [Date]: [How you addressed it]

**Status:** Awaiting Implementation

---

## Learnings & Reflections

### Technical Skills Gained

*[To be completed in Phase IV]*

### Challenges Overcome

*[To be completed in Phase IV]*

### What I'd Do Differently Next Time

*[To be completed in Phase IV]*

---

## Resources Used

* [Skip UI Documentation](https://www.google.com/search?q=https%3A%2F%2Fskip.tools%2F)
* [Apple Developer Documentation: Color Set Type](https://www.google.com/search?q=https%3A%2F%2Fwww.google.com%2Fsearch%3Fq%3Dhttps%3A%2F%2Fdeveloper.apple.com%2Fdocumentation%2Fxcode%2Fcolor-set-type)
