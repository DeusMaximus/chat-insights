---
name: chat-insights
description: Generate a usage insights report by analysing recent claude.ai conversations. Trigger when the user asks to "run insights", "generate a usage report", "analyse my recent conversations", or similar. Produces a visual HTML report covering what's working, friction points, topic breakdown, and quick wins — modelled on Claude Code's /insights command but for the claude.ai web interface. Do NOT use for general questions about conversations or memory.
metadata:
  author: DeusMaximus and Claude
  version: "1.0.0"
---

# Chat Insights Skill

Generate a usage insights report from recent claude.ai conversations, similar in spirit to Claude Code's `/insights` command. The output is a rendered HTML widget with metric cards, a what's working section, friction analysis (split by Claude-side and user-side), topic breakdown, and quick wins.

## Context and limitations

This skill uses the `recent_chats` and `conversation_search` tools, which return conversation snippets and summaries — not full transcripts. This means:

- Pattern detection is directional, not exhaustive
- Hallucination counts and friction examples are likely undercounts
- Coverage is limited to conversations the tools can surface (outside projects only if the user is outside a project)

Be explicit about these limitations in the report or as a note following it. Do not present findings with more confidence than the data warrants.

## Data collection strategy

### Step 1 — Pull recent conversations

Pull two batches using `recent_chats` (n=20 each), paginating with `before` set to the earliest `updated_at` from the first batch. This gives up to 40 conversations as the working dataset.

If the user specifies a timeframe ("last week", "this month"), use `before`/`after` parameters accordingly. Default is the most recent ~4–7 days.

### Step 2 — Targeted gap-filling (optional)

If obvious topic areas appear underrepresented based on available memory context or early patterns in the data, use `conversation_search` with 1–2 targeted queries to pull relevant conversations from those areas.

Do not run more than 3 total `conversation_search` calls. The point is to fill gaps, not to exhaustively catalogue everything.

### Step 3 — Synthesise before rendering

Read through all retrieved conversations before writing any output. Build a mental model of:

- Recurring topics and rough frequency
- Patterns in how the user opens conversations (with context vs. without)
- Sessions where Claude made errors the user caught
- Sessions that ended with an open loop or deferred decision
- MCP tools actively used
- What the user used Claude for that wasn't purely technical (sanity checks, second opinions, creative/personal topics)

Only then write the report.

## Synthesis framework

### What's working
2–4 items. Focus on interaction patterns and behaviours that are genuinely effective — not generic praise. Examples: efficient use of a specific MCP tool, a consistent correction style, strong contextual memory between sessions, effective use of Claude for a particular task type.

### What's hindering you
Split into two columns:

**Claude-side friction** — Hallucinations, reflex over-answering, wrong defaults, failure to check docs or tools before answering. Use specific examples from the retrieved conversations where possible.

**User-side friction** — Not enough context at session open, repeated open loops, decisions deferred despite having all the information needed, conversations that trailed off without resolution.

Be direct. These sections are most useful when specific and honest, not hedged.

### Topic breakdown
Categorise sessions into ~6–8 topic areas with rough counts. Use badges. Derive the categories from the actual conversation mix — do not force sessions into predetermined categories. "Misc" is fine for outliers.

Assign badge colours to encode meaning rather than sequence. Suggested mappings as a starting point: blue for technical/work topics, teal for infrastructure/homelab, purple for hardware/purchasing, coral for security or scam detection, amber for AI/meta topics. Adjust freely based on what the user actually talks about.

### Quick wins
2–3 specific, actionable suggestions. These should be things the user could actually do in the next week, not general advice. Reference specific open loops or patterns from the session data where possible.

## Visual output

Use `visualize:read_me` (modules: `["data_viz", "chart"]`) then render with `visualize:show_widget`.

### Metric cards (top of report)
Display 4 cards in a grid. Default suggestions:
- Conversations (count)
- Non-coding topics (% of sessions)
- MCP tools used (count, list key ones as subtitle)
- Hallucinations caught (count, "by you" as subtitle)

Adjust these if the data suggests different metrics are more meaningful for this particular user's conversation mix.

### Sections
Render in order: metric cards → what's working → what's hindering → topic breakdown → quick wins.

Use the CSS variable system for all colours throughout. Do not hardcode light/dark mode values.

## Output notes

After the widget, add a brief plain-text note covering:
- Approximate date range and conversation count analysed
- Any obvious gaps in coverage (e.g. Claude Code sessions don't appear here — those are best covered by Claude Code's native `/insights` command)
- Whether context limits affected the analysis

Keep this note short — 2–4 sentences.

## Known tool quirks

- `recent_chats` returns up to 20 results per call; paginate with `before` parameter set to the earliest `updated_at` from the prior batch
- Some conversations appear as summaries only (no snippet content) — treat these as partial data points
- Conversations inside Claude Projects are not searchable from outside a project context; the report will only cover the scope the user is currently in
- `conversation_search` queries should use content nouns from the actual conversations, not meta-words like "discussed" or "talked about" — the tool does text matching, not semantic search
