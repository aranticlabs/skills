---
name: dead-code-scanner
description: Scan a JavaScript/TypeScript codebase for dead code — unused files, unused exports, unreferenced types, unused dependencies, and commented-out blocks. Uses knip (or an equivalent analyzer) as the primary tool plus targeted manual checks. Works on any JS/TS project; adapts to monorepos, single packages, frontend, backend, or full-stack.
---

## Dead Code Scanner

You are performing a dead code audit of a JavaScript/TypeScript codebase. The skill is project-agnostic — discover the project's structure and tooling first, then run the audit.

### Step 0 — Discover the project

Before running any tool, learn enough about the project to scan it correctly:

1. Read `package.json` (or root `package.json` plus any workspace packages) to identify:
   - Package manager (`npm`, `pnpm`, `yarn`, `bun`) from the `packageManager` field, lockfile, or `engines`
   - Workspaces / monorepo layout (`workspaces`, `pnpm-workspace.yaml`, `turbo.json`, `nx.json`, `lerna.json`)
   - Scripts that already wrap a dead-code tool (`knip`, `ts-prune`, `depcheck`, `unimported`)
2. Check for a knip config (`knip.json`, `knip.jsonc`, `knip.config.ts`, `knip.config.js`) — its `entry` and `project` globs tell you what the project considers entry points (false-positive signal).
3. Identify the source roots. Common layouts: `src/`, `app/`, `packages/*/src/`, `apps/*/src/`. Do not assume `src/server` and `src/admin` exist — those are project-specific.
4. Identify the framework(s) in use (React, Vue, Svelte, Next.js, Remix, Astro, Express, Hono, Fastify, NestJS, etc.) so you can recognize framework-specific entry points (e.g., Next.js `app/`, Remix routes, Astro pages) that are loaded by convention, not by import.

State what you found in 3–5 lines before proceeding.

### Step 1 — Run a dead-code analyzer

Prefer in this order:

1. **knip** if it's already a dependency or there's a `knip.*` config — run `npx knip` (or `pnpm knip` / `bun knip` / `yarn knip` to match the project's package manager). If the project defines a script (e.g., `npm run knip`, `make knip`), use that.
2. **ts-prune** + **depcheck** as a fallback if knip is not configured: `npx ts-prune` for unused exports, `npx depcheck` for unused dependencies. Note these are noisier than knip.
3. **None installed** — offer to install knip as a dev dependency (`<pm> add -D knip`) before scanning, or proceed with manual checks only. Ask the user before installing.

Capture the full output. The analyzer will report some combination of:

- **Unused files** — source files never imported
- **Unused exports** — named exports that nothing imports
- **Unused exported types** — TS-specific
- **Unused dependencies** — packages in `package.json` not used in code
- **Unlisted dependencies** — imports not declared in `package.json`
- **Duplicate exports**

Read the output carefully. Common false-positive sources:

- Entry points registered in the analyzer's config (intentionally not imported)
- Dynamic imports with variable paths (`await import(\`./locales/${lang}.ts\`)`)
- Files loaded by build/bundler config (`vite.config.*`, `webpack.config.*`, `next.config.*`, `astro.config.*`, `vitest.config.*`, `tsup.config.*`)
- Framework convention-loaded files (Next.js `page.tsx`/`route.ts`, Remix `routes/*.tsx`, Astro `pages/*.astro`, NestJS controllers/modules picked up by decorators)
- Type-only re-exports intended for downstream consumers
- Files referenced only from non-code (HTML `<script>` tags, JSON manifests, CSS `url()`)

Annotate each finding as **confirmed dead** or **false positive** with a brief reason.

### Step 2 — Manual checks: commented-out and stale code

These checks apply to any codebase. Scope them to the project's source roots discovered in Step 0.

**Commented-out code blocks** (sign of incomplete cleanup):

Use Grep with `pattern: "^\s*//"` across the source roots (e.g., `src/**/*.{ts,tsx,js,jsx}`) to find densely commented sections. Flag files with 3+ consecutive commented code lines.

**TODO/FIXME without ticket references**:

Use Grep with `pattern: "TODO|FIXME|HACK|XXX"`. Flag any without a linked issue number, ticket ID, or dated owner note.

**Stale feature flags / `if (false)` guards**:

Grep for `if (false)`, `if (0)`, ` || false ` patterns, and obvious flag constants that are hard-coded to a single value.

### Step 3 — Manual checks: structural orphans

Adapt these to the project's actual layout (discovered in Step 0):

- **Module orphans** — in any folder that exports through an `index.ts`/`index.js` barrel, check whether every sibling file is referenced by the barrel or by another file in the module.
- **Unused components** — for component folders (e.g., `components/`, `ui/`, `widgets/`), grep each component name across the rest of the source tree. Zero non-self references = candidate dead component. Watch for components rendered via string keys (e.g., dynamic component maps) — those will be false positives.
- **Unused hooks / composables / stores** — same approach for `hooks/`, `composables/`, `stores/`.
- **Unused shared types** — in a monorepo, types in a shared package not imported by any consumer package.

Use the project's actual paths; do not invent ones that don't exist.

### Step 4 — Dependency audit

From the analyzer's unused-dependencies list, verify each one:

1. Is the package actually used somewhere the analyzer missed? Check:
   - Dynamic `require()` / `import()`
   - Config files (`*.config.*`, `.eslintrc.*`, `tsconfig.json` `extends`)
   - CLI invocations in `package.json` scripts
   - Peer dependencies of frameworks (some packages are loaded by name, e.g., Tailwind plugins, PostCSS plugins, Babel presets, ESLint configs)
2. If genuinely unused: note it for removal with the project's package manager (`npm uninstall`, `pnpm remove`, `yarn remove`, `bun remove`). **Do not remove without user confirmation.**

### Step 5 — Report

Structure the output as:

#### Confirmed Dead Code

| Type          | File / Symbol                    | Confidence | Recommended Action |
| ------------- | -------------------------------- | ---------- | ------------------ |
| Unused file   | `<path>`                         | High       | Delete             |
| Unused export | `<module>.<symbol>`              | High       | Remove             |
| Unused dep    | `<package>`                      | Medium     | Verify then remove |

Use the project's real paths — do not put `src/admin/...` examples in the report unless that folder actually exists.

#### False Positives / Needs Review

List anything the analyzer flagged that appears to be intentional, with your reasoning (entry point, framework convention, dynamic import, etc.).

#### Commented-Out Code

List files with significant commented-out blocks. Suggest: delete if dead, restore if needed, or open a ticket if unclear.

#### Summary

- Total confirmed dead files: N
- Total unused exports: N
- Total unused dependencies: N
- Estimated cleanup effort: Small / Medium / Large
- Suggested next step: a single concrete action the user can take (e.g., "delete the 4 confirmed-dead files in one commit", "run `pnpm remove <pkgs>` after verifying X")
