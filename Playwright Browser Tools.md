# Playwright Browser Tools

## Navigation & Page Control
| Tool | Purpose | Usage |
|------|---------|-------|
| `mcp_playwright_browser_navigate` | Navigate to a URL | Opens a specific web page by URL |
| `mcp_playwright_browser_navigate_back` | Go back to previous page | Simulates clicking the browser's back button |
| `mcp_playwright_browser_close` | Close the page | Terminates the current browser page/session |
| `mcp_playwright_browser_tabs` | Manage browser tabs | List, create, close, or select tabs |
| `mcp_playwright_browser_resize` | Resize browser window | Change the viewport dimensions (width/height) |

## Interaction Tools
| Tool | Purpose | Usage |
|------|---------|-------|
| `mcp_playwright_browser_click` | Click on elements | Performs mouse clicks (left/right/middle, single/double) on page elements |
| `mcp_playwright_browser_type` | Type text | Enters text into editable fields (textboxes, inputs) |
| `mcp_playwright_browser_hover` | Hover over elements | Moves mouse over an element (triggers hover states) |
| `mcp_playwright_browser_drag` | Drag and drop | Moves elements from one location to another |
| `mcp_playwright_browser_press_key` | Press keyboard keys | Simulates keyboard input (Enter, Arrow keys, shortcuts) |
| `mcp_playwright_browser_select_option` | Select dropdown options | Chooses values from dropdown/select elements |
| `mcp_playwright_browser_fill_form` | Fill multiple form fields | Batch-fills form inputs (textboxes, checkboxes, radios, sliders) |
| `mcp_playwright_browser_file_upload` | Upload files | Attaches files to file input elements |
| `mcp_playwright_browser_handle_dialog` | Handle dialogs | Accept or dismiss browser dialogs (alerts, confirms, prompts) |

## Page Inspection & Capture
| Tool | Purpose | Usage |
|------|---------|-------|
| `mcp_playwright_browser_snapshot` | Capture accessibility snapshot | Gets a structured representation of the page (better than screenshot for automation) |
| `mcp_playwright_browser_take_screenshot` | Take screenshot | Captures a visual image of the page or specific element |
| `mcp_playwright_browser_evaluate` | Execute JavaScript | Runs custom JavaScript code on the page or element |

## Monitoring & Debugging
| Tool | Purpose | Usage |
|------|---------|-------|
| `mcp_playwright_browser_console_messages` | Get console logs | Returns all console messages (or only errors) |
| `mcp_playwright_browser_network_requests` | Get network activity | Lists all network requests made since page load |
| `mcp_playwright_browser_wait_for` | Wait for conditions | Pauses until text appears/disappears or time elapses |

## Setup
| Tool | Purpose | Usage |
|------|---------|-------|
| `mcp_playwright_browser_install` | Install browser | Installs the browser binary if not present |

---

## Typical Workflow
1. `browser_navigate` → open a page
2. `browser_snapshot` → understand page structure and get element refs
3. `browser_click` / `browser_type` / `browser_fill_form` → interact with elements
4. `browser_wait_for` → wait for results
5. `browser_take_screenshot` → capture evidence
6. `browser_close` → clean up

---

# Comprehensive Reference Guide

## Understanding Element References (ref)

In Playwright MCP tools, the `ref` parameter is crucial. It represents the **exact target element reference from the page snapshot**. This reference comes from the accessibility tree captured by `browser_snapshot`.

### How Element References Work
1. First, call `browser_snapshot` to get the accessibility tree
2. The snapshot returns elements with unique `ref` identifiers
3. Use these `ref` values in subsequent interaction tools
4. The `element` parameter is a human-readable description for permission/logging

**Best Practice:** Always take a fresh snapshot before interacting with elements, as the DOM may have changed.

---

## Detailed Tool Reference

### 1. browser_navigate

**Purpose:** Navigate to a specific URL in the browser.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | Yes | The URL to navigate to |

**Best Practices:**
- Always use complete URLs including protocol (`https://`)
- Wait for navigation to complete before interacting with elements
- For SPAs, consider using `browser_wait_for` after navigation to ensure content loads

**Expected Outcomes:**
- Browser navigates to the specified URL
- Page begins loading (may trigger network requests)
- Returns control when initial load completes

**When to Use:**
- Starting a new browsing session
- Moving to a different page
- Opening a specific application URL

**Example Instruction:**
```
Navigate to "https://example.com/login"
```

---

### 2. browser_navigate_back

**Purpose:** Navigate to the previous page in browser history (like clicking the Back button).

**Parameters:** None required

**Best Practices:**
- Ensure there is navigation history before calling
- Use after navigating away from a page you need to return to
- Combine with `browser_wait_for` if the previous page has dynamic content

**Expected Outcomes:**
- Browser navigates to the previous URL in history
- Page state may be restored from cache

**When to Use:**
- After following a link, returning to the original page
- Testing back-button behavior
- Multi-step workflows requiring backtracking

**Limitations:**
- Does nothing if no history exists
- May not restore form data or dynamic state

---

### 3. browser_click

**Purpose:** Perform click actions on web elements.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `element` | string | Yes | Human-readable description of the element |
| `ref` | string | Yes | Exact element reference from snapshot |
| `button` | string | No | `left` (default), `right`, or `middle` |
| `doubleClick` | boolean | No | Perform double-click instead of single |
| `modifiers` | array | No | Modifier keys: `Alt`, `Control`, `ControlOrMeta`, `Meta`, `Shift` |

**Best Practices:**
- Always get a fresh snapshot before clicking
- Use `element` description to clearly identify the target
- For dynamic elements, wait for them to be stable first
- Use `ControlOrMeta` for cross-platform compatibility (Ctrl on Windows/Linux, Cmd on Mac)

**Click Types:**
| Action | Configuration |
|--------|--------------|
| Single left click | Default behavior |
| Double click | `doubleClick: true` |
| Right click (context menu) | `button: "right"` |
| Middle click (new tab) | `button: "middle"` |
| Ctrl+Click (new tab) | `modifiers: ["Control"]` |
| Shift+Click (select range) | `modifiers: ["Shift"]` |

**Expected Outcomes:**
- Element receives click event
- May trigger navigation, form submission, or JavaScript handlers
- Modifiers affect the click behavior (e.g., Ctrl+click opens in new tab)

**When to Use:**
- Clicking buttons, links, menu items
- Opening dropdown menus
- Selecting options
- Triggering JavaScript actions

**Common Pitfalls:**
- Element not visible/clickable (use `browser_wait_for` first)
- Element covered by another element (scroll into view)
- Dynamic elements changing `ref` values

---

### 4. browser_type

**Purpose:** Type text into editable elements (input fields, textareas, contenteditable).

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `element` | string | Yes | Human-readable description |
| `ref` | string | Yes | Element reference from snapshot |
| `text` | string | Yes | Text to type |
| `slowly` | boolean | No | Type character by character (triggers key handlers) |
| `submit` | boolean | No | Press Enter after typing |

**Best Practices:**
- Use `slowly: true` when the page has keystroke handlers (autocomplete, validation)
- Use `submit: true` for search fields or single-field forms
- Clear existing content first if needed (select all + type)
- For sensitive fields, consider if content should be logged

**fill() vs type() Behavior:**
| Method | Behavior | Use Case |
|--------|----------|----------|
| `fill` (default) | Instant, replaces content | Fast form filling |
| `slowly: true` | Character by character | Autocomplete, live validation |

**Expected Outcomes:**
- Text appears in the target field
- May trigger input/change events
- With `submit: true`, form may be submitted

**When to Use:**
- Filling login forms
- Search queries
- Text input fields
- Content entry

---

### 5. browser_fill_form

**Purpose:** Fill multiple form fields in a single operation.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `fields` | array | Yes | Array of field objects |

**Field Object Structure:**
| Property | Type | Description |
|----------|------|-------------|
| `name` | string | Human-readable field name |
| `type` | string | `textbox`, `checkbox`, `radio`, `combobox`, `slider` |
| `ref` | string | Element reference |
| `value` | string | Value to set (`true`/`false` for checkbox) |

**Best Practices:**
- Group related fields together
- Handle different field types appropriately
- For checkboxes: use `"true"` or `"false"` as string values
- For combobox: use the visible option text, not the value attribute
- For sliders: provide numeric value as string

**Expected Outcomes:**
- All specified fields are filled/checked/selected
- Form state updates
- Validation may trigger

**When to Use:**
- Registration forms
- Multi-field data entry
- Settings pages with multiple options

---

### 6. browser_hover

**Purpose:** Move mouse over an element (trigger hover state).

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `element` | string | Yes | Human-readable description |
| `ref` | string | Yes | Element reference |

**Best Practices:**
- Use before accessing dropdown menus that appear on hover
- Allow time for hover effects/animations to complete
- May need to hover twice for reliable drag-and-drop setup

**Expected Outcomes:**
- Element receives mouseenter/mouseover events
- CSS `:hover` styles apply
- Tooltip or dropdown may appear

**When to Use:**
- Revealing hidden menus
- Triggering tooltips
- Preview functionality
- Before drag-and-drop operations

---

### 7. browser_drag

**Purpose:** Perform drag and drop between two elements.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `startElement` | string | Yes | Description of source element |
| `startRef` | string | Yes | Source element reference |
| `endElement` | string | Yes | Description of target element |
| `endRef` | string | Yes | Target element reference |

**Best Practices:**
- Hover over both elements first for reliability
- For complex drag operations, hover twice on the target
- Wait for drag preview to appear before releasing
- Some applications may require mouse simulation instead

**Sequence for Reliable Drag-and-Drop:**
1. Hover over drag element
2. Mouse down
3. Hover over drop target
4. Hover over drop target again (ensures reliability)
5. Mouse up

**Expected Outcomes:**
- Element is moved from source to target location
- Drop event fires on target
- UI updates to reflect new position

**When to Use:**
- Kanban board interactions
- File organization
- Sortable lists
- Component positioning

---

### 8. browser_press_key

**Purpose:** Press keyboard keys or key combinations.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `key` | string | Yes | Key name or combination |

**Common Key Names:**
| Category | Keys |
|----------|------|
| Navigation | `ArrowUp`, `ArrowDown`, `ArrowLeft`, `ArrowRight`, `Home`, `End`, `PageUp`, `PageDown` |
| Actions | `Enter`, `Tab`, `Escape`, `Backspace`, `Delete`, `Space` |
| Modifiers | `Shift`, `Control`, `Alt`, `Meta` |
| Function | `F1` through `F12` |
| Characters | `a` through `z`, `A` through `Z`, `0` through `9` |

**Key Combinations:**
```
Ctrl+C (Copy)      → "Control+c"
Ctrl+V (Paste)     → "Control+v"
Ctrl+A (Select All) → "Control+a" or "ControlOrMeta+a"
Shift+Tab          → "Shift+Tab"
```

**Best Practices:**
- Use `ControlOrMeta` for cross-platform shortcuts
- Focus the target element before pressing keys
- For text selection, use Shift with arrow keys
- Use `pressSequentially` for typing with key handlers

**Expected Outcomes:**
- Key event fires on focused element
- Shortcut actions execute
- Text manipulation occurs

**When to Use:**
- Keyboard shortcuts
- Form navigation (Tab)
- Dismissing dialogs (Escape)
- Submitting forms (Enter)
- Text editing operations

---

### 9. browser_select_option

**Purpose:** Select options in dropdown/select elements.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `element` | string | Yes | Description of dropdown |
| `ref` | string | Yes | Select element reference |
| `values` | array | Yes | Array of values to select |

**Selection Methods:**
- By visible text: `["Option Text"]`
- By value attribute: `["option_value"]`
- Multiple selections: `["Option 1", "Option 2"]`

**Best Practices:**
- Use visible text for reliability
- For multi-select, pass array with multiple values
- Open the dropdown first if it's a custom component (not native `<select>`)
- Verify selection with snapshot after

**Expected Outcomes:**
- Option becomes selected
- Change event fires
- Dropdown closes (for single select)

**When to Use:**
- Native HTML select elements
- Country/state selectors
- Category filters
- Any dropdown selection

---

### 10. browser_file_upload

**Purpose:** Upload files to file input elements.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `paths` | array | No | Absolute paths to files. Omit to cancel file chooser |

**Best Practices:**
- Use absolute file paths
- Ensure files exist before upload
- For multiple files, include all paths in the array
- To cancel a file chooser dialog, call with empty `paths` or omit it

**Expected Outcomes:**
- Files are attached to the input
- File input displays selected file names
- Form ready for submission

**When to Use:**
- Profile picture uploads
- Document attachments
- Multi-file uploads
- Testing file validation

**Handling Non-Input Uploads:**
For custom upload components (not `<input type="file">`), the file chooser event must be handled:
1. Click the upload button
2. Handle the file chooser dialog
3. Set files using `setFiles`

---

### 11. browser_handle_dialog

**Purpose:** Handle JavaScript alert, confirm, and prompt dialogs.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `accept` | boolean | Yes | `true` to accept (OK), `false` to dismiss (Cancel) |
| `promptText` | string | No | Text to enter for prompt dialogs |

**Dialog Types:**
| Type | Accept | Dismiss |
|------|--------|---------|
| `alert()` | Closes dialog | Closes dialog |
| `confirm()` | Returns `true` | Returns `false` |
| `prompt()` | Returns entered text | Returns `null` |

**Best Practices:**
- Register dialog handler before triggering the dialog
- Include `promptText` for prompt dialogs even when accepting
- By default, Playwright auto-dismisses dialogs

**Expected Outcomes:**
- Dialog is closed
- JavaScript execution continues
- Return value passed to calling code

**When to Use:**
- Confirming delete actions
- Accepting terms
- Entering values in prompt dialogs
- Dismissing alerts

---

### 12. browser_snapshot

**Purpose:** Capture the accessibility tree of the current page.

**Parameters:** None required

**What It Returns:**
- YAML representation of the page's accessibility tree
- Element roles, names, and attributes
- Unique `ref` identifiers for each element
- Hierarchical structure of the page

**Best Practices:**
- Take a snapshot before any interaction
- Refresh snapshot after page changes (navigation, AJAX)
- Use for understanding page structure
- Better than screenshots for automation (structured data)

**ARIA Snapshot Structure:**
```yaml
- role: button
  name: "Submit"
  ref: "button-123"
- role: textbox
  name: "Username"
  ref: "input-456"
```

**Expected Outcomes:**
- Structured representation of visible/accessible elements
- Reference IDs for use in other tools
- Accessibility information for testing

**When to Use:**
- Before clicking, typing, or any interaction
- After page navigation
- After dynamic content loads
- Debugging element targeting issues

**Advantages Over Screenshots:**
| Aspect | Snapshot | Screenshot |
|--------|----------|------------|
| Data type | Structured | Visual |
| LLM processing | Direct | Requires vision model |
| Element targeting | Precise refs | Coordinate guessing |
| Speed | Faster | Slower |
| Accessibility testing | Built-in | Not available |

---

### 13. browser_take_screenshot

**Purpose:** Capture a visual image of the page or element.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `filename` | string | No | Output file name (defaults to `page-{timestamp}.png`) |
| `fullPage` | boolean | No | Capture entire scrollable page |
| `element` | string | No | Description of element to screenshot |
| `ref` | string | No | Element reference (required if `element` provided) |
| `type` | string | No | `png` (default) or `jpeg` |

**Screenshot Types:**
| Type | Configuration |
|------|--------------|
| Viewport only | Default |
| Full page | `fullPage: true` |
| Specific element | Provide `element` and `ref` |

**Best Practices:**
- Use meaningful filenames for organization
- Full page screenshots can be large
- Element screenshots are precise but require ref
- Set viewport size for consistent screenshots
- Use `jpeg` for smaller file sizes when quality isn't critical

**Resolution Control:**
Resolution is determined by viewport size. For high-resolution:
```javascript
// Set larger viewport before screenshot
viewport: { width: 1920, height: 1080 }
```

**Expected Outcomes:**
- Image file saved to specified path
- Visual record of page state
- Can be used for visual regression testing

**When to Use:**
- Visual documentation
- Error evidence capture
- Visual regression testing
- Debugging (when snapshot isn't enough)

---

### 14. browser_evaluate

**Purpose:** Execute JavaScript code in the page context.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `function` | string | Yes | JavaScript function to execute |
| `element` | string | No | Element description |
| `ref` | string | No | Element reference for element-scoped execution |

**Function Formats:**
```javascript
// Page context (no element)
"() => { return document.title; }"

// Element context (with ref)
"(element) => { return element.innerText; }"
```

**Best Practices:**
- Use for operations not possible with other tools
- Return serializable values (JSON-compatible)
- Handle errors gracefully within the function
- Avoid changing page state unless necessary

**Common Use Cases:**
```javascript
// Get page title
"() => document.title"

// Get element bounding box
"(el) => el.getBoundingClientRect()"

// Check element existence
"() => !!document.querySelector('.target')"

// Scroll element into view
"(el) => el.scrollIntoView()"

// Get computed style
"(el) => getComputedStyle(el).display"
```

**Expected Outcomes:**
- JavaScript executes in browser context
- Return value passed back to caller
- DOM manipulation takes effect

**When to Use:**
- Custom DOM queries
- Getting element properties
- Triggering custom JavaScript
- Scrolling operations
- Complex validation logic

---

### 15. browser_console_messages

**Purpose:** Retrieve console log messages from the page.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `onlyErrors` | boolean | No | Filter to only show errors |

**Message Types Captured:**
- `log` - Standard console.log
- `info` - Informational messages
- `warn` - Warnings
- `error` - Errors
- `debug` - Debug messages

**Best Practices:**
- Check for errors after page interactions
- Use for debugging JavaScript issues
- Monitor for unexpected warnings
- Clear understanding requires checking after each action

**Expected Outcomes:**
- Array of console messages with type and text
- Timestamps and source locations
- Arguments passed to console methods

**When to Use:**
- Debugging JavaScript errors
- Verifying expected logging
- Monitoring for warnings
- Capturing error messages for reporting

---

### 16. browser_network_requests

**Purpose:** Get all network requests made since page load.

**Parameters:** None required

**Information Captured:**
- Request URL
- HTTP method
- Status code
- Response type
- Timing information

**Best Practices:**
- Call after expected network activity completes
- Use for API debugging
- Monitor for failed requests
- Verify expected calls are made

**Expected Outcomes:**
- List of all network requests
- Request/response details
- Status codes and timing

**When to Use:**
- Debugging API calls
- Verifying data fetching
- Monitoring failed requests
- Performance analysis
- Testing network behavior

---

### 17. browser_wait_for

**Purpose:** Wait for specific conditions before proceeding.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `text` | string | No | Text to wait for to appear |
| `textGone` | string | No | Text to wait for to disappear |
| `time` | number | No | Time to wait in seconds |

**Wait Types:**
| Type | Use Case |
|------|----------|
| Text appears | Wait for content to load |
| Text disappears | Wait for loading indicator to go away |
| Time delay | Fixed pause for animations |

**Best Practices:**
- Prefer waiting for specific conditions over fixed time
- Use `textGone` for loaders/spinners
- Combine with snapshot to verify page state
- Set reasonable timeouts

**Expected Outcomes:**
- Execution pauses until condition met
- Continues when text appears/disappears
- Times out if condition not met

**When to Use:**
- After triggering async operations
- Waiting for AJAX content
- After form submissions
- Before interacting with dynamic content

---

### 18. browser_tabs

**Purpose:** Manage browser tabs (list, create, close, select).

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `action` | string | Yes | `list`, `new`, `close`, `select` |
| `index` | number | No | Tab index for `close` or `select` |

**Actions:**
| Action | Description |
|--------|-------------|
| `list` | Get all open tabs |
| `new` | Create a new empty tab |
| `close` | Close tab at index (current if omitted) |
| `select` | Switch to tab at index |

**Best Practices:**
- List tabs to understand current state
- Close tabs when done to manage resources
- Select tabs before interacting with their content
- Each tab is isolated (separate session)

**Expected Outcomes:**
- Tab list returned (for `list`)
- New tab opened (for `new`)
- Tab closed (for `close`)
- Tab becomes active (for `select`)

**When to Use:**
- Multi-tab workflows
- Testing popup behavior
- Managing parallel sessions
- Cleanup after tests

---

### 19. browser_resize

**Purpose:** Change the browser viewport dimensions.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `width` | number | Yes | Viewport width in pixels |
| `height` | number | Yes | Viewport height in pixels |

**Common Viewport Sizes:**
| Device | Width | Height |
|--------|-------|--------|
| Mobile (small) | 320 | 568 |
| Mobile (medium) | 375 | 667 |
| Tablet | 768 | 1024 |
| Desktop | 1280 | 720 |
| Desktop (large) | 1920 | 1080 |

**Best Practices:**
- Set viewport before taking screenshots
- Test responsive breakpoints
- Match target device dimensions
- Consider device pixel ratio for retina

**Expected Outcomes:**
- Viewport dimensions change
- Page may reflow/respond
- Media queries may trigger

**When to Use:**
- Responsive design testing
- Mobile emulation
- Screenshot standardization
- Testing breakpoint behavior

---

### 20. browser_close

**Purpose:** Close the current browser page and clean up resources.

**Parameters:** None required

**Best Practices:**
- Always close when done to free resources
- Close contexts before browser for graceful cleanup
- Don't close prematurely if more actions needed
- In testing, typically close at suite end, not each test

**Resource Management:**
```
Hierarchy:
Browser
  └── Context (session/profile)
       └── Page (tab)
```

**Expected Outcomes:**
- Page/tab closes
- Resources freed
- Session data cleared (if context closes)

**When to Use:**
- End of automation session
- Cleanup after tests
- Before starting fresh session
- Resource management in long runs

---

### 21. browser_install

**Purpose:** Install the browser binary if not present.

**Parameters:** None required

**Supported Browsers:**
- Chromium
- Firefox
- WebKit (Safari engine)

**Best Practices:**
- Run once during setup
- Required before first use
- Downloads appropriate binary for OS
- May take time on first run

**Expected Outcomes:**
- Browser binary downloaded
- Ready for automation
- Stored in cache location

**When to Use:**
- First-time setup
- After clearing browser cache
- When browser_navigate fails with "browser not installed"
- CI/CD pipeline setup

---

## Tool Combinations & Workflows

### Login Flow
```
1. browser_navigate → login page
2. browser_snapshot → get form elements
3. browser_fill_form → username, password
4. browser_click → submit button
5. browser_wait_for → dashboard text
6. browser_snapshot → verify logged in state
```

### Form Submission with Validation
```
1. browser_navigate → form page
2. browser_snapshot → get form structure
3. browser_type → fill first field
4. browser_wait_for → validation message (if async)
5. browser_fill_form → remaining fields
6. browser_click → submit
7. browser_wait_for → success message
8. browser_take_screenshot → evidence
```

### Data Extraction
```
1. browser_navigate → target page
2. browser_wait_for → content loaded
3. browser_snapshot → get page structure
4. browser_evaluate → extract specific data
5. (repeat for pagination)
```

### Multi-Tab Workflow
```
1. browser_navigate → main page
2. browser_click → link (opens new tab)
3. browser_tabs (list) → find new tab
4. browser_tabs (select) → switch to new tab
5. browser_snapshot → interact with new tab
6. browser_tabs (close) → close when done
7. browser_tabs (select) → return to main tab
```

### Error Investigation
```
1. browser_navigate → problematic page
2. browser_console_messages → check for JS errors
3. browser_network_requests → check for failed requests
4. browser_snapshot → examine page state
5. browser_take_screenshot → visual evidence
```

---

## Common Pitfalls & Solutions

| Problem | Cause | Solution |
|---------|-------|----------|
| Element not found | Stale ref | Take fresh snapshot before interaction |
| Click does nothing | Element not ready | Use `browser_wait_for` first |
| Type doesn't trigger events | Using fill mode | Set `slowly: true` |
| Dialog blocks execution | Unhandled dialog | Register `browser_handle_dialog` handler |
| Screenshot blank/partial | Page still loading | Wait for content before screenshot |
| Dropdown not working | Custom component | Click to open first, then select |
| Drag-drop fails | Complex implementation | Hover twice on target |
| Memory issues | Pages not closed | Close pages/contexts regularly |

---

## Best Practices Summary

1. **Always snapshot first** - Get fresh element refs before interacting
2. **Wait for conditions** - Use `browser_wait_for` over fixed delays
3. **Descriptive element names** - Help with debugging and logging
4. **Close resources** - Prevent memory leaks
5. **Handle dialogs** - Don't let alerts block execution
6. **Use appropriate click modifiers** - Match intended user behavior
7. **Test responsively** - Resize for different viewports
8. **Monitor console/network** - Catch errors early
9. **Prefer snapshots over screenshots** - Better for automation
10. **Chain actions logically** - Navigate → Snapshot → Interact → Verify
