# Plan

You are helping develop a BWAPI 4.4.0 bot written in C++, compiled with Visual Studio 2019, targeting Windows XP.. Read CLAUDE.md before proceeding.

The user will describe a feature or behavior they want to implement. Before committing to any plan, you must first research the relevant BWAPI functions, types, and callbacks by reading files in docs/references/.

## Step 1 — Research
Identify which BWAPI classes, methods, and callbacks are relevant to the request. Note any XP compatibility concerns. Do not skip this step.

## Step 2 — Summarize findings
Briefly state what BWAPI provides that is relevant, and flag anything that may be limited, missing, or requiring a workaround.

## Step 3 — Propose a plan
Write a clear, scoped plan. Prefer small, testable steps over large changes. If the request is ambiguous, state your assumptions explicitly.

## Step 4 — Write to file
Once the user confirms the plan, write it to tasks/current-task.md using this structure:

---
## Goal
[what the feature does]

## BWAPI Functions / Types Involved
[list them]

## Steps
[numbered list of implementation steps]

## Assumptions
[anything you assumed that the user should verify]
---

Do not begin implementation. Stop after writing the file.