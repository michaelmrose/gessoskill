# `gesso.components.alert`

> Legacy component: implemented as a single `gesso/components/alert.clj` file rather than the current `attr.clj` / `core.clj` directory structure.

## Purpose

Renders themed alert blocks for status messages, warnings, errors, validation feedback, and other server-returned notices.

Alerts are pure Hiccup components and work well as inline HTMX swap fragments.

## Public API

### `(alert opts & children)`

Renders the alert shell. Supports both short-form and canonical-form usage.

#### Short Form

Use the short form when the alert has a simple title and/or body.

```clojure
(alert {:title "Saved"
        :content "Your changes were saved."
        :variant :default})
```

In short form, `:title` is rendered with `alert-title` and `:content` is rendered with `alert-content`.

#### Canonical Form

Use the canonical form when the alert needs explicit structure or custom child markup.

```clojure
(alert {:variant :destructive}
  (alert-title {:text "Error"})
  (alert-content {}
    [:p "Something went wrong."]))
```

## Options

* `:variant` — Visual style. Supported values are `:default` and `:destructive`. Unknown variants fall back to default alert styling.
* `:title` — Alert heading for short-form usage.
* `:content` — Alert body for short-form usage.
* `:class` — Additional classes for the alert root.
* `:attrs` — Additional HTML attributes for the alert root.

The root element renders as a `div` with `role="alert"` and `data-alert`.

## Subcomponents

### `(alert-title opts & children)`

Renders the alert heading using the shared text system.

Short form:

```clojure
(alert-title {:text "Saved"})
```

Canonical form:

```clojure
(alert-title {}
  "Saved")
```

Options:

* `:text` — Heading text for short-form usage.
* `:class` — Additional classes.
* `:attrs` — Additional HTML attributes.

The title renders with `data-title`.

### `(alert-content opts & children)`

Renders the alert body using the shared text system.

Short form:

```clojure
(alert-content {:text "Your changes were saved."})
```

Canonical form:

```clojure
(alert-content {}
  [:p "Your changes were saved."])
```

Options:

* `:text` — Body text for short-form usage.
* `:class` — Additional classes.
* `:attrs` — Additional HTML attributes.

## Usage Examples

### Simple Status Message

```clojure
(alert {:title "Saved"
        :content "Your changes were saved successfully."
        :variant :default})
```

### Error Message

```clojure
(alert {:title "Something went wrong"
        :content "Please check the form and try again."
        :variant :destructive})
```

### Custom Content

```clojure
(alert {:variant :destructive}
  (alert-title {:text "Upload failed"})
  (alert-content {}
    [:p "The file could not be uploaded."]
    [:p "Try again with a smaller file."]))
```

### HTMX Fragment

```clojure
(alert {:variant :default
        :attrs {:id "save-status"}}
  (alert-content {}
    "Saved."))
```

## Notes

* Alerts are appropriate for server-rendered status or validation messages.
* HTMX attributes can be passed through `:attrs` when the alert itself is a swap target, trigger boundary, or fragment root.
* Use `:destructive` for errors or critical feedback.
