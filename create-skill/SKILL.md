---
name: create-skill
description: 指导创建高效的 omp Agent Skill。Skill 是教会智能体执行特定任务的 Markdown 文件。当用户想要创建 skill 时，应优先使用本技能。
---

# 在 omp 中创建 Skill

本技能指导你为 omp 创建高效的 Agent Skill。Skill 是教会智能体执行特定任务的 Markdown 文件：按团队标准审查 PR、以指定格式生成提交信息、查询数据库结构，或任何专门的工作流。

## 开始之前：收集需求

创建 skill 前，从用户处收集关键信息：

1. **目的与范围**：这个 skill 应帮助完成什么具体任务或工作流？
2. **存放位置**：个人 skill（~/.agents/skills/）还是项目 skill（.agents/skills/）？
3. **触发场景**：智能体应在什么时机自动应用此 skill？
4. **关键领域知识**：智能体需要哪些它本身不知道的专业信息？
5. **输出格式偏好**：是否有指定的模板、格式或风格要求？
6. **既有模式**：是否有现成的示例或约定可参照？

### 从上下文推断

若有此前的对话上下文，可从讨论中推断 skill。可基于对话中涌现的工作流、模式或领域知识来创建。

### 补充信息收集

需要澄清时，以文本「单问题 + 1~3 选项」发问（如已启用 ask-via-clarify 提问规范，则遵守其规则）：

```
提问示例：
- "skill 存放在哪里？" 选项如 ["个人 (~/.agents/skills/)", "项目 (.agents/skills/)"]
- "skill 是否包含可执行脚本？" 选项如 ["是", "否"]
```

---

## Skill 文件结构

### 目录布局

Skill 以目录形式存放，内含 `SKILL.md` 文件：

```
skill-name/
├── SKILL.md              # 必需 - 主指令
├── reference.md          # 可选 - 详细文档
├── examples.md           # 可选 - 使用示例
└── scripts/              # 可选 - 工具脚本
    ├── validate.py
    └── helper.sh
```

### 存放位置

| 类型 | 路径 | 作用域 |
|------|------|-------|
| 个人 | ~/.agents/skills/skill-name/ | 跨所有项目可用 |
| 项目 | .agents/skills/skill-name/ | 与使用该仓库的成员共享 |

### SKILL.md 结构

每个 skill 都要求一个带 YAML frontmatter 与 Markdown 正文的 `SKILL.md` 文件：

```markdown
---
name: your-skill-name
description: 简述此 skill 的作用与使用时机
---

# 你的 Skill 名称

## 指令
给智能体的清晰、分步指导。

## 示例
使用此 skill 的具体示例。
```

### 必需元数据字段

| 字段 | 要求 | 用途 |
|-------|------|------|
| `name` | 最长 64 字符，仅小写字母/数字/连字符 | skill 的唯一标识 |
| `description` | 最长 1024 字符，非空 | 帮助智能体决定何时应用此 skill |

---

## 撰写高质量 Description

description 对 skill 的发现**至关重要**。智能体依据它决定是否应用你的 skill。

### Description 最佳实践

1. **用第三人称书写**（description 会注入系统提示）：
    - 正确："处理 Excel 文件并生成报告"
    - 避免："我可以帮你处理 Excel 文件"
    - 避免："你可以用它处理 Excel 文件"

2. **具体并包含触发词**：
    - 正确："从 PDF 文件提取文本和表格、填写表单、合并文档。当处理 PDF 文件或用户提及 PDF、表单、文档提取时使用。"
    - 含糊："帮助处理文档"

3. **同时包含"做什么"与"何时用"**：
    - 做什么：skill 的能力（具体功能）
    - 何时用：智能体应使用它的时机（触发场景）

### Description 示例

```yaml
# PDF 处理
description: 从 PDF 文件提取文本和表格、填写表单、合并文档。当处理 PDF 文件或用户提及 PDF、表单、文档提取时使用。

# Excel 分析
description: 分析 Excel 电子表格、创建数据透视表、生成图表。当分析 Excel 文件、电子表格、表格数据或 .xlsx 文件时使用。

# Git 提交助手
description: 通过分析 git diff 生成描述性提交信息。当用户请求帮助撰写提交信息或审查暂存的改动时使用。

# 代码审查
description: 按团队标准审查代码的质量、安全性与最佳实践。当审查拉取请求、代码改动，或用户请求代码审查时使用。
```

---

## 核心撰写原则

### 1. 简洁至上

上下文窗口与对话历史、其他 skill、请求共享，每个 token 都在竞争空间。

**默认假设**：智能体本身已经非常聪明，只添加它不知道的上下文。

逐条挑战每条信息：
- "智能体真的需要这段解释吗？"
- "能否假定智能体已经知道？"
- "这段内容值它的 token 成本吗？"

**好（简洁）**：
```markdown
## 提取 PDF 文本

使用 pdfplumber 提取文本：

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
```

**差（冗长）**：
```markdown
## 提取 PDF 文本

PDF（Portable Document Format）是一种常见的文件格式，包含
文本、图片和其他内容。要提取 PDF 中的文本，你需要
一个库。虽然有很多库可用于 PDF 处理，但我们
推荐 pdfplumber，因为它易于使用并且大多数情况下效果良好...
```

### 2. SKILL.md 控制在 500 行以内

为获得最佳性能，主 SKILL.md 文件应保持简洁。详细内容采用渐进式披露。

### 3. 渐进式披露

把必要信息放在 SKILL.md 中；把详细参考资料放进单独文件，智能体按需读取。

```markdown
# PDF 处理

## 快速开始
[必要指令在此]

## 补充资源
- 完整 API 细节见 [reference.md](reference.md)
- 使用示例见 [examples.md](examples.md)
```

**引用保持一层深度**——从 SKILL.md 直接链接到参考文件。过深的嵌套引用可能导致部分读取。

### 4. 设定合适的自由度

按任务的脆弱程度匹配具体性：

| 自由度 | 适用场景 | 示例 |
|--------|---------|------|
| **高**（文字指令） | 多种可行做法、依赖上下文 | 代码审查准则 |
| **中**（伪代码/模板） | 有偏好模式但允许变化 | 报告生成 |
| **低**（具体脚本） | 易错操作、一致性关键 | 数据库迁移 |

---

## 常用模式

### 模板模式

提供输出格式模板：

```markdown
## 报告结构

使用此模板：

```markdown
# [分析标题]

## 执行摘要
[一段话概述关键发现]

## 关键发现
- 发现 1 及支撑数据
- 发现 2 及支撑数据

## 建议
1. 具体可执行的建议
2. 具体可执行的建议
```
```

### 示例模式

当输出质量依赖范例时：

```markdown
## 提交信息格式

**示例 1：**
输入：使用 JWT 添加了用户认证
输出：
```
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware
```

**示例 2：**
输入：修复了日期显示错误的 bug
输出：
```
fix(reports): correct date formatting in timezone conversion

Use UTC timestamps consistently across report generation
```
```

### 工作流模式

把复杂操作拆成带检查清单的清晰步骤：

```markdown
## 表单填写工作流

复制此清单并跟踪进度：

```
任务进度：
- [ ] 步骤 1：分析表单
- [ ] 步骤 2：建立字段映射
- [ ] 步骤 3：验证映射
- [ ] 步骤 4：填写表单
- [ ] 步骤 5：核验输出
```

**步骤 1：分析表单**
运行：`python scripts/analyze_form.py input.pdf`
...
```

### 条件工作流模式

引导通过决策点：

```markdown
## 文档修改工作流

1. 判断修改类型：

   **创建新内容？** -> 按下方"创建工作流"执行
   **编辑既有内容？** -> 按下方"编辑工作流"执行

2. 创建工作流：
   - 使用 docx-js 库
   - 从头构建文档
   ...
```

### 反馈循环模式

对质量关键的任务，实现校验循环：

```markdown
## 文档编辑流程

1. 进行编辑
2. **立即校验**：`python scripts/validate.py output/`
3. 校验失败时：
   - 查看错误信息
   - 修复问题
   - 重新运行校验
4. **校验通过后才继续**
```

---

## 工具脚本

预置脚本相比现场生成代码有诸多优势：
- 比生成代码更可靠
- 节省 token（代码不进上下文）
- 节省时间（无需生成代码）
- 保证跨次使用的一致性

```markdown
## 工具脚本

**analyze_form.py**：提取表单全部字段
```bash
python scripts/analyze_form.py input.pdf > fields.json
```

**validate.py**：检查错误
```bash
python scripts/validate.py fields.json
# 返回："OK" 或列出冲突
```
```

明确脚本是**执行**（最常见）还是作为参考**阅读**。

---

## 需要避免的反模式

### 1. Windows 风格路径
- 使用：`scripts/helper.py`
- 避免：`scripts\helper.py`

### 2. 选项过多
```markdown
# 差 - 令人困惑
"你可以用 pypdf，或 pdfplumber，或 PyMuPDF，或..."

# 好 - 提供默认方案并留逃生口
"使用 pdfplumber 提取文本。
扫描版 PDF 需 OCR 时，改用 pdf2image 配合 pytesseract。"
```

### 3. 时间敏感信息
```markdown
# 差 - 会过时
"如果你在 2025 年 8 月前执行此操作，请使用旧 API。"

# 好 - 使用"旧模式"章节
## 当前方法
使用 v2 API 端点。

## 旧模式（已废弃）
<details>
<summary>旧版 v1 API</summary>
...
</details>
```

### 4. 术语不一致
选定一个术语并全程一致使用：
- 始终用"API 端点"（不混用"URL"、"路由"、"路径"）
- 始终用"字段"（不混用"输入框"、"元素"、"控件"）

### 5. 含糊的 Skill 名称
- 好：`processing-pdfs`、`analyzing-spreadsheets`
- 避免：`helper`、`utils`、`tools`

---

## Skill 创建工作流

帮助用户创建 skill 时，遵循以下流程：

### 阶段 1：需求发现

收集以下信息：
1. skill 的目的与主要用例
2. 存放位置（个人 vs 项目）
3. 触发场景
4. 任何具体要求或约束
5. 可参照的既有示例或模式

以文本「单问题 + 1~3 选项」结构化收集；遵守 ask-via-clarify 提问规范（如已启用）。

### 阶段 2：设计

1. 拟定 skill 名称（小写、连字符、最长 64 字符）
2. 撰写具体、第三人称的 description
3. 列出需要的主要章节
4. 判断是否需要支持文件或脚本

### 阶段 3：实现

1. 创建目录结构
2. 编写带 frontmatter 的 SKILL.md 文件
3. 创建任何支持性参考文件
4. 按需创建工具脚本

### 阶段 4：验证

1. 确认 SKILL.md 在 500 行以内
2. 检查 description 具体且包含触发词
3. 确保全程术语一致
4. 确认所有文件引用都是一层深度
5. 测试 skill 可被发现并应用

---

## 完整示例

一个结构良好的 skill 完整示例：

**目录结构：**
```
code-review/
├── SKILL.md
├── STANDARDS.md
└── examples.md
```

**SKILL.md：**
```markdown
---
name: code-review
description: 按团队标准审查代码的质量、安全性与可维护性。当审查拉取请求、检查代码改动，或用户请求代码审查时使用。
---

# 代码审查

## 快速开始

审查代码时：

1. 检查正确性与潜在 bug
2. 核验安全最佳实践
3. 评估代码可读性与可维护性
4. 确认测试充分

## 审查检查清单

- [ ] 逻辑正确并处理边界情况
- [ ] 无安全漏洞（SQL 注入、XSS 等）
- [ ] 代码遵循项目风格约定
- [ ] 函数规模适中且职责聚焦
- [ ] 错误处理全面
- [ ] 测试覆盖了改动

## 反馈格式

按以下格式给出反馈：
- **严重**：合并前必须修复
- **建议**：考虑改进
- **可选**：可选增强

## 补充资源

- 详细编码标准见 [STANDARDS.md](STANDARDS.md)
- 审查示例见 [examples.md](examples.md)
```

---

## 汇总检查清单

定稿前逐项核验：

### 核心质量
- [ ] description 具体并包含关键术语
- [ ] description 同时包含"做什么"与"何时用"
- [ ] 以第三人称书写
- [ ] SKILL.md 正文在 500 行以内
- [ ] 全程术语一致
- [ ] 示例具体而非抽象

### 结构
- [ ] 文件引用保持一层深度
- [ ] 恰当使用渐进式披露
- [ ] 工作流步骤清晰
- [ ] 无时间敏感信息

### 若包含脚本
- [ ] 脚本真正解决问题而非甩锅
- [ ] 依赖包已文档化
- [ ] 错误处理显式且有用
- [ ] 无 Windows 风格路径
