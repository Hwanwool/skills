---
name: create-subagent
description: 创建面向特定 AI 任务的专用子代理（omp agent）。当用户想要新建子代理、配置任务专用代理或定制专用 AI 工作流时使用。
---

# 创建子代理（Create Subagent）

## 什么是 omp 子代理

omp 通过 `task` 工具把任务委派给专用子代理（worker agent）。子代理是 Markdown 文件（YAML frontmatter + 指令正文），存放位置：

| 类型 | 路径 | 作用域 |
|---|---|---|
| 用户级 | `~/.omp/agent/agents/<name>.md` | 所有会话可用 |
| 项目级 | `./.omp/agents/<name>.md` | 仅该项目可用 |

内置 5 个样例代理可用 `omp agents unpack` 导出参考：`reviewer`（代码审查）、`scout`（代码侦察）、`security-reviewer`（安全审查）、`sonic`、`task`（通用多步任务）。创建前先确认内置代理未覆盖目标职责。

## 文件结构

```markdown
---
name: my-agent                 # 必填，小写字母/数字/连字符
description: 一句话说明职责    # 必填，主代理据此选择委派对象
tools:                         # 可选，工具白名单（不写则按默认）
  - read
  - grep
  - bash
spawns:                        # 可选，本代理可派生的下一层代理名
  - scout
model:                         # 可选，角色名（"@task"/"@slow"/"@smol"）或模型
  - "@task"
thinkingLevel: auto            # 可选
output:                        # 可选，结构化输出 schema
  properties:                  # 必填字段
    verdict:
      metadata:
        description: 结论说明
      type: string
  optionalProperties:          # 可选字段
    findings:
      elements:
        properties:
          title:
            type: string
---

# 正文：worker 指令

- 用 `<procedure>` 写明执行步骤
- 用 `<directives>` 写明硬性规则
- 工具使用约束、产出格式写清楚
```

## 创建流程

1. 明确子代理职责与触发场景（主代理何时应委派给它）
2. 设计 frontmatter：必填 `name`、`description`；按需 `tools`、`spawns`、`model`、`output`
3. 写正文指令：步骤、规则、输出格式，只写通用模型不知道的专用内容
4. 写入 `~/.omp/agent/agents/<name>.md`（用户级）或 `./.omp/agents/<name>.md`（项目级）
5. 新会话验证：让主代理执行一个适合该子代理的任务，确认 task 工具选中它且产出符合预期

## 设计原则

- `description` 是主代理选择子代理的唯一依据：写清「做什么 + 何时用」
- `tools` 白名单最小化：只给任务必需的工具，降低越权风险
- 正文用 XML 标签分区（`<procedure>`/`<directives>`/`<criteria>`/`<output>`），与内置 agent 风格一致
- 需要结构化产出时用 `output` schema（`properties` 必填、`optionalProperties` 可选）
- 多代理协作用 `spawns` 声明可派生的下一层代理

## 反模式

- `description` 含糊（如"帮我干活"）→ 主代理无法正确选择
- 不给 `tools` 白名单（全量工具）→ 最小权限原则失效
- 正文写通用知识（如何 git commit 等）→ 浪费 token，只写专用内容
- 与内置 agent 职责重复 → 先检查 reviewer/scout/task 等是否已覆盖
