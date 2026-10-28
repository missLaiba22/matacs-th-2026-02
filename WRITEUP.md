# AI Proposal Triage Design

**Core idea:** AI reads, code decides, and a human approves.

The AI's job is only to read the proposal and turn the free-written text into structured facts with evidence. It does not decide whether a proposal passes, does not calculate the score, and does not choose the winner.

All pass/fail checks, scoring, and ranking are handled by normal code. The final decision stays with a human.

See the [flow diagram](diagram/process_flow.png).

## 1. Where I used AI vs. normal code

| Step | Type | Why |
|---|---|---|
| Extract text from PDF | Code | A standard PDF library can handle this reliably |
| Extract facts like price, licence, weeks, warranty, installs and team details | **AI** | Every proposal is written differently, so this needs language understanding |
| Validate the extracted data and evidence | Code | This should be deterministic and not depend on AI judgement |
| Detect suspicious/manipulative content | AI + code | AI can identify suspicious text, while a separate keyword check gives us another layer |
| Mandatory pass/fail checks | Code | These are simple yes/no rules |
| Scoring and ranking | Code | The same formula should produce the same result every time |
| Final decision | Human | The final responsibility should stay with a person |

The main line I drew was **reading vs. judging**.

AI is useful for reading different ways of writing the same information. But I don't want it deciding that one proposal is better than another. If AI did the scoring, two runs could potentially produce different results without a clear reason.

By limiting AI to extraction, its mistakes are easier to see. If it extracts the wrong value, we can show the value next to the exact quote it came from and let the reviewer check it.

## 2. Consistency and explainability

For extraction, I use temperature 0, one proposal per call, and a versioned prompt.

The scoring itself is completely deterministic. Once we have the extracted facts, the same facts will always produce the same score.

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
  3.75 points each for:
  - named lead
  - crew details
  - safety plan
  - performance monitoring plan

The important part is that the reviewer doesn't just see a final number.

For every proposal, they can see:

**extracted fact → evidence quote → points awarded**

There is also a short explanation of why the proposal ended up where it did.

## 3. Human in the loop

There are a few points where a human can step in.

**Before ranking:**  
If a proposal has missing information, suspicious content, or an evidence problem, it is flagged for review. The reviewer can correct the extracted fact, ask the bidder for more information, or confirm the flag.

**After ranking:**  
The reviewer can still override a rank or status. If they do, they have to provide a reason.

**Final award:**  
The tool never makes the final award. A human makes that decision.

### Audit log

I also want the system to keep enough information to reproduce a result later.

The log includes:

- PDF file hash
- prompt version
- model name
- AI input and output
- criteria version
- score breakdown
- flags
- overrides, including who made them, when and why
- final decision

So if someone asks later, "Why did this proposal get this score?", we have the information to trace it back.

## 4. What happens when the criteria change?

Criteria are versioned.

If someone changes the criteria, it creates a new version instead of changing the old one. Every score is linked to the criteria version that produced it.

This also means we don't need to call the AI again for every change.

For example, if only the **weights** change, the extracted facts are already there, so we can recalculate the scores directly in code.

If a new criterion needs a new fact that we didn't extract before, we only need to extract that new field from the existing proposals.

I also don't want different criteria versions mixed together in one ranking. The reviewer should be able to see that the criteria changed and compare the old and new rankings.

An already-approved result stays connected to the version of the criteria under which it was approved.

## 5. Risks and how I handled them

| Risk | Safeguard |
|---|---|
| **Wrong extraction** such as reading the wrong price or number of weeks | Every value needs an evidence quote, and the reviewer sees the value next to the quote |
| **Made-up facts** | The quote must actually exist in the extracted PDF text. If there is no evidence, the value becomes `null` instead of being guessed |
| **Vague promises being treated as facts** | Phrases like "to be confirmed" or "will be determined" are treated as `null` with a note |
| **Prompt injection / manipulation** | Proposal text is treated as data, not instructions. AI reports suspicious content and a separate keyword scan runs as well |
| **AI changing the ranking through an injection** | AI never assigns scores. Even if the extraction is manipulated, it cannot directly change the scoring formula |
| **Bias** | AI only extracts facts. Every proposal goes through the same rules and formula |
| **Relative scoring changing when bids are added or removed** | The reviewer is shown that scores are relative to the current set of bids, and this is recorded in the log |
| **Cost** | There is one extraction call per proposal. Extracted facts are cached, so re-scoring does not require another AI call |
| **Bad PDF text / scanned PDFs** | If the extracted text is empty or too short, the proposal is flagged for manual handling instead of blindly sending bad input to the AI |

One important point here is that **polished writing does not give a proposal extra points**. The score only comes from the facts that are actually stated and verified.

## How I handled the sample proposals

- **A (98)** and **E (85)** are eligible and ranked. A performs better on experience, timeline and team.

- **B** is **ineligible** because the bidder itself does not hold the required electrical licence. Saying that a future subcontractor will have the licence is not enough.

- **C** is **flagged** because of the instruction embedded in its proposal. If we ignore that instruction and score only the stated facts, it gets **77**, below A and E. Its install count also covers six years instead of five, so I kept the stated number but flagged it for the reviewer.

- **D** is **flagged** because the timeline is missing. With the timeline treated as 0, it gets **70**. But if the bidder confirms a timeline of around 12 weeks, its score could move above E. So this should be a human follow-up, not an automatic decision by the system.

## Assumptions

A few assumptions are important for the sample:

- A licence that will only be held by a future subcontractor does not satisfy the mandatory licence requirement.
- A missing timeline means **cannot verify**, rather than an automatic fail.
- "Weekly progress reports" do not count as a performance monitoring plan.