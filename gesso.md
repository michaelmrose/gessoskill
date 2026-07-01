# Gesso Working Context for Future Chats

This document is a pasteable context packet for future chats that need to help write Clojure web applications using Biff, Hiccup, HTMX 2.x, and Gesso/Gessokit. It is not formal public documentation. Its purpose is to give the assistant enough working memory to produce code that fits this project without needing the whole Gesso source dump.

## Core Stack and Style

The app stack is Clojure, Biff, Hiccup/Rum static rendering, HTMX 2.x, and Gesso UI components.

Use idiomatic, data-oriented Clojure:

- Prefer pure functions, immutable data, and simple transformations.
- Prefer `map`, `reduce`, `filter`, `remove`, `some->`, `->`, and `->>` when they make code clearer.
- Reserve `loop`/`recur` for algorithms that really need it.
- Handlers usually receive a single Biff/Ring `ctx` map.
- Destructure `ctx` aggressively when it improves clarity.
- Keep views pure. Do not query the database, call external APIs, deref atoms, mutate state, or perform side effects inside rendering functions.
- Shape data before rendering when that avoids complex conditionals in Hiccup.

When editing files for this user, prefer complete file contents over patches or tiny snippets unless asked otherwise.

## Application View Rules

In application view code, prefer requiring Gesso through the facade:

```clojure
[gesso.core :as g]
```

Use existing Gesso components and semantic classes first. Do not reinvent UI primitives that already exist in Gesso.

Most app views should use Gesso components such as `g/card`, `g/button`, `g/field`, `g/input`, `g/alert`, `g/badge`, `g/status-pill`, `g/dropdown-menu`, `g/dialog`, `g/tabs`, etc. Prefer `:attrs` for raw HTML, HTMX, ARIA, and `data-*` attributes.

HTMX endpoints should return HTML fragments, not JSON. Use `g/html-response` for normal Ring/Biff handlers returning Hiccup.

Use HTMX attributes as Clojure keywords:

```clojure
{:hx-get "/fragment"
 :hx-target "#target"
 :hx-swap "outerHTML"}
```

For Gesso components, pass HTMX attrs through `:attrs` unless the component explicitly exposes a more specific option:

```clojure
(g/button
 {:text "Save"
  :attrs {:hx-post "/save"
          :hx-target "#status"
          :hx-swap "outerHTML"}})
```

## Gesso Component API Model

Gesso components generally support two call shapes:

### Short Form

The short form accepts one options map and renders common content from data.

```clojure
(g/badge {:text "Waiting"
          :variant :secondary})
```

### Canonical Form

The canonical form accepts an options map followed by explicit children.

```clojure
(g/badge {:variant :outline}
  "Claimed")
```

More complex components may expose public subcomponents, such as alert title/content, card header/content/footer, dialog pieces, menu items, tab triggers/panels, etc. Do not invent subcomponent names; if the API is not known, ask for the component source.

Common options across many components:

- `:text` — simple text for short-form usage.
- `:title` — title text/content for short-form usage.
- `:content` — body content for short-form usage.
- `:variant` — visual style.
- `:size` — component size where supported.
- `:class` — extra classes merged with component defaults.
- `:attrs` — raw HTML/HTMX/ARIA/data attributes passed to the relevant element.

Do not assume every component supports every option. For exact APIs, use known source or ask for the component source.

## App-Local Gesso-Style Components

End users of Gesso are expected to write their own app-local Gesso-style components. Treat this as a first-class workflow, not an internal-only contributor workflow.

For a component with meaningful internals, prefer this directory shape:

```text
components/my_component/
  attr.clj            ; data attrs, class composition, low-level attr maps
  core.clj            ; public Hiccup rendering API
  scripts.clj         ; only when packaged client behavior is needed
  my_component.css    ; only when scoped CSS is needed
```

Flat single-file components are fine for tiny wrappers, but directory-shaped components are better once there are data hooks, scripts, CSS, or multiple public rendering functions.

### `core.clj`

`core.clj` owns the public rendering API and Hiccup structure.

It should:

- Return pure Hiccup vectors.
- Define public functions such as `input`, `panel`, `card`, `section`, etc.
- Support short-form and canonical-form APIs when both are useful.
- Compose named subcomponents for complex structures.
- Keep rendering focused on structure and composition.
- Avoid side effects.

### `attr.clj`

`attr.clj` owns attribute construction.

It should:

- Build stable `data-*` hooks.
- Build classes and merged attribute maps.
- Own ARIA attributes when they are structural.
- Own HTMX attrs when they are part of the component contract.
- Prefer Gesso semantic classes and theme tokens.
- Keep logic pure and deterministic.

Use stable `data-*` attributes for CSS and script hooks. Do not target fragile DOM shape or incidental styling classes.

### `scripts.clj`

`scripts.clj` exists only when the component has packaged client behavior.

It should:

- Package reusable behavior for the component.
- Prefer the `gesso.hyperscript` DSL when hyperscript is sufficient.
- Attach behavior through stable attrs from `attr.clj`.
- Avoid one-off app state hacks.
- Not exist when no script behavior is needed.

### Component CSS

Component CSS should be scoped to stable component data attrs and use Gesso tokens.

Prefer:

```css
[data-one-time-code-input] {
  color: var(--foreground);
  border-color: var(--input);
}
```

Avoid global selectors or fragile structure unless the component explicitly owns them.

## Gesso Authoring Helpers

`gesso.util` contains helpers used by Gesso-style components. Use these when writing app-local components rather than hand-rolling the same behavior.

Important helpers:

- `split-opts` — splits an options map into `{:props ... :class ... :attrs ...}`.
- `normalize-component-args` — supports variadic short/canonical component APIs.
- `el` — builds Hiccup while merging base attrs, caller attrs, and normalized children.
- `merge-attrs` — merges attr maps and concatenates classes instead of overwriting them.
- `class-names` — builds class strings from strings, keywords, nils, and sequences.
- `nodes` — normalizes text/Hiccup/content into child nodes.
- `normalize-children` — flattens child sequences while preserving Hiccup elements.
- `slugify` — useful for stable DOM ids.
- `request-param` — looks up params by keyword or string key.
- `checked-value?` — compares form values as strings for checkbox/radio/select state.

When source is available, follow the patterns already used by nearby Gesso components.

## Hyperscript DSL

`gesso.hyperscript` is used for reusable component behavior.

Main entry point:

```clojure
(hs [:on :click
     [:set :my.dataset.open "true"]])
```

`hs` compiles Clojure data structures into a hyperscript string.

Attach hyperscript to Hiccup through the `:_` attribute.

Use `merge-script-attr` when adding script behavior to attrs so existing behavior is preserved instead of overwritten.

Use stable `data-*` attributes from `attr.clj` as script targets. Avoid fragile selectors based on incidental class names or DOM position.

Use hyperscript for packaged component behavior. Prefer HTMX, OOB swaps, Gesso live helpers, and server-rendered fragments for application state changes.

Do not claim to have tested hyperscript behavior in a browser unless browser access was actually available in the current chat.

## Gesso Core Facade

`gesso.core` is the app-facing facade. Prefer it in app view code.

Important exported groups:

### Page and Layout

- `page`
- `page-left`
- `page-main`
- `page-right`
- `page-surface`
- `background`
- `scroll-buffer`

### Structural Blocks

- `section-block`
- `toolbar`
- `group`
- `card`

### Typography

- `text`
- `heading`
- `page-title`
- `section-title`
- `muted-text`
- `label-text`

### Interactive Components

- `button`
- `dialog`
- `dropdown-menu`
- `tabs`
- `accordion`
- `alert`
- `badge`
- `status-pill`
- `empty-state`
- `icon`

### Forms and Inputs

- `form`
- `post-form`
- `post-button`
- `field`
- `label`
- `input`
- `textarea`
- `select`
- `checkbox`
- `switch`
- `radio-group`
- `radio`

### Validation

- `field-plan`
- `render-oob-errors`
- `render-oob-error-map`

### Security

- `anti-forgery-token`
- `anti-forgery-input`

### Toasts

- `toaster`
- `toast`
- `render-toast-oob`

### HTTP

- `html-response`
- `no-content`

### HTMX OOB Helpers

- `with-oob`
- `oob-inner-html`
- `oob-outer-html`
- `oob-replace`
- `oob-beforeend`
- `oob-delete`

If a component API is unclear, ask for its source instead of guessing.

## Component Inventory and Basic APIs

This is a quick working index, not a full substitute for source. Use it to choose existing components before writing custom markup.

### `g/alert`

Purpose: themed status/warning/error/validation alert block.

Known API:

```clojure
(g/alert {:title "Saved"
          :content "Your changes were saved."
          :variant :default})

(g/alert {:variant :destructive}
  (g/alert-title {:text "Error"})
  (g/alert-content {:text "Something went wrong."}))
```

Known options:

- `:variant` — `:default` or `:destructive`; unknown variants fall back to default styling.
- `:title` — short-form title.
- `:content` — short-form body.
- `:class`
- `:attrs`

Known behavior:

- root renders with `role="alert"`.
- root supports `:attrs`, useful for HTMX swap targets/fragments.
- `alert-title` and `alert-content` support `:text`, `:class`, and `:attrs`.

### `g/badge`

Purpose: compact inline status/label/count/category metadata.

Known API:

```clojure
(g/badge {:text "Waiting"
          :variant :secondary})

(g/badge {:variant :outline}
  "Claimed")
```

Known options:

- `:text`
- `:variant` — `:default`, `:primary`, `:secondary`, `:destructive`, `:outline`; unknown variants fall back to default styling.
- `:class`
- `:attrs`

### `g/button`

Purpose: themed button for UI actions, form controls, and HTMX actions.

Known API pattern:

```clojure
(g/button {:text "Save"
           :variant :primary
           :size :md})

(g/button {:variant :destructive}
  (g/icon "trash")
  [:span "Delete"])
```

Known options from prior context:

- `:text`
- `:variant` — commonly `:default`, `:primary`, `:secondary`, `:outline`, `:ghost`, `:link`, `:destructive`
- `:size` — commonly `:md`, `:sm`, `:lg`, `:icon`, `:sm-icon`, `:lg-icon`
- `:class`
- `:attrs`

Important: Gesso buttons default to `type="button"` unless overridden. Inside forms, submit buttons must pass:

```clojure
:attrs {:type "submit"}
```

### `g/checkbox`

Purpose: themed standard checkbox input.

Known API pattern:

```clojure
(g/checkbox {:id "accept"
             :name "accept"
             :checked true})

(g/checkbox {:attrs {:id "toggle"
                     :name "toggle"
                     :hx-post "/toggle"
                     :hx-trigger "change"}})
```

Likely options from prior notes:

- `:id`
- `:name`
- `:value`
- `:checked`
- `:disabled?`
- `:required?`
- `:attrs`

Ask for source before relying on subtler behavior.

### Other Component Files Present

The Gesso component tree includes:

```text
accordion/attr.clj
accordion/core.clj
accordion/scripts.clj
alert.clj
background/core.clj
background/patterns.clj
badge.clj
bars/attr.clj
bars/core.clj
bars/scripts.clj
button.clj
card/attr.clj
card/core.clj
checkbox.clj
dialog/attr.clj
dialog/core.clj
dialog/scripts.clj
dropdown_menu/attr.clj
dropdown_menu/core.clj
dropdown_menu/scripts.clj
empty_state.clj
field/attr.clj
field/core.clj
field/scripts.clj
field.clj
form/attr.clj
form/core.clj
form/scripts.clj
group.clj
icon.clj
input.clj
label.clj
page.clj
radio_group.clj
scroll_buffer.clj
section_block.clj
select.clj
status_pill.clj
switch.clj
tabs/attr.clj
tabs/core.clj
tabs/scripts.clj
text.clj
textarea.clj
toaster/attr.clj
toaster/core.clj
toaster/scripts.clj
toolbar.clj
```

Use this list to avoid inventing components. If exact API matters, ask for the relevant source file.

## Theme and Semantic CSS

Gesso generated CSS uses theme axes on the `html` element:

- `data-color-theme`
- `data-density`
- `data-typography`
- `data-shape`
- dark mode via `html.dark`

Prefer semantic classes and CSS variables over one-off inline styling.

### Typography Classes

- `font-body`
- `font-heading`
- `font-mono`
- `text-xs-theme`
- `text-sm-theme`
- `text-base-theme`
- `text-md-theme`
- `text-lg-theme`
- `text-xl-theme`
- `text-2xl-theme`
- `text-3xl-theme`
- `text-4xl-theme`
- `text-5xl-theme`
- `leading-tight-theme`
- `leading-snug-theme`
- `leading-body`
- `leading-heading`
- `leading-loose-theme`
- `tracking-tight-theme`
- `tracking-body`
- `tracking-heading`
- `tracking-wide-theme`
- `weight-light-theme`
- `weight-normal-theme`
- `weight-medium-theme`
- `weight-semibold-theme`
- `weight-bold-theme`
- `weight-extrabold-theme`
- `italic-theme`

### Density and Control Classes

- `control-height-theme`
- `control-theme`
- `button-density`
- `interactive-row-height-theme`
- `interactive-row-theme`
- `px-row`
- `py-row`
- `pad-row`
- `px-control`
- `py-control`
- `pad-control`
- `px-button`
- `py-button`
- `pad-button`

### Layout and Gap Classes

- `stack-theme`
- `section-theme`
- `cluster-theme`
- `toolbar-theme`
- `form-theme`
- `list-theme`
- `panel-theme`
- `title-stack-theme`
- `content-stack-theme`
- `gap-stack`
- `gap-section`
- `gap-inline`
- `gap-cluster`
- `gap-field`
- `gap-form`
- `gap-list`
- `gap-panel`
- `gap-toolbar`
- `gap-title`
- `gap-content`

### Padding and Surface Classes

- `px-card`
- `py-card`
- `pad-card`
- `px-dialog`
- `py-dialog`
- `pad-dialog`
- `px-panel`
- `py-panel`
- `pad-panel`
- `px-container`
- `pad-container`

### Icon Classes

- `icon-xs-theme`
- `icon-sm-theme`
- `icon-md-theme`
- `icon-lg-theme`
- `icon-xl-theme`
- `icon-2xl-theme`

### Shape Classes

- `radius-sm`
- `radius-md`
- `radius-lg`
- `radius-xl`
- `border-theme`

### Useful Theme Tokens

Color/surface tokens:

- `--background`
- `--foreground`
- `--card`
- `--card-foreground`
- `--popover`
- `--popover-foreground`
- `--primary`
- `--primary-foreground`
- `--secondary`
- `--secondary-foreground`
- `--muted`
- `--muted-foreground`
- `--accent`
- `--accent-foreground`
- `--destructive`
- `--destructive-foreground`
- `--border`
- `--input`
- `--ring`

Typography tokens:

- `--font-body`
- `--font-heading`
- `--font-mono`
- `--text-xs` through `--text-5xl`
- `--leading-tight`
- `--leading-snug`
- `--leading-body`
- `--leading-loose`
- `--tracking-tight`
- `--tracking-normal`
- `--tracking-wide`
- `--weight-light` through `--weight-extrabold`

Spacing/control tokens:

- `--space-1` through `--space-9`
- `--control-height`
- `--control-px`
- `--control-py`
- `--button-px`
- `--button-py`
- `--interactive-row-height`
- `--interactive-row-px`
- `--interactive-row-py`

Layout tokens:

- `--panel-px`
- `--panel-py`
- `--field-gap`
- `--form-gap`
- `--stack-gap`
- `--section-gap`
- `--inline-gap`
- `--cluster-gap`
- `--toolbar-gap`
- `--list-gap`
- `--panel-gap`
- `--title-gap`
- `--content-gap`
- `--card-padding`
- `--dialog-padding`
- `--container-px`

Shape/icon tokens:

- `--radius-xs`
- `--radius-sm`
- `--radius-md`
- `--radius-lg`
- `--radius-xl`
- `--border-width`
- `--icon-size-xs` through `--icon-size-2xl`

If doing heavy visual/CSS work, it may be useful to paste the generated CSS too. For ordinary code work, this semantic cheat sheet should be enough.

## HTTP and HTMX Helpers

`gesso.http` provides:

- `html-response` — render Hiccup/Rum node to HTML response with `text/html; charset=utf-8`.
- `no-content` — return HTTP 204 response.

Use `html-response` for standard HTMX endpoints returning fragments.

Use `no-content` for actions that should not swap visible content.

`gesso.htmx` provides out-of-band swap helpers:

- `with-oob`
- `oob-attrs`
- `oob-wrapper`
- `oob-inner-html`
- `oob-outer-html`
- `oob-replace`
- `oob-beforeend`
- `oob-afterbegin`
- `oob-beforebegin`
- `oob-afterend`
- `oob-delete`

OOB helpers translate Clojure-ish swap keywords such as `:outer-html` or `:before-end` into HTMX strings such as `"outerHTML"` and `"beforeend"`.

Use `oob-wrapper` for invalid-HTML contexts such as tables or lists where a default `div` wrapper would be wrong.

## Validation

Gesso validation connects Malli schemas, HTML5 validation attrs, hyperscript behavior, and HTMX OOB server errors.

Typical pattern:

1. Call `field-plan` with schema, field key, and error id.
2. Merge returned `:attrs` into the input/control.
3. Merge returned `:script` into the control’s `:_` attr.
4. Render a stable error container.
5. On server rejection, return `render-oob-errors` or `render-oob-error-map`.

Server validation is truth. Client validation is UX.

Known validation helpers:

- `field-plan`
- `empty-field-plan`
- `render-oob-errors`
- `render-oob-error-map`
- `path->field-id`
- `path->err-id`

If using Malli-derived browser constraints, prefer explicit browser-safe patterns such as `:gesso.html/pattern` or `:html/pattern` when the backend regex is too complex for HTML5 pattern use.

## Repeated Failure Modes and Project-Specific Guidance

### Do Not Invent Capabilities

Do not claim to have run a REPL, tests, browser, server, local commands, or inspected a repo unless the current chat actually has that tool access and has used it.

If source files are missing, ask for the needed files or make a clearly labeled best-effort assumption.

Avoid advice copied from coding-agent prompts that assumes persistent repo access, live REPL access, filesystem editing, background work, or autonomous test execution unless those tools are actually available.

### HTMX Swap Edge Cases

Be very careful about which element owns HTMX/SSE/live attributes and which element gets replaced.

A recurring failure mode is replacing the element that owns behavior:

- `hx-get`
- `hx-post`
- `hx-trigger`
- `sse-connect`
- `hx-ext`
- `data-gesso-live-*`
- continuity config
- optimistic-action hooks

If an element owns behavior and is replaced with markup that does not include the same behavior, the UI may appear to work once and then silently stop updating.

Preferred pattern:

```text
stable outer root:
  owns HTMX/SSE/live/continuity attrs

replaceable inner target:
  receives fragment swaps
```

Avoid `hx-swap="outerHTML"` on a behavior-owning source element unless the server response re-renders a fully equivalent replacement with all required attrs and hooks.

When using OOB swaps or `hx-swap="none"`, remember that the visible DOM may still change through OOB fragments.

When debugging live update issues, inspect the post-swap DOM, not just the initial rendered HTML. Confirm that the behavior-owning attrs still exist after the first swap.

### Gesso Live / Continuity Pitfalls

Repeated symptoms of incorrect live/continuity structure:

- SSE frames arrive but no follow-up HTMX request happens.
- A fragment updates once and then never again.
- Scroll jumps to the top after live replacement.
- Details/accordion open state is lost after replacement.
- Focus restoration causes the browser to scroll to the wrong element.
- The element with `hx-get` / `hx-trigger="sse:..."` was replaced by content lacking those attrs.

Preserve a stable continuity root and replace only the intended fragment target.

### Optimistic UI Rules

Optimistic UI may make an interaction feel instant, but the server still owns truth.

Safe optimistic behavior:

- disable clicked buttons
- show pending text like “Sending…” or “Confirming…”
- render a temporary visual state
- prevent double submission
- restore or replace with server-confirmed markup

Avoid optimistic states that falsely claim irreversible success before the server confirms it. For example, say “Sending your code…” instead of “Code sent!” until the server response confirms the send.

If an action can fail due to validation, rate limits, auth, database failure, or provider failure, the server response must be able to replace the optimistic UI with a corrected state and error message.

### Forms and Buttons

Gesso buttons default to `type="button"` unless overridden. Inside forms, submit buttons must explicitly pass:

```clojure
:attrs {:type "submit"}
```

For HTMX form actions, prefer letting HTMX include the closest enclosing form’s values rather than manually collecting form state.

Use anti-forgery inputs where required by Biff/Gesso form helpers.

### Hiccup and Rum Rendering Gotchas

Rum expects `:style` to be a map, not a CSS string.

Correct:

```clojure
{:style {:color "var(--muted-foreground)"}}
```

Incorrect:

```clojure
{:style "color:var(--muted-foreground);"}
```

Use Hiccup vectors and attribute maps consistently. Avoid raw HTML-string habits in server-rendered Hiccup.

### Avoid a Sea of Parens

When Hiccup becomes deeply nested, break it up.

Prefer:

- small private helper functions
- named subcomponents
- local `let` bindings for major regions
- `attr.clj` functions for attrs/classes
- `core.clj` functions for visible structure
- data normalization before rendering

Good pattern:

```clojure
(let [header (card-header opts)
      body   (card-body data)
      footer (card-footer actions)]
  (g/card {:content [header body footer]}))
```

Avoid one massive nested form with all attrs, data shaping, branching, and rendering inline.

If rendering logic needs complex conditionals, compute simple view data first, then render from that view data.

### Styling Guidance

Use Gesso semantic classes and theme tokens first. Avoid one-off inline styling unless the component is experimental, the style is highly local, or no semantic class exists.

Prefer component-scoped CSS using stable `data-*` attrs when styling becomes substantial.

Do not invent external stylesheet build steps unless the project actually uses them.

### Future Chat Behavior

When helping with this codebase:

- use `[gesso.core :as g]` in app views when possible
- prefer existing Gesso components before custom HTML
- prefer HTMX/OOB/live helpers before custom JavaScript
- preserve stable behavior-owning wrapper elements
- ask for component source before guessing a non-obvious component API
- state assumptions when source is incomplete
- avoid pretending unavailable tooling exists
- prefer complete file replacements for code edits

## HumanHelp / Gessokit Notes

Current HumanHelp app work lives under `src/net/humanhelp/site`. The old generated/demo app was renamed from `humanhelp` to `example`, so do not put real product behavior into `src/net/humanhelp/example` unless intentionally maintaining the example.

Project-level reusable components live under:

```text
src/net/humanhelp/components
```

For the phone auth effort:

- `one_time_code` is a dumb visual component: one real input, numeric keyboard, `autocomplete="one-time-code"`, styled as separate digits.
- `phone_auth` is a reusable auth flow component/module that can own most phone-code auth flow behavior.
- Side effects such as SMS sending, user lookup/create, and sign-in should pass through explicit injected functions/ports.
- A default console SMS implementation is acceptable initially.
- Future default real SMS provider can live in the component/module area, with API keys configured by env vars.
- The app should be able to override SMS by passing `:send-sms!`.

Phone auth should be declarative to use: configure routes, redirects, code length/TTL/rate limits, and injected auth functions.

Reusable app-local components should be designed so they can later be promoted to Gesso: pure rendering, explicit options, stable data attrs, theme tokens, and injected side-effect functions for app-specific behavior.
