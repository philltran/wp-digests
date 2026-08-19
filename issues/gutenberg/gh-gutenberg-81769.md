# #81769: Fields: Export the package stylesheet

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Fields`
- **Merged:** [`e43dcaa`](https://github.com/WordPress/gutenberg/commit/e43dcaa48f885226fe557b65d4ddad3637ad0360)
- **Discussion:** [#81769](https://github.com/WordPress/gutenberg/pull/81769) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds `./build-style/*` to the `exports` map in `packages/fields/package.json`, so the `@wordpress/fields` stylesheet can be imported as `@wordpress/fields/build-style/style.css`. Previously, that package path was blocked by the exports allowlist, causing bundled fields such as `MediaEdit` to render unstyled without a build error. The PR also adds a README Setup section explaining how to load the required CSS inside and outside WordPress.

## Impact

- **Plugin and theme developers using `@wordpress/fields`**: You can now import the package stylesheet through the package path instead of using a relative `node_modules` path.
- **Within WordPress**: No `wp-fields` style handle is registered, so load the fields CSS with your plugin script and add the `wp-components` stylesheet as a dependency of your plugin stylesheet for load order.
- **Outside WordPress**: Install and import `@wordpress/theme/design-tokens.css`, `@wordpress/components/build-style/style.css`, and `@wordpress/fields/build-style/style.css`; RTL variants use `style-rtl.css`.
- **Site owners / hosting / headless REST consumers**: No action required unless your application directly bundles `@wordpress/fields` and needs its styles.

## Technical details

The diff adds one entry to the `exports` map in `packages/fields/package.json`:

```json
"exports": {
  ".": {
    "import": "./build-module/index.mjs",
    "require": "./build/index.cjs"
  },
  "./package.json": "./package.json",
  "./build-style/*": "./build-style/*"
}
```

This makes the previously shipped `build-style/style.css` resolvable through the package path:

```js
import '@wordpress/fields/build-style/style.css';
```

The README documents that no style handle is registered for this package's own stylesheet, so the CSS is loaded with the plugin script. It also notes RTL stylesheets are available at the same paths with `-rtl` appended, such as `style-rtl.css`, while `@wordpress/theme/design-tokens.css` has no separate RTL version.

## Contribution

The record shows @oandregal contributed to the final change, and the author noted updating the README to match DataViews. No broader design debate is present in the provided discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
