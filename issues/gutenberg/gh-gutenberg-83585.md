# #83585: Math: Add minimum width and height support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Math`
- **Merged:** [`55f9bc8`](https://github.com/WordPress/gutenberg/commit/55f9bc88021e29f845b8253be5029d28c2c1c7b2)
- **Discussion:** [#83585](https://github.com/WordPress/gutenberg/pull/83585) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The core Math block (`core/math`) now supports minimum width and minimum height through the `dimensions.minWidth` and `dimensions.minHeight` block supports. Users can set these in the block inspector and in Global Styles (including per-block styles and the mobile state). The change is part of the design tools consistency work. Width, height and aspect ratio are intentionally left out and deferred to later work.

## Impact

**Site owners / editors**
- The Math block gains Dimensions controls for min width and min height in the block inspector and under Global Styles > Blocks > Math > Dimensions.
- Existing Math blocks are unaffected until a minimum is set.

**Theme developers**
- `theme.json` can now set `styles.blocks.core/math.dimensions.minWidth` and `minHeight`, including Global Styles responsive (mobile) values.
- Dimensions control availability for the block is governed by the usual `settings.dimensions` opt-ins in `theme.json`. The PR does not describe this, so verify it in your theme.

**Plugin developers**
- Code that inspects `core/math` supports (`getBlockSupport( 'core/math', 'dimensions' )`) will now see `minHeight` and `minWidth`.
- No new hooks or APIs, and no PHP changes.

**Action required:** none.

## Technical details

The diff adds a `dimensions` entry to `supports` in `packages/block-library/src/math/block.json`:

```json
"dimensions": {
	"minHeight": true,
	"minWidth": true
},
```

The rest is generated documentation. `packages/block-library/src/math/README.md` and `docs/reference-guides/core-blocks/README.md` now list `dimensions (minHeight, minWidth)` in the block's supports.

According to the PR description, Math is a static block whose `save.js` applies the wrapper via `useBlockProps.save()`. The dimensions supports therefore serialize onto the wrapper automatically, with no PHP or `save.js` change (the diff contains neither). Per the PR's testing steps, a block-level value outranks Global Styles at every width, including mobile, because block values have no mobile state of their own. A block value set while the Mobile state is selected wins at mobile widths.

The PR notes that the block's existing `overflow-x: auto` in `style.scss` gives a long formula a scroll container, and a minimum size gives that container something to size against.

## Contribution

Authored by @aaronrobertshaw, with @ramonjd credited by props-bot. It is related to the design tools consistency tracking issue #43241. The PR states it was implemented and screenshotted by a Claude Code agent from a predefined task. The record shows no design debate, only deliberate scoping that leaves width, height and aspect ratio for later PRs.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
