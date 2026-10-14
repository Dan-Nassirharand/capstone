# Peer Review Notes (D2)

**Team:** GE Appliances, a Haier Company
**Team members:** Dan Nassirharand, Maxwell Blevins

**Project title:** GE Appliances, a Haier Company: Agentic Upgrades

**Goal statement:** Use AI agents to boost developer productivity, enabling the organization to deliver user features of higher quality and quantity.

Status: feedback received from both reviewing teams. The top two issues we raised for other teams are still `TBD`.

---

## Reviewing Teams

| Slot   | Team | Reviewers | Date |
| ------ | ---- | --------- | ---- |
| Team 1 | Team 15 | Isabella Read, Luis Ruiz | 2026-10-14 |
| Team 2 | Team 46 | Jack Keller, Andrew Ogrin-Cotarlan, Nathan Wiley, Duy Pham | 2026-10-14 |

---

## Action Items

Criteria: completeness, feasibility, clarity, risk. Priority: must-fix, worth considering, out of scope for now.

| ID   | Criterion | Issue (raised by team) | Action | Owner | Priority | Due |
| ---- | --------- | ---------------------- | ------ | ----- | -------- | --- |
| PR-1 | Risk      | Auto re-run on every push: if the second run starts before the first finishes, could there be conflicts or issues? (Team 1) | Confirm and document that EC-02 covers this: the older pass is discarded and only the latest `head_sha` is reviewed. Verify the discarded pass posts no comments or approval. Add the answer to F4 and the overview. | Maxwell | Must-fix | 2026-10-21 |
| PR-2 | Feasibility | Review agent uses knowledge base articles. Consider a project whose code architecture goes beyond, or differs from, what articles can express. (Team 1) | Note the limit in D2 (2.5 / R-n): articles cover documented conventions only; architecture not in the knowledge base falls to the manual review label or is a future extension. | Dan | Worth considering | 2026-10-21 |
| PR-3 | Clarity   | Technical aspects are well defined, but the overview needs a layman's explanation or a more descriptive one. (Team 1) | Add a plain-language overview to D2 section 1 (what each agent does, for whom, and why), with no jargon. | Maxwell | Must-fix | 2026-10-21 |
| PR-4 | Clarity   | The project seems to tackle 2 problems (PR review agent and recipe audit agent); this was not explained clearly and was hard to follow, partly from missing background knowledge. Team liked the goal of boosting developer productivity. (Team 2) | Add a short statement naming the two problems (C1/C2 PR review, C3/C4 recipe audit), how each ties to the goal, and why they are one project. Define needed background terms. Can be combined with PR-3 if desired. | Dan | Must-fix | 2026-10-21 |
---

## Summary

| Criterion    | Items |
| ------------ | ----- |
| Completeness | 0     |
| Feasibility  | 1 (PR-2) |
| Clarity      | 2 (PR-3, PR-4) |
| Risk         | 1 (PR-1) |

**Criterion with the most feedback:** Clarity. Both teams said the overview was hard to follow: Team 1 asked for a layman's explanation, and Team 2 found the two problems unclear.

**Top two issues we raised while reviewing others**

1. Being more specific with laying out their API contracts.
2. Requesting an expansion of technologies used. (Instead of saying Python, maybe specify using FastAPI.)
