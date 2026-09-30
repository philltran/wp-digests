# #83337: Video, Cover: Don't autoplay in the editor when reduced motion is preferred

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @im3dabasia
- **Labels:** `[Type] Enhancement`, `[Focus] Accessibility (a11y)`, `[Package] Block library`, `[Block] Cover`, `[Block] Video`
- **Merged:** [`ba685d7`](https://github.com/WordPress/gutenberg/commit/ba685d7aeb746c9e4de3d7901ecf395341b4eb06)
- **Discussion:** [#83337](https://github.com/WordPress/gutenberg/pull/83337) · 10 comments · 2 reactions
- **Usefulness:** 3/5

## Summary

The Video and Cover blocks in the Gutenberg editor no longer autoplay video when the user's system has `prefers-reduced-motion: reduce` enabled. The change uses the existing `useReducedMotion()` hook from `@wordpress/compose` to conditionally suppress the `autoPlay` attribute on `<video>` elements and to omit `autoplay=1` (and Vimeo's `background=1`) from embed iframe URLs. A video already playing is paused if the preference is toggled on mid-session. This is editor-only; saved block markup and frontend rendering are unchanged.

## Impact

- **Site owners / editors:** No action required. Users with OS-level reduced motion will see videos paused in the editor instead of autoplaying. Frontend behavior is unaffected.
- **Plugin & theme developers:** No breaking changes. The internal helpers `getBackgroundEmbedHtml()` and `getBackgroundVideoSrc()` in `packages/block-library/src/cover/embed-video-utils.js` gained an optional second argument (`{ autoplay = true }`); existing calls without it behave identically. No new public hooks, filters, or REST routes are introduced.
- **Hosting & platform:** No configuration or migration needed.
- **Headless & REST consumers:** No change; the block's serialized attributes and rendered frontend output are untouched.

## Technical details

Three files in the block library are modified, plus a new unit-test file.

**`packages/block-library/src/video/edit.jsx`**
- Imports `useReducedMotion` from `@wordpress/compose`.
- Adds a `useEffect` that calls `videoPlayer.current?.pause()` when `prefersReducedMotion` becomes true (the `autoPlay` attribute only fires on load, so an already-playing video needs an explicit pause).
- The `<video>` element's attribute changes from:

```jsx
autoPlay={ autoplay }
```
to:
```jsx
autoPlay={ autoplay && ! prefersReducedMotion }
```

**`packages/block-library/src/cover/edit/index.jsx`**
- Same `useReducedMotion()` import and a parallel `useEffect` that pauses `mediaElement.current` when `prefersReducedMotion && isVideoBackground`.
- The uploaded-video `<video>` element changes from a bare `autoPlay` to `autoPlay={ ! prefersReducedMotion }`.
- The embed preview HTML is now generated with the preference:

```js
getBackgroundEmbedHtml( embedPreview.html, { autoplay: ! prefersReducedMotion } )
```

**`packages/block-library/src/cover/edit/inspector-controls.jsx`**
- The `<FocalPointPicker>` in the Cover sidebar (which defaults to `autoPlay={ true }` internally) now receives `autoPlay={ ! prefersReducedMotion }` and a `key={ prefersReducedMotion }` to force a remount when the preference flips, stopping an already-playing preview.

**`packages/block-library/src/cover/embed-video-utils.js`**
- `getBackgroundEmbedHtml( html )` → `getBackgroundEmbedHtml( html, { autoplay = true } = {} )`; passes `autoplay` through to `getBackgroundVideoSrc`.
- `getBackgroundVideoSrc( src )` → `getBackgroundVideoSrc( src, { autoplay = true } = {} )`. The `autoplay=1` query parameter is now set once, conditionally, before the provider `switch`, rather than being set unconditionally inside each `case` branch. For Vimeo, `background=1` is also gated on `autoplay` because Vimeo's background mode always autoplays.

**`packages/block-library/src/cover/test/embed-video-utils.test.js`** (new)
- Vitest unit tests verifying that `getBackgroundVideoSrc` includes `autoplay=1` by default, omits it when `{ autoplay: false }` is passed, and that Vimeo's `background=1` is present only when autoplay is enabled. Also tests that `getBackgroundEmbedHtml` passes the option through to the iframe `src`.

## Contribution

Opened by @im3dabasia as part of the broader #82497 reduced-motion plan (editor portion; frontend follows in separate PRs). @adamsilverstein reviewed and tested in `wp-env`, and during review identified that the Cover sidebar's `<FocalPointPicker>` still autoplayed under reduced motion because it defaults to `autoPlay={ true }` internally; @im3dabasia added the `autoPlay` prop and a `key`-based remount to cover that path before merge. @adamsilverstein noted that uploaded videos do not resume when reduced motion is turned off without a page reload (YouTube embeds do restart immediately), and this asymmetry was accepted as-is. @afercia raised two unrelated accessibility observations (pause-control DOM ordering, focal-point-picker visibility for embeds) that were acknowledged but tracked separately. The code was drafted with Claude Code and reviewed/tested manually.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
