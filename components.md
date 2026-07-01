# Gesso Component Architecture

This document defines the standard structure and API conventions for Gesso UI components. New components should follow these patterns so component code stays consistent, composable, and easy to promote or reuse across projects.

## 1. Directory Structure

### Standard Structure

Most new components should live in a dedicated directory:

```text
gesso/
  components/
    component_name/
      attr.clj            ; Attribute builders, data attrs, and class logic
      core.clj            ; Public rendering API and Hiccup structure
      scripts.clj         ; Optional client-side interaction logic
      component_name.css  ; Optional scoped component CSS
```

Use this structure for components with more than trivial rendering logic, custom attributes, client behavior, or scoped styles.

### Flat Structure

Small or older components may exist as a single file:

```text
gesso/
  components/
    component_name.clj
```

This form is acceptable for simple components, but larger flat components should be migrated to the standard directory structure when they receive significant updates.

## 2. API Convention: Short Form and Canonical Form

Gesso components may expose two related APIs: a convenience form for common usage and a canonical form for explicit composition.

### Short Form

The short form accepts a single options map.

```clojure
(card {:title "User Profile"
       :content "..."})
```

Use it for simple, common cases where the component can render its own internal sections from data. The short form should generally be a thin wrapper around the canonical form.

### Canonical Form

The canonical form accepts an options map followed by explicit children.

```clojure
(card {:variant :primary}
  (card-header {:title "User Profile"})
  (card-content
    [:div "Custom content..."])
  (card-footer
    [:button "Save"]))
```

Use it when callers need to control layout, inject custom markup, compose interactive elements, or render named subsections directly. More complex components should be designed around this form first, with the short form layered on top.

## 3. File Responsibilities

### `core.clj`

`core.clj` defines the public component API.

It should:

* Define the main rendering functions.
* Provide short-form and canonical-form entry points when both are useful.
* Compose named subsection functions such as `card-header`, `card-content`, or `card-footer`.
* Use `attr.clj` for attribute maps, class construction, and data attrs.
* Attach scripts from `scripts.clj` only when the component needs client behavior.
* Return pure Hiccup vectors.

Rendering functions should stay focused on structure and composition. Attribute construction, class decisions, and selector details should usually live in `attr.clj`.

### `attr.clj`

`attr.clj` owns HTML attributes and styling decisions.

It should:

* Build attribute maps for component elements.
* Own component-specific `data-*` attributes used by CSS, HTMX, hyperscript, or JavaScript.
* Centralize class construction using Gesso helpers such as `class-names` and `merge-attrs`.
* Prefer Gesso semantic classes and theme variables over one-off styling.
* Keep logic pure and deterministic.

This file is the source of truth for the component’s DOM hooks.

### `scripts.clj`

`scripts.clj` exists only when the component has packaged client-side behavior.

It should:

* Define hyperscript or JavaScript behavior for the component.
* Prefer the `gesso.hyperscript` DSL when hyperscript is sufficient.
* Target elements through stable `data-*` attributes defined in `attr.clj`.
* Avoid coupling behavior to incidental class names or fragile DOM structure.

Do not create or require `scripts.clj` for components that do not need client behavior.

### `component_name.css`

Component CSS should be scoped to the component’s data attributes.

It should:

* Be discovered by the theme build process.
* Use Gesso theme variables such as color, spacing, radius, typography, density, and shadow tokens.
* Prefer semantic utilities and design tokens before hardcoded values.
* Avoid global selectors unless the component explicitly owns that global behavior.

## 4. General Rules

New components should:

* Prefer the standard directory structure when the component has meaningful internals.
* Keep rendering pure.
* Keep side effects out of visual components.
* Use stable `data-*` attributes for CSS and behavior hooks.
* Use Gesso semantic classes and theme tokens first.
* Expose a simple short form where it improves common usage.
* Expose a canonical form when composition matters.
* Avoid guessing at downstream app behavior; accept data, options, or injected functions where integration is required.

A component may be more than a visual widget, but reusable app-specific machinery should still keep its integration points explicit. For example, an auth-flow component may render forms and provide flow helpers, but app-specific persistence, session creation, redirects, or external providers should be passed in as options or functions rather than hardcoded.
