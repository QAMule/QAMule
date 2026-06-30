---
name: pytest-qamule
description: >
  Use pytest-qamule to run QAMule tests in a pytest environment. Use when running QAMule tests in a pytest environment, or when writing new tests for pytest-qamule.
user-invocable: false
---

# Pytest QAMule

This skill helps you run QAMule tests in a pytest environment based on the `pytest-qamule` plugin.

## Basics

- Always run commands as `uv run <command> [args]`. Examples below show only `<command> [args]`; prepend `uv run` when running them.
- Use `pytest --collect-only` to inspect tests without executing them.
- Add `--pause-on-failure` only when failure inspection is requested.

## Devices

- Default mode (single device): use fixture `d`, no `--device` argument
- Named device mode: pass `--device NAME:SERIAL` and use matching fixture `{NAME}` in test function
- Multi-device mode: repeat `--device NAME:SERIAL` and reference each `{NAME}` as a fixture

```bash
pytest tests --device phone:emulator-5554 --device tablet:127.0.0.1:9887
```

## Checkpoints

The `checkpoint` fixture is auto-loaded and pauses immediately when called:

```python
def test_checkpoint(checkpoint):
    response = checkpoint("verify app state")
    assert response.result is True
    assert response.message == "checkpoint accepted"
```

- Attach evidence with `checkpoint("verify app state", images=["image.jpg"])`.
- Do not use checkpoints for checks that selectors, text assertions, polling, or normal failure inspection can cover.
- Use `--qamule-checkpoint-mock-result true|false|none --qamule-checkpoint-mock-message "..."` only for local smoke checks, not normal execution.

## Run Workflow

For every executing pytest run, `qamule pause watch` is mandatory; do not rely only on pytest terminal output.

1. Start pytest in async mode:

```bash
pytest [path/options]
```

2. Read the printed session ID, for example `QAMule Session ID: s-...`.

3. Only if the user explicitly asks to view the live report, start report serving now, before watching:

```bash
qamule report serve --session-id <session-id>
```

Run report serving in async mode. Otherwise, skip this step and go directly to `qamule pause watch`.

4. Watch the run in sync mode with no timeout:

```bash
qamule pause watch --session-id <session-id>
```

`watch` stops on finish, checkpoint pause, or failure pause.

5. Handle the watch result:

- `checkpoint`: complete the requested task, then resume with a result:
  `qamule pause resume <pause_id> --session-id <session-id> --result true --message "checkpoint accepted" --image image.jpg`
- `failure`: inspect the failure state, then resume without a result:
  `qamule pause resume <pause_id> --session-id <session-id> --message "checked failure state" --image image.jpg`
- `finish`: the run is complete; inspect logs or report, and do not resume anything.

Repeat `watch` and `resume` until `watch` reports `finish`. If the run should stop at a pause, use:

```bash
qamule pause abort <pause_id> --session-id <session-id> --message "stop run" --image screenshot.png
```

## Report CLI

Reports are stored in the project root as `<session-id>`. Useful queries:

```bash
qamule report status --session-id <session-id>
qamule report list --session-id <session-id>
qamule report list --session-id <session-id> --outcome failed
qamule report list --session-id <session-id> --with evidence
qamule report show tests/test_a.py::test_example --session-id <session-id>
qamule report failures --session-id <session-id>
qamule report checkpoints --session-id <session-id>
qamule report rebuild --session-id <session-id>
```
