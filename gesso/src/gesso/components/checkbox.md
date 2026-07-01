# gesso.components.checkbox

> **Note on Architecture:** This is a **legacy component**. It uses the single-file architecture (`gesso/components/checkbox.clj`) and does not follow the current multi-file directory convention.

**Purpose:** Renders a standard HTML checkbox input with Gesso styling.

## Public API

### (checkbox opts)

- Short Form: (checkbox {:id "done" :name "done" :checked true})
- Long Form: (checkbox {:attrs {:id "done" :name "done"}})

**Options:**
- :id: DOM ID for label association.
- :name: Form field name.
- :value: Input value attribute.
- :checked: Boolean indicating initial state.
- :disabled?: Boolean for disabled state.
- :required?: Boolean for form validation.
- :attrs: Map of additional HTML attributes (e.g., hx-post, hx-target).

## Usage Examples

### Simple Toggle
    (checkbox {:id "accept-terms" :name "terms" :checked true})

### HTMX Integration
    (checkbox {:id "toggle-feature"
               :attrs {:hx-post "/toggle"
                       :hx-trigger "change"}})

## Key Rules
- **Styling:** Applies checkbox and input CSS classes.
- **Attributes:** Injects a data-checkbox="true" attribute for component targeting.
- **HTMX Integration:** The component is designed for interactive form interactions (toggles, preference saving). Because it uses standard HTML attributes, it integrates seamlessly with HTMX's hx-post or hx-trigger="change" patterns.
