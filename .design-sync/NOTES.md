# design-sync notes — @codelitdev/design-system

- **Repo history**: this package was first authored inside the SendLit monorepo
  at `sendlit/packages/design-system/` (as a `workspace:*` dep of `apps/web`),
  then extracted to its own standalone repo
  (`git@github.com:codelitdev/design-system.git`) on 2026-07-25 and published to
  npm as a public Apache-2.0 package so all four CodeLit products (CourseLit,
  MediaLit, SendLit, FrontLit — all live today) can consume it. First publish was
  `0.1.0-alpha.0` under the `alpha` dist-tag. Older notes below may say "this
  repo" / "the SendLit repo" interchangeably — they mean this package.
- No build step in this package: the only `scripts` entry is `release:alpha`
  (a publish helper, not a build), and
  `main`/`types`/`exports['.']` all point straight at `src/index.ts` (raw
  TypeScript, compiled inline by whichever app consumes it). The converter is
  pointed at `--entry ./src/index.ts` directly rather than a `dist/` build —
  confirmed with the user at sync time, not guessed.
- Every component ships a hand-authored `<Name>.prompt.md` sibling (e.g.
  `src/components/actions/Button.prompt.md`) written in the same voice/format
  design-sync itself would generate — these came from the original Claude
  Design authoring of this system. The sibling-doc auto-discovery only matches
  `<Name>.md`/`<Name>.mdx` (exact filename), so `.prompt.md` siblings are
  invisible to it — hence the full `docsMap` in config.json pinning all 13
  components to their existing `.prompt.md` files. This is a real gap in
  discovery for this repo's convention, not a lazy enumeration: don't remove
  it on a future re-sync without re-checking the sibling-match logic.
- These `.prompt.md` files are also the primary composition source for
  authored previews (design-sync §4.2 "curate before inventing") — each one
  already has a canonical JSX usage snippet.
- `cssEntry` copies its target file verbatim — it does NOT resolve local
  `@import`s. This repo's real `src/styles.css` is an @import-only manifest
  (fonts.css, colors.css, products.css, typography.css, layout.css,
  components.css) with no separate tokens package, so neither `cssEntry`
  alone nor `tokensGlob`/`tokensPkg` (which require a separate npm package)
  applied cleanly. Fix: a synthesized single file at
  `.design-sync/.cache/compiled-styles.css` — the same files concatenated in
  their original `@import` order (fonts.css first, since it itself starts
  with a remote Google Fonts `@import url(...)` that must stay first for CSS
  validity) — with `cssEntry` pointed at that. This file is regenerated, not
  committed (`.cache/` is gitignored); re-sync must regenerate it before
  building if any of the 6 source files changed. There is no automation for
  this yet — a future re-sync should re-run the same `cat` concatenation (see
  git history for the exact command) before invoking the converter, or a
  small script could be added to `.design-sync/` to do it automatically.
- Devdependencies added to `packages/design-system/package.json` to support
  the sync (all legitimate, low-risk additions — not sync-only cruft):
  `@types/react` (ts-morph prop extraction needs it — the converter's own
  `[DTS_REACT]` fix), `lucide-react@^0.544.0` (previews for IconButton/
  Tooltip needed a real icon, matching what `apps/web` already uses).
- `playwright` (latest, pins chromium build 1228 which matched the machine's
  pre-existing cached chromium) and `typescript` installed into the isolated
  `.ds-sync/node_modules` for the render check and `.d.ts` syntax check.
- Known benign warning: `package-validate.mjs`'s `.d.ts` TypeScript syntax
  check always prints "(.d.ts parse check skipped — typescript not in
  node_modules)" even with `typescript` installed in `.ds-sync/node_modules`
  — root cause is an ESM/CJS interop issue (`await import('typescript')`
  succeeds but named exports like `createSourceFile` come back `undefined`,
  which then throws and gets caught by the surrounding try/catch). Harmless:
  the check is explicitly best-effort/non-blocking, and the real `.d.ts`
  extraction (ts-morph, at build time) already succeeded 13/13 with no
  `[DTS_PARSE]` errors. Not fixed (would require forking a top-level script
  for a cosmetic message).
- `Dialog` needed `cfg.overrides.Dialog: {"cardMode": "single", "viewport":
  "480x420"}` — its scrim uses `position: fixed`, which escapes normal grid
  flow.
- All 13 components authored solo (no subagent fan-out) — small set (13
  components), so the full "solo first, then fan out" wave process wasn't
  needed.
- **Preview cards render in CodeLit base amber, not any product's accent —
  this is a deliberate decision, not a bug.** This DS is multi-product:
  `products.css` scopes `--primary`/`--primary-*`/`--ring` per
  `data-product="courselit|medialit|sendlit|frontlit"` on an ancestor, and
  with no attribute `--primary` falls back to `--brand-amber`
  (`oklch(0.66 0.14 60)`). The generated preview shell emits a bare `<body>`
  with no attribute, so cards show amber while e.g. SendLit's real UI is green
  (`oklch(0.6 0.14 150)` / `#319751`). Confirmed with the user on the first
  sync (2026-07-25): keep the cards product-neutral, since the project covers
  all four products. Do NOT "fix" this on a re-sync by scoping the previews.
  The risk it creates — an agent that forgets `data-product` silently shipping
  amber product UIs — is mitigated in `conventions.md`, which states the
  attribute is required when building for a specific product and warns not to
  infer accents from the cards. Keep that warning intact.
- design-sync's preview shell has no config hook for `<body>` attributes
  (checked: no `bodyAttrs`/`htmlAttrs` anywhere in lib/), and `lib/emit.mjs`
  must not be forked (it's the output contract with the app's self-check). So
  if cards ever DO need product scoping, the options are wrapping each
  authored preview's JSX in a `<div data-product="…">`, or exporting a real
  `ProductScope` component from the DS and wiring it via `cfg.provider`.

## Re-sync risks

- If this package ever gains a real build step (dist/ + compiled .d.ts),
  `--entry` should be repointed there — the raw-TS entry mode extracts prop
  types via ts-morph reading source directly, which is fine here but would be
  redundant/inconsistent with a proper build.
- The `docsMap` above is a full enumeration by necessity (see above) — if
  sibling-doc discovery ever adds `.prompt.md` support upstream, this map
  becomes redundant and could be dropped.
- `.design-sync/.cache/compiled-styles.css` (the `cssEntry` target) is
  regenerated, not committed — if any of the 6 source CSS files it
  concatenates change without a re-sync regenerating it first, the uploaded
  bundle silently ships stale styles. A re-sync must regenerate it (see the
  note above) before running the build/driver.
- `previews/` (13 files) compose against the current component APIs — if a
  component's props change shape, the affected preview(s) may need updating
  even though this NOTES.md won't automatically flag which ones.
- **Publish gotcha**: npm 11.x does NOT honor `publishConfig.tag`, so a bare
  `npm publish` lands a prerelease on the `latest` dist-tag (verified on npm
  11.6.0 — dry-run printed "with tag latest" despite `publishConfig.tag:
  alpha`). Always publish via `npm run release:alpha` (forces `--tag alpha`)
  or an explicit `npm publish --tag alpha`. `publishConfig.access: public` IS
  honored. When a stable release is eventually cut, THAT one goes to `latest`
  via a plain `npm publish` (no `release:alpha`).
