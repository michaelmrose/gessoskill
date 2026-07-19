#+title: Git Commit Message Best Practices
#+author: Skills Notes
#+date: 2026-07-15

* Introduction
A diff shows *what* changed, but only the commit message can tell you *why*. A well-cared-for log is a source of pride and productivity.

* The Seven Rules
1. Separate subject from body with a blank line.
2. Limit the subject line to 50 characters.
3. Capitalize the subject line.
4. Do not end the subject line with a period.
5. Use the imperative mood in the subject line.
6. Wrap the body at 72 characters.
7. Use the body to explain "what" and "why" vs. "how".

* The Imperative Test
A properly formed Git commit subject line should always complete the following sentence:
: "If applied, this commit will [your subject line here]"

- Bad: "Fixed the bug with Y" -> "If applied, this commit will fixed..." (Incorrect)
- Good: "Fix the bug with Y" -> "If applied, this commit will fix..." (Correct)

* Example Template

#+begin_src text
Summarize changes in around 50 characters or less

More detailed explanatory text, if necessary. Wrap it to about 72
characters or so. The blank line separating the summary from the
body is critical.

Explain the problem that this commit is solving. Focus on why you
are making this change as opposed to how (the code explains that).

- Bullet points are okay, too

Resolves: #123
See also: #456
#+end_src

* Tips
- If you find it hard to summarize, you are likely committing too many changes at once. Strive for atomic commits.
- Future maintainers (including your future self) will thank you for the context.
