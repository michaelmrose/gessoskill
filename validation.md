# Gesso Validation System

This module handles the orchestration between Malli schemas (for server-side/backend truth) and HTML5 validation + Hyperscript (for client-side/UX truth), as well as server-side HTMX error rendering.

## 1. Orchestration (`gesso.validation.core`)
This is the primary namespace used by form field components to wire up validation state.

### Public API
- `(field-plan schema field-key err-id)`:
  - **Purpose:** Generates the complete "validation plan" for a specific form field.
  - **Inputs:** - `schema`: A Malli map schema.
    - `field-key`: The keyword of the field to validate.
    - `err-id`: The DOM ID of the corresponding error container.
  - **Returns:** `{:attrs ... :script ... :constraints ...}`.
    - `:attrs`: HTML5 validation attributes (e.g., `required`, `pattern`). Merge these into your control.
    - `:script`: A Hyperscript string. Append this to the control's `:_` attribute.
- `(empty-field-plan)`: Returns an empty map; useful when no schema is provided.

---

## 2. HTMX OOB Error Rendering (`gesso.validation.htmx`)
Use these utilities to render validation errors from the server back to the client using HTMX Out-of-Band (OOB) swaps.

### Public API
- `(render-oob-errors explain-data)`:
  - Takes Malli `explain-data` (from `malli.core/explain-data`).
  - Automatically maps errors to DOM nodes via `path->err-id`.
  - Emits HTMX OOB fragments that inject error messages into the DOM.
- `(render-oob-error-map error-map)`:
  - Takes a map of field paths to error strings (e.g., `{:email "Invalid email"}`).
  - Emits HTMX OOB fragments.
- `(path->field-id path)` / `(path->err-id path)`:
  - **Crucial:** Used to ensure the naming convention matches what the client expects (e.g., `[:user :email]` -> `user-email-error`).

---

## 3. Translation Engine (`gesso.validation.malli`)
The internal engine that inspects Malli schemas to derive browser-compatible constraints.

### Public API
- `(extract-constraints schema field-kw)`:
  - Extracts HTML5 rules (`:minlength`, `:pattern`, etc.) and error messages from a schema.
  - **Policy:** `:gesso.html/pattern` or `:html/pattern` in the schema metadata takes precedence over the Malli `:re` regex. This allows keeping backend regex logic complex while keeping client-side pattern simple.

---

## 4. Client-Side Scripting (`gesso.validation.scripts`)
The engine for the Hyperscript "gatekeeper."

### Public API
- `(gatekeeper-script messages err-id)`:
  - Generates the Hyperscript responsible for:
    1. Clearing visual error states on valid input.
    2. Preventing form submission (`htmx:validation:validate`) if the field is invalid.
    3. Mapping native HTML5 `ValidityState` (e.g., `valueMissing`, `tooShort`) to the specific error messages extracted by `gesso.validation.malli`.

---

## Usage Pattern

1. **Setup:** Pass your Malli schema and field key to `(field-plan ...)` inside your form component.
2. **Merge:** Merge the resulting `:attrs` into your `<input>`/`<textarea>`. Append the `:script` to the control's `:_` attribute.
3. **Container:** Ensure you have a stable DOM container for errors (usually via `path->err-id`).
4. **Server Response:** When the server rejects a POST, call `(render-oob-errors explain-data)` in your component Hiccup to automatically update the error container via HTMX OOB.
