---
name: prompt-builder
description: >
  Guides the user step by step to build a high-quality prompt using the
  formula role + context + task + audience + constraints + format,
  by asking short interactive questions, one at a time, that always
  include a recommended option and a free-text "other" field. Infers an
  expert role from the task plus the detected technology/stack, even when
  the user never states it. Use this skill whenever the user's request is
  vague, ambiguous, overly broad, rambling, or under-specified; whenever
  the user asks to write, refine, improve, or brainstorm a prompt (for
  Claude itself or for another AI/model); whenever the user explicitly
  asks to be asked questions in order to build a correct prompt; or
  whenever the user's message contains the word "prompt" in a request
  sense (not just mentioning it in passing). Can also be invoked
  explicitly as a command with text right after it (e.g.
  `/prompt-builder <text>`) — that text is taken as the raw idea/prompt/
  instruction to refine.
---

# Prompt Builder

Turn a vague or messy request into a sharp prompt by walking through six
components, one question at a time, always recommending an answer.

## The formula

A good prompt has six parts. Each one closes a gap that causes bad output
if left implicit:

1. **Role** — the expert persona to adopt. Derived automatically from the
   task + the detected technology, not asked as an open question (see
   "Inferring the role" below).
2. **Context** — the subject/background the request lives in.
3. **Task** — the concrete action to perform.
4. **Audience** — who the output is for (affects tone, depth, jargon).
5. **Constraints** — scope limits, things to avoid, constraints.
6. **Format** — the shape the output should take.

Skipping any of these is exactly what makes prompts vague: a task with no
role produces generic output, no audience produces the wrong tone, no
constraints lets the model wander, no format forces you to reformat the
answer yourself.

## Step 0 — Detect the mode

Before asking anything, form a working hypothesis about what happens with
the finished prompt:

- **Draft mode**: the user wants the prompt *itself* — to paste into
  another AI, save it, or hand it to Claude later. Signals: "help me put
  together a prompt for...", "write me a prompt", "I want to refine this
  prompt", mentions of another model/tool, or explicit requests for a
  prompt as a deliverable.
- **Execute mode**: the user's own request was the vague thing, and once
  it's clarified they want the task actually *done*, not a prompt printed
  back at them. Signals: a task phrased directly ("build me...", "help me
  with...") with no mention of "prompt" as a deliverable, especially when
  the request is just messy/rambling rather than meta about prompting.

If genuinely unclear after reading the request, don't guess — resolve it
with the question in Step 3.

## Step 0.5 — Initial argument (command invocation)

This skill can also be triggered explicitly as a command, with text right
after it, e.g. `/prompt-builder <text>` on Claude Code — the exact prefix
and syntax depend on the host agent. This exists mainly as a fallback for
when automatic triggering doesn't kick in — the user can force it this
way. It's an **additional, optional** entry point; it doesn't change
anything about the flow below.

- If the skill was invoked with extra text after the command, treat that
  text as **the raw initial request** — the idea, prompt, or instruction
  the user wants refined.
- Feed that text straight into Step 1 (read it before asking) and
  continue with the normal flow from Step 2 onward, unchanged.
- Also use that text as a signal for Step 0 (detecting draft vs. execute
  mode) when it makes the mode clear on its own.
- If the skill was triggered automatically instead (no argument), this
  step is a no-op — everything works exactly as it does today.

## Step 1 — Read before asking

Read the raw request first and mentally fill in whatever it already
makes unambiguous. Only ask about the components that are actually
missing or unclear. Interrogating the user on all six components when
three are already obvious defeats the purpose — it recreates the same
friction this skill exists to remove. A request can be vague in one
dimension (e.g., no clear audience) while being crystal clear in another
(e.g., the task itself).

## Step 2 — Ask one component at a time

Ask one question per component, in this order:

**context → task → role (see below) → audience → constraints → format**

Role comes right after task because it depends on both the task and the
context already gathered.

**How to ask:** if your host agent exposes a structured multi-choice
question tool (e.g. `AskUserQuestion` in Claude Code), use it — it gives
the user clickable options plus a built-in free-text field. If no such
tool is available, ask in plain numbered text instead (e.g. "1) ... 2)
... 3) ...") and tell the user they can reply with a number or write
their own answer. Same content and rules either way — only the delivery
mechanism changes.

For every question:

- Generate 2–4 options **specific to this actual request** — never
  generic filler like "Option A" / "Option B". Pull real, concrete
  possibilities out of the request and its context.
- **Always recommend one.** For single-select, put the recommended
  option first and append `(Recommended)` to its label. For
  multi-select, append `(Recommended)` to every option you'd pick.
  Explain the reasoning briefly in that option's description.
  Recommending isn't optional — the user asked for this because
  choosing well without help is exactly what's hard right now.
- Treat components as multi-select when more than one answer can
  reasonably apply at once (constraints and format often do;
  context/task usually don't) — with a structured tool, set
  `multiSelect: true`; in plain text, invite the user to pick multiple
  numbers.
- Don't manually add an "other" option — a structured tool already gives
  the user a free-text field automatically; in plain text, the open
  invitation to "or write your own" already covers it.
- Skip the question entirely for any component Step 1 already resolved.

## Inferring the role

The role is not a generic open question — it's a derived, composite
persona: **domain expertise implied by the task + the specific
technology/stack in play**, confirmed rather than silently injected.

1. **Detect the technology.** Look at the actual project: file
   extensions, config files (`package.json`, `astro.config.*`,
   `requirements.txt`, etc.), open files, or anything said earlier in
   the conversation. Only if none of that yields an answer, ask the user
   directly what stack/technology applies before moving on.
2. **Compose the role** from task-domain + technology. The pairing is
   what makes the role useful instead of generic:
   - Design a card component → *expert designer and web/mobile layout
     specialist*.
   - Decide a software/database architecture → *experienced software
     architect and DBA*.
   - Design a hero section on an Astro site → *graphic designer expert
     in web pages and hero sections, and expert Astro developer*.
   - Optimize SEO on that same Astro site → *SEO expert with Astro*.
   The pattern: don't stop at "designer" or "architect" — name the
   specific technology so the role actually constrains how the model
   thinks, not just who it pretends to be.
3. **Confirm it, don't assume it.** Ask a single-select question (see
   "How to ask" in Step 2) with the derived role first (labeled
   `(Recommended)`), 1–3 sensible alternatives, and let the free-text
   field — or the open invitation, in plain text — cover anything you
   got wrong.

## Step 3 — Resolve the mode if still ambiguous

If Step 0 didn't settle it, ask one last single-select question: "What
do you want me to do with this?" with options along the lines of "Write
the prompt for me to use" (draft) vs. "Use this and do the task
yourself" (execute).

## Step 4a — Output for draft mode

Produce one markdown block, ready to copy, role first:

```
## Role
## Context
## Task
## Audience
## Constraints
## Output format
```

No filler — this is the deliverable, not a recap of the conversation.

## Step 4b — Output for execute mode

Don't print any markdown prompt block. Adopt the confirmed role and use
everything gathered to just do the task directly.

## Ground rules for every question

- Every question needs a recommendation — never leave the user guessing
  which option is safest.
- Options must be concrete and mutually distinct, grounded in this
  specific request — not padding to reach a count.
- Respect what Step 1 already resolved; don't re-ask it "just in case."

## Example

Vague request in an Astro project: *"help me with my site's hero, I
don't even know where to start"*.

1. Context — already fairly clear (a hero section, an Astro site) → skip
   or confirm briefly.
2. Task — ask what "help" means concretely: redesign, first draft copy,
   layout, animation... recommend based on what's likely missing.
3. Role — detect Astro from the project, propose *"graphic designer
   expert in web pages and hero sections, and expert Astro developer"
   (Recommended)*, confirm.
4. Audience — ask who visits this site.
5. Constraints — ask about brand constraints, existing components to
   reuse, things to avoid.
6. Format — ask whether they want code, a written brief, or both.

Then, if the mode turned out to be *execute*, adopt the confirmed role
and build the hero directly — no markdown block. If *draft*, hand back
the six-section markdown prompt, ready to paste elsewhere.
