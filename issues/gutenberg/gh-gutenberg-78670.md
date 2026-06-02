# #78670: Hide image dimension tools when a state is selected

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Cover`, `[Block] Image`, `[Package] Block editor`, `[Block] Post Featured Image`
- **Merged:** [`9510874`](https://github.com/WordPress/gutenberg/commit/95108749d74ae3492c8c4614e9c3b606e9c4c8c2)
- **Discussion:** [#78670](https://github.com/WordPress/gutenberg/pull/78670) · 4 comments · 0 reactions

## Summary

When a responsive style state (e.g. 'Mobile' or 'Tablet') is selected via the block card's style-state dropdown, several dimension-related controls on the Cover, Image, and Post Featured Image blocks now don't function correctly because those controls aren't yet wired up to per-state styling. As a stopgap, this PR hides those controls whenever a non-default style state is selected, preventing users from interacting with inputs that would produce broken or no-op behavior. The affected controls include Cover min height, Image focal point / aspect ratio / width / height / scale, and Featured Image aspect ratio / height / width. The controls reappear as normal when the default state is active.

## Impact

**Block / theme developers and editor users:**
- Purely an editor-UI gating change. When a responsive style state is selected on a Cover, Image, or Post Featured Image block, the listed dimension tools are hidden from the inspector; they remain fully available in the default state.
- No public API is removed or deprecated. The change is driven by a **private** selector (`hasSelectedStyleState`), so it is not part of the stable `@wordpress/block-editor` data API and should not be relied upon by third-party code.
- Aspect ratio on the Cover block uses standard block supports and continues to work across states; only Image's custom aspect-ratio implementation is gated.

**Site owners:** No action required — saved content and front-end rendering are unaffected; this only changes which controls are visible while editing in a selected style state.

## Technical details

The change adds a private selector, `hasSelectedStyleState`, in the block-editor package and uses it within the affected blocks to conditionally render their dimension `InspectorControls` fills. The diff touches `packages/block-editor/src/components/global-styles/dimensions-panel.js`, importing `isDefaultBlockStyleState` alongside the existing `DEFAULT_BLOCK_STYLE_STATE` and `hasPseudoBlockStyleState` from `../../hooks/block-style-state`, indicating the gating logic keys off whether the currently selected state is the default one.

Conceptually, each affected control fill goes from always-rendered to guarded:

```js
// Before
<InspectorControls group="dimensions">
	{ /* min height / focal point / aspect ratio / width / height / scale */ }
</InspectorControls>

// After
{ ! hasSelectedStyleState && (
	<InspectorControls group="dimensions">
		{ /* same controls */ }
	</InspectorControls>
) }
```

Controls gated this way: Cover min height; Image focal point, aspect ratio (Image's custom implementation), width/height, and scale; Post Featured Image aspect ratio and height/width. The Cover block's focal point is a block *setting* and was already hidden in style states. Reported bundle impact is +149 B total (+61 B in `block-editor`, +88 B in `block-library`). No `block.json`, REST schema, or DB changes are involved.

## Contribution

Authored by **@talldan** (work noted as assisted by OpenCode/Codex) and merged via commit `9510874`. The PR is labelled an enhancement touching the Cover, Image, and Post Featured Image blocks plus the block-editor and block-library packages. **@tellthemachines** (co-author/reviewer) noted it narrowly missed the 23.3 RC but argued for inclusion since it prevents broken behavior; **@cbravobernal** then cherry-picked it to the `release/23.3` branch (`04e18c687f7`) for the next release. The discussion frames the hiding as a temporary measure until these controls are properly supported for responsive style states.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
