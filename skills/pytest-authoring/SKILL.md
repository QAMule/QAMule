---
name: pytest-authoring
description: >
  Define how pytest-based QAMule UI tests should be authored, including test boundaries, layering, file organization, markers, fixtures, waits, parametrization, assertions, and flaky UI test stabilization. Use when writing or modifying pytest UI tests, fixtures, helpers, conftest files, markers, or parametrized cases.
user-invocable: false
---

# Pytest Authoring

Authoring conventions for pytest-based QAMule UI automation tests.

## Test Boundaries

- Keep each test focused on one user-visible behavior or tightly scoped failure mode.
- Name tests by observable behavior, not internal implementation.
- Keep prerequisite UI steps only when needed to reach the target state.
- Split the test when a later step adds an independent success criterion or failure mode.

## Layering And File Shape

- Separate entry, scenario, and mechanics.
- Entry code owns collection, markers, and light wiring only.
- Scenario code owns user-facing behavior, assertions, and deliberate validation steps.
- Mechanics code owns reusable device actions, waits, parsing, and low-level helpers.
- Do not let one test, scenario, fixture, or helper file grow indefinitely.
- Reorganize large files by behavior, feature, fixture ownership, or helper domain when they become hard to scan.

## Markers And Fixtures

- Use markers to express suite intent, feature area, environment, dependency, or execution cost.
- Reuse existing marker vocabulary when possible.
- Use fixtures for setup, not to hide the behavior under test.
- Default fixtures to the narrowest useful scope.
- Keep UI stateful setup at function scope unless wider sharing is clearly safe.
- Do not put core business assertions inside fixtures unless they validate setup prerequisites.

## Stability And Assertions

- Prefer condition-based waits with explicit bounds over fixed sleeps.
- After state-changing UI actions, wait for an observable postcondition.
- Do not add blind retries in test bodies.
- Use parametrization for the same behavior under varied inputs, states, or device shapes.
- Do not merge unrelated behaviors into one parametrized test.
- Assert user-visible outcomes or externally observable state transitions.

## Review Checklist

- The test name states the behavior being verified.
- Prerequisite steps exist only to reach the target state.
- Entry, scenario, and mechanics responsibilities are separated cleanly.
- Large files are split or reorganized when they obscure behavior boundaries or ownership.
- Markers make suite selection and execution cost clear.
- Fixture scope is no broader than necessary.
- Waits are condition-based with explicit bounds.
- Parametrization varies data or state without merging unrelated scenarios.
- Assertions verify observable outcomes, not just that an interaction was performed.
