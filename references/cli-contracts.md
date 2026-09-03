# CLI Contracts

These are invocation shapes, not a promise that credentials or local installation are present. Resolve paths from the current repository and quote paths containing spaces.

## Claude Code

Use a non-interactive run so only the main session talks to the user:

```powershell
claude -p --model opus --add-dir "$repo" --output-format text @"
Read $runDir/discovery.md, the repository instructions, existing CONTEXT.md/CONTEXT-MAP.md, and relevant ADRs.
Invoke /grill-with-docs. Inspect the repository locally before making any user-facing decision.
Update $runDir/discovery.md and $runDir/research/local.md. Preserve existing answers and evidence.
Classify uncertainty as STANDARD_CONCLUSION, RESEARCH_REQUIRED, or USER_DECISION. Return READY_FOR_SPEC only when no implementation-relevant ambiguity remains.
Task: $task
"@
```

`opus` is intentional: it is Claude Code's stable alias for the installed latest Opus model. Confirm the CLI accepts it before starting the phase; an unavailable Opus 5 model is a setup blocker, not a reason to pick Sonnet or another provider.

If the phase needs another review after Pi research, run the same command again with the same state files. The document is the durable memory; do not depend on the most recent terminal transcript.

## Pi web research

Use a focused question and ask for source-backed findings:

```powershell
pi --provider google --model gemini-3.8-flash --print @"
Research this unresolved question using current primary or authoritative sources: $question
Context from the repository: $localFindings
Write concise findings, source URLs, publication dates when available, confidence, and what decision the evidence supports to $runDir/research/web.md.
Do not invent repository facts and do not make the product decision on the user's behalf.
"@
```

If Gemini 3.8 Flash is not available in Pi's model catalog or the run cannot reach the web, record that exact blocker. Do not silently use native search, another model, or an unsourced answer.

## Codex CLI

Run the synthesis phase without an interactive user checkpoint:

```powershell
codex exec -m gpt-5.6-terra -C "$repo" -s workspace-write -o "$runDir/codex-last-message.md" @"
Read $runDir/discovery.md and all repository instructions and relevant docs.
Invoke /to-spec (the installed skill name for the requested to-specs stage), save the complete result to $runDir/spec.md, then invoke /to-tickets and save one dependency-ordered ticket per file under $runDir/issues/ when the configured tracker is local.
Preserve settled discovery decisions. Do not interview the user. If a genuinely new product decision is unavoidable, record exactly one USER_DECISION in discovery.md and stop before generating tickets.
Task: $task
"@
```

Use the installed `/to-spec` name rather than inventing `/to-specs`. If the skill setup says a tracker is not configured, stop and report that prerequisite; do not fabricate issue publication or labels.

## Implementation

For each ready ticket, the main session invokes `/implement` with pointers to `spec.md` and that ticket. Keep the ticket graph as the source of order. The workflow does not invent a branch or use a dangerous bypass flag; follow `/implement` and repository rules, including its commit contract when applicable.
