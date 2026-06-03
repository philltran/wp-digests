# #78306: Block Inserter: Animate inserter button icon to signal open state.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @juanfra
- **Labels:** `[Type] Enhancement`, `[Feature] Inserter`, `Needs Design Feedback`, `[Package] Block editor`, `[Package] Base styles`
- **Merged:** [`f7b40ab`](https://github.com/WordPress/gutenberg/commit/f7b40ab46f5f0519cffb4f818b361534a81481e2)
- **Discussion:** [#78306](https://github.com/WordPress/gutenberg/pull/78306) · 6 comments · 0 reactions

## Summary

The block inserter toggle button (`+`) now animates its icon to a `×` when the inserter panel is open, providing a clear visual cue that the panel is active and that clicking will close it. The SVG rotates 45° on open using a CSS transition with `cubic-bezier` easing, then rotates back on close. The `isPressed` prop on the toggle `Button` is also wired to stay in sync with `aria-expanded`, correcting an inconsistency in the pressed state signal. This brings the main inserter in line with the existing animation already present on the document tools inserter.

## Impact

**Plugin & theme developers**
- No action required. This is a purely visual change to first-party editor chrome with no API surface.
- If custom admin CSS targets `.block-editor-inserter__toggle svg` or the `.is-pressed` state on the inserter button, the new `transform` and `transition` rules may interact with those overrides.

**All other audiences**
- No action required.

## Technical details

Two coordinated changes land in the `@wordpress/block-editor` package:

**CSS (block-editor style and content sheets)**
The inserter toggle button's SVG icon gains a `transition` property using the same `cubic-bezier` easing already applied in the document tools inserter. When the `.is-pressed` class is present on the button, the SVG receives `transform: rotate(45deg)`, morphing the `+` visually into a `×`. The rule is mirrored in both `style.css`/`style-rtl.css` (editor canvas styles) and `content.css`/`content-rtl.css` (iframed content styles), accounting for the ~90–93 B additions to each sheet.

```css
/* new — on the inserter toggle SVG */
.block-editor-inserter__toggle svg {
  transition: transform 0.1s cubic-bezier(...);
}
.block-editor-inserter__toggle.is-pressed svg {
  transform: rotate(45deg);
}
```

**JS (`block-editor/index.min.js`, +8 B)**
The inserter toggle `Button` component now passes `isPressed` derived from the panel's open state, keeping it in sync with the existing `aria-expanded` attribute. Previously, `isPressed` was not set, so the `.is-pressed` class was never applied and the CSS rule had no effect.

Source changes are within `packages/block-editor/src/` (inserter toggle component) and the associated SCSS source for the `@wordpress/block-editor` and `@wordpress/base-styles` packages.

## Contribution

Opened and authored by @juanfra. Reviewed by @t-hamano and @jasmussen (Joen). The PR had the `Needs Design Feedback` label, indicating design sign-off was sought before merge. Discussion was light (6 comments); no alternative approaches are documented in the thread. Merged at commit `f7b40ab`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
