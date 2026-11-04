# Stonebridge Financial Closing Escalation Extension

Extend the SAP Financial Closing Assistant with company-specific escalation rules for Stonebridge Equipment Group to dramatically reduce the time needed to identify and act on critical period-end closing issues.

## Business challenge

Stonebridge Equipment Group, an industrial equipment manufacturer operating across multiple legal entities, manages a complex period-end closing process with hundreds of interdependent tasks, reconciliations, and document postings. The finance team currently takes up to two hours to identify and escalate critical closing issues. The goal is to reduce that time to 15 minutes by embedding company-specific escalation rules directly into the SAP Financial Closing Assistant.

## Business Goals & Success Criteria

| Metric | Baseline | Target | Timeline | Process / Capability | Source |
|--------|----------|--------|----------|----------------------|--------|
| Time to identify & escalate critical closing issues | 2 hours | 15 minutes | Per period-end close cycle | Period-End Financial Close – Escalation Management | user |

## Key Milestones

| Milestone | Condition |
|-----------|-----------|
| Escalation rules active | Extension deployed; assistant applies all three escalation rules (Critical, Urgent, Flag) |
| Prioritized summary delivered | Assistant produces a consolidated escalation summary ordered Critical → Urgent → Flag within a single interaction |
| Read-only compliance verified | Extension confirmed to perform no postings, completions, approvals, or schedule modifications |

## Business Architecture (RBA)

### End-to-End Process

Finance – Record to Report (generic)

### Process Hierarchy

```
Finance (E2E)
└── Record to Report (generic)
    └── Perform accounting and financial close (BPS-413)
        └── Perform financial closing
```

### Summary

The business challenge maps to the "Perform accounting and financial close" sub-process (BPS-413) within the Record to Report E2E. The manufacturing industry variant applies given Stonebridge's multi-entity industrial equipment context.

## Fit Gap Analysis

| Requirement (business) | Standard asset(s) found | API ORD ID | MCP Server ORD ID | MCP Server Version | Webhook API ORD ID | Data Product ORD ID | Gap? | Notes / assumptions |
|------------------------|------------------------|------------|-------------------|--------------------|--------------------|---------------------|------|---------------------|
| Identify high-value posting errors (>€100k) and classify as Critical | SAP Advanced Financial Closing – Local/Group Financial Closing | — | — | — | — | — | Yes | No standard rule for company-specific amount threshold; custom escalation instruction required |
| Detect incomplete tasks with <4h to period-end deadline (Urgent) | SAP Advanced Financial Closing – Local Financial Closing | — | — | — | — | — | Yes | Deadline proximity alerting at custom threshold not available out-of-the-box |
| Flag overdue tasks blocking downstream dependencies (Flag – Schedule Risk) | SAP Advanced Financial Closing – Local/Group Financial Closing | — | — | — | — | — | Yes | Dependency-aware overdue escalation requires custom logic |
| Consolidated escalation summary (Critical → Urgent → Flag) with recommended next steps | SAP Financial Closing Assistant (sap.agtcloass:agent:financial-closing-assistant:v1) | — | — | — | — | — | Yes | Base agent orchestrates financial close but lacks Stonebridge-specific prioritization rules |
| Read-only advisory mode (no postings, completions, or approvals) | SAP Financial Closing Assistant | — | — | — | — | — | No | Base agent supports read-only interaction; constraint enforced through instructions |

### Key findings

- The Financial Closing Assistant (`sap.agtcloass:agent:financial-closing-assistant:v1`) is available and supports custom instructions, tool additions, and pre/post hooks — ideal for layering escalation rules.
- All five discovered SAP Advanced Financial Closing APIs are marked unavailable in this tenant, so the extension relies entirely on the agent's built-in capabilities and custom instructions.
- The three escalation rules (Critical / Urgent / Flag) are company-specific thresholds not covered by standard SAP product capabilities and must be implemented as agent extension instructions.
- SAP Advanced Financial Closing (mandatory capabilities: Local Financial Closing, Group Financial Closing, General Ledger Accounting) provides the underlying closing process management that the base agent orchestrates.
- The post-hook capability of the base agent can be used to format and prioritize the escalation summary output after the main orchestration logic runs.

## Recommendations

### Agent Extension for Financial Closing Escalation

#### Executive Summary

Extend Financial Closing Assistant with Stonebridge-specific escalation rules

#### Recommended Solution

Implement an agent extension on top of the existing SAP Financial Closing Assistant (`sap.agtcloass:agent:financial-closing-assistant:v1`). The extension adds:

1. **Custom escalation instructions** defining the three classification rules:
   - Critical – Immediate Action Required: closing documents with posting errors exceeding €100,000
   - Urgent – Deadline Imminent: incomplete tasks with fewer than 4 hours before the period-end deadline
   - Flag – Schedule Risk: overdue tasks blocking downstream dependencies
2. **Post-hook** to generate a consolidated, prioritized escalation summary (Critical → Urgent → Flag) after the base agent's orchestration completes, including task/document references, escalation reasons, and recommended next steps.
3. **Read-only constraint** enforced through instructions ensuring no documents are posted, tasks completed, entries approved, or schedules modified.

#### Problem Statement

The finance team at Stonebridge Equipment Group spends up to two hours per period-end cycle manually identifying and escalating critical closing issues. This delay risks financial reporting deadlines and reduces the team's ability to respond in time.

#### Affected User Roles

- Finance Controllers
- Period-End Close Managers
- Accounting Team Leads (across multiple legal entities)

#### Important factors

##### Preserves existing capabilities

The extension is purely additive — it does not replace or override the Financial Closing Assistant's existing orchestration of journal entries, accruals, clearing, and analytical insights.

##### Strict read-only advisory mode

All three escalation rules produce recommendations only. The extension must not trigger any transactional actions, ensuring compliance with audit and control requirements.

##### Multi-item escalation

When multiple rules apply to the same closing item, all applicable conditions must be identified and reported together, avoiding incomplete escalation signals.

#### Potential risks

##### Dependency on base agent's data access

The escalation logic relies on the base Financial Closing Assistant's ability to retrieve closing task statuses, deadlines, and posting amounts. If the underlying data sources are incomplete or unavailable in a given period, escalation coverage will be partial.

##### Threshold calibration

The €100,000 posting error threshold and 4-hour deadline proximity window are fixed in the initial extension. Future periods may require recalibration — a process not yet defined for Stonebridge.

#### Recommended solution category

Agent Extension

#### Intent fit
95%
