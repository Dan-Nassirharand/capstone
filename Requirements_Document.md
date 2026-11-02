# Requirements Document

- **Project:** GE Appliances, a Haier Company: Agentic Upgrades
- **Team members:** Dan Nassirharand, Maxwell Blevins
- **Document version:** 1.4 (2026-10-29)
- **Companion document:** [Technical_Specification.md](Technical_Specification.md), which traces to this version.

**Contents**

1. [Version and Change Log](#1-version-and-change-log)
2. [Purpose and Scope](#2-purpose-and-scope)
3. [Stakeholders and User Goals](#3-stakeholders-and-user-goals)
4. [Glossary](#4-glossary)
5. [Functional Requirements](#5-functional-requirements)
6. [Non-Functional Requirements](#6-non-functional-requirements)
7. [MoSCoW Summary](#7-moscow-summary)
8. [Inherited Constraints](#8-inherited-constraints)
9. [Assumptions and Open Questions](#9-assumptions-and-open-questions)

---

## 1. Version and Change Log

**ID rules.** IDs are stable and never reused or renumbered. A dropped requirement stays in its section with the priority "Withdrawn", and the drop is logged here. No requirement is withdrawn as of v1.4. Any change to the text, priority, or verification of a requirement is logged here and raises the version.

### Version 1.4 (2026-10-29)

Authors: Dan Nassirharand, Maxwell Blevins

- **REQ-07 (text).** `scored_count` equals the recipes read, less the keep exceptions and repeated IDs that the audit loop skips (D1 part 3). The old wording, "equals recipes read", could not hold when any recipe is skipped.
- **NFR-11 (source).** US-03 removed from the source list. OQ-04 and W-11 say no requirement traces to US-02 or US-03, and part 3 already treats US-03 as indirect. The requirement text is unchanged.
- **Related Technical Specification v1.4 changes** (no other requirement text changed): flag counts stay exact when a write is retried, the last checkpoint also leaves `RUNNING`, replay cleanup uses strongly consistent reads, deletes match the recipe that was scored, and an unknown delete outcome keeps its claim and log row for reconciliation. These support NFR-11, NFR-08, and REQ-13.
- No ID or priority changed.

### Version 1.3 (2026-10-29)

Authors: Dan Nassirharand, Maxwell Blevins

- **NFR-11 (text and verification).** "The same flagged set as an uninterrupted run" now means an uninterrupted run over the same recipe data. If a recipe in the re-scored batch changes between the crash and the resume, the run reflects its state at resume. In every case, the flagged list matches `flagged_count`. Verification now repairs a recipe of the interrupted batch before resuming.
- **Related Technical Specification v1.3 changes** (no other requirement text changed): stale flag rows from an interrupted batch are removed before replay (supports NFR-11), and a per-recipe claim keeps deletion attribution correct when two requests target the same recipe (supports NFR-08, REQ-13).
- No ID or priority changed.

### Version 1.2 (2026-10-28)

Authors: Dan Nassirharand, Maxwell Blevins

- **NFR-01 and NFR-02 (text and verification).** Both deadlines are now measured from the triggering `opened` or `synchronize` event, not from pull request creation. REQ-03 starts a new pass on every push, so a creation-based deadline failed any pass started by a later push before it began. The first pass is still measured from creation, so AC-01.1 and AC-01.2 are unchanged. A pass discarded under REQ-03 is not measured. Verification now includes timed pushes.
- **A-03.** Raised from two to at least five eligible pilot engineers. NFR-10 needs at least 4 successes among 5 first-time users (confirmed final in OQ-06), which two people cannot verify. NFR-10's verification now names the five eligible pilot engineers; its text is unchanged. The linked plan risk R7 and milestone M18 are updated in [Plan.md](Planning/Plan.md).
- **Glossary.** Added "Triggering event". "Pilot engineer" now states who is eligible for NFR-10.
- **Related Technical Specification v1.2 changes** (no requirement text changed): deletion log entries now carry `confirmed_by` so NFR-08 can be met, and a shared active-run lock enforces one audit run at a time (W-08, A-05).
- **Layout.** Requirements, glossary, and lists rewritten for readability. No other content changed.
- No ID or priority changed. The MoSCoW split is unchanged.

### Version 1.1 (2026-10-28)

Authors: Dan Nassirharand, Maxwell Blevins

- Resolved OQ-03, OQ-04, OQ-05, and OQ-06 (part 9).
- Defined the US-01 productivity metric as review cycle time per pull request.
- Confirmed US-02 and US-03 as indirect stakeholder goals.
- Confirmed the MoSCoW split and numeric targets as final.
- Made the NFR-06 article cap explicit. No requirement text, ID, or priority changed except the NFR-06 wording.

### Version 1.0 (2026-10-28)

Authors: Dan Nassirharand, Maxwell Blevins

- Consolidated D2 requirements, [User_Stories.md](User_Stories.md), and the [Week 3 constraints essay](Constraints_Essay.md) into this document.
- Renamed FR-nn to REQ-nn (same numbers). Kept NFR-01 to NFR-09 and sharpened each to a measurable condition.
- Added NFR-10 (usability) and NFR-11 (reliability).
- Re-prioritized with MoSCoW. D2 had 16 of 22 requirements as Must; v1.0 has 9 of 24.
  - Must to Should: REQ-03, REQ-05, REQ-12, NFR-02, NFR-03, NFR-07, NFR-08.
  - Should to Could: REQ-04, REQ-06, NFR-06, NFR-09.
- Added the Won't list and CON-01 to CON-05.

### Version 0.1 (2026-10-14)

Authors: Dan Nassirharand, Maxwell Blevins

- Requirements first listed as FR-01 to FR-13 and NFR-01 to NFR-09 in [D2 part 4](Design_Diagrams/D2_Detailed_Design.md).

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

### Engineer (Software Engineer, cloud team): primary

- **Goal (US-01):** increase my output so I can contribute to key projects faster.
- **How this release serves the goal:** cited, automatic review of pull requests (UC-01) and a safe audit of the recipe database (UC-02).
- **Requirements:** REQ-01 to REQ-13, NFR-01 to NFR-11.

### Consumer using SmartHQ: secondary

- **Goal (US-02):** use the smart features of my appliance without interruption.
- **How this release serves the goal:** indirectly only, by decision (OQ-04).
- **Requirements:** none directly.

### Call center operator: hidden

- **Goal (US-03):** limit cloud failures in production so call volumes stay regulated.
- **How this release serves the goal:** indirectly only. The reliability requirements NFR-02, NFR-03, NFR-11, and REQ-12 limit failures of our own agents.
- **Requirements:** none directly, by decision (OQ-04).

---

## 4. Glossary

Every domain term and role name is defined here once. Both documents use these terms only as defined.

- **Advisor:** the project advisor chosen in Week 3, who reviews both documents.
- **Audit run:** one execution of C3 over the recipe database. Its status is RUNNING, HALTED, AWAITING_CONFIRMATION, or COMPLETE.
- **C1 PR Review Agent:** GitHub Copilot code review, configured with custom instructions to use the knowledge base and cite articles.
- **C2 Knowledge Base Index:** the tool that selects the knowledge base articles relevant to a pull request's changed files. It stores no index; it reads the knowledge base repository on each run.
- **C3 Recipe Audit Agent:** the service and command-line client that run audit runs, flag recipes, and delete confirmed recipes.
- **C4 Recipe Quality Scorer:** the component that scores one recipe from 0 to 100 against the ruleset.
- **Call center operator:** a GE Appliances employee who takes consumer calls when something fails.
- **Checkpoint:** the saved `last_cursor` of the last fully scored batch of an audit run, used to resume.
- **Citation:** the ID of the knowledge base article that backs a review comment.
- **Consumer:** a person who uses an appliance through the SmartHQ connected ecosystem.
- **Defect:** one failed ruleset rule on a recipe, with a `rule_id`, the `field`, and a `reason`.
- **Engineer:** a Software Engineer on the GE Appliances cloud team. Also the pull request author in UC-01 and the operator of C3 in UC-02.
- **Flagged recipe:** a recipe whose score is strictly below the threshold in an audit run.
- **G1 to G4:** shared pieces: G1 Authentication, G2 Notifications, G3 Configuration, G4 Logging (see D2 part 2.6).
- **Halted:** the audit run status after a failure or timeout. Nothing more is deleted until the Engineer resumes.
- **Keep exception:** an Engineer's decision that a flagged recipe stays. It applies to every later audit run.
- **Knowledge base:** the team's engineering documentation, migrated from Confluence into a read-only GitHub repository.
- **Knowledge base article:** one document in the knowledge base, with an ID and text.
- **Manual review label:** the pull request label "knowledge base unavailable - manual review required", applied when C1 cannot complete a review.
- **MoSCoW:** priority scheme: Must, Should, Could, Won't (this release).
- **Pilot engineer:** an Engineer outside the team who uses the system on real work during validation (tasks T11 and T12). For NFR-10, a pilot engineer is eligible only if they have never used the C3 command-line client, so pilot engineers are kept out of T10 testing and demonstrations before their T12 session.
- **Protected branch:** a branch that requires an approved review before merge.
- **Pull request (PR):** a GitHub request to merge changes into the protected branch.
- **Recipe:** a record in the recipe database with an ID, title, instructions (`steps`), `ingredients`, `domains`, and `cooktime`.
- **Recipe database:** the external DynamoDB table of about 200,000 recipes (an example size).
- **Removal log:** the record of each deleted recipe: run, recipe ID, score, defects, who confirmed, and when. It outlives the recipe.
- **Review pass:** one automated review of one version (`head_sha`) of a pull request by C1.
- **Ruleset:** the version-controlled file of rules, penalties, and the default threshold used by C4.
- **Sandbox:** a repository and a non-production AWS account used for testing, never production data.
- **Score:** an integer from 0 to 100 for one recipe: 100 minus the penalties of its defects, clamped.
- **SmartHQ:** GE Appliances' connected appliance ecosystem.
- **Threshold:** the score below which a recipe is flagged. Default 70; an example that is subject to change.
- **Triggering event:** the GitHub event that starts a review pass: `opened` when the pull request is created, or `synchronize` when a push updates it. Its time is the time GitHub records the event. NFR-01 and NFR-02 are measured from it.
- **Verdict:** the outcome of a review pass: `approved` (no violations remain) or `changes_requested`.

---

## 5. Functional Requirements

**How to read this part.** Each requirement is one "The system shall..." statement, followed by its priority, source, and verification method.

- **Verification methods:** **Test** (automated or scripted), **Demonstration** (run by a teammate or Advisor), **Inspection** (review of configuration or code), **Analysis** (calculation or measurement review).
- **Targets:** the numeric targets in parts 5 and 6 are final (OQ-06).
- **Scoring values:** the default threshold of 70 and the rule penalties are examples until OQ-02 is answered.

| ID     | Requirement                              | Priority |
| ------ | ---------------------------------------- | -------- |
| REQ-01 | Retrieve relevant articles               | Must     |
| REQ-02 | Cite every knowledge base finding        | Must     |
| REQ-03 | Re-review every push                     | Should   |
| REQ-04 | Dismiss informational comments           | Could    |
| REQ-05 | Fall back when the knowledge base is out | Should   |
| REQ-06 | Record the review outcome                | Could    |
| REQ-07 | Start and run an audit                   | Must     |
| REQ-08 | Flag low-scoring recipes                 | Must     |
| REQ-09 | Show flagged recipes before deletion     | Must     |
| REQ-10 | Delete only confirmed recipes            | Must     |
| REQ-11 | Honor keep exceptions                    | Should   |
| REQ-12 | Halt and resume safely                   | Should   |
| REQ-13 | Log before deleting                      | Must     |

### Pull request review (UC-01)

#### REQ-01: Retrieve relevant articles

The system shall retrieve from the knowledge base the articles relevant to a pull request's changed files, so that the mapped article is returned for at least 90% of a seeded set of 20 sandbox pull requests.

- **Priority:** Must
- **Source:** US-01, UC-01 step 2
- **Verification:** Test (seeded sandbox pull requests)

#### REQ-02: Cite every knowledge base finding

The system shall post an inline comment for each knowledge-base-based finding, each carrying the citation of an existing article (100% of such comments). Findings without an article are marked with a Low or Medium impact and confidence.

- **Priority:** Must
- **Source:** US-01, UC-01 step 3, AC-01.1
- **Verification:** Test; Inspection of posted comments

#### REQ-03: Re-review every push

The system shall start a new review pass on every push (`synchronize`), review only the latest `head_sha` (0 comments posted for an older one), and set the verdict `approved` when no violations remain.

- **Priority:** Should
- **Source:** US-01, UC-01 steps 4 and 5
- **Verification:** Test (two pushes in quick succession)

#### REQ-04: Dismiss informational comments

The system shall let the Engineer dismiss an informational comment with a one-line justification, after which the comment is shown as resolved (uses GitHub's native resolve).

- **Priority:** Could
- **Source:** US-01, UC-01 A2
- **Verification:** Demonstration

#### REQ-05: Fall back when the knowledge base is unreachable

The system shall, when the knowledge base is unreachable, apply the manual review label and mention the author in a comment in 100% of injected outages, and post no uncited review.

- **Priority:** Should
- **Source:** US-01, UC-01 E1, AC-01.2
- **Verification:** Test (simulated outage)

#### REQ-06: Record the review outcome

The system shall leave a recorded outcome (`approved` or `changes_requested`) with its citations on the pull request in 100% of completed review passes, with creation and approval timestamps available for the organization's cycle time measurement.

- **Priority:** Could
- **Source:** US-01, UC-01 postcondition
- **Verification:** Inspection; Demonstration

### Recipe audit (UC-02)

#### REQ-07: Start and run an audit

The system shall start an audit run on the Engineer's command, return a run ID immediately, and read and score the recipe database in batches until every non-excepted recipe is scored (`scored_count` equals the recipes read, less the keep exceptions and repeated IDs that are skipped).

- **Priority:** Must
- **Source:** US-01, UC-02 steps 1 and 2
- **Verification:** Test (sandbox recipe data)

#### REQ-08: Flag low-scoring recipes

The system shall flag every recipe that scores strictly below the threshold (default 70), each with at least one defect naming the `rule_id`, `field`, and `reason`. A score of 69 is flagged and 70 is not.

- **Priority:** Must
- **Source:** US-01, UC-02 step 3, AC-02.1
- **Verification:** Test (threshold boundary)

#### REQ-09: Show flagged recipes before deletion

The system shall show the Engineer every flagged recipe with its title, score, defects, and decision, at most 200 per page, before any deletion.

- **Priority:** Must
- **Source:** US-01, UC-02 step 4
- **Verification:** Demonstration; Test

#### REQ-10: Delete only confirmed recipes

The system shall delete only recipes whose IDs the Engineer confirmed (the client requires the typed word `yes`), and report the count removed. After a deletion, 100% of the reported IDs are absent from the recipe database.

- **Priority:** Must
- **Source:** US-01, UC-02 step 5
- **Verification:** Test; Demonstration

#### REQ-11: Honor keep exceptions

The system shall skip every recipe marked as a keep exception in 100% of later audit runs.

- **Priority:** Should
- **Source:** US-01, UC-02 A2
- **Verification:** Test (two successive runs)

#### REQ-12: Halt and resume safely

The system shall, on a batch read longer than 30 seconds or a failure, halt the run, keep the checkpoint, delete nothing further, and show the Engineer the halt reason and resume point. A resume continues from the checkpoint.

- **Priority:** Should
- **Source:** US-01, UC-02 E1, AC-02.2
- **Verification:** Test (injected timeout)

#### REQ-13: Log before deleting

The system shall write a removal log entry (run, recipe ID, score, defects, confirmer, UTC time) before deleting each recipe, for 100% of deletions. If the write fails, the recipe is not deleted.

- **Priority:** Must
- **Source:** US-01, UC-02 postcondition
- **Verification:** Test (injected write failure)

---

## 6. Non-Functional Requirements

Same layout and verification methods as part 5.

| ID     | Category            | Requirement                              | Priority |
| ------ | ------------------- | ---------------------------------------- | -------- |
| NFR-01 | Performance         | Review time                              | Must     |
| NFR-02 | Reliability         | Outage notice time                       | Should   |
| NFR-03 | Performance         | Batch read time limit                    | Should   |
| NFR-04 | Security            | Authorized access and confirmed deletion | Must     |
| NFR-05 | Security            | Sandbox-only testing                     | Should   |
| NFR-06 | Cost                | Model cost limits                        | Could    |
| NFR-07 | Reliability         | Repeatable scores                        | Should   |
| NFR-08 | Security            | Attributable deletions                   | Should   |
| NFR-09 | Performance (scale) | Scale to 100 times the volume            | Could    |
| NFR-10 | Usability           | First-time usability                     | Should   |
| NFR-11 | Reliability         | Crash recovery                           | Should   |

#### NFR-01: Review time (Performance)

The system shall post the review comments of a pass within 5 minutes of its triggering event (`opened` or `synchronize`), for every pass on a pull request within the review size limit. The first pass is therefore measured from pull request creation. A pass discarded under REQ-03 is not measured; the pass for the latest `head_sha` is. Any overrun is logged and counted as a failure of this requirement in pilot results.

- **Priority:** Must
- **Source:** US-01, AC-01.1, REQ-03
- **Verification:** Test (timed sandbox pull requests and later pushes); Analysis

#### NFR-02: Outage notice time (Reliability)

The system shall notify the author within 10 minutes of the triggering event (`opened` or `synchronize`) of a pass during which the knowledge base is unavailable (100% of injected outages).

- **Priority:** Should
- **Source:** US-01, AC-01.2, REQ-03
- **Verification:** Test (simulated outage on pull request creation and on a later push, timed)

#### NFR-03: Batch read time limit (Performance)

The system shall allow no recipe database batch read to run longer than 30 seconds; a read that exceeds it sets the run to HALTED.

- **Priority:** Should
- **Source:** US-01, AC-02.2
- **Verification:** Test (injected delay)

#### NFR-04: Authorized access and confirmed deletion (Security)

The system shall accept C3 requests only with a valid AWS SigV4 signature from an authorized IAM principal (others get 403 and reach nothing), and delete 0 recipes that were not listed in a confirmed deletion request.

- **Priority:** Must
- **Source:** US-01, UC-02 step 5, CON-01
- **Verification:** Test (unsigned and unlisted-ID requests); Inspection (IAM policy)

#### NFR-05: Sandbox-only testing (Security)

The system shall run all development testing of C1 and C3 in the sandbox: 0 test runs against the production recipe database or a production repository.

- **Priority:** Should
- **Source:** CON-03
- **Verification:** Inspection (account and repository configuration)

#### NFR-06: Model cost limits (Cost)

The system shall make 0 model calls to score recipes, and pass the model at most the article cap set in configuration per review pass (the cap value is fixed in task T3, due 2026-11-25).

- **Priority:** Could
- **Source:** CON-04
- **Verification:** Inspection; Test

#### NFR-07: Repeatable scores (Reliability)

The system shall give the same score and defects every time the same recipe is scored with the same ruleset version (100% across 2 repeated runs), and store the version on the audit run.

- **Priority:** Should
- **Source:** US-01, AC-02.1
- **Verification:** Test (repeated runs)

#### NFR-08: Attributable deletions (Security)

The system shall attribute 100% of deletions to a named confirmer and a `run_id` in both the removal log and the log entries.

- **Priority:** Should
- **Source:** US-01, CON-01
- **Verification:** Inspection of logs; Test

#### NFR-09: Scale to 100 times the volume (Performance, scale)

The system shall handle 100 times the current recipe volume (about 20,000,000 recipes) with no request over the 29-second gateway limit and no status page over 200 flagged recipes.

- **Priority:** Could
- **Source:** US-01, D1 part 3
- **Verification:** Analysis; Test (generated data)

#### NFR-10: First-time usability (Usability)

The system shall let an Engineer who has never used the client start an audit, review the flagged recipes, and delete one confirmed recipe in 10 minutes or less using only the built-in help, with at least 4 of 5 pilot engineers succeeding on the first try.

- **Priority:** Should
- **Source:** US-01, UC-02
- **Verification:** Test (observed pilot task, five eligible pilot engineers per A-03)

#### NFR-11: Crash recovery (Reliability)

The system shall, after a C3 restart or crash during a run, set the run to HALTED and, on resume, finish with the same flagged set as an uninterrupted run over the same recipe data (100%), re-scoring at most one batch. If a recipe in the re-scored batch changes between the crash and the resume, the run reflects the recipe as it is at resume. In every case, the flagged list matches `flagged_count`.

- **Priority:** Should
- **Source:** US-01, AC-02.2
- **Verification:** Test (kill the service mid-run, between a batch's flag writes and its checkpoint; repeat after repairing a flagged recipe of that batch before resuming)

---

## 7. MoSCoW Summary

**Count:** 24 requirements: 9 Must (37.5%), 11 Should, 4 Could. The Must set is the MVP: it shows both use cases end to end with citations, confirmed deletion, and a removal log, inside the 5-minute and authorization rules.

| Priority | Requirements                                                                           | Count |
| -------- | -------------------------------------------------------------------------------------- | ----- |
| Must     | REQ-01, REQ-02, REQ-07, REQ-08, REQ-09, REQ-10, REQ-13, NFR-01, NFR-04                 | 9     |
| Should   | REQ-03, REQ-05, REQ-11, REQ-12, NFR-02, NFR-03, NFR-05, NFR-07, NFR-08, NFR-10, NFR-11 | 11    |
| Could    | REQ-04, REQ-06, NFR-06, NFR-09                                                         | 4     |

**What the Must set covers**

- REQ-01, REQ-02, NFR-01: pull request review that cites knowledge base articles within 5 minutes.
- REQ-07, REQ-08, REQ-09, REQ-10, REQ-13: audit, flag, review, and delete with a log.
- NFR-04: authenticated access and no unconfirmed deletion.

### Won't (this release)

Written down so they cannot return without a change-log entry.

- **W-01:** Delete any recipe without the Engineer's confirmation. _Origin: D0 decision 1._
- **W-02:** Score recipes with an LLM. _Origin: D0 decision 2._
- **W-03:** Remove or edit a keep exception. _Origin: D2 Q2._
- **W-04:** Send push notifications (email or chat) for halted runs or fallbacks. The Engineer checks status. _Origin: D2 Q3, R9._
- **W-05:** Provide a web or graphical front end for C3. The command line is the only interface. _Origin: D1 part 6, R5._
- **W-06:** Review architecture or conventions that are not documented in the knowledge base. _Origin: peer review PR-2._
- **W-07:** Score recipes added after the cursor passed them in the same run. _Origin: D2 Q4._
- **W-08:** Run more than one audit run, or more than one C3 instance, at a time. _Origin: D1 part 3, EC-12._
- **W-09:** Commit code changes to a pull request or edit recipe content. The system only comments, flags, and (when confirmed) deletes. _Origin: D0 part 1._
- **W-10:** Compute review cycle time inside C1. The organization's existing measurement is used. _Origin: D2 part 4, REQ-06._
- **W-11:** Build consumer-facing SmartHQ features or call center tooling. _Origin: US-02, US-03._

---

## 8. Inherited Constraints

From the Week 3 [constraints essay](Constraints_Essay.md). What each rules out or forces is in [Technical_Specification.md](Technical_Specification.md) part 5.

- **CON-01 (Security):** Work with AI agents carries security risk, and an exploited vulnerability could harm the company socially, economically, legally, and professionally. The team shall align with security best practices.
- **CON-02 (Security):** The team shall use substantive code reviews.
- **CON-03 (Security):** The team shall test in isolated environments.
- **CON-04 (Professional):** Token usage and AI model subscriptions cost money. The team shall minimize cost through developer education, model selection, and communication with non-technical stakeholders.
- **CON-05 (Social):** The work must benefit developers and consumers, because that benefit supports GE Appliances' reputation and offsets the economic and security tradeoffs.

---

## 9. Assumptions and Open Questions

### Assumptions

- **A-01:** The knowledge base is already migrated from Confluence into a GitHub repository and is readable by C2 (UC-01 precondition).
  _Owner: Dan. Confirm by 2026-11-02._
- **A-02:** A sandbox repository, a non-production AWS account, and recipe data are provided through the Advisor (Plan R5).
  _Owner: Maxwell. Confirm by 2026-11-02._
- **A-03:** At least five eligible pilot engineers (see the glossary), plus one backup, are named through the Advisor, so NFR-10 can be measured with five first-time users of the C3 client. The same group can also pilot C1 and C2 in T11. Recruitment is planned in Plan R7.
  _Owner: Dan. Confirm by 2027-01-27 (milestone M18)._
- **A-04:** The recipe database has about 200,000 recipes. All scores, penalties, thresholds, caps, and batch sizes are examples held in configuration.
  _Owner: Maxwell. Confirm by 2026-11-25._
- **A-05:** One C3 instance runs at a time (W-08).
  _Owner: Maxwell. Confirm by 2026-11-02._

### Open questions

- **OQ-01:** Does the Copilot reviewer reliably call C2 and finish within 5 minutes (NFR-01)? Answered by task T1.
  _Owner: Dan. Answer due 2026-11-09._
- **OQ-02:** Which scoring rules, penalties, and threshold does the GE technical liaison accept as the quality standard (REQ-08)? Until answered, the threshold of 70 and the penalties are example values.
  _Owner: Maxwell. Answer due 2026-11-25._
- **OQ-07:** Is a push channel for halted runs needed after the pilot (W-04)?
  _Owner: Maxwell. Answer due 2027-04-06._

### Resolved questions (2026-10-28)

- **OQ-03: How is "increased output" (US-01) measured?**
  By review cycle time per pull request, taken from pull request timestamps by the organization's existing measurement (REQ-06, W-10). US-01 still has no Testable criterion in User_Stories.md. _Owner: Dan._
- **OQ-04: Do US-02 and US-03 get requirements of their own?**
  No. They are indirect stakeholder goals, and no requirement traces to them (W-11). _Owner: Dan._
- **OQ-05: Is the MoSCoW split in part 7 final?**
  Yes. The split is final: 9 Must, 11 Should, 4 Could. Any later change is logged in part 1. _Owner: Dan._
- **OQ-06: Are the numeric targets in parts 5 and 6 acceptable?**
  Yes. They are final: 90% of 20 seeded pull requests, 10 minutes, and 4 of 5 pilot engineers. A-03 was raised to five pilot engineers in v1.2 so the last target can be measured. _Owner: Maxwell._
