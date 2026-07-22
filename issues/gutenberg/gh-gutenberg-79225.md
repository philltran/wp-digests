# #79225: Pullquote: Migrate to text-align block support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @shimotmk
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Pullquote`
- **Merged:** [`48c291c`](https://github.com/WordPress/gutenberg/commit/48c291cb3d34fe53dde354b1bc4f6f899c1ec177)
- **Discussion:** [#79225](https://github.com/WordPress/gutenberg/pull/79225) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `core/pullquote` block now uses the centralized `typography.textAlign` block support instead of a custom `textAlign` attribute. This migration consolidates alignment handling across the block library, enables global styles support for text alignment on pullquotes, and removes duplicated editor and frontend logic. Existing pullquotes with alignment set will automatically migrate to the new `style.typography.textAlign` structure via a deprecation layer.

## Impact

- **Plugin & theme developers:** No immediate action required for standard usage. The block now respects global styles for `textAlign` under `typography`.
- **Custom block consumers / migration scripts:** The `textAlign` attribute is deprecated and removed from the current block definition. If you programmatically read or write pullquote attributes, alignment is now stored in `attributes.style.typography.textAlign`.
- **Site owners:** Existing pullquotes with alignment will migrate seamlessly on save/edit. No visual changes expected.
- **Breaking changes:** The `textAlign` attribute is removed from the live block definition. It is handled via a `v6` deprecation version that migrates legacy attributes to block supports.

## Technical details

- `packages/block-library/src/pullquote/block.json`: Removed `textAlign` from `attributes`. Added `textAlign: true` under `supports.typography`.
- `packages/block-library/src/pullquote/edit.js`: Removed `AlignmentControl`, `clsx`, and manual `setAttributes({ textAlign })` logic. Now relies on `useBlockProps` and the block supports system.
- `packages/block-library/src/pullquote/save.js`: Removed manual `has-text-align-${ textAlign }` class generation. `useBlockProps.save()` handles it.
- `packages/block-library/src/pullquote/deprecated.js`: Added `v6` deprecation that imports `migrateTextAlignAttributeToBlockSupport` from `../utils/migrate-text-align`. The `migrate` function converts the legacy `textAlign` attribute to `style.typography.textAlign`.

Before (edit.js):
```js
const { textAlign, citation, value } = attributes;
const blockProps = useBlockProps({
  className: clsx({ [ `has-text-align-${ textAlign }` ]: textAlign }),
});
// ... AlignmentControl with setAttributes({ textAlign })
```

After (edit.js):
```js
const { citation, value } = attributes;
useDeprecatedTextAlign(props);
const blockProps = useBlockProps();
// Alignment handled by block supports system
```
- Test fixtures updated to reflect the new attribute structure (e.g., `core__pullquote__deprecated-6.json` and `core__pullquote__custom-colors.json`).

## Contribution

Opened by @shimotmk as part of the broader block support consolidation tracked in #60763, the PR standardizes the Pullquote block alongside other core blocks. Co-authored by @t-hamano, the change followed a straightforward review cycle with no alternative approaches debated or rejected in the thread. The implementation relies on the existing `migrateTextAlignAttributeToBlockSupport` utility rather than introducing new migration logic.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
