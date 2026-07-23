# Scheduling: plain English → RRULE

Activations run on a schedule expressed as an **RFC 5545 RRULE** plus an **IANA timezone**. You translate the user's plain-language timing into the RRULE — they never see or write one. Always confirm with `preview_schedule` before creating the activation.

## The timezone principle (do not violate)

**Never infer the timezone from a clock, locale, or the current time.** Timezone is a config value the user owns. Ask for it, or default to `America/Los_Angeles` and *tell them you did* so they can correct it:

> "I'll schedule that for 9am Pacific — is that your timezone, or should I use a different one?"

Pass it as `schedule_timezone` (IANA form, e.g. `America/New_York`, `Europe/London`, `America/Los_Angeles`). Common ones:
- US Eastern → `America/New_York`
- US Central → `America/Chicago`
- US Mountain → `America/Denver`
- US Pacific → `America/Los_Angeles`
- UK → `Europe/London`
- Central Europe → `Europe/Berlin`

## RRULE patterns (map the request, don't hand-roll from scratch)

RRULE fields you'll use: `FREQ` (DAILY/WEEKLY/MONTHLY), `BYDAY` (MO,TU,WE,TH,FR,SA,SU), `BYHOUR`, `BYMINUTE`, `INTERVAL`, `BYMONTHDAY`.

| User says | RRULE |
|---|---|
| "every morning at 8:30" | `FREQ=DAILY;BYHOUR=8;BYMINUTE=30` |
| "weekdays at 9am" | `FREQ=WEEKLY;BYDAY=MO,TU,WE,TH,FR;BYHOUR=9;BYMINUTE=0` |
| "every Friday first thing" (clarify the hour) | `FREQ=WEEKLY;BYDAY=FR;BYHOUR=9;BYMINUTE=0` |
| "every Monday and Thursday at 2pm" | `FREQ=WEEKLY;BYDAY=MO,TH;BYHOUR=14;BYMINUTE=0` |
| "the 1st of every month at 7am" | `FREQ=MONTHLY;BYMONTHDAY=1;BYHOUR=7;BYMINUTE=0` |
| "every two weeks on Wednesday" | `FREQ=WEEKLY;INTERVAL=2;BYDAY=WE;BYHOUR=9;BYMINUTE=0` |
| "once a day" (no time given → ask) | ask for the hour; don't guess |

Notes:
- Hours are 24-hour. "first thing" / "morning" is ambiguous — pick a sensible default (9am) but say it back so they can adjust.
- If they don't give a time, ask. A schedule with no `BYHOUR` runs at an unhelpful default.

## Always preview before creating

Once you have a candidate RRULE + timezone, call:

```
preview_schedule(rrule="FREQ=WEEKLY;BYDAY=FR;BYHOUR=9;BYMINUTE=0", timezone="America/Los_Angeles")
```

It returns a plain-English `description` and the next `occurrences` (each with a human `label`). Read those back:

> "Got it — that'll run **next Friday Jul 24 at 9:00am Pacific**, then Jul 31, then Aug 7. Good?"

Only call `create_activation` after they confirm. If the RRULE is malformed, `preview_schedule` returns an error — fix the rule and preview again; never pass an unvalidated RRULE straight into `create_activation`.

## Event triggers (reactions), not just schedules

The product's vision of an "agent" includes **reactions to events** ("whenever an email comes in, …"), not only schedules. In this version, `create_activation` takes a **schedule RRULE**. If the user wants a pure event-trigger and the tool surface you have doesn't expose a trigger mechanism:
- Say plainly that scheduled runs are supported today and event-triggers are coming.
- If a frequent schedule is an acceptable stand-in (e.g. "check every 15 minutes"), offer it: `FREQ=MINUTELY;INTERVAL=15` — but flag the tradeoff (it polls rather than reacts instantly).
- Otherwise capture it as a feature request (`capture_feature_request`) so the team sees the demand.
