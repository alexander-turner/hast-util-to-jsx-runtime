# hast-util-to-jsx-runtime

[![Build][build-badge]][build]
[![Coverage][coverage-badge]][coverage]
[![Downloads][downloads-badge]][downloads]
[![Size][size-badge]][size]
[![Sponsors][sponsors-badge]][collective]
[![Backers][backers-badge]][collective]
[![Chat][chat-badge]][chat]

hast utility to transform a tree to preact, react, solid, svelte, vue, etc.,
with an automatic JSX runtime.

## Contents

*   [What is this?](#what-is-this)
*   [When should I use this?](#when-should-i-use-this)
*   [Install](#install)
*   [Use](#use)
*   [API](#api)
    *   [`toJsxRuntime(tree, options)`](#tojsxruntimetree-options)
    *   [`Options`](#options)
    *   [`Components`](#components-1)
    *   [`ElementAttributeNameCase`](#elementattributenamecase-1)
    *   [`Fragment`](#fragment-1)
    *   [`Jsx`](#jsx-1)
    *   [`JsxDev`](#jsxdev-1)
    *   [`Props`](#props)
    *   [`Source`](#source)
    *   [`Space`](#space-1)
    *   [`StylePropertyNameCase`](#stylepropertynamecase-1)
*   [Examples](#examples)
    *   [Example: Preact](#example-preact)
    *   [Example: Vue](#example-vue)
    *   [Example: Solid](#example-solid)
    *   [Example: Svelte](#example-svelte)
*   [Syntax](#syntax)
*   [Types](#types)
*   [Compatibility](#compatibility)
*   [Security](#security)
*   [Related](#related)
*   [Contribute](#contribute)
*   [License](#license)

## What is this?

This package is a utility that takes a [hast][] tree and an
[automatic JSX runtime][jsx-runtime] and turns the tree into anything you
whish.

## When should I use this?

You can use this package when you have a hast syntax tree and want to use it
with whatever framework.

This package uses an automatic JSX runtime, which is a sort of lingua franca
for frameworks to support JSX.

Notably, automatic runtimes have support for passing extra information in
development, and have guaranteed support for fragments.

## Install

This package is [ESM only][esm].
In Node.js (version 14.14+ and 16.0+), install with [npm][]:

```sh
npm install hast-util-to-jsx-runtime
```

In Deno with [`esm.sh`][esmsh]:

```js
import {toJsxRuntime} from 'https://esm.sh/hast-util-to-jsx-runtime@1'
```

In browsers with [`esm.sh`][esmsh]:

```html
<script type="module">
  import {toJsxRuntime} from 'https://esm.sh/hast-util-to-jsx-runtime@1?bundle'
</script>
```

## Use

```js
import {h} from 'hastscript'
import {toJsxRuntime} from 'hast-util-to-jsx-runtime'
import {Fragment, jsx, jsxs} from 'react/jsx-runtime'
import {renderToStaticMarkup} from 'react-dom/server'

const tree = h('h1', 'Hello, world!')

const doc = renderToStaticMarkup(toJsxRuntime(tree, {Fragment, jsx, jsxs}))

console.log(doc)
```

Yields:

```html
<h1>Hello, world!</h1>
```

## API

This package exports the identifier [`toJsxRuntime`][api-to-jsx-runtime].
There is no default export.

### `toJsxRuntime(tree, options)`

Transform a hast tree to preact, react, solid, svelte, vue, etc., with an
automatic JSX runtime.

###### Parameters

*   `tree` ([`Node`][node])
    — tree to transform
*   `options` ([`Options`][api-options], required)
    — configuration

###### Returns

Result from your configured JSX runtime (`JSX.Element`).

### `Options`

Configuration (TypeScript type).

##### Fields

###### `Fragment`

Fragment ([`Fragment`][api-fragment], required).

###### `jsx`

Dynamic JSX ([`Jsx`][api-jsx], required in production).

###### `jsxs`

Static JSX ([`Jsx`][api-jsx], required in production).

###### `jsxDEV`

Development JSX ([`JsxDev`][api-jsx-dev], required in development).

###### `development`

Whether to use `jsxDEV` when on or `jsx` and `jsxs` when off (`boolean`,
default: `false`).

###### `components`

Components to use ([`Partial<Components>`][api-components], optional).

Each key is the name of an HTML (or SVG) element to override.
The value is the component to render instead.

###### `elementAttributeNameCase`

Specify casing to use for attribute names
([`ElementAttributeNameCase`][api-element-attribute-name-case], default:
`'react'`).

###### `filePath`

File path to the original source file (`string`, optional).

Passed in source info to `jsxDEV` when using the automatic runtime with
`development: true`.

###### `passNode`

Pass the hast element node to components (`boolean`, default: `false`).

###### `space`

Whether `tree` is in the `'html'` or `'svg'` space ([`Space`][api-space],
default: `'html'`).

When an `<svg>` element is found in the HTML space, this package already
automatically switches to and from the SVG space when entering and exiting
it.

> 👉 **Note**: hast is not XML.
> It supports SVG as embedded in HTML.
> It does not support the features available in XML.
> Passing SVG might break but fragments of modern SVG should be fine.
> Use `xast` if you need to support SVG as XML.

###### `stylePropertyNameCase`

Specify casing to use for property names in `style` objects
([`StylePropertyNameCase`][api-style-property-name-case], default:
`'dom'`).

### `Components`

Possible components to use (TypeScript type).

Each key is a tag name typed in `JSX.IntrinsicElements`.
Each value is either a different tag name, or a component accepting the
corresponding props (and an optional `node` prop if `passNode` is on).

You can access props at `JSX.IntrinsicElements`.
For example, to find props for `a`, use `JSX.IntrinsicElements['a']`.

###### Type

```ts
import type {Element} from 'hast'

type Components = {
  [TagName in keyof JSX.IntrinsicElements]:
    | Component<JSX.IntrinsicElements[TagName] & ExtraProps>
    | keyof JSX.IntrinsicElements
}

type ExtraProps = {node?: Element | undefined}

type Component<ComponentProps> =
  // Function component:
  | ((props: ComponentProps) => JSX.Element | string | null | undefined)
  // Class component:
  | (new (props: ComponentProps) => JSX.ElementClass)
```

### `ElementAttributeNameCase`

Casing to use for attribute names (TypeScript type).

HTML casing is for example `class`, `stroke-linecap`, `xml:lang`.
React casing is for example `className`, `strokeLinecap`, `xmlLang`.

###### Type

```ts
type ElementAttributeNameCase = 'react' | 'html'
```

### `Fragment`

Represent the children, typically a symbol (TypeScript type).

###### Type

```ts
type Fragment = unknown
```

### `Jsx`

Create a production element (TypeScript type).

###### Parameters

*   `type` (`unknown`)
    — element type: `Fragment` symbol, tag name (`string`), component
*   `props` ([`Props`][api-props])
    — element props, `children`, and maybe `node`
*   `key` (`string` or `undefined`)
    — dynamicly generated key to use

###### Returns

An element from your framework (`JSX.Element`).

### `JsxDev`

Create a development element (TypeScript type).

###### Parameters

*   `type` (`unknown`)
    — element type: `Fragment` symbol, tag name (`string`), component
*   `props` ([`Props`][api-props])
    — element props, `children`, and maybe `node`
*   `key` (`string` or `undefined`)
    — dynamicly generated key to use
*   `isStaticChildren` (`boolean`)
    — whether more than one children are used
*   `source` ([`Source`][api-source])
    — info about source
*   `self` (`undefined`)
    — nothing (this is used by frameworks that have components, we don’t)

###### Returns

An element from your framework (`JSX.Element`).

### `Props`

Properties and children (TypeScript type).

###### Type

```ts
import type {Element} from 'hast'

type Props = {
  [prop: string]:
    | Element // For `node`.
    | Array<JSX.Element | string | null | undefined> // For `children`.
    | Record<string, string> // For `style`.
    | string
    | number
    | boolean
    | undefined
  children: Array<JSX.Element | string | null | undefined> | undefined
  node?: Element | undefined
}
```

### `Source`

Info about source (TypeScript type).

###### Fields

*   `fileName` (`string` or `undefined`)
    — name of source file
*   `lineNumber` (`number` or `undefined`)
    — line where thing starts (1-indexed)
*   `columnNumber` (`number` or `undefined`)
    — column where thing starts (0-indexed)

### `Space`

Namespace (TypeScript type).

###### Type

```ts
type Space = 'html' | 'svg'
```

### `StylePropertyNameCase`

Casing to use for property names in `style` objects (TypeScript type).

CSS casing is for example `background-color` and `-webkit-line-clamp`.
DOM casing is for example `backgroundColor` and `WebkitLineClamp`.

###### Type

```ts
type StylePropertyNameCase = 'dom' | 'css'
```

## Examples

### Example: Preact

```js
import {h} from 'hastscript'
import {toJsxRuntime} from 'hast-util-to-jsx-runtime'
import {Fragment, jsx, jsxs} from 'preact/jsx-runtime'
import {render} from 'preact-render-to-string'

const result = render(toJsxRuntime(h('h1', 'hi!'), {Fragment, jsx, jsxs}))

console.log(result)
```

Yields:

```html
<h1>hi!</h1>
```

### Example: Vue

```js
import {h} from 'hastscript'
import {toJsxRuntime} from 'hast-util-to-jsx-runtime'
import {Fragment, jsx, jsxs} from 'vue-jsx-runtime/jsx-runtime'
import serverRenderer from '@vue/server-renderer'

console.log(
  await serverRenderer.renderToString(
    toJsxRuntime(h('h1', 'hi!'), {Fragment, jsx, jsxs})
  )
)
```

Yields:

```html
<h1>hi!</h1>
```

### Example: Solid

```js
import {h} from 'hastscript'
import {toJsxRuntime} from 'hast-util-to-jsx-runtime'
import {Fragment, jsx, jsxs} from 'solid-jsx/jsx-runtime'

console.log(toJsxRuntime(h('h1', 'hi!'), {Fragment, jsx, jsxs}).t)
```

Yields:

```html
<h1 >hi!</h1>
```

### Example: Svelte

I have no clue how to render a Svelte component in Node, but you can get that
component with:

```js
import {h} from 'hastscript'
import {toJsxRuntime} from 'hast-util-to-jsx-runtime'
import {Fragment, jsx, jsxs} from 'svelte-jsx'

const svelteComponent = toJsxRuntime(h('h1', 'hi!'), {Fragment, jsx, jsxs})

console.log(svelteComponent)
```

Yields:

```text
[class Component extends SvelteComponent]
```

## Syntax

HTML is parsed according to WHATWG HTML (the living standard), which is also
followed by browsers such as Chrome, Firefox, and Safari.

## Types

This package is fully typed with [TypeScript][].
It exports the additional types [`Components`][api-components],
[`ElementAttributeNameCase`][api-element-attribute-name-case],
[`Fragment`][api-fragment], [`Jsx`][api-jsx], [`JsxDev`][api-jsx-dev],
[`Options`][api-options], [`Props`][api-props], [`Source`][api-source],
[`Space`][api-Space],
and [`StylePropertyNameCase`][api-style-property-name-case].

The function `toJsxRuntime` returns a `JSX.Element`, which means that the JSX
namespace has to by typed.
Typically this is done by installing your frameworks types (e.g.,
`@types/react`), which you likely already have.

## Compatibility

Projects maintained by the unified collective are compatible with all maintained
versions of Node.js.
As of now, that is Node.js 14.14+ and 16.0+.
Our projects sometimes work with older versions, but this is not guaranteed.

## Security

Be careful with user input in your hast tree.
Use [`hast-util-santize`][hast-util-sanitize] to make hast trees safe.

## Related

*   [`hastscript`](https://github.com/syntax-tree/hastscript)
    — build hast trees
*   [`hast-util-to-html`](https://github.com/syntax-tree/hast-util-to-html)
    — serialize hast as HTML
*   [`hast-util-sanitize`](https://github.com/syntax-tree/hast-util-sanitize)
    — sanitize hast

## Contribute

See [`contributing.md`][contributing] in [`syntax-tree/.github`][health] for
ways to get started.
See [`support.md`][support] for ways to get help.

This project has a [code of conduct][coc].
By interacting with this repository, organization, or community you agree to
abide by its terms.

## License

[MIT][license] © [Titus Wormer][author]

<!-- Definitions -->

[build-badge]: https://github.com/syntax-tree/hast-util-to-jsx-runtime/workflows/main/badge.svg

[build]: https://github.com/syntax-tree/hast-util-to-jsx-runtime/actions

[coverage-badge]: https://img.shields.io/codecov/c/github/syntax-tree/hast-util-to-jsx-runtime.svg

[coverage]: https://codecov.io/github/syntax-tree/hast-util-to-jsx-runtime

[downloads-badge]: https://img.shields.io/npm/dm/hast-util-to-jsx-runtime.svg

[downloads]: https://www.npmjs.com/package/hast-util-to-jsx-runtime

[size-badge]: https://img.shields.io/bundlephobia/minzip/hast-util-to-jsx-runtime.svg

[size]: https://bundlephobia.com/result?p=hast-util-to-jsx-runtime

[sponsors-badge]: https://opencollective.com/unified/sponsors/badge.svg

[backers-badge]: https://opencollective.com/unified/backers/badge.svg

[collective]: https://opencollective.com/unified

[chat-badge]: https://img.shields.io/badge/chat-discussions-success.svg

[chat]: https://github.com/syntax-tree/unist/discussions

[npm]: https://docs.npmjs.com/cli/install

[esm]: https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c

[esmsh]: https://esm.sh

[typescript]: https://www.typescriptlang.org

[license]: license

[author]: https://wooorm.com

[health]: https://github.com/syntax-tree/.github

[contributing]: https://github.com/syntax-tree/.github/blob/main/contributing.md

[support]: https://github.com/syntax-tree/.github/blob/main/support.md

[coc]: https://github.com/syntax-tree/.github/blob/main/code-of-conduct.md

[hast]: https://github.com/syntax-tree/hast

[node]: https://github.com/syntax-tree/hast#nodes

[hast-util-sanitize]: https://github.com/syntax-tree/hast-util-sanitize

[jsx-runtime]: https://reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html

[api-to-jsx-runtime]: #tojsxruntimetree-options

[api-components]: #components-1

[api-element-attribute-name-case]: #elementattributenamecase-1

[api-fragment]: #fragment-1

[api-jsx]: #jsx-1

[api-jsx-dev]: #jsxdev-1

[api-options]: #options

[api-props]: #props

[api-source]: #source

[api-space]: #space-1

[api-style-property-name-case]: #stylepropertynamecase-1
