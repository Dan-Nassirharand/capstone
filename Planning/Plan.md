# Task List, Milestones, Timeline, and Effort Matrix

**Team:** GE Appliances, a Haier Company
**Team members:** Dan Nassirharand, Maxwell Blevins
**Project:** Agentic Upgrades

Assumptions: spring semester Jan 11 to Apr 30, 2027 (spring break Mar 15 to 19); no work Thanksgiving (Nov 26 to 27), exams (Dec 7 to 11), or winter break (Dec 12 to Jan 10). Dates are 2026 (fall) and 2027 (spring).

## 1. Task List

See [Tasklist.md](../Tasklist.md). T1 to T10, five tasks per member.

## 2. Milestones

| ID   | Milestone                                          | Date   | Demonstrates                                    | Verified how / by                              | Status   |
| ---- | -------------------------------------------------- | ------ | ----------------------------------------------- | ---------------------------------------------- | -------- |
| M1   | A1 Professional Biography                          | Sep 2  | Member experience                               | Canvas grade / Instructor                      | Complete |
| M2   | A2 Team Formation and Project Description          | Sep 9  | Team and topic                                  | Canvas grade / Instructor                      | Complete |
| M3   | A3 Advisor Selection and Constraints Essay         | Sep 18 | Advisor and design constraints                  | Canvas submission / Instructor                 | Complete |
| M4   | A4 User Stories and Use Cases                      | Sep 28 | UC-01, UC-02 and acceptance criteria            | Canvas submission / Instructor                 | Complete |
| M5   | A5 Design D0                                       | Oct 2  | Block diagram and interfaces                    | Canvas submission / Instructor                 | Complete |
| M6   | A6 Design D1                                       | Oct 8  | Data model, algorithms, API                     | Canvas submission / Instructor                 | Complete |
| M7   | A7 Design D2 and Peer Review Notes                 | Oct 14 | Finished design, peer feedback                  | Canvas submission / Instructor, peer teams     | Complete |
| M8   | A8 Task List and Plan (this document)              | Oct 28 | Owned, dated plan                               | Canvas submission / Instructor                 | In progress |
| M9   | A9 Requirements and Technical Specification        | Oct 30 | FR and NFR list with specifications             | Canvas submission / Instructor, Advisor        | Planned  |
| M10  | A10 Test and Validation Plan Draft                 | Nov 4  | A test for each requirement                     | Canvas submission / Instructor, Advisor        | Planned  |
| M11  | Feasibility done (T1, T2)                          | Nov 10 | Review timing known; AWS stack deploys          | T1 report and `terraform apply` / teammate     | Planned  |
| M12  | A11 Ethics and Responsible-Use Statement           | Nov 11 | Responsible use of AI and deletion              | Canvas submission / Instructor                 | Planned  |
| M13  | A12 and A13 Presentation, Video, Peer Reviews      | Nov 18 | Team can present design and plan                | Video and advisor feedback / Instructor, Advisor | Planned |
| M14  | Scorer and article selection working (T3, T4)      | Nov 25 | C4 tests pass; C2 cites an article              | Test run and sandbox PR / teammate             | Planned  |
| M15  | A14 Design Report, Notebook, Self Assessment       | Dec 2  | Full fall design record                         | Canvas submission / Instructor, Advisor        | Planned  |
| M16  | A15 Project Advisor Assessments                    | Dec 8  | Advisor evaluation of fall work                 | Advisor submits / Advisor                      | Planned  |
| M17  | Spring kickoff, plan updated                       | Jan 12 | Plan matches spring schedules                   | Updated Table 1 / Advisor                      | Planned  |
| M18  | Review pass and audit loop running (T5, T6)        | Jan 26 | C1 posts cited review; C3 checkpoints a batch   | Sandbox demo / teammate                        | Planned  |
| M19  | Code complete (T7, T8)                             | Feb 11 | All components built                            | D2 element checklist / Advisor                 | Planned  |
| M20  | **Major:** Beta go/no-go (T10)                     | Mar 5  | Safe for pilot: no unconfirmed deletes, 5 min   | Test report vs NFR-01 to NFR-04 / Advisor      | Planned  |
| M21  | First pilot results                                | Mar 26 | Real PRs reviewed; one audit reviewed           | Pilot log / Advisor                            | Planned  |
| M22  | **Major:** Pilot complete (T9)                     | Apr 6  | Results per requirement                         | T9 results table / Advisor                     | Planned  |
| M23  | **Major:** Final presentation (date TBC)           | Apr 22 | Working system                                  | Live demo / Instructor, Advisor                | Planned  |
| M24  | **Major:** Spring design report (date TBC)         | Apr 30 | Complete implementation record                  | Report submitted / Instructor, Advisor         | Planned  |

## 3. Table 1: Timeline

Buffer is 15 to 20 percent of each task's work days, added before its finish date. Risk IDs refer to section 6.

| ID  | Statement                                   | Owner   | Start  | Buffer       | Finish | Depends on | Risk   |
| --- | ------------------------------------------- | ------- | ------ | ------------ | ------ | ---------- | ------ |
| T1  | Investigate Copilot review timing           | Dan     | Nov 2  | 1 day (20%)  | Nov 9  | None       | R1     |
| T2  | Develop Terraform stack for C3              | Maxwell | Nov 2  | 1 day (17%)  | Nov 10 | None       | R5     |
| T3  | Develop C2 article selection, instructions  | Dan     | Nov 18 | 1 day (20%)  | Nov 25 | T1         | R1     |
| T4  | Develop C4 scorer and ruleset               | Maxwell | Nov 18 | 1 day (20%)  | Nov 25 | None       |        |
| T5  | Develop and test C1 review pass             | Dan     | Jan 11 | 2 days (20%) | Jan 26 | T3         | R1     |
| T6  | Develop C3 audit loop                       | Maxwell | Jan 11 | 2 days (20%) | Jan 26 | T2, T4     | R2     |
| T7  | Develop G3 configuration and G4 logging     | Dan     | Jan 27 | 2 days (20%) | Feb 9  | T5, T6     | R3     |
| T8  | Develop C3 REST API and CLI                 | Maxwell | Jan 27 | 2 days (20%) | Feb 11 | T6         | R2     |
| T10 | Test C3 and C4 end to end                   | Maxwell | Feb 12 | 2 days (20%) | Feb 25 | T7, T8     | R2, R3 |
| T9  | Validate with pilot engineers               | Dan     | Mar 22 | 2 days (20%) | Apr 6  | T5, T10    | R3, R5 |

| Protected block | Dates          | Before                  |
| --------------- | -------------- | ----------------------- |
| PB1             | Nov 11 to 17   | M13 Presentation        |
| PB2             | Nov 30 to Dec 1 | M15 Design Report      |
| PB3             | Feb 26 to Mar 4 | M20 Beta go/no-go      |
| PB4             | Apr 7 to 21    | M23 Final presentation  |

Milestone completion dates are in the Date column of section 2.

**Critical path:** T2 → T6 → T8 → T10 → T9. A slip on this path uses PB3 first, then spring break.

## 4. Table 2: Effort Matrix (hours)

`*` marks the member of primary responsibility.

| ID    | Dan   | Maxwell | Total |
| ----- | ----- | ------- | ----- |
| T1    | 8*    | 2       | 10    |
| T2    | 3     | 12*     | 15    |
| T3    | 10*   | 3       | 13    |
| T4    | 2     | 9*      | 11    |
| T5    | 22*   | 5       | 27    |
| T6    | 5     | 20*     | 25    |
| T7    | 11*   | 3       | 14    |
| T8    | 6     | 23*     | 29    |
| T9    | 17*   | 4       | 21    |
| T10   | 3     | 14*     | 17    |
| Total | 87    | 95      | 182   |

**Imbalance:** Maxwell has 8 more hours, and Dan has no task from Feb 10 to Mar 21. This is fine as we both have other tasks to complete respecitively at our jobs. Both members are also willing to assist on their partners tasks if need be.

## 5. Work Breakdown Structure

```text
Agentic Upgrades
├── 1 Feasibility (Fall)
│   ├── T1 Investigate Copilot review timing
│   └── T2 Develop Terraform stack for C3
├── 2 Early components (Fall)
│   ├── T3 Develop C2 article selection, instructions
│   └── T4 Develop C4 scorer and ruleset
├── 3 Build (Spring)
│   ├── T5 Develop and test C1 review pass
│   ├── T6 Develop C3 audit loop
│   ├── T7 Develop G3 configuration and G4 logging
│   └── T8 Develop C3 REST API and CLI
└── 4 Test and validation (Spring)
    ├── T10 Test C3 and C4 end to end
    └── T9 Validate with pilot engineers
```

## 6. Risk Register

| ID | Category  | Risk                                                        | Likelihood | Impact | Strategy | Mitigation                                                   | Owner   | Trigger                              |
| -- | --------- | ----------------------------------------------------------- | ---------- | ------ | -------- | ------------------------------------------------------------ | ------- | ------------------------------------ |
| R1 | Technical | Copilot review exceeds 5 minutes or skips C2                | Medium     | High   | Reduce   | Measure first (T1); cap articles; manual review label        | Dan     | T1 median pass over 4 minutes        |
| R2 | Technical | Recipe batch reads exceed the 30-second timeout             | Medium     | High   | Reduce   | Measure in T6; batch size in config; PB3 reserve             | Maxwell | Any sandbox batch over 20 seconds    |
| R3 | Team      | Co-op rotation or illness reduces a member's spring hours   | Medium     | High   | Reduce   | 20% task buffers; teammate reviews every task and can take over | Both | Under 8 hours per week for two weeks |
| R4 | Team      | Course deliverables and exams crowd out fall tasks          | High       | Medium | Avoid    | Only T1 to T4 in fall; PB1 and PB2; no work during exams     | Maxwell | A draft not ready 2 days before due  |
| R5 | External  | Delay getting AWS account, sandbox repo, or recipe data     | Medium     | High   | Reduce   | Request on Nov 2 via advisor; fallback personal account with generated data | Maxwell | No access by Nov 10 (M11) |
