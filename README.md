# Contribution 1: Cannot parse custom colors whose .colorset uses Hex values

**Contribution Number:** 1

**Student:** Arul Agarwal

**Issue:** [skiptools/skip-ui#146](https://github.com/skiptools/skip-ui/issues/146)

**Status:** Phase I Complete

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

*[To be completed in Phase II]*

### Steps to Reproduce

1. Create a new Xcode project and add a custom color to the `.xcassets` folder.
2. In the Attributes Inspector, change the Input Method to "8-bit hexadecimal" and set a hex value.
3. Run the Skip translation process on the module.
4. *[Observe the parser failure or missing color in the resulting Android build]*

### Reproduction Evidence

* **Commit showing reproduction:** [Link to commit in your fork]
* **Screenshots/logs:** [If applicable]
* **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

*[To be completed in Phase II - Will detail the specific JSON parsing structures failing]*

### Proposed Solution

*[To be completed in Phase II - High-level description of adding conditional decoding logic]*

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The `Contents.json` parser crashes or ignores hexadecimal color representations.

**Match:** *[Look for existing hex-parsing logic or alternative JSON decoding strategies in the Skip codebase]*

**Plan:**

1. Locate the Swift struct/class responsible for decoding `.colorset` JSON.
2. Update the `Decodable` implementation to attempt decoding the float dictionary first, and fall back to the hex string if the first attempt fails.
3. Write a helper to convert the 8-bit hex string into the underlying color object Skip uses.
4. Add unit tests with dummy `Contents.json` data containing hex values.

**Implement:** *[Link to your branch/commits as you work]*

**Review:** *[Self-review checklist - does it follow the project's contribution guidelines?]*

**Evaluate:** *[How will you verify it works?]*

---

## Testing Strategy

### Unit Tests

* [ ] Test case 1: Parse standard floating-point `.colorset` to ensure regressions are avoided.
* [ ] Test case 2: Parse an 8-bit hexadecimal `.colorset` successfully.
* [ ] Test case 3: Handle malformed or missing hex values gracefully.

### Integration Tests

* [ ] Integration scenario 1: Verify the translated Compose Multiplatform code reflects the correct hex color.
* [ ] Integration scenario 2

### Manual Testing

*[To be completed in Phase III]*

---

## Implementation Notes

### Week 2 Progress

*[To be filled]*

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

* [Skip UI Documentation](https://skip.tools/)
* [Apple Developer Documentation: Color Set Type](https://www.google.com/search?q=https://developer.apple.com/documentation/xcode/color-set-type)
