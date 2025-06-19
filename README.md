# 🚀 全栈 Cloudflare SaaS 工具包

**_轻松在 Cloudflare 上构建和部署可扩展的产品。_**

Cloudflare SaaS Starter 用于在 Cloudflare 上快速构建和部署 SaaS 产品。这是一个使用 [`c3`](https://developers.cloudflare.com/pages/get-started/c3) 引导的 [Next.js](https://nextjs.org/) 项目。

这是用于构建 [Supermemory.ai](https://Supermemory.ai) 的同一套技术栈，该项目在 [git.new/memory](https://git.new/memory) 上开源。

Supermemory 现在拥有 20k+ 用户，每月仅花费 5 美元运行。可以说，它非常有效。

## 技术栈包括：

- [Next.js](https://nextjs.org/) 前端框架
- [TailwindCSS](https://tailwindcss.com/) 样式框架
- [Drizzle ORM](https://orm.drizzle.team/) 数据库访问
- [NextAuth](https://next-auth.js.org/) 身份验证
- [Cloudflare D1](https://www.cloudflare.com/developer-platform/d1/) 无服务器数据库
- [Cloudflare Pages](https://pages.cloudflare.com/) 托管服务
- [ShadcnUI](https://shadcn.com/) 组件库

## 开始使用

1. 确保您已安装 [Wrangler](https://developers.cloudflare.com/workers/wrangler/install-and-update/#installupdate-wrangler)，并且已使用 `wrangler login` 登录（您需要一个 Cloudflare 账户）

2. 克隆仓库并安装依赖：
   ```bash
   git clone https://github.com/sancijun/cloudflare-saas-starter.git
   cd cloudflare-saas-starter
   npm i -g bun
   bun install
   bun run setup
   ```

3. 运行开发服务器：
   ```bash
   bun run dev
   ```

在浏览器中打开 [http://localhost:3000](http://localhost:3000) 查看结果。

## Cloudflare 集成

除了 `dev` 脚本外，`c3` 还为 Cloudflare Pages 集成添加了额外的脚本：
- `pages:build`：使用 [`@cloudflare/next-on-pages`](https://github.com/cloudflare/next-on-pages) CLI 为 Pages 构建应用程序
- `preview`：使用 [Wrangler](https://developers.cloudflare.com/workers/wrangler/) CLI 本地预览您的 Pages 应用程序
- `deploy`：使用 Wrangler CLI 部署您的 Pages 应用程序
- `cf-typegen`：为 Cloudflare env 生成 TypeScript 类型

> __注意：__ 虽然 `dev` 脚本对本地开发最为理想，但您应该定期预览您的 Pages 应用程序，以确保它在 Pages 环境中正常工作。

## 绑定

Cloudflare [绑定](https://developers.cloudflare.com/pages/functions/bindings/) 允许您与 Cloudflare 平台资源交互。您可以在开发期间、本地预览和已部署的应用程序中使用绑定。

有关设置绑定的详细说明，请参阅 Cloudflare 文档。

## 数据库迁移
D1 设置的快速说明：
- D1 是遵循 SQLite 约定的无服务器数据库。
- 在 Cloudflare pages 和 workers 中，您可以通过绑定暴露的[客户端 API](https://developers.cloudflare.com/d1/build-with-d1/d1-client-api/)直接查询 d1（例如 `env.BINDING`）
- 您也可以通过 [REST API](https://developers.cloudflare.com/api/operations/cloudflare-d1-create-database) 查询 d1
- 在本地，wrangler 在 `bun run dev` 后会在 `.wrangler/state/v3/d1` 自动生成 sqlite 文件。
- 本地开发环境（`bun run dev`）与[本地 d1 会话](https://developers.cloudflare.com/d1/build-with-d1/local-development/#start-a-local-development-session)交互，该会话基于位于 `.wrangler/state/v3/d1` 的一些 SQLite 文件。
- 在开发模式下（`bun run db:<migrate or studio>:dev`），Drizzle-kit（migrate 和 studio）直接修改这些文件作为常规 SQLite 数据库。而 `bun run db:<migrate or studio>:prod` 使用 d1-http 驱动程序通过 REST API 与远程 d1 交互。因此我们需要在 `.env.example` 中设置环境变量

生成迁移文件：
- `bun run db:generate`

应用数据库迁移：
- 开发环境：`bun run db:migrate:dev`
- 生产环境：`bun run db:migrate:prd`

检查数据库：
- 本地数据库：`bun run db:studio:dev`
- 远程数据库：`bun run db:studio:prod`

## Cloudflare R2 存储桶 CORS / 文件上传

不要忘记向 R2 存储桶添加 CORS 策略。CORS 策略应该如下所示：

```json
[
  {
    "AllowedOrigins": [
      "http://localhost:3000",
      "https://your-domain.com"
    ],
    "AllowedMethods": [
      "GET",
      "PUT"
    ],
    "AllowedHeaders": [
      "Content-Type"
    ],
    "ExposeHeaders": [
      "ETag"
    ]
  }
]
```

您现在甚至可以设置对象上传。

## 手动设置

如果您更喜欢手动设置：

1. 创建 Cloudflare 账户并安装 Wrangler CLI。
2. 创建 D1 数据库：`bunx wrangler d1 create ${dbName}`
3. 在项目根目录创建 `.dev.vars` 文件，包含您的 Google OAuth 凭据和 NextAuth 密钥。
   1. `AUTH_SECRET`，通过命令 `openssl rand -base64 32` 或 `bunx auth secret` 生成
   2. `AUTH_GOOGLE_ID` 和 `AUTH_GOOGLE_SECRET` 用于 Google OAuth。
      1. 首先创建 [OAuth 同意屏幕](https://console.cloud.google.com/apis/credentials/consent)。提示：如果跳过徽标上传则无需等待时间。
      2. 创建[凭据](https://console.cloud.google.com/apis/credentials)。在"授权的 JavaScript 来源"中放入 `https://your-domain` 和 `http://localhost:3000`。在"授权的重定向 URI"中放入 `https://your-domain/api/auth/callback/google` 和 `http://localhost:3000/api/auth/callback/google`。
4. 生成数据库迁移文件：`bun run db:generate`
5. 运行本地迁移：`bunx wrangler d1 execute ${dbName} --local --file=migrations/0000_setup.sql` 或使用 drizzle `bun run db:migrate:dev`
6. 运行远程迁移：`bunx wrangler d1 execute ${dbName} --remote --file=migrations/0000_setup.sql` 或使用 drizzle `bun run db:migrate:prod`
7. 启动开发服务器：`bun run dev`
8. 部署：`bun run deploy`

## 这套技术栈的优势

- 完全可扩展和可组合
- 无需环境变量（使用 `env.DB`、`env.KV`、`env.Queue`、`env.AI` 等）
- 强大的工具，如 Wrangler 用于数据库管理和迁移
- 成本效益高的扩展（例如，每月 5 美元运行多个高流量项目）

只需在项目设置中更改您的 Cloudflare 账户 ID，您就可以开始了！

