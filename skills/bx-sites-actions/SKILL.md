---
name: bx-sites-actions
metadata:
  version: "1.0"
description: Operate and troubleshoot GitHub Actions workflows for ortus-boxlang/bx-sites. Use this when users ask about test failures, release runs, docs publish status, or rerunning/canceling workflows.
---

# bx-sites GitHub Actions Operations

Use this skill for CI and release operations in:
- Repository: `ortus-boxlang/bx-sites`
- Actions dashboard: `https://github.com/ortus-boxlang/bx-sites/actions`

## Workflow inventory

Primary workflows in bx-sites:

| Workflow | File | Typical purpose |
|---|---|---|
| Daily Tests | `.github/workflows/cron.yml` | Scheduled daily run of test suites |
| Pull Requests | `.github/workflows/pr.yml` | PR/branch validation |
| Test Suites | `.github/workflows/tests.yml` | Core reusable test workflow |
| Build Snapshot | `.github/workflows/snapshot.yml` | Development snapshot build after tests |
| Release | `.github/workflows/release.yml` | Stable release and version/tag flow |
| Docbox | `.github/workflows/docbox.yml` | API docs generation and publishing |
| Publish Docs | `.github/workflows/pages.yml` | Multi-theme site build + Pages deploy |

## Required troubleshooting flow

When users mention CI/build/test/workflow failures, always do this:

1. List recent runs for the target workflow or repository.
2. Inspect failed jobs and fetch logs.
3. Identify the first actionable root cause.
4. Recommend minimal remediation steps.
5. If requested, trigger rerun or cancel stuck runs.

## Suggested GitHub tool operations

- List workflows: `actions_list(method=list_workflows)`
- List runs: `actions_list(method=list_workflow_runs, resource_id=<workflow id or file>)`
- List jobs: `actions_list(method=list_workflow_jobs, resource_id=<run id>)`
- Read logs: `get_job_logs(job_id=<job id>, return_content=true)`
- Rerun run: `actions_run_trigger(method=rerun_workflow_run, run_id=<run id>)`
- Rerun failed jobs: `actions_run_trigger(method=rerun_failed_jobs, run_id=<run id>)`
- Cancel run: `actions_run_trigger(method=cancel_workflow_run, run_id=<run id>)`

## Output expectations

For each investigation, report:
- Workflow/run/job identifiers
- Failing step name
- Direct error excerpt from logs
- Root cause summary
- Minimal fix or rerun recommendation
