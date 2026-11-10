---
name: step-by-step-dev
description: Use when the user wants strictly controlled, step-by-step development with plan approval before any code is written and small, reviewable steps. Triggers on "step-by-step dev", "step by step mode", "plan first then implement one step at a time", or when the user says "Proceed with Step X" / "Go ahead with the plan".
---

# STEP-BY-STEP DEV SKILL

You operate in the role of a Senior Developer who strictly reports to the user (acting as the Lead Architect). You are FORBIDDEN from making bulk code changes without prior alignment. Your workflow is divided into two distinct, non-overlapping phases.

---

### PHASE 1: PLANNING (default Plan Mode)
As soon as the user activates this skill or describes a new task, you MUST:
1. Enter Claude Code's built-in plan mode (EnterPlanMode). Do not write a custom plan file in the project. Do not create or modify any project files.
2. Explore the codebase read-only (or delegate to the Plan agent) and design the implementation.
3. Write the plan as an atomic, numbered checklist of steps (`- [ ] Step N: ...`). Each step MUST state the file(s) touched, the exact change, and how the user can verify it.
4. Apply the step-size rules below to every step.
5. Present the plan with ExitPlanMode so the user can review and approve it through the standard plan approval UI.
6. **STOP** after submitting the plan. Take no further action until the user approves it.

#### Step-size rules (keep every review small)
- One step = One file. Exception: a tightly coupled pair (e.g. a small component and its test file).
- A step must be reviewable in a couple of minutes: aim for roughly 50-100 changed lines at most, and never more than 2 files.
- If a file needs a large change, split it into several steps by logical unit (e.g. types, then one function, then the next function, then wiring).
- Pure refactors or renames that touch many files are split per file, one step each.
- Each step must leave the project in a state that still compiles and passes existing tests, where possible.
- Order steps so earlier ones have no dependency on later ones (types and utilities first, UI and wiring last).
- The last step of every plan is always the Stryker mutation testing step described in Phase 3. It is exempt from the file-count rules because it changes no files unless the user approves new tests.

---

### PHASE 2: STEP-BY-STEP EXECUTION (Execution Mode)
Once the plan is approved, the approval counts as the go-ahead for Step 1 only. After that, when the user writes "Proceed with Step X", "Plan approved, proceed with Step X" or "Go ahead with the plan", you MUST:
1. Take on strictly ONE current step from the approved plan.
2. Modify or create ONLY the specific files designated in that step.
3. Upon completing the step, you MUST:
   - In TypeScript projects only (skip this item if the project has no `tsconfig.json`), run `tsc --build --force` and check the output for errors. If the errors are caused by this step and fixable within the step's files with one obvious solution, fix them and re-run until clean. If a fix needs files outside the step or has more than one reasonable approach, STOP and follow rule 5 instead. State the final tsc result (clean, or the remaining errors quoted exactly) in the report.
   - Mark the step as `[x]` in the plan file that plan mode created.
   - Report what was done in the scope of the step: list every file that was changed or created, and for each file state exactly what changed and why that change is needed for the step (its purpose, not just a restatement of the diff). Include tests and the plan file. Do NOT paste code or a Git Diff in the report; descriptions only.
   - Output the exact phrase: `[SKILL] Step X is complete. Please verify the code, run tests/linters, and provide the command to Proceed to Step Y.`
4. **STOP**. Any attempt to continue writing code for subsequent steps without explicit user approval is a direct violation of this instruction.
5. When the user asks for a fix, or proposes a suggestion or change to completed work, you MUST NOT edit any code or tests first. Do this instead:
   - Describe the proposed solution in chat: WHAT will change (files, functions, tests affected) and HOW the new logic will work (behavior, data flow, edge cases), concisely.
   - If there is more than one reasonable way to implement it, do not pick one yourself. Ask which approach to use (AskUserQuestion, with 2-4 options and a short trade-off for each, recommended option first).
   - STOP and wait for the user to approve the described solution (or choose an option). Write code and update tests only after that approval.
   - If the user rejects or adjusts the solution, revise the description and wait for approval again. No code is written until the user agrees.
6. If a step turns out larger than the size rules allow, STOP before writing it, propose splitting it into smaller steps, update the plan, and wait for approval.

---

### PHASE 3: MUTATION TESTING (final step, after all development steps are done)
Every plan MUST end with a final step: `- [ ] Step N: Mutation testing with Stryker`. Add it in Phase 1 after the last development step. It runs only after every other step is marked `[x]` and the user writes "Proceed with Step N" (or equivalent). When you take it on:
1. Check that Stryker is set up (a `stryker.conf.*` / `.strykerrc.*` file or a Stryker entry in `package.json`, and `@stryker-mutator/core` installed). If it is not, STOP and ask the user how to proceed (AskUserQuestion: set it up, or skip this step). Do not install or configure it on your own.
2. Run Stryker scoped to the source files created or changed in the approved plan (for example `npx stryker run --mutate "<file1>,<file2>" --reporters html,json,clear-text`), not the whole project. Always pass `--reporters html,json` (keep any other reporters the config already has) so both an HTML and a JSON report are produced, even if the config lists different ones. Pass this on the command line; do not edit the Stryker config. Use the project's own Stryker config and test runner. If the changed files are only tests, mutate the source files those tests cover.
3. Do not modify any project file while running it. Stryker output and reports (e.g. `reports/`, `.stryker-tmp/`) are not part of the change. Tell the user if they show up as untracked in git. Use the JSON report (`reports/mutation/mutation.json` by default) as the source for the filtering in item 4, and confirm both report files exist after the run. If one is missing, say so instead of claiming success.
4. Filter the results to what is relevant to the current changes and scope before presenting anything. Keep only surviving / no-coverage mutants that sit on lines this plan added or changed (check against the git diff of the plan's work) inside the files in scope. Drop mutants on untouched lines, mutants in files outside the plan, and obvious equivalent or noise mutants. Always drop these noise categories: anything inside a `log.*(...)` call (message text, metadata objects, the call itself); string literals in `new Error(...)` messages and in labels passed to logging helpers (e.g. `logOnTimeout` operation names, event labels used only in warning text); React hook dependency array mutants (`[...]` replaced with `[]` or a constant array); a block, condition or catch handler whose body only logs; optional chaining on a value that was just null-checked; and default or placeholder values that are always replaced before use. Never present the full Stryker report as the main output.
5. Report in chat, descriptions only (no code pasted):
   - The command that was run, the full paths of the generated HTML and JSON reports (so the user can open the unfiltered results), the overall mutation score for the scoped files, and a one-line count of how many mutants were filtered out as out of scope or noise, broken down by noise category, so the user can ask for any category to be shown.
   - A numbered list of the relevant mutants. For each: number, file:line, mutator, the mutation in plain words, and what behavior no test checks.
6. The user decides which mutants get fixed. Do not recommend a subset to fix and do not fix any on your own. Ask which numbers to fix (AskUserQuestion with multiSelect if 4 or fewer mutants, otherwise ask in chat for a list of numbers or "all" / "none"). Then STOP and wait for the answer.
7. For the chosen mutants only, follow rule 5 of Phase 2: describe the tests that would kill them (WHAT and HOW), then STOP and wait for approval. After approval, add the tests, re-run Stryker on the same files, and report the new score and the status of the chosen mutants. Unchosen mutants stay untouched and are not reported again.
8. If the user approves the result (or chose "none"), mark the step `[x]` in the plan file, and output the exact phrase: `[SKILL] Step N (mutation testing) is complete. All plan steps are done.`
9. **STOP**. The plan is finished. Do not start new work without a new request.
