# Technical Specification

**Project:** GE Appliances, a Haier Company: Agentic Upgrades
**Team members:** Dan Nassirharand, Maxwell Blevins
**Document version:** 1.1 (2026-10-28)
**Traces to:** [Requirements_Document.md](Requirements_Document.md) **version 1.1 (2026-10-28)**. Terms (Engineer, audit run, flagged recipe, and so on) are used as defined in its glossary.

Design sources, unchanged by this document: [D0](Design_Diagrams/D0_High_Level_Design.md), [D1](Design_Diagrams/D1_Detailed_Design.md), [D2](Design_Diagrams/D2_Detailed_Design.md) (2026-10-14).

---

## 1. Version and Change Log

| Version | Date       | Change                                                                                                                                                                                                                               | Author                            | Requirements version |
| ------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------- | -------------------- |
| 1.0     | 2026-10-28 | First consolidation of D0, D1, and D2 into one specification. No design change: D0 to D2 are as of 2026-10-14. Adds REQ, NFR, and CON IDs to the traceability matrix; adds the error body format and the table schemas in one place. | Dan Nassirharand, Maxwell Blevins | 1.0                  |
| 1.1     | 2026-10-28 | Re-traced to Requirements Document v1.1. Fixed the Week 10 test IDs as TC-001 to TC-029. Named the Advisor in part 7. No design change.                                                                                              | Dan Nassirharand, Maxwell Blevins | 1.1                  |

**Design changes since the Week 7 review:** none. The Week 7 peer review items (PR-1 to PR-4 in [Peer_Review_Notes.md](Design_Diagrams/Peer_Review_Notes.md)) are tracked there, not here. If a later change to D0 to D2 or to the Requirements Document is made, log it in this table with the new requirements version.

---

## 2. Architecture Summary

The system is two AI agent pipelines, drawn in the [D0 block diagram](Design_Diagrams/D0_High_Level_Design.md#2-block-diagram-d0). Component IDs are as in D0.

**Components.**

- **C1 PR Review Agent** and **C2 Knowledge Base Index** are GitHub configuration, not a hosted service. C1 is GitHub Copilot code review with custom instructions; C2 is a tool the reviewer is directed to call, which selects articles from the knowledge base repository.
- **C3 Recipe Audit Agent** is a Java Spring Boot service with a picocli command-line client. **C4 Recipe Quality Scorer** is linked into C3 and called as a function.
- **External:** GitHub, the knowledge base repository, the Recipe Database (DynamoDB), and the Engineer.

**Deployment model.** C1 and C2 run on GitHub and are versioned in the repository. C3 runs as one container on AWS ECS Fargate behind Amazon API Gateway with IAM (SigV4) authorization, defined in Terraform, with its audit tables in DynamoDB. The command-line client runs on the Engineer's machine and signs each request. Development uses a sandbox repository and a non-production AWS account (NFR-05).

**Data flow.** In [Flow A](Design_Diagrams/D0_High_Level_Design.md#flow-a-pull-request-review-uc-01), a pull request event starts C1 (I1). C1 asks C2 for articles (I2), C2 reads the knowledge base repository (I3), the model reviews the diff with those articles (I4), and C1 posts comments and a verdict to GitHub (I5). Each push repeats the flow. In [Flow B](Design_Diagrams/D0_High_Level_Design.md#flow-b-recipe-audit-uc-02), the Engineer starts an audit through the CLI over HTTPS (I6). C3 reads recipe batches from the Recipe Database (I7), sends each recipe to C4 (I8), and stores the run, flags, and checkpoint in the audit store (I9). The Engineer reviews the flagged recipes and confirms IDs; C3 writes the removal log, then deletes through I7.

**Pattern.** Pipeline for pull request review; client-server for the audit (D0 part 6).

---

## 3. Interfaces and Data Formats

### 3.1 Component boundaries

I1 to I8 are from [D0 part 4](Design_Diagrams/D0_High_Level_Design.md#4-interface-specification-table), revised by [D1](Design_Diagrams/D1_Detailed_Design.md) and [D2 part 2.7](Design_Diagrams/D2_Detailed_Design.md). I9, G1, and G4 are from D1 and D2.

| ID  | Boundary                     | Inputs                                                                                               | Outputs                                                                                                | Format and protocol                                                        | Error behavior                                                                                                                                              |
| --- | ---------------------------- | ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| I1  | GitHub to C1                 | `pr_number` int, `repo` string, `head_sha` string, `event_type` "opened" or "synchronize"            | None                                                                                                   | JSON; GitHub pull request configuration (no endpoint)                      | Other event types are ignored and logged. A newer `head_sha` discards the running pass.                                                                     |
| I2  | C1 to C2                     | `changed_files` string[]                                                                             | `articles` {`id`, `text`}[]                                                                            | JSON; tool call inside the Copilot reviewer                                | `KB_UNAVAILABLE`: C1 applies the manual review label and mentions the author. An empty list is valid.                                                       |
| I3  | C2 to knowledge base         | `repo` string                                                                                        | Articles as markdown                                                                                   | Markdown; GitHub REST API over HTTPS                                       | Fetch fails or is partial: `KB_UNAVAILABLE`, no partial results used.                                                                                       |
| I4  | C1 to model (GitHub Copilot) | `diff` string, `articles` object[] (at most the configured cap)                                      | `findings` {`file`, `line`, `text`, `article_id`}[]                                                    | JSON; HTTPS inside GitHub                                                  | Timeout or failure: retry once, then the manual review label.                                                                                               |
| I5  | C1 to GitHub                 | `comments` {`file`, `line`, `body`}[], `label` string, `status` "approved" or "changes_requested"    | `review_id` int                                                                                        | JSON; GitHub REST API over HTTPS                                           | Post fails: retry, then the manual review label (up to 3 label retries with backoff, then log).                                                             |
| I6  | CLI to C3                    | The seven API operations in 3.2                                                                      | JSON bodies in 3.2                                                                                     | JSON over HTTPS REST, SigV4 signed (G1)                                    | CLI exit codes: 0 success; 2 invalid arguments (nothing sent); 3 rejected with 4xx (code and message printed); 4 service or authentication failure.         |
| I7  | C3 to Recipe Database        | `cursor` string, `delete_ids` string[]; batch size and timeout from configuration                    | `recipes` {`recipeId`, `title`, `steps`, `ingredients`, `domains`, `cooktime`}[], `next_cursor` string | JSON items; AWS SDK for Java v2 (DynamoDB)                                 | Read over 30 s or fails: run HALTED (`BATCH_TIMEOUT` or `DB_ERROR`), `last_cursor` unchanged, no further deletion. Access denied: HALTED (`ACCESS_DENIED`). |
| I8  | C3 to C4                     | `recipeId`, `steps` string[], `ingredients` string[], `ruleset_version` string                       | `score` int 0 to 100, `defects` {`rule_id`, `field`, `reason`}[]                                       | Java function call, in process                                             | Missing or wrong-typed field: score 0 with defect `UNPARSEABLE`. `UNKNOWN_RULESET`: run HALTED (`CONFIG`).                                                  |
| I9  | C3 to audit store            | Start run (conditional write), write checkpoint, upsert flags, load exceptions, write removal log    | The stored items in 3.4                                                                                | DynamoDB items; AWS SDK for Java v2                                        | Failure: `STORE_ERROR`. A running audit is HALTED; deletion is refused. A removal log write failure means the recipe is not deleted.                        |
| G1  | CLI to API Gateway to C3     | AWS credentials from the standard credential chain                                                   | Authorized request, or 403 `FORBIDDEN`                                                                 | SigV4 over HTTPS; IAM authorization                                        | 403 stops the request at the gateway. Expired credentials: the CLI re-signs once, then exits 4.                                                             |
| G4  | Components to log service    | `timestamp`, `component`, `level`, `event`, `correlation_id` (`run_id` or `pr_number`), `error_code` | None                                                                                                   | Structured log entry, allowlisted fields; no recipe content or credentials | A logging failure never fails the caller; entries go to local output. Distinct from the removal log.                                                        |

### 3.2 C3 endpoint table

All endpoints: HTTPS, JSON, `/v1` prefix, SigV4 required. The major version is in the path; adding optional fields is not a breaking change (D1 part 5).

| Endpoint                                  | Inputs                                                                 | Success output                                                                                                | Errors                                                                                                                                                            |
| ----------------------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `POST /v1/audit-runs`                     | `threshold` int 0 to 100, optional, default 70                         | 202: `run_id`, `status` "RUNNING", `threshold`                                                                | 400 `INVALID_THRESHOLD`; 409 `RUN_IN_PROGRESS`                                                                                                                    |
| `GET /v1/audit-runs/{run_id}`             | `limit` int 1 to 200 (default 50), `page_token` optional               | 200: `status`, `scored_count`, `flagged_count`, `halt_reason`, `resume_point`, `flagged`[], `next_page_token` | 404 `RUN_NOT_FOUND`; 400 `INVALID_LIMIT`; 503 `STORE_UNAVAILABLE`                                                                                                 |
| `POST /v1/audit-runs/{run_id}/resume`     | `run_id`                                                               | 202: `status` "RUNNING"                                                                                       | 404 `RUN_NOT_FOUND`; 409 `RUN_NOT_HALTED`                                                                                                                         |
| `POST /v1/audit-runs/{run_id}/exceptions` | `recipeIds` string[] (1 to 200), `reason` string (1 to 500 characters) | 200: `excepted_ids`                                                                                           | 400 `INVALID_REQUEST`; 404 `RUN_NOT_FOUND`; 422 `RECIPE_NOT_FLAGGED`                                                                                              |
| `POST /v1/audit-runs/{run_id}/deletions`  | `recipeIds` string[] (1 to 200)                                        | 200: `deleted_count`, `deleted_ids` (unlisted IDs were skipped or failed and can be retried)                  | 400 `INVALID_REQUEST`; 404 `RUN_NOT_FOUND`; 409 `RUN_NOT_AWAITING_CONFIRMATION`; 422 `RECIPE_NOT_DELETABLE`; 503 `LOG_WRITE_FAILED` (only if nothing was deleted) |

The CLI maps these to `audit start`, `audit status`, `audit resume`, `audit except`, and `audit delete` ([D2 part 2.2](Design_Diagrams/D2_Detailed_Design.md)). `audit delete` asks the Engineer to type `yes` before any request is sent.

### 3.3 Message payloads (JSON Schema)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$defs": {
    "defect": {
      "type": "object",
      "required": ["rule_id", "field", "reason"],
      "properties": {
        "rule_id": { "type": "string" },
        "field": { "type": "string" },
        "reason": { "type": "string" }
      }
    },
    "flaggedRecipe": {
      "type": "object",
      "required": ["recipeId", "title", "score", "defects", "decision"],
      "properties": {
        "recipeId": { "type": "string" },
        "title": { "type": "string" },
        "score": { "type": "integer", "minimum": 0, "maximum": 100 },
        "defects": { "type": "array", "items": { "$ref": "#/$defs/defect" } },
        "decision": { "enum": ["PENDING", "EXCEPTED", "DELETED"] }
      }
    },
    "startRequest": {
      "type": "object",
      "properties": {
        "threshold": {
          "type": "integer",
          "minimum": 0,
          "maximum": 100,
          "default": 70
        }
      }
    },
    "idsRequest": {
      "type": "object",
      "required": ["recipeIds"],
      "properties": {
        "recipeIds": {
          "type": "array",
          "items": { "type": "string" },
          "minItems": 1,
          "maxItems": 200
        },
        "reason": { "type": "string", "minLength": 1, "maxLength": 500 }
      }
    },
    "statusResponse": {
      "type": "object",
      "required": ["status", "scored_count", "flagged_count", "flagged"],
      "properties": {
        "status": {
          "enum": ["RUNNING", "HALTED", "AWAITING_CONFIRMATION", "COMPLETE"]
        },
        "scored_count": { "type": "integer" },
        "flagged_count": { "type": "integer" },
        "halt_reason": { "type": ["string", "null"] },
        "resume_point": { "type": ["string", "null"] },
        "flagged": {
          "type": "array",
          "maxItems": 200,
          "items": { "$ref": "#/$defs/flaggedRecipe" }
        },
        "next_page_token": { "type": ["string", "null"] }
      }
    },
    "deletionsResponse": {
      "type": "object",
      "required": ["deleted_count", "deleted_ids"],
      "properties": {
        "deleted_count": { "type": "integer" },
        "deleted_ids": { "type": "array", "items": { "type": "string" } }
      }
    },
    "error": {
      "type": "object",
      "required": ["error_code", "message"],
      "properties": {
        "error_code": { "type": "string" },
        "message": { "type": "string" }
      }
    }
  }
}
```

`idsRequest` is used by `/exceptions` (with `reason` required) and `/deletions` (without). The `error` body (code and message) is the shape the CLI prints (D2 part 2.2); it applies to every 4xx and 5xx response. Example payloads for I4 and I7 are in [D0 part 4](Design_Diagrams/D0_High_Level_Design.md#4-interface-specification-table); a deletions example is in [D1 part 5](Design_Diagrams/D1_Detailed_Design.md#5-api-contract).

### 3.4 Database schema (Amazon DynamoDB)

Entity attributes and relationships are in the [D1 data model](Design_Diagrams/D1_Detailed_Design.md#2-data-model); `AUDIT_RUN.halt_reason` is added in [D2 part 2.1](Design_Diagrams/D2_Detailed_Design.md).

| Table                      | Partition key | Sort key   | Other attributes                                                                                                                                   | Access                                                         |
| -------------------------- | ------------- | ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| `AUDIT_RUN`                | `run_id`      | None       | `status`, `threshold`, `ruleset_version`, `started_by`, `started_at` (ISO 8601 UTC), `last_cursor`, `scored_count`, `flagged_count`, `halt_reason` | Conditional write on start (one active run); checkpoint update |
| `FLAGGED_RECIPE`           | `run_id`      | `recipeId` | `title`, `score`, `defects`, `decision`                                                                                                            | One query returns a run's flags; keyed upsert                  |
| `KEEP_EXCEPTION`           | `recipeId`    | None       | `reason`                                                                                                                                           | Loaded whole at run start                                      |
| `REMOVAL_LOG`              | `run_id`      | `recipeId` | `score`, `defects`, `confirmed_by`, `removed_at` (ISO 8601 UTC)                                                                                    | Written before each delete                                     |
| Recipe Database (external) | `recipeId`    | None       | `title`, `steps`, `ingredients`, `domains`, `cooktime` (minutes). C3 reads and deletes; it stores no copy.                                         | Paged reads by `cursor`; delete by ID                          |

The ruleset is a version-controlled file, not a table: `RULESET` {`version`, `default_threshold`} contains `RULE` {`rule_id`, `field`, `check`, `penalty`} ([D2 F3](Design_Diagrams/D2_Detailed_Design.md#23-c4-recipe-quality-scorer)).

---

## 4. Technology Stack

Full comparisons and rejected alternatives are in [D1 part 6](Design_Diagrams/D1_Detailed_Design.md#6-technology-choices-with-justification).

| Layer                      | Choice                                                     | Justification                                                                                                                            |
| -------------------------- | ---------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| PR review runtime (C1, C2) | GitHub pull request configuration and REST API             | Native events, comments, labels, and verdicts give REQ-02, REQ-03, REQ-05; nothing of ours to host or secure (CON-01).                   |
| Model (C1)                 | GitHub Copilot, model chosen after testing                 | Keeps the provider inside GitHub (CON-01); billed under the organization's plan (CON-04); time to be measured against NFR-01.            |
| Knowledge base store (C2)  | GitHub repository                                          | Documentation is already migrated there (UC-01); read on each run so there is no index to go stale (REQ-01, REQ-05).                     |
| Backend (C3, C4)           | Java with Spring Boot (Apache-2.0)                         | Team skill and mutual code review (CON-02); light workload, easily meets NFR-03 and NFR-09.                                              |
| Client (C3)                | picocli and `java.net.http`; AWS SDK v2 signer             | Command line is the only interface (W-05); help output supports NFR-10; typed confirmation implements REQ-10 and NFR-04.                 |
| API authentication         | Amazon API Gateway with IAM (SigV4) authorization          | Authentication is not hand-rolled (NFR-04, CON-01); 403 before C3 is reached.                                                            |
| Hosting (C3)               | AWS ECS Fargate (one task)                                 | Long-running container, no 15-minute function limit (REQ-07, NFR-09); restart recovery with checkpoint (NFR-11); single instance (W-08). |
| Data store                 | Amazon DynamoDB, AWS SDK for Java v2                       | Same store as the Recipe Database; key lookups only (REQ-07, REQ-13); per-request billing (CON-04).                                      |
| Scoring rules (C4)         | Plain Java code; rule penalties in a Jackson YAML ruleset  | No model call per recipe (NFR-06, CON-04); repeatable scores (NFR-07); weights change without code changes (REQ-08).                     |
| Infrastructure as code     | Terraform                                                  | Non-production and production stacks reviewed as code (NFR-05, CON-02, CON-03).                                                          |
| Logging                    | Structured entries to the AWS log service (D2 F5, node CW) | Traceable deletions and logged overruns (NFR-08, NFR-01); never blocks the caller.                                                       |
| Testing                    | JUnit 5; sandbox repository and non-production AWS account | Threshold boundary tests (REQ-08); isolated testing (NFR-05, CON-03); verification for every REQ and NFR.                                |

---

## 5. Known Constraints

### 5.1 Inherited constraints (CON)

| ID     | Constraint (see Requirements Document part 8) | What it rules out                                                                     | What it forces                                                                                                                                                              |
| ------ | --------------------------------------------- | ------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CON-01 | Security best practices for agents            | Agents with broad permissions; automatic deletion (W-01); hand-rolled authentication. | IAM authorization (G1); a task role limited to the recipe and audit tables; typed confirmation at the client and the API; GitHub-managed minimum permissions for C1 and C2. |
| CON-02 | Substantive code reviews                      | Merging an unreviewed change.                                                         | The other member reviews every task before it closes ([Plan.md](Planning/Plan.md) R3); pull requests against the repository.                                                |
| CON-03 | Testing in isolated environments              | Testing against the production recipe database or a production repository.            | A sandbox repository and a non-production AWS account (Terraform stack, task T2); sandbox data only.                                                                        |
| CON-04 | Minimize token and subscription cost          | A model call per recipe (W-02); sending the whole knowledge base to the model.        | Fixed-rule scoring; only relevant articles, up to a configured cap; per-request storage billing; the model chosen after cost and quality testing.                           |
| CON-05 | Benefit developers and consumers              | Shipping features that are not validated with users.                                  | Pilot validation with pilot engineers (tasks T11 and T12) and recorded results for every REQ and NFR.                                                                       |

### 5.2 Technical limits carried from D1 and D2

| Limit                                                       | Consequence                                                                               |
| ----------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| API Gateway allows 29 seconds per request.                  | Start and resume return 202 immediately; progress is read by polling status.              |
| Copilot review pass time is unmeasured (D2 Q1, OQ-01).      | NFR-01 is verified by task T1 and pilot results; the manual review label is the fallback. |
| One C3 instance at a time (W-08).                           | On startup, any RUNNING run becomes HALTED (`RESTART`) and the Engineer resumes it.       |
| Requests are limited to 200 IDs; status pages to 200 items. | The client splits IDs into groups of 200 and follows page tokens.                         |
| All thresholds, caps, and sizes are examples (D2 Q8).       | They live in configuration (G3), not code.                                                |

---

## 6. Traceability Matrix

One row per REQ, NFR, and CON ID. Status: Designed (in D0 to D2), In progress, or Planned (not yet designed or built). Test case IDs are the TC numbers assigned for the Week 10 test plan (A10, due 2026-11-04), one per row. No design component is blank, and there are no ORPHANED rows. The two process constraints without a D0 to D2 element (CON-02, CON-05) are tied to the plan tasks that carry them and marked Planned.

| Req ID | Description                                                            | Design Component (D0/D1/D2)                                                          | Status   | Test Case |
| ------ | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | -------- | --------- |
| REQ-01 | Retrieve relevant knowledge base articles for the changed files        | C2; D0 I2, I3; D1 `select_articles`; D2 2.5                                          | Designed | TC-001    |
| REQ-02 | Cited inline comments; uncited findings marked Low or Medium           | C1; D0 I4, I5; D1 review pass; D2 F4                                                 | Designed | TC-002    |
| REQ-03 | Re-review each push on the latest `head_sha`; approve when clean       | C1; D0 I1 (`synchronize`), decision 5; D2 F4, EC-02                                  | Designed | TC-003    |
| REQ-04 | Dismiss informational comment with a one-line justification            | C1; D0 I5; D1 UC-01 A2 (GitHub native resolve)                                       | Designed | TC-004    |
| REQ-05 | Manual review label and author mention when knowledge base unreachable | C1; D0 I2; D2 F4 fallback, G2, EC-05, EC-09                                          | Designed | TC-005    |
| REQ-06 | Recorded outcome with citations on the pull request                    | C1; D0 I5 (`status`); D2 part 4 (cycle time is organization-measured)                | Designed | TC-006    |
| REQ-07 | Start an audit run; score the recipe database in batches               | C3, C4; D0 I6, I7, I8; D1 `POST /v1/audit-runs`, `AUDIT_RUN`; D2 F1, 2.2             | Designed | TC-007    |
| REQ-08 | Flag recipes strictly below the threshold with field-level defects     | C4; D0 I8; D1 `FLAGGED_RECIPE`, scoring algorithm; D2 2.3, EC-20                     | Designed | TC-008    |
| REQ-09 | Show flagged recipes with defects, paged, before deletion              | C3 and CLI; D0 I6; D1 `GET /v1/audit-runs/{run_id}`; D2 2.2                          | Designed | TC-009    |
| REQ-10 | Delete only confirmed IDs; report the count                            | C3 and CLI; D1 `POST .../deletions`; D2 F2, 2.2 `audit delete`, EC-14                | Designed | TC-010    |
| REQ-11 | Skip keep exceptions in later runs                                     | C3; D1 `KEEP_EXCEPTION`, `POST .../exceptions`; D2 EC-14                             | Designed | TC-011    |
| REQ-12 | Halt on timeout or failure; keep checkpoint; show resume point         | C3; D0 I7; D1 `last_cursor`; D2 F1, `halt_reason`, `resume_point`, EC-17             | Designed | TC-012    |
| REQ-13 | Removal log entry before each delete                                   | C3; D1 `REMOVAL_LOG`; D2 F2, EC-18                                                   | Designed | TC-013    |
| NFR-01 | Comments within 5 minutes of pull request creation                     | C1; D0 Flow A timing; D2 F4, EC-03, G4 (Q1 open)                                     | Designed | TC-014    |
| NFR-02 | Author notified within 10 minutes when knowledge base fails            | C1; D2 G2, EC-29                                                                     | Designed | TC-015    |
| NFR-03 | No batch read longer than 30 seconds                                   | C3; D0 I7; D2 F1, G3 timeout setting, EC-17                                          | Designed | TC-016    |
| NFR-04 | Authorized requests only; no unconfirmed deletion                      | G1; API Gateway IAM; C3 task role; D2 F2, 2.2 confirmation prompt, EC-26             | Designed | TC-017    |
| NFR-05 | All testing in the sandbox                                             | Terraform stacks (D1 part 6); D2 part 4 gap (stack not yet specified)                | Planned  | TC-018    |
| NFR-06 | No model call per recipe; article cap                                  | D0 decision 2, decision 3; D2 G3 article cap, EC-06                                  | Designed | TC-019    |
| NFR-07 | Repeatable scores; ruleset version stored                              | C4; D1 `ruleset_version`; D2 F3 `RULESET`, EC-21                                     | Designed | TC-020    |
| NFR-08 | Deletions attributable to a confirmer and run                          | `REMOVAL_LOG` (`confirmed_by`); D2 G4 `correlation_id`                               | Designed | TC-021    |
| NFR-09 | 100 times volume without gateway timeout or oversized pages            | C3; D1 part 3 complexity; cursor, checkpoint, paged status (D2 part 4)               | Designed | TC-022    |
| NFR-10 | First-time Engineer completes the audit task in 10 minutes             | C3 CLI; D2 2.2 command group and `--help`; usability session not yet designed        | Planned  | TC-023    |
| NFR-11 | Crash recovery with an identical flagged set                           | C3; D1 keyed upserts and checkpoint; D2 F1, EC-13                                    | Designed | TC-024    |
| CON-01 | Security best practices                                                | G1; D0 part 6 minimal permissions; D2 F2                                             | Designed | TC-025    |
| CON-02 | Substantive code reviews                                               | Delivery process: [Plan.md](Planning/Plan.md) R3 and section 4 (no D0 to D2 element) | Planned  | TC-026    |
| CON-03 | Isolated testing environments                                          | Terraform non-production stack (D1 part 6; task T2); sandbox repository              | Planned  | TC-027    |
| CON-04 | Minimize token and subscription cost                                   | D0 decisions 2 and 3; D1 part 6 Copilot and on-demand billing; D2 G3 article cap     | Designed | TC-028    |
| CON-05 | Benefit developers and consumers                                       | D0 part 1 goal; pilot validation, tasks T11 and T12 (no D0 to D2 element)            | Planned  | TC-029    |

---

## 7. Advisor Review Record

**Status:** the review date, discussion, and resulting changes are TBD. They are recorded here, with a change log entry for each resulting change, once the review is held.

| Item               | Record                                                         |
| ------------------ | -------------------------------------------------------------- |
| Advisor            | Akshansh Gupta                                                 |
| Date of review     | TBD                                                            |
| Documents reviewed | Requirements_Document.md v1.1, Technical_Specification.md v1.1 |
| Attendees          | TBD                                                            |

**What was discussed**

| #   | Topic                                     | Advisor feedback |
| --- | ----------------------------------------- | ---------------- |
| 1   | MoSCoW split and the Won't list           | TBD              |
| 2   | Numeric targets in REQ and NFR statements | TBD              |
| 3   | Traceability matrix and ORPHANED check    | TBD              |

**Changes produced**

TBD.
