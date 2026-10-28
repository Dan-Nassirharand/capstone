# Task List, Milestones, Timeline, and Effort Matrix

**Team:** GE Appliances, a Haier Company
**Team members:** Dan Nassirharand, Maxwell Blevins
**Project:** Agentic Upgrades

**Disclaimer:** Scope and timeline are subject to change based on business need, as this is an industry project.

## Planning Assumptions

- **Fall calendar:** term Aug 24 to Dec 12, 2026. Course week 1 starts Monday Aug 31, so week 9 is Oct 26 to Nov 1 and week 14 is Nov 30 to Dec 6.
- **Spring calendar:** term Jan 11 to May 1, 2027. Classes end Apr 23. Exam week is Apr 24 to 29.
- **No-work days:** Veterans Day (Nov 11), Thanksgiving weekend (Nov 26 to 29), fall exam week (Dec 6 to 12), winter break (Dec 13 to Jan 10), Martin Luther King Jr. Day (Jan 18), and spring break (Mar 15 to 21). No task is scheduled in spring exam week (Apr 24 to 29); only the M25 report is.
- **Reading days** (Oct 12 and Nov 3) are work days. Only classes are suspended.
- **Weekly hours:** each member plans 10 to 12 hours per week on the project. Each member confirms this. The R3 trigger is under 8 hours per week.
- **Capacity:** task effort in Table 2 is less than the weekly hours available. The remaining time goes to coordinating with the Advisor, coordinating with each other, class assignments, and the buffer periods in Table 1.
- **Definitions:** a work day is a weekday that is not a no-work day. Start is the first work day of a task. Buffer is reserve time at the end of a task, 15 to 20 percent of its work days. Finish is the last day of the buffer.
- **Years:** fall dates are 2026 and spring dates are 2027.

## 1. Task List

See [Tasklist.md](../Tasklist.md). The list has 12 tasks, T1 to T12, with six per member. Each name below is the Tasklist statement shortened. The ID and owner are the same in every section. In the tables, C1 is the C1 PR Review Agent, C2 the C2 Knowledge Base Index, C3 the C3 Recipe Audit Agent, and C4 the C4 Recipe Quality Scorer.

| ID  | Name                                                                                                          | Owner   |
| --- | ------------------------------------------------------------------------------------------------------------- | ------- |
| T1  | Investigate Copilot review pass time against the 5-minute limit (C1 PR Review Agent, C2 Knowledge Base Index) | Dan     |
| T2  | Develop the Terraform stack for the C3 Recipe Audit Agent                                                     | Maxwell |
| T3  | Develop C2 Knowledge Base Index article selection and C1 PR Review Agent custom instructions                  | Dan     |
| T4  | Develop the C4 Recipe Quality Scorer and its ruleset                                                          | Maxwell |
| T5  | Develop and test the C1 PR Review Agent review pass                                                           | Dan     |
| T6  | Develop the C3 Recipe Audit Agent audit loop                                                                  | Maxwell |
| T7  | Develop the G3 Configuration loader and G4 Logging                                                            | Dan     |
| T8  | Develop the C3 Recipe Audit Agent REST API, removal log, and keep exceptions                                  | Maxwell |
| T9  | Develop the C3 Recipe Audit Agent command-line client                                                         | Dan     |
| T10 | Test the C3 Recipe Audit Agent and C4 Recipe Quality Scorer end to end                                        | Maxwell |
| T11 | Validate the C1 PR Review Agent and C2 Knowledge Base Index with pilot engineers                              | Dan     |
| T12 | Validate the C3 Recipe Audit Agent and C4 Recipe Quality Scorer with pilot engineers                          | Maxwell |

## 2. Milestones

Each date is the due date (course deliverables) or the planned completion date (internal milestones). "Teammate" means the member who does not own the task. W1 to W14 are course weeks; "Exam" is exam week; "Spring" has no course week.

| ID  | Milestone                                              | Week   | Date   | Demonstrates                                                                                                                                   | Verified how / by                                                  | Status      |
| --- | ------------------------------------------------------ | ------ | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ | ----------- |
| M1  | A1 Professional Biography                              | W1     | Sep 2  | Each member's education, skills, and experience are documented                                                                                 | Canvas grade / Instructor                                          | Complete    |
| M2  | A2 Team Formation and Project Description              | W2     | Sep 9  | The team, its two members, and the project topic area are defined                                                                              | Canvas grade / Instructor                                          | Complete    |
| M3  | A3 Advisor Selection and Constraints Essay             | W3     | Sep 18 | An advisor is chosen, and the design constraints are written up                                                                                | Canvas submission / Instructor                                     | Complete    |
| M4  | A4 User Stories and Use Cases                          | W5     | Sep 28 | Use cases UC-01 (pull request review) and UC-02 (recipe audit) have acceptance criteria                                                        | Canvas submission / Instructor                                     | Complete    |
| M5  | A5 Design D0                                           | W5     | Oct 2  | The block diagram, component responsibilities, and interfaces I1 to I8 are defined                                                             | Canvas submission / Instructor                                     | Complete    |
| M6  | A6 Design D1                                           | W6     | Oct 8  | The C3 data model, the scoring and review algorithms, the API contract, and technology choices are defined                                     | Canvas submission / Instructor                                     | Complete    |
| M7  | A7 Design D2 and Peer Review Notes                     | W7     | Oct 14 | The design is finished (lifecycle, CLI, ruleset, glue, edge cases, traceability) and peer feedback is recorded                                 | Canvas submission / Instructor, peer teams                         | Complete    |
| M8  | A8 Task List and Plan (this document)                  | W9     | Oct 28 | Every task has an owner, dates, buffer, and effort; risks are tracked                                                                          | Canvas submission / Instructor                                     | In progress |
| M9  | A9 Requirements and Technical Specification            | W9     | Oct 30 | The functional and non-functional requirements (FR, NFR) are listed with specifications                                                        | Canvas submission / Instructor, Advisor                            | Planned     |
| M10 | A10 Test and Validation Plan Draft                     | W10    | Nov 4  | A draft test plan covers every FR and NFR, with at least one test each                                                                         | Canvas submission / Instructor, Advisor                            | Planned     |
| M11 | Feasibility done (T1, T2)                              | W11    | Nov 10 | The T1 report states the Copilot review pass time against the 5-minute limit; the C3 Terraform stack deploys in the non-production AWS account | T1 report and a successful `terraform apply` / Teammate            | Planned     |
| M12 | A11 Ethics and Responsible-Use Statement               | W11    | Nov 11 | Responsible use of AI and of deletion                                                                                                          | Canvas submission / Instructor                                     | Planned     |
| M13 | A12 and A13 Presentation, Video, Peer Reviews          | W12    | Nov 18 | The team can present its design and plan                                                                                                       | Video and advisor feedback / Instructor, Advisor                   | Planned     |
| M14 | Scorer and article selection working (T3, T4)          | W13    | Nov 25 | C4 boundary tests pass; a C2 article is cited in a sandbox pull request review                                                                 | Test run and sandbox pull request / Teammate                       | Planned     |
| M15 | A14 Design Report, Notebook, Self Assessment           | W14    | Dec 2  | Full fall design record; setup and handoff notes for winter break are in the repository (R6)                                                   | Canvas submission / Instructor, Advisor                            | Planned     |
| M16 | A15 Project Advisor Assessments                        | Exam   | Dec 8  | Advisor evaluation of fall work                                                                                                                | Advisor submits / Advisor                                          | Planned     |
| M17 | Spring kickoff, plan updated                           | Spring | Jan 12 | Plan matches both members' spring schedules                                                                                                    | Updated Table 1 reviewed / Advisor                                 | Planned     |
| M18 | Review pass and audit loop running (T5, T6)            | Spring | Jan 27 | C1 posts a review that cites an article; C3 saves a checkpoint after one batch; two pilot engineers are named (R7)                             | Sandbox demo / Teammate; pilot engineer names / Advisor            | Planned     |
| M19 | Code complete (T7, T8, T9)                             | Spring | Feb 12 | All components in D0 are built                                                                                                                 | D2 element checklist / Advisor                                     | Planned     |
| M20 | Integration tested, C1 and C2 piloted (T10, T11)       | Spring | Feb 26 | C3 and C4 pass the halt, resume, and deletion error-path tests; pilot engineers reviewed real pull requests                                    | Test report and T11 results table / Teammate                       | Planned     |
| M21 | **Major:** Beta go/no-go                               | Spring | Mar 5  | C3 is safe to pilot: it never deletes an unconfirmed recipe (NFR-04), and C1 posts comments within 5 minutes (NFR-01)                          | Test report against NFR-01 to NFR-04 / Advisor decides go or no-go | Planned     |
| M22 | First pilot results (T12 in progress)                  | Spring | Mar 26 | A pilot engineer has reviewed the flagged recipes of one complete audit                                                                        | Pilot log / Advisor                                                | Planned     |
| M23 | **Major:** Pilot complete (T11, T12)                   | Spring | Apr 6  | Results for every FR and NFR                                                                                                                   | T11 and T12 results tables / Advisor                               | Planned     |
| M24 | **Major:** Final presentation (date to be confirmed)   | Spring | Apr 22 | Working system                                                                                                                                 | Live demo / Instructor, Advisor                                    | Planned     |
| M25 | **Major:** Spring design report (date to be confirmed) | Spring | Apr 30 | Complete implementation record                                                                                                                 | Report submitted / Instructor, Advisor                             | Planned     |

Spacing: fall milestones are at most 14 days apart (M7 to M8). The longest spring gap is 21 days (M21 to M22, which includes spring break). The 35 days from M16 to M17 are exam week and winter break, with no work scheduled. M12 falls on Veterans Day (a holiday), so the team finishes it by Nov 10. Oral presentations are in week 13 (Nov 23 to 25) and week 14 (Nov 30 to Dec 4); the team's slot is to be confirmed.

## 3. Table 1: Timeline

Work days follow the definitions in Planning Assumptions. Finish includes the buffer. Risk IDs refer to section 6; the Risk column lists the risks that can delay each task.

| ID  | Name                                               | Owner   | Start  | Work days | Buffer         | Finish | Depends on | Risk       |
| --- | -------------------------------------------------- | ------- | ------ | --------- | -------------- | ------ | ---------- | ---------- |
| T1  | Investigate Copilot review pass time (C1, C2)      | Dan     | Nov 2  | 5         | 1 day (20%)    | Nov 9  | None       | R1, R5     |
| T2  | Develop the Terraform stack for C3                 | Maxwell | Nov 2  | 6         | 1 day (17%)    | Nov 10 | None       | R5         |
| T3  | Develop C2 article selection and C1 instructions   | Dan     | Nov 18 | 5         | 1 day (20%)    | Nov 25 | T1         | R1         |
| T4  | Develop the C4 Recipe Quality Scorer and ruleset   | Maxwell | Nov 18 | 5         | 1 day (20%)    | Nov 25 | None       | R4         |
| T5  | Develop and test the C1 review pass                | Dan     | Jan 11 | 10        | 2 days (20%)   | Jan 27 | T3         | R1, R6     |
| T6  | Develop the C3 audit loop                          | Maxwell | Jan 11 | 10        | 2 days (20%)   | Jan 27 | T2, T4     | R2, R5, R6 |
| T7  | Develop the G3 Configuration loader and G4 Logging | Dan     | Jan 28 | 10        | 2 days (20%)   | Feb 12 | T5, T6     | R3         |
| T8  | Develop the C3 REST API, removal log, exceptions   | Maxwell | Jan 28 | 10        | 2 days (20%)   | Feb 12 | T6         | R2         |
| T9  | Develop the C3 command-line client                 | Dan     | Jan 28 | 10        | 2 days (20%)   | Feb 12 | T6         | R3         |
| T10 | Test C3 and C4 end to end                          | Maxwell | Feb 15 | 8         | 1.5 days (19%) | Feb 26 | T7, T8, T9 | R2, R3     |
| T11 | Validate C1 and C2 with pilot engineers            | Dan     | Feb 15 | 8         | 1.5 days (19%) | Feb 26 | T5, T7     | R1, R7     |
| T12 | Validate C3 and C4 with pilot engineers            | Maxwell | Mar 8  | 10        | 2 days (20%)   | Mar 30 | T10, M21   | R3, R7     |

T5 and T6 finish Jan 27 because Jan 18 is a holiday. T12 has no work days in spring break (Mar 15 to 21): its 10 work days are Mar 8 to 12 and Mar 22 to 26, and the buffer is Mar 29 and 30. T10 and T11 have 8 work days (Feb 15 to 19 and Feb 22 to 24) and a 1.5-day buffer (all of Feb 25 and the morning of Feb 26), so they finish at midday on Feb 26.

| Protected block | Dates           | Before                   | Use                                                           |
| --------------- | --------------- | ------------------------ | ------------------------------------------------------------- |
| PB1             | Nov 12 to 17    | M13 Presentation         | Rehearse and finish the video (Nov 11 is a holiday)           |
| PB2             | Nov 30 to Dec 1 | M15 Design Report        | Finish and proofread the report                               |
| PB3             | Feb 27 to Mar 4 | M21 Beta go/no-go        | Fix T10 and T11 findings (work days Mar 1 to 4); no new tasks |
| PB4             | Mar 31 to Apr 5 | M23 Pilot complete       | Write the T11 and T12 results tables                          |
| PB5             | Apr 7 to 21     | M24 Final presentation   | Fix pilot findings and prepare the demo                       |
| PB6             | Apr 23 to 29    | M25 Spring design report | Write the report (includes spring exam week)                  |

| No-work period   | Dates            | Effect on the timeline                                     | Risk |
| ---------------- | ---------------- | ---------------------------------------------------------- | ---- |
| Veterans Day     | Nov 11           | M12 is finished by Nov 10; PB1 starts Nov 12               | R4   |
| Thanksgiving     | Nov 26 to 29     | T3 and T4 finish Nov 25; no task is scheduled              | R4   |
| Fall exam week   | Dec 6 to 12      | No task; only M16 (the advisor submits)                    | R4   |
| Winter break     | Dec 13 to Jan 10 | No task; T5 and T6 start Jan 11 after an environment check | R6   |
| MLK Day          | Jan 18           | T5 and T6 finish Jan 27 instead of Jan 26                  | R6   |
| Spring break     | Mar 15 to 21     | T12 pauses for the week                                    | R3   |
| Spring exam week | Apr 24 to 29     | No task; only M25 report writing (PB6)                     | R3   |

Milestone rows (planned completion dates; the meaning of each milestone is in section 2):

| ID       | Milestone                                     | Planned completion | Gated by      | Protected block |
| -------- | --------------------------------------------- | ------------------ | ------------- | --------------- |
| M1 to M7 | A1 to A7 course deliverables                  | Sep 2 to Oct 14    | None          | Complete        |
| M8       | A8 Task List and Plan                         | Oct 28             | None          |                 |
| M9       | A9 Requirements and Technical Specification   | Oct 30             | None          |                 |
| M10      | A10 Test and Validation Plan Draft            | Nov 4              | None          |                 |
| M11      | Feasibility done                              | Nov 10             | T1, T2        |                 |
| M12      | A11 Ethics and Responsible-Use Statement      | Nov 11             | None          |                 |
| M13      | A12 and A13 Presentation, Video, Peer Reviews | Nov 18             | None          | PB1             |
| M14      | Scorer and article selection working          | Nov 25             | T3, T4        |                 |
| M15      | A14 Design Report, Notebook, Self Assessment  | Dec 2              | None          | PB2             |
| M16      | A15 Project Advisor Assessments               | Dec 8              | None          |                 |
| M17      | Spring kickoff, plan updated                  | Jan 12             | None          |                 |
| M18      | Review pass and audit loop running            | Jan 27             | T5, T6        |                 |
| M19      | Code complete                                 | Feb 12             | T7, T8, T9    |                 |
| M20      | Integration tested, C1 and C2 piloted         | Feb 26             | T10, T11      |                 |
| M21      | **Major:** Beta go/no-go                      | Mar 5              | M20           | PB3             |
| M22      | First pilot results                           | Mar 26             | T12 (partial) |                 |
| M23      | **Major:** Pilot complete                     | Apr 6              | T11, T12      | PB4             |
| M24      | **Major:** Final presentation                 | Apr 22             | M23           | PB5             |
| M25      | **Major:** Spring design report               | Apr 30             | M23           | PB6             |

**Critical path (no float):** T6 → T8 → T10 → T12. T7 and T9 also finish on Feb 12 and gate T10, so they have no float either. T1 to T4 are not on the path because T5 and T6 cannot start before Jan 11. If a task on the path runs late, the reserves are used in this order: the task's own buffer, then PB3 (for T10) or PB4 (for T12). T12 uses spring break only if the Advisor agrees. T9 is built against the D2 API contract and does not wait for T8; T10 integrates them.

## 4. Table 2: Effort Matrix (hours)

Each task is estimated by its owner with three numbers: optimistic (O), most likely (M), and pessimistic (P) total hours. Expected hours are E = (O + 4M + P) / 6, rounded to whole hours. The O, M, and P values are drafts for the owners to confirm.

| ID  | O   | M   | P   | E   |
| --- | --- | --- | --- | --- |
| T1  | 8   | 10  | 14  | 10  |
| T2  | 11  | 15  | 20  | 15  |
| T3  | 9   | 13  | 19  | 13  |
| T4  | 8   | 11  | 16  | 11  |
| T5  | 20  | 27  | 40  | 28  |
| T6  | 19  | 25  | 38  | 26  |
| T7  | 10  | 14  | 20  | 14  |
| T8  | 14  | 20  | 26  | 20  |
| T9  | 8   | 12  | 18  | 12  |
| T10 | 12  | 17  | 24  | 17  |
| T11 | 12  | 16  | 26  | 17  |
| T12 | 8   | 12  | 17  | 12  |

`*` marks the member of primary responsibility. Each row sums to E.

| ID    | Name                                               | Dan  | Maxwell | Total |
| ----- | -------------------------------------------------- | ---- | ------- | ----- |
| T1    | Investigate Copilot review pass time (C1, C2)      | 8\*  | 2       | 10    |
| T2    | Develop the Terraform stack for C3                 | 3    | 12\*    | 15    |
| T3    | Develop C2 article selection and C1 instructions   | 10\* | 3       | 13    |
| T4    | Develop the C4 Recipe Quality Scorer and ruleset   | 2    | 9\*     | 11    |
| T5    | Develop and test the C1 review pass                | 23\* | 5       | 28    |
| T6    | Develop the C3 audit loop                          | 5    | 21\*    | 26    |
| T7    | Develop the G3 Configuration loader and G4 Logging | 11\* | 3       | 14    |
| T8    | Develop the C3 REST API, removal log, exceptions   | 4    | 16\*    | 20    |
| T9    | Develop the C3 command-line client                 | 9\*  | 3       | 12    |
| T10   | Test C3 and C4 end to end                          | 3    | 14\*    | 17    |
| T11   | Validate C1 and C2 with pilot engineers            | 13\* | 4       | 17    |
| T12   | Validate C3 and C4 with pilot engineers            | 3    | 9\*     | 12    |
| Total |                                                    | 94   | 101     | 195   |

**Imbalance:** Maxwell has 7 more hours than Dan (101 and 94, or 52% and 48%). Maxwell also owns the longest chain (T6, T8, T10, T12). Dan has no task of his own after Feb 26, apart from 3 hours on T12. Rebalancing plan: if T8 has not finished its 10 work days by Feb 10, or Maxwell works under 8 hours per week for two weeks (the R3 trigger), Dan takes over the T10 deletion error-path tests (about 4 hours). The totals then become 98 for Dan and 97 for Maxwell, and Maxwell still has the largest share of T10. While Dan has no task of his own, he reviews each critical-path task before it closes. Either member helps on the other's tasks when asked.

## 5. Work Breakdown Structure

```text
Agentic Upgrades
├── 1 Feasibility (Fall)
│   ├── T1 Investigate Copilot review pass time (C1, C2) - Dan
│   └── T2 Develop the Terraform stack for C3 - Maxwell
├── 2 Early components (Fall)
│   ├── T3 Develop C2 article selection and C1 instructions - Dan
│   └── T4 Develop the C4 Recipe Quality Scorer and ruleset - Maxwell
├── 3 Build (Spring)
│   ├── T5 Develop and test the C1 review pass - Dan
│   ├── T6 Develop the C3 audit loop - Maxwell
│   ├── T7 Develop the G3 Configuration loader and G4 Logging - Dan
│   ├── T8 Develop the C3 REST API, removal log, exceptions - Maxwell
│   └── T9 Develop the C3 command-line client - Dan
└── 4 Test and validation (Spring)
    ├── T10 Test C3 and C4 end to end - Maxwell
    ├── T11 Validate C1 and C2 with pilot engineers - Dan
    └── T12 Validate C3 and C4 with pilot engineers - Maxwell
```

## 6. Risk Register

| ID  | Category  | Risk                                                                                                   | Likelihood | Impact | Strategy | Mitigation                                                                                                                              | Owner   | Trigger                                                          |
| --- | --------- | ------------------------------------------------------------------------------------------------------ | ---------- | ------ | -------- | --------------------------------------------------------------------------------------------------------------------------------------- | ------- | ---------------------------------------------------------------- |
| R1  | Technical | Copilot review exceeds 5 minutes or does not call C2                                                   | Medium     | High   | Reduce   | Measure first (T1); cap the number of articles; apply the manual review label on failure                                                | Dan     | T1 median review pass over 4 minutes                             |
| R2  | Technical | Recipe batch reads exceed the 30-second timeout                                                        | Medium     | High   | Reduce   | Measure batch read time on the first 2 days of T6; batch size in configuration; PB3 reserve                                             | Maxwell | Any sandbox batch read over 20 seconds                           |
| R3  | Team      | Co-op rotation or illness reduces a member's spring hours                                              | Medium     | High   | Reduce   | 20% task buffers; PB3 and PB4; the other member reviews every task; T10 error-path tests move to Dan (section 4)                        | Dan     | A member works under 8 hours per week for two weeks              |
| R4  | Team      | Course deliverables, Veterans Day, Thanksgiving, and fall exam week (Dec 6 to 12) crowd out fall tasks | High       | Medium | Avoid    | Only T1 to T4 in fall; PB1 and PB2; no task on Nov 11, Nov 26 to 29, or Dec 6 to 12                                                     | Maxwell | A draft is not ready 2 days before its due date                  |
| R5  | External  | Delay getting AWS account, sandbox repo, or recipe data                                                | Medium     | High   | Reduce   | Request all three via the Advisor by Oct 30, before T1 and T2 start on Nov 2; fallback: personal AWS account with generated recipe data | Maxwell | No sandbox repo or AWS access on Nov 2; no recipe data on Jan 11 |
| R6  | Team      | Winter break (Dec 13 to Jan 10) loses context and access, so T5 and T6 start late                      | Medium     | Medium | Reduce   | Setup and handoff notes in the repository by M15 (Dec 2); environment check on Jan 11; plan refresh at M17 (Jan 12)                     | Dan     | Terraform apply or Copilot review fails on Jan 11                |
| R7  | External  | Pilot engineers unavailable for T11 (Feb 15 to 26) or T12 (Mar 8 to 30)                                | Medium     | High   | Reduce   | Name two pilot engineers via the Advisor by M18 (Jan 27); fallback: replay recent real pull requests and a recipe snapshot              | Dan     | No confirmed pilot engineer by Feb 12                            |

_Scope and timeline are subject to change based on business need, as this is an industry project._
