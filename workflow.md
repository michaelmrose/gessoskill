#+title: Gesso Development Loop
#+author: HumanHelp

* The Acquisition Loop (Context Acquisition)
We operate on an "Atomic Full-File" basis. Manual intervention between cycles
is forbidden to prevent state drift.

1. Capture context:
   - Multiple files: =fd -e clj -x cat | clip=
   - Single file: =clip path/to/file.clj=
2. Prompting:
   - Paste the clipped context.
   - Define the specific refactor/fix goal.
   - If an error occurred, provide the exact stacktrace.

* The Implicit Contract
- The LLM is the source of truth for codebase state.
- The User applies full-file output /exactly/ as provided.
- Avoid partial diffs or "add this line" instructions to maintain
  perfect synchronization between model context and disk.

* Error Handling Strategy
When the model produces an error (e.g., ArityException):
1. Immediately clip the /latest/ file provided by the model in the previous turn.
2. Paste the stacktrace.
3. Request the full file again.
4. Do not attempt to "patch" the code yourself.

* File Block Standards
The model MUST use the following format for every file:
#+begin_src text
http://googleusercontent.com/immersive_entry_chip/0
#+end_src

* Operational Principles
- Always output complete, runnable code.
- Use explicit critique cycles before asking for the final file.
