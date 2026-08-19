# #81787: Command Palette: Allow commands to override the category icon

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] Commands`, `[Package] Core commands`
- **Merged:** [`42e91ce`](https://github.com/WordPress/gutenberg/commit/42e91ce20540a4b6d9b7d3df5a2b61d15090e577)
- **Discussion:** [#81787](https://github.com/WordPress/gutenberg/pull/81787) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Command Palette's per-category icon is now a fallback rather than an override. Since #75669 added command categories, every command in the `view` category was force-rendered with the `arrowRight` icon, silently discarding any `icon` the command itself registered. `CommandItem` now resolves `command.icon ?? CATEGORY_FALLBACK_ICONS[ category ]`, so a command that supplies its own icon wins and only iconless commands fall back to the category default. Core's `view` commands were audited alongside the change: the icons that were being registered but never rendered are deleted from source, leaving only "View site" and "Preview in a new tab" overriding the arrow with `external`.

## Impact

- **Plugin & theme developers registering commands** (`@wordpress/commands` — `useCommand`, `wp.data.dispatch('core/commands').registerCommand`): an `icon` passed on a `category: 'view'` command is now honored again instead of being replaced by `arrowRight`. If you were passing an icon on a `view` command and relying on the palette to normalize it away, your command's icon will now appear. Conversely, if you want the unified arrow, omit `icon` entirely.
- **Documented expectation:** the `@wordpress/commands` README now states that `view` commands should rely on the fallback and only pass their own icon when it communicates something the arrow cannot (e.g. opening a new tab).
- **Site owners / end users:** cosmetic only — "View site" and "Preview in a new tab" show the external-link icon; all other `view` commands keep the arrow.
- **No breaking API change, no deprecations, no migration required.** No public function signature, hook, or REST surface changed.

## Technical details

In `packages/commands/src/components/command-menu.js`, the `CATEGORY_ICONS` map is renamed `CATEGORY_FALLBACK_ICONS` (still just `{ view: arrowRight }`) and the resolution moves into `CommandItem`:

```js
// Before
className={ clsx( 'commands-command-menu__item', {
	'has-icon': CATEGORY_ICONS[ commandCategory ] || command.icon,
} ) }
...
{ CATEGORY_ICONS[ commandCategory ] ? (
	<Icon icon={ CATEGORY_ICONS[ commandCategory ] } />
) : (
	isValidIcon( command.icon ) && <Icon icon={ command.icon } />
) }

// After
const icon = command.icon ?? CATEGORY_FALLBACK_ICONS[ commandCategory ];
...
className={ clsx( 'commands-command-menu__item', { 'has-icon': !! icon } ) }
...
{ isValidIcon( icon ) && <Icon icon={ icon } /> }
```

A side effect of collapsing the two branches is that the category fallback icon now passes through `isValidIcon()` as well, rather than being rendered unconditionally.

Core command definitions were then cleaned up to match what actually renders. In `packages/core-commands/src/site-editor-navigation-commands.js`, the `icon` property is removed from `core/edit-site/open-template-parts` (`symbolFilled`), `core/edit-site/open-styles` (`styles`), `core/edit-site/open-navigation` (`navigation`), `core/edit-site/open-templates` (`layout`), `core/edit-site/open-patterns` (`symbol`), and `core/open-styles-css` (`brush`), with the now-unused `@wordpress/icons` imports trimmed to `post, page, layout, symbolFilled`. In `packages/editor/src/components/commands/index.js`, `core/open-shortcut-help` loses `keyboard` and `core/switch-to-previous-entity` loses `page`. The dropped icon imports shrink `build/scripts/core-commands/index.min.js` by 489 B (-11.18%). The `@wordpress/commands` CHANGELOG records the behavior under Unreleased → Enhancements.

## Contribution

The change is a follow-up to #75669, which introduced command categories and, in the same pass, hard-coded `arrowRight` for the `view` category; #66720 is also referenced. The PR record carries no design debate — the only thread activity is the props bot (which additionally credits @Mamaduka) and the bundle-size bot. The author disclosed that Claude Code assisted with the implementation and PR description, with all changes reviewed by hand.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
