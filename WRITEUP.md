# Write-up: AI Proposal Triage Design (MATACS-TH-2026-02)

## Core idea

AI reads messy text. Code makes decisions. A human awards the contract. The AI extracts facts with quotes, and plain code applies pass/fail rules and a fixed scoring formula. The AI never assigns scores or ranks.

## 1. AI vs. rules

**AI is used in two steps only:**
- **Fact extraction.** Proposals are free-form text, so AI is good at finding the price, weeks, warranty years and so on, wherever they appear.
- **Wording the reason.** After code has calculated everything, AI writes one plain sentence for the reviewer. It cannot change any number.

**Plain code or rules handle everything else:**
- Loading criteria and weights from the admin database
- Text extraction from the PDF
- Scanning for manipulation phrases
- Checking quotes and JSON format
- Mandatory pass/fail
- Missing-data checks
- Scoring and ranking

**Why this line:** AI is good at understanding language but can vary between runs and can be tricked. Pass/fail, maths and ranking must be exact and repeatable, so code does them.

## 2. Consistency and explainability

- **Same input, same output.** Extraction runs at temperature 0 with a versioned prompt and a fixed output format. Scoring is a formula, so identical facts always give an identical score.
- **Extraction can still vary slightly,** because AI is not perfectly deterministic. So extracted facts are saved once. Re-ranking uses the saved facts, not a new AI call. If the same proposals are ranked twice, the result is the same.
- **Explainability.** Every fact comes with an exact quote from the proposal, and code checks the quote really exists in the text. The reviewer sees a table per proposal: each criterion, the fact found, the quote, the sub-score, the weight and the contribution to the total. Nothing is a black box.

**Scoring rules (all in code, weights from the database):**
- Experience: tiers by number of qualifying installs
- Price: lowest eligible price divided by this price
- Timeline: shortest weeks divided by this proposal's weeks
- Warranty: based on panel and workmanship years
- Team and approach: checklist (named lead, crew, safety plan, monitoring plan)

## 3. Human in the loop

A person reviews at four points:
1. **Upload and category choice** by the reviewer.
2. **Flagged list.** Proposals with suspected manipulation or missing key data are never silently ranked or dropped. The reviewer decides, for example by asking the bidder for clarification.
3. **Ineligible list.** The reviewer can check the quote behind each fail and correct a wrong extraction.
4. **Final ranking and award.** The reviewer can override any position, but must type a reason. The award decision is always human.

**Audit log records:** who uploaded what and when, the criteria version, prompt version and model, extracted facts and quotes, validation results, scores, flags, every override with its reason, and the final decision.

## 4. Changing criteria

- Every score is stored with the **criteria version** it was calculated with.
- When an admin edits criteria or weights, existing results are marked **outdated** (never silently changed).
- If only weights or scoring rules changed, code re-scores from the saved facts. No new AI call, so it is fast, cheap and consistent.
- If a new criterion needs a fact that was never extracted, only that fact is extracted again.
- Old and new results are both kept in the audit log. The reviewer sees which version produced which ranking. Mandatory criteria changes can move a proposal between ranked and ineligible, so the tool shows that change clearly.

## 5. Risks and safeguards

| Risk | Safeguard |
|---|---|
| **Wrong extraction** | Every fact needs an exact quote, which code verifies. A fact with no matching quote becomes null and is flagged. The reviewer sees quotes. |
| **Made-up facts** | The AI must return null when information is missing or vague. It may not guess or infer. |
| **Manipulation (prompt injection)** | Proposal text is treated as data, not instructions. A rule-based scan and the AI's own detection field both look for embedded instructions. Proposal C contains such a line, so it is flagged and held for human review. Because the AI never scores, an injected command cannot raise a rank. |
| **Unfair treatment of missing data** | Proposal D gives no timeline. The tool does not guess; it flags the proposal and asks for clarification, so a bidder is not penalised or favoured by an assumption. |
| **Mandatory rules bypassed by price** | Mandatory checks run before scoring. Proposal B is cheapest but its licence is not held by the bidder, so it is ineligible no matter how good its price is. |
| **Bias** | Company name, size and location are not scoring inputs. Only the listed criteria are scored. |
| **Out-of-date experience** | Experience must be within the last 5 years. The extractor also captures the start year, and code checks whether the install count really falls in the window. |
| **Cost** | Each proposal is extracted once and cached. Re-scoring uses saved facts. A small model is enough for extraction. |
| **Over-reliance on the tool** | The tool only recommends. A human confirms, and every override is logged. |

## Assumptions

- The field names in PROMPTS.md match this solar category only. In the real tool, the extraction field list is generated from the admin's criteria for the chosen category.
- Proposal C is flagged and not ranked, because the embedded instruction is an integrity concern. An alternative is to score its genuine content and show it in the flagged list for the reviewer to decide. I chose the safer option.
- Scoring tiers (for example installs: 12+ = 100, 8 to 11 = 80) are my own example. In the real tool, the admin would set them.
- Expected scores for A and E in ranking.json come from this formula: A about 98.8 and E about 92.4.

- Price and timeline scores are relative to the proposals being scored; flagged proposals are excluded from the benchmark and scores are recalculated if a flagged proposal is later cleared.
