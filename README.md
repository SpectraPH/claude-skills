# claude-skills

Personal collection of [Claude Code](https://claude.com/claude-code) skills.

## Skills

| Skill | Description |
|-------|-------------|
| [step-by-step-dev](skills/step-by-step-dev/SKILL.md) | Strictly controlled development: plan approval first, then one small reviewable step at a time, ending with Stryker mutation testing. |

## Install

As a plugin (all skills):

```
/plugin marketplace add SpectraPH/claude-skills
/plugin install spectra-skills@spectra-marketplace
```

Or copy a single skill folder into `~/.claude/skills/`:

```bash
git clone https://github.com/SpectraPH/claude-skills
cp -r claude-skills/skills/step-by-step-dev ~/.claude/skills/
```

## Adding a new skill

1. Create `skills/<skill-name>/SKILL.md` with frontmatter:
   ```markdown
   ---
   name: <skill-name>
   description: Use when <trigger>.
   ---
   ```
2. Put supporting files (scripts, templates, references) in the same folder.
3. Add a row to the table above.
4. Commit and push. Bump `version` in `.claude-plugin/plugin.json` so installed users get the update.
