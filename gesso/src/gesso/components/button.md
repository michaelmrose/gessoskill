# gesso.components.button

> **Note on Architecture:** This is a **legacy component**. It uses the single-file architecture (`gesso/components/button.clj`) and does not follow the current multi-file directory convention.

**Purpose:** Renders an interactive button element.

## Public API

### `(button opts & children)`

- **Short Form:** `(button {:variant :primary :size :md :text "Save"})`
- **Long Form:** `(button {:variant :primary :size :md} "Save")`

**Options:**
- `:variant`: Controls visual style. Supported variants:
  - `:default` (default), `:primary`, `:secondary`, `:outline`, `:ghost`, `:link`, `:destructive`
- `:size`: Controls button dimensions. Supported sizes:
  - `:md` (default), `:sm`, `:lg`, `:icon`, `:sm-icon`, `:lg-icon`
- `:attrs`: Map of additional HTML attributes (e.g., `hx-post`, `hx-target`).

## Usage Examples

### Short Form
```clojure
(button {:variant :primary :size :sm :text "Submit"})
```

### Long Form (with Children)
```clojure
(button {:variant :destructive}
  (icon "trash")
  [:span "Delete"])
```

## Key Rules
- **Defaults:** Defaults to `type="button"` unless overridden in `:attrs`.
- **Class Management:** Uses an internal `button-classes` map to resolve style and size combinations. The `el` macro and `class-names` utility ensure correct CSS class application.
- **HTMX Integration:** Designed for HTMX action buttons. Pass `hx-*` attributes through the `:attrs` key. Because it defaults to `type="button"`, it is safe to use within forms without accidentally triggering native form submission.
