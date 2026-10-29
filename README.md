# MATACS-TH-2026-02: AI Proposal Triage Design

**Name:** Muhammad Adeel Raza
**Reference:** MATACS-TH-2026-02
**Time actually spent:** about 3 hours

## Summary

AI extracts facts from proposals with exact quotes. Code applies mandatory checks, missing-data rules and a fixed scoring formula. A human reviews flagged items and makes the final award decision. Proposals are referred to by letter (A to E) only.

## How I used AI tools

I used Claude to help me understand the brief, plan the design and draft the files, and I built the diagram myself in draw.io. While working through the scoring formula, Claude first described Proposal A's price as the lowest in the set, but Proposal D's price is actually lower. D was left out of the benchmark because it is flagged and not scored. Claude pointed this out itself when showing the calculation. I reviewed the effect (A's price score drops slightly but the ranking stays A, then E) and added an assumption to WRITEUP.md that price and timeline scores are relative to the scored proposals only, and are recalculated if a flagged proposal is cleared.

## Files

- [Process flow diagram](diagram/process-flow.png)
- [Write-up (5 questions)](WRITEUP.md)
- [AI prompts](PROMPTS.md)
- [Expected ranking](ranking.json)
