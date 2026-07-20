# #76581: Term Name: Migrate to textAlign block support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @huzaifaalmesbah
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Terms Query`
- **Merged:** [`b0f28a1`](https://github.com/WordPress/gutenberg/commit/b0f28a11d8ca5cdcd1b98c31aec99472f1ae54de)
- **Discussion:** [#76581](https://github.com/WordPress/gutenberg/pull/76581) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `core/term-name` block now uses the centralized `typography.textAlign` block support instead of a custom `textAlign` attribute. This migration consolidates alignment handling across the block library, removes manual toolbar controls, and enables global styles support for text alignment on the block. Existing blocks with the legacy attribute are automatically migrated via a new `v1` deprecation to prevent validation warnings.

## Impact

- **Plugin & theme developers:** The `textAlign` attribute is removed from `core/term-name`. If you were reading or setting this attribute via `block.json` filters or custom markup, update your code to use `style.typography.textAlign` instead.
- **Site owners & editors:** No action required. Existing Term Name blocks with left/center/right alignment will silently migrate to the new style-based system without visual changes or console errors.
- **No breaking changes for end users**, but the internal block structure changed. Developers relying on the old attribute will see a deprecation migration during block validation.

## Technical details

- **`packages/block-library/src/term-name/block.json`**: Removed the `textAlign` attribute from `attributes`. Added `"textAlign": true` under `supports.typography`.
- **`packages/block-library/src/term-name/edit.js`**: Removed `AlignmentControl` from `BlockControls` and the manual `clsx` className generation (`has-text-align-${ textAlign }`). Now relies on `useBlockProps()` to apply alignment classes via the block support system.
- **`packages/block-library/src/term-name/deprecated.js`**: New file defining a `v1` deprecation. `isEligible` checks for the legacy `textAlign` attribute, and `migrate` calls `migrateTextAlign` (from `../utils/migrate-text-align`) to convert the attribute to `style.typography.textAlign`.
- **`packages/block-library/src/term-name/index.js`**: Imports and exports the `deprecated` array.
- **Before/After (`edit.js`)**:
  ```javascript
  // Before
  const { textAlign, level = 0, isLink, levelOptions } = attributes;
  const blockProps = useBlockProps( {
      className: clsx( { [ `has-text-align-${ textAlign }` ]: textAlign } ),
  } );
  // ... <AlignmentControl value={ textAlign } onChange={ ( nextAlign ) => setAttributes( { textAlign: nextAlign } ) } />

  // After
  const { level = 0, isLink, levelOptions } = attributes;
  const blockProps = useBlockProps();
  // Alignment handled by typography.textAlign block support
  ```

## Contribution

Opened and merged by @huzaifaalmesbah, with co-authorship from @t-hamano. The change was part of a broader initiative (#60763) to standardize block alignment handling. Review focused on verifying the deprecation migration path and resolving merge conflicts before landing. No alternative approaches were debated; the PR was accepted as a straightforward consolidation.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
