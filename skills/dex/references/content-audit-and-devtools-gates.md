# Content audit + Chrome DevTools MCP gates

Use this pattern when a Dex epic turns placeholder/scaffold content into real typed product content, especially Svelte/SvelteKit content registries.

## Pattern

1. Make the content model source of truth explicit in Dex:
   - content files own prose/metadata
   - typed registry owns UI-driving actions/controls
   - device/app registry owns metadata only

2. Complete individual content tasks first, then close the parent content-build task only after verification.

3. Add a repo script for content completeness before declaring the parent task done. A lightweight Node script is enough when the app already loads content through TypeScript modules.

4. Audit rules that worked well:
   - all expected launch slugs exist
   - each content file has required frontmatter
   - techniques/tags are non-empty
   - body prose exists
   - Reference Detail contains at least one Guide Excerpt when required
   - every content slug has matching typed registry entries
   - every registry entry has id, controls, and source/citation
   - every referenced Control ID exists
   - registry slugs do not point at missing content files

5. Add the script to `package.json`, e.g. `"audit:content": "node scripts/audit-content.mjs"`.

6. Verification gate before completing Dex tasks:
   - run `npm run audit:content`
   - run `npm run check`
   - run `npm run build`
   - use Chrome DevTools MCP if requested/available to verify the real browser flow, not just DOM text.

## Chrome DevTools MCP evidence to record in Dex result

For content-first browser apps, record concise evidence:

- all expected items render in the list
- each item opens
- expandable reference/detail content renders
- Move Map/card counts match typed data
- selecting a move updates the active control/summary surface
- completing moves reaches expected progress/completion state
- console contains no errors (Vite debug connection messages are fine in dev)
- screenshot path if useful

## Pitfall

Do not mark the parent content-build task complete just because individual content files exist. Close the parent only after the audit script and browser verification pass, then unblock downstream route/layout/persistence work.