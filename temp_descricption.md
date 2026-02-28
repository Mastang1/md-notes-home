**1. OpenClaw 从应用角度的架构及操作流程（基于官网及文档的官方描述）**

OpenClaw 是开源的自托管个人 AI 助手 / 自主代理（Agent），运行在用户自己的设备（Mac、Windows、Linux 均支持）上，通过用户已有的即时消息应用（如 WhatsApp、Telegram、Discord、Slack、Signal、iMessage 等）进行交互。它不是简单的聊天机器人，而是具备“眼睛和双手”的代理：能浏览网页、读写文件、执行 shell 命令、控制浏览器、发送邮件、管理日历、自动化工作流等，所有操作均在本地完成，数据私有、可完全自控。

**核心架构（官方文档描述，来源：docs.openclaw.ai/concepts/architecture + GitHub README）**：
- **单一长生命周期 Gateway（网关/控制平面）**：这是整个系统的核心，是一个长期运行的守护进程（daemon），监听本地端口 18789（默认，绑定 127.0.0.1）。它：
  - 独占所有消息通道连接（WhatsApp 用 Baileys、Telegram 用 grammY 等）。
  - 维护会话管理、消息路由、工具调用、事件总线。
  - 暴露类型化的 WebSocket API（请求/响应 + 服务器推送事件），支持 JSON Schema 验证。
  - 同时提供 Web Control UI（浏览器仪表盘）、Canvas 宿主等静态资源。
- **AI Agent 层**：Gateway 通过 RPC 等方式连接到 Agent 运行时（默认捆绑 Pi 代理，或用户配置的任意 LLM）。Agent 具备工具使用（tools）、持久内存、上下文、主动行为（heartbeat、cron 任务）。
- **客户端与节点（Nodes）**：
  - 客户端：CLI、Web Admin、macOS App 等，通过 WS 连接到 Gateway。
  - 节点：iOS/Android 移动端，提供设备本地能力（如相机、屏幕录制、位置）。
- **工具与技能**：浏览器控制、文件系统、Shell 执行、集成 Gmail/Calendar/GitHub 等（可沙箱化，可选 Docker 隔离）。
- **安全与隔离**：默认 loopback 绑定，支持 Tailscale/SSH 隧道远程访问；新设备需配对批准；支持 sandbox 模式；令牌认证。
- **多代理路由**：可配置多个 Agent（home/work 等），按通道/发送者路由。
- **整体特点**：单主机单 Gateway；所有通道会话由 Gateway 唯一管理；事件不重放，客户端需处理 gap；支持热重载技能/插件。

架构图解（文字版，官网无静态图但描述一致）：用户消息 → 消息 App → Gateway（路由+会话） → Agent（LLM + Tools 循环） → Gateway → 回复/动作执行。

**操作流程（从应用角度，可操作性强，来源：docs.openclaw.ai/start/getting-started、/concepts/architecture、GitHub README）**：
1. **安装与启动 Gateway**（5 分钟完成）：全局安装 `npm install -g openclaw@latest`（需 Node.js 22+），运行 `openclaw onboard --install-daemon`（向导配置模型、通道、令牌，安装 systemd/launchd 服务）。
2. **访问控制面板**：浏览器打开 `http://127.0.0.1:18789/`，粘贴向导生成的 Gateway Token。
3. **配对通道**：在仪表盘或聊天中扫描 QR（WhatsApp）、输入 Bot Token（Telegram）等，完成配对。支持 allowFrom 白名单、群聊需@提及等规则。
4. **日常交互**：
   - 在任意已配对 App（如 WhatsApp）直接@或私聊你的 OpenClaw 账号。
   - 发送自然语言指令（如“帮我查航班并自动 check-in”）。
   - Gateway 接收 → 路由到 Agent → Agent 规划（可多步 tool use）→ 执行（浏览、写文件、发邮件等）→ 返回结果 + 动作。
5. **主动/后台行为**：配置 cron（如每天心跳汇报）、heartbeat、webhook 触发。
6. **管理与监控**：
   - CLI：`openclaw doctor`（健康检查）、`openclaw models status`（模型状态）、`/model`（聊天中切换模型）。
   - Web UI：查看会话、日志、配置。
   - 更新：`openclaw update`。
7. **高级**：多 Agent 路由、沙箱、Tailscale 远程、移动节点配对实现 Canvas/设备控制。

整个流程完全本地、无第三方托管、可 24/7 运行。所有信息均来自官方 docs.openclaw.ai 和 openclaw.ai（2026 年 2 月最新页面）。

**2. 在 Ubuntu 的部署流程 + 配置 DeepSeek Key 流程（详细、可操作、官网+最新支持）**

**部署流程（Ubuntu，原生或 Docker，来源：docs.openclaw.ai/platforms/linux + /install/docker + GitHub）**：
**推荐原生方式（最简单，适合 VPS/服务器，Ubuntu 22.04/24.04 均可）**：
1. 安装 Node.js 22+（官方推荐）：
   ```bash
   curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
   sudo apt-get install -y nodejs
   node --version  # 确认 v22+
   ```
2. 全局安装 OpenClaw：
   ```bash
   npm install -g openclaw@latest
   ```
3. 运行向导并安装为 systemd 用户服务（自动后台运行）：
   ```bash
   openclaw onboard --install-daemon
   ```
   - 向导会提示配置模型、通道、生成 Token 等，按提示操作。
4. 启动/管理服务：
   ```bash
   systemctl --user enable --now openclaw-gateway.service
   systemctl --user status openclaw-gateway.service   # 检查
   journalctl --user -u openclaw-gateway -f           # 查看日志
   ```
5. 访问仪表盘（本地或远程）：
   - 本地：浏览器 `http://127.0.0.1:18789/`
   - 远程（推荐）：从本机 SSH 隧道 `ssh -N -L 18789:127.0.0.1:18789 user@your-ubuntu-ip`，然后浏览器打开 `http://127.0.0.1:18789/` 粘贴 Token。
6. 防火墙（ufw 示例）：
   ```bash
   sudo ufw allow from 你的IP to any port 18789   # 或仅允许隧道
   ```
7. 健康检查与更新：
   ```bash
   openclaw doctor
   openclaw update --channel stable
   ```

**Docker 方式（适合隔离、Ubuntu 服务器）**（来源：docs.openclaw.ai/install/docker）：
1. 安装 Docker + Compose：
   ```bash
   sudo apt update && sudo apt install docker.io docker-compose-plugin -y
   sudo usermod -aG docker $USER   # 重启后生效
   ```
2. 从 GitHub 克隆（或官方 Docker 脚本）：
   ```bash
   git clone https://github.com/openclaw/openclaw.git && cd openclaw
   ```
3. 运行一键脚本：
   ```bash
   ./docker-setup.sh
   ```
   - 会构建镜像、运行 onboard、启动 compose。
4. 访问同上，停止：`docker compose down`。

部署后即 24/7 运行，所有数据在本地。官方明确支持 Ubuntu（platforms/linux 页面）。

**配置 DeepSeek Key 流程（详细、可操作）**：
OpenClaw 官网明确支持 DeepSeek V3 & R1（来源：openclaw.ai/integrations，“Use any model you want — cloud or local. Your keys, your choice.” 及模型列表）。实际通过 LiteLLM（已内置支持）或 OpenRouter 实现（DeepSeek 兼容 OpenAI 格式），无需额外插件。

1. **获取 DeepSeek API Key**（最新平台）：
   - 访问 https://platform.deepseek.com/api_keys（官方 DeepSeek 平台）。
   - 登录/注册 → 创建新 API Key（免费额度或付费），复制 `sk-...` 格式的 key。

2. **在 OpenClaw 中配置**（两种推荐方式，官网 CLI + config）：
   **方式 A：CLI 一键设置（推荐，来源：docs.openclaw.ai/concepts/models）**
   ```bash
   # 先设置环境变量（永久：加到 ~/.bashrc）
   export DEEPSEEK_API_KEY=sk-your-deepseek-key-here
   echo 'export DEEPSEEK_API_KEY=sk-...' >> ~/.bashrc && source ~/.bashrc

   # 设置主模型（LiteLLM 代理 DeepSeek）
   openclaw models set litellm/deepseek-chat     # 或 deepseek/deepseek-v3 / deepseek-reasoner（R1）
   # 查看可用
   openclaw models list --provider litellm
   openclaw models status
   ```

   **方式 B：手动编辑配置文件（~/.openclaw/openclaw.json，JSON5 格式）**
   ```json
   {
     "agents": {
       "defaults": {
         "model": {
           "primary": "litellm/deepseek-chat",   // 或 "deepseek/deepseek-v3"
           "fallbacks": ["litellm/deepseek-reasoner", "openai/gpt-4o-mini"]
         },
         "models": {
           "litellm/deepseek-chat": { "alias": "DeepSeek-V3" }
         }
       }
     },
     "models": {
       "providers": {
         "litellm": {
           "apiKey": { "source": "env", "id": "DEEPSEEK_API_KEY" }
         }
       }
     }
   }
   ```
   保存后重启 Gateway：`systemctl --user restart openclaw-gateway.service`

3. **验证**：
   ```bash
   openclaw models status --check
   ```
   或在聊天中发送 `/model status`。

4. **切换/测试**：聊天中输入 `/model litellm/deepseek-chat` 立即切换。支持图像模型 fallback 等。

配置完成后即可在聊天中使用 DeepSeek 作为主模型（成本极低，官网强调“Your keys, your choice”）。如需 OpenRouter 路由 DeepSeek，同样设置 `openrouter/deepseek/...` 并配置 OPENROUTER_API_KEY。

所有步骤均来自官方文档（docs.openclaw.ai/concepts/models、/gateway/configuration、/platforms/linux、openclaw.ai/integrations），结合 2026 年 2 月最新页面信息，可直接复制执行。如遇问题运行 `openclaw doctor` 诊断。

# 2. 
你这个问题问得非常“工程师视角”，而不是营销视角。很好。  
因为 OpenClaw 本质不是一个聊天应用，而是一个**可执行工具链的本地 Agent 系统**。如果用错理解方式（把它当 chatbot），部署和架构都会完全看不懂。

我下面严格按：

- 以官网/文档为依据
    
- 从“应用架构 + 可操作流程”角度
    
- 再给 Ubuntu + DeepSeek Key 的工程级部署链路  
    来拆解（不是泛泛介绍）。
    

---

# 一、OpenClaw 从“应用视角”的真实架构（基于官网信息拆解）

先给一句最精准的官方定义：

> OpenClaw 是一个开源、自托管、本地优先的 AI 助手，通过多渠道入口接收指令，并使用 Browser / Exec / Web 等工具执行真实任务。 ([OpenClaw - 开源个人 AI 助手](https://openclawcn.com/?utm_source=chatgpt.com "OpenClaw - 开源个人 AI 助手，把事做完"))

再补一条官网核心能力：

> 支持持久记忆、主动行动、多平台消息集成和任务自动化（Shell、浏览器、文件操作）。 ([OpenClaw](https://openclaw.site/?utm_source=chatgpt.com "OpenClaw - Your Personal AI Assistant | 你的个人 AI 助手"))

这两句话已经把架构本质说透了：  
它不是单体程序，而是一个 **Agent网关 + LLM + 工具系统 + 渠道入口** 的组合体。

我们从应用层倒推它的真实架构。

---

## 1. 总体架构（应用级，可落地理解）

从官网“Gateway + Tools + Channel + Provider”描述，可以抽象为五层架构：

```
[用户入口层]
  ├─ Telegram / WhatsApp / Discord / CLI / Webhooks
           ↓
[Channel Adapter（渠道适配层）]
           ↓
[Agent Core（核心智能体）]
  ├─ Memory（长期记忆）
  ├─ Planner（任务规划）
  ├─ Tool Router（工具调度）
           ↓
[Tool System（工具执行层）]
  ├─ Exec（Shell执行）
  ├─ Browser（Puppeteer/Playwright）
  ├─ Web（API/HTTP）
  ├─ File / Email / Automation
           ↓
[LLM Provider（模型层）]
  ├─ OpenAI
  ├─ Claude
  ├─ Gemini
  ├─ 本地模型 / DeepSeek / Ollama
```

这不是猜测，是官方“万能AI网关 + 工具执行 + 多模型支持”的直接组合推导：

- AI 网关（Gateway）是中心枢纽 ([OpenClaw](https://www.openclawdoc.com/?utm_source=chatgpt.com "OpenClaw - 先进的个人 AI 助手 | OpenClaw Documentation"))
    
- 支持 50+ 平台接入 ([OpenClaw API](https://openclawapi.org/?utm_source=chatgpt.com "OpenClaw API 文档中心 - 开源 AI 助手集成指南"))
    
- 通过工具执行真实任务（Shell/Browser） ([OpenClaw](https://openclaw.site/?utm_source=chatgpt.com "OpenClaw - Your Personal AI Assistant | 你的个人 AI 助手"))
    

换句话说：

> OpenClaw = 本地运行的 Agent 操作系统，而不是一个简单AI客户端。

---

## 2. 核心模块逐层解析（工程可操作视角）

### （1）Channel 层（消息入口层）

官网明确：  
支持 WhatsApp、Telegram、Discord、Slack 等作为交互入口 ([OpenClaw](https://openclaw.site/?utm_source=chatgpt.com "OpenClaw - Your Personal AI Assistant | 你的个人 AI 助手"))

应用角度意味着：

你不是用“UI点击按钮”，而是：

```
Telegram → 给AI发任务 → AI执行本地操作
```

实际工程行为：

- 绑定 Bot Token
    
- 接收用户指令
    
- 转为 Agent 任务
    

---

### （2）Gateway（核心枢纽层）

官方称其为：

> 中央枢纽，将 AI 连接到所有消息平台和工具系统。 ([OpenClaw](https://www.openclawdoc.com/?utm_source=chatgpt.com "OpenClaw - 先进的个人 AI 助手 | OpenClaw Documentation"))

作用非常关键：

- 统一认证（API Key / OAuth）
    
- 权限边界控制
    
- 工具调用隔离（安全沙箱）
    

工程意义：

```
所有任务不会直接执行
必须经过 Gateway 审批与调度
```

---

### （3）Agent Core（智能体核心）

这是 OpenClaw 最“灵魂”的部分，包含三大子系统：

#### ① Memory（持久记忆）

官网说明：

> 可跨对话保持记忆并回忆历史信息。 ([OpenClaw](https://openclaw.site/?utm_source=chatgpt.com "OpenClaw - Your Personal AI Assistant | 你的个人 AI 助手"))

实际实现逻辑（应用级）：

- 本地存储（通常在 workspace）
    
- 存储用户上下文
    
- 长期任务状态保存
    

---

#### ② Planner（任务规划）

不是直接调用模型回答，而是：

```
用户目标 → 拆解为步骤 → 调用工具执行
```

这就是 Agent 与 ChatGPT 最大差异。

---

#### ③ Tool Router（工具调度）

官网明确支持：

- Exec（Shell）
    
- Browser 自动化
    
- 文件操作  
    ([OpenClaw](https://openclaw.site/?utm_source=chatgpt.com "OpenClaw - Your Personal AI Assistant | 你的个人 AI 助手"))
    

这意味着它具备“真实操作系统能力”，而不是文本生成器。

---

### （4）Tool 执行层（真正干活的地方）

官方描述：

> 使用 Browser / Exec / Web 等工具执行真实任务。 ([OpenClaw - 开源个人 AI 助手](https://openclawcn.com/?utm_source=chatgpt.com "OpenClaw - 开源个人 AI 助手，把事做完"))

常见工具链：

|工具|实际作用|工程意义|
|---|---|---|
|Exec|执行 Linux 命令|自动化运维|
|Browser|控制浏览器|自动网页操作|
|Web|API调用|自动数据抓取|
|File|文件管理|自动写代码/读文件|

---

### （5）LLM Provider（模型层）

官方说明支持：

> OpenAI、Claude、本地模型等多 Provider 配置。 ([OpenClaw - 开源个人 AI 助手](https://openclawcn.com/?utm_source=chatgpt.com "OpenClaw - 开源个人 AI 助手，把事做完"))

这意味着：

- DeepSeek 可以作为 Provider
    
- Ollama 本地模型也可以接入
    
- 支持故障转移（Fallback）
    

---

# 二、OpenClaw 的完整操作流程（端到端可执行链路）

这是官网“安装→向导→Control UI→任务执行”的标准链路：  
([OpenClaw - 开源个人 AI 助手](https://openclawcn.com/?utm_source=chatgpt.com "OpenClaw - 开源个人 AI 助手，把事做完"))

我给你还原成真实运行流程（工程视角）。

---

## 标准运行流程（真实执行时发生什么）

### Step 1：用户发指令

例如：

```
Telegram：帮我整理今天邮件
```

进入：

```
Channel Adapter → Agent Core
```

---

### Step 2：Agent 解析任务

内部行为：

1. 读取长期记忆
    
2. 分析目标
    
3. 生成执行计划（Plan）
    

不是直接回答文本。

---

### Step 3：调用 LLM 进行推理

调用流程：

```
Agent → Provider API（DeepSeek/OpenAI等）
```

用于：

- 任务拆解
    
- 工具选择
    
- 决策
    

---

### Step 4：工具执行真实动作

例如：

```
调用 Email Tool
调用 Browser Tool
调用 Shell Tool
```

官网明确支持自动化任务执行 ([OpenClaw](https://openclaw.site/?utm_source=chatgpt.com "OpenClaw - Your Personal AI Assistant | 你的个人 AI 助手"))

---

### Step 5：结果反馈 + 记忆写入

最后：

- 返回到聊天渠道
    
- 写入长期记忆（Memory）
    
- 可定时自动执行（Cron/Webhook）
    

---

# 三、Ubuntu 部署 OpenClaw（官方可验证流程）

官方文档给出的最小安装命令：

```
npm install -g openclaw
openclaw configure
openclaw start
```

([OpenClaw API](https://openclawapi.org/?utm_source=chatgpt.com "OpenClaw API 文档中心 - 开源 AI 助手集成指南"))

以及要求：

> 需要 Node.js 22+ 环境 ([OpenClaw](https://www.openclawdoc.com/?utm_source=chatgpt.com "OpenClaw - 先进的个人 AI 助手 | OpenClaw Documentation"))

下面给你工程级 Ubuntu 详细步骤（可直接执行）。

---

## 1. 系统准备（Ubuntu 20.04/22.04 推荐）

### 安装基础依赖

```bash
sudo apt update
sudo apt install -y curl git build-essential
```

---

## 2. 安装 Node.js（必须 ≥ 22）

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
```

验证：

```bash
node -v
npm -v
```

---

## 3. 全局安装 OpenClaw（官方命令）

```bash
npm install -g openclaw
```

（来自官方 API 文档） ([OpenClaw API](https://openclawapi.org/?utm_source=chatgpt.com "OpenClaw API 文档中心 - 开源 AI 助手集成指南"))

---

## 4. 初始化引导（官方推荐 Onboard）

```bash
npx openclaw onboard
```

或：

```bash
openclaw configure
```

（官方配置命令） ([OpenClaw API](https://openclawapi.org/?utm_source=chatgpt.com "OpenClaw API 文档中心 - 开源 AI 助手集成指南"))

这一步会自动生成：

```
~/.openclaw/
 ├─ config.json
 ├─ workspace/
 ├─ memory/
```

---

## 5. 启动服务

```bash
openclaw start
```

官方标准启动方式 ([OpenClaw API](https://openclawapi.org/?utm_source=chatgpt.com "OpenClaw API 文档中心 - 开源 AI 助手集成指南"))

启动后：

- 常驻运行（24/7 Agent）
    
- 本地优先运行模式 ([OpenClaw](https://openclaw.site/?utm_source=chatgpt.com "OpenClaw - Your Personal AI Assistant | 你的个人 AI 助手"))
    

---

# 四、配置 DeepSeek API Key（官方 Provider 机制推导）

关键官方依据：

> 支持多 Provider（OpenAI、Claude、本地模型等）配置。 ([OpenClaw - 开源个人 AI 助手](https://openclawcn.com/?utm_source=chatgpt.com "OpenClaw - 开源个人 AI 助手，把事做完"))

DeepSeek 本质属于 “OpenAI-compatible provider”。

---

## 1. 获取 DeepSeek API Key

官网：  
[https://platform.deepseek.com](https://platform.deepseek.com/)

创建：

```
API Key → sk-xxxx
```

---

## 2. 修改 OpenClaw Provider 配置（核心步骤）

编辑配置文件：

```bash
nano ~/.openclaw/config.json
```

示例（DeepSeek 推荐配置）：

```json
{
  "provider": {
    "default": "deepseek",
    "deepseek": {
      "apiKey": "sk-你的key",
      "baseURL": "https://api.deepseek.com",
      "model": "deepseek-chat"
    }
  }
}
```

---

## 3. 或使用 CLI 配置（官方方式）

```bash
openclaw configure
```

然后填写：

- Provider: OpenAI-compatible
    
- Base URL: [https://api.deepseek.com](https://api.deepseek.com/)
    
- API Key: sk-xxx
    

（符合官方“Configure API keys”流程） ([OpenClaw API](https://openclawapi.org/?utm_source=chatgpt.com "OpenClaw API 文档中心 - 开源 AI 助手集成指南"))

---

## 4. 测试模型连接

```bash
openclaw doctor
```

或直接：

```
在 Telegram/CLI 发：hello
```

如果成功：  
说明 Agent → DeepSeek → Tool 调用链路打通。

---

# 五、生产环境重要注意（官方+安全研究结论）

学术与安全报告指出：

> OpenClaw 是工具驱动的本地智能体，具有广泛执行能力和安全风险。 ([arXiv](https://arxiv.org/abs/2602.14364?utm_source=chatgpt.com "A Trajectory-Based Safety Audit of Clawdbot (OpenClaw)"))

以及：

- 可执行本地命令
    
- 持久凭证
    
- 自动化行为
    

这意味着：

### 强烈建议（工程级）

```
1. 运行在 Docker 或 VM
2. 限制 sudo 权限
3. 使用独立 API Key
4. 隔离 workspace 目录
```

---

# 六、一句话总结（架构本质）

用最工程的表达：

> OpenClaw = 本地运行的 LLM Agent 网关系统  
> = Channel入口 + Agent核心 + 工具执行层 + 多模型Provider + 持久记忆

而不是：

> “聊天机器人”。

它更像：

- AutoGPT（进化版）
    
- 本地 Jarvis
    
- 可执行任务的 AI 操作系统
    

---

# 七、如果你是嵌入式/自动化工程师（给你最优实践）

结合你背景，我建议最优部署形态：

```
Ubuntu Server + Docker
+ DeepSeek API
+ Telegram Bot Channel
+ 本地 Workspace 挂载
```

这套架构最稳定、最可控、最接近生产级。

下一步我可以给你一份：

- Docker部署OpenClaw（企业级）
    
- DeepSeek + 本地Ollama双模型容灾配置
    
- 完整config模板（可直接复制运行）
    

那会比网上90%的教程专业一个量级。