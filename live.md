# Gesso Live Working Context

This is a pasteable context packet for future chats helping with Gesso/Gessokit live-update work. It is not formal public documentation and it is not a replacement for source. Its purpose is to preserve the mental model, invariants, and repeated failure modes around Gesso live behavior.

Use this alongside `gesso.md` and `htmx.md`.

## Scope

“Live” in this project means server-rendered UI that stays fresh through HTMX, SSE, invalidation events, and Gesso continuity/optimistic helpers.

The preferred model is:

```text
server mutation
  -> durable state changes
  -> publish/invalidate affected scope(s)
  -> client receives SSE notification
  -> client issues HTMX GET for fresh fragment
  -> server returns authoritative HTML
  -> client swaps fragment while preserving continuity state where appropriate
```

The server remains truth. SSE is a notification mechanism, not the source of application state.

## Core Principle

Live should make server-rendered HTML feel immediate without turning the app into a client-side state machine.

Prefer:

- server-rendered fragments
- small HTMX requests
- scoped invalidations
- coalesced notifications
- stable behavior-owning DOM roots
- continuity restoration
- optimistic pending states that the server confirms or corrects

Avoid:

- sending large application state through SSE
- ad hoc page-specific JavaScript
- replacing elements that own SSE/HTMX/live behavior
- duplicating server truth in client state
- optimistic UI that claims final success before confirmation

## Basic Live Vocabulary

### Mutation

A server-side action that changes durable application state.

Examples:

- create request
- claim request
- unclaim request
- mark done
- cancel request
- change board options
- update helper assignment

A mutation should publish invalidations only after the server-side state transition succeeds.

### Scope

A logical region of interest that can be invalidated.

Examples:

```text
store board
request detail
request toolbar
helper queue
toast region
status counter
```

A scope should be broad enough to avoid exploding subscription counts but narrow enough to avoid broadcasting everything to everyone.

### Interest

A client’s declaration that it cares about a scope.

Example:

```text
client A is looking at store 21 board
client B is looking at request 123 details
client C is looking at store 21 toolbar
```

### Invalidation

A notification that a scope is stale.

Invalidation should not usually contain the full new state. It tells the client to refetch the relevant fragment.

### Coalescing

Combining many invalidations for the same scope into one notification or one client refetch.

This is crucial for high fanout. The important goal is timely freshness, not one SSE event per mutation.

### Revision

A monotonic or comparable value used to decide whether a rendered view is stale, fresh, or should show a “new updates available” affordance.

In HumanHelp board work, `visible-revision` mattered because a user could intentionally view an older board state until choosing to refresh.

### Continuity

Client-side preservation/restoration of user-visible state across swaps.

Examples:

- scroll position
- focused element
- details/accordion open state
- selected card
- input/caret state where appropriate
- visible revision
- locally pending optimistic state

### Optimistic Action

A temporary local UI state applied immediately after user interaction while the server request is in flight.

Examples:

- disable clicked buttons
- show “Confirming…”
- visually mark card pending
- prevent double submit

Server response must replace or correct the optimistic state.

## Preferred Live DOM Shape

The most important live rule:

```text
Stable behavior-owning root stays in the DOM.
Replaceable inner target receives fragment swaps.
```

Do not casually `outerHTML`-replace the element that owns:

```text
hx-ext
sse-connect
hx-get
hx-trigger
data-gesso-live-*
continuity config
optimistic action hooks
```

Preferred shape:

```clojure
[:section
 {:id "request-toolbar-live"
  :data-gesso-live-root true
  :hx-ext "sse"
  :sse-connect "/app/streams/request-toolbar"
  :hx-get "/app/fragments/request-toolbar"
  :hx-trigger "sse:live-update"
  :hx-target "#request-toolbar-fragment"
  :hx-swap "outerHTML show:none focus-scroll:false"}

 [:div
  {:id "request-toolbar-fragment"
   :data-gesso-live-fragment true}
  ;; current server-rendered fragment content
  ]]
```

The stable root owns the behavior. The inner fragment is replaced.

If the element with `hx-get` / `hx-trigger="sse:..."` is itself replaced, the replacement must include all behavior attrs again. That pattern is easier to break, so prefer the stable-root pattern.

## SSE Patterns

### Trigger an HTMX Refetch

This is the usual Gesso live pattern:

```html
<div hx-ext="sse" sse-connect="/events"
     hx-get="/fragment"
     hx-trigger="sse:live-update"
     hx-target="#fragment"
     hx-swap="outerHTML">
  <div id="fragment">...</div>
</div>
```

The SSE event only says “something changed.” The HTMX GET obtains fresh HTML from the server.

### Direct SSE Swap

Direct `sse-swap` can be useful for tiny cases, but it is less flexible for Gesso app state:

```html
<div hx-ext="sse" sse-connect="/events">
  <div sse-swap="message"></div>
</div>
```

Prefer HTMX refetch for app fragments because it keeps server rendering as the source of truth.

### SSE Event Names

The event name must match the server’s `event:` field.

Common shape:

```text
event: live-update
data: ...
```

The client listens with:

```html
hx-trigger="sse:live-update"
```

Unnamed SSE messages are `message`.

## Repeated Failure Mode: Replacing the Source

A common bug is this:

```text
1. Initial DOM has element A with hx-get + hx-trigger="sse:live-update".
2. SSE event arrives.
3. HTMX GET fires.
4. Response replaces element A with new content.
5. New content does not include hx-get/hx-trigger/SSE attrs.
6. Future SSE frames arrive but no HTMX request happens.
```

Symptoms:

- works once, then stops
- EventSource still receives frames
- network tab shows no follow-up GET after later SSE events
- initial HTML looked correct, but post-swap DOM is missing behavior attrs

Fix:

```text
Keep behavior attrs on a stable root.
Swap only an inner target.
```

Or ensure the replacement re-renders the full live wrapper exactly.

## Live Fragment Endpoint Rules

A live fragment endpoint should be safe to call repeatedly.

Good live fragment endpoint properties:

- GET request
- idempotent
- reads durable server state
- returns HTML fragment
- does not mutate application state
- can be called after any invalidation
- renders the same wrapper/ids needed for the current swap target
- preserves expected `id`s for OOB/continuity/focus restoration

Avoid mixing mutation and live fragment rendering in one endpoint unless there is a strong reason.

## Mutation Endpoint Rules

A mutation endpoint should:

1. Validate request/auth.
2. Apply durable state change.
3. Publish/invalidate affected scopes.
4. Return an appropriate HTML response, OOB response, redirect, or no-content response.

If the mutation is triggered by an optimistic UI, the response must be able to:

- confirm the optimistic state
- replace pending UI with authoritative markup
- show validation/provider/DB errors
- re-enable or correct controls if needed

Do not publish invalidations before the durable state change succeeds.

## Coalescing and Fanout

The project has already seen that naive fanout can produce huge expected SSE counts while actual useful freshness is much smaller.

Important live performance principle:

```text
The goal is timely notification, not one client event per backend mutation.
```

Coalesce invalidations by scope before fanout when possible.

Example:

```text
100 changes to store 21 board in a short window
  -> one “store 21 board changed” notification
  -> one fragment refetch per interested client
```

Avoid broadcasting every mutation to every client.

Prefer:

- per-flow or per-scope interest indexes
- broad-enough scopes such as store board
- coalescing before fanout
- only notifying clients that declared interest

Avoid:

- global broadcast for every mutation
- per-request subscriptions for every tiny child if store-level invalidation is enough
- duplicate parent/child invalidations when parent subsumes child

## Interest Indexing

A live system needs to know who cares about what.

Conceptually:

```text
scope -> interested clients
client -> active scopes
```

When a client opens a board, it becomes interested in board-related scopes.

When the client leaves, disconnects, or replaces the live root, interests should eventually disappear.

For multi-node deployment, interest propagation may involve Valkey or another shared bus. The immediate design preference has been:

```text
nodes advertise interests
nodes publish invalidations only to interested node inboxes
nodes coalesce local delivery to clients
```

Do not assume a full peer-to-peer mesh is desirable. For a small domain, a custom in-process/Valkey-backed interest bus may be preferable to heavyweight infrastructure at first.

## Scope Design Guidelines

Choose scopes based on UI freshness needs.

Good scope examples:

```text
[:store-board store-id]
[:request request-id]
[:request-toolbar store-id]
[:helper-queue store-id]
[:toast user-id]
```

A scope should answer:

```text
Who needs to know when this changes?
What fragment should they refetch?
Can this scope subsume narrower scopes?
Will this scope over-notify too many clients?
```

Subsumption matters.

If a client is already interested in `[:store-board 21]`, then a change to request 123 in store 21 may not need a separate `[:request 123]` notification for that same board fragment.

## Continuity

Continuity exists because server-rendered swaps can otherwise destroy useful client-visible state.

State worth preserving may include:

- scroll position
- selected card/request
- expanded details
- focused input
- caret position in some cases
- open dropdown/dialog state in some cases
- visible revision
- local pending state

Continuity should not preserve stale application truth. It should preserve interaction state around fresh server truth.

### Continuity Root

Use a stable root for continuity capture and restore.

Do not replace the continuity root unless the replacement includes equivalent continuity attrs/hooks and the continuity system is designed for that.

### Scroll Pitfalls

Repeated scroll problems have occurred.

Potential causes:

- replacing too high in the DOM
- focus restoration causing browser scroll
- target missing after swap
- selected element id changing
- continuity root replaced
- OOB swap path not participating in capture/restore
- browser default focus scroll

Use swap modifiers such as:

```text
show:none
focus-scroll:false
```

where appropriate.

### Details / Accordion State

If `<details>` or accordion state should persist, continuity must know how to capture and restore it, or the server must render it from explicit state.

Do not assume browser open state survives `outerHTML` replacement.

## Optimistic Actions

Optimistic actions are useful for making server-rendered HTMX UI feel instant.

Good optimistic behavior:

- immediately disable clicked controls
- show pending pill/text such as “Confirming…”
- prevent double click/double submit
- keep layout stable while pending
- replace with authoritative server markup

Bad optimistic behavior:

- claiming success before server confirmation
- allowing repeated clicks while pending
- leaving controls disabled forever on error
- making local state impossible for server response to correct
- moving important content so the card resizes/jumps

For HumanHelp request cards, the desired behavior included:

```text
Confirming… pill near the normal status pill
buttons disabled while pending
no double submit
server result replaces/corrects the optimistic card
```

If a pending indicator changes layout height, prefer absolute positioning or reserved space.

## Live + Forms

When a form lives inside a live fragment:

- preserve hidden state inputs needed by the server
- preserve anti-forgery inputs
- be explicit about submit button type
- avoid replacing the form owner in a way that drops HTMX attrs
- ensure optimistic submit behavior does not prevent server correction

HTMX non-GET requests include the associated form’s values. Prefer this over manual value gathering.

For board state, hidden inputs have included values such as:

```text
q
selected
visible-revision
created-order
mine-first?
unclaimed-first?
show-terminal?
```

Use the actual current source as authoritative.

## Live + OOB

OOB swaps are useful with live for secondary updates:

- toast region
- error region
- counters
- toolbar
- status banners
- validation messages

Remember:

```text
hx-swap="none" can still process OOB swaps.
```

A response can update several UI regions without replacing the normal target.

Be careful with OOB wrappers in table/list contexts.

## Live + Error Handling

Live update requests can fail.

A robust live fragment should handle:

- auth expiration
- 303 redirect to sign-in in dev/direct curl contexts
- missing/invalid client id
- stale selected item
- request no longer visible due to filters
- invalid visible revision
- DB/pool failure
- network interruption
- SSE reconnect

Do not assume a curl request to a live stream endpoint works if the endpoint requires login. Prior debugging saw direct curl return 303 because the user was not signed in.

## Live Stream Debugging Checklist

When live updates fail, check all of these:

### SSE Connection

- Is the EventSource connected?
- Is `hx-ext="sse"` on an ancestor of the listener?
- Is `sse-connect` pointing at the right endpoint?
- Is the endpoint authenticated?
- Is the browser receiving SSE frames?
- Does the event name match `hx-trigger="sse:<event>"`?
- Is the EventSource closing with `nodeMissing` or `nodeReplaced`?

### HTMX Refetch

- After SSE event, does an HTMX GET fire?
- Is the element with `hx-get` still in the DOM after prior swaps?
- Does `hx-trigger` still exist after the first swap?
- Is inherited `hx-target` or `hx-swap` affecting the request?
- Is the target selector correct?
- Is the server returning HTML with the expected target id?

### Swap Shape

- Are you replacing the stable root by accident?
- Does the replacement preserve required ids?
- Does the replacement contain behavior attrs if it owns behavior?
- Is `outerHTML` necessary, or should only an inner target update?
- Is the returned HTML valid for its DOM context?

### Continuity

- Is the continuity root stable?
- Does capture happen before the swap path being used?
- Does restore happen after normal swaps and OOB swaps?
- Is focus restoration causing scroll jumps?
- Are selected/open elements identifiable after the swap?

### Server Side

- Was the mutation successful before invalidation?
- Was the right scope invalidated?
- Are clients actually registered as interested in that scope?
- Is coalescing delaying or dropping incorrectly?
- Are there errors in the server logs?
- Is DB connection pool state healthy?
- Did a namespace reload/restart leave old code running?

## Known Project Failure Modes

### Behavior Owner Replaced

The element that owned `hx-get` / `hx-trigger="sse:..."` was replaced by rendered content lacking those attrs.

Result:

```text
SSE frames arrive, but no follow-up request happens.
```

Fix:

```text
Stable root owns behavior.
Inner target is swapped.
```

### SSE Works But Fragment Does Not Refresh

If live toast or simple shared counter works but another fragment does not, suspect DOM shape or replaced behavior attrs before suspecting the whole SSE infrastructure.

### Curl Gives 303

Some live endpoints require login. Direct curl may return 303 to sign-in and is not proof the SSE endpoint is broken for authenticated browser sessions.

### Browser Reports Interrupted EventSource

Firefox/Chromium may log interrupted EventSource connections during reloads, route changes, or node replacement. Distinguish expected teardown from persistent reconnect failure.

### Old Source Still Running

If dependency wiring points Gessokit at a released Gesso dependency instead of local root, editing local Gesso files may not affect the running app.

If behavior does not match source changes, check dependency wiring and restart/reload state.

### Connection Pool / DB Failure

A server-side DB/pool failure can surface as a broken HTMX response or “something went wrong” page after an action. Do not assume the client-side live code caused every failure.

Known example class of error:

```text
java.sql.SQLException: Connection is closed
```

## Live Test Guidance

Tests should cover both model behavior and live wiring expectations.

Useful test categories:

- pure model/view-state normalization
- visible request filtering/sorting
- terminal vs pending semantics
- route parameter normalization
- fragment rendering contains stable ids
- live wrapper contains required `hx-*`/SSE attrs
- mutation invalidates expected scopes
- coalescing reduces duplicate events
- optimistic state can be corrected by server response
- OOB fragments have expected ids and swap attrs

For UI live bugs, source-level tests are helpful but not sufficient. The post-swap DOM shape is often the real issue.

## Code Organization Guidance

Live code gets hard to read quickly. Avoid a sea of parens.

Prefer separating:

```text
model/data normalization
scope calculation
fragment view data
live wrapper rendering
inner fragment rendering
mutation handler
invalidation publish
continuity attrs
optimistic attrs/scripts
```

Good shape:

```clojure
(defn board-view-data [ctx params] ...)
(defn board-fragment [view-data] ...)
(defn board-live-root [ctx view-data] ...)
(defn handle-board-fragment [ctx] ...)
(defn handle-claim [ctx] ...)
```

Avoid one giant handler that:

```text
reads params
queries DB
normalizes state
builds every attr
renders nested Hiccup
mutates DB
publishes live
returns response
```

For components, prefer:

```text
component_name/
  attr.clj
  core.clj
  scripts.clj        ; only if needed
  component_name.css ; only if needed
```

For live-specific components, `attr.clj` is especially useful for stable ids, `data-*` hooks, and HTMX/SSE attrs.

## Suggested Live Wrapper Helper Shape

If writing a reusable helper, think in terms of:

```clojure
(live-fragment
 {:id "request-toolbar"
  :stream-url "/app/streams/request-toolbar"
  :fragment-url "/app/fragments/request-toolbar"
  :event "live-update"
  :target-id "request-toolbar-fragment"
  :scope [:store-toolbar store-id]
  :swap "outerHTML show:none focus-scroll:false"}
 (toolbar-fragment view-data))
```

This is conceptual, not a confirmed exact Gesso API. Use actual source as authoritative.

A helper should make the safe DOM shape easy:

```text
stable root owns stream and trigger
inner child owns replaceable id
```

## Multi-Node Live Direction

The long-term multi-node direction has been:

```text
multiple Aleph web nodes
embedded XTDB nodes
Valkey or similar for live invalidation/interest bus
Redpanda/Kafka eventually for XTDB log durability/upgrades if needed
MinIO/object storage for XTDB storage layer where applicable
```

For live invalidation specifically, the preferred near-term idea is not full mesh and not global broadcast.

Better direction:

```text
each node has local client interest registry
nodes advertise their interests
mutations publish invalidations by scope
only nodes with matching interests receive the invalidation
each node coalesces delivery to its local clients
```

This avoids every node sending every change to every other node/client.

## What Future Chats Should Do

When helping with Gesso live:

- preserve stable roots
- replace inner fragments
- inspect post-swap DOM assumptions
- use HTMX 2 + SSE extension patterns
- avoid HTMX 4 syntax
- keep server as truth
- treat SSE as invalidation/refetch trigger
- coalesce invalidations
- design scopes deliberately
- use OOB for secondary updates
- handle optimistic UI as pending, not confirmed success
- avoid claiming tests/REPL/browser were run unless available
- ask for exact live source when APIs matter

When source and this document conflict, source is authoritative.
