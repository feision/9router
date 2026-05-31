# feision/9router — 9Router Fork

基于 [decolua/9router](https://github.com/decolua/9router)，以下为定制修改记录。

---

## 定制修改

### User-Agent 伪装

**修改文件**：`open-sse/utils/proxyFetch.js`

在全局 fetch wrapper `patchedFetch` 中注入 Python UA，覆盖所有 provider 的原始 UA：

```javascript
async function patchedFetch(url, options = {}) {
  const headers = { ...options.headers };
  headers["User-Agent"] = "python-requests/2.31.0";
  return proxyAwareFetch(url, { ...options, headers }, null);
}
```

**为什么改这里**：
- `BaseExecutor.buildHeaders()` 被所有 executor 重写，无效
- `proxyAwareFetch` 里改也可能被 Docker 缓存绕过
- `patchedFetch` 是真正的 `globalThis.fetch` 拦截点，所有出口请求必经之路

**保留原始 UA 的 provider**（不能统一覆盖）：
- Gemini CLI：`GeminiCLI/0.34.0/model (linux; x64)`
- Antigravity：`antigravity/1.104.0 linux/x64`
- GitHub Copilot：`GitHubCopilotChat/0.38.0`
- Kiro：`AWS-SDK-JS/3.0.0 kiro-ide/1.0.0`
- iFlow：`iFlow-Cli`
- Claude CLI：`claude-cli/2.1.92 (external, sdk-cli)`

---

## 自动构建

GitHub Actions 自动 build + 推送 Docker 镜像：

- **镜像**：`feision/9router:latest`（Docker Hub）+ `ghcr.io/feision/9router`（GitHub Container Registry）
- **触发**：push 到 `master` 分支自动构建
- **平台**：仅 `linux/amd64`（约 5-8 分钟）
- **Secrets**：`DOCKERHUB_USERNAME`、`DOCKERHUB_TOKEN`

---

## VPS 部署

```bash
# 在 Japan VPS (43.167.159.200) 上执行
docker pull feision/9router:latest
docker stop 9router && docker rm 9router
docker run -d --name 9router --restart unless-stopped \
  -p 20128:20128 \
  -v /home/ubuntu/.9router:/app/data \
  feision/9router:latest
```

或用脚本：
```bash
bash /tmp/update-9router.sh
```

---

## 上游同步

上游 `decolua/9router` 发新版后：

1. GitHub 页面 → "Sync fork" 合并上游代码
2. GitHub Actions 自动 build + push
3. VPS 执行部署命令

---

## 本地开发调试

```bash
# 克隆 fork
git clone https://github.com/feision/9router.git
cd 9router

# 安装依赖
npm install

# 开发模式
npm run dev
```

---

## 注意事项

- 本地 npm 安装版（`npm i -g 9router`）代码结构与 Docker 镜像不同，是 webpack 打包的 bundle，调试需用 Docker 版
- GitNexus 可用于代码分析：`npm install -g gitnexus && gitnexus analyze`
