# #83609: Verse: Add minimum width support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Verse`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`ed30826`](https://github.com/WordPress/gutenberg/commit/ed30826515a15c05941dc516b2b574277502e452)
- **Discussion:** [#83609](https://github.com/WordPress/gutenberg/pull/83609) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Verse (Poetry) block now supports `dimensions.minWidth` in its `block.json`, bringing it in line with other core blocks that already expose minimum width via Global Styles and the block inspector. A CSS specificity fix in the block's stylesheet ensures that a minimum width set through Global Styles actually applies, rather than being silently overridden by the block's built-in `min-width: 1em` float guard.

## Impact

- **Site owners / editors:** Can now set a minimum width on the Poetry block via Global Styles (Blocks → Poetry → Dimensions → Minimum width) or the block inspector. Previously this control was absent.
- **Theme & block developers:** No code changes required. The Verse block saves its own wrapper via `useBlockProps.save()`, so no PHP render callback change was needed. If you had previously attempted to set `min-width` on `.wp-block-verse` through Global Styles, it will now take effect (it was silently ignored before due to a specificity conflict).
- **No breaking changes.** The existing `min-width: 1em` float guard is preserved; it simply moves to a rule with matching specificity so source order decides the winner.

## Technical details

Two files carry the functional change:

**`packages/block-library/src/verse/block.json`** — adds `"minWidth": true` to the existing `dimensions` supports object (which already had `minHeight: true`).

**`packages/block-library/src/verse/style.scss`** — restructures the base rule to fix a specificity conflict:

```scss
/* Before */
pre.wp-block-verse {
  box-sizing: border-box;
  overflow: auto;
  white-space: pre-wrap;
  min-width: 1em;       /* specificity 0-1-1 */
  word-break: normal;
  overflow-wrap: break-word;
}

/* After */
pre.wp-block-verse {
  box-sizing: border-box;
  overflow: auto;
  white-space: pre-wrap;
  word-break: normal;
  overflow-wrap: break-word;
}

:root :where(pre.wp-block-verse) {
  min-width: 1em;       /* specificity 0-1-0, matches Global Styles */
}
```

The old `pre.wp-block-verse` selector (0-1-1) outranked the `:root :where(.wp-block-verse)` rule that Global Styles emits (0-1-0), so any `min-width` set in Global Styles was always lost. Moving the 1em guard into `:root :where(pre.wp-block-verse)` brings it to 0-1-0, matching Global Styles; because block-library styles load first, source order lets the Global Styles value win when one is set.

The two README files (`docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/verse/README.md`) are regenerated to list `minWidth` under the `dimensions` supports.

## Contribution

Opened by @aaronrobertshaw as part of the design-tools consistency work tracked in #43241. @ramonjd is credited as co-author. The PR was implemented by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, both from the GitHub Actions bot reporting bundle size and performance metrics); no design debate or alternative approaches are recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
