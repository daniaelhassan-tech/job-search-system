# Job Search System — Skill Entry Point

<!-- This file makes the system work as a Claude Code / Cowork skill. -->
<!-- Drop this entire folder into .claude/skills/ to activate it. -->

## Setup

This skill manages a structured job search using four interconnected markdown files. Before first use:

1. Copy the files from `templates/` into your working directory (or the folder you've selected in Cowork)
2. Customize `job-search-lens.md` with your own evaluation criteria, viability threshold, comp targets, and constraints
3. Clear the example data from the other three files and start adding your own companies
4. If using Google Workspace + Granola, append the contents of `instructions/google-workspace-config.md` to your project instructions

## On Every Session

**Always load `job-search-state.md` first.** It's the index into the entire system.

Then load whichever additional files the task requires:
- **Evaluating or comparing opportunities** → also load `job-search-lens.md`
- **Prepping for a specific call** → also load the relevant section of `job-search-dossiers.md` and `job-search-insights.md`
- **Debriefing a call** → also load `job-search-dossiers.md` and `job-search-insights.md` for that company
- **General check-in or sync** → scan all four files for aging items

## Behavioral Instructions

Follow the instructions in `instructions/base-instructions.md`. Key behaviors:

1. **Be a critical collaborator.** Push back on reasoning. Flag contradictions between the lens (what the user says they want) and the state (what they're spending energy on).
2. **Update the system, don't just summarize.** Every substantive conversation should produce proposed updates to at least one file.
3. **Viability threshold first.** Run every opportunity against the threshold before evaluating whether it's exciting.
4. **Track aging items.** Open threads >1 week old, drafts sitting unsent, pipeline entries with no activity in 2+ weeks — flag all of them.
5. **Trigger post-call debrief.** When a call just ended, don't wait — prompt the user to debrief and draft updates.

## File Architecture

```
job-search-state.md      ← Pipeline status, actions, deadlines (the "what's happening")
job-search-lens.md       ← Evaluation framework, constraints, tensions (the "how I think")
job-search-dossiers.md   ← Structured facts per company (the "what I know")
job-search-insights.md   ← Per-conversation narrative + interpretation (the "what I've learned")
```

State is the entry point. Lens is the decision framework. Dossiers are facts. Insights are narrative. They reference each other but serve distinct purposes — don't collapse them into one file.
