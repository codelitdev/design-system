# @codelitdev/design-system

Shared tokens, Tailwind preset, and React primitives for CourseLit, MediaLit, SendLit, and FrontLit.

The **tokens + Tailwind preset are the product**; the React components are
reference/fallback (see step 5). Consuming apps typically use only the CSS and
the preset and keep their own shadcn components.

## Install

Published to npm as a public package. The current release line is a prerelease
under the `alpha` dist-tag.

> [!WARNING]
> **`latest` currently also points at `0.1.0-alpha.0`.** npm always sets
> `latest` on a package's first-ever publish, regardless of `--tag` — this is
> unavoidable until a real stable version is cut, at which point publishing it
> moves `latest` forward automatically. Until then, **always pin the exact
> version** — do not `npm install @codelitdev/design-system` bare, and do not
> use a caret/tilde range (`^0.1.0-alpha.0` would still resolve to a
> prerelease, which is at least explicit — but a bare install with no version
> silently means "whatever `latest` is," which right now is this alpha).

```sh
npm install @codelitdev/design-system@0.1.0-alpha.0
```

Inside a workspace/monorepo you can still consume it as `workspace:*` when the
package is vendored locally.

## Use in a shadcn app

1. Import the token layer once, e.g. in `app/globals.css`:
```css
@import "@codelitdev/design-system/styles.css";
```
This replaces the default shadcn `:root`/`.dark` block — variable names match, so existing shadcn components need no code changes.

2. Extend your Tailwind config:
```js
// tailwind.config.js
const codelitPreset = require("@codelitdev/design-system/tailwind-preset");
module.exports = {
  presets: [codelitPreset],
  content: [...],
};
```

3. Scope the product on your root layout:
```tsx
<body data-product="sendlit">  {/* courselit | medialit | sendlit | frontlit, omit for CodeLit amber */}
```

4. Fonts: load Hanken Grotesk + Spline Sans Mono via `next/font/google` (or your framework's font loader), mapped to `--font-sans` / `--font-mono`.

5. Components in `src/components/` (Button, IconButton, Badge, Card, Tabs, Dialog, Toast, Tooltip, Checkbox, Input, Radio, Select, Switch) are plain React + CSS classes (`components.css`), not Radix-based — treat them as reference/fallback. For shadcn's Radix-based primitives (Select, Dialog, Tabs, Tooltip), keep your app's existing shadcn components and port the visual deltas from `components.css` into their CVA variants instead of importing these directly.

6. Logos: `@codelitdev/design-system/assets/logo-<product>.svg`.

## Publishing

Prereleases go under the `alpha` dist-tag so a plain `npm install` never grabs
one. **Always publish with an explicit tag** — `publishConfig.tag` is not
honored by npm 11.x, so a bare `npm publish` would land on `latest`:

```sh
npm run release:alpha            # → npm publish --tag alpha
# add --otp=<code> if 2FA is enabled
```

Verify: `npm view @codelitdev/design-system dist-tags` should show the
prerelease under `alpha` and leave `latest` untouched.

## License

Apache-2.0 — see [LICENSE](./LICENSE).
