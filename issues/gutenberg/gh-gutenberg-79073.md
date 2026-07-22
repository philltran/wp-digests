# #79073: Correct behaviour of flex child fixed width and introduce max width option

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `[Feature] Layout`
- **Merged:** [`94137f7`](https://github.com/WordPress/gutenberg/commit/94137f798bff6eff56203b5dbf43ae4bbd591248)
- **Discussion:** [#79073](https://github.com/WordPress/gutenberg/pull/79073) · 8 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The block editor's flex child sizing control now distinguishes between "Max width/height" and "Fixed width/height". Previously, the "Fixed" option only set a `flex-basis` but allowed the element to shrink, which caused persistent confusion. This change renames the existing behavior to "Max" and introduces a new "Fixed" option that applies `flex-shrink: 0` to strictly prevent shrinking. Existing blocks using the legacy setting automatically retain their current behavior for backwards compatibility.

## Impact

- **Block & theme developers:** No immediate code changes required. Existing blocks using `selfStretch: 'fixed'` will continue to render as shrinkable max-width elements.
- **Site owners / editors:** The block editor UI now shows "Max" and "Fixed" as separate options. Selecting "Fixed" will now strictly prevent the block from shrinking below its set size.
- **Custom code / theme developers:** If you manually set `selfStretch: 'fixed'` in block attributes or theme.json, it will continue to map to the legacy shrinkable behavior. To get the new non-shrinking behavior, use `selfStretch: 'fixedNoShrink'`.
- **No action required** for standard block usage or existing sites.

## Technical details

The change updates the `selfStretch` attribute values for flex child layouts. The UI component `ChildLayoutControl` (`packages/block-editor/src/components/child-layout-control/index.js`) now maps the "Max" option to the serialized value `'fixed'` and the "Fixed" option to `'fixedNoShrink'`. The PHP layout renderer (`lib/block-supports/layout.php`) and JS hook (`packages/block-editor/src/hooks/layout-child.js`) were updated to check for these values. When `selfStretch` is `'fixedNoShrink'`, the renderers now output `flex-shrink: 0` alongside `flex-basis` and `box-sizing: border-box`. Legacy `'fixed'` values continue to output only `flex-basis` and `box-sizing`, preserving the original shrinkable behavior. Viewport overrides correctly transition between these states. Documentation in `docs/reference-guides/block-api/block-supports.md` was updated to reflect the new UI labels.

## Contribution

Opened to resolve long-standing confusion around the "Fixed" flex child option, the PR initially faced UI label spacing constraints during review. A reviewer suggested shortening the label to "Max" to save space, which was implemented. The author and reviewers acknowledged that mapping the legacy behavior to `'fixed'` while introducing `'fixedNoShrink'` for the new behavior was a necessary trade-off to maintain backwards compatibility without breaking existing editor state. The change was merged after addressing UI tightness and test coverage for viewport overrides.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
