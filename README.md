# solid-export-repro

Reproduction repo for an Astro + Solid SSR import/export mismatch.

## Workspace layout

- app: Astro app
- lib: local linked Solid component library (package name: solid-lib)

The app depends on the local library via link:../lib, so the repro flow explicitly builds lib first.

## Reproduction steps

1. Install dependencies from repository root.

```bash
pnpm install
```

2. Build the local library.

```bash
pnpm -C lib build
```

3. Start Astro dev server.

```bash
pnpm -C app dev
```

4. Open the local URL printed by Astro.

## Current error

After startup and page request, Astro fails with:

```text
[ERROR] SyntaxError: [vite] The requested module 'solid-js/web' does not provide an export named 'use'
```

## Why this happens

- lib build output imports client helpers from solid-js/web:

```js
import { use, insert, template } from 'solid-js/web';
```

- During Astro SSR, module resolution targets solid-js/web server entry.
- In the installed solid-js version used by this repro, the server entry exports insert/template as server stubs but does not export use.
- Because use is missing at import time, Vite throws a SyntaxError before rendering can continue.

In short: the built library expects a client-side export shape, but Astro server runtime resolves a server-side export shape that is not fully isomorphic for this symbol.

## Relation to the referenced upstream work

Reference: https://github.com/ryansolid/dom-expressions/pull/345

That PR discusses making server exports more isomorphic (exporting client-side APIs from server builds and throwing runtime errors when called) so imports do not fail at module-evaluation time.

This repro demonstrates the class of failure that occurs when a symbol expected by compiled output (use) is not exported by the resolved server module.
