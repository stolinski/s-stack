# SvelteKit content model: raw Markdown Mission Files + typed registries

Use this when a SvelteKit app needs author-editable Markdown prose/high-level metadata while keeping UI-driving data type-safe.

Pattern:

- Put prose and broad metadata in Markdown/MDsveX-style files with frontmatter.
- Keep interaction-driving structures in typed TypeScript registries.
- Join them in a catalog/normalizer module.
- Validate the catalog at module load so bad content fails during `npm run check` / build / dev, not silently in the UI.

Example shape:

```ts
// mission-files.ts
export type MissionFile = {
	slug: string;
	deviceSlug: string;
	title: string;
	songGoal: string;
	difficulty: 'first session' | 'comfortable' | 'deep dive';
	time: string;
	techniques: string[];
	referenceDetail: string;
	filePath: string;
};

const missionModules = import.meta.glob('./missions/*.{md,svx}', {
	eager: true,
	query: '?raw',
	import: 'default'
}) as Record<string, string>;
```

Catalog validation checklist:

- every Mission File references a known Device
- every Mission File has matching Move Registry data
- every Move references a known Mission File slug
- every Move Control references a known `ControlId`
- Markdown body / Reference Detail is non-empty
- mission order is explicit, not filesystem-order accidental, when ordering matters

Why this is better than frontmatter-only:

- prose stays authorable
- UI-driving Move data remains typed
- invalid control IDs become TypeScript/catalog failures
- Mission Files do not turn into a YAML swamp

Verification:

1. Run `npm run check`.
2. Run `npm run build`.
3. Browser-verify that all loaded content appears: count cards, switch to a late item, inspect headings/technique chips/reference detail, and check console errors.
