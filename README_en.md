# project-kanban-setup

Hermes Agent skill: bootstrap a new software project with the Hermes Kanban board.

[中文说明](README.md)

---

## Overview

This skill guides you through setting up a new software project on the Hermes Kanban board. The flow:

1. **Collect project details**
   - Project name (required, used as unique identifier)
   - Storage path (default: `/data/workspace/<project-name>`)
   - Frontend technology (default: React)
   - Backend technology (default: Python/FastAPI)
   - Database choice (default: PostgreSQL)

2. **Dry-run preview**
   - Print the full task dependency graph before creating anything
   - Wait for user confirmation (yes/no) before proceeding

3. **Idempotent task creation**
   - Each task uses `--idempotency-key` to prevent duplicates
   - Re-running the skill never creates duplicate tasks

4. **Initial task graph**
   - UI Design → Frontend Implementation → Frontend Tests
   - Database Model → Backend API → Backend Tests
   - API Documentation / User Manual
   - Integration → End-to-End Tests

## Usage

In your Hermes Agent session, simply say:

```
Create a fixed assets management system
```

The agent will automatically:
1. Load this skill
2. Prompt for project parameters one by one (press Enter to accept defaults)
3. Print the task preview table and wait for your confirmation
4. Initialize the Kanban board if not already done
5. Create all tasks in dependency order
6. Report back with task IDs

You can then optionally trigger the dispatcher to start work immediately.

## File Structure

```
project-kanban-setup/
├── SKILL.md          # Skill definition
├── README.md         # Chinese documentation
├── README_en.md      # This file (English)
└── references/       # Additional resources (if any)
```

## Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| Project name | English or pinyin, used as tenant identifier | (required) |
| Storage path | Where source code will be stored | `/data/workspace/<project-name>` |
| Frontend | Frontend framework or library | `React` |
| Backend | Backend language/framework | `Python/FastAPI` |
| Database | Database type | `PostgreSQL` |

## Task Dependencies

```
T1: UI Design                   (no parents)
T2: Database Model              (no parents)
T3: Frontend Implementation     ← T1
T4: Backend API                 ← T2
T5: Frontend Tests              ← T3
T6: Backend Tests                ← T4
T7: API Documentation            ← T4
T8: User Manual                  ← T3
T9: Frontend-Backend Integration ← T3, T4
T10: End-to-End Tests            ← T5, T6, T9
```

## Common Commands

```bash
# List tasks for a project
hermes kanban list --tenant <slug>

# Trigger the dispatcher to start work
hermes kanban dispatch --tenant <slug>

# Re-initialize the board (idempotent)
hermes kanban init
```

## Requirements

- Hermes Agent installed
- GitHub Token configured in `.env` (for repository creation)
- Git configured with username and email

## License

MIT
