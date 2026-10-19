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

## Folding and indentation

Folding and indent markers cover the HTML block elements, the `{#if}`/`{#each}`/`{#await}`/
`{#key}`/`{#snippet}` blocks, and the four `svelte:` tags that can hold markup: `svelte:head`,
`svelte:fragment`, `svelte:element`, `svelte:component`.

The childless `svelte:` tags (`window`, `body`, `document`, `self`, `options`) are deliberately
excluded: the self-closing guard ends at the first `>`, which inside an inline handler such as
`<svelte:window onkeydown={(e) => go(e)} />` is the arrow's own `>`, so including them would
open a fold and add an indent level for a self-closing element.

Components (`<Card>`) are not fold starts — `HTML.tmbundle` does not fold custom elements
either.

## Known gaps

1. Types in snippet and component signatures are not scoped as types: in
   `{#snippet row(item: Item)}` the annotation `Item` is `variable.other.readwrite.ts`, not
   `entity.name.type.ts`. Parameter *names* and the `?`/`:` markers are correct; what follows
   the colon is handed to `source.ts`, which has no way to know it is in a type position.
2. In a snippet parameter list, `...rest: any[]` leaves `rest` unscoped as a parameter, a
   parameter whose `?`/`=`/`:` sits on a later line than its name is not recognised, and
   destructured parameters (`({p, q}: Props)`) are parsed by `source.ts` as an object pattern
   rather than parameter-scoped.
3. `{:else if cond}` falls through to the generic `{…}` rule: `else`/`if` come out as
   `keyword.control.conditional.ts` rather than `keyword.control.conditional`. Same colour class
   in practice, so it is cosmetic. `{#await p then v}` scopes `v` as `variable`.
4. `<li>x</li>` on one line matches both the fold-start and the fold-stop pattern. The engine
   resolves that as a start, so the line counts as an unclosed fold. Identical to
   `HTML.tmbundle`; not fixable without deviating from it.
5. Folding cannot give `{:else}`, `{:then}` and `{:catch}` their own level. In
   `Frameworks/layout/src/folds.cc` a line matching both the start and the stop marker takes
   the start branch and the stop is never processed, so a marker that should close the previous
   branch and open the next one instead opens an unclosed fold and steals a later closer. This
   is a limitation of the line-marker model, not of these patterns.
6. A bare `&` in text is `invalid.illegal.bad-ampersand.html` (so `R&D` shows as an error) and
   a bare `<` in text is unscoped. Both come from the upstream `#entities` rules and match
   `HTML.tmbundle` behaviour.

## License

MIT — see [LICENSE](LICENSE). The grammar is derivative of `jasonlyu123/svelte-textmate-grammar`;
the bundle wrapper, folding and indent rules are new work.
