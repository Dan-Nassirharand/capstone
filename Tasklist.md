# Task List

**Team:** GE Appliances, a Haier Company
**Team members:** Dan Nassirharand, Maxwell Blevins

T1. Investigate GitHub Copilot review pass time and whether it reliably calls the C2 Knowledge Base Index, against the 5-minute limit (C1 PR Review Agent). Owner: Dan.

T2. Develop the Terraform stack for the C3 Recipe Audit Agent (ECS Fargate, API Gateway with G1 Authentication, DynamoDB audit tables) in a non-production AWS account. Owner: Maxwell.

T3. Develop the C2 Knowledge Base Index article selection and the C1 PR Review Agent custom instructions that require article citations. Owner: Dan.

T4. Develop the C4 Recipe Quality Scorer and its ruleset, with JUnit 5 tests at the threshold boundary. Owner: Maxwell.

T5. Develop and test the C1 PR Review Agent review pass in a sandbox repository, including the stale `head_sha` discard and the G2 Notifications manual review label fallback. Owner: Dan.

T6. Develop the C3 Recipe Audit Agent audit loop with the `last_cursor` checkpoint and the HALTED and resume states. Owner: Maxwell.

T7. Develop the G3 Configuration loader and G4 Logging for the C1 PR Review Agent and the C3 Recipe Audit Agent. Owner: Dan.

T8. Develop the C3 Recipe Audit Agent REST API with the removal log and keep exceptions. Owner: Maxwell.

T9. Develop the C3 Recipe Audit Agent command-line client with the deletion confirmation prompt. Owner: Dan.

T10. Test the C3 Recipe Audit Agent and C4 Recipe Quality Scorer end to end, including halt and resume and the deletion error paths. Owner: Maxwell.

T11. Validate the C1 PR Review Agent and C2 Knowledge Base Index with pilot engineers and document the results against FR-01 to FR-06, NFR-01, NFR-02, and NFR-06. Owner: Dan.

T12. Validate the C3 Recipe Audit Agent and C4 Recipe Quality Scorer with pilot engineers and document the results against FR-07 to FR-13, NFR-03 to NFR-05, and NFR-07 to NFR-09. Owner: Maxwell.

---

_Scope and timeline are subject to change based on business need, as this is an industry project._
