### `gesso.build.themes`

**Purpose:** A build-time CSS compiler for Gesso that bundles preset themes, shared utility CSS, and component-specific CSS into a single `themes.css` file. It automatically rewrites CSS selectors to map theme axes to data attributes on the `html` root.

**API & Build Configuration:**

* `(build! opts)`: The core entry point for the build task.
* `:input-dir`: Directory containing theme axes (default: `"resources/themes"`).
* `:output-file`: Destination path for the CSS bundle (default: `"resources/public/gesso/themes.css"`).
* `:component-css-dirs`: List of paths to search for component-specific CSS files (default: `["src/gesso/components"]`).


* `(-main & _args)`: CLI entry point for Babashka tasks.

**Key Mechanisms:**

* **Selector Rewriting:** - Automatically rewrites `:root` selectors in preset files to target data attributes (e.g., `:root` -> `html[data-color-theme="vercel"]`).
* Automatically rewrites `.dark` selectors to handle scoped dark mode (e.g., `.dark` -> `html.dark[data-color-theme="vercel"]`).


* **Theme Axes:** Supports four axes: `:color-theme`, `:density`, `:typography`, and `:shape`. Each requires a corresponding directory in the input path.
* **Bundling Order:**
1. Preset CSS files (rewritten for theme scoping).
2. Utility CSS (from `gesso/theme-utilities.css` on classpath).
3. Component-specific CSS files (discovered automatically).


* **Component CSS:** Expects component CSS files to be nested in directories named after the component (e.g., `src/gesso/components/button/button.css`).

**Key Rules:**

* The build task is designed to be run via Babashka.
* Color themes are strictly required to have both `:root` and `.dark` blocks defined in their CSS.
* Component CSS files are discovered silently; if a component directory exists but lacks a `.css` file matching its name, it is skipped.
