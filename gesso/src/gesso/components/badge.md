# `gesso.components.badge`

> **Note on Architecture:** This is a **legacy component**. It uses the single-file architecture (`gesso/components/badge.clj`) and does not follow the current multi-file directory convention.

**Purpose:** Renders a compact visual indicator (badge) for status, categories, or counts.

## Public API

### `(badge opts & children)`

- **Short Form:** `(badge {:text "Waiting" :variant :secondary})`
- **Long Form:** `(badge {:variant :outline} "Claimed")`

**Options:**
- `:variant`: Controls visual style. Supported variants:
  - `:default`
  - `:primary`
  - `:secondary`
  - `:destructive`
  - `:outline`

## Usage Examples

### Short Form
```clojure
(badge {:text "New" :variant :primary})
