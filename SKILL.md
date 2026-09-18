---
name: web-ai-workspace
description: Coordinate browser-based AI cross-review and workflow handoffs across remote Git, cloud-drive, and local courier work.
metadata:
  short-description: Coordinate AI workspace collaboration
---

# Web AI Workspace

Use this skill for browser-based multi-AI cross-review and workflow collaboration involving remote Git, local Codex execution, cloud-drive artifacts, reviews, edits, handoffs, or final synthesis.

## Browser selection and fallback

Prefer the built-in browser for browser-based work unless the user explicitly requests an external browser.

When the user explicitly requests an external browser, use that browser first. If the external-browser invocation fails twice consecutively, stop retrying it and continue with the built-in browser. Record the failed attempts and the fallback in the run facts; do not treat either failure as task success. If the user explicitly forbids fallback or the built-in browser is unavailable, mark the task `BLOCKED` instead.

## Default semantic-work routing

By default, content modification, review, and architecture or design work must be performed by an authorized worker/model in an external browser. If no authorized external worker is available, route the task as `BLOCKED` rather than performing the semantic work locally.

Local Codex is a routing and courier layer only: it may route requests, observe provider/workspace state, and transfer messages and artifacts exactly and losslessly. It must preserve payload, order, provenance, versions, and partial or failed states, and must not summarize, rewrite, translate, interpret, review, architect, or otherwise alter semantic content. Explicitly authorized mechanical workspace and version-control operations remain allowed for exact worker-provided artifacts.

## Core separation

Workers perform semantic work.

The courier/controller performs routing, transport, observation, exact transfer, and mechanical workspace operations.

Workspace adapters expose deterministic storage and version operations.

Publication proves only that an artifact was durably placed at a recorded version. It does not prove correctness or semantic acceptance.

The controller is a traffic coordinator and courier, not a hidden reviewer, editor, synthesizer, or final author.

## Non-negotiable authority boundary

While acting as courier/controller, the browser controller, local Codex courier, local executor, and workspace adapters must not:

- review or critique content;
- summarize, rewrite, translate, polish, or restructure worker content;
- perform code review;
- decide whether code, prose, analysis, or conclusions are correct;
- repair worker reasoning or choose between reviewer conclusions;
- interpret or resolve semantic merge conflicts;
- turn prose change instructions into code or document edits;
- decide that a clean textual merge is semantically safe;
- generate a final recommendation, synthesis, or conclusion;
- select excerpts and present them as a courier-authored summary;
- convert partial, interrupted, failed, blocked, ambiguous, or late output into success.

The courier may mechanically copy, upload, download, fetch, push, apply an exact patch, store a complete replacement file, move or mirror immutable artifacts, compare identifiers/hashes/revisions/SHAs, inspect paths, detect textual conflict markers, validate declared schemas, record observed states, and report transport/workspace facts.

When an action requires semantic judgment, route it to an authorized worker or human. If no authorized semantic worker exists, mark the task or run `BLOCKED`.

## Roles

### Worker

An AI or human role authorized to perform a declared semantic task. Workers may read declared inputs, reason about them, review them, edit within scope, create worker-owned artifacts, generate conclusions, and resolve assigned semantic questions.

### Courier/controller

May identify browser tabs/conversations, transmit authorized messages, observe provider state, transfer exact worker outputs, publish exact artifacts, operate Git/cloud adapters mechanically, record provenance, route handoffs, and record failures/retries. Seeing source material does not grant reviewer/editor authority.

### Finalizer

A worker, not a controller feature. It may synthesize, reconcile findings, make final edits/recommendations, and perform semantic integration. The courier may deliver its inputs and publish its output but may not synthesize it.

### Workspace adapter

A deterministic Git or cloud connector. It may locate, read, write, create, copy, version, publish, verify, and report identifiers/revisions. It must not decide what content means.

## Execution mode, semantic role, and allowed actions

Keep these concepts separate:

- execution mode: where/how the worker operates;
- semantic role: reviewer, editor, integrator, finalizer, etc.;
- allowed actions: operations authorized for the task.

Supported examples include `remote_web_git + reviewer`, `remote_web_git + editor`, `local_courier`, `cloud_document + reviewer`, and `cloud_document + editor`.

Browser capability is a contractual boundary, not a guarantee that the platform technically prevents every unauthorized click. If a reviewer makes an undeclared branch, commit, PR, or other repository write, record `UNAUTHORIZED_WRITE`, do not silently legitimize it, block automatic promotion, and route it for worker/human resolution.

## Turn state versus task state

Provider turn completion and semantic task completion are different facts.

Provider states may include `WAITING`, `BUSY`, `TURN_COMPLETE`, `INTERRUPTED`, `PROVIDER_ERROR`, and `MISSING`.

Task states may include `PENDING`, `RUNNING`, `SUCCEEDED`, `BLOCKED`, `FAILED`, and `CANCELLED`.

Preserve these non-equivalences:

`TURN_COMPLETE != TASK_SUCCEEDED`

`TASK_SUCCEEDED != PUBLISHED`

`PUBLISHED != DEPENDENCY_READY`

`NO_TEXT_CONFLICT != SEMANTIC_SAFE`

Timeout expiry, restored input UI, tab title, one DOM observation, or stopped streaming is never alone proof of semantic task success.

## Worker completion manifest

Every semantic worker task must return a machine-readable manifest in addition to free-form output. Minimum fields:

- `task_id`;
- `attempt_id`;
- `result_status`;
- `input_versions_used`;
- `output_artifact_refs`;
- `next_action_or_error`.

`result_status` is worker-declared and may be `SUCCEEDED`, `BLOCKED`, `FAILED`, or `CANCELLED`. The manifest is an interface contract, not proof of semantic correctness.

The courier may parse and structurally validate it, compare declared IDs/versions with delivery facts, and route according to it. The courier must not infer missing fields from prose, rewrite a malformed manifest, or declare success because prose looks complete.

Absent, malformed, inconsistent, or unparsable manifest => `task = BLOCKED` unless an explicit policy routes repair to a worker.

## Artifact identity, materialization, and publication

Separate logical artifact from immutable produced version.

Minimum identity:

- `artifact_id`;
- `artifact_version_id`;
- `attempt_id`;
- `content_digest`;
- `derived_from[]` when applicable.

Use immutable references such as repository + commit SHA + path, cloud file ID + revision ID, immutable object ID, or exact artifact version + digest. A filename, branch name, tab title, folder label, or current branch tip is not sufficient identity.

Materialization may be `MISSING`, `PARTIAL`, or `AVAILABLE`.

Publication may be `UNPUBLISHED`, `PUBLISHED`, or `PUBLICATION_FAILED`.

`SUPERSEDED` and `INVALIDATED` are not synonyms. `CONFLICT` normally describes a concurrency/integration event, not immutable artifact identity.

Partial artifacts may be preserved for audit/recovery but cannot satisfy ordinary dependencies, be concatenated with retry output, or be promoted as completed results. A retry creates a new attempt and immutable output version.

Publication records identify artifact version, destination workspace/locator, resulting versioned identity, evidence, and outcome. `PUBLISHED` means only that the recorded version exists durably; it is not semantic endorsement.

Preserve content provenance where transport differs from authorship:

- `content_author`;
- `materializer`;
- `publisher`.

## Dependency and exact-version rules

Normal downstream readiness requires all three:

- upstream task status == `SUCCEEDED`;
- required publication == `PUBLISHED`;
- exact declared artifact version.

Failure reports, blocked reports, diagnostics, or critiques may be downstream inputs only when explicitly declared. Never send every published artifact to a finalizer.

Handoffs use exact-version snapshot semantics. `SUPERSEDED` means a newer version exists; the older version remains historically valid. `INVALIDATED` means an authorized action explicitly forbids use for a specified task/dependency. Publishing v4 does not automatically cancel a reviewer assigned v3.

## Handoff and retry contract

Every handoff records:

- `handoff_id`;
- `task_id`;
- `attempt_id`;
- source and destination worker/thread;
- required semantic role and requested operation;
- exact input artifact references/versions;
- output artifact target;
- mechanical acceptance conditions;
- semantic acceptance owner.

The input list is the frozen snapshot. State: use these exact versions, do not silently substitute, report missing/unreadable inputs, return the manifest, and do not claim success before output is complete.

Every message that can trigger work carries `handoff_id`. If delivery has an unknown outcome, observe the destination and check whether the same handoff is already present or acknowledged. Resend only when safe. If reliable duplicate detection is impossible, use `UNKNOWN_OUTCOME`; do not blindly resend high-impact work.

Every retry uses a distinct `attempt_id`. Only the active attempt may satisfy dependencies, promote authoritative output, update authoritative pointers, or trigger ordinary continuation. A late/old attempt is preserved for audit but cannot replace the active result automatically.

## Remote-web Git reviewer/editor modes

### Reviewer

A semantic worker whose normal authority is review, not repository modification. Allowed actions must state whether it may read content, inspect diffs, produce findings, or write review comments. The web UI exposing commit controls does not grant editor authority.

### Editor

A semantic worker authorized to create a declared change. It may edit, branch, commit, push, or create/update a PR only as authorized. It reports repository, branch, commit SHA, paths, PR URL where applicable, and its completion manifest. Its change has a worker-owned authoritative producer version.

## Local Codex courier mode

A local Codex process acting as courier is not an editor. It may mechanically handle only exact artifacts:

- immutable Git commit;
- exact patch;
- complete replacement file;
- exact binary artifact.

It may fetch, checkout, create a declared branch/worktree, apply an exact patch, write an exact complete file, commit an explicitly worker-provided exact materialized change, push, and verify SHAs.

It must not turn prose such as “change function X so it behaves like Y” into code. That is semantic implementation and belongs to an editor worker. There is no small-change exception.

If remote editor commit `A` is integrated into derived commit `B`, preserve `B derived_from A`. Do not reconstruct A as an unrelated local commit or erase provenance.

## Git ownership and concurrency

Default to branch-per-worker, worktree-per-worker when useful, explicit write scope, and recorded base SHA. Independent workers should not write the same branch by default.

For a managed remote ref, record the expected tip and update only if the current tip still matches. A mismatch is `CONCURRENT_UPDATE`. Never auto-recover with undeclared pull/merge/push.

Mechanical applicability (patch applies, cherry-pick applies, no textual conflict) is not semantic integration. Semantic compatibility, conflict meaning, line choice, and approval belong to an authorized worker or human.

## Cloud-drive collaboration

Default to `artifact-per-worker` in a declared run folder. Use separate durable artifacts for primary output, each review, research, editor output, and final synthesis. Use file IDs and revision IDs rather than names alone.

Do not concurrently edit one authoritative file by default. Verify-write-verify is not atomic concurrency safety. Same-file automatic replacement requires conditional write such as `write only if current_revision == expected_revision`. Without that, use immutable artifacts, serialized/human-controlled replacement, or mark the operation unsupported.

## Acceptance boundaries

Mechanical acceptance may check manifests, IDs, file/revision/SHA/path/digest matches, publication success, and required references.

Semantic acceptance includes review quality, code correctness, valid conclusions/findings, intended meaning of edits, and resolution of competing interpretations. It belongs to an authorized worker, integration worker, finalizer, or human.

The courier may execute mechanical merge/cherry-pick/apply only with pre-existing authorization identifying source, target, and operation. It must never invent `semantic_safe = true` from a clean merge.

## Side effects and durable recovery

Message send, upload, cloud write, Git push, remote ref update, and PR creation may have unknown results. Preserve operation identity, re-observe the destination, determine mechanically whether the side effect exists, and retry only when idempotent or otherwise safe. `UNKNOWN_OUTCOME` is valid; “not immediately visible” is not proof of failure.

Before side effects, record durable intent:

- `operation_id`;
- `task_id / attempt_id`;
- operation type;
- target;
- expected precondition;
- payload or artifact identity.

Intent must survive controller restart. Lost acknowledgements must not create duplicate side effects.

Interrupted/error output becomes `PARTIAL` and requires a new attempt for retry. `BLOCKED` or `FAILED` reports may be published, but publication does not make the task successful.

## Authorized automatic execution

When the user has authorized the workflow, worker set, destination scope, and ordinary handoff operations, send declared worker messages and perform ordinary mechanical handoffs without repeatedly asking for confirmation.

This does not authorize unrelated edits, undeclared destructive operations, credential/permission changes, semantic work by the courier, undeclared reviewer-to-editor escalation, or unsafe shared-file overwrite.

## Standard orchestration sequence

1. Create `run_id`.
2. Register workers, roles, modes, allowed actions, workspace scopes, write scopes, and dependencies.
3. Create `task_id`, `attempt_id`, and `handoff_id`.
4. Deliver exact declared input versions.
5. Observe provider state without treating turn completion as task success.
6. Receive output and manifest; invalid/missing manifest => `BLOCKED`.
7. Verify mechanical output conditions.
8. Materialize and publish exact worker artifacts without semantic alteration.
9. Record immutable identity and publication evidence.
10. Start reviewers only when exact dependencies are ready.
11. Publish each worker result independently.
12. Freeze finalizer dependency snapshot.
13. Send only declared ready dependencies to finalizer.
14. Let finalizer perform semantic synthesis and return its manifest.
15. Publish finalizer output mechanically.
16. Report orchestration facts and exact worker-authored result references.

## Final report and required tests

Report run ID, workers/roles, courier mode, task/attempt statuses, late/orphaned attempts, exact artifact references, Git SHAs, cloud IDs/revisions, conflicts, retries, unknown outcomes, missing dependencies, and the worker-authored final reference.

Do not independently summarize the finalizer. Use only its exact artifact or structured summary authored by the finalizer.

At minimum verify that:

- courier review/summary/rewrite/reviewer-choice requests route to a worker;
- a complete “input inaccessible” response is turn-complete but not task-success;
- missing manifest becomes blocked;
- uploaded failure report is published but not dependency-ready;
- prose code instructions are not implemented by local courier;
- exact patch is mechanically applied only when authorized;
- unexpected reviewer commit is unauthorized and not promoted;
- remote A and derived B preserve provenance;
- v3 reviewer remains on v3 after v4 unless invalidated/cancelled;
- late retry cannot replace active attempt;
- partial output cannot satisfy normal dependency;
- duplicate handoff is checked before resend;
- clean merge receives no semantic approval;
- Git tip mismatch produces concurrent-update without auto merge;
- unsafe cloud overwrite is rejected;
- unknown push/upload outcomes are re-observed before retry;
- final report uses worker references or finalizer-authored structured summary.

## Prohibited shortcuts

Never use courier judgment as an invisible reviewer; treat prose as a manifest; convert natural-language edits into code; rebuild remote commits and erase provenance; equate upload success with task success; use filenames/branch names as immutable identity; call pre-check/write/post-check concurrency-safe; silently substitute versions; cancel exact-version review merely because a newer version exists; let late attempts promote themselves; concatenate partial and retry output; infer semantic safety from clean merges; auto-recover Git concurrency through undeclared merge; blindly retry unknown sends; or present courier-selected excerpts as semantic summary.

This Skill defines an orchestration contract. It does not require a particular database, full distributed transaction system, full WAL, browser-level enforcement of every action, or proof that worker self-reports are truthful. Stronger mechanisms are allowed, but the authority, identity, version, fencing, publication, concurrency, and failure boundaries above are mandatory.
