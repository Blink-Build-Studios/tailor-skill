# Authoring the activation prompt

The activation's `prompt_content` is the instruction the agent runs on each schedule. **You write it** from what the user described. They never write it and never approve the wording — they approve the *result* of a test run. This file is how you write a good one.

## Principles

- **The prompt is a task brief for an autonomous run, not a chat message.** It executes unattended on a schedule, with only the tools in its permission set. Write it so a capable agent could run it start-to-finish with no follow-up questions.
- **State the task, the sources, the destination, and the output shape — plainly.** Ambiguity produces bad runs.
- **Only reference capabilities in the permission set.** Don't instruct it to do things its tools can't (you chose the scope; keep the prompt inside it).
- **Prefer concrete over clever.** No role-play, no "you are a world-class…" fluff. Say what to do.
- **Make the destination explicit.** "Post the summary to the user's Slack DM" beats "share the summary."
- **Bound it.** If it summarizes, say how long. If it lists, say the order. If there's nothing to report, say what to do (e.g. "if there was no activity, send 'Quiet week — nothing to report.'").

## A reliable structure

```
<One sentence: what this run does.>

Steps:
1. <Read step — which source, what to pull, what window (e.g. "the last 7 days").>
2. <Process step — summarize / filter / rank, with any format rules.>
3. <Deliver step — exact destination and format.>

If <empty/edge case>, <what to do instead>.
Keep <the output constraint: length, tone, ordering>.
```

## Example

User said: *"Every Friday, summarize what happened in Linear this week and DM it to me on Slack. Lead with anything blocked."*

Authored `prompt_content`:

```
Produce a weekly engineering summary from Linear and deliver it to the user via Slack DM.

Steps:
1. Pull all Linear issues updated in the last 7 days.
2. Write a concise summary grouped as: Blocked (first), In progress, Shipped this week. Within each group, one bullet per issue: title + one-line status.
3. Send the summary as a direct message to the user on Slack.

If there was no Linear activity in the last 7 days, send: "Quiet week in Linear — nothing to report."
Keep it under ~15 bullets and lead with the Blocked section so it's the first thing the user sees.
```

Notice: it names the source (Linear), the window (7 days), the ordering rule (Blocked first — the thing the user emphasized), the destination (Slack DM), the empty-case behavior, and a length bound. All derived from one plain-language request.

## What to show the user (not the prompt)

Show a **one-line plain-English summary**, not the prompt text:

> "Every Friday at 9am I'll pull the week's Linear activity, summarize it with blockers up top, and DM it to you on Slack."

Then let the **test run** be what they actually judge. If the test output is wrong, revise the `prompt_content` (via `update_activation`) and re-run — don't debate prompt wording with them.

## Revising from test feedback

Map their feedback to a prompt edit, then re-run:
- "Too long" → add/tighten the length bound.
- "Lead with X" → reorder the steps / add an ordering rule.
- "Wrong channel / wrong person" → fix the destination line (and check the permission set has the right tool).
- "It missed Y" → widen the read step (window, filter, or an additional source — which may mean adding a tool to the permission set first).
