---
name: problem-triage
description: >
  General problem diagnosis and failure triage methodology. Use when investigating failures, flaky behavior, unexpected app or test states, blocked automation, environment issues, dependency failures, regression analysis, root-cause classification, or when producing a structured diagnosis and next action.
user-invocable: false
---

# Problem Triage

Problem triage is a general investigation workflow for unexpected failures, blocked execution, flaky behavior, and mismatches between an intended objective and the observed state.

Use this skill to move from observation to a defensible next action without guessing beyond the available evidence.

## Goal

When something fails or behaves unexpectedly, determine the most likely cause class and the most appropriate next action.

The outcome should answer:

- what objective or expectation failed;
- what was actually observed;
- whether the observation is relevant to the objective;
- which owner or layer is most likely responsible;
- what should happen next.

## Boundaries

- Do not use triage to replace checks, assertions, or validations that can be expressed directly.
- Do not assume product behavior is wrong until the expected behavior is anchored in requirements, existing tests, product knowledge, or comparable flows.
- Do not mechanically clear dialogs, retry, refresh, or reset state before identifying whether the blocker is the cause or only a symptom.
- Do not record project-specific conclusions in the skill. Put stable reusable facts in the project knowledge base instead.
- Do not write speculative conclusions back to the knowledge base.

## Use Knowledge During Triage

Before deciding on root cause, consult available project knowledge for reusable facts:

1. Read relevant flow, screen, feature, or environment preconditions.
2. Check known quirks, recovery paths, and dependency notes.
3. For system dialogs, third-party screens, backend services, SDKs, or cross-app flows, check dependency knowledge when available.

Use knowledge for stable facts such as:

- login, permission, account, seed data, or first-run prerequisites;
- environment assumptions such as device model, OS version, locale, network, build flavor, or feature flags;
- known flaky states, loading behavior, retry paths, and timing constraints;
- known dependency app screens, service outages, selectors, or recovery paths;
- confirmed differences across versions, ROMs, regions, or account types.

If the current observation is new, confirmed, stable, and likely reusable, write it back to the knowledge base after the investigation.

## Standard Triage Loop

1. Confirm the failing objective.
   - What was the task, test, command, or user flow trying to prove?
   - Which assertion, step, expectation, or output failed?
   - What evidence defines the expected behavior?

2. Capture the observed state.
   - What is visible now, and what state is the system actually in?
   - Which app, process, page, screen, command, service, or dependency owns the current state?
   - Is there a dialog, error, loading state, crash, timeout, wrong screen, stale data, missing data, or unexpected branch?

3. Compare expected and actual behavior.
   - Is the observed state directly blocking the objective or merely adjacent noise?
   - Did the system fail to transition, or did the check run too early or against the wrong target?
   - Is this a reproducible state, a transient condition, or insufficiently observed?

4. Check reusable knowledge.
   - Is this state already documented as a known precondition, quirk, branch, dependency behavior, or recovery path?
   - Does the knowledge change the likely owner or next action?

5. Classify the likely cause.
   - `precondition_missing`: required state such as login, permission, data, configuration, feature flag, setup, or first-run completion was absent.
   - `environment_interference`: device, OS, network, locale, clock, resource pressure, overlay, permissions manager, CI host, or other environment state blocked the flow.
   - `test_or_automation_gap`: selectors, waits, setup, assertions, fixture state, scripted branches, mocks, or automation assumptions no longer match reality.
   - `product_issue`: the product behavior itself appears wrong for the intended requirement or stable expected flow.
   - `external_dependency`: backend, third-party app, SDK, identity provider, WebView, map, payment, chooser, or other external dependency failed or changed behavior.
   - `data_issue`: input data, account state, fixtures, caches, migrations, or backend records are missing, stale, corrupted, or inconsistent.
   - `unknown`: evidence is insufficient, contradictory, or points to multiple plausible causes.

6. Decide the next action.
   - Continue or resume if the pause only required documentation and the run can proceed soundly.
   - Repair setup or fixtures if a reusable precondition is missing.
   - Update automation if the script no longer models the actual flow.
   - Report or escalate a product issue if product behavior contradicts an anchored expectation.
   - Mark blocked if an external dependency or missing evidence prevents a sound conclusion.
   - Gather one more targeted piece of evidence if it can distinguish between two likely causes cheaply.

7. Produce a structured diagnosis.
   - Summarize the observed blocker, impact, classification, likely owner, evidence, and next action.

## High-Frequency Scenarios

### Dialog, Permission, Or Interstitial

Ask:

1. Is the prompt expected for this entry path and environment?
2. Should it be prepared by setup, handled by the workflow, or treated as unexpected product behavior?
3. Is the prompt the direct blocker or a symptom of an earlier missed state transition?

Typical outcomes:

- Expected first-run or setup prompt not prepared -> `precondition_missing` or `environment_interference`
- Expected in-flow branch not covered by automation -> `test_or_automation_gap`
- Unexpected timing or prompt type -> `product_issue` or `environment_interference`

### Login, Account, Or Session State

Ask:

1. Did the flow assume authenticated, authorized, or entitled state?
2. Is expiry, logout, account restriction, or region gating expected in this environment?
3. Should the state be restored by shared setup instead of per-flow logic?

### Loading, Timeout, Or Wrong Destination

Ask:

1. Did the system fail to navigate, or did validation run before the postcondition was observable?
2. Is the current state a known loading, retry, empty, permission, or error branch?
3. Is there evidence of backend latency, local resource pressure, stale cache, or dependency failure?

### Dependency Or External Screen

Ask:

1. Which package, process, service, or provider owns the current state?
2. Does project knowledge already describe this dependency behavior?
3. Is there a known recovery path, or is this a new dependency behavior worth recording?

### Data Or Fixture Mismatch

Ask:

1. What exact data did the objective require?
2. Was that data created, seeded, synchronized, migrated, or selected correctly?
3. Is the observed issue caused by missing data, stale data, duplicate data, or an assumption about ordering or defaults?

## Structured Diagnosis Template

Use a concise structured reason in this shape:

`observed=<what failed or blocked the objective>; impact=<how it affected the objective>; classification=<one class>; likely_owner=<test|automation|environment|product|external|data|unknown>; evidence=<key evidence used>; next_action=<recommended follow-up>`

Example:

`observed=login screen appeared before the checkout confirmation step; impact=confirmation assertion could not run against the intended page; classification=precondition_missing; likely_owner=automation; evidence=session-dependent flow reached auth gate before target screen; next_action=restore authenticated session in shared setup or explicitly model the login branch`

## Knowledge Write-Back Rule

Write back to the knowledge base only when all of the following are true:

1. The observation is confirmed, not guessed.
2. The observation is likely reusable in future investigations.
3. The observation has meaningful scope such as app version, build flavor, OS version, device type, account state, environment, dependency version, or entry path.

Do not write back one-off noise, unresolved suspicions, or conclusions without enough supporting evidence.
