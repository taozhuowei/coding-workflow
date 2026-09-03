---
name: coding-workflow
description: "Run a complete, documented engineering workflow for complex feature work, refactors, adjustments, or greenfield tasks: Claude discovery and grilling, evidence-first decisions, Codex specification and tickets, then ticket-by-ticket implementation."
---

# Coding Workflow

Use this skill when the user wants a non-trivial change designed and delivered, not merely a quick code edit. The input is the user's task description. The output is working code plus a concise handoff that explains the goal, progress, remaining issues, and next recommendation.

This is a stateful workflow. Keep its source of truth in `.scratch/<feature-slug>/` so a later turn can resume without reconstructing decisions from chat. Read the current state before doing anything; never restart a completed phase or silently discard an answer.

## Non-negotiable contracts

- The main session is the only human-facing coordinator. Claude Code, Pi, and Codex CLI work in the background and write artifacts; their raw transcripts are not the user interface.
- Use Claude Code with the Opus 5 model (`--model opus`, the CLI alias for the installed latest Opus model) and make it invoke `/grill-with-docs`. If the CLI cannot resolve that model or skill, stop and report the setup problem instead of silently substituting another model or skipping the phase.
- Use Pi for web research with Google Gemini 3.8 Flash (`--provider google --model gemini-3.8-flash`). Use web research only when Claude still has an unresolved factual or design question, but complete both local inspection and web research before asking the user about that question.
- Use Codex CLI with `gpt-5.6-terra`. The installed Matt Pocock skill is named `/to-spec`, although the user may call it “to-specs”; invoke the installed name, then invoke `/to-tickets`.
- Treat `/grill-with-docs`, `/to-spec`, `/to-tickets`, and `/implement` as explicit workflow stages. Load or invoke them rather than paraphrasing them.
- Ask at most one user question in a turn. Explain the context first in beginner-friendly language, explain why the answer matters, give a recommended answer, and then ask only that one question. Record the exact answer in the state document before resuming an agent.
- A standards-based, unambiguous best solution does not become a user question. Claude must record it as a conclusion with the standard or evidence that supports it.
- An unresolved question is askable only after the local codebase, repository documentation, and relevant project state have been inspected, and Pi has performed a web search for external facts or standards. If research resolves it, update the conclusion and continue without asking.
- Do not expose a long questionnaire, raw agent output, a full ticket dump, or unexplained jargon to the user. Reveal only the next decision or the next useful summary.
- Preserve user answers and prior evidence. Append decisions; do not rewrite history to make a later conclusion look original.
- Keep implementation focused on the current task. Do not create a compatibility layer, fallback implementation, speculative abstraction, or branch unless the current task or the invoked implementation skill explicitly requires it. The workflow itself does not invent commits; follow `/implement` and repository rules when that skill requires a commit.

## State layout

At the start, derive a stable lowercase feature slug and create the run directory if it does not exist:

```text
.scratch/<feature-slug>/
├── discovery.md
├── research/
│   ├── local.md
│   └── web.md
├── spec.md
├── summary.md
└── issues/
```

Use [references/discovery-document.md](references/discovery-document.md) as the structure for `discovery.md`. Keep generated tickets in the local tracker layout expected by `/to-tickets` when this repository uses a local tracker. If the repository uses a real tracker, retain local pointers and use the tracker as the publication target; do not invent issue IDs.

Maintain this state block at the top of `discovery.md`:

```text
Phase: discovery | awaiting-user | spec | tickets | implementation | complete | blocked
Next action: <one concrete action>
Open question: <one question or None>
```

## Phase 1 — Claude discovery and grilling

1. Capture the user's task verbatim in `discovery.md`. Inspect repository instructions, project configuration, existing domain docs, relevant code, tests, history, and local tool availability. Claude owns this local exploration.
2. Run Claude Code non-interactively with the Opus 5 alias, passing the repository as its working directory and the task/state document as context. The prompt must tell Claude to invoke `/grill-with-docs`, read existing `CONTEXT.md`, ADRs, and `discovery.md`, and update `discovery.md` plus `research/local.md`.
3. Require Claude to classify every uncertainty as one of `STANDARD_CONCLUSION`, `RESEARCH_REQUIRED`, or `USER_DECISION`.
4. When `RESEARCH_REQUIRED` or `USER_DECISION` remains, run Pi with Gemini 3.8 Flash to research the unresolved facts. Give Pi only the focused question, relevant local findings, and links or paths it needs; tell it to write source links, dates, and confidence to `research/web.md`.
5. Call Claude again with the updated documents. It must re-read `discovery.md`, `research/local.md`, and `research/web.md`, re-run `/grill-with-docs`, and resolve anything that evidence settles. Repeat this research-review pair until Claude either marks the discovery `READY_FOR_SPEC` or emits exactly one `USER_DECISION` as the next question.
6. If a question remains, show the user a short explanation and that single question. Wait. On the next turn, treat the user's reply as the answer to the recorded question unless they clearly start a different request; append the answer and rationale to `discovery.md`, then resume at step 4. Never ask the next question in the same turn.
7. Finish this phase only when Claude's `/grill-with-docs` review says there are no unresolved decisions that would make implementation guess. Set `Phase: spec` and preserve the final conclusions, glossary terms, ADR pointers, scope, and answer log.

### Claude invocation contract

Use the command shape in [references/cli-contracts.md](references/cli-contracts.md). The important properties are non-interactive output, `--model opus`, repository access, and an explicit prompt to invoke `/grill-with-docs`; do not use a hidden model fallback. If a run fails, diagnose the command, credentials, skill discovery, or permissions before asking the user for any product answer.

## Phase 2 — Codex specification and tickets

1. Invoke Codex CLI with `gpt-5.6-terra`, pointed at the repository and the completed `discovery.md`. Tell it to read the current repository and docs, invoke `/to-spec` (the installed name for the requested “to-specs”), save a complete `spec.md`, and then invoke `/to-tickets` to produce dependency-ordered tracer-bullet tickets.
2. The Codex prompt must preserve the decisions already recorded. It may refine wording from repository evidence, but it must not reopen settled questions or ask the user to repeat discovery. If a genuine new decision appears, write it into `discovery.md` as one `USER_DECISION`, return to Phase 1, and do not generate implementation tickets yet.
3. Ensure the spec has problem statement, solution, extensive user stories, implementation decisions, testing decisions, out of scope, and further notes. Ensure tickets are vertical, independently verifiable slices with explicit blocking edges.
4. If `/to-tickets` asks for approval of ticket granularity or blocking edges, record its first unresolved decision in `discovery.md` and stop. The main session explains the trade-off and asks exactly one question; after the answer, resume Codex with the updated documents. Repeat until the ticket graph is approved.
5. Verify that `spec.md` exists and that each ticket is readable, has acceptance criteria, and has a blocker relationship. If the configured tracker publishes issues, verify local pointers and identifiers without fabricating missing ones.
6. Write `summary.md` with no implementation-level detail: the task goal, the high-level approach, and the few points that deserve attention. Present that short summary to the user, keeping the explanation suitable for someone unfamiliar with the repository.

## Phase 3 — Main-session implementation

1. Read `spec.md`, all ticket files, repository instructions, and the blocker graph. Work the ready frontier in dependency order; a ticket becomes ready only when every blocker is complete.
2. For exactly one ready ticket at a time, invoke `/implement` in the main session. Point it at the spec and the ticket, require test-first work at the agreed highest seam where practical, and require regular focused validation.
3. After each ticket, inspect the diff and validation result, update the ticket status and workflow state, and report only a short progress note. If implementation reveals a missing product decision, stop, record it, and return to the single-question discovery loop; do not guess.
4. Continue until all tickets are complete or a concrete blocker prevents safe progress. Run the repository's appropriate final validation when the existing project supports it. Keep unrelated failures separate from regressions caused by this work.
5. On completion, report four items in this order: task goal, progress by ticket or milestone, remaining issues, and next-step recommendation. Mention tests or checks actually run and distinguish verified results from assumptions.

## Resume and failure handling

- On every invocation, read `discovery.md` first. `awaiting-user` means the next user message is the answer checkpoint; `spec` or `tickets` means resume the corresponding CLI phase; `implementation` means continue with the next unblocked ticket.
- A tool failure is not a product question. Record the command, exit status, and actionable diagnosis in the state document. Retry only after changing the failed input or environment, and stop after the same failure has been reproduced twice.
- If a required model, CLI, skill, credential, tracker, or permission is unavailable, state the exact missing prerequisite and the smallest setup action. Do not substitute a different model or claim that the phase completed.
- Keep user-facing output progressive: one status sentence while working, one question when a decision is required, one concise phase summary, and one final handoff.
