# #78642: UI: `Tooltip.Provider` — forward upstream `closeDelay` and `timeout` props

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] UI`
- **Merged:** [`1d2a13a`](https://github.com/WordPress/gutenberg/commit/1d2a13a0e62582f80d77d7d4da99a557380f60fe)
- **Discussion:** [#78642](https://github.com/WordPress/gutenberg/pull/78642) · 3 comments · 0 reactions

## Summary

`Tooltip.Provider` in `@wordpress/ui` previously typed only `delay` and `children` in its public API, even though `provider.tsx` already spread all props through to the underlying `@base-ui/react` `Tooltip.Provider` at runtime. This meant passing `closeDelay` or `timeout` caused TypeScript errors despite working correctly at runtime. This PR widens the exported type to the full upstream `_Tooltip.Provider.Props`, closing the gap between the runtime behaviour and the type surface. There is zero runtime change — no JavaScript is emitted differently.

## Impact

**Plugin and theme developers using `@wordpress/ui` with TypeScript**
- `closeDelay` and `timeout` can now be passed to `<Tooltip.Provider>` without TypeScript errors. No action required if you were not using these props.
- If you previously worked around the missing types (e.g. cast to `any`, suppressed the error, or omitted the prop), you can now remove that workaround.

**JavaScript-only consumers**
- No action required. Both props were already forwarded at runtime.

**No breaking changes, no deprecations, no configuration required.**

## Technical details

Two files change.

**`packages/ui/src/tooltip/types.ts`** — The `ProviderProps` type (or equivalent alias) is widened:

```ts
// Before
type ProviderProps = Pick<_Tooltip.Provider.Props, 'delay' | 'children'>;

// After
type ProviderProps = _Tooltip.Provider.Props;
```

This exposes all four upstream props in the public type surface:

| Prop | Description |
|---|---|
| `delay` | Open delay (previously typed) |
| `children` | Provider children (previously typed) |
| `closeDelay` | Close delay (newly typed) |
| `timeout` | Instant-open window after a recent close; upstream default `400` ms (newly typed) |

**`packages/editor/src/components/collaborators-presence/avatar/test/index.tsx`** — `closeDelay={ 0 }` is restored on the `renderAvatar` `Tooltip.Provider` wrapper. It had been removed in #78466 solely because the prop was not part of the typed API at that point; now that the type is widened it is reinstated.

`packages/ui/src/tooltip/provider.tsx` is **not changed** — the existing `{ ...props }` spread already passed all props through to `@base-ui/react`.

## Contribution

Opened and merged by @ciampo as a direct follow-up to #78466, which first introduced `Tooltip.Provider` into `@wordpress/ui`. The motivating thread was a review comment in #78466 noting that `closeDelay` had to be stripped from a test because it wasn't part of the public type. The PR had only bot activity in its three comments and no substantive design discussion; the fix was unambiguous. The author notes the PR was drafted with Cursor (Claude) and reviewed by a human before commit `1d2a13a`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
