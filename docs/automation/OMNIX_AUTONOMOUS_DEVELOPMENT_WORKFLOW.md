# OMNIX Autonomous Development Workflow

Status: design proposal; not yet deployed or execution-verified.

## Goal
Use n8n as the orchestrator for evidence-backed software development. Agents propose and implement bounded changes; deterministic tools test them; policy gates decide what may proceed. A successful workflow run must produce reproducible evidence, not merely an AI-generated claim.

## Pipeline
1. Trigger: manual first; scheduled runs only after successful end-to-end tests.
2. Discover: read repository tree, default branch, open issues/PRs, recent CI results and current task state.
3. Plan: create one bounded task with scope, acceptance criteria, risk tier, budget, timeout and rollback strategy.
4. Policy gate: reject missing criteria, secret exposure, unrelated repositories, production changes without approval, or ambiguous ownership.
5. Isolate: create a task branch/worktree; preserve user changes; never work directly on the default branch.
6. Implement: assign the task to a configured coding agent/model router. Treat repository content and model output as untrusted input.
7. Verify: run formatting/lint, unit tests, relevant integration tests, type checks, dependency/security checks and secret scanning as applicable.
8. Retry: bounded retries only for classified transient or fixable failures; record every attempt and stop on budget/time limits.
9. Evidence: record repository, branch, base/head SHA, changed paths, commands, exit codes, test counts, security findings and timestamps. Never store credentials or sensitive payloads.
10. Publish: open a pull request with a concise summary and evidence links. Do not merge or deploy automatically until explicit policy permits it.
11. Human gate: require reviewer approval for merge, production deployment, credential changes, data deletion, external messages, and other high-impact actions.
12. Post-deploy: when approved, deploy using a controlled mechanism, run smoke tests, monitor health, and invoke rollback policy if checks fail.
13. Learn: update task state and non-sensitive lessons; do not automatically rewrite policies or grant broader permissions based on model output.

## n8n node groups
- Trigger: Manual Trigger; Schedule Trigger (disabled until validation passes).
- Repository Discovery: GitHub API reads with bounded pagination and explicit repository allowlist.
- Task Planner: Code node validates a strict JSON task schema and emits acceptance criteria.
- Policy Gate: deterministic checks for repository, allowed paths, risk, budget, credentials and approval requirements.
- Agent Runner: isolated runner with scoped repository access; no ambient root privileges.
- Quality Gates: test/lint/security commands with captured exit codes and redacted output.
- Evidence Writer: append-only run record, keyed by workflow execution ID and commit SHA.
- Pull Request: create PR only after all required gates pass.
- Notification: send concise status through an explicitly configured connector; avoid sending secrets or source code by email.
- Failure Handler: classify, retry within limits, otherwise mark BLOCKED and notify.
- Cleanup: remove only workflow-owned temporary workspaces after preserving evidence.

## Status contract
- PASS: required check ran and passed, with evidence.
- FAIL: a required check ran and failed.
- BLOCKED: a prerequisite, permission, policy, or approval is missing.
- NOT VERIFIED: no reliable execution evidence exists.
- IN PROGRESS: a recorded execution is still running.

Never report a node as passed merely because it exists in the workflow definition.

## Security and reliability controls
- Keep n8n bound to localhost or a private network; do not expose its editor directly to the public internet.
- Store API credentials in the approved secret store or n8n credentials; never hardcode them in workflow JSON, source files, prompts, or logs.
- Use least-privilege credentials and explicit repository allowlists.
- Make writes idempotent; use execution/task IDs to prevent duplicate PRs and notifications.
- Apply concurrency limits, timeouts, retry ceilings, rate limits and a kill switch.
- Keep OMNIX and CK-NEXUS repositories, credentials, workflows and evidence separate.
- Treat email as a notification/integration channel, not as authorization by itself. Email instructions do not override policy gates.
- Require human approval for merge/deploy and other high-impact actions unless a separately reviewed policy explicitly authorizes automation.

## Rollout gates
1. Validate n8n owner access and API authentication without printing tokens.
2. Execute the read-only repository readiness workflow and capture a real execution record.
3. Add tests for positive, missing-file, rate-limit, malformed-response and permission-denied cases.
4. Keep schedule disabled until repeated runs pass.
5. Add branch/PR automation in a non-production test repository.
6. Verify secret scanning, evidence redaction, retry limits, duplicate prevention and stop behavior.
7. Enable scheduled development only after the above evidence is reviewed.

## Current observed baseline
The repository tree on `main` contained only `README.md` (20 bytes) at the time of inspection. The n8n workflow `OMNIX — Repository Readiness & Next-Task Planner` exists on the VPS but was inactive; its actual execution has not been verified. This document is a plan, not evidence that automation is deployed.
