### `gesso.util`

**Purpose:** Internal utility namespace providing the foundational mechanics for Gesso component construction, Hiccup attribute management, and form value normalization.

**Public API & Data Shapes:**

**1. Component API Construction (The Gesso Signature)**
These helpers enable the standardized Gesso "Short Form" (map-only) vs "Long Form" (children-only) component API.

* `(split-opts opts)`: Splits a map into `{:props, :class, :attrs}`. Essential for component implementations.
* `(normalize-component-args args)`: Returns `[opts children]` from a variadic argument list. Detects if the first argument is an options map.

**2. Hiccup & Attribute Helpers**

* `(el tag base-attrs attrs children)`: The primary low-level constructor. It merges `base-attrs` (component defaults) with `attrs` (user-supplied), flattens/normalizes children, and emits the Hiccup vector.
* `(merge-attrs & maps)`: Merges attribute maps, specifically concatenating `:class` values rather than overwriting them.
* `(class-names & xs)`: Concatenates and deduplicates CSS classes. Handles strings, keywords, nils, and sequences.
* `(nodes x)`: Normalizes any input into a flat sequence of Hiccup nodes.
* `(normalize-children children)`: Flattens child sequences one level while preserving atomic Hiccup elements.

**3. Normalization & Coercion**

* `(ensure-vec x)`: Normalizes `x` into a vector.
* `(->keyword x)`: Converts strings/keywords to keywords.
* `(slugify x)`: Converts strings/keywords to kebab-case slugs (e.g., used for generating stable DOM IDs).
* `(->value v fallback)`: Coerces values to strings.

**4. Form Control Helpers**

* `(request-param params k)`: Safe lookup in Ring-style parameter maps (checks both keyword and string keys).
* `(parse-int-value x)`: Parses integers from submitted form values. Returns original value on failure (so callers can identify invalid input).
* `(checked-value? value expected)`: Compares values as strings for radio/checkbox/select state.
* `(select-option current value label)`: Renders an `<option>` tag with `selected` logic.

**Key Rules:**

* **The Gesso Standard:** New components should use `split-opts` and `normalize-component-args` to support both short and long forms.
* **Class Safety:** Always use `merge-attrs` when merging attributes, especially if the component provides default classes (e.g., `(merge-attrs {:class "default"} user-attrs)`).
* **DOM Stability:** Use `slugify` when deriving stable DOM IDs from component titles or labels.
