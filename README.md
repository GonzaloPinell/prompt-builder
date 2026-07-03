# prompt-builder

An agent skill that turns a vague or messy request into a sharp, well-formed
prompt. Instead of guessing, it walks the user through a proven six-part
formula — **role + context + task + audience + constraints + format** — one
question at a time, always suggesting a recommended answer so the user never
has to choose blind.

Key behaviors:

- **Reads before asking.** It only asks about the components that are
  actually missing or unclear, skipping anything the original request already
  makes obvious.
- **Infers the role automatically.** Rather than asking an open-ended "what
  role should I play?", it derives a composite expert persona from the task
  plus the detected technology/stack (e.g. *"graphic designer expert in hero
  sections, and expert Astro developer"*), then confirms it with the user.
- **Tool-agnostic.** If the host agent exposes a structured multi-choice
  question tool (e.g. `AskUserQuestion` in Claude Code), it uses that for
  clickable options plus a free-text field. Otherwise it falls back to plain
  numbered text questions — same content and logic either way.
- **Two output modes.** *Draft mode* hands back a ready-to-copy markdown
  prompt block; *execute mode* adopts the confirmed role and does the task
  directly, with no prompt block printed.

## Install

Install into any project or agent supported by the [`skills`
CLI](https://skills.sh):

```bash
npx skills add GonzaloPinell/prompt-builder
```

## Usage

The skill triggers automatically whenever a request is vague, rambling, or
under-specified, or whenever you explicitly ask to have a prompt written,
refined, or brainstormed. For example:

> "help me with my site's hero, I don't even know where to start"

The skill will ask, one at a time, about the task, the role (with a
recommended option already derived from your project's stack), the audience,
constraints, and the desired format — then produce either a finished prompt
or the completed task itself, depending on what you actually needed.

It can also be invoked explicitly as a command, passing the raw idea right
after it:

```
/prompt-builder help me write a prompt for a Python data-cleaning script
```

## License

Released under the [MIT License](./LICENSE).
