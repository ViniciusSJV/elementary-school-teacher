# elementary-school-teacher

**Explains your code so your grandma gets it. And so do you.**

The most patient senior dev in the room, as an agent skill.

## The problem

Ask an AI agent to explain a project, a workflow or a PR, and you get dense jargon, acronyms and implementation detail. Only an expert can follow it.

This skill makes the agent explain like a great elementary school teacher: short, concrete, never condescending, and still correct.

**Before**

> Fixed a TOCTOU race in the idempotency middleware: GET and SETNX weren't atomic, so concurrent retries could both pass the check and double-charge. Replaced with a single SET NX EX.

**After**

> The bug: a customer could be charged twice if their payment was sent twice at almost the same moment.
>
> Imagine two cashiers who both ask "was this receipt paid?" at the same second. Both hear "no", and both charge.
>
> The fix turns "check" and "mark as paid" into one single step. Now the second cashier always hears "yes, paid".
>
> Want to see the exact line that changed?

## What it does

- Big picture first: what it is and why it exists, before how it works.
- Everyday words. Unavoidable terms get defined once, in plain words.
- One analogy from daily life, when it helps. Never a pile of them.
- Flows told as a short numbered story.
- Simplifies, but never lies. If something important is left out, it says so.
- Answers what you asked, then offers to go one level deeper.

## Install

**Claude Code (plugin)**

```
/plugin marketplace add ViniciusSJV/elementary-school-teacher
/plugin install elementary-school-teacher@elementary-school-teacher
```

**Claude Code (personal skill, no plugin)**

```bash
git clone https://github.com/ViniciusSJV/elementary-school-teacher
cp -r elementary-school-teacher/skills/elementary-school-teacher ~/.claude/skills/
```

**Codex**

Copy [`AGENTS.md`](AGENTS.md) into your project root. To use it everywhere, add its content to `~/.codex/AGENTS.md`.

**Cursor**

Copy [`.cursor/rules/elementary-school-teacher.mdc`](.cursor/rules/elementary-school-teacher.mdc) into your project's `.cursor/rules/` folder.

**Windsurf**

Copy [`.windsurf/rules/elementary-school-teacher.md`](.windsurf/rules/elementary-school-teacher.md) into your project's `.windsurf/rules/` folder.

**Any other agent**

Copy [`AGENTS.md`](AGENTS.md) into your project, or paste it into the agent's rules or system prompt.

## Usage

It kicks in when you ask for an explanation:

- "explain this", "what does this do"
- "analyze this project / workflow / code"
- "I don't understand", "ELI5", "in plain words"
- "summarize this repo"
- onboarding someone non-technical

In Claude Code you can also call it directly: `/elementary-school-teacher`.

### Depth levels

| Level | Reader |
|-------|--------|
| `kid` | A 10-year-old could follow it. |
| `newcomer` | A smart person who isn't a programmer. **Default.** |
| `junior` | A developer new to this codebase: real terms, each one defined. |

Switch with `/teacher junior` (in Claude Code: `/elementary-school-teacher junior`), or just say "explain it at kid level".

### Turn it off

Say "stop teacher", "normal mode", or `/teacher off`.

## Files

- [`skills/elementary-school-teacher/SKILL.md`](skills/elementary-school-teacher/SKILL.md): the prompt. Source of truth.
- [`AGENTS.md`](AGENTS.md): compact version for any agent that reads a rules file.
- `.cursor/` and `.windsurf/`: the same rules as `AGENTS.md`, in each editor's format.

## Credits

Inspired by [Ponytail](https://github.com/DietrichGebert/ponytail), "the laziest senior dev in the room": one strong, opinionated prompt and no bloat.

## License

[MIT](LICENSE)
