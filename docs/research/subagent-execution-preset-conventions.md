# Execution presets: configuration conventions and validation seams

Research for [Research model and effort preset configuration conventions](https://github.com/S2P2/pi-packages/issues/9). AFK evidence gathering only; **no format, schema, trust policy, or downstream decision is selected**.

## Answer in brief

Pi already supplies the useful execution primitives: exact provider/model lookup, credential-aware availability, model-specific supported thinking levels, JSON settings with user/project layering, and an exported YAML frontmatter parser. None of these requires a specialist persona. Its first-party subagent example is a **prompt-profile registry**, not a ready-made deterministic execution-preset registry. A package can reuse the primitives without copying that registry's required descriptions/system prompts, fuzzy model selection, or workflow machinery. [P1–P6]

The baseline's order is **explicit overrides → selected execution preset → configured default preset → parent settings**. That is a package contract, not Pi's native startup order. Pi's normal SDK/CLI paths can clamp effort or fall back to another model; strict deterministic execution therefore needs preflight and verification rather than simply forwarding a selector. [B, P2, P4, P5]

## Inspection boundary and pins

Inspected on **2026-10-07 UTC**:

- Pi `@earendil-works/pi-coding-agent` and `pi-ai` **1.0.4**, official tag `v1.0.4`, commit **`7c10bd4337495ee613f2224843ecdf349b80d1df`**. Source checked in a temporary clone; installed release declarations/examples also inspected. The bundled catalog is release-generated data, not checked into this tag. [P0]
- OpenCode official repository commit **`b1fe25ab5ecc9f9bc911a8a2e6a51565cc764322`**; docs and frontmatter implementation fetched at that commit. [O1–O3]
- Oh My Pi official repository commit **`ed8a0f73a12a8d1b0158c973c6345382bc63cb8c`**; task discovery documentation and named model-preset implementation. Its conventions are not Pi APIs. [M1–M2]
- DeepSeek Harness **`5badb15009ae1756c3afe0ae0cef1faafc290ccc`**, the same developer-preview pin as earlier local legwork; preset registry and subagent route-selection sources. [D1–D3]
- Claude Code and Cursor official live documentation fetched on the inspection date. These are **mutable documentation observations**, not commit-pinned implementation guarantees or claims about an installed product version. Claude's page includes explicit version qualifications; the model-order description below is the page's post-`2.1.251` order. [C, U]

Local survey and prior handoff/human-controls notes were context only, not API authorities. Root glossary defines Subagent and Delegation; no ADRs existed in this checkout. The baseline validation remains a human ticket. [B]

## 1. Pi primitives worth separating

### Settings and identity

User settings are `<agent-dir>/settings.json` (normally `~/.pi/agent`); project settings are **the child's cwd** `.pi/settings.json`. Normal project settings require trust. This is not the subagent example's nearest-ancestor `.pi/agents` search. Global defaults include `defaultProvider`, `defaultModel`, `defaultThinkingLevel`, and `modelThinkingLevels` keyed by exact `provider/modelId`. `models.json` defines endpoints/catalog overrides, not task execution presets. Credentials belong to normal auth/provider configuration, not a new preset asset. [P1, P2, P6]

`SettingsManager` uses JSON parsing, recursively merges nested settings objects, and exposes `getSettings()`, `getGlobalSettings()`, `getProjectSettings()`, `applyOverrides()`, and `inMemory()`. The `Settings` interface has no package-specific execution-preset schema. Unknown data surviving runtime parsing is not a supported typed extension schema or automatic validation. Also, generic recursive merge could accidentally combine a project preset's model with a user preset's effort: a human must choose **whole-entry replacement versus field merge** explicitly. [P2]

Resource-list discovery has additional combining behavior; do not infer preset collision rules from extension/skill/prompt lists. `SettingsManager.create` defaults its `projectTrusted` option to true: code constructing it directly must not mistake that default for a resolved human trust decision. [P1, P2]

A model's identity is the **provider plus full model ID**, not its display name or underlying vendor family. A model ID can itself contain slashes (e.g. OpenRouter names); a separate `{provider, modelId}` representation avoids ambiguous splitting. Pi CLI selectors support more permissive matching and thinking suffixes, and model restoration/startup has fallback paths. Exact `ModelRuntime.getModel(providerId, modelId)` and credential-aware `getAvailable()` are better seams for deterministic preflight than free-form selectors. [P4, P5, P6]

### Thinking / effort

Pi's vocabulary is `thinkingLevel`: `off`, `minimal`, `low`, `medium`, `high`, `xhigh`, `max`. `getSupportedThinkingLevels(model)` in `pi-ai` returns only `off` for non-reasoning models. For reasoning models, a `null` mapping disables a level; `xhigh` and `max` require an explicit non-undefined mapping. Thus `reasoning: true` alone does **not** imply every level is usable. This is catalog/adapter metadata, not a quality guarantee. [P3]

`clampThinkingLevel()` chooses an available level, looking upward first and then downward. The SDK factory and session thinking setters clamp automatically. An unsupported request can therefore become a **higher** effort, not just a cheaper downgrade. For example, if a model disables `off` but permits `minimal`, Pi clamps `off` to `minimal`. [P3, P4]

For a strict preset, a useful non-final validation sequence is:

1. Validate object shape, string fields, known keys, and the Pi level enum. Preserve explicit `off`; do not treat it as omission.
2. Resolve the final exact model after override layering; reject unknown/unavailable routes rather than substituting a provider default. Check credential configuration, while acknowledging it cannot prove remote authorization or quota.
3. Validate the final thinking level against that **resolved model** using `getSupportedThinkingLevels`. A known level and a supported level are different checks.
4. Pass the actual model object and resolved thinking level to `createAgentSession`; inspect the resulting `session.model` and `session.thinkingLevel` before prompting. Record requested/resolved values and configuration provenance.

This sequence is a research-derived integration sketch, **not an adopted error/clamping policy**. Human policy still needs to specify what to do with an inherited unsupported level. [P3–P5]

### Prompt profiles, names, and parser reuse

Pi prompt templates use Markdown/YAML frontmatter for description and argument hints; the filename becomes a slash-command name and the body expands into user-message text. A template's `model` metadata does not become an execution override through the documented template interface. Skills supply instructions, not another conversation context. [P7]

Pi exports `parseFrontmatter()`; it normalizes BOM/newlines and calls the `yaml` parser (release dependency `yaml` **2.9.0**). It returns a generic cast, not schema validation. YAML can yield booleans, lists, numbers, null, or a non-mapping root, and malformed YAML can throw. Consumers need runtime validation and per-file diagnostics. It is reusable parsing, not a format-independent registry or validator. [P8]

The first-party subagent example requires `name` and `description`, uses the Markdown body as `systemPrompt`, accepts tools as CSV or array, and a string `model`. User agents live under `<agent-dir>/agents`; project discovery searches upward for the nearest `.pi/agents`. With `both` scope project definitions replace user definitions by exact name. Discovery occurs again per invocation. It has no preset thinking-level field. When `model` is omitted, launch copies the dispatching session's active model **and thinking**; when it is supplied, the example does not pass the parent's thinking flag. This is a concrete inheritance difference, not the baseline's four-stage precedence. [P9]

OpenCode's inspected Markdown implementation instead uses `gray-matter`, with a sanitization retry for unquoted colon-containing scalars and a wrapper that reports frontmatter errors. This is evidence of a different parser tolerance, not a reason to assume cross-harness schema compatibility. [O3]

Names can therefore be optional for direct model/thinking overrides; only named reusable presets need a registry. A registry could expose a sorted list of names, scope, source path, description (optional), and validated effective route without injecting a persona or asking a model to infer a specialist. Duplicate-name, reload, and source-provenance semantics still need specification. [P9, M2; proposed adaptation]

### Trust is a separate axis

Normal Pi project trust protects settings/extensions/skills/prompts/themes, not arbitrary new `presets.json` files just because they are under `.pi`. A package's custom discovery must deliberately honor the resolved trust decision. Pi's first-party example adds user/project scope and interactive confirmation for project agents; that is example-specific behavior, not universal discovery. [P1, P10, P9]

Headless print/JSON/RPC cannot show Pi's built-in trust prompt: absent an override, extension, or saved decision, `ask`/`never` skips protected project resources. Child cwd/worktree canonical paths matter. Parent trust is not blanket authorization for every child cwd, and ordinary `AGENTS.md` context loads independently of trust. Trust and tool filtering are not sandboxes. Pure model/effort configuration is not executable code by itself, but it can change cost and provider/data destination; a trusted executable extension can change much more. [P10; implications]

## 2. First-party comparison

| Convention | What the inspected source actually provides | Fit for generic execution presets / mismatch |
|---|---|---|
| **Pi native JSON settings + SDK** | User/project defaults, exact registry lookup, supported levels; session factory accepts model/thinking independently of prompts. [P1–P5] | Closest runtime seam. A package still owns preset schema, discovery and stricter validation; core startup defaults are not the agreed four-stage order. |
| **Pi example Markdown + YAML frontmatter** | Named agents, required description, body as system prompt, scope/collision conventions, model string; parent model/thinking inheritance when model absent. [P9] | Reuse parser and scoped registry ideas, not required personas, chain/parallel workflow API, or free-form selectors. No existing execution-preset-only loader. |
| **Claude Code Markdown / CLI JSON agent definitions** | `name`/`description`, prompt body, model alias/full ID/`inherit`; project definitions outrank user definitions (also managed/CLI/plugin sources). Per-invocation model → definition → subagent-model environment default → parent; effort frontmatter overrides session effort, but not `CLAUDE_CODE_EFFORT_LEVEL`. Levels depend on model. [C] | Familiar discoverable profiles, but prompt-role centered and aliases are provider-specific. Its blocked-model substitution and environment precedence conflict with copying our baseline mechanically. |
| **OpenCode JSON/JSONC or Markdown** | Agent map keyed by name, or global/project `agents/*.md` whose filename supplies name; `mode: subagent`, provider/model string, prompt and permissions. Extra options such as `reasoningEffort` are provider-specific pass-through. Standard project configs override global conflicting keys; nonconflicting keys merge. [O1–O3] | Shows one normalized representation behind two file formats. Pass-through provider options are not portable Pi effort validation; large authority/prompt schema is unnecessary for a model/effort-only preset. |
| **Oh My Pi YAML config + Markdown task agents** | `config.yml` `modelRoles` decouples named routing from prompt profiles. Agent model selectors can be ordered lists; task agent settings overrides precede frontmatter, then parent/default fallback. Separately, `modelPresets` are named snapshots of role assignments and default thinking, with sorted discovery and whole-entry ownership by layer. Applying one persists roles and changes the live session. [M1–M2] | Strong evidence that named execution configuration need not equal persona. Reuse separation/provenance/whole-entry ideas; do not copy fallback chains, coarse effort hints, persistent root-setting mutations, or OMP APIs into Pi. |
| **DeepSeek Harness YAML plugin composition** | Declarative plugin rows configure subagent `agentOptions` (provider, model, adapter-owned effort) separately from optional `persona`. Preset registry composes plugins, not directory-scanned prompt agents. Live adapter preflight checks routes; provider/model must be supplied together; a changed route clears inherited route-owned effort unless supplied explicitly. [D1–D3] | Clear separation of execution route and persona; useful counterexample to blindly merging effort across models. Full Cordis plugin composition is much larger than a Pi preset file; its adapter effort IDs are not Pi's enum and YAML/plugins can execute host code. |
| **Cursor Markdown** | Optional name (filename fallback), description, `model: inherit` or model ID; project and Cursor-native location precedence. Square-bracket model parameters include effort/context. It can fall back under account/admin restrictions. [U] | Useful optional naming precedent, but coupled selector syntax and substitution are not deterministic Pi identity/effort rules. |

Execution presets answer **how to run** (route and thinking); specialist prompt profiles answer **what instructions to apply**. These can be composed independently, without requiring `agent: reviewer` to request a cheaper or stronger model. Neither naming format proves security or authorization. [P4, D2, M1; synthesis]

## 3. Decision-ready format alternatives

These are design comparisons, not upstream APIs or recommendations adopted by this research.

| Candidate | Reuse / advantage | Remaining cost or risk |
|---|---|---|
| **Dedicated JSON registry** (e.g. `<agent-dir>/subagent-presets.json`, trusted cwd `.pi/subagent-presets.json`) | Standard `JSON.parse`; easy strict schema and mapping-by-name; matches Pi's main config language; no prompt body to confuse with persona. | New file discovery/trust/reload layer; comments unavailable unless deliberately adding JSONC; schema validation still required. |
| **Package namespace in `settings.json`** | Can reuse location/trust and effective-settings inspection. | Core typed settings do not define it; generic nested merge can hybridize same-name presets. Need package-owned accessor/validation and an explicit merge rule, not assumption that core understands it. |
| **Dedicated YAML registry** | Comments/readable mappings; expressive scalar/list structures; resembles OMP/DeepSeek composition style. | Direct YAML parsing dependency/API and strict schema still needed; reject unintended tags/non-mappings and duplicate definitions. Do not import executable plugin features just to select a route. |
| **Markdown frontmatter-only preset files** | Reuse exported Pi frontmatter parser; convenient individual file discovery/description. | Must explicitly state body is absent/ignored/non-executable, rather than silently becoming system prompt. Filename versus frontmatter name, duplicates, and per-file parse errors need policy. Persona familiarity can be misleading. |

### Illustrative, non-final equivalent schemas

`example-provider` and `example-model-id` below are **placeholders**, not verified model identifiers. `schemaVersion`, filenames, field names, and descriptions are illustrative only.

JSON registry:

```json
{
  "schemaVersion": 1,
  "defaultPreset": "economy",
  "presets": {
    "economy": {
      "model": { "provider": "example-provider", "modelId": "example-model-id" },
      "thinkingLevel": "low"
    }
  }
}
```

Equivalent YAML:

```yaml
schemaVersion: 1
defaultPreset: economy
presets:
  economy:
    model:
      provider: example-provider
      modelId: example-model-id
    thinkingLevel: low
```

One frontmatter-only `economy.md` alternative:

```markdown
---
name: economy
model:
  provider: example-provider
  modelId: example-model-id
thinkingLevel: low
---
```

Direct invocation should also be possible without naming a preset, conceptually `{ task, model: { provider, modelId }, thinkingLevel }`. None of these schemas specifies a system prompt, tools, permissions, workflow, or automatic model candidates.

## 4. Preserve the agreed precedence without hiding unresolved edges

The baseline says explicit overrides, selected preset, configured default preset, then parent settings, with **no silent model substitution**. [B] Keep two operations distinct:

1. **Configuration resolution:** which user/project definitions and default preset name are eligible, trusted, and owning each named entry.
2. **Execution resolution:** apply the agreed override order to eligible data, resolve the final model/effort pair, validate it, and freeze it for this launch.

The source record does not settle whether selecting preset A overlays default preset B field-by-field or excludes B entirely. For example, A specifies only a model and B specifies `high` thinking: does A inherit `high` from B or current parent thinking? Both can be implemented while loosely describing the same order. A field-wise illustrative interpretation is `firstDefined(explicit.field, selected.field, default.field, parent.field)`; whole-preset selection is another. **Do not silently finalize this distinction.** Route identity should not become a hybrid of provider from one layer and model ID from another; this is a proposed invariant, supported by DeepSeek's paired-route validation, not an adopted Pi schema. [B, D3]

Similarly, changing only the model can make inherited thinking invalid. Choices include explicit rejection with supported levels, requiring complete presets, or a disclosed normalization rule. Pi's default clamp is evidence of existing behavior, not permission to adopt it. Named preset missing/malformed, credential absence, unknown model, and unsupported effort should be distinguishable in validation/reporting; a missing preset should not masquerade as successful parent inheritance. [P3–P5, M2; proposed diagnostics]

## 5. Identifier check and known unknowns

A bounded **offline catalog check**, not inference requests, inspected `pi-ai` 1.0.4's installed `dist/providers/data/{anthropic,openai}.json`. It contains `anthropic/claude-haiku-4-5`, `anthropic/claude-sonnet-4-5`, and `openai/gpt-5-mini`. The Anthropic entries mark reasoning true with no level map (Pi's function therefore supports through `high`, not `xhigh`/`max`); GPT-5 mini disables `off`, `xhigh`, and `max`. This establishes only membership/metadata in that release, **not current provider availability, account permission, pricing, or a valid production choice**. These entries are not used to bind the illustrative schemas. Release catalog JSON SHA-256: Anthropic `3ff00b68990382e42626ebd1b65dcc28ae3cad9e6faa8920331ba92f067f1b3a`; OpenAI `87faafcdd168c9b6fdaef521137834a207437daf0d71641fd0a2f3dddf4d1ecb`. [P0, P3]

Still unresolved for the human specification:

- Exact format/path/schema; names optional at invocation versus required in registry; body handling for Markdown.
- Whole-entry user/project replacement or nested merge; selected/default preset composition; allowed partial overrides and route-change thinking behavior.
- Unknown keys, duplicate names, project scope/ancestor traversal, symlink handling, reload timing, and provenance display.
- Trust behavior for custom config and headless children; child cwd versus invoking cwd for preset lookup (normal Pi resource discovery remains at child cwd).
- Strict rejection versus disclosed effort normalization, especially inherited levels and non-reasoning models.
- Whether deterministic presets permit **virtual models**. Pi virtual selection can route each request to another physical model/level; selection and dispatch are distinct, and unavailable virtual restoration can fall back to a physical model. If allowed, deterministic selection does not imply deterministic physical execution. [P11]
- Snapshot versus re-resolution for resumed/forked children: Pi SDK restoration can restore saved model/thinking when no explicit pair is supplied. Baseline override precedence must be consciously reconciled with native persistence, not assumed. [P4]

No runtime package was implemented and no authenticated provider requests were made. Source checking included pinned-source/doc inspection, release metadata/catalog inspection, and independently matching the catalog hashes against the npm release tarball. A local assertion script passed JSON/YAML equivalence and exported-Pi-frontmatter parsing of the illustrative schemas, and exercised the installed release's supported-level/clamping helpers against the three inspected catalog entries. This checks examples and cited behavior, not a production implementation. No downstream HITL ticket is resolved by this asset.

## Primary sources

- **[B]** [Validate the Pi subagent design baseline](https://github.com/S2P2/pi-packages/issues/2), initiating-conversation record (fetched with `gh`; validation remains human-owned).
- **[P0]** [Pi v1.0.4 source tag](https://github.com/earendil-works/pi/tree/7c10bd4337495ee613f2224843ecdf349b80d1df); [pi-ai 1.0.4 release](https://www.npmjs.com/package/@earendil-works/pi-ai/v/1.0.4) ([release tarball](https://registry.npmjs.org/@earendil-works/pi-ai/-/pi-ai-1.0.4.tgz), catalog paths/hash above).
- **[P1]** [Configuration](https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/configuration.md) and [Settings](https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/settings.md).
- **[P2]** [SettingsManager](https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/core/settings-manager.ts): `Settings`, `deepMergeObjects`, `create`, `loadFromStorage`, `getSettings`, `applyOverrides`.
- **[P3]** [Supported levels and clamping](https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/ai/src/models.ts#L1220-L1262).
- **[P4]** [SDK docs](https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/sdk.md), [factory model/restoration/thinking resolution](https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/core/sdk.ts), [session thinking setters](https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/core/agent-session.ts#L2580-L2671).
- **[P5]** [ModelRuntime exact lookup and availability](https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/core/model-runtime.ts#L445-L502), [selector/startup/restore logic](https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/core/model-resolver.ts).
- **[P6]** [Models, authentication, compatible endpoints](https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/models.md).
- **[P7]** [Prompt templates](https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/prompt-templates.md), [Skills](https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/skills.md).
- **[P8]** [Frontmatter parser](https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/utils/frontmatter.ts), [export](https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/src/index.ts#L489), [dependency](https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/package.json).
- **[P9]** [First-party subagent example](https://github.com/earendil-works/pi/tree/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/examples/extensions/subagent): `agents.ts`, `index.ts`, `README.md`.
- **[P10]** [Pi security/project trust](https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/security.md).
- **[P11]** [Virtual models](https://github.com/earendil-works/pi/blob/7c10bd4337495ee613f2224843ecdf349b80d1df/packages/coding-agent/docs/virtual-models.md).
- **[C]** [Claude Code subagent definitions, scopes, frontmatter and model order](https://code.claude.com/docs/en/sub-agents#choose-a-model), live docs, accessed 2026-10-07.
- **[O1]** [OpenCode agent formats/options](https://github.com/anomalyco/opencode/blob/b1fe25ab5ecc9f9bc911a8a2e6a51565cc764322/packages/web/src/content/docs/agents.mdx).
- **[O2]** [OpenCode config merge/order](https://github.com/anomalyco/opencode/blob/b1fe25ab5ecc9f9bc911a8a2e6a51565cc764322/packages/web/src/content/docs/config.mdx).
- **[O3]** [OpenCode core Markdown parser](https://github.com/anomalyco/opencode/blob/b1fe25ab5ecc9f9bc911a8a2e6a51565cc764322/packages/core/src/config/markdown.ts), [consumer/error wrapping](https://github.com/anomalyco/opencode/blob/b1fe25ab5ecc9f9bc911a8a2e6a51565cc764322/packages/opencode/src/config/markdown.ts).
- **[M1]** [Oh My Pi task agent discovery/config/routing](https://github.com/can1357/oh-my-pi/blob/ed8a0f73a12a8d1b0158c973c6345382bc63cb8c/docs/task-agent-discovery.md).
- **[M2]** [Oh My Pi named model presets](https://github.com/can1357/oh-my-pi/blob/ed8a0f73a12a8d1b0158c973c6345382bc63cb8c/packages/coding-agent/src/config/model-presets.ts).
- **[D1]** [DeepSeek preset registry](https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/preset/agent-preset-registry/README.md).
- **[D2]** [DeepSeek subagent configuration and model selection](https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/tool-subagent/README.md).
- **[D3]** [DeepSeek paired-route merge/adapter validation](https://github.com/deepseek-ai/deepseek-harness/blob/5badb15009ae1756c3afe0ae0cef1faafc290ccc/packages/subagent/tool-subagent/src/model-selection.ts).
- **[U]** [Cursor subagent frontmatter, model parameters and substitution](https://cursor.com/docs/subagents), live docs, accessed 2026-10-07.
