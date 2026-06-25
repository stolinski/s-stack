# Chrome DevTools MCP verification for Dex implementation tasks

Use this when a Dex task changes a browser app and the project expects in-browser verification, especially Scott's SvelteKit projects.

## Pattern

1. Start the Dex task before editing:
   ```bash
   dex start <task_id>
   dex show <task_id> --full
   ```

2. Make the smallest coherent implementation change.

3. Run the code gates:
   ```bash
   npm run check
   npm run build
   ```

4. Verify with Chrome DevTools MCP, not just DOM/source inspection:
   - navigate or reload the running app
   - take an accessibility snapshot
   - click through the changed flow
   - check console messages
   - take a screenshot when visual layout or highlighting matters

5. Put concrete MCP observations in the Dex completion result:
   - route/page loaded
   - visible labels/counts changed as expected
   - interaction changed state as expected
   - console errors/warnings observed, or "only Vite debug messages"
   - screenshot path if one was captured

## Example completion evidence

```text
Verification: npm run check passed with 0 errors/0 warnings; npm run build passed; Chrome DevTools MCP verified page reload, mission list count, Reference Detail expansion, Move Map selection, active control list update, completion reaching 100%, and console messages showed only Vite debug connection logs. Screenshot saved at /tmp/example.png.
```

## Pitfalls

- Do not complete a Dex task for UI work with only `npm run check`/`build`; browser behavior is part of the task.
- Do not say "browser tested" generically. Name the exact flow clicked and the observed state change.
- DevTools MCP snapshots are evidence, but visual work also needs screenshot/eyeball verification when layout, clipping, or highlighting matters.
