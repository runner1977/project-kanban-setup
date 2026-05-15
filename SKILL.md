---
name: project-kanban-setup
description: Bootstrap a new software project on the Hermes Kanban board. Collects project details, validates inputs, previews the task graph, then creates it with idempotency guarantees.
version: 1.3.0
metadata:
  hermes:
    tags: [project-setup, kanban, automation]
---

# Project Kanban Setup Skill

Guides the agent through setting up a new project on the Hermes Kanban board: collect parameters interactively, validate them, print a task-preview for user confirmation, then create the full task graph with proper dependencies.

## When to invoke

When the user expresses intent to start a new project (e.g. "create a fixed assets management system", "start the book lending app"). Load this skill and follow the steps below.

---

## Step 0 – Pre-flight check

Before prompting the user, set a working directory for tasks that need a local filesystem:

```bash
export HERMES_KANBAN_WORKSPACE=<storage_path>
```

---

## Step 1 – Collect parameters (one by one)

Ask the user for each value. Accept raw Enter to use the suggested default. **Do not proceed** until a non-empty answer is given for every field.

| # | Field | Prompt text | Default (Enter to accept) |
|---|-------|-------------|--------------------------|
| 1 | `PROJECT_NAME` | "项目名称是什么？（英文或拼音，用于生成唯一标识）" | *(none – required)* |
| 2 | `STORAGE_PATH` | "项目代码存放路径是？" | `/data/workspace/<PROJECT_NAME>` |
| 3 | `FRONTEND` | "前端技术选型？（如 React / Vue / Svelte / 纯 HTML+JS）" | `React` |
| 4 | `BACKEND` | "后端技术选型？（如 Python/FastAPI / Node/Express / Java/Spring Boot）" | `Python/FastAPI` |
| 5 | `DATABASE` | "数据库选型？（如 PostgreSQL / MySQL / SQLite）" | `PostgreSQL` |

### Validation rules

- `PROJECT_NAME`: if empty after stripping, re-prompt. No default allowed.
- `STORAGE_PATH`: if empty, use the default. Must be an absolute path.
- `FRONTEND` / `BACKEND` / `DATABASE`: if empty, use the defaults above.
- Warn the user if the storage path already exists and contains files, but do not block.

### Derived variables

After all fields are collected, compute:

```bash
SLUG=$(echo "$PROJECT_NAME" | tr '[:upper:]' '[:lower:]' | tr ' ' '-' | tr -s '-')
export HERMES_TENANT=$SLUG
```

---

## Step 2 – Tenant / board existence check

Run:

```bash
hermes kanban list --tenant $SLUG 2>/dev/null
```

- If the command returns 0 and shows existing tasks → inform the user:
  > 项目 **$SLUG** 的看板已存在，已创建的任务不会被重复创建（幂等键保护）。可以继续添加新任务，或跳过初始化直接开始。
  > 
  > 已有任务：`hermes kanban list --tenant $SLUG`
- If the command returns non-zero → run `hermes kanban init` to initialize the board.

---

## Step 3 – Dry-run: print the task graph for confirmation

**Before creating any tasks**, print the following table and ask the user to confirm before proceeding. This avoids wasted or duplicate work.

```
项目: $PROJECT_NAME ($SLUG)
技术栈: 前端=$FRONTEND | 后端=$BACKEND | 数据库=$DATABASE
存储路径: $STORAGE_PATH

任务预览:
────────────────────────────────────────────────────────────────────
#  任务名称                      角色            依赖
────────────────────────────────────────────────────────────────────
1  设计 ${PROJECT_NAME} 界面原型     ui-designer     (无)
2  设计 ${PROJECT_NAME} 数据库模型    backend-dev     (无)
3  实现前端页面                  frontend-dev     #1
4  实现后端 API                  backend-dev      #2
5  编写前端自动化测试             qa-dev           #3
6  编写后端 API 测试              qa-dev           #4
7  编写 API 接口文档              tech-writer      #4
8  编写用户操作手册              tech-writer      #3
9  前后端联调集成                 backend-dev      #3, #4
10 代码审查                      code-reviewer    #3
11 安全审查                      code-reviewer    #4
12 SQL 审查                      code-reviewer    #2
13 性能审查                      code-reviewer    #4
14 端到端系统测试                 qa-dev           #5, #6, #9, #10, #11, #13
────────────────────────────────────────────────────────────────────
```

Ask: "以上是即将创建的任务预览，确认创建吗？（yes/no）"

- If **yes** → proceed to Step 4.
- If **no** → abort. Tell the user: "已取消。如需调整参数，请重新说 '创建 XX 项目'。"
- If any other answer → re-ask the confirmation question.

---

## Step 4 – Create tasks (idempotent, with error handling)

For each task: capture the JSON output, parse the `id` with `jq -r .id`, and store it in a named variable. If the command fails (non-zero exit code), **stop and report the failure** — do not continue to the next task.

The idempotency key for every task is `<slug>-<task-key>`. If a task with that key already exists, `hermes kanban create` returns the existing task's id (not a new one). This means re-running the skill is always safe.

### 4.1 UI Design

```bash
UI_DESIGN=$(hermes kanban create \
  "设计 ${PROJECT_NAME} 界面原型" \
  --assignee ui-designer \
  --priority 1 \
  --body "根据用户需求设计 ${PROJECT_NAME} 的界面原型图，包括列表页、详情页、添加/编辑表单。使用前端技术：${FRONTEND}。" \
  --idempotency-key ${SLUG}-ui-design \
  --json | jq -r '.id // empty')
echo "UI_DESIGN=$UI_DESIGN"
```

### 4.2 Database Model

```bash
DB_MODEL=$(hermes kanban create \
  "设计 ${PROJECT_NAME} 数据库模型" \
  --assignee backend-dev \
  --priority 1 \
  --body "设计 ${PROJECT_NAME} 的核心数据表结构（字段、关联关系、索引）。使用数据库：${DATABASE}。" \
  --idempotency-key ${SLUG}-db-model \
  --json | jq -r '.id // empty')
echo "DB_MODEL=$DB_MODEL"
```

### 4.3 Frontend Implementation (depends on UI Design)

```bash
FRONTEND_IMPL=$(hermes kanban create \
  "实现 ${PROJECT_NAME} 前端页面" \
  --assignee frontend-dev \
  --parent $UI_DESIGN \
  --body "基于 UI 原型实现前端页面，使用 ${FRONTEND}。实现列表展示、详情页、添加/编辑表单等功能。" \
  --idempotency-key ${SLUG}-frontend-impl \
  --json | jq -r '.id // empty')
echo "FRONTEND_IMPL=$FRONTEND_IMPL"
```

### 4.4 Backend API (depends on DB Model)

```bash
BACKEND_API=$(hermes kanban create \
  "实现 ${PROJECT_NAME} 后端 API" \
  --assignee backend-dev \
  --parent $DB_MODEL \
  --body "使用 ${BACKEND} 实现 ${PROJECT_NAME} 的 RESTful API，提供增删改查接口。" \
  --idempotency-key ${SLUG}-backend-api \
  --json | jq -r '.id // empty')
echo "BACKEND_API=$BACKEND_API"
```

### 4.5 Frontend Tests (depends on Frontend Implementation)

```bash
TEST_FRONTEND=$(hermes kanban create \
  "编写 ${PROJECT_NAME} 前端自动化测试" \
  --assignee qa-dev \
  --parent $FRONTEND_IMPL \
  --body "使用适合 ${FRONTEND} 的测试框架（如 Jest + Testing Library / Cypress 等）编写组件和交互测试。" \
  --idempotency-key ${SLUG}-frontend-test \
  --json | jq -r '.id // empty')
echo "TEST_FRONTEND=$TEST_FRONTEND"
```

### 4.6 Backend Tests (depends on Backend API)

```bash
TEST_BACKEND=$(hermes kanban create \
  "编写 ${PROJECT_NAME} 后端 API 测试" \
  --assignee qa-dev \
  --parent $BACKEND_API \
  --body "使用 ${BACKEND} 对应的测试框架（如 Pytest / JUnit / SuperTest 等）编写 API 单元测试和集成测试。" \
  --idempotency-key ${SLUG}-backend-test \
  --json | jq -r '.id // empty')
echo "TEST_BACKEND=$TEST_BACKEND"
```

### 4.7 API Documentation (depends on Backend API)

```bash
DOC_API=$(hermes kanban create \
  "编写 ${PROJECT_NAME} API 接口文档" \
  --assignee tech-writer \
  --parent $BACKEND_API \
  --body "使用 Swagger/OpenAPI 或类似工具生成 ${PROJECT_NAME} 的 API 文档，包含请求示例、响应结构和错误码说明。" \
  --idempotency-key ${SLUG}-doc-api \
  --json | jq -r '.id // empty')
echo "DOC_API=$DOC_API"
```

### 4.8 User Manual (depends on Frontend Implementation)

```bash
DOC_USER=$(hermes kanban create \
  "编写 ${PROJECT_NAME} 用户操作手册" \
  --assignee tech-writer \
  --parent $FRONTEND_IMPL \
  --body "编写图文并茂的用户手册，指导用户完成 ${PROJECT_NAME} 的常见操作流程。" \
  --idempotency-key ${SLUG}-doc-user \
  --json | jq -r '.id // empty')
echo "DOC_USER=$DOC_USER"
```

### 4.9 Integration (depends on Frontend Implementation + Backend API)

```bash
INTEGRATION=$(hermes kanban create \
  "${PROJECT_NAME} 前后端联调集成" \
  --assignee backend-dev \
  --parent $FRONTEND_IMPL $BACKEND_API \
  --body "将 ${PROJECT_NAME} 的前端页面与后端 API 进行联调，确保数据交互正常，处理跨域、认证等问题。" \
  --idempotency-key ${SLUG}-integration \
  --json | jq -r '.id // empty')
echo "INTEGRATION=$INTEGRATION"
```

### 4.10 End-to-End Test (depends on all three above)

```bash
E2E_TEST=$(hermes kanban create \
  "${PROJECT_NAME} 端到端系统测试" \
  --assignee qa-dev \
  --parent $TEST_FRONTEND $TEST_BACKEND $INTEGRATION \
  --body "执行完整的 ${PROJECT_NAME} 用户流程测试：验证增删改查全流程，确保系统整体功能正确。" \
  --idempotency-key ${SLUG}-e2e-test \
  --json | jq -r '.id // empty')
echo "E2E_TEST=$E2E_TEST"
```

### Error handling

After each `hermes kanban create`, check `$?` or whether the variable is empty. If a task fails:

```
错误：创建任务 "<task name>" 失败（exit code: $?）。
已创建的任务未受影响。租户看板路径：hermes kanban list --tenant $SLUG
```

Do not continue to the next task after a failure.

---

## Step 5 – Report

Print a clean summary in Chinese:

```
项目 [$PROJECT_NAME] ($SLUG) 看板任务创建完成。

技术栈：前端=${FRONTEND} | 后端=${BACKEND} | 数据库=${DATABASE}
代码路径：$STORAGE_PATH

已创建 / 更新的任务：
  T1  UI 设计          $UI_DESIGN
  T2  数据库模型        $DB_MODEL
  T3  前端实现          $FRONTEND_IMPL
  T4  后端 API         $BACKEND_API
  T5  前端测试         $TEST_FRONTEND
  T6  后端测试         $TEST_BACKEND
  T7  API 文档         $DOC_API
  T8  用户手册         $DOC_USER
  T9  前后端集成        $INTEGRATION
  T10 端到端测试       $E2E_TEST

查看看板：hermes kanban list --tenant $SLUG
触发调度：hermes kanban dispatch --tenant $SLUG
```

---

## Step 6 – Optional: start the dispatcher

If the user says "开始工作" or "启动调度器", run:

```bash
hermes kanban dispatch --tenant $SLUG
```

---

## Skill Git push

When updating this skill and pushing to GitHub, see `references/git-push-pattern.md` for the correct workflow (work from the skill dir, not /tmp clone, handle branch name mismatch).

---

## Design notes for the agent

- **幂等键**：每个任务用 `--idempotency-key ${SLUG}-<key>` 创建。重复执行此 skill 不会产生重复任务，已存在的任务会直接返回原 ID。
- **变量占位符**：任务 body 文本中的 `${PROJECT_NAME}` / `${FRONTEND}` 等在 Step 1 收集完毕后必须全部展开，再执行 `hermes kanban create`。不要留下字面量 `<frontend>` 等占位符。
- **强制校验**：`PROJECT_NAME` 不得为空，存储路径必须是绝对路径。默认值只在用户敲 Enter 时生效，用户输入了任何字符都必须使用用户值。
- **不自己做实现**：agent 的角色是构建任务图并分派，不负责写代码。所有任务 assign 给专用的 profile（ui-designer / frontend-dev / backend-dev / qa-dev / tech-writer）。
- **Dry-run 优先**：永远先打印预览再执行，这是防止浪费工作和收集用户意图的最后一道检查。

---

> **End of skill.** When the user says "create <X> project", load this skill (`skill_view(name='project-kanban-setup')`) and follow the steps above.
