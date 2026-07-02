[← Contribution Log index](../README.md)

# Contribution 3: Bookmarks — graphical inconsistency right after creating a folder in edit mode

**Contribution Number:** 3

**Student:** Arul Agarwal ([@arulagarwal](https://github.com/arulagarwal))

**Issue:** [mozilla-mobile/firefox-ios#24741](https://github.com/mozilla-mobile/firefox-ios/issues/24741)

**Fork:** _pending — created in Phase II (`arulagarwal/firefox-ios`)_

**Status:** **Phase I complete** — issue claimed via [comment (2026-07-02)](https://github.com/mozilla-mobile/firefox-ios/issues/24741#issuecomment-4861353761). (Working #26736 first; this cycle is queued.)

---

## Why I Chose This Issue

A second Firefox-iOS bug in my mobile / Swift focus area, and another `Contributor OK` + `Good first issue`, so it fits the same welcomed-contribution process. I picked it because it is a **well-scoped UI-layout bug**: immediately after a new bookmark folder is created while the list is in edit mode, the row's elements overlap / mis-align.

It complements #26736 (selection logic) with a different learning goal — **list / collection-view edit-mode layout** in a production iOS codebase. It also already has useful direction: a maintainer narrowed the cause to *"a problem with the edit state, for cells,"* which gives a concrete starting point for Phase II.

---

## Understanding the Issue

### Problem Description

On a fresh install, opening Bookmarks → tapping **Edit** → adding a new folder and saving it produces a layout where the elements **overlap** right after the folder is created, instead of being laid out correctly.

### Expected Behavior

All elements are correctly displayed / laid out after the new folder is created in edit mode.

### Current Behavior

The elements overlap. A maintainer's investigation notes it appears tied to the **cell edit state**.

### Affected Components

_Phase II — to be confirmed._ Likely the bookmarks list cell layout and its handling of the editing state (the transition into / refresh of edit mode after inserting a new row).

---

## Reproduction Process

### Environment Setup

_Phase II — in progress (shares the `mozilla-mobile/firefox-ios` fork / clone / build from Contribution 2)._

### Steps to Reproduce

1. Fresh-install the app.
2. Navigate to Hamburger menu → Bookmarks.
3. Tap the **Edit** button.
4. Add a new Folder and save it.
5. Check the layout right after the new folder is created.
6. **Expected:** all elements are correctly displayed.
7. **Actual:** the elements overlap.

### Reproduction Evidence

_Phase II — to be added._

---

## Solution Approach

### Analysis

_Phase II — to be added (build on the maintainer's "cell edit state" lead)._

### Proposed Solution

_Phase II — to be added._

### Implementation Plan

_Phase II — UMPIRE plan to be added, naming specific files to modify._

---

## Testing Strategy

_Phase III — to be added (follow the project's existing bookmarks / UI test patterns)._

---

## Implementation Notes

### Progress

_Phase III — to be added._

### Code Changes

_Phase III — to be added._

### Challenges Faced

_Phase III — to be added._

### Plan Revisions

_To be added._

---

## Pull Request

**PR Link:** _Phase IV — to be added._

**Branch:** _to be created in the fork, named after the issue._

**Summary:** _Phase IV — to be added._

**Status:** _Phase IV — to be added._

**Maintainer Feedback Log:**

| Date | Feedback | My Response | Commit / Ref |
|------|----------|-------------|--------------|
| 2026-07-02 | Claimed the issue per the repo's `Contributor OK` process ([comment](https://github.com/mozilla-mobile/firefox-ios/issues/24741#issuecomment-4861353761)) | Queued behind #26736 | — |

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
