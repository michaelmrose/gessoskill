### `gesso.theme`

**Purpose:** Configures theme-based application state (color, density, typography, shape) for the UI. It emits both HTML attributes for CSS targeting and `<meta>` tags that the `gesso-theme.js` runtime uses to hydrate client-side theme behavior.

**Public API:**

* `(theme theme-or-config mode)`: The primary facade for Biff's `base-html`. Returns a map of `{:base/html-attrs ... :base/head ...}`.
* `theme-or-config`: Either a single string/keyword (treated as `:color-theme`) or a map of theme axes (e.g., `{:color-theme "vercel" :density :comfortable}`).
* `mode`: Optional mode keyword (e.g., `:light`, `:dark`, `:system`).


* `(html-theme-attrs theme-or-config mode)`: Generates the attribute map for the `<html>` element.
* `(theme-head theme-or-config mode)`: Generates the `<meta>` tag sequence for the document `<head>`.

**Key Rules:**

* **Axis-Based Configuration:** Supports four theme axes: `:color-theme`, `:density`, `:typography`, and `:shape`.
* **Space-Separated Multi-Values:** You can pass vectors to axes (e.g., `{:density [:compact :compactish]}`) which are rendered as space-separated attribute values (e.g., `data-density="compact compactish"`), allowing for flexible CSS rule matching via `[data-density~="compact"]`.
* **Biff Integration:** The `theme` function is designed to be passed directly into Biff's `base-html` map.
* **HTML/JS Sync:** The system creates a source-of-truth in the HTML tag (via `data-*` attributes) and the document head (via `<meta>` tags) to prevent theme-flicker and ensure the JS runtime is correctly initialized.
