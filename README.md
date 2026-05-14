# project-kanban-setup

Hermes Agent 技能：通过看板系统引导新软件项目启动。

[English README](README_en.md)

---

## 功能概述

此技能帮助您通过 Hermes 看板系统快速启动一个新的软件项目，流程如下：

1. **收集项目信息**
   - 项目名称（必填，用于生成唯一标识）
   - 存储路径（默认为 `/data/workspace/<项目名称>`）
   - 前端技术选型（默认 React）
   - 后端技术选型（默认 Python/FastAPI）
   - 数据库选型（默认 PostgreSQL）

2. **任务预览（Dry-run）**
   - 正式创建前打印完整任务依赖图
   - 等待用户确认（yes/no）后再执行

3. **幂等任务创建**
   - 使用 `--idempotency-key` 防止重复
   - 重复执行不会创建重复任务

4. **创建初始任务图**
   - UI 设计 → 前端实现 → 前端测试
   - 数据库模型 → 后端 API → 后端测试
   - API 文档 / 用户手册
   - 前后端集成 → 端到端测试

## 使用方法

在 Hermes Agent 对话中直接告诉它要创建的项目，例如：

```
创建一个固定资产管理系统
```

Agent 将自动：
1. 加载此 Skill
2. 依次询问项目参数（可直接回车接受默认值）
3. 打印任务预览表，等待您确认
4. 初始化看板（如尚未初始化）
5. 按依赖顺序创建所有任务
6. 返回创建结果和任务 ID

确认后可选择立即触发调度器开始工作。

## 文件结构

```
project-kanban-setup/
├── SKILL.md          # 技能定义文件
├── README.md         # 本说明文档（中文）
├── README_en.md      # English documentation
└── references/       # 其他资源（如有）
```

## 参数说明

| 参数 | 说明 | 默认值 |
|------|------|--------|
| 项目名称 | 英文或拼音，用于生成租户标识 | （必填） |
| 存储路径 | 项目代码存放路径 | `/data/workspace/<项目名称>` |
| 前端技术 | 前端框架或库 | `React` |
| 后端技术 | 后端语言/框架 | `Python/FastAPI` |
| 数据库 | 数据库类型 | `PostgreSQL` |

## 任务依赖关系

```
T1: UI 设计                   (无依赖)
T2: 数据库模型                (无依赖)
T3: 前端实现                 ← T1
T4: 后端 API                 ← T2
T5: 前端测试                 ← T3
T6: 后端测试                 ← T4
T7: API 文档                 ← T4
T8: 用户手册                 ← T3
T9: 前后端集成               ← T3, T4
T10: 端到端测试              ← T5, T6, T9
```

## 常用命令

```bash
# 查看已创建的任务
hermes kanban list --tenant <slug>

# 触发调度器开始工作
hermes kanban dispatch --tenant <slug>

# 重新初始化看板（幂等操作）
hermes kanban init
```

## 前置要求

- 已安装 Hermes Agent
- `.env` 文件中配置了 GitHub Token（用于创建仓库）
- Git 已配置用户名和邮箱

## License

MIT
