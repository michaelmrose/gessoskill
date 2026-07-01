### `gesso.htmx`

**Purpose:** Low-level HTMX attribute builders and Hiccup out-of-band (OOB) fragment constructors. (Note: These are usually consumed via the `gesso.core` facade).

**Public API & Data Shapes:**

* `(oob-attrs {:id ... :swap ... :selector ... :attrs ...})`: Generates HTMX OOB attribute maps (e.g., `{:id "foo" :hx-swap-oob "outerHTML"}`).
* `(with-oob node swap-style)` / `(with-oob target swap-style node)`: Merges OOB attributes into an existing Hiccup node.
* `(oob-wrapper tag target swap-style & children)`: Creates a custom wrapper (like `:tbody` or `:ul`) configured for an OOB swap.
* **OOB Constructors** (all return `[:div {:id target :hx-swap-oob "..."} children]` unless using `with-oob` / `oob-wrapper`):
* `(oob-inner-html target & children)`
* `(oob-outer-html node)` / `(oob-outer-html target node)` / `(oob-replace ...)`
* `(oob-beforeend target & children)`
* `(oob-afterbegin target & children)`
* `(oob-beforebegin target & children)`
* `(oob-afterend target & children)`
* `(oob-delete target)`: Returns an empty `:div` with `:hx-swap-oob "delete"`.



**Key Rules:**

* **Keyword Swaps:** Accepts idiomatic Clojure keywords for swaps (e.g., `:outer-html`, `:before-end`) and automatically translates them to HTMX standard strings (`"outerHTML"`, `"beforeend"`).
* **ID Normalization:** Functions automatically strip leading `#` symbols when resolving target IDs (e.g., passing `"#app-toaster"` safely normalizes to `"app-toaster"`).
* **Default Wrapper:** The explicit structural helpers (`oob-inner-html`, `oob-beforeend`, etc.) wrap content in a `:div`. If you are swapping table rows or list items where a `:div` is invalid HTML, you must use `(oob-wrapper :tbody ...)` or `(oob-wrapper :ul ...)` to avoid browser DOM-parsing errors.
