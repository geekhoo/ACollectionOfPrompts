# ASP.NET Core Razor Pages CSS Isolation with DevExtreme Widgets

## Overview & goal

This note explains how to write **CSS‑isolated rules for Razor Pages** (`Page.cshtml.css`) that can still style **JS‑appended DevExtreme elements** (for example `.dx-menu-item` created after widget initialization) **without breaking Razor CSS isolation**. It aligns with the project convention of wrapping widgets in a root container, such as:

```html
<div class="mf-component-top-menu" data-component="TopMenu">
    <div class="top-menu"><!-- DevExtreme menu target --></div>
</div>
```

Key constraint: the app uses Razor Pages CSS isolation, which rewrites selectors with a generated scope attribute (for example `b-3xxtam6d07`), bundles them into `{APP ASSEMBLY}.styles.css` (or `v2.styles.css`), and loads that in `Pages/Shared/_Layout.cshtml`.

The problem: DevExtreme injects descendants such as `.dx-menu`, `.dx-menu-item`, etc., **after** the Razor page renders. If the isolation engine tries to scope those elements directly (for example via `.dx-menu-item[b-xxxx]`), the CSS will not match because the JS‑created elements never get the scope attribute.

The solution: **apply the scope attribute to the Razor‑rendered container** and use `:deep` (`::deep` in docs, `:deep(...)` in selectors) so that DevExtreme‑appended descendants are styled through normal descendant selectors, while container isolation remains intact. `:global` exists as an escape hatch when you intentionally need an unscoped rule.

---

## Official Microsoft docs used

### Razor Pages CSS isolation (core behavior)

**Docs:**

- Razor Pages CSS isolation (concepts & bundling):  
  https://learn.microsoft.com/en-us/aspnet/core/razor-pages/?view=aspnetcore-10.0#css-isolation
- Razor Pages CSS isolation configuration:  
  https://learn.microsoft.com/en-us/aspnet/core/razor-pages/?view=aspnetcore-10.0#css-isolation-configuration

**Key points from the docs (summarized):**

- To isolate styles to a page/view, place CSS in a `Pages/Index.cshtml.css` (Razor Pages) or `Views/Index.cshtml.css` (MVC) file.
- CSS isolation runs at **build time**. The framework **rewrites CSS selectors** to match the markup rendered by the app’s pages or views.
- Rewritten CSS is **bundled as a static asset** named `{APP ASSEMBLY}.styles.css` (for example `WebApp.styles.css`).
- The layout (`Pages/Shared/_Layout.cshtml` or `Views/Shared/_Layout.cshtml`) includes:

  ```html
  <link rel="stylesheet" href="~/{APP ASSEMBLY}.styles.css" />
  ```

- Within the bundled CSS, each page/view is associated with a **scope identifier** of the form `b-{STRING}` (a 10‑character string). Example from the docs:

  ```css
  /* /Pages/Index.cshtml.rz.scp.css */
  h1[b-3xxtam6d07] {
      color: red;
  }
  ```

  and the corresponding HTML:

  ```html
  <h1 b-3xxtam6d07>Index</h1>
  ```

- The scope identifier is **unique to the app** and is appended as an HTML attribute to elements rendered by the matching page or view.
- CSS isolation applies only to **HTML elements** (not Tag Helpers themselves).
- Configuration options include:
  - `CssScope` metadata to customize scope identifiers or share scopes across multiple `.cshtml.css` files (for example, base/derived layouts).
  - `StaticWebAssetBasePath` to control where static web assets (including scoped CSS) are rooted.
  - `DisableScopedCssBundling` to opt out of automatic bundling if you use a custom pipeline.

### MVC Views CSS isolation (same engine, more examples)

**Docs:**

- MVC views CSS isolation overview:  
  https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview?view=aspnetcore-10.0#css-isolation
- MVC views CSS isolation configuration:  
  https://learn.microsoft.com/en-us/aspnet/core/mvc/views/overview?view=aspnetcore-10.0#css-isolation-configuration

The MVC article uses the same isolation engine. It repeats and reinforces:

- Scope identifiers use `b-{STRING}` and are injected into selectors as attributes (for example `a[b-3xxtam6d07]`).
- The bundled CSS file for the project references per‑page CSS via imports and uses the same `b-…` scoping mechanism.
- CSS preprocessors (Sass/Less) can be integrated as long as they compile **before** the framework rewrites selectors during build.

These Razor Pages/MVC docs confirm **how selectors are rewritten** but don’t describe `:deep` / `:global`. For those, the Blazor CSS isolation docs are used, which rely on the same selector‑rewriter.

### Blazor CSS isolation (`::deep` / `:deep` semantics)

**Docs:**

- Blazor CSS isolation overview:  
  https://learn.microsoft.com/en-us/aspnet/core/blazor/components/css-isolation?view=aspnetcore-10.0
- Child component support and `::deep`:  
  https://learn.microsoft.com/en-us/aspnet/core/blazor/components/css-isolation?view=aspnetcore-10.0#child-component-support

**Key points used from the Blazor docs (adapted to Razor Pages):**

- CSS isolation for components uses the **same scope identifier pattern** `b-{STRING}` and is bundled into `{PACKAGE ID/ASSEMBLY NAME}.styles.css`.
- The docs explicitly describe how **`::deep` affects where the scope attribute is applied**:

  > When you define a CSS rule in a scoped CSS file, the scope is applied to the rightmost element. For example: `div > a` is transformed to `div > a[b-{STRING}]`.
  >
  > If you instead want the rule to apply to a different selector, the `::deep` pseudo-element allows you to do so. For example, `div ::deep > a` is transformed to `div[b-{STRING}] > a`.

- `::deep` is defined as a pseudo‑element that means “apply this rule to **descendants** of the scoped element,” and it effectively moves the `b-…` attribute from the right‑most selector to the selector **to the left of `::deep`**.
- The docs also note that scoped CSS only applies to HTML elements, not components or Tag Helpers.

Although these docs are written for `.razor.css` files, the same isolation engine and rewriting rules are used for Razor Pages `.cshtml.css` and MVC Views `.cshtml.css`. The key behavior we rely on is:

- **Without `:deep`**: `div > a` → `div > a[b-3xxtam6d07]`
- **With `:deep`**: `div :deep(> a)` / `div ::deep > a` → `div[b-3xxtam6d07] > a`

We apply that pattern to `.top-menu` and `.mf-component-top-menu` in Razor Pages so the scope attribute lands on the **container**, not on DevExtreme’s `.dx-menu-item` elements.

### `:global` behavior

The official docs don’t explicitly document `:global` for Razor Pages/MVC, but the isolation pipeline accepts `:global(...)` similarly to other scoped‑CSS systems:

- A selector wrapped in `:global(...)` is **not rewritten** by the isolation engine.
- Using `:global` in a scoped CSS file effectively yields a **global rule** that bypasses attribute scoping.

For example, in a page `.cshtml.css` file:

```css
.top-menu :global(.dx-menu-item) {
    border-radius: 0;
}
```

ends up in the bundle as:

```css
.top-menu .dx-menu-item {
    border-radius: 0;
}
```

with **no** `[b-…]` attribute added anywhere in the selector. This is useful as an escape hatch but should be used sparingly because it violates isolation by design.

---

## How Razor CSS isolation rewrites selectors

### Basic rewriting rule

From the Razor Pages/MVC docs, a simple rule in `Pages/Index.cshtml.css`:

```css
h1 {
    color: red;
}
```

is rewritten into the scoped bundle as:

```css
/* /Pages/Index.cshtml.rz.scp.css */
h1[b-3xxtam6d07] {
    color: red;
}
```

and the markup rendered from `Index.cshtml` becomes:

```html
<h1 b-3xxtam6d07>Index</h1>
```

For complex selectors, the same rule applies: **the scope attribute is attached to the right‑most simple selector**.

Examples:

```css
/* In Index.cshtml.css */
div > a {
    text-decoration: none;
}

.top-menu .dx-menu-item {
    padding: 0 12px;
}
```

rewritten as:

```css
div > a[b-3xxtam6d07] {
    text-decoration: none;
}

.top-menu .dx-menu-item[b-3xxtam6d07] {
    padding: 0 12px;
}
```

This is **unsafe** for JS‑appended DevExtreme elements because they never get `b-3xxtam6d07`.

### Using `:deep` to move the scope to the container

By using `:deep`, we instruct the isolation engine to move the `b-…` attribute **left** from the right‑most selector to the selector immediately before `:deep`.

From the Blazor docs:

- `div > a` → `div > a[b-3xxtam6d07]`
- `div ::deep > a` → `div[b-3xxtam6d07] > a`

Applied to DevExtreme scenarios:

```css
/* In Pages/Shared/_TopMenu.cshtml.css */
.top-menu :deep(.dx-menu-item) {
    padding: 0 12px;
}

.mf-component-top-menu :deep(.dx-menu-item.dx-state-hover) {
    background-color: #0050b3;
    color: #fff;
}
```

are rewritten to something like:

```css
/* /Pages/Shared/_TopMenu.cshtml.rz.scp.css */
.top-menu[b-3xxtam6d07] .dx-menu-item {
    padding: 0 12px;
}

.mf-component-top-menu[b-3xxtam6d07] .dx-menu-item.dx-state-hover {
    background-color: #0050b3;
    color: #fff;
}
```

Now the **container** (`.top-menu` / `.mf-component-top-menu`) receives the scope attribute, while the DevExtreme‑appended `.dx-menu-item` remains unmodified. Because `.dx-menu-item` is a descendant of `.top-menu[b-…]`, the rule correctly applies.

### When and why to use `:global`

`:global` tells the isolation engine **not to scope** part of a selector. For example:

```css
.top-menu :global(.dx-menu-item) {
    border-radius: 0;
}
``;

is emitted into the bundle as:

```css
.top-menu .dx-menu-item {
    border-radius: 0;
}
```

This can be useful when:

- You’re overriding **global DevExtreme theme styles** and want the rule to apply across multiple pages/components using the same `.top-menu`.
- You have a selector that must participate in the **global cascade** without the extra `[b-…]` specificity.

Trade‑offs:

- You lose isolation guarantees: the rule may affect widgets in other pages if they share `.top-menu .dx-menu-item`.
- You must manage **specificity** carefully; you’re now competing directly with site‑wide CSS and DevExtreme theme CSS.

In most cases, `:deep` + container scoping is preferred; `:global` should be a last resort.

---

## Minimal patterns for JS‑appended DevExtreme descendants

### Recommended: container‑scoped rules using `:deep`

In `Pages/Shared/_TopMenu.cshtml.css` (or any page/partial using the top menu):

```css
/* Layout for Razor-rendered container, safely scoped via [b-…] */
.mf-component-top-menu {
    display: flex;
    align-items: stretch;
    height: 48px;
    background-color: #0f172a;
}

.top-menu {
    flex: 1 1 auto;
}

/* DevExtreme menu reset & layout */
.top-menu :deep(.dx-menu) {
    background: transparent;
    border: none;
    height: 100%;
}

.top-menu :deep(.dx-menu-horizontal) {
    height: 100%;
}

/* Individual menu items (works for dynamic items) */
.top-menu :deep(.dx-menu-item) {
    padding: 0 16px;
    height: 100%;
    display: flex;
    align-items: center;
    color: #e5e7eb;
}

/* Hover/active/selected states */
.mf-component-top-menu :deep(.dx-menu-item.dx-state-hover),
.mf-component-top-menu :deep(.dx-menu-item.dx-state-focused),
.mf-component-top-menu :deep(.dx-menu-item.dx-state-selected) {
    background-color: #1d4ed8;
    color: #ffffff;
}

/* Optional: icon & text alignment */
.mf-component-top-menu :deep(.dx-menu-item .dx-icon) {
    margin-right: 8px;
}

.mf-component-top-menu :deep(.dx-menu-item .dx-menu-item-text) {
    white-space: nowrap;
}
```

Expected compiled shape (conceptual):

```css
/* /Pages/Shared/_TopMenu.cshtml.rz.scp.css */
.mf-component-top-menu[b-3xxtam6d07] {
    display: flex;
    /* … */
}

.top-menu[b-3xxtam6d07] {
    flex: 1 1 auto;
}

.top-menu[b-3xxtam6d07] .dx-menu {
    background: transparent;
    border: none;
    height: 100%;
}

.top-menu[b-3xxtam6d07] .dx-menu-horizontal {
    height: 100%;
}

.top-menu[b-3xxtam6d07] .dx-menu-item {
    padding: 0 16px;
    height: 100%;
    display: flex;
    align-items: center;
    color: #e5e7eb;
}

.mf-component-top-menu[b-3xxtam6d07] .dx-menu-item.dx-state-hover,
.mf-component-top-menu[b-3xxtam6d07] .dx-menu-item.dx-state-focused,
.mf-component-top-menu[b-3xxtam6d07] .dx-menu-item.dx-state-selected {
    background-color: #1d4ed8;
    color: #ffffff;
}

.mf-component-top-menu[b-3xxtam6d07] .dx-menu-item .dx-icon {
    margin-right: 8px;
}

.mf-component-top-menu[b-3xxtam6d07] .dx-menu-item .dx-menu-item-text {
    white-space: nowrap;
}
```

Important characteristics:

- The `b-3xxtam6d07` attribute appears only on **containers the Razor page renders** (`.mf-component-top-menu`, `.top-menu`).
- DevExtreme’s appended descendants (`.dx-menu`, `.dx-menu-item`, `.dx-menu-item-text`) do **not** have the scope attribute but are targeted as descendants of the scoped containers.
- Isolation is preserved at the container level; no unintended global leakage.

### When a fully unscoped rule is necessary: `:global`

Use `:global` only when a rule truly must be global and you are comfortable breaking isolation.

Example:

```css
/* This rule is global despite being in a scoped .cshtml.css file */
.top-menu :global(.dx-menu-item) {
    border-radius: 0;
}
```

Compiled result:

```css
.top-menu .dx-menu-item {
    border-radius: 0;
}
```

Considerations:

- Safe if `.top-menu` is a uniquely used class or always wrapped in `.mf-component-top-menu` in your app.
- Risky if other parts of the app also use `.top-menu` with DevExtreme menus and **should not** share this styling.
- Prefer to increase specificity under the component root rather than using `:global` if possible.

---

## Drop-in `_TopMenu.cshtml.css` snippet

Assuming the markup:

```html
<div class="mf-component-top-menu" data-component="TopMenu">
    <div class="top-menu"></div>
</div>
```

`Pages/Shared/_TopMenu.cshtml.css`:

```css
/* Root component box; isolation remains on this container */
.mf-component-top-menu {
    display: flex;
    align-items: stretch;
    height: 48px;
    background-color: #0f172a;
}

.top-menu {
    flex: 1 1 auto;
}

/* DevExtreme menu base (respects container scoping) */
.top-menu :deep(.dx-menu) {
    background: transparent;
    border: none;
    height: 100%;
}

.top-menu :deep(.dx-menu-horizontal) {
    height: 100%;
}

/* Menu items (dynamic) */
.top-menu :deep(.dx-menu-item) {
    padding: 0 16px;
    height: 100%;
    display: flex;
    align-items: center;
    color: #e5e7eb;
}

.mf-component-top-menu :deep(.dx-menu-item.dx-state-hover),
.mf-component-top-menu :deep(.dx-menu-item.dx-state-focused),
.mf-component-top-menu :deep(.dx-menu-item.dx-state-selected) {
    background-color: #1d4ed8;
    color: #ffffff;
}

.mf-component-top-menu :deep(.dx-menu-item .dx-icon) {
    margin-right: 8px;
}

.mf-component-top-menu :deep(.dx-menu-item .dx-menu-item-text) {
    white-space: nowrap;
}
```

This file assumes `v2.styles.css` (or `{APP ASSEMBLY}.styles.css`) is already correctly linked in `_Layout.cshtml`, per the official docs; **no changes to asset ordering are required**.

---

## Verification steps

To confirm that isolation + DevExtreme styling works as intended:

1. **Build the project and inspect the bundle**  
   After building, open `wwwroot/v2.styles.css` (or `{APP ASSEMBLY}.styles.css`). Search for `_TopMenu.cshtml` or `.mf-component-top-menu`. You should see selectors shaped like:

   ```css
   .top-menu[b-XXXXXXXXXX] .dx-menu-item { … }
   .mf-component-top-menu[b-XXXXXXXXXX] .dx-menu-item.dx-state-hover { … }
   ```

   where `XXXXXXXXXX` corresponds to the generated scope identifier.

2. **Confirm attribute placement**  
   Ensure the scope attribute `b-XXXXXXXXXX` is attached to **containers** (`.mf-component-top-menu`, `.top-menu`) and **not** to `.dx-menu-item` or other DevExtreme descendants.

3. **Runtime DOM inspection**  
   Load a page using `_TopMenu`. In browser dev tools, inspect a dynamically created `.dx-menu-item`. It should **not** have a `b-…` attribute. Inspect ancestor elements up to `.top-menu` / `.mf-component-top-menu`; they should have `b-…` attributes matching what you saw in the CSS bundle.

4. **Visual behavior**  
   Hover and click on menu items. Confirm that hover/selected styles from `_TopMenu.cshtml.css` apply (background color, text color, padding) without using `!important`.

5. **Specificity sanity check**  
   In dev tools, under the “Styles” pane for a `.dx-menu-item`, verify that your container‑scoped rules (for example `.mf-component-top-menu[b-…] .dx-menu-item.dx-state-hover`) are not being overridden by the DevExtreme theme. If they are, increase specificity under the component root (for example `.mf-component-top-menu[data-component="TopMenu"] .dx-menu-item.dx-state-hover`) rather than adding `!important`.

---

## Fallback if `:deep` isn’t supported in the current SDK

If you find that `:deep` / `::deep` isn’t being rewritten as expected (for example, selectors are not moving the `[b-…]` attribute to the container), you can use a **narrowly scoped global stylesheet** instead of relying on the `:deep` behavior.

### 1. Keep `_TopMenu.cshtml.css` for container layout only

```css
/* Pages/Shared/_TopMenu.cshtml.css */
.mf-component-top-menu {
    display: flex;
    align-items: stretch;
    height: 48px;
    background-color: #0f172a;
}

.top-menu {
    flex: 1 1 auto;
}
```

This preserves Razor Pages isolation for the **container** only.

### 2. Create a global, component‑prefixed stylesheet

`wwwroot/css/components.top-menu.css`:

```css
/* Narrow global scope: only affect DevExtreme menus inside the component wrapper */
.mf-component-top-menu .dx-menu {
    background: transparent;
    border: none;
}

.mf-component-top-menu .dx-menu-horizontal {
    height: 100%;
}

.mf-component-top-menu .dx-menu-item {
    padding: 0 16px;
    height: 100%;
    display: flex;
    align-items: center;
    color: #e5e7eb;
}

.mf-component-top-menu .dx-menu-item.dx-state-hover,
.mf-component-top-menu .dx-menu-item.dx-state-focused,
.mf-component-top-menu .dx-menu-item.dx-state-selected {
    background-color: #1d4ed8;
    color: #ffffff;
}

.mf-component-top-menu .dx-menu-item .dx-icon {
    margin-right: 8px;
}

.mf-component-top-menu .dx-menu-item .dx-menu-item-text {
    white-space: nowrap;
}
```

Characteristics:

- This CSS is **global**, not isolated, but is safely constrained by the `.mf-component-top-menu` prefix.
- DevExtreme widgets elsewhere in the app are unaffected unless they’re wrapped in `.mf-component-top-menu`.

### 3. Load the global stylesheet in `_Layout.cshtml`

In `Pages/Shared/_Layout.cshtml`, alongside the existing `{APP ASSEMBLY}.styles.css` reference, add:

```cshtml
<link rel="stylesheet" href="~/css/components.top-menu.css" />
```

Do **not** reorder existing assets; just add this link after your global CSS but before per‑page scripts if possible.

### Specificity and `!important` guidance

- Prefer **component‑root prefixes** (`.mf-component-top-menu .dx-menu-item`) over `!important`.
- If DevExtreme theme styles still win, increase specificity in a controlled way:

  ```css
  .mf-component-top-menu[data-component="TopMenu"] .dx-menu-item.dx-state-hover {
      background-color: #1d4ed8;
  }
  ```

- Avoid ID selectors and `!important` unless there is no alternative; they make future overrides and refactors difficult.

---

## Summary

- Razor Pages/MVC CSS isolation rewrites `.cshtml.css` selectors by appending a `b-{STRING}` scope attribute to the **right‑most selector** and bundles them into `{APP ASSEMBLY}.styles.css`, linked from `_Layout.cshtml`.
- DevExtreme widgets add elements like `.dx-menu-item` **after** Razor renders the page and they do **not** get the scope attribute, so rules like `.dx-menu-item[b-…]` will never match.
- By using `:deep`/`::deep` so that the scope attribute lands on the Razor‑rendered container (for example `.top-menu[b-…] .dx-menu-item`), we maintain isolation while still styling JS‑appended DevExtreme descendants.
- `:global` provides an explicit escape hatch when you need a fully unscoped selector, but it should be used sparingly and always with a strong component‑root prefix.
- If `:deep` isn’t supported in your SDK, you can fall back to a **narrowly scoped global stylesheet** (for example `components.top-menu.css`) keyed off the component root (`.mf-component-top-menu`) and loaded from `_Layout.cshtml`.
