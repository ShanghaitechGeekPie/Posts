---
title: "OpenClaw 安全总结：供应链攻击、配置指南与安全指引"
layout: blog
date: "2026-03-15"
author: "Henry"
summary: 本文将结合 OpenClaw 历史上著名的供应链攻击事件，以及北京大学计算中心最新发布的安全指引，为您详细梳理 OpenClaw 的安全风险及最佳实践。
tags:
  - OpenClaw
  - 安全
draft: false
---

OpenClaw 作为一款功能强大的开源 AI 智能体，极大地提升了我们的工作效率，但其引发的安全问题也不容忽视。本文将结合 OpenClaw 历史上著名的「ClawHavoc」供应链攻击事件，以及北京大学计算中心最新发布的[安全指引](https://its.pku.edu.cn/announce/tz20260310195800.jsp)，为您详细梳理 OpenClaw 的安全风险及最佳实践。

## 一、重大安全事件回顾：ClawHavoc 供应链攻击

ClawHavoc 供应链攻击是 OpenClaw 历史上最严重的安全事件之一，每个「养虾人」都必须引以为戒。

### 时间线

| 日期 | 事件 |
|---|---|
| 1月27日 | 首个恶意 Skill 出现在 ClawHub 上，伪装成专业工具 |
| 1月28-30日 | 攻击者快速上传大量恶意 Skill，利用 ClawHub 缺乏审查机制的漏洞 |
| 1月31日 | 攻击全面爆发，多名用户报告异常行为 |
| 2月1日 | Koi Security 正式命名该攻击为「ClawHavoc」 |
| 2月上旬 | 社区展开大规模审计和清理 |

### 攻击规模

当时 ClawHub 技能总数约为 2,857 个，初步确认恶意 Skills 341 个（约 12%），后续扫描发现的恶意 Skills 超过 800 个（约 20%）。其中可追溯到同一协调行动的达 335 个，受影响设备超过 135,000 台。这意味着当时随机安装 5 个 Skill，大概率至少有 1 个是恶意的。

### 攻击手法

攻击者的手法相当精密：

1. 上传看似专业的 Skill，名称和描述都很正常（如 `advanced-code-review`, `smart-scheduler`）。
2. 诱导用户安装后，Skill 会建议安装一个「helper agent」来增强功能。
3. 实际植入的是 Atomic macOS Stealer（AMOS）信息窃取木马。
4. **更危险的是：** 攻击专门针对 OpenClaw 的持久记忆文件（`SOUL.md` 和 `MEMORY.md`），篡改 Agent 的长期行为指令。

篡改 `SOUL.md` 意味着你的 Agent 被「洗脑」了。它的核心行为准则被改写，可能在后续所有交互中执行恶意操作，而你完全不知情。

### 应对建议

1. **安装前审查源码：** 永远不要盲目安装 ClawHub 上的 Skill。去 GitHub 查看源码，确认 `SKILL.md` 中没有可疑的指令。特别注意任何要求额外安装「helper」或「agent」的内容。
2. **使用 SecureClaw 扫描：** 社区推出了开源安全工具 SecureClaw，可以扫描已安装的 Skills 检查恶意内容。
   ```bash
   # 安装SecureClaw
   npm install -g secureclaw
   
   # 扫描已安装的skills
   secureclaw scan ~/.openclaw/skills/
   ```
3. **优先使用精选列表：** 参考 [awesome-openclaw-skills](https://github.com/awesome-openclaw-skills) 项目（近 31.4K Stars）的精选列表，过滤掉大量垃圾和恶意 Skill。
4. **定期检查记忆文件：** 养成习惯，定期检查 `SOUL.md` 和 `MEMORY.md` 有没有被异常修改。

**关键认知：** OpenClaw 的 Skill 本质上是受信任代码。一旦安装，它就拥有和你的 OpenClaw 实例相同的权限。没有沙箱隔离，没有权限分级。这和 npm 生态早期面临的问题一模一样，但后果可能更严重，因为 OpenClaw 可以访问你的邮件、日历、消息和文件系统。

---

## 二、全面风险排查清单

根据北京大学计算中心在 2026 年 3 月 10 日发布的[安全指引](https://its.pku.edu.cn/announce/tz20260310195800.jsp)，OpenClaw 在缺乏有效安全控制的情况下存在四大核心风险：默认无身份认证、存在远程代码执行漏洞（CVE-2026-25253）、第三方技能供应链攻击（如上述的 ClawHavoc）以及 API 密钥明文存储。

以下是详细的安全配置最佳实践与排查清单：

### 1. API 密钥安全与身份认证

绝对不要将 API 密钥明文存储在代码中或提交到公开仓库。

**正确做法：**

*   **使用环境变量：** `export OPENAI_API_KEY="sk-xxx"`
*   **设置文件权限：** `chmod 600 ~/.openclaw/config.json`
*   **配置 Git 忽略：** `echo ".openclaw/config.json" >> .gitignore`
*   **定期轮换密钥：** 建议每 3 个月更换一次。
*   **启用身份认证：** 在配置文件中添加强认证 Token：
    ```json
    {
      "gateway": {
        "auth": {
          "token": "你的至少32位强随机字符串"
        }
      }
    }
    ```

### 2. 排查公网暴露与网络安全配置

千万不要让未启用认证的实例暴露在公网，并利用防火墙或网关配置限制访问。

*   **排查暴露：** 检查默认端口 18789 是否监听在 `0.0.0.0`
    *   Linux: `ss -tlnp | grep 18789`
    *   macOS: `lsof -i :18789`
    *   Windows: `netstat -ano | findstr ":18789"`
*   **限制本地访问：** 修改配置仅监听本地：
    ```json
    {
      "gateway": {
        "mode": "local",
        "port": 18789, 
        "bind": "loopback",
        "allowIPs": [
          "127.0.0.1",
          "192.168.1.0/24"
        ],
        "ssl": {
          "enabled": true,
          "cert": "/path/to/cert.pem",
          "key": "/path/to/key.pem"
        }
      }
    }
    ```

### 3. 数据隐私与文件访问控制

不要在 OpenClaw 中存储或处理极度敏感的信息（如密码、银行卡）。

**配置数据脱敏与路径限制：**

```json
{
  "privacy": {
    "enabled": true,
    "rules": [
      { "type": "phone", "action": "mask", "pattern": "\\d{11}" },
      { "type": "email", "action": "mask" },
      { "type": "idcard", "action": "block" }
    ]
  },
  "files": {
    "allowPaths": [
      "~/Documents/OpenClaw",
      "~/Desktop"
    ],
    "denyPaths": [
      "~/.ssh",
      "~/Documents/Private",
      "~/Documents/Finance",
      "~/Documents/Medical"
    ]
  }
}
```

### 4. 最小权限原则与防范 XSS 劫持

*   **警惕不明链接：** CVE-2026-25253 漏洞允许攻击者通过恶意网页劫持本地 OpenClaw 会话，只需在浏览器中打开链接即可触发。对来源不明的链接务必保持警惕。
*   **低权限运行：** 绝对不要以 `root` 或管理员身份运行 OpenClaw，为其分配专用的低权限账户。

### 5. 审计日志与备份策略

开启日志记录，并利用 v2026.3.8 新增的备份工具保障数据安全。

*   **启用审计：**
    ```json
    {
      "audit": {
        "enabled": true,
        "logLevel": "info",
        "logFile": "~/.openclaw/logs/audit.log",
        "retention": 90
      }
    }
    ```
*   **检查日志：** 定期执行 `tail -n 100 ~/.openclaw/logs/audit.log` 等命令检查是否有异常的 `delete` 或 `upload` 操作。
*   **合理备份：**
    ```bash
    # 创建、验证与恢复备份
    openclaw backup create
    openclaw backup verify <backup-file>
    openclaw backup restore <backup-file>
    ```
    建议每周自动备份，重要操作前手动备份，并定期测试恢复流程。

---

安全无小事。作为一个迅速发展的开源项目，OpenClaw 为我们带来便利的同时也伴随着风险。请务必养成定期更新、审查代码、最小化权限和定期备份的良好习惯。
