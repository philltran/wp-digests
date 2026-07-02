# #79655: FormFileUpload: Hard deprecate 40px default size

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Components`, `[Package] Media Utils`
- **Merged:** [`b159247`](https://github.com/WordPress/gutenberg/commit/b1592479bcfb6e4a3d484307985bafdbe37cca8f)
- **Discussion:** [#79655](https://github.com/WordPress/gutenberg/pull/79655) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Hard deprecates the `__next40pxDefaultSize` prop on the `FormFileUpload` component within `@wordpress/components`. The opt-in cycle for the larger default button size has concluded; the component now unconditionally renders a 40px `Button` internally, and passing the prop to consumers is obsolete.

## Impact

- **Plugin & theme developers**: If you directly import `<FormFileUpload>` from `@wordpress/components`, remove any instances of the `__next40pxDefaultSize` prop. Passing it is no longer supported and will trigger an ESLint error under the updated `components-no-missing-40px-size-prop` rule, which now treats the prop as forbidden rather than required.
- **Internal/Platform teams**: No action required; internal consumers like `MediaUploadModal` have already been updated to strip the prop.

## Technical details

In `packages/components/src/form-file-upload/index.tsx`, the component signature now destructures and ignores the prop (`__next40pxDefaultSize: _next40pxDefaultSize`), while hardcoding it onto the internal render path: `<Button onClick={openFileDialog} __next40pxDefaultSize { ...props }>`.

The runtime warning from `maybeWarnDeprecated36pxSize` was removed, and `types.ts` marks the prop as `@deprecated` with `@ignore`. The ESLint rule `components-no-missing-40px-size-prop` no longer tracks `FormFileUpload` in its optional exemption set; instead, `tools/eslint/config.mjs` moves it to the `restrictedSyntax` forbid list, flagging any remaining JSX usage. Internal packages like `packages/media-utils/src/components/media-upload-modal/index.tsx` have been cleaned of the prop.

## Contribution

Merged by @mirka (co-authored with @ciampo), this PR closed out a multi-release deprecation cycle for the component's sizing API. The change was reviewed as a standard breaking cleanup, validated against Storybook and editor media flows, and applied across components, media-utils, and ESLint configurations to align linting behavior with the finalized default size.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
