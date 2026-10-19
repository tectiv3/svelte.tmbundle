# Svelte bundle for TextMate 2

Grammar, folding and indentation for Svelte components (`.svelte`) in TextMate 2.
Language scope: `source.svelte`.

There is no official Svelte bundle in the TextMate managed bundle list. This one wraps
the grammar from [`jasonlyu123/svelte-textmate-grammar`](https://github.com/jasonlyu123/svelte-textmate-grammar)
(MIT) — the classic James Birtles grammar forked from `sveltejs/language-tools` before
that project moved to an injection-based grammar. The grammar is vendored into
`Syntaxes/Svelte.tmLanguage`; it is not synced from upstream.

## Install

```sh
git clone https://github.com/tectiv3/svelte.tmbundle.git \
  ~/Library/Application\ Support/TextMate/Bundles/Svelte.tmbundle
```

For development, symlink a checkout instead of copying it:

```sh
ln -s /path/to/svelte.tmbundle ~/Library/Application\ Support/TextMate/Bundles/Svelte.tmbundle
```

Then **Bundles → Reload Bundles** (or restart TextMate). Bundles in
`~/Library/Application Support/TextMate/Bundles` take priority over `Pristine Copy/Bundles`,
`Managed/Bundles`, `/Library/...` and the app's `SharedSupport/Bundles`.

## Hard dependency: a TypeScript grammar

Every `<script>` rule in the grammar — including plain JavaScript — declares
`contentName: source.ts` and `include: source.ts`. **Without a `source.ts` grammar, script
bodies get no highlighting at all**, only a flat scope. Install
[`stanger/TypeScript-TextMate`](https://github.com/stanger/TypeScript-TextMate) (also absent
from the official bundle list). The Svelte grammar references three rules from it
(`type`, `comment`, `punctuation-comma`), which that bundle provides.

`<style>` needs no extra bundle: `source.css` ships with TextMate.

## Installation with unsupported preprocessors

`<style lang="scss|less|sass|postcss">`, `<script lang="coffee">` and `<template lang="pug">`
include grammar scopes that TextMate does not ship (`source.css.scss`, `source.css.less`,
`source.sass`, `source.postcss`, `source.coffee`, `text.pug`). Without the corresponding
bundle installed, those blocks render flat. This is deliberate: silently falling back to
`source.css` or `source.ts` would produce wrong highlighting rather than none.

## Known gaps

1. Element text content is scoped only as `source.svelte` — no `text.html.basic`. The classic
   grammar never wrapped element bodies.
2. Snippet parameters scope as TypeScript object-literal labels (`item` →
   `entity.name.label.ts`), not `variable.parameter.ts`.
3. `{:else if cond}` falls through to the generic `{…}` rule and is scoped as TypeScript,
   while `{:else}` is `keyword.control.conditional`.
4. `{#await p then v}` gives `then`/`v` no keyword scope.
5. `<li>x</li>` on one line matches both the fold-start and fold-stop pattern, so it counts as
   a fold start. Inherited from `HTML.tmbundle`.
6. Folding has no level for `{:else}`, `{:then}`, `{:catch}`; `<svelte:…>` tags are not fold
   starts.

## License

MIT — see [LICENSE](LICENSE). The grammar is derivative of `jasonlyu123/svelte-textmate-grammar`;
the bundle wrapper, folding and indent rules are new work.
