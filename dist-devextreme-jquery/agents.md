# Agents guide for @DevExtreme\dist-devextreme-jquery

## Purpose
This folder exports everything a web-based coding agent needs to compose static HTML/CSS/JS pages using DevExtreme jQuery widgets without installing npm packages. The bundle includes the compiled CSS themes, fonts, icons, the `dx.all.js` bundle (plus localization and helper files), and per-component prompt templates under `prompts/`.

## Folder layout in brief
- `css/` holds `dx.common.css`, a theme grid (light, material, fluent, etc.), and `fonts/` + `icons/` subfolders that the styles depend on; always keep the CSS, fonts, and icons together so relative references remain valid.
- `js/` contains `dx.all.js` (debug version too), `dx.ai-integration.js`, ASP.NET helpers, localization scripts, vectormap data, and FileSaver helpers.
- `prompts/` contains one Markdown prompt per `dx*` widget; each prompt reminds agents how to include the shared assets and instantiate the widget as a jQuery plugin.

## Web-search agent best practices
1. **Stay focused on documentation** – Prefer queries like `site:js.devexpress.com devextreme dxDataGrid configuration` or `DevExtreme localization dx.all.js` to quickly find authoritative info. Avoid drifting into unrelated topics.
2. **Validate versions** – The bundle is generated from DevExtreme 25.2.0; include the version when you search (e.g., `DevExtreme 25.2 notes`).
3. **Cite relevant sections** – Record the URL and a short summary of any page you rely on so that downstream developers can trace guidance back to the official docs.
4. **Use search refinements** – Add keywords like `jQuery`, `dx.all.js`, `css themes`, and `examples` to narrow results.
5. **Note locale files** – If localization is in scope, look for `<locale>.js` in the `js/localization/` folder and match it with search results for the same locale.

## Syntax reference reminders
- **Include order**: `<link rel="stylesheet" href="css/dx.common.css">`, `<link rel="stylesheet" href="css/dx.light.css">`, `<script src="https://code.jquery.com/jquery-3.7.0.min.js"></script>`, `<script src="js/dx.all.js"></script>`.
- **Widget initialization**: `$('#target').dxWidgetName({ /* options */ });` (refer to prompts for each `dxWidgetName`).
- **Localization injection**: Insert `js/localization/<locale>.js` before `dx.all.js` if non-English text is required.
- **Theme swapping**: Replace `dx.light.css` with another theme (`dx.material.blue.light.css`, `dx.fluent.blue.dark.css`, etc.) and ensure the corresponding fonts/icons remain accessible.
- **Assets path assumption**: Snippets assume pages live beside `dist-devextreme-jquery`, so the references use `css/` and `js/`; adjust the `../` segments if the folder is relocated.
