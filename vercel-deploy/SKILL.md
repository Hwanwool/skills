---
name: vercel-deploy
description: 将 Web 项目部署到 Vercel，自动化 CLI 安装、OAuth 登录、构建与生产部署。当用户要求部署到 Vercel 时使用。
---

# Vercel 部署（Vercel Deploy）

## 触发场景

当用户提出以下需求时启用本技能：

- "部署到 Vercel"、"上线 Vercel"、"发布到 Vercel"
- 需要把 Web 项目发布到生产环境并获取可访问 URL
- 需要配置预览部署、环境变量或自定义域名

## 前置检查

部署前逐项确认：

1. **构建脚本**：`package.json` 中已配置 `build` 脚本
2. **Node 环境**：本地 Node 版本可用（`node --version`）
3. **项目类型**：确认框架类型（Next.js / Vite / CRA / 静态站点等），Vercel 会自动检测
4. **环境变量**：整理运行期所需的全部环境变量（密钥类走 Vercel 密钥存储，不写入代码）

## 执行流程

### 步骤 1：CLI 安装/验证

```bash
# 未安装时全局安装
npm install -g vercel

# 验证可用
vercel --version
```

也可不经安装直接用 `npx vercel`（按需拉取）。

### 步骤 2：OAuth 登录

```bash
vercel login
```

该命令打开浏览器完成 OAuth 授权，成功后 CLI 获得部署凭据。

非交互环境（CI/无头）改用令牌：设置 `VERCEL_TOKEN` 环境变量后 `vercel login` 可跳过，直接进入部署。

### 步骤 3：项目配置

首次在项目目录执行 `vercel` 时，按交互提示确认：

- **框架检测**：通常自动识别，无需改动
- **根目录/构建命令/输出目录**：按项目实际结构确认（构建产物目录如 `.next`、`dist`、`build`）
- 配置会写入项目 `.vercel/project.json`（orgId/projectId 关联）

环境变量通过命令配置：

```bash
vercel env add API_KEY
vercel env add API_KEY production
```

### 步骤 4：部署

```bash
# 预览部署（默认，每次部署生成唯一预览 URL）
vercel

# 生产部署（发布到正式域名）
vercel --prod
```

### 步骤 5：验证

- [ ] 部署命令输出中包含部署 URL（如 `https://<project>.vercel.app`）
- [ ] 打开生产 URL 确认页面正常、HTTPS 有效
- [ ] 检查构建日志无报错（`vercel logs <deployment-url>`）
- [ ] 环境变量在目标环境生效

## 注意事项

- **SPA 路由**：单页应用需要在根目录配置 `vercel.json` 的重写规则（`rewrites` 指向入口文件），避免子路由 404
- **回滚**：部署出错时可用 `vercel promote <deployment-id>` 或 `vercel rollback` 回退到上一个正常版本
- **密钥安全**：任何密钥一律通过 `vercel env` 配置，绝不写入仓库或构建产物
- **无头环境**：优先使用 `VERCEL_TOKEN`，不要在 CI 日志中暴露令牌
