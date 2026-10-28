# MATACS-TH-2026-02: AI Proposal Triage Design

- **Name:** Laiba Idrees
- **Reference:** MATACS-TH-2026-02
- **Time actually spent:** _[would add later]_

## Files

| File | Contents |
|---|---|
| [diagram/process-flow.png](diagram/process-flow.png) | Process flow from proposal upload to the final ranked output, showing where AI, code and human review are used |
| [WRITEUP.md](WRITEUP.md) | Design decisions, consistency and explainability, human review, changing criteria, risks, sample results and assumptions |
| [PROMPTS.md](PROMPTS.md) | The extraction prompt and how the application handles the AI output |
| [ranking.json](ranking.json) | Expected ranking and results for proposals A–E |

## Design in one line

AI reads each proposal and extracts facts with evidence. Code handles the mandatory checks, scoring and ranking. A human reviews the results, can override them with a reason, and makes the final decision.

## How I used AI tools
