### `gesso.http`

**Purpose:** Provides minimalist Ring response helpers specifically tailored for server-rendered HTML fragments and HTMX interactions.

**Public API:**

* `(html-response hiccup-node)`: Takes a Hiccup vector or Rum node, renders it to a static HTML string, and returns a standard Ring response map with `Content-Type: text/html; charset=utf-8`.
* `(no-content)`: Returns an empty HTTP 204 response. Useful for fire-and-forget HTMX operations (e.g., deletes or state changes that don't require a DOM swap).

**Key Rules:**

* Use `html-response` for all standard HTMX endpoints where you are sending HTML fragments back to the client.
* `no-content` is the standard way to acknowledge a request without causing the browser to render new content or perform a DOM swap.
* These helpers abstract the Ring-level response structure (`:status`, `:headers`, `:body`) from your component-level rendering logic.
