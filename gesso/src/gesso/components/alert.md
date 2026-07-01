# `gesso.components.alert`

> **Note on Architecture:** This is a **legacy component**. It uses the single-file architecture (`gesso/components/alert.clj`) and does not follow the current multi-file directory convention (`attr.clj`, `core.clj`, etc.).

**Purpose:** Renders system-level alert/notification blocks. Useful for status messages, warnings, or errors.

## Public API

### `(alert opts & children)`
The main alert shell.

- **Short Form:** `(alert {:title "..." :content "..." :variant :default})`
- **Long Form:** `(alert {:variant :destructive} (alert-title ...) (alert-content ...))`

**Options:**
- `:variant`: Choose between `:default` (standard alert) or `:destructive` (error/critical alert).
- `:title` / `:content`: When using the short form, these render the subcomponents automatically.

### Subcomponents
- `(alert-title {:text "..."})`: Renders the alert heading using the standard heading system.
- `(alert-content {:text "..."})`: Renders the alert body text using the standard body typography.

## Usage Examples

### Short Form (Recommended for simple messages)
```clojure
(alert {:title "Saved"
        :content "Your changes were saved successfully."
        :variant :default})
