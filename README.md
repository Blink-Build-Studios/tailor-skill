# Tailor — Stitchwork skill for Claude

**Tailor** lets you build and run [Stitchwork](https://app.stitchwork.ai) automations by conversation. You describe what you want automated; Tailor drives the Stitchwork MCP tools to set it up — connections, a least-privilege permission set, an agent, and a scheduled activation — so you never have to touch permission sets, RRULEs, tool scopes, or prompts by hand.

This repo is a [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces) that distributes the Tailor skill. The skill is generated from the Stitchwork source repo and kept in lockstep with the live tool surface.

## Install

Tailor is **two pieces**: the MCP connection (gives Claude the tools) and this skill (teaches Claude how to drive them). You need both.

**1. Connect Claude to Stitchwork** (one time):

```
claude mcp add --transport http tailor https://app.stitchwork.ai/mcp
```

Approve the connection in your browser when prompted.

**2. Install the Tailor skill:**

```
/plugin marketplace add Blink-Build-Studios/tailor-skill
/plugin install tailor@tailor
```

Then just tell Claude what you want to automate — e.g. *"Have Tailor send me a weekly Linear summary on Slack every Friday."*

## Updating

The skill auto-updates: run `/plugin marketplace update` to pull the latest. Every change to the skill is published here automatically.

## Source of truth

The skill is authored in the Stitchwork product repo (`.claude/skills/tailor/`) and synced here on every change, so it always matches the deployed MCP tool surface. Don't edit the skill files here directly — edit them in the product repo.
