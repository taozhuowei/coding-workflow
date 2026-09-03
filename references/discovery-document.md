# Discovery Decision Document

Use this template for `.scratch/<feature-slug>/discovery.md`. Claude may add detail under the existing headings, but it keeps the state block, decisions, and answer log intact.

```md
# <Task title>

Phase: discovery
Next action: <one concrete action>
Open question: None

## User request

<The user's task description, preserved verbatim.>

## Conclusions

### C-001: <conclusion>

- Classification: STANDARD_CONCLUSION | RESEARCH_REQUIRED | USER_DECISION
- Decision: <what the workflow will do>
- Why: <user-facing rationale>
- Evidence: <repository path, standard, ADR, or research source>
- Status: Open | Decided

## Scope

### In scope

- <observable outcome>

### Out of scope

- <explicit boundary>

## Domain language

- **<Term>**: <precise meaning used in this task>

## Local evidence

See `research/local.md` for paths inspected, commands run, relevant history, and conclusions.

## Web evidence

See `research/web.md` for focused queries, source URLs, publication dates, and confidence.

## Open questions

Keep only questions that remain after local and web research. List them in dependency order; the coordinator asks only the first one.

### Q-001: <one question>

- Why this matters: <concrete consequence>
- Recommended answer: <default recommendation>
- Options: <short mutually exclusive options, if needed>
- Research completed: local + web
- Status: Open | Answered
- Answer: <exact user answer, when answered>

## Answer log

| Question | User answer | Recorded decision | Date |
| --- | --- | --- | --- |

## Review gate

- Grill status: READY_FOR_SPEC | NEEDS_USER_DECISION | BLOCKED
- Last reviewed by: Claude Code / grill-with-docs
- Review note: <why no implementation-relevant ambiguity remains>
```

## Writing rules

- Conclusions describe observable decisions and their rationale, not a transcript.
- Facts belong in evidence sections; user choices belong in the answer log and the relevant question.
- A standard conclusion includes the standard or repository evidence that makes asking unnecessary.
- An open question is singular, concrete, and answerable. Keep later dependent questions in the document but do not ask them until their prerequisites settle.
