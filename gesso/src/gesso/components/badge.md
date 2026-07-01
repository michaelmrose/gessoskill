# `gesso.components.badge`

> Legacy component: implemented as a single `gesso/components/badge.clj` file rather than the current `attr.clj` / `core.clj` directory structure.

## Purpose

Renders a compact themed badge for statuses, labels, categories, counts, or other short metadata.

## Public API

### `(badge opts & children)`

Renders a `span` badge. Supports both short-form and canonical-form usage.

#### Short Form

Use the short form when the badge content can be passed as text.

```clojure
(badge {:text "Waiting"
        :variant :secondary})
```

#### Canonical Form

Use the canonical form when the badge content should be provided explicitly.

```clojure
(badge {:variant :outline}
  "Claimed")
```

## Options

* `:text` — Badge text for short-form usage.
* `:variant` — Visual style. Supported values are `:default`, `:primary`, `:secondary`, `:destructive`, and `:outline`. Unknown variants fall back to default badge styling.
* `:class` — Additional classes for the badge.
* `:attrs` — Additional HTML attributes for the badge.

## Usage Examples

### Status Badge

```clojure
(badge {:text "Waiting"
        :variant :secondary})
```

### Count Badge

```clojure
(badge {:variant :primary}
  "3")
```

### Outline Badge

```clojure
(badge {:text "Claimed"
        :variant :outline})
```

### Error Badge

```clojure
(badge {:text "Failed"
        :variant :destructive})
```

## Notes

* Badges are intended for short inline content.
* Use `:attrs` for additional HTML attributes such as `:id`, `:title`, `:aria-label`, or `data-*` hooks.
* Use `:class` to add layout or spacing classes around the badge’s built-in variant styling.
