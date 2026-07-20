# #79600: A11y: replace local aria-live regions with speak()

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @SainathPoojary
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `[Package] Components`, `[Package] Block editor`, `[Package] DataViews`, `[Package] Fields`
- **Merged:** [`8c97560`](https://github.com/WordPress/gutenberg/commit/8c9756016c301d4b1cc669e66ac29323bc615510)
- **Discussion:** [#79600](https://github.com/WordPress/gutenberg/pull/79600) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request replaces hardcoded inline `<div aria-live="polite">` regions used for validation error announcements across several Gutenberg form and media components with the centralized `@wordpress/a11y` `speak()` utility. The change addresses unreliable screen reader behavior caused by dynamically mounted and unmounted live regions, and prevents duplicate announcements when combined with native `aria-describedby` attributes. By routing announcements through a single utility, validation messages and media editor state changes are now announced consistently without spamming assistive technology users.

## Impact

- **Plugin & theme developers:** No breaking changes or public API modifications. Components like `ControlWithError`, `ValidatedDateControl`, and `MediaEdit` retain their existing props and rendering contracts.
- **Block editor & Site Editor users:** Screen reader announcements for form validation errors and media editor actions (crop, rotate, flip) will be more reliable and free of duplicate readouts.
- **No action required.** The refactoring is internal to the `@wordpress/components`, `@wordpress/dataviews`, `@wordpress/fields`, and `@wordpress/media-editor` packages.

## Technical details

The diff removes `<div aria-live="polite">` wrappers from the JSX in `packages/components/src/validated-form-controls/control-with-error.tsx`, `packages/dataviews/src/components/dataform-controls/date.tsx`, `packages/fields/src/components/media-edit/index.tsx`, and `packages/media-editor/src/image-editor/react/components/cropper.tsx`. For `ValidatedDateControl` and `MediaEdit`, a `useEffect` hook was added to watch `isTouched` and `customValidity?.message`, triggering `speak( customValidity.message )` when a validation error is present. `ControlWithError` instead relies on the native `aria-describedby` attribute to avoid duplicate readouts. The `useAriaAnnouncer` hook in `packages/media-editor/src/image-editor/react/hooks/use-aria-announcer.ts` was updated to call `speak()` directly instead of returning a string for an inline live region. The `@wordpress/a11y` package was added as a dependency to `dataviews`, `fields`, and `media-editor` via their `package.json` files, and corresponding `tsconfig.json` references were updated. Unit tests were adjusted to mock `@wordpress/a11y` and assert that `speak` is called with the expected message strings.

## Contribution

Opened by @SainathPoojary to address issue #79530, the PR was reviewed and refined by @ramonjd, @ciampo, @t-hamano, and accessibility experts including @afercia and @joedolson. The team confirmed that replacing inline live regions with the centralized `speak()` utility would resolve double-announcement bugs and improve reliability across rapidly unmounting React components. The PR was merged after resolving merge conflicts and updating package dependencies and CHANGELOGs.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
