# #82353: Media Utils: Preserve arrays in multipart form data

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @antwonw
- **Labels:** `[Type] Bug`, `First-time Contributor`, `[Package] E2E Tests`, `[Package] Media Utils`, `Backported to WP Core`, `[Feature] Client Side Media`
- **Merged:** [`542808f`](https://github.com/WordPress/gutenberg/commit/542808f00ab7343826d1b7afc85057a9133aa32e)
- **Discussion:** [#82353](https://github.com/WordPress/gutenberg/pull/82353) · 8 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

`flattenFormData()` in `@wordpress/media-utils` now serializes array values as indexed multipart fields (`image_size[0]`, `image_size[1]`) instead of coercing them with `String()` into a single comma-joined value. This fixes a regression in client-side media processing where registered image sizes with identical dimensions are grouped into one sideload request. The sideload endpoint previously received a combined name like `post-thumbnail,gform-image-choice-lg`, rejected it as an unknown size, and could leave the Image block showing a broken image right after upload.

## Impact

**Site owners / plugin & theme developers**
- Affected sites are those with client-side media processing active (Chrome/Edge) that register two or more image sizes via `add_image_size()` sharing the same width, height and crop settings. Uploads on those sites could fail the sideload request with a validation error. After this fix the request succeeds and every grouped size name is registered against the one generated file.
- No code or configuration changes are required.

**Gutenberg / package consumers**
- `CreateSideloadFile.image_size`, `SideloadAdditionalData.image_size` and `SubSizeData.image_size` are widened from `string` to `string | string[]`. TypeScript code that reads `image_size` from these types as a plain string may need to handle the array form.
- `flattenFormData()` now takes `data: unknown`, so callers no longer need to cast.

**Headless & REST consumers**
- Clients posting to `/wp/v2/media/{id}/sideload` should send grouped sizes as `image_size[0]`, `image_size[1]`, and so on. Comma-joined values are not interpreted as multiple sizes.

The PR was flagged for the WordPress 7.1 maintenance backport (labeled `Backported to WP Core`).

## Technical details

**`packages/media-utils/src/utils/flatten-form-data.ts`**

The recursion condition changed from a plain-object check to one that also covers arrays. `Object.entries()` on an array yields index keys, so the same loop produces indexed multipart keys:

```ts
// before
export function flattenFormData( formData, key, data: string | undefined | Record<string,string> ) {
	if ( isPlainObject( data ) ) { /* recurse with `${key}[${name}]` */ }
	// arrays fell through to String( data ) -> "a,b"
}

// after
export function flattenFormData( formData, key, data: unknown ) {
	if ( Array.isArray( data ) || isPlainObject( data ) ) {
		for ( const [ name, value ] of Object.entries( data ) ) {
			flattenFormData( formData, `${ key }[${ name }]`, value );
		}
	}
	// ...
}
```

Resulting payload: `image_size[0]=post-thumbnail`, `image_size[1]=gform-image-choice-lg`.

**Other changes**
- `sideload-to-server.ts` and `upload-to-server.ts`: removed the `value as string | Record<string,string> | undefined` casts before calling `flattenFormData()`.
- `types.ts`: `image_size` widened to `string | string[]` on `CreateSideloadFile`, `SideloadAdditionalData` and `SubSizeData`.
- JSDoc on `flattenFormData` aligned with the directory's TypeScript convention (no type braces).
- `CHANGELOG.md`: adds a Bug Fixes entry under Unreleased.

**Tests**
- Vitest unit test in `test/flatten-form-data.ts` asserts that `['post-thumbnail', 'gform-image-choice-lg']` yields `image_size[0]` and `image_size[1]` entries. The existing nested-data test drops its cast.
- New e2e plugin `packages/e2e-tests/plugins/image-size-duplicates.php` registers `duplicate-size-one` and `duplicate-size-two` at 400×400 uncropped.
- New e2e spec in `client-side-media-processing.spec.js` records every `/wp/v2/media/{id}/sideload` response status, asserts none are >= 400, and checks both size names exist in `media_details.sizes` with the same `source_url`.

## Contribution

Opened by first-time contributor @antwonw to close #82348, as a follow-up to the image-size grouping work in #77035 and #77036. @adamsilverstein confirmed it as a clear regression, pushed an e2e test to the branch ahead of the 7.1 backport, and verified the fix locally before and after. A Claude Code review then suggested three nits (merge the array and plain-object branches, drop a leftover cast in `upload-to-server.ts`, align JSDoc style), which @antwonw applied in a follow-up commit. The author kept the e2e test in the same PR rather than splitting it out.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
