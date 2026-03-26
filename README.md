# Job Search System

A structured information architecture for managing a senior/executive job search with AI as a collaborative partner — not just a tool.

I built this system during my own job search for VP/Director-level product leadership roles, managing a pipeline of ~20 companies across big tech, growth-stage, and early-stage simultaneously. The system ran on Claude (Anthropic) using Cowork and Claude Code, with integrations into Gmail, Google Calendar, and Granola for meeting transcripts.

This repo contains the genericized, ready-to-use version. All personal data has been replaced with fictional examples that demonstrate how the system works in practice.

## Why This Exists

Networking, updating your criteria, scoring invididual companies based on incomplete information, updating resumes, finding new opportunities and remembering key insights or learnings - that's a full time job. Sales teams have CRMs, what do most professionals have at the time where they're reinventing themselves?

Most job search tools are trackers — spreadsheets and Kanban boards that tell you where things stand. They don't help you *think* about your search.

A senior job search isn't a funnel. It's a multi-threaded decision process where every conversation changes the information landscape, evaluation criteria evolve as you learn, and the hardest part isn't tracking status — it's maintaining strategic coherence across 10+ simultaneous conversations while managing your own biases.

I designed this system to solve three problems I kept running into:

**1. Context fragmentation.** After 20+ calls across 10 companies, I couldn't keep straight what I'd learned, what I'd said, what signals I'd observed, and what questions remained open. I had notes but they'd get wiped out every time I reset the context window — they didn't connect to each other or to my decision framework.

**2. Evaluation drift.** I'd get excited about a conversation and forget to check whether the opportunity actually met my constraints. Or I'd keep a dead pipeline entry alive because killing it felt like losing optionality. I needed a system that enforced my own evaluation discipline.

**3. AI as yes-man.** Generic AI assistance defaults to being agreeable — summarizing what you tell it, generating whatever you ask for. I needed an AI collaborator that would push back, flag contradictions, and catch patterns I was too close to see.

## Information Architecture

The system uses four markdown files, each with a distinct purpose and a defined relationship to the others:

```
┌──────────────────────────────────────────────────────────────┐
│                    job-search-state.md                        │
│              Pipeline status, actions, deadlines              │
│                    (the "what's happening")                   │
│                                                              │
│              *** ALWAYS LOAD THIS FIRST ***                  │
└──────────┬────────────────────┬──────────────────────────────┘
           │                    │
           ▼                    ▼
┌─────────────────────┐  ┌─────────────────────────────────────┐
│  job-search-lens.md │  │      job-search-dossiers.md         │
│  Evaluation criteria│  │      Structured facts per company   │
│  Viability threshold│  │      (the "what I know")            │
│  Financial targets  │  │                                     │
│  Tensions & biases  │  │                                     │
│  (the "how I think")│  │                                     │
└─────────────────────┘  └──────────────┬──────────────────────┘
                                        │
                                        ▼
                         ┌──────────────────────────────────────┐
                         │     job-search-insights.md           │
                         │     Per-conversation narrative        │
                         │     Signals, interpretation, threads  │
                         │     (the "what I've learned")         │
                         └──────────────────────────────────────┘
```

### Why four files instead of one?

When I realized that Claude was tuned to work for coders, I realized that I needed to pattern match to Object Oriented programming or a system of microservices or agents working together is the mental model that got me to access the right information for the right tasks. I also could be more token efficent, not burning huge amounts of tokens with heavier files overloaded with information and context. 

Each file has a different **update cadence** and a different **reader context**:

| File | Updated when... | Loaded when... |
|---|---|---|
| **state.md** | Any status change, new action, or deadline shift | Every session (it's the index) |
| **lens.md** | Rarely — when your criteria or constraints evolve | Evaluating opportunities, making decisions |
| **dossiers.md** | New facts surface (from calls, research, recruiter intel) | Prepping for a specific call |
| **insights.md** | After every meaningful interaction | Debriefing, prepping follow-ups, spotting patterns |

Collapsing them into one file creates a document that's either too long to scan quickly (you lose state.md's utility as an index) or too compressed to capture nuance (you lose insights.md's narrative value). The separation is intentional.

### Design decisions worth noting

**State as the entry point, not a database.** State.md is a lightweight index, not a CRM. It has just enough information to know what needs attention *right now*. Deep context lives in dossiers and insights, which get loaded selectively.

**Lens as a first-class file.** Most job search systems bury evaluation criteria in your head or in a static doc you wrote once. Making it a living file that the AI references on every evaluation forces you to be explicit about your constraints — and makes it much harder to rationalize past them.

**Facts vs. interpretation, separated.** Dossiers hold what you *know* (company raised $14M, team is 15 people, CTO is ex-Meta). Insights hold what you *think* (this feels like the most interesting role, but the comp gap is real). This separation matters because facts are durable and interpretation evolves. When you prep for call #3 with a company, you want the facts to be clean and the interpretation to show how your thinking developed over time.

**Insights as an append-only log.** New entries get appended, never edited. This preserves the evolution of your thinking — you can see what you thought after call #1 vs. call #3. The AI uses this history to flag when your current assessment contradicts what you observed earlier.

## AI Collaboration Design

The second design layer is how the AI interacts with the system of markdown files. This isn't just "use AI to help with my job search" — it's a set of behavioral instructions that shape the AI into a specific kind of collaborator. When you use this, you can paste these into the project instructions in Claude Cowork.

### Behavioral architecture

The AI instructions (`instructions/base-instructions.md`) define five core behaviors:

**1. Critical collaborator, not reference tool.** The AI is instructed to push back when reasoning is off, flag contradictions between stated preferences (lens.md) and actual behavior (state.md), and not just agree. This is the single most important instruction — without it, AI assistance degrades into expensive autocomplete.

**2. Update the system, don't just summarize.** After every substantive conversation, the AI proposes specific file updates — which file, which section, what changes. If the files aren't being updated, the system atrophies and becomes stale. This instruction keeps it alive.

**3. Viability threshold first.** Every opportunity gets checked against hard constraints (comp floor, location, benefits, positioning value) *before* the AI engages with whether it's exciting. This catches the most common failure mode in job searches: rationalizing past dealbreakers because the opportunity feels good.

**4. Aging tracker.** Open threads >1 week, unsent drafts, pipeline entries with no activity in 2+ weeks — all get flagged proactively. Stale items represent unacknowledged decisions. The AI surfaces them so you can decide consciously rather than by default.

**5. Post-call debrief protocol.** After detecting a call ended, the AI initiates a structured debrief: pull transcript if available, cross-reference with existing dossier, ask for reactions, then draft updates across all four files. This is the highest-value workflow in the system — it turns a 30-minute conversation into structured, searchable, connected knowledge within minutes.

### Integration layer

The base instructions are tool-agnostic. The `instructions/google-workspace-config.md` file adds specific protocols for Gmail (email scanning, draft tracking), Google Calendar (meeting detection, upcoming prep), and Granola (transcript extraction). This separation means you can swap tools without rewriting the core behavioral instructions.

## Using This System

### As a Claude skill (Claude Code / Cowork)

1. Copy the entire repo into your `.claude/skills/` directory (or your Cowork project folder)
2. Customize `templates/job-search-lens.md` with your own criteria
3. Clear example data from the other templates and start adding your companies
4. Add `instructions/base-instructions.md` to your Claude project instructions
5. Optionally append `instructions/google-workspace-config.md` if you use those tools

### As a standalone system (any AI assistant)

1. Copy the four template files into wherever you keep working documents
2. Customize the lens file
3. Paste the contents of `instructions/base-instructions.md` into your AI's system prompt or project instructions
4. Start conversations by asking the AI to load state.md first

### Customization guide

The system is designed to be personalized. Here's what to change:

| What | Where | Notes |
|---|---|---|
| Your profile | `instructions/base-instructions.md`, "Who You Are" section | Be specific — level, domain, target roles | Uploading your resume is a good kick start to this in your first conversation
| Evaluation criteria | `templates/job-search-lens.md`, "What I Evaluate" section | Delete/add lenses. These should be *your* dimensions, not generic ones | Tell the system to update your lens doc when you reach key conclusions.
| Viability threshold | `templates/job-search-lens.md`, "Viability Threshold" section | Set your actual constraints. The comp floor is a floor, not a wish | Note: I set up a similar system for financials to evaluate offers from startups and public companies side by side. If you're interested, I can publish this system next.
| Blind spots | `instructions/base-instructions.md`, "Watch for my blind spots" | Name your actual patterns. Common: over-weighting domain fit, under-scrutinizing financials, avoiding pipeline kills |
| Tool integrations | `instructions/google-workspace-config.md` | Swap for your tools or remove entirely |

## What Makes This a Product Artifact

This system is a product design exercise in miniature. The decisions embedded in it — information architecture, separation of concerns, state management, behavioral specification, protocol design — are the same decisions that go into designing any multi-user, multi-state, information-rich product.

If you're a PM looking at this repo, here's what I'd highlight:

**The IA problem is real.** A job search generates unstructured, multi-dimensional, time-varying information across dozens of entities. Designing a schema that stays useful as the search evolves — without becoming so rigid it fights you — is the same challenge as designing a data architecture for your product.

**The AI collaboration problem is underexplored.** Most AI-assisted workflows treat the AI as a tool that does what you ask. The more interesting design space is developing expertise and behavioral coaching collaborator — defining how an AI should interact with a human decision-maker over time, including when to push back, what to surface proactively, and how to maintain institutional memory across sessions.

**The protocol design matters more than the templates.** The debrief protocol, sync protocol, and viability-first evaluation flow are where the system's value lives. The templates are just containers. The protocols define how information flows through them. So I don't accidentally burn togents with a totally agentic system, and to maintain editorial control, I've set this up as manual push to update my files which is as simple as "update the state doc based on the emails I sent today." 

## License

MIT. Use it, fork it, customize it. If you build something interesting on top of it, I'd love to hear about it.
