# chat-insights

A [claude.ai](https://claude.ai) skill that generates a visual usage report from recent conversations — what's working, where you and Claude get stuck, what you're actually spending time on, and a few concrete things to try next week.

Modelled on Claude Code's `/insights` command, but for the claude.ai web interface.

> **Note:** This is a skill for claude.ai, **not** for Claude Code. It runs inside a conversation on the web app using the `recent_chats`, `conversation_search`, and `visualize` tools.

## What you get

A rendered HTML widget with:

- **Metric cards** — conversation count, non-coding topic share, MCP tools used, hallucinations you caught
- **What's working** — specific interaction patterns that are genuinely effective
- **What's hindering you** — split into Claude-side friction (hallucinations, wrong defaults, reflex answers) and user-side friction (weak session openings, open loops, deferred decisions)
- **Topic breakdown** — badge-coloured categories derived from your actual conversation mix
- **Quick wins** — 2–3 concrete things to try in the next week

Followed by a short plain-text note on date range, coverage gaps, and any context limits that affected the analysis.

## Installing

1. Download or clone this repo.
2. In claude.ai, upload the `chat-insights/` folder as a skill via Settings → Capabilities → Skills.
3. The skill activates automatically when you ask for it (see triggers below).

## Using it

Trigger phrases:

- "run insights"
- "generate a usage report"
- "analyse my recent conversations"
- "chat insights"

Optionally specify a timeframe:

- "run insights for last week"
- "usage report for this month"

Without a timeframe, the skill analyses the most recent ~40 conversations.

## Limitations

The underlying tools return snippets and summaries, not full transcripts. So:

- Pattern detection is directional, not exhaustive
- Hallucination counts and friction examples are lower bounds
- Only conversations in the current scope are visible — from outside a Project, conversations inside Projects aren't searchable, and vice versa
- Claude Code sessions don't show up here; use Claude Code's native `/insights` for those

The skill will tell you when the dataset is too thin (fewer than ~10 usable conversations) rather than produce a low-signal report.

## Structure

```
chat-insights/
  SKILL.md     # skill definition + synthesis framework
README.md      # this file
```

## Author

DeusMaximus and Claude.

## License

MIT — see [LICENSE](LICENSE).
