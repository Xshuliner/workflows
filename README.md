# workflows 公共工作流仓库

本仓库收集并维护了多种常用的 GitHub Actions 工作流，适用于自动化版本号管理、构建发布、静态资源上传、以及多平台通知等场景。所有工作流均可被其他项目直接复用，极大提升自动化运维效率。

---

## 目录

- [功能概览](#功能概览)
- [快速开始](#快速开始)
- [工作流说明](#工作流说明)
  - [1. 自动递增版本号（auto-increment-tag.yml）](#1-自动递增版本号auto-increment-tagyml)
  - [2. 发布 Release（deploy-release.yml）](#2-发布-releasedeploy-releaseyml)
  - [3. 上传 HTML 静态资源（upload-html.yml）](#3-上传-html-静态资源upload-htmlyml)
  - [4. 多平台通知（notify-third-party.yml）](#4-多平台通知notify-third-partyyml)
- [示例](#示例)
- [Webhook 配置](#webhook-配置)
- [常见问题](#常见问题)
- [贡献与许可](#贡献与许可)

---

## 功能概览

- **自动递增版本号**：支持主版本、功能、修复递增，自动打 tag，可选同步 package.json。
- **自动发布 Release**：自动打包构建产物并发布到 GitHub Release。
- **静态资源上传**：支持通过 SCP 上传 HTML 资源到服务器，并自动生成预览链接。
- **多平台通知**：支持企业微信、飞书、钉钉、Teams 等多平台消息通知，支持多平台并发推送。

---

## 快速开始

1. **Fork 或直接引用本仓库的 workflow 文件**
2. **在你的项目仓库配置所需的 Secrets（如 webhook、服务器信息等）**
3. **在你的 workflow 中通过 `uses` 语法调用本仓库的工作流**

---

## 工作流说明

### 1. 自动递增版本号（auto-increment-tag.yml）

**功能**：自动递增 Git tag 版本号，支持 `major`、`feat`、`fix`、`repeat` 四种模式，并可选同步 package.json。

**主要参数**：
- `version_type`：递增类型（major/feat/fix/repeat）
- `update_package_json_path`：可选，指定 package.json 路径

**输出**：
- `new_version`：递增后的新版本号

---

### 2. 发布 Release（deploy-release.yml）

**功能**：将构建产物打包并自动发布到 GitHub Release，支持自定义产物名。

**主要参数**：
- `version`：版本号（通常为 tag）
- `project_name`：可选，项目名
- `download_artifact_name`：产物目录名

---

### 3. 上传 HTML 静态资源（upload-html.yml）

**功能**：将构建好的 HTML 资源通过 SCP 上传到服务器，自动生成预览链接，并在生产环境下上报部署 API。

**主要参数**：
- `env`：环境类型（prod/uat）
- `version`：版本号
- `project_name`：可选，项目名
- `rm`：上传前是否清空目标目录
- `strip_components`：上传时去除的目录层级
- `download_artifact_name`：产物目录名

**Secrets**：
- `SSH_HOST`、`SSH_PORT`、`SSH_USERNAME`、`SSH_KEY`、`SERVER_HTML`（服务器信息）

**输出**：
- `preview_url`：预览链接
- `repository_path`：服务器路径

---

### 4. 多平台通知（notify-third-party.yml）

**功能**：支持企业微信、飞书、钉钉、Teams 多平台消息通知，支持多平台并发推送，支持自定义通知内容和状态。

**主要参数**：
- `env`：环境类型（prod/uat）
- `version`：版本号
- `preview_url`：可选，预览地址
- `project_name`：可选，项目名称
- `third_type`：通知平台（qywx, feishu, dingtalk, teams，可逗号分隔多平台）
- `status`：通知状态（success, failure, warning）

**Secrets**：
- `WEBHOOK_QYWX`、`WEBHOOK_FEISHU`、`WEBHOOK_DINGTALK`、`WEBHOOK_TEAMS`

---

## 示例

详见 [`examples/deploy-with-notify.yml`](examples/deploy-with-notify.yml) 和 [`examples/test-notify.yml`](examples/test-notify.yml)。

**多平台通知示例**：

```yaml
- name: Notify deployment
  uses: ./.github/workflows/notify-third-party.yml
  with:
    env: "prod"
    version: "v1.0.0"
    preview_url: "https://example.com"
    project_name: "我的项目"
    third_type: "qywx,feishu,dingtalk,teams"
    status: "success"
  secrets:
    WEBHOOK_QYWX: ${{ secrets.WEBHOOK_QYWX }}
    WEBHOOK_FEISHU: ${{ secrets.WEBHOOK_FEISHU }}
    WEBHOOK_DINGTALK: ${{ secrets.WEBHOOK_DINGTALK }}
    WEBHOOK_TEAMS: ${{ secrets.WEBHOOK_TEAMS }}
```

---

# Webhook 配置指南

本文档介绍如何为各种第三方平台配置webhook，以便接收GitHub Actions的部署通知。

## 企业微信 (QYWX)

### 1. 创建机器人
1. 在企业微信群中，点击右上角的设置图标
2. 选择"群机器人" → "添加机器人"
3. 选择"自定义"机器人
4. 设置机器人名称和头像

### 2. 获取Webhook地址
1. 创建完成后，复制webhook地址
2. 格式：`https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxxxxxxx`

### 3. 配置GitHub Secrets
在GitHub仓库的Settings > Secrets and variables > Actions中添加：
```
WEBHOOK_QYWX=https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxxxxxxx
```

## 飞书 (Feishu)

### 1. 创建机器人
1. 在飞书群中，点击右上角的设置图标
2. 选择"群设置" → "群机器人" → "添加机器人"
3. 选择"自定义机器人"
4. 设置机器人名称和头像

### 2. 获取Webhook地址
1. 创建完成后，复制webhook地址
2. 格式：`https://open.feishu.cn/open-apis/bot/v2/hook/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

### 3. 配置GitHub Secrets
在GitHub仓库的Settings > Secrets and variables > Actions中添加：
```
WEBHOOK_FEISHU=https://open.feishu.cn/open-apis/bot/v2/hook/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

## 钉钉 (DingTalk)

### 1. 创建机器人
1. 在钉钉群中，点击右上角的设置图标
2. 选择"群设置" → "智能群助手" → "添加机器人"
3. 选择"自定义"机器人
4. 设置机器人名称和头像

### 2. 获取Webhook地址
1. 创建完成后，复制webhook地址
2. 格式：`https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxx`

### 3. 配置GitHub Secrets
在GitHub仓库的Settings > Secrets and variables > Actions中添加：
```
WEBHOOK_DINGTALK=https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxx
```

## Microsoft Teams

### 1. 创建Incoming Webhook
1. 在Teams频道中，点击"..." → "管理连接器"
2. 找到"Incoming Webhook"并点击"配置"
3. 设置webhook名称和头像
4. 点击"创建"

### 2. 获取Webhook地址
1. 创建完成后，复制webhook地址
2. 格式：`https://xxxxx.webhook.office.com/webhookb2/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx@xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/IncomingWebhook/xxxxxxxx/xxxxxxxx`

### 3. 配置GitHub Secrets
在GitHub仓库的Settings > Secrets and variables > Actions中添加：
```
WEBHOOK_TEAMS=https://xxxxx.webhook.office.com/webhookb2/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx@xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/IncomingWebhook/xxxxxxxx/xxxxxxxx
```

## 安全配置

### 企业微信
- 支持IP白名单
- 支持关键词过滤
- 支持签名验证

### 飞书
- 支持IP白名单
- 支持签名验证
- 支持关键词过滤

### 钉钉
- 支持IP白名单
- 支持签名验证
- 支持关键词过滤

### Teams
- 支持IP白名单
- 支持身份验证

## 测试Webhook

### 使用curl测试
```bash
# 企业微信
curl -H 'Content-Type: application/json' \
     -X POST \
     -d '{"msgtype": "text", "text": {"content": "测试消息"}}' \
     https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxxxxxxx

# 飞书
curl -H 'Content-Type: application/json' \
     -X POST \
     -d '{"msg_type": "text", "content": {"text": "测试消息"}}' \
     https://open.feishu.cn/open-apis/bot/v2/hook/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

# 钉钉
curl -H 'Content-Type: application/json' \
     -X POST \
     -d '{"msgtype": "text", "text": {"content": "测试消息"}}' \
     https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxx

# Teams
curl -H 'Content-Type: application/json' \
     -X POST \
     -d '{"text": "测试消息"}' \
     https://xxxxx.webhook.office.com/webhookb2/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx@xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/IncomingWebhook/xxxxxxxx/xxxxxxxx
```

### 使用GitHub Actions测试
1. 在仓库中运行测试workflow：`examples/test-notify.yml`
2. 选择要测试的平台和状态
3. 检查是否收到通知消息

## 常见问题

- **消息未收到？** 检查 webhook 地址、机器人是否在群、Secrets 是否配置正确。
- **格式异常？** 检查参数内容、特殊字符、平台消息格式要求。
- **权限问题？** 检查机器人权限、群组设置、webhook 有效性。

### 1. 消息发送失败
- 检查webhook地址是否正确
- 确认机器人是否被添加到群组
- 检查IP白名单设置
- 查看GitHub Actions执行日志

### 2. 消息格式错误
- 确认webhook支持的消息格式
- 检查JSON格式是否正确
- 验证特殊字符是否被正确转义

### 3. 权限问题
- 确认机器人有发送消息权限
- 检查群组权限设置
- 验证webhook是否仍然有效

## 最佳实践

1. **安全性**
   - 定期轮换webhook地址
   - 使用IP白名单限制访问
   - 启用签名验证

2. **可靠性**
   - 配置多个通知平台作为备份
   - 监控webhook的可用性
   - 设置失败重试机制

3. **可维护性**
   - 使用有意义的机器人名称
   - 记录webhook配置信息
   - 定期测试通知功能

## 相关链接

- [企业微信机器人文档](https://developer.work.weixin.qq.com/document/path/91770)
- [飞书机器人文档](https://open.feishu.cn/document/ukTMukTMukTM/ucTM5YjL3ETO24yNxkjN)
- [钉钉机器人文档](https://open.dingtalk.com/document/robots/custom-robot-access)
- [Teams Incoming Webhook文档](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook) 

---

## 贡献与许可

- 欢迎提交 Issue 和 PR 共同完善本仓库。
- 代码遵循 MIT License。

如需更多帮助，请查阅各 workflow 文件头部注释、示例文件和文档目录。
