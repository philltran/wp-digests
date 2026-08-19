# #81738: Dashboard Widgets: resolve widget types without a metadata module

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`, `[Package] Widget primitives`
- **Merged:** [`ab98811`](https://github.com/WordPress/gutenberg/commit/ab9881109519f3024ea8b2a4ef1207442709dd26)
- **Discussion:** [#81738](https://github.com/WordPress/gutenberg/pull/81738) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`useWidgetTypes()` in `@wordpress/widget-primitives` previously dropped any widget-module record whose `widget_module` was `null`, which forced every bundled Dashboard widget to ship a `widget.ts` stub that exported nothing but its own name. The hook now resolves such a record from its own REST fields — `title`, `description`, `icon`, `actions`, `presentation`, `category`, `help`, `keywords` — as long as the record carries a `render_module`, so a widget can be declared entirely by its `widget.json` manifest. Six bundled widgets (`site-preview`, `welcome`, `news`, `quick-draft`, `hello-dolly`, `site-health`) lose their `widget.ts` as a result. The change also introduces a `DEFAULT_API_VERSION = 1` fallback that applies on both resolution paths, closing a hole where a module omitting `apiVersion` produced an `undefined` masked by the `as WidgetType` cast.

## Impact

**Dashboard widget authors (experimental)**
- `widget.ts` is now optional. A widget whose metadata is fully expressible in `widget.json` needs no metadata module at all; keep `widget.ts` only for values that cannot be serialized — `attributes` (Edit components, translated labels, field-type references) and `example`. `activity`, `events`, and `hello-world` retain theirs for exactly that reason.
- A record with **no** `render_module` still drops, module or not — there is nothing to mount.

**Hosts consuming `useWidgetTypes()`**
- Behavior change, not a breaking one: records with `widget_module: null` that were previously filtered out of the returned `WidgetType[]` will now appear if they have a `render_module`. Anything counting on the old silent drop (e.g. inserter counts) should be re-checked.
- `apiVersion` is now guaranteed present on every returned `WidgetType` instead of possibly `undefined`. A module-declared `apiVersion` still wins.

**Site owners / plugin & theme developers outside the Dashboard project**
- No action required. This is confined to the experimental `widget-primitives` package and the bundled `widgets/` directory; bundle impact is +56 B on `build/modules/widget-primitives/index.min.js`.

## Technical details

In `packages/widget-primitives/src/hooks/use-widget-types.ts`, the guard at the top of the per-record async mapper changes from an unconditional bail to a manifest-only resolution path:

```ts
// Before
if ( ! record.widget_module ) {
	return null;
}

// After
if ( ! record.widget_module ) {
	if ( ! record.render_module ) {
		return null;
	}
	return {
		apiVersion: DEFAULT_API_VERSION,
		title: record.title ?? record.name,
		...( record.icon ? { icon: pendingIcon } : {} ),
		...( record.actions
			? { actions: withRenderableIcons( record.actions, true ) }
			: {} ),
		...recordOverlay( record ),
	} as WidgetType;
}
```

A new module-scoped helper, `recordOverlay( record )`, collects the fields both paths emit identically — `name` (cast to `WidgetName`), `renderModule` (`record.render_module ?? ''`), and the conditionally-spread `presentation`, `category`, `description`, `help`, `keywords`. The module path had those inline; they are now removed from it and the helper is spread **last**, preserving the existing precedence in which record values win over the module's. `title` keeps its three-step fallback in the module path (`record.title ?? metadata.title ?? record.name`) and a two-step one in the manifest-only path.

`const DEFAULT_API_VERSION = 1;` is declared alongside, and `apiVersion: DEFAULT_API_VERSION` is placed **before** `...metadata` in the module path so a module-declared value still overrides it. Icon handling is unchanged in shape: a manifest-only record with an `icon` gets `pendingIcon` as a stand-in and resolves through the registered icon resolver, and action icons go through `withRenderableIcons( ..., true )` — the same `holdPending` behavior as record-supplied actions on the module path.

On the authoring side, `widgets/{site-preview,welcome,news,quick-draft,hello-dolly,site-health}/widget.ts` are deleted; each was a default export of `{ name: 'core/…' }` that `widget.json` already declared. `widgets/news/widget.json` gains an `actions` entry — `news-visit`, `icon: "core/external"`, `relevance: "high"`, `href: "https://wordpress.org/news/"`, `openInNewTab: true` — moving that surface into the manifest. Docs are synced: `docs/explanations/architecture/dashboard-widgets.md` now calls `widget.ts` an "optional metadata module" and the pipeline SVG reads `widget.json + render.tsx + optional widget.ts`. Three Jest cases are added to `packages/widget-primitives/src/hooks/test/use-widget-types.test.tsx` covering the `apiVersion` default, manifest-only resolution (including async icon resolution and a one-element `actions` array), and the drop of a record with neither module.

## Contribution

Authored by **@retrofox** with review from **@chihsuan**; both are credited via props-bot on the merge commit. The review produced two substantive changes: the `apiVersion` default, which @retrofox noted was previously an `undefined` hidden by the `as WidgetType` cast, and a request to collapse the two resolution branches into one. @retrofox declined the full collapse and extracted the shared `recordOverlay()` helper instead, arguing that the module path legitimately owns the dynamic import and its failure modes, the `moduleIcon ?? pendingIcon` precedence, and the actions `holdPending` flag keyed on which source supplied them — so a single expression would read worse than the duplication it removed. A stale line in the architecture pipeline diagram flagged in review was fixed in the same round. The PR is part of the Dashboard Overview effort (#77616), tracked under #79231 and the `widget.json` metadata-schema thread (#77629); a reported e2e failure in `tabs.spec.js` was flagged by CI as flaky and unrelated.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
