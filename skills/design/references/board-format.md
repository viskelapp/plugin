# The dc board format

Derived from viskel `b00c7f5c` (`runtime/dc-core.js`, `spec/fixtures/format/`, `Templates::boardHtml()`, `ValidateBoardHtml.php`). Where this file and the server disagree, the server wins: trust what `validate` and `edit_file` report.

## Skeleton

A board is `pages/<page>/artboards/<Name>.dc.html`. `add_board` writes this skeleton; keep its shape.

```html
<!doctype html>
<html>
<head>
<meta charset="utf-8">
<title>Home</title>
<script src="/runtime/dc-runtime.js"></script>
</head>
<body>
<x-dc>
<helmet>
<style>
body { margin: 0; font-family: var(--font-sans, system-ui, sans-serif); color: var(--color-text, #17171c); background: var(--color-bg, #ffffff); }
</style>
</helmet>
<main style="padding: 48px;">
  <h1 style="margin: 0;">{{title}}</h1>
</main>
</x-dc>
<script type="text/x-dc" data-props='{"title":{"editor":"text","default":"Home"},"$preview":{"width":1440,"height":900}}'>
class Component extends DCLogic {
  renderVals() {
    return {};
  }
}
</script>
</body>
</html>
```

- `<x-dc>` is the root; without it the runtime renders nothing.
- `<helmet>` children go to `<head>`: `<style>`, and a Google Fonts `<link>` if needed.
- The server adds `data-dc-src="<File>.dc.html:<line>"` to elements when it serves a board. Don't write it yourself.
- Design-system boards also load `/ds/<ns>/styles.css` and `/ds/<ns>/_ds_bundle.js` in `<head>`; `add_board` adds them.

## data-props

JSON in single quotes on the `text/x-dc` script; write a `'` inside it as `&#39;`. Each key is a prop: `{"editor": "text", "default": …}`. Its `default` is the prop's value unless a `<dc-import>` passes one. Keys starting with `$` are settings, not props (`$preview: {width, height}`). Invalid JSON is an error.

## Holes: `{{path}}`

- A dotted lookup in one scope: props, then `this.state`, then `renderVals()` merged over them, so on a clash `renderVals()` wins, then state, then props: `{{title}}`, `{{user.name}}`, `{{items.0}}`.
- Literals work too: numbers, `"quoted"` strings, `true`, `false`, `null`.
- No expressions or calls (`{{a + b}}`, `{{fn()}}`): compute the value in `renderVals()` and look it up.
- In an attribute, a lone `{{x}}` keeps its type (function, object, boolean); mixed with text it becomes a string. `style="color: {{c}}"` works.

## Loops and conditions

```html
<sc-for list="{{items}}" as="item" index="i">
  <li>{{i}}: {{item.label}}</li>
</sc-for>
<sc-if value="{{open}}">…</sc-if>
<sc-if test="count > 2">…</sc-if>
<sc-if test="!done">…</sc-if>
```

- `sc-for`: `list="{{xs}}"` or `each="xs"`; `as` defaults to `item`; `index` is optional.
- `sc-if`: `value="{{cond}}"` (truthy), or `test` with a path, `!path`, or `a OP b` where OP is `===`, `!==`, `==`, `!=`, `>`, `<`, `>=`, `<=`. `==` is strict.

## Logic: `DCLogic`

One classic script (no ES `import`), defining `class Component extends DCLogic`.

- `this.props`, `this.state`, `this.setState(patch | fn)` (or `this.set(patch)`), re-renders.
- `renderVals()` returns extra values for the holes, including handler functions.
- `componentDidMount()` and `componentWillUnmount()` run if defined.

```js
class Component extends DCLogic {
  constructor(props) { super(props); this.state = { open: false }; }
  renderVals() {
    return { toggle: () => this.setState({ open: !this.state.open }) };
  }
}
```

Events:

- `onclick="{{toggle}}"`: a function from `renderVals()`. Inline JS strings are ignored.
- `data-on="click:toggle,input:onType"`: calls methods on the component.
- `data-goto="Checkout.dc.html"`, or `<a href="Checkout.dc.html">`: go to another board (prototype flow).
- Events: click, dblclick, change, input, submit, keydown, keyup, keypress, focus, blur, mouseenter, mouseleave, mousedown, mouseup, mouseover, mouseout, pointerdown, pointerup, scroll, wheel, close.
- A static `value`/`checked` on a form field without a handler becomes its default, so the field stays editable.

## Reuse: `<dc-import>` and `<x-import>`

- `<dc-import name="Header" active-tab="{{tab}}"></dc-import>` renders another board as a component. `name` is the board's file name without `.dc.html`, from this page, the page's `parts/`, or another page. Attributes become camelCase props; children are passed as `children`. A name with no board is an error.
- `<x-import component-from-global-scope="NS.Component" variant="primary">` renders a design-system React component. Use only namespaces the design's CLAUDE.md names; others are warned about.
- `hint-*` attributes on either are ignored by the runtime.

## Assets and styles

- Images, fonts, video, PDF: `upload_asset` to the page, then reference `../assets/<name>`.
- Tokens arrive as CSS variables (`--color-*`, `--font-*`); use them with a fallback, `var(--color-text, #17171c)`.
- Design-system files live under `/ds/<ns>/…`.

## Forbidden

`validate` reports these as errors:

- Network URLs in `src`, `href`, `action`, `poster`, `data`, CSS `url()` or `@import`. The one exception is a Google Fonts stylesheet: `<link href="https://fonts.googleapis.com/css2?…">`.
- `<iframe>`.
- In board logic: ES `import`, `fetch`, `XMLHttpRequest`, `eval`, `new Function`, `.innerHTML =`, `.appendChild(`, and `window`/`document` key listeners.
- Invalid `data-props` JSON; a `<dc-import>` naming a missing board; a board name used on two pages.

And as warnings: no `<x-dc>` root, no `/runtime/dc-runtime.js`, a `{{…}}` that is not a plain path, and an `<x-import>` namespace no design system provides.
