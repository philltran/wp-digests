# #82364: Fix: Responsive horizontal orientation does not override a vertical layout.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jorgefilipecosta
- **Labels:** `[Type] Bug`, `[Package] Block editor`
- **Merged:** [`40cd9bb`](https://github.com/WordPress/gutenberg/commit/40cd9bbcdbbc6b151f9fd4770c231a6356ce4214)
- **Discussion:** [#82364](https://github.com/WordPress/gutenberg/pull/82364) · 3 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Fixes responsive layout overrides for Flex-based blocks (Group Stack, Row, etc.): when a base layout is `vertical` and a tablet/mobile viewport override sets `orientation` to `horizontal`, the block previously stayed vertical. The viewport rule emitted only the justification, so the base `flex-direction: column` kept applying. The fix emits an explicit `flex-direction: row` in the viewport rule, on both the PHP frontend output and the editor.

## Impact

- **Site owners / content authors:** Blocks configured with a horizontal orientation on tablet or mobile over a vertical base layout now render side by side on the frontend and in the editor preview. Existing content with such overrides will change appearance, matching what the editor UI already implied.
- **Plugin & theme developers:** No API changes. If you worked around the bug with custom CSS (e.g. forcing `flex-direction: row` on a viewport), that may now be redundant. Visual regression tests covering responsive flex layouts may need updated baselines.
- **Headless & REST consumers:** No action required; there is no schema change.
- **Hosting & platform:** No action required.

## Technical details

The diff adds one new declaration in each of the two layout implementations:

- `lib/block-supports/layout.php`, in `gutenberg_get_layout_style()`: inside the `'horizontal' === $layout_orientation` branch, if `$viewport_overrides` is non-null and `$has_viewport_property_override( 'orientation' )` is true, it pushes a `flex-direction: row` declaration onto `$layout_styles` for the selector.
- `packages/block-editor/src/layouts/flex.jsx`, in `getLayoutStyle`: inside the `orientation === 'horizontal'` branch, `if ( hasViewportOverride( 'orientation' ) ) rules.push( 'flex-direction: row' )`.

Because `row` is the flex default, base horizontal layouts still omit the declaration (preserving backwards-compatible output). It is emitted only when the viewport override itself specifies `orientation`; an override that changes only `justifyContent` on a horizontal layout does not add it.

Expected output examples from the tests:

```css
/* vertical base, viewport orientation: horizontal, justifyContent: left */
.wp-layout{flex-direction:row;justify-content:flex-start;}

/* horizontal base, viewport overrides only justifyContent: right */
.wp-layout{justify-content:flex-end;}
```

Tests were added in `phpunit/block-supports/layout-test.php` and `packages/block-editor/src/layouts/test/flex.jsdom.test.jsx`, plus a `packages/block-editor/CHANGELOG.md` entry. A `backport-changelog/7.2/13370.md` file links the change to wordpress-develop PR #13370, indicating a planned core backport targeting 7.2.

## Contribution

Authored by @jorgefilipecosta to fix issue #82266, with AI-assisted drafting disclosed in the PR. The discussion is bot-only (CodeRabbit reported no actionable comments and rated merge risk minimal); the props bot also credits @talldan and @firdaus666 from the linked issue. A core backport is tracked through the 7.2 backport-changelog entry.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
