# Run Sheet: Agentic Upgrades Final Presentation

Team: GE Appliances, a Haier Company. Members: Dan Nassirharand, Maxwell Blevins. Advisor: Akshansh Gupta.
Deck: GE_Appliances_Presentation_Final (PPTX and PDF). Total planned time: 300 seconds (5:00).

## Roles

| Role | Person |
| ---- | ------ |
| Presenters | Dan Nassirharand (slides 1, 3, 5, 7, 9), Maxwell Blevins (slides 2, 4, 6, 8, 10) |
| Timekeeper | Maxwell Blevins |
| Slide advancer | Presenter of the current slide |
| Question answering | Owner listed per question below; the other member adds only if asked |
| Demo operator (Expo) | Dan runs the pull request flow, Maxwell runs the recipe audit flow |

## Slide run sheet

| Slide | Title | Speaker | Planned start | Seconds | Hand-off line |
| ----- | ----- | ------- | ------------- | ------- | ------------- |
| 1 | Agentic Upgrades | Dan | 0:00 | 15 | "Next, Maxwell will introduce the team." |
| 2 | Team and Advisor | Maxwell | 0:15 | 10 | "Dan will walk through the abstract." |
| 3 | Abstract | Dan | 0:25 | 30 | "Maxwell will show who this is for." |
| 4 | Who We Serve | Maxwell | 0:55 | 25 | "Dan will show how the system fits together." |
| 5 | System Overview (D0) | Dan | 1:20 | 60 | "Maxwell will cover our constraints." |
| 6 | Major Constraints | Maxwell | 2:20 | 30 | "Dan will review our progress." |
| 7 | Progress So Far | Dan | 2:50 | 30 | "Maxwell will cover what we finish this term." |
| 8 | By End of Fall Term | Maxwell | 3:20 | 25 | "Dan will explain how we split the work." |
| 9 | Division of Work | Dan | 3:45 | 25 | "Maxwell will show the Expo demo." |
| 10 | Expo Demo | Maxwell | 4:10 | 50 | "That is our presentation. We welcome questions." |
| | End | | 5:00 | 300 | |

## Anticipated questions

| # | Question | Owner | Short answer |
| - | -------- | ----- | ------------ |
| 1 | How do you stop the agent from deleting good recipes? | Maxwell | Nothing is deleted until the engineer types yes. The removal log row is written before the delete, and engineers can mark a recipe as a keep exception. |
| 2 | Can the review finish within 5 minutes? | Dan | That limit is a requirement. We measure the Copilot review time in feasibility work. If it fails, the pull request gets a manual review label. |
| 3 | What happens if the knowledge base is unavailable? | Dan | The pull request is labeled "knowledge base unavailable - manual review required" and the author is notified. |
| 4 | How is the recipe audit secured? | Maxwell | Every call is signed with AWS IAM. The service sits behind a private load balancer, and every deletion is tied to the engineer who confirmed it. |
| 5 | What if a database batch is slow or fails? | Maxwell | A batch over 30 seconds halts the run, saves a checkpoint at the last scored batch, and the engineer resumes from it. No more records are deleted. |
| 6 | How will you know it works? | Dan | We wrote 49 test cases and will validate with pilot engineers, including five first-time users of the command-line client. |
| 7 | What does this cost to run? | Dan | We cap the number of articles sent per review and set model cost limits as a requirement. |
| 8 | Do you use real GE data? | Maxwell | No. The repository is public, so tests use synthetic recipes and nothing from GE systems is committed. |
| 9 | What is left after this term? | Maxwell | Build in January and February, beta go/no-go on March 5, pilot results April 6, and the final demo April 22. |
| 10 | How did you split the work? | Dan | Dan owns the pull request review side and the command-line client. Maxwell owns the recipe audit side, scorer, and AWS stack. Each reviews the other's work. |

## Rehearsal log

Log at least one full-group timed rehearsal.

| Rehearsal | Date | Attendees | Total time | Notes |
| --------- | ---- | --------- | ---------- | ----- |
| 1 | 2026-11-16 | Dan, Maxwell | 5:30 (330 seconds) | 30 seconds over the 300-second plan; within the 5 to 10 minute video limit. Slide 5 ran longest. |

Per-section time (seconds):

| Slide | Planned | Rehearsal 1 |
| ----- | ------- | ----------- |
| 1 | 15 | 17 |
| 2 | 10 | 11 |
| 3 | 30 | 33 |
| 4 | 25 | 28 |
| 5 | 60 | 66 |
| 6 | 30 | 32 |
| 7 | 30 | 34 |
| 8 | 25 | 27 |
| 9 | 25 | 28 |
| 10 | 50 | 54 |
| Total | 300 | 330 |

## Recording and submission plan

| Item | Plan |
| ---- | ---- |
| Recording software | PowerPoint Record Slide Show. Record a short test clip first. |
| Video length | 5 to 10 minutes; target 5:00 to 5:30 |
| Links | Post the deck link and video link in our row of the Senior Design Team Table spreadsheet; Maxwell posts them |
| Canvas | Each student submits the slides and video with both names |
| LMS upload | One member uploads GE_Appliances_Presentation_Final before class on presentation day. No edits after the team is called. |
| Backup copy | Same deck on a USB drive or offline cloud storage, tested on a machine that is not ours |
| Peer reviews | Each student reviews at least three nearby teams, adds each review to that team's first empty Peer-Reviews column ending with their email, and submits the list of project names with comments on Canvas |
| During other teams' slots | Listen; be ready to ask a question |

## Expo demo and fallback

1. Dan opens a pull request in the sandbox repository; the agent posts comments citing knowledge base articles within 5 minutes.
2. Maxwell runs a recipe audit from the command line, reviews flagged recipes, types yes to delete one, and shows the removal log naming who confirmed.

If the demo fails, play the pre-recorded 90-second screen capture stored on the laptop. Use synthetic data only.
