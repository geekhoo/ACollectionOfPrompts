# Figma MCP Tools Guide (VS Code)

This document describes the Figma MCP tools exposed in this workspace (per the editor tool registry). It covers:
- What each tool does
- Parameters (shape + meaning)
- Expected responses (high-level)
- Constraints / gotchas
- Common use cases
- Tool composition patterns (how to chain tools)

> Note
> - This guide is based on the tool schemas exposed in the editor (namespaces, parameter fields, and their documented behavior).
> - Some tools require a valid `fileKey` and `nodeId` from a Figma URL. Without real IDs and access rights, actual calls will fail.
> - Node IDs may appear as `123:456` or `123-456` in URLs. These tools accept either, but internally treat them as the same identifier.

---

## General Concepts

### `fileKey`
A Figma file identifier taken from URLs like:
- Design file: `https://figma.com/design/<fileKey>/<fileName>?node-id=1-2`
- Branch URL: `https://figma.com/design/<fileKey>/branch/<branchKey>/<fileName>` (use `<branchKey>` as the `fileKey` for these tools)
- FigJam board: `https://figma.com/board/<fileKey>/<fileName>?node-id=1-2`

### `nodeId`
The ID of a node (frame, component, page, etc.) from the URL query `node-id=`.
- Accepts forms like `1:2` or `1-2`.
- Tools often require a *node*, not just a page—except where explicitly stated.

### Code Connect
“Code Connect” is Figma’s mapping between design components (nodes) and code components (e.g., React component source URLs). Some tools can leverage this mapping to produce better “design context” aligned to your codebase.

---

## Tool: `mcp_figma_whoami`

### What it does
Returns information about the currently authenticated Figma user for the MCP connection. Useful for diagnosing permission/auth problems.

### Parameters
None.

### Expected response
A small JSON-like structure describing the authenticated user (e.g., id/handle/email/team/org info depending on what the integration exposes).

### Constraints
- If authentication is not configured, this will fail.
- The returned fields can vary by integration and Figma permission scope.

### Use cases
- Verify which Figma account the MCP server is using.
- Debug “permission denied” errors when other tools fail.

### Composition
- Use before any other tool when calls fail unexpectedly, to confirm identity and access.

---

## Tool: `mcp_figma_get_metadata`

### What it does
Fetches *structural* metadata for a node or page in an XML-like format (overview only): node IDs, layer types, names, positions, and sizes.

### Parameters
- `nodeId` (string, required): Node/page id like `123:456` or `123-456`.
- `fileKey` (string, required): Figma file key (or branch key as noted above).
- `clientLanguages` (string, optional): Comma-separated languages (e.g., `typescript,html,css`). Used for logging.
- `clientFrameworks` (string, optional): Comma-separated frameworks (e.g., `react,nextjs`). Used for logging.

### Expected response
- XML document (or XML-as-string) describing the node tree at a high level.

### Constraints
- This is **not** a full design export; it intentionally omits many styling details.
- Best for discovery (finding node IDs to query with richer tools).

### Use cases
- Explore page structure and identify candidate node IDs.
- Quick inventory of frames/components and their sizes/positions.

### Composition
- `get_metadata` → pick a relevant nodeId → `get_design_context` for code + richer semantics.

---

## Tool: `mcp_figma_get_design_context`

### What it does
Generates a “design context” for a specific node. This is the primary tool for turning design into usable implementation context and/or UI code.

It may:
- Return code (HTML/CSS/React/etc. depending on server capabilities)
- Provide structured info about layout, styles, components
- Provide asset download URLs for referenced images/icons

### Parameters
- `nodeId` (string, required): Node id.
- `fileKey` (string, required): File key.
- `clientLanguages` (string, optional): Comma-separated languages. Used for logging.
- `clientFrameworks` (string, optional): Comma-separated frameworks. Used for logging.
- `forceCode` (boolean, optional): If true, attempt to return code even if large.
- `disableCodeConnect` (boolean, optional): If true, do not use Code Connect mappings.

### Expected response
- Typically an object containing:
  - `code` (string): Generated UI code (may be omitted if too large and `forceCode` is false).
  - `assets` (object/map): Download URLs keyed by asset name/id.
  - Possibly additional metadata/context fields.

### Constraints
- Output size limits may cause code to be omitted unless `forceCode` is enabled.
- Requires access to the file and node.
- Code quality varies by node complexity (auto-layout vs freeform positioning).

### Use cases
- Generate starter UI code from a frame/component.
- Extract design tokens (colors, spacing, typography) in context.
- Identify required assets (icons/images) and fetch their URLs.

### Composition
- `get_code_connect_map` (optional) → `get_design_context` to align generated output to existing components.
- `get_variable_defs` → `get_design_context` to reconcile variables/tokens with generated code.
- `get_screenshot` → manual verification of generated code rendering.

---

## Tool: `mcp_figma_get_code_connect_map`

### What it does
Returns a mapping between a design node and the corresponding code component(s) according to Figma Code Connect.

### Parameters
- `nodeId` (string, required): Node id.
- `fileKey` (string, required): File key.
- `codeConnectLabel` (string, optional): Selects which mapping to return when multiple languages/frameworks are configured.

### Expected response
A JSON map of the form:
```text
{
  "<nodeId>": {
    "codeConnectSrc": "<url-or-path>",
    "codeConnectName": "<componentName>"
  },
  ...
}
```

### Constraints
- Only works where Code Connect is configured in the Figma file.
- The mapping can be incomplete or outdated if not maintained.

### Use cases
- Locate the real implementation source for a component represented in design.
- Drive automated “design-to-code” workflows: pick node → find code → update props/variants.

### Composition
- `get_code_connect_map` → open referenced source in repo → use `get_design_context` to compare expected vs actual.

---

## Tool: `mcp_figma_get_variable_defs`

### What it does
Fetches variable definitions (design tokens) relevant to a node.

Variables can represent reusable values such as:
- Colors
- Typography values
- Spacing/sizes

### Parameters
- `nodeId` (string, required): Node id.
- `fileKey` (string, required): File key.
- `clientLanguages` (string, optional): Logging.
- `clientFrameworks` (string, optional): Logging.

### Expected response
A JSON object mapping variable names/keys to values, e.g.:
```text
{
  "color/bg/default": "#ffffff",
  "spacing/2": "8",
  "radius/s": "6"
}
```

### Constraints
- Values may be represented in Figma’s internal formats depending on variable type.
- You may need to interpret units (px) and modes (light/dark) externally if not included.

### Use cases
- Generate or update a design-token file for a codebase.
- Validate that a component uses tokenized values rather than hardcoded values.

### Composition
- `get_variable_defs` → generate CSS variables / JSON token file.
- `get_variable_defs` + `get_design_context` → replace literals in generated code with token references.

---

## Tool: `mcp_figma_get_screenshot`

### What it does
Generates a rendered screenshot image for a given node.

### Parameters
- `nodeId` (string, required): Node id.
- `fileKey` (string, required): File key.
- `clientLanguages` (string, optional): Logging.
- `clientFrameworks` (string, optional): Logging.

### Expected response
- Typically a URL to an image (PNG) or a small object containing the URL and metadata.

### Constraints
- Requires access to the node.
- Screenshot fidelity depends on Figma rendering and tool settings.

### Use cases
- Visual verification for design-to-code.
- Attachments for PRs/issues referencing a specific design node.

### Composition
- `get_screenshot` → compare against app screenshot / storybook snapshot.
- `get_screenshot` + `get_design_context` → ensure generated code matches design.

---

## Tool: `mcp_figma_get_figjam`

### What it does
Generates UI code for a FigJam node (FigJam only).

### Parameters
- `nodeId` (string, required): FigJam node id.
- `fileKey` (string, required): FigJam file key.
- `clientLanguages` (string, optional): Logging.
- `clientFrameworks` (string, optional): Logging.
- `includeImagesOfNodes` (boolean, optional, default: true): Whether to include images of nodes.

### Expected response
- A representation of the FigJam content suitable for implementation (often code + assets/URLs).

### Constraints
- Works only for **FigJam** files (URLs like `/board/`).
- Not for standard Figma design files.

### Use cases
- Turn FigJam flow diagrams into documentation/HTML.
- Extract workshop/brainstorm artifacts into a repo-friendly form.

### Composition
- `get_figjam` → convert into markdown documentation in the repo.

---

## Tool: `mcp_figma_create_design_system_rules`

### What it does
Generates “design system rules” for the repository based on a Figma node (often a design system page/frame). Intended to produce guidelines that can be applied to code generation or review.

### Parameters
- `nodeId` (string, optional): Node id to base the rules on.
- `clientLanguages` (string, optional): Comma-separated languages used by the client (logging).
- `clientFrameworks` (string, optional): Comma-separated frameworks used by the client (logging).

### Expected response
- A set of rules/guidelines (typically markdown or structured text) describing design system constraints, tokens, component usage, naming conventions, etc.

### Constraints
- Quality depends heavily on selecting the correct node (design system hub/page).
- If `nodeId` is omitted, behavior depends on the MCP server (may use current selection, may fail).

### Use cases
- Establish repo-specific UI implementation conventions from the design system.
- Provide guidance for automated code generation agents.

### Composition
- `create_design_system_rules` → store in repo (e.g., `design-system-rules.md`) → use as prompt context for codegen/review.

---

# Cross-Tool Workflows (Composition Patterns)

## 1) Discover → Extract → Generate
1. `mcp_figma_get_metadata` (page) to find relevant node IDs
2. `mcp_figma_get_variable_defs` (node) to gather tokens
3. `mcp_figma_get_design_context` (node) to generate code + asset links
4. `mcp_figma_get_screenshot` (node) to validate visually

## 2) Align Design With Existing Code (Code Connect)
1. `mcp_figma_get_code_connect_map` (node) to find mapped component
2. Inspect the mapped code component in the repo
3. `mcp_figma_get_design_context` with `disableCodeConnect=false` (default) to bias output toward mapped components

## 3) Build/Update Token System
1. `mcp_figma_get_variable_defs` across representative nodes
2. Consolidate into a token registry (CSS variables / JSON / TS)
3. Use `mcp_figma_create_design_system_rules` to codify usage rules

## 4) FigJam to Docs
1. `mcp_figma_get_figjam` with `includeImagesOfNodes=true`
2. Convert result into markdown docs under `/Specify` or `/Research`

---

# Practical Constraints & Tips

- **Permissions**: If anything fails, run `mcp_figma_whoami` and confirm access to the target file.
- **Correct IDs**: Ensure `fileKey` and `nodeId` come from the same URL.
- **Branch URLs**: For URLs containing `/branch/<branchKey>/`, use `branchKey` as `fileKey`.
- **Output size**: For large frames, prefer smaller sub-frames or enable `forceCode` when needed.
- **Token modes**: If your design uses modes (light/dark), you may need multiple queries or manual reconciliation.

---

# How to “Try” These Tools

To empirically validate behavior, provide any Figma URL(s) you can access, such as:
- A design file node URL (frame/component)
- A design system page URL (for rules)
- A FigJam board node URL (for FigJam tool)

With those, you can run a quick validation matrix:
- `whoami`
- `get_metadata` on the page
- `get_design_context` on a selected frame
- `get_variable_defs` on the same frame
- `get_screenshot` on the same frame
- `get_code_connect_map` on a component (if Code Connect is configured)
- `get_figjam` on a FigJam node
