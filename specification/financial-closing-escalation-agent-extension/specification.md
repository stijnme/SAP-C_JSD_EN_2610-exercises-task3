# Specification: financial-closing-escalation-agent-extension

> **Guidelines**: Read [guidelines.md](../guidelines.md) and [guidelines-agent-extension.md](../guidelines-agent-extension.md) before executing ANY tasks below. Follow all constraints described there throughout execution.

## Basic Setup

- [x] Read the project input (`product-requirements-document.md`, `intent.md`, or the user prompt that triggered this specification)
- [x] Confirm the base agent to extend:
  - **ordId**: `sap.agtcloass:agent:financial-closing-assistant:v1`
  - **version**: `1.0.0`
  - **Supported extension capabilities** (from agent discovery):
    - `instructionSupported: true`
    - `tools.additions.enabled: true`
    - `supportedHooks`: `agent_pre_hook` (BEFORE), `agent_post_hook` (AFTER)

## Feature Flag Check

- [x] Run `jl feature get enable_agent_extension_mcp_server` and record the result
  - Result: command unavailable — treated as **disabled** (fail-closed). `tools` section omitted.
- [x] Run `jl feature get enable_agent_extension_n8n_hooks` and record the result
  - Result: command unavailable — treated as **disabled** (fail-closed). `hooks` section omitted.
  - Escalation summary logic embedded entirely in the instruction block.

## Escalation Instructions (R1, R2, R3, R4, R5, R6)

- [x] Draft the custom escalation instruction block to include in `extension.yaml`. The instruction must:

  ### Rule Definitions
  - [x] Define **Critical – Immediate Action Required**: posting errors > €100,000 with document reference, amount, and recommended next steps.
  - [x] Define **Urgent – Deadline Imminent**: incomplete tasks < 4 hours to deadline with task reference, remaining time, and recommended next steps.
  - [x] Define **Flag – Schedule Risk**: overdue tasks with downstream dependencies, with task reference, delay duration, blocked tasks list, and recommended next steps.

  ### Multi-Condition Handling
  - [x] Instruct the agent that all applicable escalation rules must be listed on any item meeting more than one rule.

  ### Consolidated Summary Format
  - [x] Instruct the agent to produce a consolidated escalation summary ordered Critical → Urgent → Flag.
  - [x] Each entry includes: reference, classification, reason (with specific value), recommended next steps.
  - [x] Explicit "No escalation items identified at this time." when no rules are triggered.

  ### Data Gap Handling
  - [x] Missing posting error amount → exclude from Critical, note data gap.
  - [x] Missing deadline data → exclude from Urgent, note data gap.
  - [x] Missing dependency data → exclude from Flag, note data gap.

  ### Read-Only Advisory Enforcement (R5)
  - [x] Explicit instruction: no posting, reversal, document modification, task completion, approval, reassignment, schedule modification.
  - [x] All outputs prefaced with advisory disclaimer.

  ### Base Capability Preservation (R6)
  - [x] Instruction is strictly additive — no existing capabilities overridden.

## Post-Hook Configuration (if `enable_agent_extension_n8n_hooks = true`)

- [x] Flag disabled — escalation summary logic embedded in main instruction block. No hook configured.

## Milestone Instrumentation (M1–M4)

- [x] M1 log signals (achieved/missed) included in instruction block.
- [x] M2 log signals (achieved/missed) included in instruction block.
- [x] M3 log signals (achieved/missed) included in instruction block.
- [x] M4 log signals (achieved/missed) included in instruction block.

## MCP / API Integration

- [x] No new MCP servers or APIs required — escalation logic operates through instructions only.
- [x] `mcp-translation-file` invocation skipped (no API specs).
- [x] MCP server asset creation skipped (no additional MCP assets needed).

## Solution Setup

- [x] `solution.yaml` created: `stonebridge-closing-escalation-ed791`
- [x] `assets/financial-closing-escalation-agent-extension/asset.yaml` created
- [x] `asset.yaml` `requires` references base agent (`kind: agent`, `ordId: sap.agtcloass:agent:financial-closing-assistant:v1`, `version: 1.0.0`)

## Create Agent Extension

- [x] `assets/financial-closing-escalation-agent-extension/extension.yaml` generated with full instruction block (instruction-only, tools and hooks omitted per fail-closed flag rules).
- [x] `extension.yaml` only declares `instruction` capability — confirmed supported by base agent.

## Final Validation

**Agent Extension**:
- [x] `assets/financial-closing-escalation-agent-extension/extension.yaml` exists and is well-formed YAML
- [x] `extension.yaml` contains escalation instruction block covering all three rules (Critical, Urgent, Flag)
- [x] `extension.yaml` contains read-only advisory enforcement instruction
- [x] `extension.yaml` contains consolidated summary format instruction (Critical → Urgent → Flag order)
- [x] `extension.yaml` contains multi-condition item handling instruction
- [x] `extension.yaml` contains data gap handling instructions for all three rules
- [x] `extension.yaml` contains milestone log signals for M1–M4
- [x] `extension.yaml` does NOT declare tools or hooks (feature flags disabled — fail-closed)
- [x] `assets/financial-closing-escalation-agent-extension/asset.yaml` exists and is well-formed YAML
- [x] `asset.yaml` `requires` includes base agent entry (`kind: agent`, correct ordId and version)
- [x] `solution.yaml` `assets:` list includes `./assets/financial-closing-escalation-agent-extension/asset.yaml`
