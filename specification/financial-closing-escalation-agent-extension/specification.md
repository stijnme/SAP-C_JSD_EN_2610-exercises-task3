# Specification: financial-closing-escalation-agent-extension

> **Guidelines**: Read [guidelines.md](../guidelines.md) and [guidelines-agent-extension.md](../guidelines-agent-extension.md) before executing ANY tasks below. Follow all constraints described there throughout execution.

## Basic Setup

- [ ] Read the project input (`product-requirements-document.md`, `intent.md`, or the user prompt that triggered this specification)
- [ ] Confirm the base agent to extend:
  - **ordId**: `sap.agtcloass:agent:financial-closing-assistant:v1`
  - **version**: `1.0.0`
  - **Supported extension capabilities** (from agent discovery):
    - `instructionSupported: true`
    - `tools.additions.enabled: true`
    - `supportedHooks`: `agent_pre_hook` (BEFORE), `agent_post_hook` (AFTER)

## Feature Flag Check

- [ ] Run `jl feature get enable_agent_extension_mcp_server` and record the result
  - If `true` → MCP tool additions are allowed; proceed with tools section in `extension.yaml`
  - If `false` or error → omit `tools` entirely from `extension.yaml` and skip all MCP-related steps
- [ ] Run `jl feature get enable_agent_extension_n8n_hooks` and record the result
  - If `true` → n8n hooks are allowed; proceed with hooks section in `extension.yaml`
  - If `false` or error → omit `hooks` entirely from `extension.yaml`
  - **Note**: This extension does not require an n8n workflow — the post-hook escalation summary is implemented purely through instructions. If hooks flag is false, the escalation summary logic is embedded entirely in the instruction block.

## Escalation Instructions (R1, R2, R3, R4, R5, R6)

- [ ] Draft the custom escalation instruction block to include in `extension.yaml`. The instruction must:

  ### Rule Definitions
  - [ ] Define **Critical – Immediate Action Required**: identify any closing document that has a posting error where the error amount exceeds €100,000. For each such document, record: document reference, error amount, and recommended next step (e.g. contact the responsible accountant immediately and verify the posting before the close deadline).
  - [ ] Define **Urgent – Deadline Imminent**: identify any incomplete closing task where fewer than 4 hours remain before the official period-end deadline. For each such task, record: task reference, remaining time to deadline, and recommended next step (e.g. escalate to the task owner and Period-End Close Manager immediately).
  - [ ] Define **Flag – Schedule Risk**: identify any closing task that has passed its scheduled completion time AND has downstream dependencies. For each such task, record: task reference, delay duration, list of blocked downstream tasks, and recommended next step (e.g. contact the task owner to unblock the dependent tasks and update the schedule).

  ### Multi-Condition Handling
  - [ ] Instruct the agent that when a single closing item meets more than one escalation rule, all applicable conditions must be listed on that item in the summary — do not deduplicate or suppress any rule that applies.

  ### Consolidated Summary Format
  - [ ] Instruct the agent to produce a consolidated escalation summary after evaluating all closing tasks and documents, structured in the following order:
    1. **Critical – Immediate Action Required** (posting errors > €100,000)
    2. **Urgent – Deadline Imminent** (incomplete tasks < 4 hours to deadline)
    3. **Flag – Schedule Risk** (overdue tasks blocking downstream dependencies)
  - [ ] Each entry in the summary must include:
    - Task or document reference
    - Escalation classification (Critical / Urgent / Flag)
    - Escalation reason (the specific condition that was met, including the value, e.g. "Posting error amount: €245,000")
    - Recommended next steps
  - [ ] If no items meet any escalation rule, the summary must explicitly state: "No escalation items identified at this time."

  ### Data Gap Handling
  - [ ] If posting error amount data is unavailable for a document → exclude from Critical classification and note the data gap in the summary.
  - [ ] If deadline data is unavailable for a task → exclude from Urgent classification and note the data gap.
  - [ ] If dependency data is unavailable for a task → exclude from Flag classification and note the data gap.

  ### Read-Only Advisory Enforcement (R5)
  - [ ] Include an explicit instruction that the extension operates in read-only advisory mode at all times:
    - It must never post, reverse, or modify any financial document.
    - It must never complete, approve, reassign, or close any closing task.
    - It must never modify the closing schedule or period-end deadline.
    - All outputs must be prefaced with an advisory disclaimer (e.g. "The following is an advisory escalation summary. No actions have been taken.").

  ### Base Capability Preservation (R6)
  - [ ] Confirm the instruction does not override or disable any existing Financial Closing Assistant capability (journal entries, accruals, clearing, insights). The escalation logic is strictly additive.

## Post-Hook Configuration (if `enable_agent_extension_n8n_hooks = true`)

- [ ] If the n8n hooks flag is enabled, configure `agent_post_hook` in `extension.yaml` to trigger the escalation summary output after the base agent's main orchestration logic completes.
- [ ] If the n8n hooks flag is disabled, ensure the escalation summary instruction is self-contained and executes within the main instruction block without requiring a hook.

## Milestone Instrumentation (M1–M4)

- [ ] Ensure the instruction block includes log/trace signals for each milestone:
  - **M1**: `M1.achieved: all three escalation rules (Critical, Urgent, Flag) evaluated and applied successfully` — emitted when all three rules have been evaluated in an interaction.
  - **M1 miss**: `M1.missed: one or more escalation rules failed to evaluate — check extension instructions and post-hook configuration`
  - **M2**: `M2.achieved: consolidated escalation summary produced — Critical: {n}, Urgent: {n}, Flag: {n}` — emitted when the prioritized summary is produced.
  - **M2 miss**: `M2.missed: escalation summary not produced or order incorrect — check post-hook execution`
  - **M3**: `M3.achieved: multi-condition escalation correctly applied — {n} items with multiple classifications identified`
  - **M3 miss**: `M3.missed: multi-condition item handling not verified — review rule evaluation logic in instructions`
  - **M4**: `M4.achieved: read-only compliance confirmed — no transactional actions triggered`
  - **M4 miss**: `M4.missed: transactional action detected during escalation interaction — review instruction guardrails immediately`

## MCP / API Integration

- [ ] No new MCP servers or APIs are required for this extension — escalation logic operates through instructions applied to data the base Financial Closing Assistant retrieves via its own tools.
- [ ] Skip `mcp-translation-file` invocation (no API specs to process).
- [ ] Skip MCP server asset creation (no additional MCP assets needed).

## Solution Setup

- [ ] Invoke `setup-solution` skill to create:
  - `solution.yaml` (if not already present)
  - `assets/financial-closing-escalation-agent-extension/asset.yaml`
- [ ] Confirm `asset.yaml` `requires` references the base agent:
  - `kind: agent`
  - `ordId: sap.agtcloass:agent:financial-closing-assistant:v1`
  - `version: 1.0.0`

## Create Agent Extension

- [ ] Invoke the `create-agent-extension` skill from `assets/financial-closing-escalation-agent-extension/` to generate `extension.yaml` and populate:
  - `capabilityImplementations[].instruction`: the full escalation instruction block drafted above
  - `capabilityImplementations[].hooks`: post-hook configuration (only if `enable_agent_extension_n8n_hooks = true`)
  - `capabilityImplementations[].tools`: omit or populate based on `enable_agent_extension_mcp_server` flag result
- [ ] Validate `extension.yaml` only declares capabilities confirmed as supported by the base agent:
  - Instructions: always allowed (`instructionSupported: true`)
  - Tools: only if flag = `true`
  - Hooks: only if flag = `true`

## Final Validation

Confirm all files are present and correctly cross-referenced before marking this spec complete:

**Agent Extension**:
- [ ] `assets/financial-closing-escalation-agent-extension/extension.yaml` exists and is well-formed YAML
- [ ] `extension.yaml` contains the escalation instruction block covering all three rules (Critical, Urgent, Flag)
- [ ] `extension.yaml` contains the read-only advisory enforcement instruction
- [ ] `extension.yaml` contains the consolidated summary format instruction (Critical → Urgent → Flag order)
- [ ] `extension.yaml` contains multi-condition item handling instruction
- [ ] `extension.yaml` contains data gap handling instructions for all three rules
- [ ] `extension.yaml` contains milestone log signals for M1–M4
- [ ] `extension.yaml` does NOT declare any capabilities the base agent does not support
- [ ] `assets/financial-closing-escalation-agent-extension/asset.yaml` exists and is well-formed YAML
- [ ] `asset.yaml` `requires` includes the base agent entry (`kind: agent`, correct ordId and version)
- [ ] `solution.yaml` `assets:` list includes `./assets/financial-closing-escalation-agent-extension/asset.yaml`
