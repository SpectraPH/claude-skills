# claude-skills

Personal collection of [Claude Code](https://claude.com/claude-code) skills and a global `CLAUDE.md`.

## Contents

```
.
├── .claude-plugin/        plugin + marketplace manifests
├── skills/                one folder per skill, each with a SKILL.md
│   └── step-by-step-dev/
└── config/
    └── CLAUDE.md          global instructions (commit, PR, branch, bug ticket formats)
```

| Item | Description |
|------|-------------|
| [step-by-step-dev](skills/step-by-step-dev/SKILL.md) | Strictly controlled development: plan approval first, then one small reviewable step at a time, ending with Stryker mutation testing. |
| [config/CLAUDE.md](config/CLAUDE.md) | Global Claude Code instructions: commit message, PR title/description, branch naming and bug ticket formats. |

## How to use

### Option 1: install all skills as a plugin

Inside Claude Code:

```
/plugin marketplace add SpectraPH/claude-skills
/plugin install spectra-skills@spectra-marketplace
```

Update later:

```
/plugin marketplace update spectra-marketplace
```

### Option 2: copy a single skill

```bash
git clone https://github.com/SpectraPH/claude-skills
cp -r claude-skills/skills/step-by-step-dev ~/.claude/skills/
```

Restart Claude Code. The skill loads automatically when the request matches its `description`, or invoke it by name, e.g. `/step-by-step-dev`.

### Use the global CLAUDE.md

Global instructions apply to every project. Copy the file to your user config:

```bash
cp claude-skills/config/CLAUDE.md ~/.claude/CLAUDE.md
```

If you already have a `~/.claude/CLAUDE.md`, merge the sections you want instead of overwriting.

Example values (ticket IDs such as `ABC-123`) are placeholders. Adjust them, and the commit/PR/branch rules, to your own workflow.

### Use as a project skill (shared with a team)

Copy a skill folder into a project repo under `.claude/skills/<name>/` and commit it. Everyone who clones the project gets the skill.

## Add a new skill

1. Create `skills/<skill-name>/SKILL.md`:
   ```markdown
   ---
   name: <skill-name>
   description: Use when <trigger situation>.
   ---

   # Skill title

   Instructions for Claude.
   ```
2. Put supporting files (scripts, templates, references) in the same folder and reference them from `SKILL.md`.
3. Add a row to the table above.
4. Bump `version` in `.claude-plugin/plugin.json` so plugin users get the update.
5. Commit and push.

The `description` is what Claude uses to decide when to load the skill, so state clearly when it should trigger.
