# #80082: Button: hide Core focus ring when button as link is pressed

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @fushar
- **Labels:** `[Type] Bug`, `[Package] Components`, `[Package] Base styles`
- **Merged:** [`f871483`](https://github.com/WordPress/gutenberg/commit/f871483f5245467c845b09985c88c0143ecd3981)
- **Discussion:** [#80082](https://github.com/WordPress/gutenberg/pull/80082) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request fixes a visual bug in the Gutenberg `Button` component where a flash of the WordPress core focus ring appears when a button rendered as a link is pressed. The fix moves a `box-shadow: none` guard from the shared `outset-ring__focus` SCSS mixin into the component-specific stylesheet. This ensures the conflicting core ring is suppressed specifically for anchor-based buttons without affecting other elements using the mixin.

## Impact

- **Block & UI developers**: No code changes required. The fix applies automatically to `Button` components rendered as links in the Site Editor and block editor.
- **Theme/Plugin developers using Gutenberg components**: If you directly import or extend `@wordpress/components` button styles, you will see the corrected focus behavior.
- **No breaking changes, deprecations, or migration steps.**
- **No action required** for existing implementations.

## Technical details

The change modifies two SCSS files in the `@wordpress/base-styles` and `@wordpress/components` packages. Previously, `packages/base-styles/_mixins.scss` applied a `&:is(a) { box-shadow: none; }` rule inside the `outset-ring__focus` mixin. This was moved to `packages/components/src/button/style.scss` as a direct rule scoped to the button component:

```scss
.components-button {
	// Defend against a conflicting focus ring from WordPress common.css on `a:focus`.
	&:focus:is(a) {
		box-shadow: none;
	}

	&:focus:not(:active) {
		@include mixins.outset-ring__focus();
	}
}
```

By scoping the `box-shadow: none` guard to the button component, it correctly targets `a:focus` elements inside the button while keeping the shared mixin clean for other consumers. The compiled CSS size increases by ~2 bytes.

## Contribution

Opened and merged by @fushar, with co-authors @ciampo and @mirka. During review, @mirka suggested moving the defensive style from the shared mixin into the `Button` stylesheet for better encapsulation. The author implemented this approach, which was accepted and merged. The PR addresses a regression introduced by recent focus-ring PRs (#78646, #79837, #74106).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
