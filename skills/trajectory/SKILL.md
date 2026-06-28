---
name: trajectory
description: >
  Manage VLM trajectory data. Use when creating sessions, writing trajectory.json, validating samples, or viewing the trajectory.
user-invocable: false
---

# Trajectory

This skill helps you to create, update, validate and view VLM trajectory sessions.

## Init Session Directory

Initialize once at the beginning of a session. This will create a new directory for the session and initialize the trajectory.json file.

```bash
uv run qamule-trajectory init trajectory/open_wifi_20260626_120000 \
	--instruction "Open Wi-Fi in Settings" \
	--app com.android.settings \
	--device-model "Pixel 8" \
	--resolution 1080 2400 \
	--android 14
```

It will print the created session directory.

## Screenshot

Save screenshots as `step_{NNN}.jpg` (3-digit zero padding) in the session directory.

## Append Steps

```bash
uv run qamule-trajectory append trajectory/{session_dir} \
  --screenshot step_001.jpg \
  --current-app com.example.app/.MainActivity \
  --thought "I can see the main screen and the login button is visible." \
  --action '{"type":"click","x":512,"y":750}' \
  --step-success true
```

### Available Actions

| type | params | notes |
|------|--------|-------|
| `app_start` | `app, activity(optional)` | Launch app |
| `click` | `x, y` | Raw pixel coords |
| `long_click` | `x, y` | Raw pixel coords |
| `swipe` | `x1, y1, x2, y2` | Raw pixel coords |
| `type` | `text` | Into focused field |
| `press` | `key` | Hardware/soft key |
| `wait` | `duration` | Sleep N seconds (e.g. waiting for page load) |
| `finish` | `reason` | Terminal, task completed |
| `impossible` | `reason` | Terminal, task cannot be completed |

## Validation

Validate the trajectory data to ensure it is well-formed and complete.

```bash
# Validate a single session directory
uv run qamule-trajectory validate trajectory/{session_dir}
# Validate all sessions in a directory
uv run qamule-trajectory validate trajectory
```

## View trajectory in a web browser

```bash
uv run qamule-trajectory view trajectory
```
