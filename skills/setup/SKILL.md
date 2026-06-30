---
name: setup
description: "Set up a base UV project for QAMule testing and install the core dependencies. Use when setting up a new project, first-time setup, or when user says: setup, init, initialize, setup project."
argument-hint: "<project-name>"
disable-model-invocation: true
---

# Project Setup

Set up a base QAMule test project in the user's workspace.

## Procedure

### 1. Initialize Project

```bash
uv init --name <project-name> --no-package --bare
```

This creates the base UV project scaffold.

### 2. Add Dependencies

```bash
uv add pytest "pytest-qamule>=0.2.0" "uiautomator2>=3.7.0"
```

### 3. Install Dependencies

```bash
uv sync
```

After these steps, the workspace has a minimal QAMule-ready Python project with the core runtime dependencies installed.

### 4. Suggest Next Steps

After setting up the project, suggest useful next actions to the user, such as:

- Use the `knowledge-base` skill to import the basic knowledge base for testing.
- Ask the `QA` agent to explore or test the application.
- Ask the `Distiller` agent to generate VLM training data for the application.
