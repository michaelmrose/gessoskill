### `gesso.build.find-path`

**Purpose:** A build-time utility for Babashka (`bb`) tasks to resolve the filesystem directory path of a specific classpath resource.

**Public API:**

* `(path-for target)`: Resolves the absolute path to the parent directory of a resource found on the classpath. Returns `nil` if the resource cannot be found.
* `(-main & args)`: CLI entry point for usage via `bb -m gesso.build.find-path <resource>`.

**Key Rules:**

* **Build-Time Only:** This namespace is intended for use in build scripts (e.g., finding where theme CSS files are located on disk during a task run).
* **Environment:** Relies on Babashka (`babashka.fs`).
* **Resource Resolution:** Uses `clojure.java.io/resource` as the source of truth, meaning the `target` must be on the classpath.
