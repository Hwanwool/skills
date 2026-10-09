---
name: create-plugin
description: 创建 omp 扩展包（extension）：TypeScript/JavaScript 运行时工厂 + 可捆绑的 skills、commands、rules、prompts、hooks、自定义 tools 与 MCP 配置。当用户想要为 omp 创建新扩展、打包能力或分发扩展时使用。
---

# 创建扩展（Create Plugin）

## 是什么

omp 扩展 = TS/JS 模块（运行时工厂）+ 可捆绑的声明式能力。需要可执行行为（自定义工具、斜杠命令、工具策略、UI、provider 集成）时用扩展；只需要 `SKILL.md` 时直接创建 skill，不必做扩展。

## 最小结构

```
hello-extension/
├── package.json      # 含 "omp": { "extensions": ["./src/index.ts"] }
└── src/
    └── index.ts      # 默认导出 (pi: ExtensionAPI) => void
```

package.json：

```json
{
  "name": "hello-extension",
  "version": "1.0.0",
  "type": "module",
  "files": ["src"],
  "omp": { "extensions": ["./src/index.ts"] }
}
```

index.ts 核心 API（`import type { ExtensionAPI } from "@oh-my-pi/pi-coding-agent"`）：

- `pi.registerCommand("名字", { description, handler })` — 用户斜杠命令
- `pi.registerTool({ name, description, parameters: pi.zod.object({...}), approval: "read"|"exec", execute })` — 模型可调用工具
- `pi.on("session_start", (event, ctx) => ...)` — 会话事件钩子
- 运行时动作（如 `pi.sendMessage()`）只能在事件/命令/工具处理器里调用，不能在工厂加载期调用

## 可捆绑的声明式能力

扩展包根目录可附带以下约定目录（无需在 package.json 声明字段）：

| 路径 | omp 发现的内容 |
|---|---|
| `skills/<name>/SKILL.md` | 按需 skill |
| `commands/*.md` | 用户斜杠命令 |
| `rules/*.{md,mdc}` | 项目/工作流规则 |
| `prompts/*.md` | 提示模板 |
| `hooks/pre/*`、`hooks/post/*` | 工具前后钩子 |
| `tools/*` | 自定义工具（脚本/Markdown/JSON/TS/JS 格式） |
| `.mcp.json` 或 `mcp.json` | MCP 服务器定义 |

## 创建与加载流程

1. 建目录 + `package.json`（写 `omp.extensions`）+ `src/index.ts`（默认导出工厂）
2. 开发验证：`omp --extension ./hello-extension`（单会话加载，`-e` 为简写）
3. 本地安装：`omp plugin link ./hello-extension`（symlink，改动下次启动生效）；用 `omp plugin list` 确认链接、`omp plugin doctor` 体检
4. 不想安装时，把绝对路径写进 `~/.omp/agent/config.yml` 的 `extensions:` 列表
5. 打包分发：`npm pack --dry-run` 检查产物 → `npm publish`

## 安全与注意事项

- 扩展与 omp 同进程、同权限，不做沙箱——保持包可审计、少依赖、向用户说明外部访问需求
- 分发包时 `files` 必须包含工厂代码与所有捆绑能力目录
- 本地路径安装是 symlink，编辑在下次启动生效，不热重载；改代码后需退出并重启会话
- 扩展工厂在会话启动时初始化
