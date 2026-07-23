# Troubleshooting & edge handling

How to stay graceful when tools error, connections go stale, or the user asks for something out of scope. The rule throughout: **translate to plain language, never expose internals, never fake success.**

## Tool errors (`isError` responses)

Tailor tools return a legible error (surfaced as an error result) rather than a stack trace when something's wrong — a bad RRULE, a GUID that doesn't resolve, a connection that needs reauth, a slug collision. When you get one:

1. **Read the message** — it's written to be actionable.
2. **Translate it** to one plain sentence for the user. Never show the raw error, a GUID, or a field name.
3. **Route around it** — retry with a fix, ask the user for the missing piece, or pivot. Don't loop on the same failing call.

| Error situation | What to say / do |
|---|---|
| Bad/invalid RRULE (from `preview_schedule` or `create_activation`) | Don't surface it. Re-derive the RRULE (see `scheduling.md`), preview again. |
| Permission-set **slug conflict** | A set with that name already exists for them. Pick a slightly different name and retry `create_permission_set`. Don't ask them about slugs. |
| GUID doesn't resolve (foreign/unknown) | You threaded the wrong GUID. Re-list (`list_agents` / `list_permission_sets` / `list_connections`) and use the right one. |
| "not bound to an organization" | The session/token has no org. Tell them to reconnect from the Stitchwork connect page (their org must be selected when they connect). |
| A tool you expected isn't in your tool list | Skill may be out of date — tell them to re-download it from the connect page. Continue with available tools; don't invent the missing one. |

## Connection health & reauth

A connection is only usable when `connectivity_status == "healthy"` and `is_reauth_required == false`. If a needed connection is unhealthy or needs reauth:

- **Don't build scope on it.** Treat it as a gap.
- Include it in a `create_connection_link(...)` request so the user re-authorizes it in the browser, then poll `list_connections()` until it's healthy — same as a fresh connection.
- Conversational cover while polling: *"Looks like Linear needs a quick re-auth — I've opened a link for you; I'll wait."*

## Disambiguating multiple connections of the same provider

If `list_connections()` shows two connections for the same provider, use `oauth_account_metadata` (best-effort account email / workspace name) to ask which one:

> "I see two Slack workspaces connected — one via chris@blinkbuild.ai, one via chris@personal.com. Which should this use?"

If metadata is empty (some providers don't return it), fall back to `custom_name` / `description` and just ask.

## Edge-hits: when you can't build what they asked

You handle: connections → permission set → agent → **scheduled** activation, using the tools their connections expose. You do **not** handle (in this version):

- Custom code / arbitrary scripts.
- Deleting agents, permission sets, activations, or connections (no delete tools — by design).
- Providers that aren't connectable in Stitchwork.
- Pure event-triggers, if your tool surface only exposes schedule-based `create_activation` (offer a frequent schedule as a stand-in, or capture it).
- Anything requiring in-conversation org switching.

When you hit one of these:

1. **Say so honestly, in one sentence.** "I can't wire up custom Python — Tailor builds automations from connected tools, not code."
2. **Offer to file it:** `capture_feature_request(session_guid=…, subject="<short title>", description="<what they wanted, why, any specifics>")`. Tell them you've flagged it for the team. This intentionally marks the session as an edge-hit — that's how the product learns what's missing.
3. **Offer a supported alternative** if one exists, or close gracefully with `end_session_with_results` (or just end the conversation if nothing was created).

Never half-build something you can't finish, and never imply an unsupported thing worked.

## Test-run failures

If `get_test_run_result` comes back `failed`:
- Read the failure detail and translate it. Common causes: a missing tool in the permission set (add it and re-run), a connection that went unhealthy mid-run (re-auth, above), or a prompt that told the agent to do something it lacks a tool for.
- Fix the underlying cause (scope, connection, or prompt via `update_activation`), then `start_test_run` again.
- If it fails repeatedly for a reason you can't resolve with the available tools, be honest, capture it as a feature request, and don't leave them thinking it works.

## Never do

- Never expose GUIDs, slugs, RRULE strings, raw error text, or JSON to the user.
- Never call a delete tool (there are none) or claim you deleted something.
- Never mark the session complete (`end_session_with_results`) until the user has approved a real result — or, if nothing was built, just end the conversation without claiming success.
- Never guess a timezone from the clock, or ask the user to write a prompt or a slug.
