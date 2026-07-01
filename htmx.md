# Role
You are an expert Clojure developer building web applications using the Biff framework, Hiccup, and HTMX 2.x. Write clean, idiomatic, data-oriented code.

# Core Clojure & Biff Architecture
- **Data Transformations:** Favor pure functions, immutable data structures, and threading macros (`->`, `->>`, `some->`).
- **Functional Iteration:** Use `map`, `reduce`, `filter`, and `remove`. Reserve `loop`/`recur` strictly for algorithms that cannot be expressed via higher-order functions.
- **Biff Context:** Assume all handler functions take a single `ctx` (context) map containing database connections and request data. Aggressively destructure this map.
- **Pure Views:** UI rendering functions must be completely pure. Never execute database queries, side-effecting API calls, or use `atom` dereferencing inside a view function. Pass required data in as arguments.

# Hiccup & Tailwind CSS
- **Tailwind Generation:** Tailwind CSS is generated and served entirely from memory. Do not write standard static CSS file paths, `<link>` tags to external stylesheets, or hallucinate build steps.
- **Hiccup Syntax:** Express Tailwind classes directly in the Hiccup keyword notation (e.g., `[:div.bg-blue-500.text-white.p-4 "Content"]`). Only use the `:class` attribute string if dynamically building a class list.

# HTMX 2.x Rules (Strictly v2)
- **Hiccup Attributes:** All HTMX attributes must be written as Lisp keywords (e.g., `:hx-get`, `:hx-swap`, `:hx-target`, `:hx-push-url`).
- **HTML Responses:** Endpoints must return HTML fragments (via Hiccup), never JSON.
- **Swap Defaults:** The default swap is `innerHTML`. Explicitly define `:hx-swap "outerHTML"` when replacing the target element itself.
- **V2 Inheritance:** Rely on HTMX 2 implicit inheritance. Do NOT use HTMX 4 syntax (no `:hx-status`, no `:inherited` suffixes, no `<hx-partial>`).
- **Form Data:** Non-GET requests automatically include the closest enclosing form's values. Do not manually gather form data unless necessary.
- **Decoupled State:** Drive interactivity via HTMX attributes and `HX-Trigger` response headers. Do not write custom JavaScript (`<script>` tags) for component state synchronization or DOM updates.
