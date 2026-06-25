---
name: setup-pre-commit
description: Set up Husky pre-commit hooks with lint-staged, Prettier, type checking, and tests in the current repo. Use when adding pre-commit hooks, configuring lint-staged, or adding commit-time formatting/typechecking/testing.
---

# Setup Pre-Commit Hooks

Set up a repo-appropriate Husky pre-commit hook using the existing package manager and scripts. Prefer the smallest change that fits the repo.

## What This Sets Up

- Husky pre-commit hook.
- `lint-staged` running Prettier on staged files.
- Prettier config only if the repo does not already have one.
- Existing typecheck/check and test scripts in the pre-commit hook when available.

## Process

### 1. Inspect The Repo

Read `package.json` and check for lockfiles:

- `package-lock.json` - npm.
- `pnpm-lock.yaml` - pnpm.
- `yarn.lock` - yarn.
- `bun.lock` or `bun.lockb` - bun.

Use the package manager already used by the repo. Default to npm only if unclear.

Check existing scripts before writing the hook. Prefer, in order:

- Type checking: `typecheck`, `check`.
- Tests: `test`, `test:unit`.
- Formatting: `format`, otherwise `lint-staged` with Prettier.

### 2. Install Dependencies

Install these as dev dependencies using the detected package manager:

```text
husky lint-staged prettier
```

Skip dependencies already present.

### 3. Initialize Husky

Use the package-manager equivalent of Husky init. For npm:

```bash
npx husky init
```

This creates `.husky/` and usually adds `prepare: "husky"` to `package.json`.

### 4. Write `.husky/pre-commit`

Use the detected package manager's runner. Example for npm:

```bash
npx lint-staged
npm run typecheck
npm run test
```

Adapt the script names to what actually exists in `package.json`. If the repo has no typecheck/check or test script, omit that line and mention it in the final response.

Husky v9+ hook files do not need a shebang.

### 5. Configure Lint-Staged

Create `.lintstagedrc` only if no lint-staged config already exists:

```json
{
  "*": "prettier --ignore-unknown --write"
}
```

If a lint-staged config already exists, preserve it and make the smallest needed edit.

### 6. Configure Prettier

Create `.prettierrc` only if no Prettier config exists. Use these defaults:

```json
{
  "useTabs": false,
  "tabWidth": 2,
  "printWidth": 80,
  "singleQuote": false,
  "trailingComma": "es5",
  "semi": true,
  "arrowParens": "always"
}
```

If the repo already has a Prettier config, do not replace it.

### 7. Verify

Verify the setup before finishing:

- `.husky/pre-commit` exists and is executable.
- `prepare` script in `package.json` is present.
- lint-staged config exists.
- Prettier config exists or the repo intentionally relies on defaults.
- Run the package-manager equivalent of `lint-staged` if feasible.

## Rules

- Do not create a commit unless the user explicitly asks.
- Do not overwrite an existing formatting, lint-staged, or hook setup.
- If the hook would be too slow because the repo has expensive tests, ask before adding the full test command.
- Keep the final response factual: files changed, scripts added, and verification run.
