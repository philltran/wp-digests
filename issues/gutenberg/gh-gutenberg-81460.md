# #81460: Normalize block gap values in layout

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Enhancement`, `[Feature] Layout`
- **Merged:** [`d90ebd7`](https://github.com/WordPress/gutenberg/commit/d90ebd76e83d84e47c07e8e81c89b8d53fc4d071)
- **Discussion:** [#81460](https://github.com/WordPress/gutenberg/pull/81460) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Gutenberg's layout block support now normalizes block gap values so that they are always strings, numeric zero (converted to the string `'0'`), or arrays containing only those values. Malformed input such as objects, booleans, non-zero numbers, nested arrays, or empty/whitespace-only strings is dropped, and the layout code falls back to the default gap instead of triggering PHP warnings or fatals. This closes a long-standing issue (#52099).

## Impact

- **Plugin & theme developers:** Code that passes `blockGap` through block attributes, `theme.json`/global styles, or `__experimentalDefault` in block supports, or that calls `gutenberg_get_layout_style()` directly, now gets sanitized values. Numeric `0` (int or float) is emitted as `gap: 0`. Any other non-string scalar (e.g. `1`, `true`) is discarded rather than serialized.
- **Site owners:** No visible change for valid values (`px`, `vw`, `rem`, presets, `0`). Sites with malformed stored gap values should see fallback gaps instead of PHP errors.
- **Core backport:** A backport changelog entry targets WordPress 7.2 (wordpress-develop PR 13011), so the same normalization is expected in core's `wp_*` layout functions.
- No action required for well-formed values.

## Technical details

Changes are in `lib/block-supports/layout.php`:

- `gutenberg_sanitize_block_gap_value()` is now recursive over arrays. Each entry is sanitized; entries that don't resolve to a string are `unset()`, and an array that ends up empty returns `null`. Scalars: int/float equal to zero return `'0'`; any other non-string returns `null`; empty or whitespace-only strings return `null`. The existing regex `%[\\\(&=}]|/\*%` still rejects unsafe strings.
- `gutenberg_get_layout_style()` now sanitizes `$gap_value` and `$fallback_gap_value` on entry (`$fallback_gap_value = ... ?? '0.5em'`), because the function has direct callers. Docblock types are widened to `string|string[]|int|float|null`, with the note that only zero is accepted as numeric.
- In the combined-gap branch, the check `null !== $gap_value` became `'' !== $gap_value`, so an empty combined gap no longer emits a `gap` declaration.
- `gutenberg_render_layout_support_flag()` sanitizes `__experimentalDefault` from block supports (falling back to `'0.5em'`). Global-styles gap resolution replaced the `??` chain with a loop over candidates (variation, per-block, global `spacing.blockGap`), picking the first that survives sanitization. Previously an invalid value in a higher-priority slot would shadow valid lower-priority ones.

Behavior change for arrays: previously invalid entries were set to `null` and retained keys; now they are removed (e.g. `array('top' => array('1rem'), 'left' => '2rem')` becomes `array('left' => '2rem')`).

PHPUnit tests in `phpunit/block-supports/layout-test.php` add a data provider for the sanitizer and new flex/grid layout cases for malformed gaps (expected output `gap:0.5em 2rem;`) and an empty gap (no output).

## Contribution

Authored by @tellthemachines as overdue cleanup closing #52099, with @ramonjd and @andrewserong credited by props-bot. The PR description discloses use of AI tooling (codex). Discussion in the thread is limited to bot comments, including an unrelated flaky e2e test report on the interactivity router.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
