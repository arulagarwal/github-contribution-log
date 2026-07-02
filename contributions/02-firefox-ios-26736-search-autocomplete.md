[← Contribution Log index](../README.md)

# Contribution 2: Wrong or no option is autocompleted in the search bar

**Contribution Number:** 2

**Student:** Arul Agarwal ([@arulagarwal](https://github.com/arulagarwal))

**Issue:** [mozilla-mobile/firefox-ios#26736](https://github.com/mozilla-mobile/firefox-ios/issues/26736)

**Fork:** _pending — created in Phase II (`arulagarwal/firefox-ios`)_

**Status:** **Phase I complete** — issue claimed via [comment (2026-07-02)](https://github.com/mozilla-mobile/firefox-ios/issues/26736#issuecomment-4861353461); starting Phase II (reproduce & plan).

---

## Why I Chose This Issue

This is my second contribution cycle, and I chose it to move from a niche cross-platform framework (Skip) into a **large, high-impact production iOS app**: Firefox for iOS (Mozilla, ~13k★). It aligns directly with my mobile focus — native iOS/Swift — and Mozilla explicitly labels the issue `Contributor OK`, so external contributions are welcomed through a clear, documented process.

I chose this specific issue because it is a **self-contained selection-logic bug** rather than a sprawling feature: the search bar autocompletes the wrong suggestion (or none) when a suggestion is tapped, which looks like a wrong-index mapping between the tapped row and the backing suggestions list. That kind of precise, testable defect matches the debugging approach I used on my first contribution, while the learning goal — navigating and building a large, unfamiliar UIKit/SwiftUI codebase and its test suite — is exactly the step up I want for this cycle.

---

## Understanding the Issue

### Problem Description

In the URL/search bar, entering a partial query shows a list of suggestions / autocompletions. Tapping the autocomplete "arrow" on a suggestion should fill that suggestion into the search field. Instead, it fills **either none of the suggestions or the wrong one** — the reporter notes it looks like the *wrong index* is autofilled when tapping.

### Expected Behavior

Tapping a suggestion's autocomplete control fills **that** suggestion into the search bar.

### Current Behavior

The wrong suggestion (or no suggestion) is entered — consistent with an off-by-one / mis-mapped index between the tapped UI row and the backing suggestions array.

### Affected Components

_Phase II — to be confirmed._ Likely the search / awesomebar suggestions view and its cell-selection handling (the code that maps a tapped suggestion cell to the value written back into the toolbar / search field).

---

## Reproduction Process

### Environment Setup

_Phase II — in progress._ Will fork `mozilla-mobile/firefox-ios`, clone, and follow the project's setup docs (bootstrap script / Xcode build), documenting any setup challenges and how they were resolved.

### Steps to Reproduce

_From the issue (to be verified on the current build — the maintainer's last note asked whether it still reproduces):_

1. Open Firefox iOS.
2. Tap into the search bar at the top.
3. Enter an incomplete query (e.g. `germ`).
4. A list of suggestions / autocompletions appears.
5. Tap the autocomplete arrow of one of the suggestions.
6. **Expected:** the tapped suggestion is filled into the search bar.
7. **Actual:** none, or the wrong suggestion, is filled.

### Reproduction Evidence

_Phase II — to be added (screen recording / build info once reproduced locally)._

---

## Solution Approach

### Analysis

_Phase II — to be added (root cause: identify where the tapped index is resolved to a suggestion value)._

### Proposed Solution

_Phase II — to be added._

### Implementation Plan

_Phase II — UMPIRE plan to be added (Understand / Match / Plan / Implement / Review / Evaluate), naming the specific files to modify._

---

## Testing Strategy

_Phase III — to be added._ Will follow the project's existing test patterns (e.g. XCUITest / unit tests for the awesomebar / search suggestions) and add a test that exercises the corrected index → suggestion mapping.

---

## Implementation Notes

### Progress

_Phase III — to be added (files modified + key commit hashes)._

### Code Changes

_Phase III — to be added._

### Challenges Faced

_Phase III — to be added._

### Plan Revisions

_To be added as the plan meets reality._

---

## Pull Request

**PR Link:** _Phase IV — to be added._

**Branch:** _to be created in the fork, named after the issue._

**Summary:** _Phase IV — to be added._

**Status:** _Phase IV — to be added._

**Maintainer Feedback Log:**

| Date | Feedback | My Response | Commit / Ref |
|------|----------|-------------|--------------|
| 2026-07-02 | Claimed the issue per the repo's `Contributor OK` process ([comment](https://github.com/mozilla-mobile/firefox-ios/issues/26736#issuecomment-4861353461)) | Starting Phase II (reproduce & plan) | — |

---

## Learnings & Reflections

### Technical Skills Gained

_Phase IV — to be added._

### Challenges Overcome

_Phase IV — to be added._

### What I'd Do Differently Next Time

_Phase IV — to be added._

---

## Resources Used

* [Firefox for iOS repository](https://github.com/mozilla-mobile/firefox-ios)
* [Firefox iOS CONTRIBUTING guide](https://github.com/mozilla-mobile/firefox-ios/blob/main/CONTRIBUTING.md)
