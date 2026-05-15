# project-kanban-setup

Hermes Agent skill: bootstrap a new software project with the Hermes Kanban board.

[中文说明](README.md)

---

## Overview

This skill guides you through setting up a new software project on the Hermes Kanban board. The flow:

### 1. Collect project details
- Project name (required, used as unique identifier)
- Storage path (default: `/data/workspace/<project-name>`)
- Frontend technology (default: React)
- Backend technology (default: Python/FastAPI)
- Database choice (default: PostgreSQL)

### 2. Idempotent initialization
- Check if the tenant board already exists to avoid duplicate creation
- Safe to re-run without creating duplicate tasks

### 3. Dry-run preview
- Print the full task dependency graph before creating anything
- Wait for user confirmation (yes/no) before proceeding

### 4. Initial task dependency graph (14 tasks)

```
T1:  UI Design                   (no parents)
T2:  Database Model              (no parents)
T3:  Frontend Implementation     ← T1
T4:  Backend API                 ← T2
T5:  Frontend Tests               ← T3
T6:  Backend Tests                ← T4
T7:  API Documentation            ← T4
T8:  User Manual                  ← T3
T9:  Frontend-Backend Integration ← T3, T4
T10: Code Review                  ← T3 (frontend code)
T11: Security Review              ← T4 (backend code)
T12: SQL Review                  ← T2 (database model)
T13: Performance Review          ← T4 (backend API)
T14: End-to-End Tests            ← T5, T6, T9, T10, T11, T13
```

Tasks T10–T13 (Code Review, Security Review, SQL Review, Performance Review) are executed by a dedicated **code-reviewer** profile, providing comprehensive quality assurance across the entire codebase.

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
T1:  UI Design                   (no parents)
T2:  Database Model              (no parents)
T3:  Frontend Implementation     ← T1
T4:  Backend API                 ← T2
T5:  Frontend Tests              ← T3
T6:  Backend Tests                ← T4
T7:  API Documentation            ← T4
T8:  User Manual                  ← T3
T9:  Frontend-Backend Integration ← T3, T4
T10: Code Review                  ← T3 (frontend code)
T11: Security Review              ← T4 (backend code)
T12: SQL Review                  ← T2 (database model)
T13: Performance Review          ← T4 (backend API)
T14: End-to-End Tests            ← T5, T6, T9, T10, T11, T13
```

## Role Definitions

| Role | Responsibility |
|------|----------------|
| ui-designer | UI prototype design |
| frontend-dev | Frontend page implementation |
| backend-dev | Backend API implementation, integration |
| qa-dev | Unit tests, E2E tests |
| tech-writer | API docs, user manual |
| code-reviewer | Code review, security review, SQL review, performance review |

## File Structure

```
project-kanban-setup/
├── SKILL.md          # Skill definition
├── README.md         # Chinese documentation
├── README_en.md      # This file (English)
└── references/       # Additional resources (if any)
```

## Common Commands

```bash
# List tasks for a project
hermes kanban list --tenant <slug>

# Trigger the dispatcher to start work
hermes kanban dispatch --tenant $SLUG

# Re-initialize the board (idempotent)
hermes kanban init
```

## Requirements

- Hermes Agent installed
- GitHub Token configured in `.env` (for repository creation)
- Git configured with username and email

## Install this Skill

```bash
hermes skills install https://github.com/runner1977/project-kanban-setup/raw/main/SKILL.md
```

## License

MIT
