# #82375: Update navigation link block name, use a default variation for custom link

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Navigation`
- **Merged:** [`9ba120a`](https://github.com/WordPress/gutenberg/commit/9ba120a305fb8658bf521290b74ddd9c56782a1f)
- **Discussion:** [#82375](https://github.com/WordPress/gutenberg/pull/82375) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The canonical title of the `core/navigation-link` block changes from "Custom Link" to "Navigation Link". "Custom Link" is now a static default block variation (`core/navigation-link/custom-link`) with a new description, "Add a custom link to your navigation." The rename aims to fix confusion where Navigation Link did not appear to be available in Global Styles > Blocks because it was listed as "Custom Link". The PR also moves the block's `deprecated` definitions into their own file.

## Impact

- **Site owners / editors:** The editor experience is intended to be unchanged. The inserter still offers "Custom Link", now as a variation. In Global Styles > Blocks, the block is listed as "Navigation Link". The block name and markup are unchanged, so no content migration is involved.
- **Plugin & theme developers:**
  - `name` (`core/navigation-link`) and attributes are unchanged, so existing `theme.json` block styles, `allowedBlocks`, and templates keep working.
  - Code or tests that match on the block *title* (e.g. Playwright locators using `Block: Custom Link`, or `getBlockType( 'core/navigation-link' ).title`) will now see "Navigation Link".
  - Code that references `core/navigation-link` in inserter prioritization now needs the `core/navigation-link/custom-link` variation ID to target the custom link entry.
  - E2E locators for `Block: Navigation` may now match multiple elements (e.g. "Block: Navigation Link"), so they need `exact: true`. The PR applies this to Gutenberg's own specs.
- **Docs consumers:** The block reference entry is now titled "Navigation Link". The slug `core-block-navigation-link` is unchanged.
- **Hosting / headless:** No action required.

## Technical details

Changes from the diff:

- `navigation-link/block.json`: `title` changes from `Custom Link` to `Navigation Link`. `docs/manifest.json`, `core-blocks/README.md`, and `category-design.md` are updated to match.
- New `navigation-link/variations.js` exports one variation and adds it to `settings.variations`:

```js
{
  name: 'custom-link',
  title: __( 'Custom Link' ),
  description: __( 'Add a custom link to your navigation.' ),
  icon: linkIcon,
  attributes: { kind: 'custom' },
  isActive: ( a ) => a.kind === 'custom',
  isDefault: true,
  scope: [ 'inserter' ],
}
```

- `navigation/constants.js`: `PRIORITIZED_INSERTER_BLOCKS` changes `'core/navigation-link'` to `'core/navigation-link/custom-link'`. With Custom Link as the default variation, the plain `core/navigation-link` ID would no longer match in the Navigation block inserter (noted in the discussion).
- New `navigation-link/deprecated.tsx` holds the existing `nofollow` → `rel` deprecation unchanged, with types added. `index.jsx` is renamed to `index.js` because it no longer contains JSX, and its `react/jsx-filename-extension` entry is removed from `tools/eslint/suppressions.json`.
- The existing dynamic variations (page, post, category, etc.) are generated in PHP and merged with the static `block.json`/JS variations, so declaring variations in both places is safe (per the author).
- E2E: `getByRole( 'document', { name: 'Block: Navigation', exact: true } )` in `navigation.spec.js` and `navigation-submenu-visibility.spec.js`.

The bundle-size report shows `build/scripts/block-library/index.min.js` growing by about 100 B.

## Contribution

@talldan proposed the rename to reduce confusion over Navigation Link appearing missing in Global Styles, and @MaggieCabrera and @aaronrobertshaw supported it. @jeryj raised a concern about how the existing dynamic PHP-generated variations interact with a static one. @talldan had already checked and confirmed they are merged, so both can coexist. During review, @talldan spotted that making Custom Link the default variation broke the Navigation block's inserter prioritization, which led to the `PRIORITIZED_INSERTER_BLOCKS` change. A later commit tightened E2E locators after the new title caused ambiguous matches. @talldan also suggested, as separate follow-up work, that the Global Styles block search should match variation names.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
