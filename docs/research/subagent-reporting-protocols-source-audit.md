# Child reporting protocols: pinned source audit

Research for [Investigate child reporting and structured completion conventions](https://github.com/S2P2/pi-packages/issues/6), part of [Find the way to a Pi subagent package specification](https://github.com/S2P2/pi-packages/issues/1). Research only: **no protocol selected**, no production implementation, no HITL decision. This updates/extends the read-only prior legwork in `subagent-handoff-options.md`.

## Answer in brief

All four candidates remain viable experiments, but they solve different problems. Ordinary final output is broadly evidenced; a host-owned settlement notice makes missing reports and abnormal endings visible. Strict structured capture is evidenced specifically by DeepSeek's **in-process one-shot** path, not by all its providers or continuable children. Oh My Pi (OMP) requests `yield` and applies bounded reminders, but is **not universally mandatory**: successful schema-free prose and valid JSON fallback can survive missing yield. Its `strict` schema mode enforces final validity, not exclusive use of a submission tool. Pi durable's examples report ordinary answers and implement delivery deduplication in host code, not a child finish tool. [D1], [D2], [D3], [D4], [D5], [P1], [P2], [P3], [O1], [O2], [O3], [O4]

A runtime stop reason does not independently establish assignment success. Task guidance, authorization, and partial evidence need separate treatment; none of these inspected result contracts supplies a universal `blocked-needs-parent-guidance` disposition. A new disposition would be package policy, not a borrowed API fact. [D1], [D5], [P1], [O1], [O5]

## Inspected pins and method

Source trees were cloned from the first-party repositories, and code plus relevant regression tests were inspected. Pins (rather than mutable `main` URLs) own every API claim below:

| Implementation | Exact commit | Package version at pin |
|---|---|---|
| DeepSeek Harness | `5badb15009ae1756c3afe0ae0cef1faafc290ccc` | subagent `0.2.1-alpha.1` |
| Pi durable **and** Pi bundled coding-agent subagent example | `f10993bc7f28145df1375f3ff39c7f5c4cfc05f0` | durable/coding-agent `1.0.4` |
| Oh My Pi | `ed8a0f73a12a8d1b0158c973c6345382bc63cb8c` | coding-agent `18.8.2` |

Version evidence: [DeepSeek package][DV], [Pi durable package][PV], [Pi coding-agent package][PCV], [OMP package][OV]. Pi durable explicitly labels its API experimental. [P3] The local survey was treated as a lead, not evidence. This is a static source audit: cited tests were read, **not executed**, and no live-model success rates, cost, or latency are claimed. Existing glossary terms Subagent and Delegation are used; no ADRs were present.

## Decision-ready candidate comparison

The costs and proposed handling in this table are analysis, not vendor promises.

| Candidate | Primary-source precedent | Missing/invalid report | Guidance and continuation | Trade-off to measure |
|---|---|---|---|---|
| **A. Final assistant output + host lifecycle notice** | DeepSeek ordinary result and continuable settlement; durable foreground/background examples; bundled Pi example (foreground only). | Host can always state ending and absence of output; there is no schema failure without a schema. Prose never proves success. | A question can be final output; parent responds in another run. Disposition remains unknown unless explicitly declared by a separate contract. | Lowest additional reporting burden; parent interpretation and output-selection ambiguity. |
| **B. Mandatory finish/yield** | DeepSeek requires `structured_output` only in schema-bearing in-process one-shots. OMP's yield ladder is a useful bounded-enforcement precedent, **not** an exact mandatory-only contract. | Specify validation errors, enforcement budget, and explicit missing-report failure. DeepSeek uses immediate missing-capture error with **no** nudge; OMP uses up to three reminders and fallbacks. | Proposed finish variants could report done, partial, or guidance-blocked. Accepted finish ends the current run, not the saved session; parent guidance starts another run. | Typed disposition versus extra requests/forced false precision; must test terminality and sibling calls. |
| **C. Preferred submission + prose fallback** | OMP schema-free prose fallback; its schema-bearing JSON fallback is narrower than arbitrary prose. DeepSeek `send_message` guidance + unconditional settlement is a two-channel reporting precedent, not a preferred terminal tool. | Preserve prose, mark omitted disposition unknown. Do not silently convert invalid submissions into success; that policy is still to be designed. | A prose question can return without violating a strict tool requirement. New run must not reuse the prior run's accepted report. | Best-effort machine-readable reports; ambiguity and semantic duplicates remain possible. |
| **D. Optional strict schema per delegation, ordinary output by default** | DeepSeek spawn/fork in-process one-shot `outputSchema`; OMP per-spawn `outputSchema` + `schemaMode`. | Strict validity and strict tool-only submission are **different axes**. DeepSeek is tool-only with no capture => error; OMP strict mode can accept schema-valid final JSON without yield. | Choose a schema that can represent partial/guidance-blocked outcomes if those are legitimate; otherwise a blocked child cannot honestly satisfy a success-only schema. | General-purpose prose default plus machine-readable mode for consumers that need it; schema/provider compatibility burden. |

Evidence behind the table is expanded below. There is no source-based reason to infer that the package must mandate finish globally.

## 1. DeepSeek Harness: ordinary, strict one-shot, and continuable are different modes

### Ordinary one-shot output and runtime ending

`SubagentResult` contains final assistant `output`, optional `structured`, optional provider-authored `diagnostic`, and `stopReason`. The output is the last non-empty assistant message, falling back to accumulated streamed assistant text. The stop vocabulary includes `completed`, `aborted`, `error`, `max-tokens`, and `refusal`; output selection is independent of the ending, so partial output can survive. `completed` follows normal turn completion rather than external verification of the delegated assignment. Diagnostic text is not assistant-authored evidence. [D1], [D2], [D3]

### Optional strict structured capture: exactly where supported

- Spawn-in-process and fork-in-process advertise `outputSchema: true` and share the one-shot driver. The driver attaches child-local `structured_output` only when a schema is requested. The contract is an object-rooted JSON Schema in an explicitly enforced subset, not arbitrary JSON Schema. [D1], [D3], [D6], [D7]
- The child instruction says only the tool call counts. Invalid arguments become an `INVALID_ARGS` tool error and allow in-turn correction. A normal ending without accepted capture is immediately returned as error, retaining ordinary output; there is deliberately **no host re-prompt**. Abnormal stops retain their honest runtime reason. Tests assert the no-nudge behavior and request count. [D3], [D4], [DT]
- Acceptance is stronger than entering the tool body: data is staged per execution and committed only after the authoritative successful tool result. In programmatic tool-calling (PTC), capture also waits for the enclosing `run_code` result. Policy or transport failure cannot promote staged data into success. [D4], [DT]
- Accepted capture concludes the turn with no extra model request. A monotonic guard blocks **later** tool dispatch, including later calls in the same response and duplicate capture. Calls preceding capture are not undone. Tests cover both ordering directions and policy wrappers. This is concrete terminality evidence rather than a prompt convention. [D4], [DT]
- DeepSeek's ACP provider advertises no schema support. Codex and Claude Code adapters use `NO_START_CAPABILITIES`; DSH SDK adds only `agentOptions`, not `outputSchema`. Unsupported requested start features are rejected, not silently ignored. These are facts about **these Harness adapters**, not claims that the underlying Codex/Claude products cannot do structured output. [D1], [D8], [D9], [D10], [D11]
- `ContinuableStartSpec.request` expressly omits `outputSchema`; continuable children bypass the one-shot driver. Do not apply its structured completion enforcement to continuable mode. [D1], [D3]

### Continuable reporting and duplicate-looking results

Initial guidance tells the child to send self-contained results with `send_message`, and to report findings earlier when they matter. Sending a message **does not end its turn**. Independently the manager emits a host-attributed `subagent-settled` notice with ending and closing text, or an explicit no-closing-message statement. It distinguishes this from `agent-message` attribution. A test explicitly requires settlement even if the child already sent a result; content may therefore repeat across channels without being a duplicate lifecycle event. [D5], [DC]

The notice says a completed child will do no further work unless sent more. This supports a saved-child/new-activation mental model; it does not make every final answer the end of the child's session. Runtime lifecycle pairs have a run identity distinct from stable child identity. A package can borrow that distinction without adopting DeepSeek's complete continuation machinery. [D1], [D5]

## 2. Pi durable: answer-based reporting with host deduplication

The foreground example waits for an answered input (`done`, `type: input`), looks up its `AssistantEntry`, joins text blocks, and returns that text. Other settled statuses throw. It creates/reuses the child through owner-task lookup and submits with `requestId: subagent:<taskId>` for replay safety. No completion tool or task-disposition enum is attached. [P1]

The background example preserves conversations across requests and implements reporting through host-owned Reporter tasks:

- Child `send` can steer active work or queue a follow-up. Each reporter waits for settlement, then atomically records the answer ID in `reported`. Several messages can resolve to the same answer, which is reported once. Stable child and parent report `requestId`s prevent repeated submission across task replay. This is **answer-ID/request-ID dedup**, not semantic prose dedup. [P2]
- Reports are ordinary parent follow-up inputs: `[subagent <name> answered, no reply needed] <text>`. A non-abort unanswered input reports failure; an unanswered input whose reason is `aborted` deliberately produces **no report**. Thus this example is not evidence for a notify-on-every-stop contract. [P2]
- A queued report can be dropped by parent Esc, and a report arriving after Esc wakes the parent. That differs from the initiating baseline's preserve-report/suppress-auto-wake policy. Do not import this behavior accidentally. [P2]; [baseline issue #2](https://github.com/S2P2/pi-packages/issues/2)

Durable exposes `control: { terminate: true }` for custom tools. It only terminates without another model request when **every call of the tool round** requests termination; sibling calls need not be blocked and all outcomes are collected first. It is not equivalent to DeepSeek's accepted-capture dispatch guard. Also, durable's `onYield` is a generation hook, **not** OMP's model-facing `yield` submission tool. [P3], [P4]

## 3. Pi's bundled subagent extension: simple foreground prose

The bundled coding-agent example launches a separate `pi --mode json -p --no-session` process, collects completed message events, and extracts the first text block from the latest assistant message containing text (scanning backward). It does not join every text block and has no schema or finish tool. Failure classification checks process exit plus assistant `error`/`aborted`; it is not a task-success validator. Error text/stderr can take precedence over final assistant output. [P5]

Unlike durable's background example, this is foreground single/parallel/chain delegation, not a persistent background session protocol. `--no-session` means it does not supply the agreed native saved-child continuation behavior. Cancellation throws after process collection, so retained internal messages are not evidence for a reliable delivered partial-result-on-cancel contract. Copying this example unmodified would not meet that baseline. [P5]

For a package built against Pi coding-agent 1.0.4, the SDK documents `message_end` as authoritative finalized message and **`agent_settled`** as the point when Pi will not automatically continue; `agent_end` alone can precede recovery/queued work. The lifecycle signal belongs to host reporting, not model-authored task disposition. This is an integration constraint worth exercising separately from handoff enforcement. [P6]

## 4. Oh My Pi: yield-preferred, bounded, incremental, and schema-aware

### Missing and malformed submissions

OMP supplies `yield` with `{data: ...}` success or `{error: "..."}` failure. A non-empty string-array `type` means incremental sections; a string or absent `type` is terminal. A schema-free `type: "result"` can intentionally use a bound current/immediately preceding reporting turn as its data. Schema-bearing last-turn prose without accumulated sections is rejected as retryable because prose cannot satisfy that structured contract. [O2], [O3]

The executor drives to yield with up to **three reminders**, pinning tool choice on the last retry when its model API supports it. It skips nudges after a terminal model error and first settles pending owner work. The final forced yield can make an incremental submission terminal, preventing infinite responses to a pinned tool choice. This is bounded submission encouragement; it can spend additional model requests. [O1], [O4]

Finalization is more permissive than the prompts suggest:

- On a clean non-aborted run, parseable JSON (including extracted `data`) that validates is accepted even without yield, **also in `strict` schema mode**. This is validated-JSON fallback, not arbitrary-prose structured success. [O1]
- Without a usable output schema, clean non-empty prose can succeed without yield. A usable schema with no valid fallback, or an empty clean result, becomes missing-yield failure. Other failure/abort paths are not freely promoted by fallback. [O1]
- Tool validation rejects incompatible data for three consecutive attempts, then accepts the fourth with `schemaOverridden`. Permissive finalization may preserve it with an explicit warning/invalid metadata; strict finalization rejects invalid/overridden data as nonzero `schema_violation`. Unknown incremental labels under closed schemas are rejected separately. Four repeated empty untyped submissions explicitly abort rather than accepting emptiness. Read the exact counters, not merely the prose label “3 retries.” [O1], [O2], [OT]
- Caller schema/mode takes precedence over agent/session schema configuration; default mode is permissive. Invalid caller schemas (in either mode), and invalid effective schemas in strict mode, fail preflight; permissive inherited invalid schemas can proceed. Public task plumbing forwards per-spawn `outputSchema` and `schemaMode`. OMP exposes `valid`/`invalid`/`unavailable` structured metadata where available. Provider strict tool-schema representability is yet another axis: an unrepresentable declaration can fall back to a loose wire schema while final validation uses the original output contract. [O1], [O2], [O5], [O6], [OT]

### Terminality, partials, continued runs, duplicates

A successful terminal yield triggers host termination; errored tool results do not. Accepted incremental yields accumulate instead. **Pending owner async jobs make terminal yield a scheduling pause**: the executor waits for quiescence, delivers results, invalidates the older report, and demands a fresh yield. So “tool accepted” is not always “assignment run finished.” Source comments and tests expose this exception. It should not be generalized into a same-response side-effect guard like DeepSeek's. [O1], [O2], [OA]

An error yield is model-authored task failure, named `status: aborted`; finalization can set exit code zero while the returned `aborted` flag and lifecycle remain aborted. A real prior failure is not erased by successful yielded data; wall-clock timeout also wins over a yield racing teardown. Do not consume only exit code, only “success” yield status, or only structured validity. [O1], [OR]

Incremental sections can preserve partial findings. Array-schema sections accumulate as arrays; repeated scalar-schema sections replace the earlier value. There is no general exactly-once semantic dedup of repeated findings here. Workpool mode separately rejects an already-submitted item key and ends when all keys are submitted. [O2], [O7]

Cancelled/timeout runs have a last-assistant-text salvage path when no output remains; yielded data and invalid parsed structured data can remain available in artifacts. Follow-up runs avoid overwriting an earlier authoritative artifact when they produce no yield; new output clears stale JSON sidecars rather than leaving old structured data attached to a new result. These are useful continuation hazards to test even if the package does not copy OMP's artifact scheme. [O1], [OS]

## 5. Guidance questions are not authorization

No universal blocked-guidance protocol was found in these inspected completion paths. They permit communicating a question in prose/data, or in DeepSeek's continuable messages; communicating and authorizing an action remain different concerns. [D1], [D5], [P1], [P2], [O2]

Provider/mode-specific authorization examples make a global generalization unsafe:

- DeepSeek's **ACP adapter** can select an allow option under deployment `permission: allow`, otherwise responds cancelled. It is configured policy, not parent task guidance. [DA]
- Its **Claude Code adapter** disables `AskUserQuestion`, denies interactive permission requests except in explicitly configured bypass mode, declines MCP elicitation, and cancels user dialogs. [DL]
- Its **Codex adapter** declines/cancels command/file interactive approvals, grants no additional requested turn permissions, returns empty user-input answers, and declines elicitation; native unattended permission mode is separately configured. [DX]
- OMP sets child `tools.approvalMode: yolo` on the basis that parent task approval is its authorization boundary, while retaining user approval policies. That is an explicit architectural difference from the initiating baseline's unsupported-child-approval-fails-closed requirement, **not a policy recommendation for this package**. [O8]; [baseline issue #2](https://github.com/S2P2/pi-packages/issues/2)

Experimentally, distinguish an informational question (“which acceptance criterion applies?”) from an authorization request (“may I run this privileged command?”). The parent model can answer the former without thereby acquiring human authority for the latter. Under a finish protocol, a blocked-guidance report should end only its current run; resuming with guidance must be explicit and should not count the earlier partial report as final success. These are proposed semantics to test, not already-settled API terms.

## Bounded experiment proposal (not executed; no selection)

Build one throwaway runner with the same child session backend, task corpus, tools, and host notification envelope; vary only A/B/C/D. Do not compare OMP's entire scheduler against the bundled Pi subprocess example and attribute every difference to handoff.

1. **Deterministic contract fixtures first.** Script model responses for clean prose, no text, thinking-only, valid/invalid/missing/duplicate submission, schema-invalid then corrected, rejected authoritative result, submission then side-effect sibling (both call orders), cancellation/timeout before and after capture, provider error/truncation/refusal, and partial incremental findings. Assert the selected output and runtime ending separately from submitted disposition. Count extra requests precisely.
2. **Lifecycle/continuation fixtures.** Have a child ask for task guidance, return partial evidence, receive a follow-up, and complete in a new run. Exercise steering while active, several requests resolving to one answer, old accepted report surviving into a new run, delayed/duplicate delivery, child stop, parent interruption, and parent session replacement. Exercise nested owned work settling after a submitted report. Parent receives one logical settlement per run; progress messages and settlement may coexist but need distinct attribution. Preserve evidence on failure. Evaluate the baseline's pending/no-auto-wake rule independently of protocol A/B/C/D.
3. **Authorization fixtures.** Simulate an unsupported approval/dialog in headless mode. Require explicit denial and report it separately from guidance, without asking the parent model to impersonate a human. No privileged real actions are needed.
4. **Small paired live-model trials only after fixtures.** Use pinned model/provider/API/effort settings, fixed schema subset and known tool exposure; run the same research, code-edit, review, and ambiguous-guidance assignments across candidates. Include both direct calls and codemode/nested transport where supported. An unsupported provider/mode is a rejected configuration, not an unexplained trial failure.

Record: usable-result rate; omitted disposition (including “unknown”); truthful guidance-blocked/partial reports; tool/schema correction rate; duplicate lifecycle and repeated-content counts separately; side effects after accepted finish; requests spent solely on enforcement; tokens/cost/p50-p95 latency; artifact/evidence retrieval success; continuation correctness; parent workflow usability. Pre-register safety gates (no false authorization, no cross-parent routing, no swallowed abnormal endings). Human evaluation chooses the acceptable enforcement budget and usability trade-off; this research does not.

A useful **provisional experiment envelope**, not a spec, records stable child/session identity, unique run identity, runtime stop reason, optional child-declared task disposition, report provenance/channel, submission validity, summary/partial evidence/artifact references, and a guidance question when present. Missing disposition stays unknown. Test whether this envelope works across all four candidates before choosing final names/schema.

## Known unknowns and decision boundaries

- Static tests are implementation evidence, not measured model reliability. No live-model benchmark or project prototype was run here; cost and latency comparisons remain hypotheses.
- A tool name/prompt does not establish terminal dispatch safety. DeepSeek has an explicit guard; durable termination is round-wide; OMP additionally has pending-async-work semantics. Exact Pi extension enforcement, concurrent sibling behavior, and codemode terminal propagation need a package prototype.
- Strict schemas are not universally portable. DeepSeek enforces a subset; OMP normalizes/loosens tool schemas when necessary. Trial/provider support must be stated explicitly rather than inferred from a generic “structured output” feature.
- How much task disposition to require, and whether blocked guidance terminates or parks a run, are deferred human choices. No inspected generic stop vocabulary resolves those questions.
- Neither answer-ID dedup nor a terminal guard guarantees end-to-end exactly-once notification after every possible host failure. Package delivery IDs, persisted pending reports, and consumption/acknowledgment rules still need specification; automatic crash recovery is outside the current baseline.
- Cross-workflow usability with chief-of-staff and implement-spec, enforcement limits, final terminology, artifact schema, and authorization UX remain downstream decisions. This asset provides candidates and falsifiable tests, not a winner.

## Primary sources (commit-pinned)

[D1]: https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent/src/types.ts
[D2]: https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent/src/assistant-output.ts
[D3]: https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent-in-process-driver/src/index.ts
[D4]: https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent-in-process-driver/src/structured.ts
[D5]: https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent/src/continuation-messages.ts
[D6]: https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent-spawn-in-process/src/index.ts
[D7]: https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent-fork-in-process/src/index.ts
[D8]: https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent-acp/src/index.ts
[D9]: https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent-codex/src/index.ts
[D10]: https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent-claude-code/src/index.ts
[D11]: https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent-dsh-sdk/src/index.ts
[DT]: https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent-in-process-driver/tests/structured.spec.ts
[DC]: https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent/tests/continuation.spec.ts#L2715-L2751
[DA]: https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent-acp/src/run.ts#L430-L451
[DL]: https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent-claude-code/src/run.ts#L316-L367
[DX]: https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent-codex/src/wire.ts#L573-L627
[DV]: https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/subagent/package.json
[P1]: https://github.com/earendil-works/pi/blob/f10993bc7f28145df1375f3ff39c7f5c4cfc05f0/packages/durable/test/examples/22-subagent-foreground.ts
[P2]: https://github.com/earendil-works/pi/blob/f10993bc7f28145df1375f3ff39c7f5c4cfc05f0/packages/durable/test/examples/23-subagent-background.ts
[P3]: https://github.com/earendil-works/pi/blob/f10993bc7f28145df1375f3ff39c7f5c4cfc05f0/packages/durable/README.md
[P4]: https://github.com/earendil-works/pi/blob/f10993bc7f28145df1375f3ff39c7f5c4cfc05f0/packages/durable/src/harness/generation.ts#L593-L649
[P5]: https://github.com/earendil-works/pi/blob/f10993bc7f28145df1375f3ff39c7f5c4cfc05f0/packages/coding-agent/examples/extensions/subagent/index.ts
[P6]: https://github.com/earendil-works/pi/blob/f10993bc7f28145df1375f3ff39c7f5c4cfc05f0/packages/coding-agent/docs/sdk.md#subscribing-to-events
[PV]: https://github.com/earendil-works/pi/blob/f10993bc7f28145df1375f3ff39c7f5c4cfc05f0/packages/durable/package.json
[PCV]: https://github.com/earendil-works/pi/blob/f10993bc7f28145df1375f3ff39c7f5c4cfc05f0/packages/coding-agent/package.json
[O1]: https://github.com/can1357/oh-my-pi/blob/ed8a0f73a12a8d1b0158c973c6345382bc63cb8c/packages/coding-agent/src/task/executor.ts
[O2]: https://github.com/can1357/oh-my-pi/blob/ed8a0f73a12a8d1b0158c973c6345382bc63cb8c/packages/coding-agent/src/tools/yield.ts
[O3]: https://github.com/can1357/oh-my-pi/blob/ed8a0f73a12a8d1b0158c973c6345382bc63cb8c/packages/coding-agent/src/prompts/tools/yield.md
[O4]: https://github.com/can1357/oh-my-pi/blob/ed8a0f73a12a8d1b0158c973c6345382bc63cb8c/packages/coding-agent/src/task/executor.ts#L2296-L2528
[O5]: https://github.com/can1357/oh-my-pi/blob/ed8a0f73a12a8d1b0158c973c6345382bc63cb8c/packages/coding-agent/src/task/structured-subagent.ts
[O6]: https://github.com/can1357/oh-my-pi/blob/ed8a0f73a12a8d1b0158c973c6345382bc63cb8c/packages/coding-agent/src/task/index.ts
[O7]: https://github.com/can1357/oh-my-pi/blob/ed8a0f73a12a8d1b0158c973c6345382bc63cb8c/packages/coding-agent/src/task/yield-assembly.ts
[O8]: https://github.com/can1357/oh-my-pi/blob/ed8a0f73a12a8d1b0158c973c6345382bc63cb8c/packages/coding-agent/src/task/executor.ts#L1119-L1125
[OT]: https://github.com/can1357/oh-my-pi/blob/ed8a0f73a12a8d1b0158c973c6345382bc63cb8c/packages/coding-agent/test/tools/yield-extraction.test.ts
[OA]: https://github.com/can1357/oh-my-pi/blob/ed8a0f73a12a8d1b0158c973c6345382bc63cb8c/packages/coding-agent/test/task/executor-async-quiescence.test.ts
[OR]: https://github.com/can1357/oh-my-pi/blob/ed8a0f73a12a8d1b0158c973c6345382bc63cb8c/packages/coding-agent/test/task/executor-subagent-reminders.test.ts
[OS]: https://github.com/can1357/oh-my-pi/blob/ed8a0f73a12a8d1b0158c973c6345382bc63cb8c/packages/coding-agent/test/task/executor-structured-sidecar.test.ts
[OV]: https://github.com/can1357/oh-my-pi/blob/ed8a0f73a12a8d1b0158c973c6345382bc63cb8c/packages/coding-agent/package.json
