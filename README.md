# workbuddy-signin
WorkBuddy 每日签到青龙面板脚本
# WorkBuddy 每日签到脚本（青龙面板版）

自动领取 **WorkBuddy（腾讯 AI 编程助手）** 每日签到积分的青龙面板脚本，适用于Docker 青龙、各种 NAS 设备。

---

## ✨ 特性

| 特性 | 说明 |
|---|---|
| 📦 **零依赖** | 纯 Python 标准库，无需 `pip install` 任何包 |
| 👥 **多账号** | 多账号批量签到，互不干扰 |
| ✅ **幂等安全** | 先查状态，未签才领，重复运行不会多领 |
| ⏰ **Token 过期预警** | 自动解码 JWT 过期时间，到期前在通知中主动提醒 |
| 📣 **多渠道通知** | 青龙面板 / PushPlus 微信推送 / 控制台输出 |
| 🔒 **加密发布版** | 提供加密版本，防止随意篡改核心逻辑 |

---

## 📥 安装（青龙面板）

### 方法一：青龙拉取（推荐）

在青龙面板的**命令行**里执行：

```bash
ql raw https://raw.githubusercontent.com/hdxiaoke/workbuddy-signin/main/workbuddy_signin_protected.py
```

拉取后，脚本会自动加入定时任务。

### 方法二：手动上传

1. 下载 [workbuddy_signin_protected.py](workbuddy_signin_protected.py) 或 [workbuddy_signin.py](workbuddy_signin.py)
2. 进入青龙面板 → **脚本管理** → 点击「添加文件」上传

---

## ⚙️ 环境变量配置

在青龙面板的**环境变量**中添加以下变量：

### 必填项

| 变量名 | 示例值 | 说明 |
|---|---|---|
| `WORKBUDDY_TOKEN` | `eyJhbGciOi...` | WorkBuddy 的 accessToken |
| `WORKBUDDY_UID` | `123456789` | WorkBuddy 的 uid |

**多账号配置**：用 `&` 分隔，数量和顺序必须一一对应：

```
WORKBUDDY_TOKEN = token1&token2&token3
WORKBUDDY_UID   = uid1&uid2&uid3
```

### 可选项

| 变量名 | 默认值 | 说明 |
|---|---|---|
| `WORKBUDDY_EXTRA` | 留空 | 额外字段，格式：`enterpriseId#domain#endpoint`，多账号用 `&` 分隔 |
| `WORKBUDDY_ENDPOINT` | `https://copilot.tencent.com` | 全局默认接口地址 |
| `WORKBUDDY_TOKEN_WARN_DAYS` | `2` | Token 过期预警天数，剩余 ≤ 此值时通知预警 |
| `PUSHPLUS_TOKEN` | 留空 | PushPlus 推送 token，配置后微信接收通知 |

---

## 🔑 如何获取 accessToken 和 uid

### 第一步：安装并登录 WorkBuddy 桌面端

下载地址：https://copilot.tencent.com

### 第二步：找到凭据文件

登录桌面端后，凭据文件会自动生成，路径如下：

| 系统 | 文件路径 |
|---|---|
| **Windows** | `%LOCALAPPDATA%\CodeBuddyExtension\Data\Public\auth\workbuddy-desktop.info` |
| **macOS** | `~/Library/Application Support/CodeBuddyExtension/Data/Public/auth/workbuddy-desktop.info` |
| **Linux** | `~/.config/CodeBuddyExtension/Data/Public/auth/workbuddy-desktop.info` |

> 💡 **快速打开（Windows）**：按 `Win + R`，粘贴 `%LOCALAPPDATA%\CodeBuddyExtension\Data\Public\auth`，回车即可。

### 第三步：提取关键字段

用文本编辑器打开 `workbuddy-desktop.info`，这是一个 JSON 文件：

```json
{
  "auth": {
    "accessToken": "eyJhbGciOi...（一长串，填到 WORKBUDDY_TOKEN）",
    "domain": "...",
    "endpoint": "https://copilot.tencent.com"
  },
  "account": {
    "uid": "123456789（填到 WORKBUDDY_UID）",
    "enterpriseId": "..."
  }
}
```

| JSON 字段 | 填入环境变量 |
|---|---|
| `auth.accessToken` | `WORKBUDDY_TOKEN` |
| `account.uid` | `WORKBUDDY_UID` |
| `account.enterpriseId` + `auth.domain` + `auth.endpoint` | `WORKBUDDY_EXTRA`（格式：`enterpriseId#domain#endpoint`，一般不用填） |

---

## 📣 PushPlus 微信推送（可选）

配置后，签到结果会直接推送到你的微信。

1. 访问 https://www.pushplus.plus ，**微信扫码登录**
2. 在「一对一推送」页面**获取 token**
3. 在青龙面板添加环境变量：

| 变量名 | 值 |
|---|---|
| `PUSHPLUS_TOKEN` | 你获取的 PushPlus token |

4. 执行脚本，微信会收到推送通知 ✉️

---

## ⏰ 定时任务

青龙面板会自动读取脚本头部的 `cron: 5 0 * * *`（每天 00:05）。

如需手动调整：

| Cron 表达式 | 说明 |
|---|---|
| `5 0 * * *` | 每天 00:05 签到（推荐，避开零点拥堵） |
| `0 8 * * *` | 每天 08:00 签到 |
| `0 */12 * * *` | 每 12 小时签到一次（用于排查，不推荐） |

---

## 📋 脚本说明

| 文件 | 用途 | 适用人群 |
|---|---|---|
| [workbuddy_signin.py](workbuddy_signin.py) | 开发版，源码可读 | 想二次开发、调试、学习的用户 |
| [workbuddy_signin_protected.py](workbuddy_signin_protected.py) | 发布版，核心逻辑加密 | 普通用户使用，防止核心逻辑被随意篡改 |

两个版本**功能完全一致**，只是是否加密的区别。

---

## ⚠️ 常见问题

### Q1：签到成功后还会重复领取吗？
不会。脚本先调用「查询签到状态」接口，只有未签到时才会执行领取操作，重复运行安全。

### Q2：Token 过期了怎么办？
过期时通知中会明确提示。只需：
1. 重新登录 WorkBuddy 桌面端
2. 打开凭据文件，复制新的 `accessToken`
3. 更新青龙面板里的 `WORKBUDDY_TOKEN` 环境变量即可

### Q3：Token 多久过期？
一般 7-30 天，由 WorkBuddy 官方决定。脚本开启了过期预警（默认提前 2 天提醒），你可以调整 `WORKBUDDY_TOKEN_WARN_DAYS` 来改变预警时间。

### Q4：为什么没有推送通知？
- 青龙面板通知：确认青龙自身通知配置正确（如 Server 酱等）
- PushPlus 通知：确认 `PUSHPLUS_TOKEN` 填对，且在 PushPlus 网站上能正常发送测试消息
- 没有配置任何通知时，结果会直接输出到控制台/日志


---

## 📝 免责声明

本脚本为非官方工具，签到接口系从桌面端逆向得到，仅供个人自动化使用。接口可能随时变动且不另行通知，使用风险自负。请遵守 WorkBuddy 官方的用户协议和服务条款，合理使用本脚本。

---

## ⭐ 支持

如果这个脚本对你有帮助，点个 Star ⭐ 就是最大的支持！
