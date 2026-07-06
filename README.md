# briefing-card

**English** · [中文](README.zh-CN.md) · [日本語](README.ja.md)

A Claude Code plugin: chat-rendered **decision / progress / confirmation cards** (with clickable buttons), plus a **T0–T3 reporting-granularity router** that decides, every turn, whether a status or decision should be plain text, a quick question, an interactive card, or an HTML file.

## Why a plugin, not just a skill

A skill's `description` reliably fires only on *explicit* triggers — you asking for a card. It can't carry *ambient* auto-triggering: "draw a card whenever you're about to report a decision or progress." That needs a standing instruction resident in context every turn. A bare skill folder doesn't ship one, so on a fresh machine it feels like nothing installed.

This plugin fixes that with a **SessionStart hook** (`hooks/session-start`) that injects the routing rules as resident context on every session start / clear / compact. Install and it just works — no editing your `CLAUDE.md`.

## Install

In Claude Code (not your shell):

```
/plugin marketplace add vincent-wen789/claude-briefing-cards
/plugin install briefing-card@vincent-plugins
```

Then **start a new session** — the hook injects at session start, so it won't affect the session you installed from.

## Verify

- **Quick:** in a new session, ask *"Do you have standing T0–T3 briefing-card instructions?"* If it recites the routing, the hook is live.
- **Real:** without asking for a card, have it give you a multi-line progress report or two options with trade-offs. It should auto-draw a card / route, not dump plain text.

## Structure

```
.claude-plugin/plugin.json        # plugin manifest
.claude-plugin/marketplace.json   # marketplace entry (source: "./")
hooks/hooks.json                  # SessionStart → run session-start
hooks/session-start               # injects the resident routing context
hooks/session-context.md          # the routing block (edit behavior here)
skills/briefing-card/             # the skill itself (templates + routing)
```

## Note

This is a **personal edition** — the voice, trigger words, and the "default to a card" bias are tuned for one user and written in Chinese. To ship it for strangers, de-personalize the voice and make the aggressive default opt-in (otherwise it over-draws cards).
