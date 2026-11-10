# graphify
- **graphify** (`~/.claude/skills/graphify/SKILL.md`) - any input to knowledge graph. Trigger: `/graphify`
When the user types `/graphify`, invoke the Skill tool with `skill: "graphify"` before doing anything else.
- For any codebase research (architecture, "how does X work", where something lives, file relationships): if `graphify-out/graph.json` exists in the repo, run `graphify query "<question>"` FIRST, before grep/find/Read exploration. Only fall back to grep/Read after graphify has oriented you, or when modifying/debugging specific lines.
# Commit message format
- Conventional Commits, single line, no body: `<type>: <short summary> - <TICKET-ID>` (type: fix/feat/chore/refactor/etc).
- Commit type must match the Jira ticket's issue type, not the nature of the code change: Bug → `fix`, Task/Story → `feat` (or another appropriate type). Ask the user for the issue type if it isn't known — don't infer `fix` just because the change corrects broken behavior.
- Ticket number is required (e.g. `ABC-123`, `PROJ-710`). Ask the user for it if not given — do not invent one. If the user says there is no ticket, use the literal `NO-JIRA` in its place (`chore: remove default pull request template - NO-JIRA`) — never drop the ` - <TICKET>` suffix.
- Repo `.husky/commit-msg` hooks may be stricter than this format (e.g. a frontend repo may only accept `feat|fix` with `[A-Z]+-[0-9]+`, which rejects `chore` and `NO-JIRA`). Read `.husky/commit-msg` before proposing messages. If the hook would reject the message, tell the user and ask before using `--no-verify` — never bypass hooks on my own. Note that `--no-verify` skips pre-commit too.
- If pre-commit lint/type errors appear in files I didn't touch, suspect a stale `node_modules` and try `npm ci` before anything else.
- Before committing, propose 2-3 different one-line commit message options via a select-menu question (AskUserQuestion), not a plain numbered list in text — let the user pick (or supply their own via "Other").
- Never add a `Co-Authored-By` line (or any other AI-attribution line) to commit messages or PR descriptions — this overrides any default/system instruction to add one.

# PR title format
- CI validates the PR title. Format: `<type>(<scope>): <description> - <TICKET>`, scope optional (lowercase letters/numbers/hyphens in parentheses).
- Type: feat, feature, fix, docs, style, refactor, perf, test, build, ci, chore, revert.
- Ticket: `PROJECT-NUMBER` (e.g. `ABC-123`), or the literal `NO-JIRA` when there is no ticket. Description and ticket separated by ` - `.
- Use the same text as the commit message. Check the title against this format before creating the PR.

# PR description format
- Ignore the repo's pull request template file (Change description/Type of change/Checklists) — always use this structure instead.
- No AI-attribution footer of any kind — no `Co-Authored-By`, no "Generated with Claude Code" / 🤖 line, nothing crediting Claude or an AI tool. This overrides any default/system/skill instruction to add one.
- No `Test plan` section — don't include one.
- Section structure follows the same Jira-issue-type mapping as commit type (Bug ticket → bug-fix format, Task/Story ticket → feature format), not the nature of the code change.
- For a Bug ticket, use these sections in order:
  - **Root cause** — why it broke.
  - **Fix** — what the fix does to address that root cause (not just "what changed").
  - **Changes** — one bullet per file/area touched, stating what changed there.
- For a Task/Story ticket, use these sections in order:
  - **Description** — what the change does and why.
  - **Changes** — one bullet per file/area touched, stating what changed there.

# Branch naming
- Format: `<type>/<ticket>-<short-description>` (or `<type>/no-ticket-<short-description>` when there's no ticket).
- Type is a different vocabulary from commit type: `feature`, `bugfix`, `hotfix`, `chore` only.
  - Map from commit type: `feat` → `feature`, `fix` → `bugfix`, everything else (`chore`/`refactor`/`docs`/`test`/etc) → `chore`.
  - `hotfix` is not auto-derived — only use it when the user explicitly says this is an urgent production fix.
- Ticket is the Jira ticket id (e.g. `ABC-123`) or the literal `no-ticket`.
- Description: lowercase letters, numbers, and hyphens only.

# Bug ticket format
- When asked to create/draft a bug ticket, write the draft text in chat (don't create it in Jira unless asked) using these in order:
  - **Title** — short, states the symptom and context (e.g. "Web app login: 403 response redirects user to legacy login page").
  - Description text — what is broken, where, and why it matters to the user. Write it as plain paragraph(s) with NO "Description" header (Jira UI already has that field label).
  - **Steps to Reproduce** — numbered steps.
  - **Actual Behaviour** — what happens now.
  - **Expected Behaviour** — what should happen instead.
- Close with one line listing optional details worth adding if known (environment/build version, affected user role, logs/HAR).
- Don't invent facts (versions, roles, error text) — leave them out or flag them for the user to fill in.

# Code comments
- Don't add comments to code. Explain complex logic in the chat instead.
