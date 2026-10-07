# Subagent handoff options

## Decision status

The handoff protocol is deliberately unresolved. Compare approaches experimentally before selecting one; the canonical decision belongs to [Choose the child handoff and guidance protocol](https://github.com/S2P2/pi-packages/issues/8). This note is prior research, not an approved specification.

## Candidates to test

1. Ordinary final assistant response plus host-owned completion notice.
2. Mandatory explicit child completion tool with validation and bounded enforcement.
3. Preferred explicit child completion tool with a final-response fallback; omitted task outcome stays unknown rather than inferred from prose.
4. Optional strict structured-output mode per delegation, with ordinary final responses as the default.

Test completed assignments, guidance blockers, permission denial, cancellation, provider failure, omitted tool calls, malformed submissions, continuation, and nested delegation. Compare result usability, omitted/duplicate notifications, extra model requests, latency, cost, and workflow compatibility. No candidate is approved yet.

## DeepSeek Harness primary-source findings

Inspected commit: `5badb15009ae1756c3afe0ae0cef1faafc290ccc`. This is a developer-preview implementation, not a stable compatibility promise.

- The result contract separates `output`, optional `structured`, optional non-assistant `diagnostic`, and runtime `stopReason`. Known stop reasons include completed, aborted, error, max-tokens, and refusal. Completed describes normal turn completion, not independently verified assignment success. [Types](https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent/src/types.ts)
- Ordinary output is the last non-empty assistant message, falling back to accumulated streamed assistant text. Selection is independent of the stop reason, preserving potentially partial output. [Output selection](https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent/src/assistant-output.ts)
- In-process one-shot delegation optionally accepts an output schema. Only when supplied does the driver attach the child-scoped `structured_output` tool. [Driver](https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent-in-process-driver/src/index.ts)
- Structured mode explicitly requires the tool rather than plain text, validates arguments, concludes the turn, commits capture only after the authoritative tool result succeeds, and guards later dispatch after accepted capture. If a normally completed schema-requested run lacks a valid capture, the driver reports error rather than silently treating prose as structured success. [Structured runtime](https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent-in-process-driver/src/structured.ts), [Driver](https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent-in-process-driver/src/index.ts)
- Continuable children receive guidance to send self-contained results to the parent with `send_message`, including earlier findings when relevant. Sending does not end the turn. Independently, the manager builds a host-owned settlement notice containing the stop reason and closing text, or an explicit no-closing-message notice. Model-authored messages and host-owned settlement notices have distinct attribution. [Continuation messages](https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent/src/continuation-messages.ts)

## Implication for this design

DeepSeek is evidence for separating ordinary reporting, optional strict structured capture, and runtime lifecycle notifications. It does not establish that our package should mandate a finish tool, nor that task-guidance questions are authorization. A guidance-blocked task disposition would be part of our protocol, not one of the inspected generic DeepSeek stop reasons.
