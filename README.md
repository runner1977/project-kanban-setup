# project-kanban-setup

Hermes Agent skill：通过看板系统引导新软件项目启动。

[English README](README_en.md)

---

## 功能概述

此技能帮助您通过 Hermes 看板系统快速启动一个新的软件项目，流程如下：

### 1. 收集项目信息
- 项目名称（必填，用作唯一标识）
- 存储路径（默认：`/data/workspace/<项目名称>`）
- 前端技术选型（默认：React）
- 后端技术选型（默认：Python/FastAPI）
- 数据库选型（默认：PostgreSQL）

### 2. 幂等初始化
- 检查租户看板是否已存在，避免重复创建
- 支持重复执行，不会产生重复任务

### 3. Dry-run 预览
- 在创建任何任务前，打印完整任务依赖图
- 等待用户确认（yes/no）后再执行

### 4. 创建初始任务依赖图（14 个任务）

```
T1:  UI 设计                   （无依赖）
T2:  数据库模型                 （无依赖）
T3:  前端实现                   ← T1
T4:  后端 API                  ← T2
T5:  前端自动化测试             ← T3
T6:  后端 API 测试              ← T4
T7:  API 接口文档               ← T4
T8:  用户操作手册               ← T3
T9:  前后端联调集成              ← T3, T4
T10: 代码审查                   ← T3
T11: 安全审查                   ← T4
T12: SQL 审查                   ← T2
T13: 性能审查                   ← T4
T14: 端到端系统测试              ← T5, T6, T9, T10, T11, T13
```

其中 T10-T13（代码审查、安全审查、SQL 审查、性能审查）由专门的 **code-reviewer** profile 执行，负责对前后端代码进行全方位质量把关。

## 使用方法

在 Hermes Agent 对话中直接告诉它要创建的项目，例如：

```
创建一个固定资产管理系统
```

Agent 将自动：
1. 加载此 Skill
2. 逐项询问参数（直接回车接受默认值）
3. 打印任务预览表并等待确认
4. 初始化看板（如尚未初始化）
5. 按依赖顺序创建所有任务
6. 返回已创建的任务 ID

您可以随后选择立即触发调度器开始工作。

## 参数说明

| 参数 | 说明 | 默认值 |
|------|------|--------|
| 项目名称 | 英文或拼音，作为租户唯一标识 | （必填） |
| 存储路径 | 源代码存放路径 | `/data/workspace/<项目名称>` |
| 前端 | 前端框架或类库 | `React` |
| 后端 | 后端语言/框架 | `Python/FastAPI` |
| 数据库 | 数据库类型 | `PostgreSQL` |

## 任务依赖关系

```
T1:  UI 设计                   （无依赖）
T2:  数据库模型                 （无依赖）
T3:  前端实现                   ← T1
T4:  后端 API                  ← T2
T5:  前端自动化测试             ← T3
T6:  后端 API 测试              ← T4
T7:  API 接口文档               ← T4
T8:  用户操作手册               ← T3
T9:  前后端联调集成              ← T3, T4
T10: 代码审查                   ← T3（前端代码）
T11: 安全审查                   ← T4（后端代码）
T12: SQL 审查                   ← T2（数据库模型）
T13: 性能审查                   ← T4（后端 API）
T14: 端到端系统测试              ← T5, T6, T9, T10, T11, T13
```

## 常用命令

```bash
# 查看看板任务
hermes kanban list --tenant <slug>

# 触发调度器开始工作
hermes kanban dispatch --tenant <slug>

# 重新初始化看板（幂等操作）
hermes kanban init
```

## 角色说明

| 角色 | 职责 |
|------|------|
| ui-designer | UI 界面原型设计 |
| frontend-dev | 前端页面实现 |
| backend-dev | 后端 API 实现、联调集成 |
| qa-dev | 单元测试、E2E 测试 |
| tech-writer | API 文档、用户手册 |
| code-reviewer | 代码审查、安全审查、SQL 审查、性能审查 |

## 文件结构

```
project-kanban-setup/
├── SKILL.md          # 技能定义文件
├── README.md         # 本说明文档（中文）
├── README_en.md      # 英文说明文档
└── references/       # 其他资源（如有）
```

## 前置要求

- 已安装 Hermes Agent
- `.env` 文件中配置了 GitHub Token
- Git 已配置用户名和邮箱

## 安装此 Skill

```bash
hermes skills install https://github.com/runner1977/project-kanban-setup/raw/main/SKILL.md
```

## License

MIT
