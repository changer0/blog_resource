---
title: DevSpace_ChatGPT_MCP_接入指南_脱敏版
date: 2026-07-08 09:05:05
---

# DevSpace 接入 ChatGPT：本地项目 MCP 连接指南

> 适用目标：将**指定的本地代码目录**通过 DevSpace 暴露为 MCP 服务，并连接到 ChatGPT，使其能够在明确授权范围内读取文件、检索代码、执行终端命令与修改代码。
>
> 本文已移除实际项目名、本机用户名、真实目录、真实公网地址与密码，统一使用占位符表示。

---

## 1. 接入架构与边界

```text
ChatGPT 自定义 MCP 应用
        │ HTTPS（公网）
        ▼
Cloudflare Tunnel / 其他 HTTPS 隧道
        │ 转发至本机 127.0.0.1:<LOCAL_PORT>
        ▼
DevSpace MCP Server
        │ 仅允许访问 configured roots
        ▼
<PROJECT_ROOT>
```

DevSpace 是运行在开发机上的本地 MCP 服务。它允许 MCP 客户端在**已授权的项目根目录**内进行文件读写、代码搜索和 Shell 命令执行。

因此，应将它视为“受限的远程开发权限”，而不是普通只读文件共享：

- 仅授权一个具体项目或较小的工作目录；
- 不要授权整个用户目录，例如 `~`、`/Users/<USER>`、`C:\Users\<USER>`；
- Owner Password、隧道地址和本机配置文件均应视为敏感信息；
- ChatGPT 的“只读检查”提示只是协作约束，不是底层权限隔离。底层安全边界仍取决于 DevSpace 配置的允许目录。

---

## 2. 占位符说明

| 占位符 | 含义 | 示例形式 |
|---|---|---|
| `<PROJECT_ROOT>` | 允许 ChatGPT 访问的项目绝对路径 | `/Users/<USER>/work/<PROJECT_NAME>` |
| `<LOCAL_PORT>` | DevSpace 本机监听端口 | `7676` |
| `<PUBLIC_ORIGIN>` | HTTPS 隧道的根地址，不含 `/mcp` | `https://<RANDOM>.trycloudflare.com` |
| `<PUBLIC_MCP_URL>` | 提供给 ChatGPT 的 MCP 地址 | `https://<RANDOM>.trycloudflare.com/mcp` |
| `<OWNER_PASSWORD>` | DevSpace 连接确认密码 | 保存在 `~/.devspace/auth.json` |

---

## 3. 前置条件

### 3.1 必备软件

DevSpace 需要：

- Node.js：`>=22.19` 且 `<27`
- npm
- Git
- Bash（macOS/Linux 自带；Windows 需 Git Bash、WSL、MSYS2 或 Cygwin Bash）
- 一个将本机服务暴露为公网 HTTPS 的隧道或反向代理

在 macOS 终端检查：

```bash
node -v
npm -v
git --version
bash --version
```

若 Node.js 版本不满足要求，可使用 Homebrew 安装 Node 22：

```bash
brew install node@22
```

Apple Silicon Mac 通常需要将 Node 22 加入 PATH：

```bash
echo 'export PATH="/opt/homebrew/opt/node@22/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
node -v
```

> Intel Mac 常见 Homebrew 路径为 `/usr/local`，请按实际环境调整。

### 3.2 安装 Cloudflare Tunnel 客户端

本文采用 Cloudflare Quick Tunnel 做首次验证：

```bash
brew install cloudflared
cloudflared --version
```

---

## 4. 第一次接入：按顺序执行

### 步骤 1：启动 Cloudflare Quick Tunnel

在**独立终端窗口 A**执行：

```bash
cloudflared tunnel --url http://127.0.0.1:<LOCAL_PORT>
```

默认端口为 `7676`，因此首次使用通常为：

```bash
cloudflared tunnel --url http://127.0.0.1:7676
```

终端会输出类似内容：

```text
Your quick Tunnel has been created! Visit it at:
https://<RANDOM>.trycloudflare.com
```

记录该根地址：

```text
<PUBLIC_ORIGIN> = https://<RANDOM>.trycloudflare.com
```

注意：

- 保持终端 A 持续运行；关闭它，公网地址会失效。
- Quick Tunnel 每次重启可能获得不同地址。
- 日志中关于未找到 `config.yml` / `config.yaml` 的提示，对 Quick Tunnel 通常不影响使用。

---

### 步骤 2：初始化 DevSpace

在**另一个终端窗口 B**执行：

```bash
npx @waishnav/devspace init
```

初始化会依次要求填写以下信息。

#### 2.1 项目根目录

填写要授权访问的**绝对路径**：

```text
<PROJECT_ROOT>
```

示例：

```text
/Users/<USER>/workspace/<PROJECT_NAME>
```

推荐仅填写当前项目目录，而不是上级工作目录，更不要填写整个用户目录。

#### 2.2 本地端口

建议首次保留默认值：

```text
7676
```

#### 2.3 公网根地址

填写 Quick Tunnel 输出的根地址：

```text
<PUBLIC_ORIGIN>
```

**这里不要追加 `/mcp`。**

正确示例：

```text
https://<RANDOM>.trycloudflare.com
```

错误示例：

```text
https://<RANDOM>.trycloudflare.com/mcp
```

初始化完成后，DevSpace 会创建：

```text
~/.devspace/config.json
~/.devspace/auth.json
```

其中 `auth.json` 存放 Owner Password。不要将密码发送到聊天、截图、日志或提交到 Git 仓库。

---

### 步骤 3：启动 DevSpace MCP 服务

继续在终端 B 执行：

```bash
npx @waishnav/devspace serve
```

正常启动的关键信息大致如下：

```text
devspace listening on http://127.0.0.1:<LOCAL_PORT>/mcp
public base url: <PUBLIC_ORIGIN>
allowed roots: <PROJECT_ROOT>
auth: Owner password approval required
```

不要关闭终端 B。

此时本机地址为：

```text
http://127.0.0.1:<LOCAL_PORT>/mcp
```

提供给 ChatGPT 的公网 MCP 地址为：

```text
<PUBLIC_MCP_URL>
```

即：

```text
<PUBLIC_ORIGIN>/mcp
```

---

### 步骤 4：运行本机健康检查

在**终端窗口 C**执行：

```bash
npx @waishnav/devspace doctor
```

重点确认：

- Node 版本符合要求；
- Git 与 Bash 已被识别；
- 本机端口、`publicBaseUrl` 和 allowed roots 正确；
- SQLite 原生依赖状态正常。

如出现 `better-sqlite3` 无法加载的问题，常见原因是切换过 Node 版本。可尝试：

```bash
npm rebuild better-sqlite3
npx @waishnav/devspace doctor
```

---

## 5. 在 ChatGPT 中创建自定义 MCP 应用

在 ChatGPT 网页端进入：

```text
设置 → 应用与连接器（Apps & Connectors）→ 新应用 / 创建自定义 MCP
```

按页面填写：

| 字段 | 建议填写 |
|---|---|
| 名称 | `<PROJECT_NAME>` 或 `DevSpace Local` |
| 描述 | `访问本机指定代码目录的 DevSpace MCP 服务` |
| 连接方式 | 服务器 URL |
| 服务器 URL | `<PUBLIC_MCP_URL>` |
| 身份验证 | 按页面默认 OAuth 配置保留即可；DevSpace 后续会进行 Owner Password 确认 |

服务器 URL **必须含 `/mcp`**：

```text
https://<RANDOM>.trycloudflare.com/mcp
```

完成风险确认后创建连接器。首次连接时，根据页面提示输入保存在以下文件中的 Owner Password：

```bash
cat ~/.devspace/auth.json
```

> 不要复制该命令输出到聊天窗口；仅在本机授权页面中输入密码。

---

## 6. 首次打开工作区

连接成功后，模型可能不知道实际允许路径，或错误猜测目录位置。不要只说“打开 `<PROJECT_NAME>`”，而应明确要求它调用 `open_workspace`，并给出**完整绝对路径**。

推荐首条指令：

```text
请调用 DevSpace 的 open_workspace 工具，打开以下绝对路径的项目：

<PROJECT_ROOT>

请先只读检查目录结构、AGENTS.md、CLAUDE.md、Git 状态和最近提交；
暂时不要修改文件，也不要执行会修改工作区的命令。
```

如果出现类似错误：

```text
当前工程目录未映射到可访问工作区
workspace path rejected
```

通常表示模型使用了错误路径，而非连接失败。处理方式：

1. 在本机查看 DevSpace 配置：

   ```bash
   npx @waishnav/devspace config get
   ```

2. 确认 `<PROJECT_ROOT>` 是否处于 `allowed roots` 中；
3. 在 ChatGPT 中再次发送真实绝对路径；
4. 让模型通过 `open_workspace` 打开该路径。

---

## 7. 验收建议

连接成功后，可按以下顺序验证。

### 7.1 只读验证

```text
请打开 <PROJECT_ROOT>，只读输出：
1. 顶层目录结构；
2. AGENTS.md、CLAUDE.md 等工程约束文件是否存在；
3. 当前 Git 分支与工作区状态；
4. 最近 5 条提交摘要。
不要修改文件。
```

### 7.2 最小写入验证（可选）

确认只读流程无误后，创建一个无业务影响的临时文件并要求模型展示差异；验证完成后立即删除。不要将首次写入测试放在生产分支或关键仓库中。

### 7.3 检查运行日志

在 DevSpace 服务终端观察连接、授权、工作区打开与工具调用日志。异常时保留报错文本，但应打码以下内容：

- Owner Password；
- 访问令牌；
- 内网 IP、域名及敏感文件路径；
- 业务代码或配置中的密钥。

---

## 8. 常见问题与处理

### 8.1 `cloudflared: command not found`

安装 Cloudflare Tunnel：

```bash
brew install cloudflared
```

然后确认：

```bash
cloudflared --version
```

---

### 8.2 `devspace: command not found`

无需全局安装，统一使用 `npx`：

```bash
npx @waishnav/devspace init
npx @waishnav/devspace serve
```

也可全局安装：

```bash
npm install -g @waishnav/devspace
```

全局安装后若仍找不到命令，检查 npm 全局 bin 路径是否在 `PATH` 中。

---

### 8.3 Node.js 版本不支持

检查：

```bash
node -v
```

DevSpace 要求：

```text
>=22.19 且 <27
```

建议使用 Node 22 LTS，并在切换 Node 版本后重新执行 `doctor`。

---

### 8.4 ChatGPT 连接不上

依次确认：

1. Cloudflare Tunnel 终端仍在运行；
2. DevSpace 服务终端仍在运行；
3. DevSpace 监听的是 `127.0.0.1:<LOCAL_PORT>`；
4. 隧道转发目标也是 `http://127.0.0.1:<LOCAL_PORT>`；
5. ChatGPT 中填写的是 `<PUBLIC_ORIGIN>/mcp`；
6. 初始化配置中填写的是 `<PUBLIC_ORIGIN>`，而不是带 `/mcp` 的地址；
7. 运行：

   ```bash
   npx @waishnav/devspace doctor
   ```

---

### 8.5 Quick Tunnel 重启后地址变化

Quick Tunnel 的随机地址变化后，需要同时更新两处：

1. DevSpace 的公网根地址；
2. ChatGPT 自定义 MCP 应用的服务器 URL。

临时覆盖本次启动地址：

```bash
DEVSPACE_PUBLIC_BASE_URL="https://<NEW_RANDOM>.trycloudflare.com" \
npx @waishnav/devspace serve
```

或持久化更新：

```bash
npx @waishnav/devspace config set publicBaseUrl https://<NEW_RANDOM>.trycloudflare.com
```

随后将 ChatGPT 中的服务器 URL 改为：

```text
https://<NEW_RANDOM>.trycloudflare.com/mcp
```

长期使用建议改为 **Cloudflare Named Tunnel + 自有固定子域名**，避免每次重启修改配置。

---

### 8.6 403、Host Header 或允许主机错误

运行：

```bash
npx @waishnav/devspace doctor
```

检查 configured public URL 的主机名是否出现在 allowed hosts 中。若 Quick Tunnel 地址已变更，应先更新 `publicBaseUrl` 再重启服务。

不要将 `DEVSPACE_ALLOWED_HOSTS="*"` 作为长期方案；它只应在受控的本机调试场景中临时使用。

---

### 8.7 Owner Password 无法通过

确认使用的是当前配置对应的密码：

```bash
cat ~/.devspace/auth.json
```

若密码意外泄露、遗失或需要旋转，重新初始化：

```bash
npx @waishnav/devspace init --force
```

重置后需要重新确认项目根目录、端口与公网根地址。新的密码不要通过聊天工具传递。

---

## 9. 日常使用顺序

每次需要连接本机项目时，按以下顺序启动：

```text
1. 启动 HTTPS 隧道
2. 启动 DevSpace：npx @waishnav/devspace serve
3. 在 ChatGPT 中启用该自定义 MCP 应用
4. 输入 Owner Password 完成授权（如页面要求）
5. 让模型通过 open_workspace 打开 <PROJECT_ROOT>
```

临时使用完成后，按以下顺序停止：

```text
1. 在 DevSpace 终端按 Ctrl + C
2. 在 cloudflared 终端按 Ctrl + C
3. 在 ChatGPT 中停用该应用，或保留但不要在不使用时授权
```

---

## 10. 最终检查清单

- [ ] Node.js 版本在 `>=22.19` 且 `<27` 范围内
- [ ] npm、Git、Bash 可用
- [ ] `cloudflared` 已安装
- [ ] Quick Tunnel 或固定 HTTPS 隧道正在运行
- [ ] DevSpace 服务正在监听本机 `<LOCAL_PORT>`
- [ ] `allowed roots` 仅包含必要的 `<PROJECT_ROOT>`
- [ ] ChatGPT 配置的是 `<PUBLIC_ORIGIN>/mcp`
- [ ] Owner Password 未泄露、未写入项目文件、未提交 Git
- [ ] 模型使用 `open_workspace` 打开的是准确绝对路径
- [ ] 首次接入先完成只读验证，再逐步开放写入性任务

---

## 11. 参考命令速查

```bash
# 检查环境
node -v
npm -v
git --version
cloudflared --version

# 安装 cloudflared（macOS）
brew install cloudflared

# 启动临时公网 HTTPS 隧道
cloudflared tunnel --url http://127.0.0.1:7676

# 初始化 DevSpace
npx @waishnav/devspace init

# 启动 DevSpace
npx @waishnav/devspace serve

# 健康检查
npx @waishnav/devspace doctor

# 查看当前配置
npx @waishnav/devspace config get

# 更新公网根地址
npx @waishnav/devspace config set publicBaseUrl https://<PUBLIC_HOST>

# 重新初始化并旋转 Owner Password
npx @waishnav/devspace init --force
```

---

> 版本说明：本文按 `@waishnav/devspace` 的公开安装与配置流程编写。实际界面名称、MCP 认证流程和 CLI 参数可能随 DevSpace、ChatGPT 或 Cloudflare 的版本变化而调整；接入异常时优先执行 `npx @waishnav/devspace doctor` 并核对当前官方文档。
