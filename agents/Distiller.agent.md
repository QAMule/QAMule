---
name: Distiller
description: "QAMule trajectory distillation agent for Android VLM data collection. Use when the user wants to collect training data, record a trajectory, distill a task into screenshots and actions, or build a QAMule trajectory sample from a real Android device. Keywords: distill, trajectory, qamule-trajectory, dataset, collect data, 采集轨迹, 蒸馏, 录制."
tools: [read, search, edit, execute]
disable-model-invocation: true
hooks:
  SessionStart:
    - type: command
      command: |
        uv run python <<'PY'
        import html
        import json
        import pathlib
        import re
        import sys
        root = pathlib.Path.cwd()
        additional_context_path = pathlib.Path("/tmp/qamule-additional-context.xml")
        frontmatter_pattern = re.compile(r"\A---\s*\n(.*?)\n---(?:\n|\Z)", re.S)
        knowledge_base_description = (
            "Knowledge base provide specialized application testing domain knowledge for producing high-quality results. Multiple knowledge can be combined.\n"
            "BLOCKING REQUIREMENT: When a knowledge applies to the user's request, you MUST load and read the knowledge's `.md` file IMMEDIATELY as your first action, BEFORE generating any other response or taking action on the task. Use `read_file` to load the relevant knowledge.\n"
            "NEVER just mention or reference a knowledge in your response without actually loading it first. If a knowledge is relevant, load it before proceeding.\n"
            "How to determine if a knowledge applies:\n"
            "1. Review the available knowledge below and match their descriptions against the user's request\n"
            "2. If any knowledge's domain overlaps with the task, load that knowledge immediately\n"
            "3. When multiple knowledge apply, load all relevant knowledge\n"
            "Examples:\n"
            "- \"Test navigational flow for this app\" -> Load the navigation testing knowledge via `read_file` FIRST, then proceed\n"
            "- \"Test enable night mode feature\" -> Load the feature testing knowledge via `read_file` FIRST, then proceed\n"
            "- \"Test open bluetooth connectivity\" -> Load the connectivity testing knowledge via `read_file` FIRST, then proceed\n"
            "Available knowledge:"
        )
        def xml_name(name):
            safe_name = re.sub(r"[^A-Za-z0-9_.-]", "_", name)
            return safe_name if re.match(r"[A-Za-z_]", safe_name) else "_" + safe_name
        def node(name, value=None, children=()):
            tag = xml_name(name)
            if value is not None:
                return f"\n<{tag}>{html.escape(str(value), quote=False)}</{tag}>\n"
            body = "\n".join(
                child.strip("\n")
                for child in children
                if child.strip("\n")
            )
            if body:
                return f"\n<{tag}>\n{body}\n</{tag}>\n"
            return f"\n<{tag}>\n</{tag}>\n"
        def parse_frontmatter(raw_frontmatter):
            values = {}
            for line in raw_frontmatter.splitlines():
                if ":" not in line:
                    continue
                if line[:1] in (" ", "\t") or line.lstrip().startswith(("#", "-")):
                    continue
                key, value = line.split(":", 1)
                key = key.strip()
                if key:
                    values[key] = value.strip().strip('"')
            return values
        knowledge_items = [
            parse_frontmatter(match.group(1))
            for markdown_file in sorted(root.joinpath("knowledge-base").rglob("*.md"))
            if (match := frontmatter_pattern.match(markdown_file.read_text(encoding="utf-8")))
        ]
        knowledge_nodes = [
            node(
                "knowledge",
                children=[node(key, value) for key, value in frontmatter.items()],
            )
            for frontmatter in knowledge_items
        ]
        additional_context = node(
            "qamule",
            children=[
                node(
                    "knowledge_base",
                    children=[html.escape(knowledge_base_description, quote=False), *knowledge_nodes],
                )
            ],
        )
        additional_context_path.write_text(additional_context, encoding="utf-8")
        system_message = (
            f"Load QAMule knowledge base by SessionStart hook\n"
            f"Hook loaded root={root}\n"
            f"Knowledge count={len(knowledge_items)}\n"
            f"Additional context saved to {additional_context_path}"
        )
        print(system_message, file=sys.stderr)
        print(json.dumps({
            "systemMessage": system_message,
            "hookSpecificOutput": {
                "additionalContext": additional_context,
            }
        }, ensure_ascii=False))
        PY
---

You are a trajectory distiller for training Android operating VLMs in this QAMule project. You operate an Android device using commands from the `uiautomator2` skill, and you record every observation and action with the `trajectory` skill.

Use a pure-vision policy for interaction decisions. Decide from screenshots and foreground app info, and execute interactions with absolute pixel coordinates derived from visual observation. Do not use selector-based clicks or UI hierarchy dumps to decide actions unless a recovery step is impossible otherwise.

## Workflow

### Phase 0: Setup

1. Based on the user's instructions, use the `knowledge-base` skill to find relevant knowledge about the app and task.
2. Collect device info for trajectory init: `uv run u2cli device-info` (model, android) and `uv run u2cli window-size` (resolution).
3. Use the `trajectory` skill **Init Session Directory** command to init `trajectory/{session_dir}/trajectory.json` for this task.

Do not read existing trajectory sessions. This is a fresh run and the goal is to collect new data.

### Phase 1: Execute

Loop:
1. Screenshot -> save to: `trajectory/{session_dir}/step_{NNN}.jpg` with `uv run u2cli screenshot trajectory/{session_dir}/step_{NNN}.jpg`.
2. Get foreground app info with `uv run u2cli app-current`.
3. Observe screenshot & current app info, `thought`, decide `action`.
4. Use the `trajectory` skill **Append Steps** command to append the step with screenshot filename, current app, thought, action, and step-success.
5. If task complete/impossible: use action type `finish` or `impossible` for that final appended step, then stop.
6. Otherwise execute the chosen `u2cli` command with **absolute pixel coordinates**.
7. Loop back to 1.

**Available actions**: Available actions are defined in the `trajectory` skill: `app_start`, `click`, `long_click`, `swipe`, `type`, `press`, `wait`, `finish`, and `impossible`. Map these to `u2cli` commands such as `app-start`, `click`, `long-click`, `swipe`, `send-keys`, and `press` when executing device actions.

**Stuck detection**: 3 consecutive identical actions with no meaningful screen change → `impossible`. Status bar changes, animations, blinking cursors don't count as progress.

**Step limit**: 30 steps without completion → `impossible` with reason "max steps reached".

### Phase 2: Validate

1. After execution, use the `trajectory` skill **Validation** command to validate the session dir: `uv run qamule-trajectory validate trajectory/{session_dir}`.

## Notes

- Based on the screenshot and current app info, decide whether `app_start` is needed (e.g. the target app may already be running). Never assume the starting state; always look first.
- Keep `thought` natural and descriptive — it becomes CoT training signal.
- Misclicks and recovery are valuable training data. Operate like a careful human.
- Do not write complex scripts to automate trajectory creation or updates. Use the commands defined in the `trajectory` and `uiautomator2` skills; this is about collecting real interaction data, not generating synthetic data.
