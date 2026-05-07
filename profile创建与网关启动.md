
从零开始创建新 Profile、配置模型并启动网关的完整操作流程。

<!-- 轴二：使用方法 -->

## 前提条件

已通过一行命令完成安装：

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
source ~/.bashrc
```

安装器自动处理所有依赖（Python 3.11、Node.js v22、ripgrep、ffmpeg），仅需确保 **Git** 可用。

---

## 第一步：创建新 Profile

```bash
hermes profile create mybot
```

### 三种创建模式

| 模式             | 命令                                                       | 说明                                        |
| -------------- | -------------------------------------------------------- | ----------------------------------------- |
| 空白 Profile     | `hermes profile create mybot`                            | 全新配置，需从头设置一切                              |
| 克隆配置           | `hermes profile create mybot --clone`                    | 复制 `config.yaml`、`.env`、`SOUL.md`，会话和记忆全新 |
| 全量克隆           | `hermes profile create mybot --clone-all`                | 复制一切——配置、记忆、会话历史、技能、Cron、插件               |
| 从指定 Profile 克隆 | `hermes profile create mybot --clone --clone-from coder` | 从指定的已有 Profile 克隆配置                       |

创建后自动生成独立命令别名，此后 `mybot chat` 等价于 `hermes -p mybot chat`。所有 `hermes` 子命令均可通过别名使用：

```bash
mybot chat                    # 对话
mybot setup                   # 配置向导
mybot gateway start           # 启动网关
mybot doctor                  # 健康检查
mybot skills list             # 技能列表
mybot config set model.default anthropic/claude-sonnet-4
```

也可以随时用 `-p` 标志显式指定 Profile：

```bash
hermes -p mybot chat
hermes --profile=mybot chat
```

设置默认 Profile：

```bash
hermes profile use mybot      # 此后直接 hermes chat 即指向 mybot
hermes profile use default    # 切回默认
```

---

## 第二步：配置模型

### 方式 A：交互式向导（推荐）

```bash
mybot setup
```

启动配置向导，依次完成：选择 LLM 供应商 → 选择模型 → 输入 API Key。

### 方式 B：交互式选模型

```bash
mybot model
```

从菜单选择供应商并配置。支持的供应商包括：

| 供应商 | 配置方式 |
|---|---|
| Nous Portal | OAuth 登录 |
| OpenAI Codex | 设备代码认证 |
| Anthropic | OAuth（Max 计划）或 API Key |
| OpenRouter | API Key |
| DeepSeek | `DEEPSEEK_API_KEY` |
| Kimi / Moonshot | `KIMI_API_KEY` |
| MiniMax | `MINIMAX_API_KEY` 或 OAuth |
| 自定义端点 | Base URL + API Key（VLLM/SGLang/Ollama） |

**最低上下文要求 64K tokens**。低于此值会被拒绝启动。

### 方式 C：命令行直接设置

```bash
# 以 OpenRouter 为例
mybot config set OPENROUTER_API_KEY sk-or-v1-your-key-here
mybot config set model.default anthropic/claude-sonnet-4
```

### 设置存储位置

- 密钥和 Token → `~/.hermes/profiles/mybot/.env`
- 非密钥设置 → `~/.hermes/profiles/mybot/config.yaml`

使用 `hermes config set` 命令会自动将值写入正确的文件。

---

## 第三步：验证 CLI 对话正常

```bash
mybot chat -q "Hello, confirm you can see my current directory"
```

**验证成功的标准**：
- 横幅显示所选模型/供应商
- Agent 正常回复，无报错
- 能使用工具（terminal、file read、web search 等）
- 多轮对话正常

**铁律**：如果基础对话都不正常，不要添加任何额外功能。先修好基础。

### 设置工作目录

```bash
mybot config set terminal.cwd /absolute/path/to/project
```

> **注意**：`terminal.cwd: "."` 表示从 Hermes 被启动的目录开始，**不是** profile 目录。如果需要固定的项目目录，必须设为绝对路径。

---

## 第四步：配置消息平台并启动网关

### 4a. 配置平台

```bash
mybot gateway setup
```

交互式引导完成：
1. 选择平台（Telegram / Discord / Slack / WhatsApp / Signal 等）
2. 输入 Bot Token
3. 设置 Home Channel
4. 配置用户授权（allowlist 或 DM pairing）

以 Telegram 为例的 Platform 能力矩阵：

| 功能 | 支持 |
|---|---|
| 语音 | ✅ |
| 图片 | ✅ |
| 文件 | ✅ |
| 线程/主题 | ✅ |
| 输入中指示 | ✅ |
| 流式输出 | ✅ |
| 回应 | — |

### 4b. 手动编辑配置

```bash
# 编辑 Token
nano ~/.hermes/profiles/mybot/.env
# TELEGRAM_BOT_TOKEN=123456:ABC...
```

```bash
# 编辑平台配置
nano ~/.hermes/profiles/mybot/config.yaml
```

```yaml
gateway:
  platforms:
    telegram:
      enabled: true
      extra:
        token: "123456:ABC..."
```

### 4c. 启动网关

```bash
mybot gateway start
```

网关是一个长期运行的后台进程，处理消息、管理会话、执行 Cron 任务。

### 4d. 设为系统服务（开机自启）

```bash
mybot gateway install         # Linux: systemd, macOS: launchd
```

每个 Profile 有独立的服务名（如 `hermes-gateway-mybot`），互不干扰。

---

## 第五步：验证网关

```bash
mybot gateway status
```

在配置的消息平台向 Bot 发送消息，确认正常回复。

---

## 常用管理命令

| 命令 | 用途 |
|---|---|
| `mybot config show` | 查看当前配置 |
| `mybot doctor` | 诊断配置问题 |
| `mybot gateway stop` | 停止网关 |
| `mybot gateway restart` | 重启网关 |
| `mybot gateway status` | 查看网关状态 |
| `mybot profile show` | 查看 Profile 详情 |
| `mybot profile list` | 列出所有 Profile |
| `mybot sessions list` | 查看会话列表 |
| `hermes profile rename mybot newbot` | 重命名 Profile |
| `hermes profile delete mybot` | 删除 Profile（停止网关+移除服务） |

---

## 关键注意事项

- 每个 Profile 有独立的 `HERMES_HOME`（默认 `~/.hermes/profiles/mybot/`），配置、会话、记忆完全隔离
- Profile 不提供文件系统沙箱——Agent 仍拥有用户级别的文件访问权限
- 两个 Profile 不能使用同一 Bot Token（Token 锁会阻止为第二个网关启动）
- Gateway 启动后在后台持续运行，处理来自所有已配置平台的消息
- 更新 `hermes update` 会同步新技能到所有 Profile，用户修改的技能从不覆盖

<!-- 跨页关联 -->

## 与其他概念的关系

- [[Profiles]]：多 Agent 实例管理
- [[安装]]：初始安装
- [[快速开始]]：首次对话指南
- [[配置参考]]：config.yaml 完整参考
- [[网关内部机制]]：网关架构与实现
- [[安全]]：用户授权与命令审批

## 参考来源

- `raw/hermes_docs/user-guide/profiles.md`
- `raw/hermes_docs/getting-started/quickstart.md`
- `raw/hermes_docs/getting-started/installation.md`
- `raw/hermes_docs/user-guide/messaging/index.md`
