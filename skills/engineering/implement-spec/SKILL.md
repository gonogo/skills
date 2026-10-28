---
name: implement-spec
description: "Implement the result of /to-spec and /to-tickets in code."
disable-model-invocation: true
---

You have been provided a spec. This spec should have tickets associated with it, describing how to implement the spec.

The issue tracker should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.

The goal is the entire spec implemented on a single **integration branch**, with every ticket resolved the way the issue tracker closes work.

The tickets are not a list of steps. They are a **task graph** with blocking relationships between them. This means there is always a **frontier** of tickets which are ready to be grabbed.

Communication to and from subagents should be sparse. Communicate primarily through **context pointers**: to the spec, tickets, research notes, and previous commits. Don't duplicate information already available via pointers.

**Implementer subagents** run in the background. Cap concurrent implementers at **4** unless the user sets another number. Treat test timeouts as **config-owned**: an implementer may re-run a failing test once; if it still fails, report it rather than editing timeout values in test files.

## Hard rules

- **Unanswered decisions:** when you need a user decision, write the questions to the decision file (see Ledger) *before* asking. If answers are missing, ask again. Act only on answers you have received; never invent "safe defaults" for unanswered decision questions.
- **Fix forward:** repair the integration branch with a new commit. Never reset, rebase, or force-push it.
- **Origin tip:** implementers sync from `origin/<integration-branch>`, not a local checkout of it. The merger pushes the integration branch only after checks pass, so `origin/<integration-branch>` is the shared tip.

## Ledger

Keep a **live ledger** file for the run (default: `.scratch/<feature-slug>/ledger.md`). Create it when the integration branch is created. Update it at **every successful merge** (not only at wind-down) with, for each ticket: ticket id, implementer branch, merge commit SHA, and status (`in-progress` / `merged` / `blocked` / `failed`). Keep open issues and decision questions beside it in the same directory (`decisions.md` for unanswered questions).

## Steps

1. Read the spec and tickets to understand the task graph.

2. (optional) Use an **exploration subagent** to conduct any exploration required by the tickets - relevant codebase files or external documentation. Ensure the exploration subagent can save files - it should save its markdown notes in a directory outside the repo, accessible by all future subagents. This lets **implementer subagents** focus on implementation rather than exploration.

3. Create the integration branch and the live ledger. If the issue tracker closes work through PRs, or the user asks for one, open a draft PR after the first merge in step 5 (a branch with no commits ahead of main can't open one), marked as closing the spec and tickets.

4. Use **implementer subagents** to implement each ticket, each in its own worktree on its own branch, keeping at most **4** implementers running at once. Each implementer subagent:
   - confirms its worktree is based on `origin/<integration-branch>` before starting, and resets onto that remote tip if not;
   - calls the Skill tool with `tdd` to build the ticket;
   - merges `origin/<integration-branch>` into its own branch before reporting done (never merges a local-only integration checkout)

5. Once an **implementer subagent** completes, merge its work to the integration branch with a **merger subagent**. The merger:
   - runs a **per-ticket standards review** on that ticket's diff against `CODING_STANDARDS.md` and `docs/architecture.md` (or the repo's equivalents) before merging; fix standards findings on the ticket branch, then merge;
   - runs the repo's check command and pushes the integration branch only when checks pass;
   - updates the live ledger with ticket / branch / SHA / status;
   - on conflict or a bad merge tip: **fix forward** with a new commit — never reset, rebase, or force-push the integration branch

6. If this changes the **frontier** of available tickets, kick off more **implementer subagents** (still capped at 4 concurrent) to work on the new tickets.

7. Once all tickets are merged: update the ledger to final status, ensure open issues and any unanswered decision questions are written beside it, point the user at that hand-off, and **stop**. Do **not** run `code-review` or launch fixers in this session.

8. Clean up all **implementer subagent** worktrees.

## Fresh session: review and fixes

A **new session** picks up from the ledger and open issues. That session calls the Skill tool with `code-review` on the integration branch, fixes findings (still **fix forward** only), and — if a draft PR exists — marks it ready for review. Otherwise it resolves each ticket the way the issue tracker closes work and reports the integration branch.
