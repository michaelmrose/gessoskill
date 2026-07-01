### `gesso.hyperscript`

**Purpose:** A Clojure DSL for authoring [Hyperscript](https://hyperscript.org/) using data structures. This is used by Gesso component scripts to safely generate and attach browser-side interaction logic to Hiccup nodes.

**Public API:**

* **Script Generation:**
* `(hs & forms)`: The main entry point. Compiles Clojure data structures into a formatted Hyperscript string.
* *Usage:* `(hs [:on :click [:set :my.dataset.open "true"]])`
* *Supported Statements:* `[:on event & body]`, `[:if test then else]`, `[:for binding source & body]`, `[:set target value]`, `[:let target value]`
* *Supported Scopes:* `[:local name]`, `[:global name]`, `[:element name]`




* **Attribute Manipulation:**
* `(merge-script-attr attrs script-string)`: Merges a Hyperscript string into a Hiccup attribute map under the `:_` key.
* `(attach-script-to-node node predicate script)`: Conditionally attaches a script to a Hiccup node if the node matches the predicate.
* `(attach-script-to-children-by-tag children tag script)`: Applies a script to all children within a Hiccup vector that match a specific tag.



**Key Rules:**

* **Script Generation:** Use `hs` to generate raw Hyperscript strings. These are almost always assigned to the `:class` or `:_` (Hyperscript) attribute of a Hiccup node.
* **Naming:** Namespaced symbols are not supported inside `hs` forms. Use simple symbols or keywords.
* **Hyperscript vs. Clojure:** Hyperscript logic in `hs` is transpiled. Do not mix Clojure evaluation logic with the `hs` DSL; once inside an `hs` block, you are writing in the Hyperscript DSL.
* **Composition:** Always use `merge-script-attr` when combining scripts, as it correctly preserves existing `:on` or `:_` logic in the attribute map.
