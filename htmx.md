# HTMX / Biff / Hiccup / Gesso Working Context

This is a pasteable context packet for future chats helping with Clojure web apps that use Biff, Hiccup/Rum, HTMX 2.x, and Gesso/Gessokit. It is not a complete HTMX reference. It is a compact, project-specific guide meant to prevent bad HTMX advice, especially accidental HTMX 4 suggestions and recurring swap/SSE bugs.

## Scope

This project uses HTMX 2.x.

Do not suggest HTMX 4 syntax unless the user explicitly says the project has upgraded.

Avoid suggesting:

- `hx-status`
- `hx-config`
- `hx-ignore`
- `hx-action`
- `hx-method`
- `:inherited` suffixes
- HTMX 4 event names such as `htmx:before:swap`
- HTMX 4-only inheritance behavior

Use HTMX 2.x names and behavior.

Known project runtime examples have used:

```html
<script src="https://unpkg.com/htmx.org@2.0.7"></script>
<script src="https://cdn.jsdelivr.net/npm/htmx-ext-sse@2.2.4"></script>
<script src="https://unpkg.com/htmx-ext-ws@2.0.2/ws.js"></script>
<script src="https://unpkg.com/hyperscript.org@0.9.14"></script>
```

## Core Clojure and Biff Rules

Use clean, idiomatic, data-oriented Clojure.

- Biff/Ring handlers usually receive one `ctx` map.
- Destructure `ctx` where useful.
- Keep views pure. Do not query databases, call APIs, mutate state, log, or deref atoms inside rendering functions.
- Shape data before rendering if it avoids deeply nested Hiccup conditionals.
- Prefer complete file replacements over tiny patches when changing route/view/component/HTMX behavior.

Endpoints used by HTMX should return HTML fragments, not JSON.

Use `g/html-response` for normal handlers returning Hiccup/Rum nodes.

Use `g/no-content` or HTTP 204 for successful actions that should not render or swap content.

## Hiccup Attribute Rules

HTMX attributes are ordinary Clojure keyword attrs:

```clojure
[:button {:hx-post "/save"
          :hx-target "#status"
          :hx-swap "outerHTML"}
 "Save"]
```

When using Gesso components, pass raw HTML, HTMX, ARIA, and `data-*` attrs through the component’s `:attrs` option unless the component exposes a more specific public option:

```clojure
(g/button
 {:text "Save"
  :attrs {:type "submit"
          :hx-post "/save"
          :hx-target "#status"
          :hx-swap "outerHTML"}})
```

Rum expects `:style` to be a map, not a CSS string.

Correct:

```clojure
{:style {:color "var(--muted-foreground)"}}
```

Incorrect:

```clojure
{:style "color:var(--muted-foreground);"}
```

Use Hiccup vectors and maps consistently. Avoid raw HTML-string habits in server-rendered Hiccup.

## Gesso Styling and Component Rules

For Gesso/Gessokit/HumanHelp work, prefer:

1. Existing Gesso components.
2. Gesso semantic classes.
3. Gesso theme tokens.
4. Component-scoped CSS using stable `data-*` attributes.
5. Inline style maps only when local, experimental, or unavoidable.

Do not default to Tailwind-heavy Hiccup unless the current file is already using that style.

In app view code, prefer:

```clojure
[gesso.core :as g]
```

Use `:attrs` on Gesso components for `hx-*` attrs.

Gesso buttons default to `type="button"` unless overridden. Inside forms, submit buttons must explicitly set:

```clojure
:attrs {:type "submit"}
```

This is a frequent source of “button does nothing” bugs.

## HTMX 2.x Baseline

HTMX expects HTML responses.

Default swap is `innerHTML`.

Use `hx-swap="outerHTML"` only when you intend to replace the target element itself.

Common `hx-swap` values:

```text
innerHTML
outerHTML
textContent
beforebegin
afterbegin
beforeend
afterend
delete
none
```

Useful modifiers include:

```text
show:none
focus-scroll:false
swap:<time>
settle:<time>
```

Example:

```clojure
{:hx-swap "outerHTML show:none focus-scroll:false"}
```

`hx-swap="none"` suppresses the normal target insertion but still allows out-of-band swaps in the response to be processed.

## Request Attributes

Core request attrs:

```clojure
:gx-get     ; typo: do not use
:hx-get
:hx-post
:hx-put
:hx-patch
:hx-delete
```

Use only actual HTMX attrs such as:

```clojure
{:hx-get "/fragment"}
{:hx-post "/action"}
{:hx-delete "/item/123"}
```

Default triggers:

- `input`, `textarea`, `select` default to `change`.
- `form` defaults to `submit`.
- Most other elements default to `click`.

Use `hx-trigger` for explicit behavior:

```clojure
{:hx-trigger "click"}
{:hx-trigger "keyup changed delay:500ms"}
{:hx-trigger "load"}
{:hx-trigger "every 2s"}
{:hx-trigger "sse:live-update"}
```

Common trigger modifiers:

```text
once
changed
delay:<time>
throttle:<time>
from:<selector>
target:<selector>
consume
queue:first
queue:last
queue:all
queue:none
```

`hx-trigger` is not inherited.

## Targets and Swaps

Without `hx-target`, the requesting element is the target.

Target examples:

```clojure
{:hx-target "#results"}
{:hx-target "this"}
{:hx-target "closest form"}
{:hx-target "closest [data-card]"}
{:hx-target "find [data-content]"}
{:hx-target "next .row"}
{:hx-target "previous .row"}
```

Use `hx-select` to select part of the response:

```clojure
{:hx-select "#results"
 :hx-target "#results"}
```

Use `hx-select-oob` when a normal response should also select OOB fragments:

```clojure
{:hx-select "#main"
 :hx-select-oob "#toast,#toolbar"}
```

## Attribute Inheritance in HTMX 2

Most HTMX attrs inherit in HTMX 2. This is useful but can produce surprising behavior.

Attributes that commonly inherit include:

```text
hx-target
hx-swap
hx-confirm
hx-include
hx-params
hx-headers
hx-vals
hx-ext
```

Attributes that do not inherit include:

```text
hx-trigger
hx-on*
hx-swap-oob
hx-preserve
hx-history-elt
hx-validate
```

Use inheritance deliberately, but inspect the effective DOM when behavior is surprising.

Use `hx-disinherit` to block inherited attrs:

```clojure
{:hx-disinherit "hx-target"}
{:hx-disinherit "*"}
```

Use `unset` when a child should clear an inherited value:

```clojure
{:hx-confirm "unset"}
```

## Forms and Parameters

Input `name` controls submitted parameter names.

A form submission includes the form’s inputs.

Non-GET HTMX requests include values from the associated form, usually the nearest enclosing form.

Prefer placing related controls inside the form and letting HTMX submit values naturally.

Use `hx-include` when a request must include values outside the normal form:

```clojure
{:hx-include "#extra-state"}
{:hx-include "closest form"}
{:hx-include "#board-state-form"}
```

Use `hx-params` to filter submitted params:

```clojure
{:hx-params "name,email"}
{:hx-params "not password"}
{:hx-params "none"}
```

Use `hx-vals` for explicit extra values, not as a replacement for normal form structure:

```clojure
{:hx-vals "{\"source\":\"toolbar\"}"}
```

Use `hx-encoding` for file uploads:

```clojure
{:hx-encoding "multipart/form-data"}
```

## Response Headers

Useful HTMX response headers:

```text
HX-Trigger
HX-Trigger-After-Swap
HX-Trigger-After-Settle
HX-Redirect
HX-Location
HX-Retarget
HX-Reswap
HX-Reselect
HX-Push-Url
HX-Replace-Url
```

Use `HX-Trigger` to decouple server decisions from hardcoded DOM swaps when event-driven behavior is cleaner.

Use `HX-Redirect` when a successful HTMX action should navigate.

For direct browser requests and HTMX requests to the same route, check `HX-Request` server-side and return either a full page or fragment as appropriate. If caching is involved, set `Vary: HX-Request`.

## Out-of-Band Swaps

OOB swaps let one response update elements outside the normal target.

Prefer Gesso OOB helpers when available:

```clojure
(g/with-oob node {:swap :outerHTML})
(g/oob-inner-html "target-id" ...)
(g/oob-outer-html "target-id" node)
(g/oob-replace "target-id" node)
(g/oob-beforeend "target-id" ...)
(g/oob-delete "target-id")
```

Depending on the Gesso namespace/version, lower-level helpers may include names like:

```clojure
with-oob
replace-oob
inner-oob
append-oob
prepend-oob
before-oob
after-oob
delete-oob
```

Use source as authoritative for exact exported names.

OOB is good for:

- validation errors
- status messages
- toasts
- counters
- secondary panels
- refreshing related fragments after a POST
- server-confirmed optimistic action results

Be careful with invalid HTML wrappers. A default `div` wrapper is wrong in table/list contexts. Use an appropriate wrapper such as `tbody`, `ul`, `ol`, `template`, or a Gesso `oob-wrapper` helper if available.

## SSE Extension

HTMX 2 SSE uses the `sse` extension.

Basic shape:

```html
<div hx-ext="sse" sse-connect="/events">
  <div hx-get="/fragment" hx-trigger="sse:live-update"></div>
</div>
```

Relevant attrs:

```text
hx-ext="sse"
sse-connect="/events"
sse-swap="event-name"
sse-close="event-name"
hx-trigger="sse:event-name"
```

`hx-trigger="sse:event-name"` triggers an HTMX request when an SSE event arrives.

`sse-swap="event-name"` directly swaps the SSE event data into the element.

The event name must match the server’s `event:` field. Unnamed SSE messages use `message`.

SSE close event details may include types such as:

```text
nodeMissing
nodeReplaced
message
```

A very important failure mode: replacing the SSE owner may close the connection or remove the behavior that was supposed to keep updates alive.

## Gesso Live / SSE Pattern

For live fragments, preserve a stable outer root that owns HTMX/SSE/live/continuity attrs.

Preferred shape:

```text
stable outer root:
  hx-ext="sse"
  sse-connect="..."
  hx-get="..."
  hx-trigger="sse:live-update"
  hx-target="#inner-fragment"
  hx-swap="outerHTML ..."
  data-gesso-live-*
  data-gesso-live-continuity*

replaceable inner target:
  id="inner-fragment"
  actual rendered content
```

Do not casually `outerHTML`-replace the behavior-owning root. Replace the inner target.

Recurring symptoms of wrong structure:

- SSE frames arrive but no follow-up HTMX request happens.
- Fragment updates once and then never again.
- Scroll jumps to top after replacement.
- Details/accordion open state is lost.
- Focus restoration scrolls to the wrong element.
- An element with `hx-get` / `hx-trigger="sse:..."` was replaced by markup lacking those attrs.

When debugging, inspect the DOM after the first swap, not just the initial server-rendered HTML.

## `hx-swap="none"` and OOB

`hx-swap="none"` means the main response is not inserted into the normal target.

It does not mean “nothing changes.”

OOB fragments in the response still process and can update the DOM.

This matters for:

- toast responses
- validation responses
- lifecycle-action responses
- optimistic UI confirmation/recovery
- continuity capture/restore paths

## Error Responses

In HTMX 2, non-2xx responses are not swapped by default.

Options:

- Return 200 with an error fragment if the error is part of normal UI flow.
- Use OOB fragments for validation errors/status updates.
- Add explicit `htmx:beforeSwap` handling if the app intentionally swaps non-2xx bodies.
- Use `HX-Retarget` / `HX-Reswap` when useful.

For form validation in this project, prefer Gesso validation/OOB helpers where available.

## Polling

`hx-trigger="every 2s"` polls.

Server can stop polling by returning status 286.

Prefer server-driven poll termination over complicated client-side conditions.

For repeated load polling:

```clojure
{:hx-get "/status"
 :hx-trigger "load delay:1s"
 :hx-swap "outerHTML"}
```

The server can return a replacement with the same trigger to continue, or without it to stop.

## Semantic Elements and Accessibility

Use semantic controls.

Good:

```clojure
[:button {:hx-post "/claim"} "Claim"]
[:a {:href "/requests"
     :hx-get "/requests"
     :hx-target "#main"}
 "Requests"]
```

Avoid:

```clojure
[:div {:hx-post "/claim"} "Claim"]
[:a {:hx-delete "/request/1"} "Delete"]
```

Use buttons for actions. Use anchors with `href` for navigation.

Do not use `hx-post` or `hx-delete` on anchors unless there is a strong reason. It is usually semantically wrong.

Use live regions for dynamic updates that should be announced:

```clojure
[:div {:id "search-results"
       :aria-live "polite"
       :aria-atomic "false"}]
```

Manage focus intentionally after large swaps. In Gesso live/continuity contexts, prefer existing Gesso continuity behavior rather than ad hoc focus JavaScript.

## Security Notes

Escape user content server-side.

Do not render untrusted user HTML as executable markup.

If an area intentionally renders user-generated HTML, prevent HTMX from processing injected `hx-*` attrs. In HTMX 2 this is `hx-disable`.

```clojure
[:div {:hx-disable true}
 user-rendered-content]
```

Do not use HTMX 4’s `hx-ignore` unless the project has upgraded to HTMX 4.

## Indicators, Disabled Elements, and Sync

Use `hx-indicator` for request indicators:

```clojure
{:hx-indicator "#spinner"}
```

HTMX applies request lifecycle classes such as `htmx-request`.

Use `hx-disabled-elt` in HTMX 2 to disable elements while a request is active:

```clojure
{:hx-disabled-elt "this"}
{:hx-disabled-elt "closest form"}
```

Do not suggest HTMX 4’s renamed disabling attr.

Use `hx-sync` to control concurrent requests:

```clojure
{:hx-sync "closest form:drop"}
{:hx-sync "this:queue last"}
{:hx-sync "this:abort"}
```

For mobile tap storms and double-submit protection, `closest form:drop` is often useful.

## Optimistic UI

Optimistic UI may make the interface feel instant, but the server remains truth.

Safe optimistic behavior:

- disable clicked controls
- show pending text like “Sending…” or “Confirming…”
- prevent double submission
- render temporary state
- replace with server-confirmed markup

Avoid text that claims final success before confirmation. Use “Sending your code…” instead of “Code sent!” until the server confirms.

If validation, rate limits, auth, provider failure, or DB failure can occur, the server response must be able to replace the optimistic UI with corrected state and an error.

## JavaScript Rule

Do not write ad hoc page-specific JavaScript for ordinary component state sync or DOM updates.

Prefer, in order:

1. Server-rendered HTMX fragments.
2. HTMX OOB swaps.
3. `HX-Trigger` response headers.
4. Gesso live/continuity/optimistic helpers.
5. Reusable component behavior in `scripts.clj`, preferably via `gesso.hyperscript`.

Custom JavaScript is acceptable when packaged as reusable component behavior or when a browser API requires it, but do not use it to reinvent HTMX.

## Htmx Events in This Project

Use HTMX 2 event names.

Common useful events:

```text
htmx:configRequest
htmx:beforeRequest
htmx:beforeSend
htmx:afterRequest
htmx:beforeSwap
htmx:afterSwap
htmx:afterSettle
htmx:responseError
htmx:sendError
htmx:load
htmx:beforeCleanupElement
htmx:validation:validate
htmx:validation:failed
htmx:oobBeforeSwap
htmx:oobAfterSwap
htmx:sseOpen
htmx:sseError
htmx:sseBeforeMessage
htmx:sseMessage
htmx:sseClose
```

In `hx-on:` attributes, HTML lowercases attribute names, so use kebab-case event spelling when needed:

```html
<form hx-on:htmx:before-request="showSpinner()"></form>
```

For this project, prefer Gesso hyperscript and component scripts over scattered `hx-on:` JavaScript.

## `hx-boost` Guidance

`hx-boost="true"` can progressively enhance normal links/forms, but do not blindly boost the whole app shell or body.

Use it narrowly and deliberately.

Boosted anchors should have real `href`s.

Boosted forms should still be valid forms.

If using boosted navigation, ensure server responses work both as normal full-page responses and HTMX responses where appropriate.

## Debugging Checklist

When HTMX behavior fails, check:

- Did the request fire in the network tab?
- Was the request sent to the expected URL/method?
- Did the response return HTML, not JSON?
- Is the status code one HTMX will swap?
- Is the target selector correct and present exactly where expected?
- Is `hx-swap` correct for replacing children vs replacing the element itself?
- Did `outerHTML` replace the element that owned `hx-*`, SSE, or continuity attrs?
- Does the post-swap DOM still contain the required behavior attrs?
- Are inherited `hx-target`, `hx-swap`, `hx-include`, or `hx-confirm` affecting this element?
- Are form values actually submitted under the expected names?
- Is a Gesso button inside a form missing `:attrs {:type "submit"}`?
- Does the response contain valid HTML for its target context?
- Are OOB fragments using valid wrappers?
- Is `hx-swap="none"` being used with OOB fragments as intended?
- Is the SSE owner being replaced?
- Are you looking at initial HTML only, or the DOM after the first swap?

## Future Chat Instruction

When helping with this codebase:

- Use HTMX 2.x behavior only.
- Do not suggest HTMX 4 syntax unless explicitly asked.
- Use Hiccup keyword attrs.
- In Gesso components, pass HTMX attrs via `:attrs`.
- Prefer Gesso helpers and OOB helpers when available.
- Preserve stable behavior-owning roots.
- Replace inner targets, not roots that own SSE/live/continuity behavior.
- Ask for source when exact Gesso component or live helper APIs matter.
- Do not pretend to have REPL, test, browser, filesystem, or server access unless tools are actually available in the current chat.
