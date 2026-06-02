# #78672: Hide paragraph Drop Cap and Fit Text controls when a state is selected

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Paragraph`, `[Package] Block editor`
- **Merged:** [`71bab86`](https://github.com/WordPress/gutenberg/commit/71bab86d18f4a39887524e43d29e690817b7f1ec)
- **Discussion:** [#78672](https://github.com/WordPress/gutenberg/pull/78672) · 7 comments · 0 reactions

## Summary

The Paragraph block's **Drop Cap** toggle and **Fit Text** control are now hidden in the block inspector whenever a responsive style state (Mobile or Tablet) is active. Both features lack per-viewport implementation, so surfacing the controls in those states was misleading — changes appeared to apply but had no effect. The fix gates both controls on the absence of a selected style state rather than adding placeholder UI or disabling the inputs.

## Impact

**Plugin & theme developers**
- No breaking changes or deprecated APIs.
- Developers who extend the Paragraph block's `InspectorControls` with typography-adjacent controls should be aware of the pattern: consumer controls are not automatically hidden in responsive states. If your control is also viewport-unaware, apply the same guard.

**Site owners / content editors**
- Drop Cap and Fit Text controls simply disappear when Mobile or Tablet is selected in the block inspector. Switching back to the default (desktop) state restores them.

**All other audiences**
- No action required.

## Technical details

The implementation relies on `hasSelectedStyleState`, a selector introduced in the prerequisite PR #78670 (in `@wordpress/block-editor`). In the Paragraph block's edit component (`packages/block-library/src/paragraph/edit.js`), both the Drop Cap `ToggleControl` and the Fit Text control are now conditionally rendered: they are omitted from `InspectorControls` when `hasSelectedStyleState` returns `true`.

Before this change, both controls rendered unconditionally in the Typography panel regardless of which responsive state was active. After:

```jsx
// Conceptual — controls are only rendered when no style state is selected
{ !hasSelectedStyleState && (
  <DropCapControl … />
) }
{ !hasSelectedStyleState && (
  <FitTextControl … />
) }
```

The bundle impact is minimal: `build/scripts/block-editor/index.min.js` +46 B, `build/scripts/block-library/index.min.js` +16 B. No new hooks, filters, block.json fields, REST schema changes, or DB changes are introduced.

## Contribution

PR opened and authored by @talldan (talldanwp), stacked on #78670. The initial description covered only Drop Cap; after opening, @talldan self-noted that Fit Text has the same problem and expanded the PR to cover both before requesting review. Co-authored with @tellthemachines (isabel_brison). A brief discussion touched on whether a first-paint regression visible in perf CI was concerning; @talldan assessed it as noise unrelated to these changes. Merged at commit `71bab86`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
