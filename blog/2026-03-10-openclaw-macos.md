---
title: macOS 环境下 OpenClaw 完全部署与配置教程
layout: blog
date: 2026-03-10
author: Henry
summary: 本文记录了在 macOS 环境下安装 OpenClaw 的完整过程，包含 Homebrew 自动化安装、Node.js 环境配置、Gateway 服务启动，以及常用命令和插件管理的详细步骤。
tags:
  - OpenClaw
  - 教程
  - macOS
  - 配置
---

# macOS 环境下 OpenClaw 完全部署与配置教程

[Windows 安装教程点我](https://geekpie.club/posts/blog/2026-03-09-openclaw/)
OpenClaw 是一款强大的开源 AI Agent 工具，相比 Windows 环境，macOS 的安装体验更加流畅，得益于 Homebrew 包管理器的自动化支持。本文将详细记录在 macOS 上从零开始安装、配置 OpenClaw 的完整流程。

---

## 一、环境准备

### 1. 系统要求

- macOS 10.15 或更高版本（本教程基于 macOS 15.4.1）
- 已安装 Homebrew 包管理器
- 稳定的网络连接

#### 如何查看自己的 macOS 版本？
1. 打开终端（也叫Terminal）

![终端图片](/res/terminal.png)

2. 输入```sw_vers```并回车，```ProductVersion```后的即为版本号

![版本号](/res/macos-version.png)

### 2. 检查 Homebrew

如果尚未安装 Homebrew，请先访问 [brew.sh](https://brew.sh/) 安装，或在终端执行：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

验证 Homebrew 安装：

```bash
brew --version
```

---

## 二、一键安装 OpenClaw

### 1. 执行官方安装脚本

OpenClaw 提供了针对 macOS 优化的安装脚本，会自动处理 Node.js 版本升级和依赖安装。在终端中执行：

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash
```

### 2. 安装过程说明

安装脚本会自动完成以下步骤：

1. **检测系统环境**：识别 macOS 系统并选择合适的安装方式
2. **安装 Node.js v22**：通过 Homebrew 安装 `node@22`（OpenClaw 要求 Node.js 22+）
3. **安装 OpenClaw**：通过 npm 全局安装最新版本
4. **配置环境变量**：在 `~/.local/bin/openclaw` 创建启动脚本

安装成功后，你会看到类似以下输出：

```
🦞 OpenClaw installed successfully (OpenClaw 2026.3.8)!
Home sweet home. Don't worry, I won't rearrange the furniture.
```

### 3. 配置 PATH 环境变量

为了在任何位置都能使用 `openclaw` 命令，需要将 Node.js 22 添加到 PATH。编辑 `~/.zshrc` 文件：

```bash
echo 'export PATH="/opt/homebrew/opt/node@22/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

验证安装：

```bash
openclaw --version
```

应该输出类似：`OpenClaw 2026.3.8 (3caab92)`

---

## 三、初始化配置

### 1. 执行配置向导

首次使用需要在终端里执行：

```bash
openclaw onboard --flow quickstart
```

在连续的风险提示与初始询问中：
- 看到 `I understand this is powerful and inherently risky. Continue?`，请选择 `Yes`。

![风险确认](/res/risk-confirm-macos.png)

后续配置项建议按需简化（可使用最小化跑通再深层配置）：
- **模型/Model**：选择所需模型，若暂未准备 API 可选 `skip for now`。
- **API Keys**：提供对应的 API Key (例如 Kimi、DeepSeek 或 MiniMax)，其余保持默认。
  
下面以 MiniMax 为例获取 API Key：
  
  1. 打开浏览器前往 [MiniMax 开发者平台 (platform.minimaxi.com)](https://platform.minimaxi.com/)，并通过手机号快速注册/登录。
  2. 登录后，在页面右上角中找到并点击 **账户管理** ，左侧栏选择 **接口密钥** ，
     
     ![MiniMax 控制台配置 API Keys](/res/minimax-api-keys.png)
  3. 点击 **创建新的 API Key** 或 **+ 新建**，给这个 Key 随意命名（比如 "OpenClaw"）并确认。
  4. 系统会生成一串以 `sk-` 开头的密钥。**该密钥仅会完整显示一次**，请务必立刻**复制并妥善保存**至密码管理器或安全位置，切勿发给他人。
     
     ![获取并复制 API Key](/res/minimax-sk-key.png)

![API Key配置](/res/api-key-config-macos.png)

- **Select Channel/Search Provider/Skills***：可以先 `skip for now`，先验证安装完成，后续单独配置。

- **Hooks**：全部勾选。上下箭头选择，空格选中/取消选中，回车提交。

![hooks设置](/res/hooks-config-macos.png)

- **Hatch your bot**：选择 `open in web ui`，将启动网关服务并在浏览器弹出控制台页面 `http://127.0.0.1:18789/`。控制台启动完成并可发消息回复，即代表基础运行成功。*(注：目前 Web UI 界面仍处于早期阶段，体验还不是很完善。对于熟悉命令行的用户，强烈推荐使用命令行交互，后续可通过终端输入 `openclaw tui` 进入。)*

![Web控制台](/res/openclaw-web-macos.png)

你可以在 Web UI 中：
- 发送消息与 AI 对话
- 查看会话历史
- 管理设备配对
- 配置模型和插件

---

## 四、配置 AI 模型
如果后续额外配置其他AI模型，可以参考以下步骤：
```bash
openclaw config
```
选择```Local```→```Model```

随后选择自己的提供商。如果没有则选择```Custom Provider```。随后同上方 初始化配置```API Keys```处所提的相同方式配置。

### 验证配置

```bash
openclaw health
```

### 动态切换模型

如果需要在聊天过程中临时切换不同的 AI 模型，可以在终端使用 TUI（文本用户界面）进行操作：
1. 在终端输入 `openclaw tui` 打开命令行交互界面。
2. 在对话框中输入 `/model`（或者 `/models`）并回车。
3. 使用键盘的上下方向键挑选想要的模型，最后回车确认，即可切换成功。

![动态切换模型](/res/change-model-macos.png)

---

## 五、插件管理

### 1. 查看所有插件

```bash
openclaw plugins list
```

OpenClaw 内置 38+ 插件，包括：
- 通讯平台：Telegram、Discord、Slack、飞书、微信等
- 语音功能：voice-call、talk-voice
- 内存管理：memory-core、memory-lancedb
- 其他工具：browser、diffs、lobster 等

### 2. 启用插件

以飞书插件为例：

```bash
openclaw plugins enable feishu
```

### 3. 安装第三方插件

```bash
openclaw plugins install @m1heng-clawd/feishu
```

---

## 六、常用命令速查

### 服务管理

```bash
# 启动 Gateway
openclaw gateway

# 查看健康状态
openclaw health

# 查看实时日志
openclaw logs follow

# 重启服务
openclaw restart
```

### 配置管理

```bash
# 查看配置
openclaw config get

# 设置配置项
openclaw config set key value

# 删除配置项
openclaw config unset key

# 打开配置文件
openclaw config file
```

### 诊断与修复

```bash
# 运行诊断
openclaw doctor

# 自动修复问题
openclaw doctor --fix

# 查看详细日志
openclaw logs follow --level debug
```

### 设备管理

```bash
# 生成配对码
openclaw devices pair

# 查看已配对设备
openclaw devices list

# 打开控制台
openclaw dashboard
```

---

## 七、常见问题排查

### 1. 端口被占用

如果 18789 端口被占用，可以指定其他端口：

```bash
openclaw gateway --port 19000
```

或强制关闭占用进程：

```bash
openclaw gateway --force
```

### 2. Node.js 版本不匹配

确保使用 Node.js 22：

```bash
node --version  # 应显示 v22.x.x
```

如果版本不对，重新加载环境变量：

```bash
source ~/.zshrc
```

### 3. 权限问题

如果遇到权限错误，检查目录权限：

```bash
ls -la ~/.openclaw
chmod -R 755 ~/.openclaw
```

### 4. 网络连接问题

如果安装过程中下载缓慢，可以配置 npm 镜像：

```bash
npm config set registry https://registry.npmmirror.com
```

完整文档：[https://docs.openclaw.ai](https://docs.openclaw.ai)

整个 macOS 环境下的 OpenClaw 部署流程到此完成。

—— GeekPie 社团，2026 年 3 月 10 日
