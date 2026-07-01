### `gesso.core`

**Purpose:** The central facade namespace re-exporting all Gesso UI components, layout shells, form controls, validation utilities, and HTMX OOB helpers into a single requireable API.

**Public API & Data Shapes:**
*(Note: Most UI components accept either a short-form map `(component {:text "..." :variant :primary})` or a long-form with children `(component {:variant :primary} "Child")`.)*

- **Page & Layout:** `(page)`, `(page-left)`, `(page-main)`, `(page-right)`, `(page-surface)`, `(background)`, `(scroll-buffer)`
- **Structural Blocks:** `(section-block ...)`, `(toolbar ...)`, `(group ...)`
- **Typography:** `(text)`, `(heading)`, `(page-title)`, `(section-title)`, `(muted-text)`, `(label-text)`
- **Interactive Components:** `(button)`, `(card)`, `(dialog)`, `(dropdown-menu)`, `(tabs)`, `(accordion)`, `(alert)`, `(badge)`, `(status-pill)`, `(empty-state)`, `(icon)`
- **Forms & Inputs:** `(form)`, `(post-form)`, `(post-button)`, `(field)`, `(label)`, `(input)`, `(textarea)`, `(select)`, `(checkbox)`, `(switch)`, `(radio-group)`, `(radio)`
- **Validation (HTMX):** `(field-plan)`, `(render-oob-errors explain-data)`, `(render-oob-error-map error-map)`
- **Security:** `(anti-forgery-token ctx)`, `(anti-forgery-input ctx)`
- **Toasts:** `(toaster)`, `(toast)`, `(render-toast-oob)`
- **HTTP Responses:** `(html-response hiccup)`, `(no-content)`
- **HTMX OOB Swap Helpers:** - `(with-oob node swap-style)`: Attaches `:hx-swap-oob` to an existing Hiccup node.
  - `(oob-inner-html target & children)`
  - `(oob-outer-html target node)` / `(oob-replace target node)`
  - `(oob-beforeend target & children)`
  - `(oob-delete target)`

**Key Rules:**
- **Strict Facade:** Require only this namespace (e.g., `[gesso.core :as g]`) in application view code. Never require `gesso.components.*` or `gesso.validation.*` directly.
- **Attribute Passing:** Standard HTML attributes and HTMX attributes (`:hx-get`, `:hx-target`, etc.) should generally be passed via the `:attrs` key inside a component's options map (e.g., `(g/button {:attrs {:hx-get "/foo"}})`).
- **Pure Data:** All components evaluate to pure Hiccup vectors. They must be wrapped in `(g/html-response ...)` when returning them from a standard Ring/Biff HTTP handler.
