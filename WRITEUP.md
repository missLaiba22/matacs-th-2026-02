# AI Proposal Triage Design

**Core idea:** AI reads, code decides, and a human approves. The AI only turns free-written proposal text into structured facts with evidence. It does not decide whether a proposal passes, calculate the score, or choose the winner. See the [flow diagram](diagram/process_flow.png).

## 1. Where I used AI vs. normal code

| Step | Type | Why |
|---|---|---|
| Extract text from PDF | Code | A standard PDF library can handle this reliably |
| Extract facts (price, licence, weeks, warranty, installs, team) | **AI** | Every proposal is written differently, so this needs language understanding |
| Validate the extracted data and evidence | Code | This should be deterministic and not depend on AI judgement |
| Detect suspicious/manipulative content | AI + code | AI flags suspicious text; a separate keyword check adds another layer |
| Mandatory pass/fail checks | Code | These are simple yes/no rules |
| Scoring and ranking | Code | The same formula should produce the same result every time |
| Final decision | Human | Responsibility stays with a person |

The main line I drew was **reading vs. judging**. AI is useful for reading different ways of writing the same information, but I don't want it deciding that one proposal is better than another. If AI did the scoring, two runs could produce different results without a clear reason. Limited to extraction, its mistakes are easier to see: the reviewer sees each value next to the exact quote it came from.

## 2. Consistency and explainability

Extraction uses temperature 0, one proposal per call, and a versioned prompt. Scoring is deterministic: the same facts always produce the same score.

For the sample set, I used this scoring formula:

- **Experience: 30 points**  
  installs ÷ highest installs × 30

- **Price: 25 points**  
  lowest price ÷ bid price × 25

- **Timeline: 15 points**  
  shortest timeline ÷ bid timeline × 15

- **Warranty: 15 points**  
  half for panel warranty and half for workmanship warranty, both relative to the best bid

- **Team: 15 points**  
  3.75 points each for named lead, crew details, safety plan and performance monitoring plan

The reviewer doesn't just see a final number. For every proposal they see **extracted fact → evidence quote → points awarded**, plus a short reason for its position.

## 3. Human in the loop

**Before ranking:**  
If a proposal has missing information, suspicious content, or an evidence problem, it is flagged for review. The reviewer can correct the extracted fact, ask the bidder for more information, or confirm the flag.

**After ranking:**  
The reviewer can still override a rank or status. If they do, they have to provide a reason.

**Final award:**  
The tool never makes the final award. A human makes that decision.

### Audit log

The log keeps enough to reproduce any result later: PDF file hash, prompt version, model name, AI input and output, criteria version, score breakdown, flags, overrides (who, when and why) and the final decision.

## 4. What happens when the criteria change?

Criteria are versioned. A change creates a new version instead of editing the old one, and every score is linked to the version that produced it.

If only the **weights** change, the extracted facts are already there, so scores are recalculated in code with no new AI calls. If a new criterion needs a new fact, only that field is extracted from the existing proposals.

Different versions are never mixed in one ranking. The reviewer can see that the criteria changed and compare old and new rankings. An already-approved result stays tied to the version it was approved under.

## 5. Risks and how I handled them

| Risk | Safeguard |
|---|---|
| **Wrong extraction** such as reading the wrong price or number of weeks | Every value needs an evidence quote, and the reviewer sees the value next to the quote |
| **Made-up facts** | The quote must actually exist in the extracted PDF text. If there is no evidence, the value becomes `null` instead of being guessed |
| **Vague promises being treated as facts** | Phrases like "to be confirmed" or "will be determined" are treated as `null` with a note |
| **Prompt injection / manipulation** | Proposal text is treated as data, not instructions. AI reports suspicious content and a separate keyword scan runs as well. AI never assigns scores, so an injection cannot change the formula |
| **Bias** | AI only extracts facts, every proposal goes through the same rules and formula, and polished writing earns no extra points |
| **Relative scores shift when bids are added or removed** | The reviewer is told scores are relative to the current bids, and this is logged |
| **Cost** | One extraction call per proposal; facts are cached, so re-scoring needs no AI call |
| **Bad PDF text / scanned PDFs** | If the extracted text is empty or too short, the proposal is flagged for manual handling instead of being sent to the AI |

## How I handled the sample proposals

- **A (98)** and **E (85)** are eligible and ranked. A performs better on experience, timeline and team.

- **B** is **ineligible** because the bidder itself does not hold the required electrical licence; a future subcontractor's licence is not enough.

- **C** is **flagged** because of the instruction embedded in its proposal. Scored only on stated facts, it gets **77**, below A and E. Its install count also covers six years instead of five, so I kept the stated number but flagged it for the reviewer.

- **D** is **flagged** because the timeline is missing. With timeline as 0 it gets **70**, but a confirmed timeline of around 12 weeks could move it above E, so this needs human follow-up.

## Assumptions

- A missing timeline means **cannot verify**, rather than an automatic fail.
- "Weekly progress reports" do not count as a performance monitoring plan.