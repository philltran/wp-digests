# #83452: Cover: Don't autoplay embedded background videos when reduced motion is preferred

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @im3dabasia
- **Labels:** `[Type] Enhancement`, `[Focus] Accessibility (a11y)`, `[Package] A11y`, `[Package] Block library`, `[Block] Cover`
- **Merged:** [`9e72568`](https://github.com/WordPress/gutenberg/commit/9e72568e79e68ba6ea80ae643a3266bb71dfe07f)
- **Discussion:** [#83452](https://github.com/WordPress/gutenberg/pull/83452) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Cover block no longer autoplays embedded background videos (YouTube, Vimeo, VideoPress) on the front end when the visitor's OS has reduced motion enabled. The change introduces a new `prefersReducedMotion()` function in `@wordpress/a11y` as a front-end counterpart to the existing `useReducedMotion` hook in `@wordpress/compose`, and adds Cover's first view module to swap the iframe `src` for a non-autoplaying variant at runtime. The server-rendered markup is unchanged (still includes `autoplay`), so full-page caching and saved content are unaffected.

## Impact

- **Site owners / visitors:** Cover blocks with embedded video backgrounds will not autoplay for visitors who have enabled "Reduce motion" in their OS. No site-side configuration is needed.
- **Plugin & theme developers:** New public API `prefersReducedMotion()` is exported from `@wordpress/a11y` (both script and module entries). Use it in block view scripts or any non-React front-end code to check the preference once. It is a one-shot read; for reactive behavior, watch the media query yourself.
- **Hosting & platform:** No DB migration, no deprecation, no change to saved markup. The view module is enqueued only when a Cover block renders an embed background, so pages without such blocks are unaffected.
- **No action required** for existing sites. The change is purely additive on the front end.

## Technical details

**New function — `packages/a11y/src/shared/prefers-reduced-motion.ts`:**

```js
export function prefersReducedMotion(): boolean {
	return (
		window.matchMedia?.( '(prefers-reduced-motion: reduce)' )?.matches ===
		true
	);
}
```

Exported from both `packages/a11y/src/index.ts` (script entry) and `packages/a11y/src/module/index.ts` (module entry).

**Cover view module — `packages/block-library/src/cover/view.js` (new file):**

```js
import { getContext, store } from '@wordpress/interactivity';
import { prefersReducedMotion } from '@wordpress/a11y';

store(
	'core/cover',
	{
		state: {
			get videoSrc() {
				const { src, reducedMotionSrc } = getContext();
				if ( reducedMotionSrc && prefersReducedMotion() ) {
					return reducedMotionSrc;
				}
				return src;
			},
		},
	},
	{ lock: true }
);
```

This is the first block view module to depend on a package other than `@wordpress/interactivity`; the dependency is picked up automatically into `module_dependencies`.

**Server render — `packages/block-library/src/cover/index.php`:**

- Autoplay query params (`autoplay=1`, and for Vimeo `background=1`) are now collected into a separate `$autoplay_params` array rather than merged into `$query_params`.
- Two URLs are built: `$iframe_src` (with autoplay params, rendered into the HTML) and `$reduced_motion_src` (without them).
- Both are passed to the client via `data-wp-context` as `{ src, reducedMotionSrc }`.
- The wrapper `<div>` gains `data-wp-interactive="core/cover"` and the `<iframe>` gains `data-wp-bind--src="state.videoSrc"`.
- `wp_enqueue_script_module( '@wordpress/block-library/cover/view' )` is called only when the background type is an embed.

**`block.json` change:** `"interactivity"` gains `"interactive": true` alongside the existing `"clientNavigation": true`.

**`package.json` change:** `wpScriptModuleExports` gains `"./cover/view": "./build-module/cover/view.mjs"`.

**Vimeo note:** `background=1` is moved into `$autoplay_params` because Vimeo's background mode always autoplays, making it incompatible with the reduced-motion source.

## Contribution

Opened by @im3dabasia as part of the broader #82497 reduced-motion effort (the editor-side fix was #83337). @joedolson provided a11y review and approval. @Mamaduka asked whether the existing `useReducedMotion` hook in `@wordpress/compose` could consume the new function to avoid duplicating the media-query string; @im3dabasia explained the two serve different purposes (the hook re-renders on change, the function is a one-shot read) and that the only shared code is the query string, and @Mamaduka agreed the one-liner duplication was acceptable. The PR was drafted with Claude Code and then manually reviewed and tested. Merged as `9e72568`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
