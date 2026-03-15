---
name: codex-orchestrator
description: Coordinate multiple child repos through Codex App Server-connected child agents with a shared plan and execution log. Use when Codex should act as an orchestrator across any two or more separate repos; connect to child App Server processes; deeply understand each child repo through its code, commit history, branches, and documentation; delegate work to child agents; coordinate parallel implementation; and keep `plan.md` and `runs.md` updated as the source of truth.
---

# Codex Orchestrator

## Overview

Run one OpenAI Codex session as the orchestrator and treat all child repos as delegated execution contexts. Keep the user talking only to the orchestrator, prefer App Server discovery or explicit endpoint configuration for child capabilities, and maintain enough shared documentation that the orchestration can survive session restarts. A common example is coordinating work or debugging issues that span frontend, backend, edge, or microcontroller firmware repos, but do not assume fixed child roles unless the user explicitly defines them.
This skill is designed to work with the App Server functionality in Codex.
The required order is: connect to child App Servers, deeply understand each repo, understand how the repos work together, write the plan, and only then execute.
Prefer a single kickoff prompt that includes the problem statement and the pasted child repo working directories. After that, proceed mostly autonomously and ask the user for more input only when a step genuinely depends on human action or judgment.
Child App Server terminals are headless server processes. Do not expect them to provide meaningful run visibility; treat the orchestrator session and `runs.md` as the main execution view.

## Workflow

1. Start with the user setup handshake.
   If the user has not already provided them, ask the user to start `codex app-server` in each child repo, paste the absolute working directory for each child repo, and state the problem to solve.
   Also ask for any optional logging, instrumentation, or pre-execution test requests only if the user has not already provided them.
   Then proceed to detailed inspection of each repo and the cross-repo system without waiting for further guidance.
2. Create or update the orchestration files directly.
   If `plan.md`, `runs.md`, or `orchestrator.config.yaml` are missing, write them in the current orchestrator repo using concise Markdown and YAML. Use `plan.template.md` and `runs.template.md` as the starting point when they exist.
   Record the setup facts you already know in `plan.md` and `runs.md` immediately.
3. Discover the children.
   Prefer App Server discovery or configured endpoints from each child. If discovery is incomplete, read `orchestrator.config.yaml`.
4. Build deep context for each child repo before planning execution.
   Use the child App Server to inspect the codebase, current branch, git status, recent commits, relevant history, and repo documentation. Focus on repo purpose, active task, tests/builds, commit and release policy, human checkpoints, and the current implementation state implied by both code and git history.
   When the child times out on broad requests, split discovery into smaller passes: code structure, git state, recent commits, docs, and then subsystem-specific walkthroughs. Explicitly set or restate the child repo path/cwd when there is any ambiguity.
   Build cross-repo understanding too: shared interfaces, dependencies, sequencing, and parallelism boundaries.
   Update `plan.md` and `runs.md` as new facts are discovered, not just at the end of discovery.
5. Write or refresh `plan.md` before delegation starts.
   Capture the user goal, success criteria, child context, cross-repo contracts, dependencies, and parallelizable work. Base the plan on the actual repo state, not just user description or child summaries.
   Explicitly reflect any requested logging, instrumentation, or pre-execution test runs in `plan.md`.
6. Delegate work to child agents.
   Do not start this step until the understanding and planning steps are complete. Keep tasks scoped, explicit, and verifiable. Use parallel delegation only when contracts are already clear or tasks are isolated.
7. Update `runs.md` after each meaningful child result.
   Record summaries, tests, artifacts, blockers, and any App Server connection notes worth preserving. Do not paste full transcripts.
   Also update `plan.md` whenever execution changes the plan, reveals a new dependency, or creates a new user checkpoint.
8. Route all human interaction through the orchestrator.
   If a child needs device testing, browser QA, credentials, release approval, product judgment, or any other action that only the user can perform, stop at the orchestrator and wait there.
   When user input or testing is required, state exactly what the user should do, what result is expected, and what decision the orchestrator is waiting on before more execution can happen.

## Operating Rules

- Treat the orchestrator as the only user interface.
- Prefer discovery over hardcoded child metadata, but keep a fallback config because child repos can live anywhere.
- Connect the orchestrator to already-running child App Server processes whenever possible; the orchestrator does not need to be responsible for launching them.
- This pattern depends on App Server because it gives the orchestrator more usable structure than sharing a Codex session as an MCP server.
- Start by collecting child repo working directories from the user before deep inspection begins.
- Build child understanding from primary repo evidence first: code, git state, branch structure, commit history, and repo docs.
- Build cross-repo understanding before execution: how the repos interact matters as much as what each repo contains.
- Prefer multiple small child requests over one broad synthesis request when deep discovery is timing out.
- Ask the user only when the next step genuinely depends on human action or human judgment.
- Prefer to infer the next step and extend the plan autonomously instead of asking the user for routine coordination decisions.
- Keep user requests at the orchestrator layer; child repos should never become the direct user interface.
- Treat the orchestrator session and `runs.md` as the main visibility surface for execution status.
- Respect repo-specific workflow differences. Do not force the same commit cadence, build process, or release sequence onto every child repo.
- Reconcile shared interfaces before asking multiple children to implement against them in parallel.

## Documentation Discipline

- Use `plan.md` for the current operating truth.
- Use `runs.md` for the execution ledger and connection notes.
- Create these files directly from the skill when they do not exist; do not require a helper bootstrap script.
- Treat `plan.template.md` and `runs.template.md` as tracked examples, and treat `plan.md` and `runs.md` as local generated working files.
- Keep both files concise and current enough that another orchestrator session can recover quickly.
- Update `plan.md` and `runs.md` continuously as the orchestrator learns more; do not wait for a single final synthesis.
- Summarize child code and git findings in the shared docs when they materially affect planning or execution.
- When child documentation changes the plan, update `plan.md` before delegating more work.

## Discovery Rules

- Assume the child Codex App Server processes are already running unless the user explicitly wants launch automation.
- First ask each child App Server for its available capabilities or any self-description it exposes.
- If discovery is incomplete, read `orchestrator.config.yaml` for child names, repo paths, docs, and workflow hints.
- Record only the child facts that matter for coordination:
  - repo purpose
  - active objective
  - tests and build commands
  - commit or release rules
  - human checkpoints
  - App Server connection details when relevant

## Deep Repo Understanding

- Use each child App Server to inspect the repository directly before planning or delegation.
- Prefer primary repo evidence over summaries:
  - read the relevant code paths
  - inspect current branch and git status
  - inspect recent commits and branch history relevant to the task
  - inspect repo documentation such as `AGENTS.md`, architecture notes, build docs, and release docs
- Treat code, git history, and docs as one combined context. Do not rely on only one of them.
- Record the parts of that context that materially affect orchestration in `plan.md` and `runs.md`.
- If the child App Server times out on broad prompts, chunk discovery into smaller requests in this order:
  - repo path and cwd confirmation
  - current branch and git status
  - recent commits and relevant branch history
  - key docs
  - one subsystem walkthrough at a time
- Avoid asking for a full architecture summary until the smaller slices have already been gathered.

## Human In The Loop

- Keep user interaction at the orchestrator layer.
- Ask the user for input only when the next step depends on human action or human judgment.
- Typical user-input checkpoints:
  - device or hardware testing
  - browser, simulator, or UX verification
  - credentials, secrets, or external access the agent does not have
  - release approval or go/no-go decisions
  - product decisions when repo evidence does not settle the question
- When asking for user testing or input:
  - stop delegation for that branch of work
  - add a human checkpoint in `plan.md` and `runs.md`
  - state exactly what the user should do
  - state exactly what result the orchestrator expects back
  - state what work is blocked until that input arrives
- Do not push the user into child sessions. The orchestrator remains the only user interface.

## Child Repo Checklist

The orchestrator should be able to answer these questions for each child repo before major execution:

- What is this repo responsible for?
- What branch or change context is active?
- What does the current git status imply about in-flight work?
- What recent commits or branch history matter to the current task?
- What local constraints matter right now?
- What is the current task?
- What changed?
- What remains blocked?
- What tests or builds were run?
- How does the orchestrator connect to this child App Server?
- What short state summary is needed to continue?
- When is it acceptable to commit?
- What release or distribution steps matter?
- What requires human validation?

## Commit And Release Policy

- Do not assume the same commit cadence across repos.
- Inspect each child repo's documented workflow before pushing for commits, releases, or submissions.
- Record any repo-specific rule that changes orchestration behavior, such as:
  - commit only after device verification
  - build before commit
  - release artifact required before QA

## Plan And Run Log Guidance

### `plan.md`

Use `plan.md` as the living source of truth for the current objective.

Include:

- the user-facing goal
- success criteria
- child context and constraints
- the current work plan
- parallelization notes
- human checkpoints
- unresolved risks

Update it when:

- the objective changes
- new child context changes the implementation plan
- dependencies between children are clarified
- human validation becomes required

### `runs.md`

Use `runs.md` as the compact execution ledger.

Each entry should capture:

- timestamp
- actor or child
- intent
- result summary
- tests or checks
- artifacts
- connection note if relevant
- next step

Avoid full transcripts. Store only what a future orchestrator session needs in order to reconnect to the child App Servers and continue coordination quickly.

## Resources

- Use the repo-level templates as starting points when they exist, but prefer writing or updating the live files directly from the skill.

## Exit Conditions

Stop only when the orchestrator can state:

- what each child changed or attempted
- which checks passed or failed
- what still needs human input
- how to reconnect to each relevant child App Server if needed
