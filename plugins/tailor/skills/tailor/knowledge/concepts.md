# Stitchwork concepts & build order

The four building blocks, and why they compose in this order. Understand these so you can explain them in plain language and never build them out of order.

## The pieces

- **Connection** — an authenticated link to an upstream service (Slack, Linear, Gmail, GitHub, Notion, …). It's what gives Stitchwork *tools* to use. One connection can be reused across many permission sets and automations. A connection has a health status (`healthy`, degraded, disconnected) and may need re-authentication.

- **Tool** — a single capability a connection exposes (e.g. Slack's "post message", Linear's "list issues"). Every tool carries risk flags: `read_only`, `destructive`, `idempotent`, `open_world`. You use these to pick least-privilege scopes. A single provider can expose *dozens to hundreds* of tools — never grant them all; grant only what the workflow needs.

- **Permission set** — a named bundle of `(connection, [specific tools])` rows. This is the security boundary: an automation can only ever see and call the tools in its permission set. It literally cannot touch anything else. Edit the set once and every automation using it picks up the change (scope is inherited live). This is the whole value proposition — the automation "cannot knock anything over."

- **Agent** — a named container that groups automations, like a teammate with a job ("Executive Assistant", "Marketing"). Automations live inside an agent. Personal by default.

- **Activation** — one automation: a prompt + a schedule (RRULE + timezone) + a permission-set scope, inside an agent. This is the actual recurring/triggered task. (In the codebase an activation is sometimes called an "automation"; to the user it's just "the automation" or "the thing that runs".)

## Build order (always this sequence)

```
Connections (healthy)  →  Permission set (least-privilege scope over those connections)
       →  Agent (container)  →  Activation (prompt + schedule, bound to the permission set)
       →  Test run  →  iterate  →  done
```

Why the order is forced:
- You can't build a **permission set** until the **connections** it scopes are healthy (you need to list their tools).
- You can't create an **activation** until both its **agent** (container) and its **permission set** (scope) exist — `create_activation` needs both GUIDs.
- You always **test** before declaring done, because the user judges the result, not the config.

## GUIDs, not names

Every tool refers to things by GUID (`agent_guid`, `permission_set_guid`, `activation_guid`, `connection_guid`), never by internal integer id and usually not by name. When you create something, hold onto the GUID it returns — you'll need it downstream. The user never sees a GUID; you translate to plain language ("your Executive Assistant agent").

## One org per session

Everything you create lands in the user's currently-selected organization — the one named on the OAuth consent screen when they connected. There is no in-conversation org switching in this version. If the user needs to work in a different org, they switch it in the Stitchwork web app and reconnect (which mints a token bound to the new org).
