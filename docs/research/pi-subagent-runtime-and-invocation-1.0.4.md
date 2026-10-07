# Pi subagent execution × invocation: capability comparison

Research for [Compare Pi session execution and tool invocation capabilities](https://github.com/S2P2/pi-packages/issues/3), 2026-10-07. **Evidence, not a backend, exposure-policy, or human-authorization decision.** “Subagent” and “delegation” use the root glossary. The baseline in [Validate the Pi subagent design baseline](https://github.com/S2P2/pi-packages/issues/2) remains a human-validation ticket.

## Pins and method

- Installed `@earendil-works/pi-coding-agent`, `pi-agent-core`, `pi-ai`, and `pi-codemode`: **1.0.4**, under `/home/jo/.pi/agent/install/releases/1.0.4/node_modules/@earendil-works/`. Inspected exported declarations, docs, examples, and upstream source at tag `v1.0.4`, commit **`7c10bd4337495ee613f2224843ecdf349b80d1df`**. Links below are immutable commit links. [Package/export manifest][manifest], [public exports][exports].
- Also inspected the relevant source diff against upstream main snapshot **`f10993bc7f28145df1375f3ff39c7f5c4cfc05f0`**, not a released compatibility target. It adds SDK `tools: ["+name", "-name"]` modifier lists and changes codemode output formatting; **do not backport those assumptions to 1.0.4**. [Snapshot SDK][main-sdk], [snapshot codemode][main-code].
- Primary evidence is Pi-owned source/docs/tests. The user's cross-harness survey and prior local handoff/human-control notes are context only. No production changes, live model calls, approval choices, or handoff-protocol selection were made.

## Decision-ready summary

**Two independent axes:** SDK versus RPC determines where the *child session executes*. Direct versus codemode determines how the *parent model invokes delegation operations*. All four combinations are technically possible through an extension/tool adapter; neither invocation form supplies a child manager automatically. [SDK factory][sdk], [RPC protocol][rpc], [tool contracts][extensions].

| Axis | Gain | Cost / boundary |
|---|---|---|
| In-process SDK child | Full typed session access, custom child tools and UI bindings, direct event subscriptions; no JSON transport | Shared process, globals, memory, permissions, and failure domain; host must manage extension startup and shutdown |
| RPC subprocess child | Separate process/heap and normal CLI startup/discovery; bidirectional protocol supports continuing sessions | Serialization, process and pipe supervision, protocol/UI adapter, startup overhead; custom child tools need loaded child extensions rather than parent closures; not a filesystem/network sandbox |
| Direct delegation tools | Explicit model-visible parameter schema and per-operation tool-result transcript; no script required | More declarations/results in model context; batching/filtering must live in tool implementation or separate calls |
| Codemode delegation tools | JS batching, parallel calls and result filtering; structured programmatic values | Script output can omit results; nested calls are not separate transcript messages; script cancellation applies to pending calls, not a durable child-ownership policy |

The first two rows follow [SDK][sdk-doc], [RPC][rpc], and [security][security]; the latter two follow [codemode][codemode], [extensions][extensions], and [nested-call implementation][nested]. This comparison does not assert a measured performance winner.

## Execution capabilities

| Requirement | In-process SDK | RPC subprocess | Package responsibility / caveat |
|---|---|---|---|
| Fresh construction | `createAgentSession({cwd, sessionManager: SessionManager.create(cwd, dir, {parentSession})})` | Spawn CLI in child `cwd`, without `--no-session`; `new_session` accepts `parentSession` | Persisted is default. Fresh factory construction does not inherit the parent model, settings object, tools, or runtime automatically. [sdk][sdk] [manager][manager] [commands][commands] |
| Completed-turn history fork | Open a **separate manager**, then `createBranchedSession(entryId)` copies the selected ancestry; pass that manager to a new child factory | Host can prepare the branch file and launch with `--session <path>`; RPC `fork`/`clone` replace that subprocess's own active session | Do not fork the live parent's manager/runtime: these methods mutate/replace their receiver. RPC `fork` is before a user entry; `clone` is at current leaf. SDK runtime `fork(entryId,{position:"at"})` accepts an entry but still replaces its own active session. [manager][manager] [runtime][runtime] [commands][commands] |
| Cross-working-directory fork | `SessionManager.forkFrom(sourcePath,targetCwd,dir)` | CLI `--fork` uses native construction | **`forkFrom` copies all non-header entries, not just the active branch**, and reconstructs leaf from file order. It is not by itself the baseline's selected completed-turn branch copy. Select/snapshot the desired branch explicitly before relocating. No native “only balanced completed turns” option was found. [manager][manager] [cli][cli] |
| Durable log / manual resume | `SessionManager.open(path)`, `list`, `continueRecent`; native v3 JSONL | `--session`, `--continue`, `--resume`; RPC `switch_session`, entries/tree/messages/export | File materializes once a user or assistant message exists, not merely on construction. Completed messages persist; in-flight deltas are not continuously checkpointed. Orderly cancellation can retain partial assistant output; a hard kill cannot promise the unsaved streaming tail. [manager][manager] [format][format] [messages][messages] |
| Skills/extensions/context | `DefaultResourceLoader` at explicit `cwd`; resource overrides/exclusions and custom tools | Normal CLI resource discovery and `--no-*`, explicit resource flags | Normal discovery is **rediscovery**, not cloning parent's live extension closure, inbox, children, or mutable objects. Persisted custom entries/tool details *can* be copied in a history fork and extensions may rebuild state from them. Filter those deliberately where necessary. [loader][loader] [extensions][extensions] |
| MCP / codemode | Must add `createMcpExtension`, `createCodemodeExtension`, `createToolSearchExtension` as applicable, then `session.bindExtensions({…})` to emit startup | CLI installs these built-ins unless disabled/replaced | SDK defaults do **not** load CLI built-ins. MCP connects on `session_start`; extension replacements can alter behavior. Tool allowlists alone do not remove all indirectly callable MCP tools. [sdk-doc][sdk-doc] [sdk-example][sdk-example] [mcp][mcp] |
| Model / thinking per run | Explicit factory `model` and `thinkingLevel`; `session.setModel(model)`, `setThinkingLevel(level)` before subsequent prompt | Startup `--provider/--model/--thinking`; `set_model`, `set_thinking_level`, inspect `get_state` | Host resolves preset/explicit/parent precedence. Resume factory can substitute a model and return `modelFallbackMessage`; thinking is clamped to model capabilities. Resolve exact provider/ID, reject/report fallback, and inspect effective thinking rather than promising exact unsupported effort. CLI model matching may be fuzzy. [sdk][sdk] [session][session] [commands][commands] [cli][cli] |
| Steering / follow-ups | `steer`, `followUp`, or streaming `prompt` with explicit behavior | `steer`, `follow_up`, streaming `prompt` behavior | Steer arrives after current turn's tool calls, not instantaneous preemption. Follow-up waits for pending work. Queuing acceptance is not delivery/completion; input handlers can consume/transform input. To start an idle next run use `prompt`, not an assumed wake from a queued follow-up. [sdk-doc][sdk-doc] [commands][commands] |
| Final settlement | `message_end` authoritative; subscribe to `agent_settled`; accepted SDK `prompt()` resolves after run/recovery | Subscribe before `prompt`; acceptance `disposition` is separate from settlement | `agent_end` may precede retries, recovery or queued work. `agent_settled` has no assignment-success verdict or run ID. Inspect stop reason/output; host tracks run identity and task disposition. `agent_before_settle` can request continuation, but is not a final report. [json][json] [extensions][extensions] |
| Cancel / timeout | `abort()` waits for idle; `clearQueue()` to remove pending inputs; `dispose()` is synchronous | RPC `clear_queue`, `abort`; EOF requests runtime disposal; own process deadline and signal escalation | **Abort alone can continue queued inputs**. Neither factory supplies a whole-delegation deadline or shared-tree cancellation. Native transient-request retries are distinct from restarting the whole assignment; a timeout budget must include retry/recovery time. Await bounded abort/cleanup before discarding listeners; tools that ignore signals can stall cooperative shutdown. [session][session] [commands][commands] [rpc-mode][rpc-mode] |
| Session replacement / reload / exit | Runtime replacement awaits abort, emits shutdown, invalidates old contexts, creates new services; reload emits shutdown with `reason:"reload"` | CLI runtime manages same lifecycle; orderly stdin EOF disposes runtime | Bare `session.dispose()` does **not** emit `session_shutdown`; built-in MCP closes connections in that handler. `AgentSessionRuntime.dispose()` emits shutdown then calls synchronous disposal; explicitly settle active work if transcript-tail retention matters. Subscriptions must be rebound after replacement. Host-owned child tree must implement idempotent shutdown/report suppression on these boundaries. [runtime][runtime] [session][session] [mcp-source][mcp-source] |
| Lineage / depth / shared active cap | Native header has `parentSession`; custom entries can hold extra metadata | `new_session.parentSession` and native fork headers | Lineage metadata is not an ownership controller. Neither exported session factory nor RPC command union implements delegation depth/tree budgets. A package manager must reserve/release active-run slots atomically, enforce depth, and reject capacity explicitly. For RPC descendants, independent per-process counters cannot enforce a tree-wide cap without shared authority. [sdk][sdk] [rpc-types][rpc-types] [manager][manager] |
| Usage | `getSessionStats`, assistant/tool usage, usage entries, compaction/summary usage | `get_session_stats`, messages/events and persisted entries | Full-session totals include abandoned/compacted history. Forks copy historical usage; summing child totals double-counts inherited history. Capture a run baseline or count newly appended usage-bearing entries. Detached child costs do not automatically appear in the parent; package aggregates once. Counts/cost are reported estimates, not an independent billing ledger. [stats][stats] [format][format] [nested][nested] |
| TUI / headless | Session core reusable; bind host mode/UI explicitly. Default extension runner is print/no-op UI | RPC has dialog protocol; print/JSON no interactive UI | `ctx.hasUI` is true in RPC but not evidence of a terminal. Guard terminal components with `ctx.mode === "tui"`; human list/inspect/stop presentation requires a mode-aware adapter. [runner][runner] [rpc-ui][rpc-ui] [extensions][extensions] |

### Approval boundary: neither backend is automatically safe

Pi does not ask approval for every tool call, and all enabled tools/extensions retain OS permissions. A subprocess changes the process boundary, not file/network authority. [Security][security]

- SDK no-op `confirm` returns `false`; select/input/editor return `undefined`. A permission gate that explicitly blocks on non-approval fails closed, but an extension that skips gates when `!hasUI` does not. Supply explicit denial/reporting bindings or verified policy hooks; do not treat a task answer from the parent model as authorization. [Runner defaults][runner], [permission example][extensions]
- RPC dialogs emit `extension_ui_request` and wait. A parent adapter must send a matching cancelled/false response for unsupported approvals and record the denial; ignoring an untimed request can hang the child. `RpcClient` 1.0.4 has no public typed UI-response method; custom JSONL transport or a child-side policy extension is needed for that path. [UI protocol][rpc-ui], [client source][client]
- CLI project trust and tool approvals are separate. CLI headless trust defaults skip protected project resources when no decision is available. **Raw SDK `SettingsManager.create` defaults `projectTrusted` to true**; `createAgentSession()` does not invoke the CLI trust chooser. To match intended discovery/authorization semantics use explicit trust state and `DefaultResourceLoader.reload({resolveProjectTrust})`/service setup. Never silently pass `--approve` merely to obtain discovery parity. [Security][security], [settings source][settings-source], [loader][loader], [sdk][sdk]

## Invocation and structured returns

`ToolDefinition.exposure` is an actual exported contract, not a binary direct/codemode toggle: [types][tool-types], [extensions][extensions].

| Exposure | Declared to model | Callable through `ctx.executeTool` / codemode |
|---|---|---|
| `direct` (default) | While active | While active |
| `model-only` | While active | Never |
| `codemode` | If explicitly activated | Whenever registered; listed in codemode subject to description budget |
| `deferred` | If activated/searched | Whenever registered; discoverable rather than inline-listed |
| `hidden` | Never | Never |

With codemode enabled, `mode:"on"` retains direct declarations; `mode:"only"` hides callable declarations while leaving tools active/callable. This changes exposure, **not execution backend**. SDK callers must install/activate codemode explicitly. Scripts have no Node/network/filesystem/timers and cannot call codemode recursively. [Codemode][codemode], [settings][settings]

Return contracts are distinct:

- Every custom tool returns model-facing `content` plus `details` (possibly `undefined`). `details` is host/rendering state, **not** codemode's return value.
- Declare `outputSchema` and return matching JSON `structuredContent` for programmatic data. Direct model calls still receive `content`. Codemode returns structured content when present with an output schema, otherwise text; errors without structured content reject. `isError:true` plus structured content resolves as data, so a delegation result schema needs an explicit status/error discriminator. MCP scripts receive the complete `CallToolResult`, including `isError` and `structuredContent`.
- Source describes output-schema matching as the author's contract; the inspected execution path validates **input arguments**, not a universal output-schema validator. Validate structured data in the package; do not assume Pi verifies arbitrary returned data. Content-redaction handlers must handle structured content too.
- Nested calls run through the same argument validation and permission hooks and emit events with `parentToolCallId`; they do not create individual transcript tool-result entries. Pi keeps bounded call metadata (256 calls, 8 KiB per-call/32 KiB total arguments), never nested results. Result/artifact durability must be provided by the delegation manager/native child log, not by the script's printed output. [Extensions][extensions], [script bridge][execute], [agent pipeline][pipeline], [nested calls][nested]

### What survives script completion/cancellation?

Source ownership is precise: the codemode host maintains an abort controller for each **pending** tool call and aborts those on normal return, exception, timeout, cancellation or sandbox close. Resolved calls have been removed from that pending map; the host does not later abort their call signals. Side effects are not rolled back. Successful `store()` writes persist branch-relative IDs/data, not live child sessions; failed-script writes are discarded. [Sandbox host][sandbox], [bridge][execute], [codemode docs][codemode]

**Implication for background delegation (not a chosen API):** await a bounded launch/registration operation that returns a child/run handle. The long-lived child needs a **manager-owned** controller and session/process registry, not the parent tool/script signal. Before establishment, cancellation must clean up or roll back a provisional launch. Once established, completed-script cancellation must not kill that child. A tool that awaits the child's entire run is still pending and remains script-owned. An unawaited `tools.launch()` is therefore not a reliable detachment mechanism. Do not use either the launch tool's signal or `ctx.signal` as the established child's lifetime signal. Likewise, merely caching `ctx`/`pi` in a long-lived closure is unsafe after parent reload/switch; contexts are invalidated. [Sandbox][sandbox], [runtime][runtime], [runner][runner]

Manager-owned run IDs, idempotent registration and originating-parent identity are needed to recover the handle when launch succeeded but its script failed before printing/storing it. Native persistence preserves the child conversation, not parent report queues or a cross-process ownership registry. This follows from the contracts above; it is a package design obligation, not built-in Pi behavior.

## Source-checked sharp edges and verification

Credential-free Node probes against the **installed 1.0.4** packages passed:

1. Fresh persistent session has a path but no file; appending a user message creates it.
2. `forkFrom` copied both experimental branches; `createBranchedSession(selectedEntry)` on a separate manager copied only selected ancestry and left the source manager ID unchanged.
3. At codemode return, an awaited/resolved launch call's signal remained un-aborted while an unawaited hanging call received abort. `sandbox.close()` did not abort the resolved signal. Cancelling an active script aborted its pending call.
4. Bare SDK `session.dispose()` did not dispatch an installed `session_shutdown` handler.

These were small throwaway probes outside the repository, not a full integration suite. Relevant upstream evidence/tests: [manager implementation][manager], [sandbox tests][sandbox-tests], [runtime event tests][runtime-tests].

Additional source discrepancy: docs recommend `RpcClient.promptAndWait()` for race-free subscription, but **1.0.4's implementation unconditionally waits for settlement even if `prompt()` returns `"handled"`**. Its `collectEvents`/`waitForIdle` waits are event-based timeouts, not process-exit-aware completion supervision. Its `stop()` stops stdout reading, sends SIGTERM and escalates to SIGKILL after one second; it is not the protocol's EOF-and-drain orderly-close sequence. Use disposition-aware completion and explicit process supervision rather than assuming that helper satisfies all package lifecycle requirements. [Client][client], [RPC docs][rpc]

The official [subagent example][subagent] uses one-shot JSON subprocesses with `--no-session` and caller-signal-linked killing. Its concurrency limit is per batch, not a shared delegation-tree cap. It is useful for display/streaming patterns, **not** evidence that the persistent/background/parent-interruption baseline is already implemented.

## Known unknowns / downstream decision inputs

- No provider-backed SDK/RPC parity, multi-child extension coexistence, platform process-tree cleanup, or TUI interaction test was run. OS process-group cleanup and non-cooperative tools need explicit verification before promising subtree stop.
- In-process sessions share process globals: reload calls `resetApiProviders()` and resource loading clears extension caches. Independent conversation state is not proof that arbitrary provider/extensions are multi-session-safe. Test the supported extension set; RPC reduces this particular shared-global risk. [Session reload][session], [loader][loader]
- A request-count budget is not a native delegation limit. Assistant-turn counts omit summarization/cache-warming/nested model requests; define the counted operations before enforcing such a limit. Retry controls are not equivalent to a total request budget. [Stats][stats], [format][format], [settings][settings]
- Native fork APIs do not guarantee a selected completed-turn boundary, exclusion of copied extension state, or prompt-cache hits. The package must specify its snapshot boundary and test copied custom/system/compaction entries against fresh child resources. Cache behavior depends on provider and actual request prefix. [Manager][manager], [extensions][extensions]
- How unsupported approvals are reported, whether effective thinking clamping is rejected versus disclosed, the result schema, ownership/auto-wake suppression, handoff enforcement and v1 exposure/backend remain **human decisions**. Research establishes the available hooks and gaps, not those choices.
- Pin a supported Pi version/range and test against it: main already differs from 1.0.4. No backwards-compatibility promise for older `@mariozechner/*` packages or pre-codemode APIs is established here.

[manifest]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/package.json
[exports]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/index.ts
[sdk]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/core/sdk.ts
[sdk-doc]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/sdk.md
[sdk-example]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/examples/sdk/14-codemode-mcp.ts
[manager]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/core/session-manager.ts
[runtime]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/core/agent-session-runtime.ts
[session]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/core/agent-session.ts
[stats]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/core/agent-session.ts#L4163-L4220
[loader]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/core/resource-loader.ts
[settings-source]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/core/settings-manager.ts#L384-L460
[runner]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/core/extensions/runner.ts
[tool-types]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/core/extensions/types.ts
[extensions]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/extensions.md
[rpc]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/rpc.md
[commands]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/rpc-commands.md
[rpc-ui]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/rpc-extension-ui.md
[rpc-types]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/modes/rpc/rpc-types.ts
[rpc-mode]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/modes/rpc/rpc-mode.ts
[client]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/modes/rpc/rpc-client.ts
[json]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/json.md
[format]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/session-format.md
[messages]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/message-types.md
[cli]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/cli.md
[security]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/security.md
[mcp]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/mcp.md
[mcp-source]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/extensions/mcp/index.ts#L1132-L1140
[codemode]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/codemode.md
[settings]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/settings.md
[execute]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/extensions/codemode/execute.ts#L341-L479
[sandbox]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/codemode/src/runtime/host.ts
[nested]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/core/nested-tool-calls.ts
[pipeline]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/agent/src/agent-loop.ts#L790-L940
[sandbox-tests]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/codemode/test/sandbox.test.ts#L315-L332
[runtime-tests]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/test/agent-session-runtime-events.test.ts
[subagent]: https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/examples/extensions/subagent/index.ts
[main-sdk]: https://github.com/earendil-works/pi/blob/f10993bc7f28145df1375f3ff39c7f5c4cfc05f0/packages/coding-agent/src/core/sdk.ts
[main-code]: https://github.com/earendil-works/pi/blob/f10993bc7f28145df1375f3ff39c7f5c4cfc05f0/packages/coding-agent/src/extensions/codemode/execute.ts
