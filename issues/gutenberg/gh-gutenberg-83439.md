# #83439: Media Utils: Stabilize `unstableFeaturedImageFlow` as `featuredImageFlow`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Enhancement`, `[Feature] Media`, `[Package] Editor`, `[Package] Block editor`, `[Package] E2E Tests`, `[Package] Media Utils`, `[Package] Fields`
- **Merged:** [`c6e9fda`](https://github.com/WordPress/gutenberg/commit/c6e9fda8de864ee1483221985081ea60f282aa73)
- **Discussion:** [#83439](https://github.com/WordPress/gutenberg/pull/83439) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `MediaUpload` component in `@wordpress/media-utils` gains a stable `featuredImageFlow` prop that opens the featured-image media frame (with the "Set featured image" button), replacing the previously unstable `unstableFeaturedImageFlow`. The old prop is now formally deprecated via `@wordpress/deprecated` (since 7.2, alternative `featuredImageFlow`, version 7.4). Core's own featured-image panels pass both names simultaneously so that third-party plugins reading the old prop from `editor.MediaUpload` filter callbacks continue to work without a silent break.

## Impact

- **Plugin & theme developers extending `editor.MediaUpload`:** If your filter callback inspects `props.unstableFeaturedImageFlow` to detect the featured-image context (e.g. Jetpack External Media), migrate to `props.featuredImageFlow`. The old name still works for now but will emit a deprecation warning when passed without the new name, and will be removed in a future release.
- **Plugin & theme developers (no extension of `MediaUpload`):** No action required.
- **Site owners / end users:** No visible change; the featured-image modal behaves identically.
- **Hosting & platform:** No configuration or migration steps. The `@wordpress/media-utils` package gains a dependency on `@wordpress/deprecated` (already a core package).

## Technical details

In `packages/media-utils/src/components/media-upload/index.js`, the `openModal()` method now destructures both `featuredImageFlow` and `unstableFeaturedImageFlow` from `this.props`. A deprecation guard fires only when the old prop is present and the new one is absent:

```js
if (
  unstableFeaturedImageFlow !== undefined &&
  featuredImageFlow === undefined
) {
  deprecated(
    'wp.mediaUtils.MediaUpload unstableFeaturedImageFlow prop',
    { since: '7.2', alternative: 'featuredImageFlow', version: '7.4' }
  );
}
```

The frame-selection logic uses nullish coalescing so either name triggers the featured-image frame:

```js
if ( featuredImageFlow ?? unstableFeaturedImageFlow ) {
  this.buildAndSetFeatureImageFrame();
}
```

Core call sites pass **both** props to suppress the warning while preserving backward compatibility:

- `packages/editor/src/components/post-featured-image/index.jsx` (classic panel)
- `packages/fields/src/fields/featured-image/edit.tsx` (DataForm-based field)

The `@wordpress/media-utils` package adds `@wordpress/deprecated` to its `package.json` dependencies and `tsconfig` references. The `block-editor` README documents the new `featuredImageFlow` prop (Boolean, default `false`). The E2E test plugin (`packages/e2e-tests/plugins/media-upload-filter/index.js`) and the `MediaEdit` jsdom test were updated to read/assert the new prop name.

## Contribution

Authored by @ntsekouras as a follow-up to PR #82678, with @Mamaduka credited as co-author. The PR carries only two comments and no visible design debate; the key decision—passing both prop names from core to avoid silently breaking plugins like Jetpack External Media—is explained in the PR body rather than in a review thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
