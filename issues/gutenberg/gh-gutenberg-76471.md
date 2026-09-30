# #76471: Components: Add allowForms prop to SandBox component

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @dhasilva
- **Labels:** `[Type] Enhancement`, `[Package] Components`
- **Merged:** [`11bae0c`](https://github.com/WordPress/gutenberg/commit/11bae0c3c0cd17eb8acd9bbf362c25a9d6d09b08)
- **Discussion:** [#76471](https://github.com/WordPress/gutenberg/pull/76471) · 11 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`@wordpress/components`' `SandBox` component gains an opt-in `allowForms` boolean prop. When true, `allow-forms` is added to the iframe's `sandbox` attribute so embedded content can submit forms (for example an age-gate date-of-birth form). It defaults to `false`, so existing behavior is unchanged. It mirrors the recently added `allowPopups` prop.

## Impact

- **Plugin & theme developers (block/editor UI):** Can pass `allowForms` to `SandBox` when previewing content that contains submittable forms. No change unless you opt in.
- **Site owners / hosting / headless:** No action required.
- **Breaking changes / deprecations:** None. The default sandbox tokens are unchanged.
- **Security note:** Enabling the prop relaxes the iframe sandbox for that instance; only enable it for content that legitimately needs form submission.

## Technical details

The diff touches `packages/components/src/sandbox/`:

- `types.ts`: adds `allowForms?: boolean` (documented, `@default false`) to `SandBoxProps`.
- `index.tsx`: both `IsolatedSandBox` and `SameOriginSandBox` destructure `allowForms = false` and add `'allow-forms': allowForms` to the `clsx()` call that builds the `sandbox` attribute, alongside the existing `'allow-popups': allowPopups`.
- `test/index.tsx`: two tests. One asserts `allow-forms` is absent by default; the other asserts that with `allowForms` the attribute equals `allow-scripts allow-presentation allow-forms`.
- `stories/index.story.tsx`: adds a `WithForm` story and `allowForms: false` to `Default`.
- `storybook/components-manifest.yml` lists `allowForms`, and `CHANGELOG.md` gets an entry.

Because `clsx` builds the string, the trailing-space nit raised in review does not apply to the final implementation.

```tsx
// Before: form submissions inside the sandbox are blocked
<SandBox html={ html } title="Preview" />

// After: opt in
<SandBox html={ html } title="Preview" allowForms />
```

No hooks, REST, or DB changes.

## Contribution

Closes long-standing issue #48942. The main debate was API shape: @aduth questioned adding a prop per `allow-*` token when `allow-scripts` and `allow-same-origin` are far riskier. Options raised were making `allow-forms` a default or opening up the whole `sandbox` attribute instead of going piecemeal. @simison pointed to the parallel `allowPopups` work in #69617, and @Mamaduka noted the PR needed a rebase on it and asked the project to settle the format for future tokens. The PR ultimately merged following the existing per-prop pattern. Claude Code was disclosed as used to generate tests.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
