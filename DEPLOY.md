# YG 部署指南（本 Fork）

本 fork 已配置好一切，两条部署路线任选。

## 路线 A：GitHub Actions 自动部署（推荐）

1. Cloudflare Dashboard → My Profile → API Tokens → Create Token → 编辑 `Workers Scripts` 模板，权限：
   - Account — Workers Scripts — Edit
   - Zone — Workers Routes — Edit（绑自定义域时需要）
2. 记下 Account ID（Workers 概览页右侧）。
3. 本仓库 Settings → Secrets and variables → Actions → New repository secret，添加两个：
   - `CF_API_TOKEN` = 上一步创建的 token
   - `CF_ACCOUNT_ID` = 账户 ID
4. 任意 push 到 main，或 Actions 页手动 Run workflow —— 自动 `wrangler deploy`。

订阅链接：`https://yg-sub-worker.<你的子域>.workers.dev/auto`

改配置：编辑 `wrangler.toml` 的 `[vars]` 后 push 即自动重部。

## 路线 B：Cloudflare Pages 连接 Git

1. Cloudflare Dashboard → Workers & Pages → Create → Pages → Connect to Git → 选本仓库。
2. 构建命令留空，输出目录 `/`。
3. Settings → Environment variables 添加（与 wrangler.toml 的 vars 相同）：
   `TOKEN=auto`、`UUID=c22d869c-3a2d-484f-9546-8126c5bd3144`、`ADDAPI`、`ADDCSV`、`SUBAPI` 等。
4. 绑定自定义域（推荐 `sub.你的域名` CNAME 到 `<项目名>.pages.dev`）。

## 关键提醒（血泪教训）

- **不要部署 VLESS 隧道服务端到 workers.dev / pages.dev 裸域名**：Cloudflare 滥用系统会识别 `?ed=2560` WS 特征并封号（本机 2026-10-30 实测，error 8000119）。
- 订阅生成器（本仓库）本身安全——它不含隧道逻辑。
- 验证服务端连通请用真实客户端，不要裸发 WS 探测请求。
- VLESS 服务端必须绑自定义域名（CF DNS 代理模式）。

## 变量说明

完整变量表见上游 README.md。核心：
- `TOKEN`：订阅路径（默认 auto → `/auto`）
- `HOST` / `UUID` / `PATH`：内置节点的服务端信息
- `ADDAPI` / `ADDCSV`：优选 IP 源
- `SUBAPI` / `SUBCONFIG`：clash/singbox 订阅转换后端
