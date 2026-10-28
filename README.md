# MATACS-TH-2026-02: AI Proposal Triage Design

- **Name:** Laiba Idrees
- **Reference:** MATACS-TH-2026-02
- **Time actually spent:** Around 70–80 minutes

## Files

| File | Contents |
|---|---|
| [diagram/process_flow.png](diagram/process_flow.png) | Process flow from proposal upload to the final ranked output, showing where AI, code and human review are used |
| [WRITEUP.md](WRITEUP.md) | Design decisions, consistency and explainability, human review, changing criteria, risks, sample results and assumptions |
| [PROMPTS.md](PROMPTS.md) | The extraction prompt and how the application handles the AI output |
| [ranking.json](ranking.json) | Expected ranking and results for proposals A–E |

## Design in one line

AI reads each proposal and extracts facts with evidence. Code handles the mandatory checks, scoring and ranking. A human reviews the results, can override them with a reason, and makes the final decision.

## How I used AI tools

I used Claude to help me understand the brief, draft the extraction prompt, check the scoring maths, and write first drafts of the write-up and diagram, and ChatGPT to review and refine the diagram. I disagreed with the first diagram Claude produced: it used technical terms like "schema", "injection scan" and "PDF hash", and had a separate "Admin criteria" box with dashed arrows that made the flow harder to follow. Since the reviewer and award decision-maker are not necessarily developers, I rewrote the steps in plain language, moved the criteria into a short note on the scoring step, made the three outcomes (ranked, flagged, ineligible) visually distinct, and kept the technical detail in WRITEUP.md instead.