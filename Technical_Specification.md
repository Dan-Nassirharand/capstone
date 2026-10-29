# Technical Specification

- **Project:** GE Appliances, a Haier Company: Agentic Upgrades
- **Team members:** Dan Nassirharand, Maxwell Blevins
- **Document version:** 1.4 (2026-10-29)
- **Traces to:** [Requirements_Document.md](Requirements_Document.md) **version 1.4 (2026-10-29)**. Terms (Engineer, audit run, flagged recipe, triggering event, and so on) are used as defined in its glossary.
- **Design sources:** [D0](Design_Diagrams/D0_High_Level_Design.md), [D1](Design_Diagrams/D1_Detailed_Design.md), [D2](Design_Diagrams/D2_Detailed_Design.md) (2026-10-14). Where this specification and those documents differ, this specification governs. Every difference is listed in part 1.

**Contents**

1. [Version and Change Log](#1-version-and-change-log)
2. [Architecture Summary](#2-architecture-summary)
3. [Interfaces and Data Formats](#3-interfaces-and-data-formats)
4. [Technology Stack](#4-technology-stack)
5. [Known Constraints](#5-known-constraints)
6. [Traceability Matrix](#6-traceability-matrix)
7. [Advisor Review Record](#7-advisor-review-record)

---

## 1. Version and Change Log

Each version names the Requirements Document version it traces to. Any later change to D0 to D2 or to the Requirements Document is logged here with the new requirements version.

### Version 1.4 (2026-10-29), traces to Requirements v1.4

Authors: Dan Nassirharand, Maxwell Blevins

These changes answer five review comments on 3.7 and 3.8. A full review of both documents also found two gaps, which are fixed here: a flagged recipe that was repaired could never leave `PENDING`, and REQ-07 and NFR-11 had wording that did not match the design.

- **Re-traced to Requirements v1.4.** REQ-07 states which recipes `scored_count` leaves out. NFR-11 drops US-03 as a source.
- **Exact flag counts on retry (3.7).** A flag write now succeeds again for a row of the current batch, so a retry after a lost response no longer fails and under-counts. `flagged_count` grows by the number of distinct recipes the batch flagged.
- **Final checkpoint leaves `RUNNING` (3.7).** When the cursor is exhausted, the checkpoint also sets `AWAITING_CONFIRMATION` in the same update. Before, a crash between the two writes made resume scan from the beginning and inflate `scored_count`. A completion check after that checkpoint covers a run with nothing to delete.
- **Complete cleanup before replay (3.7).** The cleanup query is strongly consistent and reads every page before any replay. The resumed worker also waits out a quiet period so a late write from the stopped worker cannot slip in (3.5).
- **Deletes match what was scored (3.8).** The delete is conditional on the recipe's `steps` and `ingredients` equalling what was scored. A recipe repaired after the re-score is not deleted. Added the decision `RESOLVED` for a flagged recipe that is repaired or already gone; without it, such a recipe stayed `PENDING` and the run could never complete.
- **Unknown delete outcome (3.8).** A timeout or lost response keeps the claim and removal-log row, and a reconciliation step settles them. Only a definitive rejection releases the claim, so a recipe that was actually deleted never loses its attribution.
- **Added:** optional `not_deleted` in the deletions response, event `RECIPE_DELETE_UNCERTAIN`, and the attributes `halted_at`, `claimed_at`, and the setting `settle_seconds`.
- **Supersedes** the v1.3 "Flag writes" and "Resume" rules in 3.7, the delete, release, and orphaned-claim rules in 3.8, and the D1 part 3 audit loop, which set `AWAITING_CONFIRMATION` in a separate step after the cursor was exhausted.

### Version 1.3 (2026-10-29), traces to Requirements v1.3

Authors: Dan Nassirharand, Maxwell Blevins

- **Re-traced to Requirements v1.3.** NFR-11 now states its guarantee when recipes change during recovery.
- **Design change: replay of an interrupted batch (NFR-11).** Flag rows are tagged with the batch that wrote them. On every resume, rows from the uncommitted batch are deleted before that batch is re-scored, and the status endpoint lists only committed rows (3.7). This supersedes D1 part 3 ("re-scoring a batch after a crash changes nothing") and the v1.2 note in 3.5. A recipe repaired before resume used to keep its stale flag row while the recomputed `flagged_count` dropped it.
- **Design change: per-recipe deletion claim (NFR-08, REQ-13).** Before deleting a recipe, a request claims it on its `FLAGGED_RECIPE` row and writes the removal log in one transaction. Only the claim holder can delete the recipe or release the claim, and a released claim removes its log row (3.8). This supersedes D1 part 3 ("the log row stays and a retry overwrites it"). Two requests for the same recipe could both pass the D2 F2 checks, and the losing request could overwrite the winner's removal-log row.

### Version 1.2 (2026-10-28), traces to Requirements v1.2

Authors: Dan Nassirharand, Maxwell Blevins

- **Re-traced to Requirements v1.2.** NFR-01 and NFR-02 are now measured from the triggering event, not pull request creation. The part 6 descriptions are updated to match.
- **Design change: deletion attribution (NFR-08).** G4 log entries gain `confirmed_by` and `recipe_id`, both required on every deletion event. `confirmed_by` and `REMOVAL_LOG.confirmed_by` come from the IAM principal that API Gateway verified, never from the request body (3.6). This supersedes the G4 field list in D2 part 2.6, which had no confirmer, so a `run_id` alone could not tell apart two Engineers confirming deletions in the same run.
- **Design change: one active run at a time (W-08).** A shared lock item, written in the same transaction as the new run, admits one audit run at a time. Resume and checkpoint writes are conditional on the run's status. The lock is released only when the run completes, and it survives restarts (3.5). This supersedes the "conditional write on `AUDIT_RUN`" in D1 part 3 and D2 EC-12. That write checked only the new run's own key, so two starts with different `run_id` values could both succeed.
- **Layout.** Interfaces, endpoints, tables, stack, and constraints rewritten as readable sections. No other content changed.

### Version 1.1 (2026-10-28), traces to Requirements v1.1

Authors: Dan Nassirharand, Maxwell Blevins

- Re-traced to Requirements Document v1.1.
- Fixed the Week 10 test IDs as TC-001 to TC-029.
- Named the Advisor in part 7.
- No design change.

### Version 1.0 (2026-10-28), traces to Requirements v1.0

Authors: Dan Nassirharand, Maxwell Blevins

- First consolidation of D0, D1, and D2 into one specification. No design change: D0 to D2 are as of 2026-10-14.
- Adds REQ, NFR, and CON IDs to the traceability matrix.
- Adds the error body format and the table schemas in one place.

**Week 7 peer review.** The peer review items (PR-1 to PR-4) are tracked in [Peer_Review_Notes.md](Design_Diagrams/Peer_Review_Notes.md), not here. The only design changes since that review are the ones listed under v1.2 to v1.4.

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

I1 to I8 are from [D0 part 4](Design_Diagrams/D0_High_Level_Design.md#4-interface-specification-table), revised by [D1](Design_Diagrams/D1_Detailed_Design.md) and [D2 part 2.7](Design_Diagrams/D2_Detailed_Design.md). I9, G1, and G4 are from D1 and D2; I9 and G4 are revised by this specification (part 1, v1.2 to v1.4).

| ID  | Boundary                  | Format and protocol                                      |
| --- | ------------------------- | -------------------------------------------------------- |
| I1  | GitHub → C1               | JSON; GitHub pull request configuration (no endpoint)    |
| I2  | C1 → C2                   | JSON; tool call inside the Copilot reviewer              |
| I3  | C2 → knowledge base       | Markdown; GitHub REST API over HTTPS                     |
| I4  | C1 → model (Copilot)      | JSON; HTTPS inside GitHub                                |
| I5  | C1 → GitHub               | JSON; GitHub REST API over HTTPS                         |
| I6  | CLI → C3                  | JSON over HTTPS REST, SigV4 signed (G1)                  |
| I7  | C3 → Recipe Database      | JSON items; AWS SDK for Java v2 (DynamoDB)               |
| I8  | C3 → C4                   | Java function call, in process                           |
| I9  | C3 → audit store          | DynamoDB items; AWS SDK for Java v2                      |
| G1  | CLI → API Gateway → C3    | SigV4 over HTTPS; IAM authorization                      |
| G4  | Components → log service  | Structured log entry with allowlisted fields             |

#### I1: GitHub → C1

- **Inputs:** `pr_number` int, `repo` string, `head_sha` string, `event_type` "opened" or "synchronize".
- **Outputs:** none.
- **On error:** other event types are ignored and logged. A newer `head_sha` discards the running pass.

#### I2: C1 → C2

- **Inputs:** `changed_files` string[].
- **Outputs:** `articles` {`id`, `text`}[].
- **On error:** `KB_UNAVAILABLE`: C1 applies the manual review label and mentions the author. An empty list is valid.

#### I3: C2 → knowledge base

- **Inputs:** `repo` string.
- **Outputs:** articles as markdown.
- **On error:** fetch fails or is partial: `KB_UNAVAILABLE`, and no partial results are used.

#### I4: C1 → model (GitHub Copilot)

- **Inputs:** `diff` string, `articles` object[] (at most the configured cap).
- **Outputs:** `findings` {`file`, `line`, `text`, `article_id`}[].
- **On error:** timeout or failure: retry once, then the manual review label.

#### I5: C1 → GitHub

- **Inputs:** `comments` {`file`, `line`, `body`}[], `label` string, `status` "approved" or "changes_requested".
- **Outputs:** `review_id` int.
- **On error:** post fails: retry, then the manual review label (up to 3 label retries with backoff, then log).

#### I6: CLI → C3

- **Inputs and outputs:** the five API operations and JSON bodies in 3.2.
- **CLI exit codes:**
  - `0` success
  - `2` invalid arguments (nothing sent)
  - `3` rejected with 4xx (code and message printed)
  - `4` service or authentication failure

#### I7: C3 → Recipe Database

- **Inputs:** `cursor` string; for a deletion check, `recipeId` (strongly consistent read); for a delete, {`recipeId`, `steps`, `ingredients`} (the delete applies only if both fields still equal these values, 3.8). Batch size and timeout come from configuration.
- **Outputs:** `recipes` {`recipeId`, `title`, `steps`, `ingredients`, `domains`, `cooktime`}[], `next_cursor` string.
- **On error:** a read over 30 s or a failed read sets the run to HALTED (`BATCH_TIMEOUT` or `DB_ERROR`), leaves `last_cursor` unchanged, and stops further deletion. Access denied: HALTED (`ACCESS_DENIED`).

#### I8: C3 → C4

- **Inputs:** `recipeId`, `steps` string[], `ingredients` string[], `ruleset_version` string.
- **Outputs:** `score` int 0 to 100, `defects` {`rule_id`, `field`, `reason`}[].
- **On error:** a missing or wrong-typed field scores 0 with defect `UNPARSEABLE`. `UNKNOWN_RULESET` sets the run to HALTED (`CONFIG`).

#### I9: C3 → audit store

- **Operations:**
  - Start a run and acquire the active-run lock (one transaction, 3.5)
  - Resume a run (conditional status update, 3.5), then clean up uncommitted flag rows (3.7)
  - Write a checkpoint (conditional on `status = RUNNING` and `checkpoint_seq`; the last one also sets `AWAITING_CONFIRMATION`, 3.7)
  - Write flags (conditional on the row being absent or from the current batch, 3.7)
  - Load exceptions
  - Claim a recipe and write its removal log (one transaction, 3.8)
  - Finalize a claimed recipe as deleted, or release or reconcile the claim (3.8)
  - Complete a run and release the lock (one transaction, 3.5)
- **Outputs:** the stored items in 3.4.
- **On error:** `STORE_ERROR`: a running audit is HALTED and deletion is refused. If the claim and removal-log transaction fails, the recipe is not deleted. A held lock returns 409 `RUN_IN_PROGRESS`.

#### G1: CLI → API Gateway → C3

- **Inputs:** AWS credentials from the standard credential chain.
- **Outputs:** an authorized request carrying the verified caller identity (3.6), or 403 `FORBIDDEN`.
- **On error:** 403 stops the request at the gateway. Expired credentials: the CLI re-signs once, then exits 4.

#### G4: Components → log service

- **Fields (allowlist):**
  - On every entry: `timestamp`, `component`, `level`, `event`, `correlation_id` (`run_id` or `pr_number`), `error_code`.
  - On deletion events, also required: `confirmed_by` (the verified IAM principal ARN of the caller, 3.6) and `recipe_id`.
- **Deletion events:** `RECIPE_DELETED`, `RECIPE_DELETE_SKIPPED`, `RECIPE_DELETE_FAILED`, `RECIPE_DELETE_UNCERTAIN`, and `LOG_WRITE_FAILED`. Each has `correlation_id` set to the `run_id`. `RECIPE_DELETE_SKIPPED` carries `error_code` `CLAIMED_ELSEWHERE`, `NO_LONGER_ELIGIBLE`, `RECIPE_CHANGED`, or `RECIPE_ABSENT` (3.8). `confirmed_by` is always the holder of the claim that the event reports on.
- **Interface:** `log(event, level, correlation_id, error_code, confirmed_by, recipe_id)`. The last two are null except on deletion events.
- **Excluded:** recipe content and credentials. A principal ARN and a recipe ID are identifiers, not content (D2 EC-36).
- **On error:** a logging failure never fails the caller; entries go to local output. G4 is distinct from the removal log.

### 3.2 C3 endpoints

All endpoints use HTTPS and JSON, have the `/v1` prefix, and require SigV4. The major version is in the path; adding optional fields is not a breaking change (D1 part 5). Every 4xx and 5xx response uses the `error` body in 3.3.

#### `POST /v1/audit-runs`: start a run

- **Input:** `threshold` int 0 to 100, optional, default 70.
- **Success:** 202 with `run_id`, `status` "RUNNING", `threshold`.
- **Errors:** 400 `INVALID_THRESHOLD`; 409 `RUN_IN_PROGRESS` (another run holds the active-run lock, 3.5).

#### `GET /v1/audit-runs/{run_id}`: run status

- **Input:** `limit` int 1 to 200 (default 50), `page_token` optional.
- **Success:** 200 with `status`, `scored_count`, `flagged_count`, `halt_reason`, `resume_point`, `flagged`[], `next_page_token`. `flagged` lists only rows from committed batches, so it always matches `flagged_count` (3.7).
- **Errors:** 404 `RUN_NOT_FOUND`; 400 `INVALID_LIMIT`; 503 `STORE_UNAVAILABLE`.

#### `POST /v1/audit-runs/{run_id}/resume`: resume a halted run

- **Input:** `run_id`.
- **Success:** 202 with `status` "RUNNING".
- **Errors:** 404 `RUN_NOT_FOUND`; 409 `RUN_NOT_HALTED` (also returned to the second of two concurrent resumes, 3.5).

#### `POST /v1/audit-runs/{run_id}/exceptions`: keep recipes

- **Input:** `recipeIds` string[] (1 to 200), `reason` string (1 to 500 characters).
- **Success:** 200 with `excepted_ids`.
- **Errors:** 400 `INVALID_REQUEST`; 404 `RUN_NOT_FOUND`; 422 `RECIPE_NOT_FLAGGED`.

#### `POST /v1/audit-runs/{run_id}/deletions`: delete confirmed recipes

- **Input:** `recipeIds` string[] (1 to 200). There is no confirmer field; the confirmer is the verified caller (3.6).
- **Success:** 200 with `deleted_count`, `deleted_ids`, and, when any ID was not deleted, `not_deleted` (each ID with a reason: `CLAIMED_ELSEWHERE`, `NO_LONGER_ELIGIBLE`, `RECIPE_CHANGED`, `RECIPE_ABSENT`, `DELETE_FAILED`, or `OUTCOME_UNKNOWN`). IDs that were not deleted can be retried (3.8); an `OUTCOME_UNKNOWN` ID is settled by the retry or by recovery, never deleted twice.
- **Errors:** 400 `INVALID_REQUEST`; 403 `FORBIDDEN` (no verified caller identity, 3.6); 404 `RUN_NOT_FOUND`; 409 `RUN_NOT_AWAITING_CONFIRMATION`; 422 `RECIPE_NOT_DELETABLE`; 503 `LOG_WRITE_FAILED` (only if nothing was deleted).

**CLI mapping.** The CLI maps these to `audit start`, `audit status`, `audit resume`, `audit except`, and `audit delete` ([D2 part 2.2](Design_Diagrams/D2_Detailed_Design.md)). `audit delete` asks the Engineer to type `yes` before any request is sent, then prints the count deleted and each ID that was not deleted with its reason (`not_deleted`, 3.2).

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
        "decision": {
          "enum": ["PENDING", "DELETING", "EXCEPTED", "DELETED", "RESOLVED"]
        }
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
        "deleted_ids": { "type": "array", "items": { "type": "string" } },
        "not_deleted": {
          "type": "array",
          "items": {
            "type": "object",
            "required": ["recipeId", "reason"],
            "properties": {
              "recipeId": { "type": "string" },
              "reason": {
                "enum": [
                  "CLAIMED_ELSEWHERE",
                  "NO_LONGER_ELIGIBLE",
                  "RECIPE_CHANGED",
                  "RECIPE_ABSENT",
                  "DELETE_FAILED",
                  "OUTCOME_UNKNOWN"
                ]
              }
            }
          }
        }
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

Entity attributes and relationships are in the [D1 data model](Design_Diagrams/D1_Detailed_Design.md#2-data-model). `AUDIT_RUN.halt_reason` is added in [D2 part 2.1](Design_Diagrams/D2_Detailed_Design.md). `ACTIVE_RUN_LOCK` is added by this specification (3.5).

#### `AUDIT_RUN`

- **Keys:** partition `run_id`; no sort key.
- **Attributes:** `status`, `threshold`, `ruleset_version`, `started_by` (verified caller, 3.6), `started_at` (ISO 8601 UTC), `last_cursor`, `checkpoint_seq` (number of committed batches, 3.7), `scored_count`, `flagged_count`, `halt_reason`, `halted_at` (ISO 8601 UTC, set when the run becomes `HALTED`, 3.7).
- **Access:** created in the start transaction with the lock (3.5); checkpoint updates are conditional on `status = RUNNING` and the previous `checkpoint_seq` (3.7).

#### `ACTIVE_RUN_LOCK`

- **Keys:** partition `lock_id`, always the value `"AUDIT"`; no sort key. The table holds at most one item.
- **Attributes:** `run_id` (the run holding the lock), `acquired_by` (verified caller), `acquired_at` (ISO 8601 UTC).
- **Access:** put in the start transaction; deleted in the completion transaction (3.5).

#### `FLAGGED_RECIPE`

- **Keys:** partition `run_id`, sort `recipeId`.
- **Attributes:** `title`, `score`, `defects`, `decision` (`PENDING`, `DELETING`, `EXCEPTED`, `DELETED`, or `RESOLVED`), `batch_seq` (the batch that wrote the row, 3.7), `claim_id`, `claimed_by`, and `claimed_at` (set only while a deletion claim is held or after it succeeds, 3.8).
- **Access:** one query returns a run's flags. Written only if absent or from the current batch (3.7). `decision` changes only through conditional updates on its current value (3.8). `RESOLVED` means the recipe was repaired or was already gone, so nothing is left to delete.

#### `KEEP_EXCEPTION`

- **Keys:** partition `recipeId`; no sort key.
- **Attributes:** `reason`.
- **Access:** loaded whole at run start.

#### `REMOVAL_LOG`

- **Keys:** partition `run_id`, sort `recipeId`.
- **Attributes:** `score`, `defects`, `confirmed_by` (verified IAM principal ARN of the deletion request, 3.6), `claim_id` (the claim that wrote the row, 3.8), `removed_at` (ISO 8601 UTC, set when the claim is made, before the delete).
- **Access:** written only if absent, in the same transaction as the claim and before the delete. Removed only when that claim is released (3.8).

#### Recipe Database (external)

- **Keys:** partition `recipeId`; no sort key.
- **Attributes:** `title`, `steps`, `ingredients`, `domains`, `cooktime` (minutes). C3 reads and deletes; it stores no copy.
- **Access:** paged reads by `cursor`; strongly consistent read by ID for a deletion check; delete by ID, conditional on `steps` and `ingredients` (3.8).

**Ruleset.** The ruleset is a version-controlled file, not a table: `RULESET` {`version`, `default_threshold`} contains `RULE` {`rule_id`, `field`, `check`, `penalty`} ([D2 F3](Design_Diagrams/D2_Detailed_Design.md#23-c4-recipe-quality-scorer)).

### 3.5 Active-run guard

W-08 and A-05 allow one audit run at a time. A DynamoDB conditional write checks only the item at its own key, so a condition on the new `AUDIT_RUN` item cannot stop a second start with a different `run_id`. A single Spring Boot instance also serves requests on many threads, so the instance count alone does not serialize starts. Every start therefore claims one shared item.

**Admission (start).** `POST /v1/audit-runs` writes two items in one `TransactWriteItems` call:

1. Put the `ACTIVE_RUN_LOCK` item, on condition `attribute_not_exists(lock_id)`.
2. Put the `AUDIT_RUN` item with status `RUNNING`, on condition `attribute_not_exists(run_id)`.

Both items are written or neither is. Every start writes the same lock key, so DynamoDB admits at most one, whatever thread or instance sends it. If the lock condition fails, or the transaction is cancelled by a conflicting start, nothing is written and the caller gets 409 `RUN_IN_PROGRESS`, with the lock holder's `run_id` in the message. The scoring worker starts only after the transaction succeeds.

**Resume.** `POST .../resume` changes the status with a conditional update from `HALTED` to `RUNNING`. Of two concurrent resumes, one succeeds and the other gets 409 `RUN_NOT_HALTED`, so at most one worker runs per run. The lock is not touched; it already belongs to this run.

**Worker writes.** Every checkpoint write by the audit loop is conditional on `status = RUNNING`. A worker whose run was halted, for example by restart recovery, fails its next checkpoint write and stops without changing the run. Any flag rows it wrote for that batch are uncommitted; the next resume deletes them before re-scoring the batch (3.7). A worker that halts its own run sets `HALTED` and `halted_at` only after its outstanding calls have returned or timed out, and the resumed worker waits out the quiet period (below) before its cleanup.

**Quiet period.** The setting `settle_seconds` (G3, default 60) must exceed the SDK `apiCallTimeout` (default 10 s) and the ECS stop timeout (default 30 s). Once that long has passed since a worker stopped, no write it issued can still be applied. Replay cleanup (3.7) and claim recovery (3.8) wait for it.

**Release.** The lock is held while the run is `RUNNING`, `HALTED`, or `AWAITING_CONFIRMATION`, because a run in any of these states can still delete recipes. It is released only in the transaction that completes the run, once no flag row is `PENDING` or `DELETING` (3.8). The completion check is one strongly consistent query that reads every page. It runs after the final checkpoint, at the end of every deletion or exceptions request, and at startup recovery. Once the run is `AWAITING_CONFIRMATION`, no new flag rows appear, and rows only move out of `PENDING` or `DELETING`, except when a released claim returns a row to `PENDING`. That release is itself followed by a check, so a run is never left open with nothing to trigger its completion.

The completion transaction has two steps:

1. Update `AUDIT_RUN` to `COMPLETE`, on condition `status = AWAITING_CONFIRMATION`.
2. Delete the `ACTIVE_RUN_LOCK` item, on condition that its `run_id` is this run.

The lock has no expiry. A time-based lease could lapse while a run is legitimately halted or awaiting confirmation, and then admit a second run.

**Restart recovery.** The lock lives in DynamoDB, not in memory, so it survives a crash or restart. On startup, C3 waits out the quiet period without serving requests. It then sets any `RUNNING` run to `HALTED` (`RESTART`) with a conditional update and leaves the lock with that run. It reconciles the deletion claims the crash left behind (3.8), runs the completion check, and only then serves requests. No new run can start until the Engineer resumes the halted run and it completes. The lock and the run's status change only in the same transactions, so a crash cannot leave them in disagreement.

**Deployments.** The ECS service runs one task with `deployment_minimum_healthy_percent = 0` and `deployment_maximum_percent = 100`, so the old task stops before the new one starts. Without this, the new task's startup recovery could halt a run the old task is still scoring. The quiet period then covers any write the old task had in flight.

**Limit.** An abandoned run blocks new runs until it is completed, that is, until every flagged recipe is deleted, kept, or resolved. This release has no cancel operation (5.2).

### 3.6 Caller identity and deletion attribution

NFR-08 requires every deletion to name its confirmer and its run in both the removal log and the log entries. A `run_id` alone cannot tell apart two Engineers confirming deletions in the same run, and `started_by` names whoever started the run, not whoever confirmed.

**Where the identity comes from.** API Gateway verifies the SigV4 signature, then passes the caller's IAM principal ARN (`$context.identity.userArn`) to C3 in a request header set by the integration. The integration overwrites any value the client sent in that header. C3 is reachable only through API Gateway, over a VPC link to a private load balancer, so the header cannot be forged. With AWS IAM Identity Center, the principal ARN ends in the Engineer's user name.

**How C3 uses it.**

- `AUDIT_RUN.started_by` and `ACTIVE_RUN_LOCK.acquired_by` are the principal of the start request.
- For each deletion request, `confirmed_by` is the principal of that request. It is written to every `REMOVAL_LOG` row and every G4 deletion event that the request produces, together with the `run_id` and `recipe_id`. The claim protocol in 3.8 keeps this correct when two requests target the same recipe.
- The request body has no confirmer field, so a client cannot name someone else.
- If the identity header is missing, C3 returns 403 `FORBIDDEN` and deletes nothing.

### 3.7 Replay of an interrupted batch

**Problem.** The audit loop writes a batch's flag rows, then writes the checkpoint. A crash or halt between the two leaves flag rows from a batch that was never committed. The resumed run re-scores that batch from `last_cursor`. If a recipe in it was repaired in the meantime, the replay no longer flags it, but its old row stays. The recomputed `flagged_count` then leaves it out, so the count and the flagged list disagree. C3 stores no copy of recipes, so it cannot replay the batch with its original inputs. It removes the uncommitted rows instead and re-scores the batch against the data as it is at resume.

**Batch tagging.** `AUDIT_RUN.checkpoint_seq` counts committed batches and starts at 0. The worker tags every flag row it writes with `batch_seq = checkpoint_seq + 1`. Any row with `batch_seq` greater than `checkpoint_seq` belongs to the uncommitted batch.

**Flag writes.** Each flag row is put on condition `attribute_not_exists(recipeId) OR batch_seq = :current`, where `:current` is `checkpoint_seq + 1`. The worker keeps the set of recipe IDs it has flagged in this batch.

- **Condition passes:** the row is written, and the ID is added to the set. A retry after a lost response writes the same row again, because the first write is a row of the current batch, so a retry never fails on its own earlier write.
- **Condition fails:** the recipe was already flagged by a committed batch (a duplicate across pages, D1 part 3). It is skipped and not counted.

The batch's count is the size of the set, so a recipe that is written twice is counted once.

**Checkpoint.** One conditional update on `AUDIT_RUN`, on condition `status = RUNNING` and `checkpoint_seq` equal to the value the batch started with. It sets these fields:

- `last_cursor` to the batch's `next_cursor`
- `checkpoint_seq` to the previous value plus 1
- `scored_count` to the previous value plus the recipes the batch scored
- `flagged_count` to the previous value plus the size of the batch's set
- `status` to `AWAITING_CONFIRMATION`, only in the final checkpoint, when `next_cursor` is empty (an empty batch is the last batch, D1 part 3)

The final checkpoint leaves `RUNNING` in the same update that saves the exhausted cursor. A crash can then never leave a run `RUNNING` with a finished cursor, which a resume would scan again from the start. The worker then runs the completion check (3.5), which completes a run that has nothing flagged.

**Resume.** On every resume, whatever the halt reason, the worker does these steps before it reads any recipe:

1. Wait until `settle_seconds` have passed since `halted_at` (3.5). A resume after a restart has already waited.
2. Query the run's `FLAGGED_RECIPE` rows with a strongly consistent read (`ConsistentRead`), filtered to `batch_seq` greater than `checkpoint_seq`. Follow `LastEvaluatedKey` until it is empty. The filter is applied after each page is read, so a page can be empty while later pages are not. Collect every matching row before step 3.
3. Delete each collected row, on condition that `batch_seq` is still greater than `checkpoint_seq`.
4. Reload the keep exception list.
5. Read from `last_cursor` and score as normal.

Step 2 reads the base table, which supports strong consistency; a secondary index would not. A row written just before the crash is therefore always found, so no uncommitted row can become visible at a later checkpoint without being counted. These rows are always `PENDING`, because deletions are refused until the run is `AWAITING_CONFIRMATION` (D2 F2).

**Invariant.** After every checkpoint, `flagged_count` equals the number of the run's `FLAGGED_RECIPE` rows with `batch_seq` up to `checkpoint_seq`. The status endpoint lists only those rows, so the list and the count agree even while a batch is in progress.

**What NFR-11 guarantees.** A resume re-scores only the uncommitted batch (at most one), using the recipe data as it is at resume.

- If no recipe in that batch changed, the flagged set is the same as an uninterrupted run's.
- If a recipe changed (repaired, edited, or deleted), the flagged set reflects its state at resume, as an uninterrupted run reading the batch at that moment would.
- In both cases the flagged list matches `flagged_count`.

Recipes in committed batches are not re-read. If one of them changes after it was flagged, the re-score at deletion catches it (3.8).

### 3.8 Deletion claims

**Problem.** The active-run lock does not serialize requests within a run. Two deletion requests for the same recipe can both pass the D2 F2 checks. Request B can then overwrite A's removal-log row before A deletes successfully and B's delete fails. The removal log would name B, while A's `RECIPE_DELETED` event names A. Each recipe therefore has a claim that exactly one request can hold.

**Check.** For each ID, C3 first reads the recipe with a strongly consistent read and re-scores it, after re-checking the exception list (D2 F2).

- **Recipe absent:** set the row to `RESOLVED` (on condition `decision = PENDING`) and skip the ID with `RECIPE_ABSENT`.
- **No longer below the threshold:** set the row to `RESOLVED` the same way and skip the ID with `NO_LONGER_ELIGIBLE`.
- **Still below:** keep the `steps` and `ingredients` it read, and the score and defects, for the claim and the delete.

**Claim.** C3 makes one `TransactWriteItems` call on the audit tables:

1. Update the `FLAGGED_RECIPE` row: set `decision = DELETING`, `claim_id` to a value unique to this request (the API Gateway request ID), `claimed_by` to the confirmer (3.6), and `claimed_at` to now. Condition: `decision = PENDING`.
2. Put the `REMOVAL_LOG` row with the score and defects from the check, `confirmed_by`, and `claim_id`. Condition: `attribute_not_exists(recipeId)`.

The outcome decides what happens next:

- **Transaction succeeds:** this request holds the claim and goes on to delete.
- **Cancelled on a condition:** another request holds the claim, or the row is already `DELETED`, `EXCEPTED`, or `RESOLVED`. If the row is `DELETING` and no request on this instance holds its claim, C3 reconciles it first (below). Otherwise the ID is skipped with `CLAIMED_ELSEWHERE`.
- **Store failure:** the D2 F2 log-write-failure path applies. The loop stops, and the request returns 503 `LOG_WRITE_FAILED` if nothing was deleted.

**Delete.** The claim holder deletes the recipe from the Recipe Database (I7) with a conditional delete. The condition is that `steps` and `ingredients` equal the values read in the check, so a recipe repaired after it was scored is not deleted. A delete that returns the failed item (`ReturnValuesOnConditionCheckFailure = ALL_OLD`) lets C3 see what changed. The delete is safe to repeat, because the same condition applies each time. Transient errors are retried up to 3 times with backoff. The outcome falls into one of these cases:

- **Delete succeeds:** update the row to `decision = DELETED`, on condition `claim_id` equals this request's. Emit `RECIPE_DELETED`. If that update fails, the row stays `DELETING` with the correct attribution, and reconciliation finishes it (below).
- **Condition fails and the recipe is present (it changed):** nothing was deleted, so this is a definitive rejection. Re-score the returned item. Release the claim (below), with the row set to `PENDING` and the new score and defects if it is still below the threshold, or to `RESOLVED` if it is not. Skip the ID with `RECIPE_CHANGED` or `NO_LONGER_ELIGIBLE`.
- **Condition fails and the recipe is absent:** if an earlier attempt of this request had an unknown outcome, that attempt applied, so finalize as `DELETED` as above. Otherwise the recipe was gone before the delete: release the claim with the row set to `RESOLVED`, and skip the ID with `RECIPE_ABSENT`.
- **Other definitive rejection** (for example access denied or a validation error): release the claim with the row back to `PENDING`, and emit `RECIPE_DELETE_FAILED`.
- **Unknown outcome after the retries** (a timeout, a server error, or a lost connection): the delete may have been applied. Keep the claim, the `DELETING` row, and the removal-log row exactly as they are. Emit `RECIPE_DELETE_UNCERTAIN`, leave the ID out of `deleted_ids`, and report it in `not_deleted` as `OUTCOME_UNKNOWN`. Reconciliation settles it (below).

**Release.** The claim is released in one transaction, on condition `claim_id` equals this request's:

1. Delete the `REMOVAL_LOG` row.
2. Update the `FLAGGED_RECIPE` row to the decision named above, and remove `claim_id`, `claimed_by`, and `claimed_at`.

A release happens only when nothing was deleted, so the removal log never loses the row of a recipe that was removed. After a release, the request runs the completion check (3.5). A later request for a `PENDING` ID makes a new claim and a new log row.

**Reconciliation.** This settles a `DELETING` row whose request did not finish. It runs at startup (3.5) and whenever a deletion request meets a `DELETING` row whose `claim_id` is not held by a request in flight on this instance. Claims are never taken over otherwise, because a claim that looks stuck may belong to a request still in progress. C3 reads the recipe with a strongly consistent read:

- **Absent:** the claimed delete was applied. Set the row to `DELETED`, on condition `claim_id` is unchanged, and emit `RECIPE_DELETED` with the claimant as `confirmed_by`. The removal log already names them.
- **Present, claim younger than `settle_seconds`:** leave it. The claimed delete could still be applied. The request skips the ID with `CLAIMED_ELSEWHERE`.
- **Present, claim at least `settle_seconds` old:** the claimed delete was not applied and cannot be applied later (3.5 quiet period). Release the claim with the row back to `PENDING`.

One limit applies. Reconciliation treats an absent recipe as deleted by the claim holder. C3 is the only path that deletes these recipes, so the attribution holds, but a deletion by a person outside C3 during an uncertain window would be credited to the claimant.

**Attribution rule.** A removal-log row is written only together with a claim, cannot be overwritten, and is removed only when its own claim is released, which happens only when nothing was deleted. Only the claim holder deletes. So the removal log, `FLAGGED_RECIPE.claimed_by`, and the `RECIPE_DELETED` event always name the same Engineer: the one whose request deleted the recipe. This holds across concurrent attempts, retries, lost responses, and crashes.

**Exceptions.** The exceptions endpoint changes `decision` from `PENDING` to `EXCEPTED` with the same `decision = PENDING` condition, so an exception and a deletion cannot both succeed. An ID that is not `PENDING` returns 422 `RECIPE_NOT_FLAGGED`.

**Orphaned claims.** Only a crash or an unknown delete outcome can separate a claim from its request. Both are settled by reconciliation, which also runs for every `DELETING` row of the active run at startup, after the quiet period (3.5). At that point every leftover claim is older than `settle_seconds`, so each one is finalized or released.

**Completion.** A run moves to `COMPLETE` only when no flag row is `PENDING` or `DELETING`. `EXCEPTED`, `DELETED`, and `RESOLVED` rows are finished. The completion check in 3.5 decides.

---

## 4. Technology Stack

Full comparisons and rejected alternatives are in [D1 part 6](Design_Diagrams/D1_Detailed_Design.md#6-technology-choices-with-justification).

- **PR review runtime (C1, C2): GitHub pull request configuration and REST API.** Native events, comments, labels, and verdicts give REQ-02, REQ-03, and REQ-05, with nothing of ours to host or secure (CON-01).
- **Model (C1): GitHub Copilot, model chosen after testing.** Keeps the provider inside GitHub (CON-01). Billed under the organization's plan (CON-04). Time to be measured against NFR-01.
- **Knowledge base store (C2): GitHub repository.** Documentation is already migrated there (UC-01). Read on each run, so there is no index to go stale (REQ-01, REQ-05).
- **Backend (C3, C4): Java with Spring Boot (Apache-2.0).** Team skill and mutual code review (CON-02). The light workload easily meets NFR-03 and NFR-09.
- **Client (C3): picocli and `java.net.http`, with the AWS SDK v2 signer.** The command line is the only interface (W-05). Help output supports NFR-10. Typed confirmation implements REQ-10 and NFR-04.
- **API authentication: Amazon API Gateway with IAM (SigV4) authorization.** Authentication is not hand-rolled (NFR-04, CON-01). Requests are rejected with 403 before C3 is reached, and the verified caller is passed to C3 for attribution (NFR-08, 3.6).
- **Hosting (C3): AWS ECS Fargate, one task.** A long-running container has no 15-minute function limit (REQ-07, NFR-09). Restart recovery uses the checkpoint (NFR-11). The deployment settings in 3.5 keep a single instance (W-08).
- **Data store: Amazon DynamoDB, AWS SDK for Java v2.** Same store as the Recipe Database. Key lookups only (REQ-07, REQ-13). Transactions give the active-run guard (W-08, 3.5) and the deletion claims (NFR-08, 3.8). Per-request billing (CON-04).
- **Scoring rules (C4): plain Java code, with rule penalties in a Jackson YAML ruleset.** No model call per recipe (NFR-06, CON-04). Repeatable scores (NFR-07). Weights change without code changes (REQ-08).
- **Infrastructure as code: Terraform.** Non-production and production stacks reviewed as code (NFR-05, CON-02, CON-03).
- **Logging: structured entries to the AWS log service (D2 F5, node CW).** Traceable deletions with `confirmed_by` and logged overruns (NFR-08, NFR-01). Never blocks the caller.
- **Testing: JUnit 5, a sandbox repository, and a non-production AWS account.** Threshold boundary tests (REQ-08), isolated testing (NFR-05, CON-03), and verification for every REQ and NFR.

---

## 5. Known Constraints

### 5.1 Inherited constraints (CON)

The constraints are stated in the Requirements Document part 8.

#### CON-01: Security best practices for agents

- **Rules out:** agents with broad permissions; automatic deletion (W-01); hand-rolled authentication.
- **Forces:** IAM authorization (G1); a task role limited to the recipe and audit tables; typed confirmation at the client and the API; a confirmer taken from the verified caller (3.6); GitHub-managed minimum permissions for C1 and C2.

#### CON-02: Substantive code reviews

- **Rules out:** merging an unreviewed change.
- **Forces:** the other member reviews every task before it closes ([Plan.md](Planning/Plan.md) R3); pull requests against the repository.

#### CON-03: Testing in isolated environments

- **Rules out:** testing against the production recipe database or a production repository.
- **Forces:** a sandbox repository and a non-production AWS account (Terraform stack, task T2); sandbox data only.

#### CON-04: Minimize token and subscription cost

- **Rules out:** a model call per recipe (W-02); sending the whole knowledge base to the model.
- **Forces:** fixed-rule scoring; only relevant articles, up to a configured cap; per-request storage billing; the model chosen after cost and quality testing.

#### CON-05: Benefit developers and consumers

- **Rules out:** shipping features that are not validated with users.
- **Forces:** pilot validation with pilot engineers (tasks T11 and T12) and recorded results for every REQ and NFR.

### 5.2 Technical limits carried from D1 and D2

- **API Gateway allows 29 seconds per request.** Start and resume return 202 immediately; progress is read by polling status.
- **Copilot review pass time is unmeasured (D2 Q1, OQ-01).** NFR-01 is verified by task T1 and pilot results; the manual review label is the fallback.
- **One C3 instance and one audit run at a time (W-08).** On startup, any RUNNING run becomes HALTED (`RESTART`) and the Engineer resumes it. The active-run lock enforces one run (3.5).
- **An abandoned run holds the active-run lock (3.5).** New runs are refused until the Engineer completes it by deleting, keeping, or resolving every flagged recipe. A cancel operation is a candidate for a later release if the pilot needs it.
- **Requests are limited to 200 IDs and status pages to 200 items.** The client splits IDs into groups of 200 and follows page tokens.
- **All thresholds, caps, and sizes are examples (D2 Q8).** They live in configuration (G3), not code.

---

## 6. Traceability Matrix

One row per REQ, NFR, and CON ID.

- **Status:** Designed (in D0 to D2 or this specification), In progress, or Planned (not yet designed or built).
- **Test case:** the TC numbers assigned for the Week 10 test plan (A10, due 2026-11-04), one per row.
- **Coverage:** no design component is blank, and there are no ORPHANED rows. The two process constraints without a D0 to D2 element (CON-02, CON-05) are tied to the plan tasks that carry them and marked Planned.

| Req ID | Description | Design component | Status | Test case |
| ------ | ----------- | ---------------- | ------ | --------- |
| REQ-01 | Retrieve relevant knowledge base articles for the changed files | C2; D0 I2, I3; D1 `select_articles`; D2 2.5 | Designed | TC-001 |
| REQ-02 | Cited inline comments; uncited findings marked Low or Medium | C1; D0 I4, I5; D1 review pass; D2 F4 | Designed | TC-002 |
| REQ-03 | Re-review each push on the latest `head_sha`; approve when clean | C1; D0 I1 (`synchronize`), decision 5; D2 F4, EC-02 | Designed | TC-003 |
| REQ-04 | Dismiss informational comment with a one-line justification | C1; D0 I5; D1 UC-01 A2 (GitHub native resolve) | Designed | TC-004 |
| REQ-05 | Manual review label and author mention when knowledge base unreachable | C1; D0 I2; D2 F4 fallback, G2, EC-05, EC-09 | Designed | TC-005 |
| REQ-06 | Recorded outcome with citations on the pull request | C1; D0 I5 (`status`); D2 part 4 (cycle time is organization-measured) | Designed | TC-006 |
| REQ-07 | Start an audit run; score the recipe database in batches | C3, C4; D0 I6, I7, I8; D1 `POST /v1/audit-runs`, `AUDIT_RUN`; D2 F1, 2.2; spec 3.5 active-run lock, 3.7 checkpoint | Designed | TC-007 |
| REQ-08 | Flag recipes strictly below the threshold with field-level defects | C4; D0 I8; D1 `FLAGGED_RECIPE`, scoring algorithm; D2 2.3, EC-20 | Designed | TC-008 |
| REQ-09 | Show flagged recipes with defects, paged, before deletion | C3 and CLI; D0 I6; D1 `GET /v1/audit-runs/{run_id}`; D2 2.2; spec 3.7 committed rows only | Designed | TC-009 |
| REQ-10 | Delete only confirmed IDs; report the count | C3 and CLI; D1 `POST .../deletions`; D2 F2, 2.2 `audit delete`, EC-14; spec 3.8 deletion claims and conditional delete | Designed | TC-010 |
| REQ-11 | Skip keep exceptions in later runs | C3; D1 `KEEP_EXCEPTION`, `POST .../exceptions`; D2 EC-14 | Designed | TC-011 |
| REQ-12 | Halt on timeout or failure; keep checkpoint; show resume point | C3; D0 I7; D1 `last_cursor`; D2 F1, `halt_reason`, `resume_point`, EC-17 | Designed | TC-012 |
| REQ-13 | Removal log entry before each delete | C3; D1 `REMOVAL_LOG`; D2 F2, EC-18; spec 3.8 claim and log in one transaction | Designed | TC-013 |
| NFR-01 | Comments within 5 minutes of the triggering event (latest `head_sha` only) | C1; D0 Flow A timing; D2 F4, EC-02, EC-03, G4 (Q1 open) | Designed | TC-014 |
| NFR-02 | Author notified within 10 minutes of the triggering event when the knowledge base fails | C1; D2 F4 fallback, G2, EC-29 | Designed | TC-015 |
| NFR-03 | No batch read longer than 30 seconds | C3; D0 I7; D2 F1, G3 timeout setting, EC-17 | Designed | TC-016 |
| NFR-04 | Authorized requests only; no unconfirmed deletion | G1; API Gateway IAM; C3 task role; D2 F2, 2.2 confirmation prompt, EC-26; spec 3.6 | Designed | TC-017 |
| NFR-05 | All testing in the sandbox | Terraform stacks (D1 part 6); D2 part 4 gap (stack not yet specified) | Planned | TC-018 |
| NFR-06 | No model call per recipe; article cap | D0 decision 2, decision 3; D2 G3 article cap, EC-06 | Designed | TC-019 |
| NFR-07 | Repeatable scores; ruleset version stored | C4; D1 `ruleset_version`; D2 F3 `RULESET`, EC-21 | Designed | TC-020 |
| NFR-08 | Deletions attributable to a confirmer and run | `REMOVAL_LOG.confirmed_by`; G4 `correlation_id`, `confirmed_by`, `recipe_id` (spec 3.1, 3.6); per-recipe claim and reconciliation (spec 3.8) | Designed | TC-021 |
| NFR-09 | 100 times volume without gateway timeout or oversized pages | C3; D1 part 3 complexity; cursor, checkpoint, paged status (D2 part 4) | Designed | TC-022 |
| NFR-10 | First-time Engineer completes the audit task in 10 minutes | C3 CLI; D2 2.2 command group and `--help`; five eligible pilot engineers (Requirements A-03); usability session not yet designed | Planned | TC-023 |
| NFR-11 | Crash recovery with the same flagged set over the same data; list matches count | C3; D1 checkpoint; D2 F1, EC-13; spec 3.5 restart recovery and quiet period, 3.7 replay of an interrupted batch | Designed | TC-024 |
| CON-01 | Security best practices | G1; D0 part 6 minimal permissions; D2 F2; spec 3.6 | Designed | TC-025 |
| CON-02 | Substantive code reviews | Delivery process: [Plan.md](Planning/Plan.md) R3 and section 4 (no D0 to D2 element) | Planned | TC-026 |
| CON-03 | Isolated testing environments | Terraform non-production stack (D1 part 6; task T2); sandbox repository | Planned | TC-027 |
| CON-04 | Minimize token and subscription cost | D0 decisions 2 and 3; D1 part 6 Copilot and on-demand billing; D2 G3 article cap | Designed | TC-028 |
| CON-05 | Benefit developers and consumers | D0 part 1 goal; pilot validation, tasks T11 and T12 (no D0 to D2 element) | Planned | TC-029 |

---

## 7. Advisor Review Record

**Status:** the review date, discussion, and resulting changes are TBD. They are recorded here, with a change log entry for each resulting change, once the review is held.

- **Advisor:** Akshansh Gupta
- **Date of review:** TBD
- **Documents reviewed:** Requirements_Document.md v1.4, Technical_Specification.md v1.4
- **Attendees:** TBD

### What was discussed

1. **MoSCoW split and the Won't list.** Advisor feedback: TBD.
2. **Numeric targets in REQ and NFR statements.** Advisor feedback: TBD.
3. **Traceability matrix and ORPHANED check.** Advisor feedback: TBD.

### Changes produced

TBD.
