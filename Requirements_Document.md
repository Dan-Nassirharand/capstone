# Requirements Document

**Project:** GE Appliances, a Haier Company: Agentic Upgrades
**Team members:** Dan Nassirharand, Maxwell Blevins
**Document version:** 1.1 (2026-10-28)
**Companion document:** [Technical_Specification.md](Technical_Specification.md), which traces to this version.

---

## 1. Version and Change Log

| Version | Date       | Change                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | Author                            |
| ------- | ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| 0.1     | 2026-10-14 | Requirements first listed as FR-01 to FR-13 and NFR-01 to NFR-09 in [D2 part 4](Design_Diagrams/D2_Detailed_Design.md).                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | Dan Nassirharand, Maxwell Blevins |
| 1.0     | 2026-10-28 | Consolidated D2 requirements, [User_Stories.md](User_Stories.md), and the [Week 3 constraints essay](Constraints_Essay.md) into this document. Renamed FR-nn to REQ-nn (same numbers). Kept NFR-01 to NFR-09 and sharpened each to a measurable condition. Added NFR-10 (usability) and NFR-11 (reliability). Re-prioritized with MoSCoW: D2 had 16 of 22 requirements as Must; v1.0 has 9 of 24 (REQ-03, REQ-05, REQ-12, NFR-02, NFR-03, NFR-07, and NFR-08 moved from Must to Should; REQ-04, REQ-06, NFR-06, and NFR-09 moved from Should to Could). Added the Won't list and CON-01 to CON-05. | Dan Nassirharand, Maxwell Blevins |

| 1.1 | 2026-10-28 | Resolved OQ-03, OQ-04, OQ-05, and OQ-06 (part 9). Defined the US-01 productivity metric as review cycle time per pull request. Confirmed US-02 and US-03 as indirect stakeholder goals. Confirmed the MoSCoW split and numeric targets as final. Made the NFR-06 article cap explicit. No requirement text, ID, or priority changed except the NFR-06 wording. | Dan Nassirharand, Maxwell Blevins |

**ID rules.** IDs are stable and never reused or renumbered. A dropped requirement stays in its table with the priority "Withdrawn", and the drop is logged here. No requirement is withdrawn as of v1.1. Any change to the text, priority, or verification of a requirement is logged here and raises the version.

---

## 2. Purpose and Scope

**Purpose.** This document states what the system must do and for whom. How it is built is in [Technical_Specification.md](Technical_Specification.md).

**The system.** Two AI agents for the GE Appliances cloud engineering team, with one goal: boost developer productivity so the organization can deliver user features of higher quality and quantity.

1. **Pull request review (UC-01).** C1 and C2 post review comments on pull requests, each citing an article from the team knowledge base.
2. **Recipe audit (UC-02).** C3 and C4 score the recipe database, flag low-quality recipes, and delete only the recipes an engineer confirms.

**In scope for this release** (the capstone release, final presentation 2027-04-22): C1, C2, C3, C4, the C3 command-line client, and the shared configuration, notification, authentication, and logging pieces (G1 to G4). Architecture is in [D0](Design_Diagrams/D0_High_Level_Design.md).

**Out of scope:** see the Won't list in part 7.

**Sources.** [User_Stories.md](User_Stories.md) (US-01 to US-03, UC-01, UC-02, AC-01.1 to AC-02.2), [Constraints_Essay.md](Constraints_Essay.md), [D0](Design_Diagrams/D0_High_Level_Design.md), [D1](Design_Diagrams/D1_Detailed_Design.md), [D2](Design_Diagrams/D2_Detailed_Design.md), and [Plan.md](Planning/Plan.md).

---

## 3. Stakeholders and User Goals

From Week 4 ([User_Stories.md](User_Stories.md)).

| Stakeholder                              | Type      | Goal (story)                                                              | How this release serves the goal                                                                            | Requirements                       |
| ---------------------------------------- | --------- | ------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | ---------------------------------- |
| Engineer (Software Engineer, cloud team) | Primary   | US-01: increase my output so I can contribute to key projects faster.     | Cited, automatic review of pull requests (UC-01) and a safe audit of the recipe database (UC-02).           | REQ-01 to REQ-13, NFR-01 to NFR-11 |
| Consumer using SmartHQ                   | Secondary | US-02: use the smart features of my appliance without interruption.       | Indirect only, by decision (OQ-04): no requirement traces to US-02.                                         | None directly                      |
| Call center operator                     | Hidden    | US-03: limit cloud failures in production so call volumes stay regulated. | Indirect only. Reliability requirements NFR-02, NFR-03, NFR-11 and REQ-12 limit failures of our own agents. | None directly, by decision (OQ-04) |

---

## 4. Glossary

Every domain term and role name is defined here once. Both documents use these terms only as defined.

| Term                     | Definition                                                                                                                                                            |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Advisor                  | The project advisor chosen in Week 3, who reviews both documents.                                                                                                     |
| Audit run                | One execution of C3 over the recipe database. Its status is RUNNING, HALTED, AWAITING_CONFIRMATION, or COMPLETE.                                                      |
| C1 PR Review Agent       | GitHub Copilot code review, configured with custom instructions to use the knowledge base and cite articles.                                                          |
| C2 Knowledge Base Index  | The tool that selects the knowledge base articles relevant to a pull request's changed files. It stores no index; it reads the knowledge base repository on each run. |
| C3 Recipe Audit Agent    | The service and command-line client that run audit runs, flag recipes, and delete confirmed recipes.                                                                  |
| C4 Recipe Quality Scorer | The component that scores one recipe from 0 to 100 against the ruleset.                                                                                               |
| Checkpoint               | The saved `last_cursor` of the last fully scored batch of an audit run, used to resume.                                                                               |
| Citation                 | The ID of the knowledge base article that backs a review comment.                                                                                                     |
| Consumer                 | A person who uses an appliance through the SmartHQ connected ecosystem.                                                                                               |
| Call center operator     | A GE Appliances employee who takes consumer calls when something fails.                                                                                               |
| Defect                   | One failed ruleset rule on a recipe, with a `rule_id`, the `field`, and a `reason`.                                                                                   |
| Engineer                 | A Software Engineer on the GE Appliances cloud team. Also the pull request author in UC-01 and the operator of C3 in UC-02.                                           |
| Flagged recipe           | A recipe whose score is strictly below the threshold in an audit run.                                                                                                 |
| G1 to G4                 | Shared pieces: G1 Authentication, G2 Notifications, G3 Configuration, G4 Logging (see D2 part 2.6).                                                                   |
| Halted                   | The audit run status after a failure or timeout. Nothing more is deleted until the Engineer resumes.                                                                  |
| Keep exception           | An Engineer's decision that a flagged recipe stays. It applies to every later audit run.                                                                              |
| Knowledge base           | The team's engineering documentation, migrated from Confluence into a read-only GitHub repository.                                                                    |
| Knowledge base article   | One document in the knowledge base, with an ID and text.                                                                                                              |
| Manual review label      | The pull request label "knowledge base unavailable - manual review required", applied when C1 cannot complete a review.                                               |
| MoSCoW                   | Priority scheme: Must, Should, Could, Won't (this release).                                                                                                           |
| Pilot engineer           | An Engineer, other than the team, who uses the system on real work during validation (tasks T11 and T12).                                                             |
| Protected branch         | A branch that requires an approved review before merge.                                                                                                               |
| Pull request (PR)        | A GitHub request to merge changes into the protected branch.                                                                                                          |
| Recipe                   | A record in the recipe database with an ID, title, instructions (`steps`), `ingredients`, `domains`, and `cooktime`.                                                  |
| Recipe database          | The external DynamoDB table of about 200,000 recipes (an example size).                                                                                               |
| Removal log              | The record of each deleted recipe: run, recipe ID, score, defects, who confirmed, and when. It outlives the recipe.                                                   |
| Review pass              | One automated review of one version (`head_sha`) of a pull request by C1.                                                                                             |
| Ruleset                  | The version-controlled file of rules, penalties, and the default threshold used by C4.                                                                                |
| Sandbox                  | A repository and a non-production AWS account used for testing, never production data.                                                                                |
| Score                    | An integer from 0 to 100 for one recipe: 100 minus the penalties of its defects, clamped.                                                                             |
| SmartHQ                  | GE Appliances' connected appliance ecosystem.                                                                                                                         |
| Threshold                | The score below which a recipe is flagged. Default 70; an example that is subject to change.                                                                          |
| Verdict                  | The outcome of a review pass: `approved` (no violations remain) or `changes_requested`.                                                                               |

---

## 5. Functional Requirements

Verification methods: **Test** (automated or scripted), **Demonstration** (run by a teammate or Advisor), **Inspection** (review of configuration or code), **Analysis** (calculation or measurement review). The numeric targets in these tables are final (OQ-06). The scoring values (the default threshold of 70 and the rule penalties) are examples until OQ-02 is answered.

| ID     | Requirement ("The system shall...")                                                                                                                                                                                                    | Priority | Source story                 | Verification                          |
| ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ---------------------------- | ------------------------------------- |
| REQ-01 | Retrieve from the knowledge base the articles relevant to a pull request's changed files, so that the mapped article is returned for at least 90% of a seeded set of 20 sandbox pull requests.                                         | Must     | US-01, UC-01 step 2          | Test (seeded sandbox pull requests)   |
| REQ-02 | Post an inline comment for each knowledge-base-based finding, each carrying the citation of an existing article (100% of such comments). Findings without an article are marked with a Low or Medium impact and confidence.            | Must     | US-01, UC-01 step 3, AC-01.1 | Test; Inspection of posted comments   |
| REQ-03 | Start a new review pass on every push (`synchronize`), review only the latest `head_sha` (0 comments posted for an older one), and set the verdict `approved` when no violations remain.                                               | Should   | US-01, UC-01 steps 4 and 5   | Test (two pushes in quick succession) |
| REQ-04 | Let the Engineer dismiss an informational comment with a one-line justification, after which the comment is shown as resolved (uses GitHub's native resolve).                                                                          | Could    | US-01, UC-01 A2              | Demonstration                         |
| REQ-05 | When the knowledge base is unreachable, apply the manual review label and mention the author in a comment in 100% of injected outages, and post no uncited review.                                                                     | Should   | US-01, UC-01 E1, AC-01.2     | Test (simulated outage)               |
| REQ-06 | Leave a recorded outcome (`approved` or `changes_requested`) with its citations on the pull request in 100% of completed review passes, with creation and approval timestamps available for the organization's cycle time measurement. | Could    | US-01, UC-01 postcondition   | Inspection; Demonstration             |
| REQ-07 | Start an audit run on the Engineer's command, return a run ID immediately, and read and score the recipe database in batches until every non-excepted recipe is scored (`scored_count` equals recipes read).                           | Must     | US-01, UC-02 steps 1 and 2   | Test (sandbox recipe data)            |
| REQ-08 | Flag every recipe that scores strictly below the threshold (default 70), each with at least one defect naming the `rule_id`, `field`, and `reason`. A score of 69 is flagged and 70 is not.                                            | Must     | US-01, UC-02 step 3, AC-02.1 | Test (threshold boundary)             |
| REQ-09 | Show the Engineer every flagged recipe with its title, score, defects, and decision, at most 200 per page, before any deletion.                                                                                                        | Must     | US-01, UC-02 step 4          | Demonstration; Test                   |
| REQ-10 | Delete only recipes whose IDs the Engineer confirmed (the client requires the typed word `yes`), and report the count removed. After a deletion, 100% of the reported IDs are absent from the recipe database.                         | Must     | US-01, UC-02 step 5          | Test; Demonstration                   |
| REQ-11 | Skip every recipe marked as a keep exception in 100% of later audit runs.                                                                                                                                                              | Should   | US-01, UC-02 A2              | Test (two successive runs)            |
| REQ-12 | On a batch read longer than 30 seconds or a failure, halt the run, keep the checkpoint, delete nothing further, and show the Engineer the halt reason and resume point. A resume continues from the checkpoint.                        | Should   | US-01, UC-02 E1, AC-02.2     | Test (injected timeout)               |
| REQ-13 | Write a removal log entry (run, recipe ID, score, defects, confirmer, UTC time) before deleting each recipe, for 100% of deletions. If the write fails, the recipe is not deleted.                                                     | Must     | US-01, UC-02 postcondition   | Test (injected write failure)         |

---

## 6. Non-Functional Requirements

| ID     | Category            | Requirement ("The system shall...")                                                                                                                                                                                                             | Priority | Source                      | Verification                                                      |
| ------ | ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | --------------------------- | ----------------------------------------------------------------- |
| NFR-01 | Performance         | Post the review comments of a pass within 5 minutes of pull request creation, for every pass on a pull request within the review size limit. Any overrun is logged and counted as a failure of this requirement in pilot results.               | Must     | US-01, AC-01.1              | Test (timed sandbox pull requests); Analysis                      |
| NFR-02 | Reliability         | Notify the author within 10 minutes of pull request creation when the knowledge base is unavailable (100% of injected outages).                                                                                                                 | Should   | US-01, AC-01.2              | Test (simulated outage, timed)                                    |
| NFR-03 | Performance         | Allow no recipe database batch read to run longer than 30 seconds; a read that exceeds it sets the run to HALTED.                                                                                                                               | Should   | US-01, AC-02.2              | Test (injected delay)                                             |
| NFR-04 | Security            | Accept C3 requests only with a valid AWS SigV4 signature from an authorized IAM principal (others get 403 and reach nothing), and delete 0 recipes that were not listed in a confirmed deletion request.                                        | Must     | US-01, UC-02 step 5, CON-01 | Test (unsigned and unlisted-ID requests); Inspection (IAM policy) |
| NFR-05 | Security            | Run all development testing of C1 and C3 in the sandbox: 0 test runs against the production recipe database or a production repository.                                                                                                         | Should   | CON-03                      | Inspection (account and repository configuration)                 |
| NFR-06 | Cost                | Make 0 model calls to score recipes, and pass the model at most the article cap set in configuration per review pass (the cap value is fixed in task T3, due 2026-11-25).                                                                       | Could    | CON-04                      | Inspection; Test                                                  |
| NFR-07 | Reliability         | Give the same score and defects every time the same recipe is scored with the same ruleset version (100% across 2 repeated runs), and store the version on the audit run.                                                                       | Should   | US-01, AC-02.1              | Test (repeated runs)                                              |
| NFR-08 | Security            | Attribute 100% of deletions to a named confirmer and a `run_id` in both the removal log and the log entries.                                                                                                                                    | Should   | US-01, CON-01               | Inspection of logs; Test                                          |
| NFR-09 | Performance (scale) | Handle 100 times the current recipe volume (about 20,000,000 recipes) with no request over the 29-second gateway limit and no status page over 200 flagged recipes.                                                                             | Could    | US-01, D1 part 3            | Analysis; Test (generated data)                                   |
| NFR-10 | Usability           | Let an Engineer who has never used the client start an audit, review the flagged recipes, and delete one confirmed recipe in 10 minutes or less using only the built-in help, with at least 4 of 5 pilot engineers succeeding on the first try. | Should   | US-01, UC-02                | Test (observed pilot task)                                        |
| NFR-11 | Reliability         | After a C3 restart or crash during a run, set the run to HALTED and, on resume, finish with the same flagged set as an uninterrupted run (100%), re-scoring at most one batch.                                                                  | Should   | US-01, US-03, AC-02.2       | Test (kill the service mid-run)                                   |

---

## 7. MoSCoW Summary

**Count:** 24 requirements: 9 Must (37.5%), 11 Should, 4 Could. The Must set is the MVP: it shows both use cases end to end with citations, confirmed deletion, and a removal log, inside the 5-minute and authorization rules.

**Must**

- REQ-01, REQ-02, NFR-01 (pull request review that cites knowledge base articles within 5 minutes)
- REQ-07, REQ-08, REQ-09, REQ-10, REQ-13 (audit, flag, review, delete with a log)
- NFR-04 (authenticated access; no unconfirmed deletion)

**Should**

- REQ-03, REQ-05, REQ-11, REQ-12
- NFR-02, NFR-03, NFR-05, NFR-07, NFR-08, NFR-10, NFR-11

**Could**

- REQ-04, REQ-06, NFR-06, NFR-09

**Won't (this release).** Written down so they cannot return without a change-log entry.

| ID   | Won't                                                                                                                        | Origin            |
| ---- | ---------------------------------------------------------------------------------------------------------------------------- | ----------------- |
| W-01 | Delete any recipe without the Engineer's confirmation.                                                                       | D0 decision 1     |
| W-02 | Score recipes with an LLM.                                                                                                   | D0 decision 2     |
| W-03 | Remove or edit a keep exception.                                                                                             | D2 Q2             |
| W-04 | Send push notifications (email or chat) for halted runs or fallbacks. The Engineer checks status.                            | D2 Q3, R9         |
| W-05 | Provide a web or graphical front end for C3. The command line is the only interface.                                         | D1 part 6, R5     |
| W-06 | Review architecture or conventions that are not documented in the knowledge base.                                            | Peer review PR-2  |
| W-07 | Score recipes added after the cursor passed them in the same run.                                                            | D2 Q4             |
| W-08 | Run more than one audit run, or more than one C3 instance, at a time.                                                        | D1 part 3, EC-12  |
| W-09 | Commit code changes to a pull request or edit recipe content. The system only comments, flags, and (when confirmed) deletes. | D0 part 1         |
| W-10 | Compute review cycle time inside C1. The organization's existing measurement is used.                                        | D2 part 4, REQ-06 |
| W-11 | Build consumer-facing SmartHQ features or call center tooling.                                                               | US-02, US-03      |

---

## 8. Inherited Constraints

From the Week 3 [constraints essay](Constraints_Essay.md). What each rules out or forces is in [Technical_Specification.md](Technical_Specification.md) part 5.

| ID     | Essay area   | Constraint                                                                                                                                                                                               |
| ------ | ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CON-01 | Security     | Work with AI agents carries security risk, and an exploited vulnerability could harm the company socially, economically, legally, and professionally. The team shall align with security best practices. |
| CON-02 | Security     | The team shall use substantive code reviews.                                                                                                                                                             |
| CON-03 | Security     | The team shall test in isolated environments.                                                                                                                                                            |
| CON-04 | Professional | Token usage and AI model subscriptions cost money. The team shall minimize cost through developer education, model selection, and communication with non-technical stakeholders.                         |
| CON-05 | Social       | The work must benefit developers and consumers, because that benefit supports GE Appliances' reputation and offsets the economic and security tradeoffs.                                                 |

---

## 9. Assumptions and Open Questions

**Assumptions**

| ID   | Assumption                                                                                                                                  | Owner   | Confirm by |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------- | ------- | ---------- |
| A-01 | The knowledge base is already migrated from Confluence into a GitHub repository and is readable by C2 (UC-01 precondition).                 | Dan     | 2026-11-02 |
| A-02 | A sandbox repository, a non-production AWS account, and recipe data are provided through the Advisor (Plan R5).                             | Maxwell | 2026-11-02 |
| A-03 | Two pilot engineers are named through the Advisor (Plan R7).                                                                                | Dan     | 2027-01-27 |
| A-04 | The recipe database has about 200,000 recipes. All scores, penalties, thresholds, caps, and batch sizes are examples held in configuration. | Maxwell | 2026-11-25 |
| A-05 | One C3 instance runs at a time (W-08).                                                                                                      | Maxwell | 2026-11-02 |

**Open questions**

| ID    | Question                                                                                                                                                                                       | Owner   | Answer due |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- | ---------- |
| OQ-01 | Does the Copilot reviewer reliably call C2 and finish within 5 minutes (NFR-01)? Answered by task T1.                                                                                          | Dan     | 2026-11-09 |
| OQ-02 | Which scoring rules, penalties, and threshold does the GE technical liaison accept as the quality standard (REQ-08)? Until answered, the threshold of 70 and the penalties are example values. | Maxwell | 2026-11-25 |
| OQ-07 | Is a push channel for halted runs needed after the pilot (W-04)?                                                                                                                               | Maxwell | 2027-04-06 |

**Resolved questions**

| ID    | Question                                             | Answer (2026-10-28)                                                                                                                                                                            | Owner   |
| ----- | ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- |
| OQ-03 | How is "increased output" (US-01) measured?          | By review cycle time per pull request, taken from pull request timestamps by the organization's existing measurement (REQ-06, W-10). US-01 still has no Testable criterion in User_Stories.md. | Dan     |
| OQ-04 | Do US-02 and US-03 get requirements of their own?    | No. They are indirect stakeholder goals, and no requirement traces to them (W-11).                                                                                                             | Dan     |
| OQ-05 | Is the MoSCoW split in part 7 final?                 | Yes. The split is final: 9 Must, 11 Should, 4 Could. Any later change is logged in part 1.                                                                                                     | Dan     |
| OQ-06 | Are the numeric targets in parts 5 and 6 acceptable? | Yes. They are final: 90% of 20 seeded pull requests, 10 minutes, and 4 of 5 pilot engineers.                                                                                                   | Maxwell |
