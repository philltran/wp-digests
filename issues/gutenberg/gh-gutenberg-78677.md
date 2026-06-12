# #78677: Fix: Custom HTML block preview keeps expanding when iframe uses height:100vh

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @hbhalodia
- **Labels:** `[Type] Bug`, `[Package] Components`, `[Block] HTML`
- **Merged:** [`2cbccc1`](https://github.com/WordPress/gutenberg/commit/2cbccc116bafb32fe060a68372cf8e0e76356125)
- **Discussion:** [#78677](https://github.com/WordPress/gutenberg/pull/78677) · 5 comments · 0 reactions

## Summary

Fixes an infinite resize loop in the Custom HTML block preview when user-supplied HTML contains iframes using viewport-relative CSS units like `height: 100vh`. The SandBox component's sandbox stripping logic previously contained a malformed regex that failed to strip these values, causing the preview iframe to continuously expand based on its own calculated height. With the corrected pattern, viewport units are properly stripped before the first measurement, stabilizing the preview layout and restoring editor performance.

## Impact

- **Theme & plugin developers**: No breaking changes or API shifts. Developers embedding iframes in Custom HTML blocks will see stable previews without needing to adjust CSS or add workarounds.
- **Site editors/WYSIWYG users**: Affected by a UX fix; preview panes no longer exhibit runaway vertical scrolling or performance degradation during editing.
- **Action required**: None. The fix applies transparently to existing block editor instances and requires no configuration or migration.

## Technical details

- **File**: `packages/components/src/sandbox/index.tsx`
- **Symbol**: `VIEWPORT_UNIT_VALUE_REGEX` (exported for testing)
- **Root cause**: The original regex `/^\\d+(vw|vh|...)$/` contained a double backslash (`\\d`) in the source representation, matching a literal `\d` string rather than a digit. Consequently, viewport-relative units were never matched, leaving them intact inside the sandbox iframe.
- **Behavioral change**: The `SandBox` resize observer measures `document.body.getBoundingClientRect()`, posts it to the parent, and resizes the iframe. With `100vh` still applied, each resize recalculates to a larger pixel height, triggering an infinite feedback loop. Stripping the unit breaks this cycle.
- **Diff details**: The pattern was updated to `/^\d*\.?\d+(?:vw|vh|svw|lvw|dvw|svh|lvh|dvh|vi|svi|lvi|dvi|vb|svb|lvb|dvb|vmin|svmin|lvmin|dvmin|vmax|svmax|lvmax|dvmax)$/`. This correctly matches integers (`100vh`), decimals (`50.5vh`, `.5vh`), and all modern viewport units.
- **Serialization guard**: The regex is duplicated inline inside the serialized `observeAndResizeJS` function that gets embedded into the iframe's `srcdoc`. A new unit test verifies that `VIEWPORT_UNIT_VALUE_REGEX.source` matches exactly what appears in the sandbox srcdoc to prevent future drift.

## Contribution

Opened by @hbhalodia and merged via commit 2cbccc1. Review was led by @ciampo, with feedback from @t-hamano noting the issue predates WordPress 7.0 and did not require backporting. The PR was rebased onto trunk and a changelog entry was moved to the `Unreleased` section before merge. No alternative architectural approaches were discussed; the resolution centers on correcting the regex escape sequence and adding explicit unit tests for decimal viewport values.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
