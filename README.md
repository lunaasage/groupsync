# GroupSync

**AI-Powered Group Chat Planning Engine**

GroupSync is an LLM-powered chat extension that reads a friend group's conversation and automates the coordination work nobody wants to do. The AI parses chat context — scheduling signals, budget cues, food preferences, veto signals — and uses that understanding to pre-populate a planning dashboard, run swipe-to-vote activity selection, and surface smart venue suggestions with real-time deals, all grounded in what the group is actually saying.

## The Problem

Every friend group has a "Bea" — the one who ends up organizing plans in the group chat. Scheduling alone eats 30+ messages, nobody agrees on what to do, and by the time something's picked, nobody's checked deals or budget fit. The planner burns out, the group hits decision fatigue, and half the time everyone defaults to "nvm I'll just stay in."

The insight: the information needed to make a plan **already exists in the conversation**. People say when they're free, what they're craving, what they can afford. Nobody's synthesizing it. The job-to-be-done isn't "create a poll" — it's "resolve group indecision without killing the vibe."

## How It Works

GroupSync operates as a chat-native widget (WhatsApp/iMessage) with a three-stage funnel:

**When** (availability poll) &rarr; **What** (swipe-to-vote on activity type) &rarr; **Where** (smart suggestions with real-time deals)

Only the planner installs it. Everyone else participates inline — no downloads, no signups. The AI reads the group chat and pre-populates each stage based on what the group is already saying.

## Architecture

The system has three AI components working in sequence:

**Trigger Detection** — An LLM classifier monitors the chat stream and decides when to surface the planning widget. Uses prompt-based intent classification with few-shot examples to distinguish genuine planning intent from casual conversation, with a confidence threshold to minimize false positives.

**Context Extraction Engine** — Given a raw chat transcript (last 50–100 messages), a single-pass LLM call with chain-of-thought reasoning extracts a structured group profile: available dates, budget signals, food/activity preferences, vetoes, urgency level, and group size. Every extracted field links back to the source message for transparency. Built with Claude API + Pydantic schema enforcement.

**Suggestion Ranker (RAG Pipeline)** — The extracted group profile is embedded and used to retrieve the top venue candidates from a pgvector store (~50 Miami venues). An LLM reranks the candidates against the full group profile, explaining the match per venue, and injects simulated deal data. Built with LangChain + Supabase pgvector.

| Layer | Component | Technology |
|---|---|---|
| Intelligence | Chat trigger detector | Claude API + prompt engineering |
| Intelligence | Context extraction engine | Claude API + Pydantic structured output |
| Intelligence | Suggestion ranker | LangChain + pgvector + LLM reranking |
| Data | Venue + deals store | Supabase (PostgreSQL + pgvector) |
| Data | Chat transcript store | Curated scenario transcripts |
| UI | Simulated WhatsApp shell | HTML/CSS/JS prototype |
| Eval | Extraction + suggestion quality | Custom scorers + RAGAS |

## Evaluation Strategy

GroupSync has three separate quantitative evaluation layers — extraction accuracy (per-field precision/recall against hand-labeled ground truth across 10 curated chat transcript scenarios), trigger classifier performance (precision, recall, F1 on a 30-window labeled dataset), and suggestion quality (RAGAS faithfulness + answer relevancy).

The 10 chat transcripts were designed to exercise the full edge case space: the easy group, the broke friend, the group that never agrees, the last-minute plan, the quiet group with incomplete signals, mixed/contradictory preferences, and negative cases where the trigger should not fire.

## Live Prototype

The interactive HTML prototype simulates the full GroupSync user flow inside a WhatsApp-style interface. Open `prototype/index.html` in any browser to walk through:

1. **Group Chat** — A realistic friend group conversation with embedded planning signals
2. **AI Dashboard** — Bea's planning panel pre-populated with extracted dates, preferences, budget signals, and activity cues
3. **Availability Poll** — Inline date voting with AI-detected conflicts highlighted
4. **Swipe Cards** — Tinder-style activity voting (food, drinks, adventure)
5. **Venue Suggestions** — Ranked recommendations with deal badges and match explanations

## Project Structure

```
groupsync/
├── README.md
├── prototype/
│   └── index.html                    # Interactive UI prototype (open in browser)
└── docs/
    ├── prototype_description.pdf     # Product concept & design rationale (BTE 646)
    └── sprint_plan.docx              # Technical roadmap & sprint breakdown (MAS 691)
```

## Tech Stack

- **LLM** — Claude (Anthropic API) for trigger detection, context extraction, and suggestion reranking
- **Embeddings** — OpenAI text-embedding-3-small for venue semantic retrieval
- **Database** — Supabase (PostgreSQL + pgvector) for venues, transcripts, and vector search
- **Orchestration** — LangChain for retrieval chain and prompt management
- **Schema** — Pydantic for structured output enforcement
- **Frontend** — HTML/CSS/JS prototype simulating WhatsApp chat-native widget
- **Evaluation** — RAGAS + custom scorers for extraction accuracy and suggestion quality

## Key Differentiators

Doodle does scheduling. Yelp does recommendations. Groupon does deals. Nothing ties all three together where the conversation already happens — and nothing starts from what the group is already saying.

- **AI grounded in chat context** — reads preferences, budget cues, and scheduling signals from the actual conversation rather than starting from scratch
- **Chat-native** — only the planner installs; everyone else participates inline inside their existing messaging app
- **Source attribution** — every AI decision links back to the specific message that drove it, building user trust
- **Swipe-based voting** — fast, low-friction group decision-making inspired by familiar interaction patterns
