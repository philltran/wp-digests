# #80500: Math: Use ValidatedTextareaControl for LaTeX input

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `Needs Design Feedback`, `[Package] Block library`, `[Package] Format library`, `[Block] Math`
- **Merged:** [`10f3b3e`](https://github.com/WordPress/gutenberg/commit/10f3b3e20b6f9c4e508d4021690525c9be080963)
- **Discussion:** [#80500](https://github.com/WordPress/gutenberg/pull/80500) · 10 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Math block and inline Math format now use `ValidatedTextareaControl` and `ValidatedTextControl` from `@wordpress/components` to handle LaTeX input validation. This replaces the previous custom badge and `speak()`-based screen reader announcements with standard HTML5 form validation. Errors now surface when the input field loses focus rather than on every keystroke, preventing constant interruptions while typing complex formulas.

## Impact

- **Block & format developers:** No action required. The underlying `latex` and `mathML` attributes and the `latexToMathML` parsing contract remain unchanged.
- **End users / editors:** Improved typing experience in the Math block and inline Math format. Validation errors no longer interrupt composition; they appear on blur and update live afterward.
- **Accessibility:** Screen reader users no longer hear repeated parse errors on every keystroke. Validation messages are now conveyed via standard `aria-describedby`/`aria-invalid` patterns instead of `aria-live` regions.

## Technical details

- Replaced `TextareaControl` and `TextControl` with `ValidatedTextareaControl` and `ValidatedTextControl` (exported via `componentsPrivateApis` in `@wordpress/components`).
- Wrapped inputs in a `<form>` element and attached an `onFocusOutside` handler to the `Popover` that calls `formRef.current?.reportValidity()`, triggering native HTML5 validation.
- Removed manual `speak()` calls from `@wordpress/a11y` and custom `WCBadge` rendering.
- Updated `packages/format-library/src/math/style.scss` to target `.block-editor-format-toolbar__math-input input` and apply `$font-family-mono`.
- E2E tests in `test/e2e/specs/editor/blocks/math.spec.js` now verify `toHaveAccessibleDescription` on blur instead of checking `aria-live` text.

**Validation handling pattern change:**
```jsx
// Before
<TextareaControl onChange={ ( val ) => {
  try { parse(val); setError(null); }
  catch(e) { speak(e.message); setError(e.message); }
}} />
{ error && <WCBadge>{ error }</WCBadge> }

// After
<form ref={ formRef }>
  <ValidatedTextareaControl
    customValidity={ error ? { type: 'invalid', message: error } : undefined }
    onChange={ ( val ) => {
      try { parse(val); setError(null); }
      catch(e) { setError(e.message); }
    }}
  />
</form>
```

## Contribution

Opened by @Mamaduka as a refactoring that evolved into a UX improvement. Design feedback from @jasmussen and @ellatrix confirmed the popover behavior and explicitly rejected adding explicit "Cancel/Submit" buttons, noting that immediate sync matches established LaTeX editor patterns. The PR was co-authored by @mirka, @ellatrix, and @jasmussen, and merged after confirming the blur-first validation approach.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
