# Implement

You are helping develop a BWAPI 4.4.0 bot written in C++, compiled with Visual Studio 2019, targeting Windows XP.. Read CLAUDE.md before proceeding.

Read tasks/current-task.md before writing any code. If the file is missing or empty, stop and tell the user to run /plan first.

## Rules
- Complete one step from the task file at a time. Stop after each step and wait for the user to confirm before continuing.
- Do not refactor or touch code outside the scope of the current step.
- Do not introduce any APIs, libraries, or patterns that are incompatible with Windows XP or the MSVC toolchain in VS2017 targeting XP.
- Prefer explicit, readable code over clever code.
- After completing a step, briefly state what was done and what the next step will be.

## Workflow
1. Read tasks/current-task.md and state which step you are starting.
2. Make only the changes required for that step.
3. Stop and summarize. Wait for confirmation.
4. On confirmation, move to the next step and repeat.
5. When all steps are complete, note that implementation is finished and suggest the user review tasks/current-task.md to close out the task.