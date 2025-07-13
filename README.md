# workflows

## 公共工作流仓库

这是一个包含常用 GitHub Actions 工作流的公共仓库。

## 工作流列表

### 1. Auto Increment Tag (自动版本号递增)

**功能描述**: 自动递增 Git 标签版本号，支持语义化版本控制

**详细步骤**:
1. **获取最新标签**: 从 Git 仓库中获取最新的版本标签，如果没有则使用 `v0.0.0`
2. **版本号解析**: 将版本号拆分为主版本号(major)、功能版本号(feat)、修复版本号(fix)
3. **版本递增**: 根据指定的递增类型进行版本号更新：
   - `major`: 主版本号+1，功能版本号和修复版本号重置为0
   - `feat`: 功能版本号+1，修复版本号重置为0
   - `fix`: 修复版本号+1
   - `repeat`: 保持当前版本号不变
4. **更新 package.json**: 可选择更新指定路径的 package.json 文件中的版本号
5. **创建新标签**: 创建新的 Git 标签并推送到远程仓库

**输入参数**:
- `version_type`: 版本递增类型 (major/feat/fix/repeat)，默认: "fix"
- `update_package_json_path`: package.json 文件路径，可选

**输出参数**:
- `new_version`: 递增后的新版本号

### 2. Deploy Release (部署到 Release)

**功能描述**: 将构建产物打包并发布到 GitHub Release

**详细步骤**:
1. **获取仓库信息**: 确定仓库名称和生成带时间戳的压缩包名称
2. **下载构建产物**: 从之前的构建步骤中下载指定的构建产物
3. **打包压缩**: 将构建产物打包成 ZIP 文件
4. **创建 Release**: 在 GitHub 上创建新的 Release，包含详细的发布信息
5. **上传资源**: 将打包好的 ZIP 文件作为 Release 附件上传

**输入参数**:
- `version`: 版本号，必需
- `repository_name`: 仓库名称，可选，默认使用当前仓库名
- `download_dist_name`: 构建产物名称，默认: "dist"

**Release 信息包含**:
- 仓库名称和版本号
- 分支和提交信息
- 构建时间
- 下载链接

### 3. Upload Html (上传 HTML)

**功能描述**: 将构建的 HTML 文件上传到服务器并发送通知

**详细步骤**:
1. **下载构建产物**: 从之前的构建步骤中下载 HTML 文件
2. **确定部署路径**: 根据环境类型和仓库名称确定服务器上的部署路径
3. **上传文件**: 使用 SCP 将文件上传到指定服务器
4. **生成预览链接**: 根据部署路径生成预览 URL
5. **发送成功报告**: 向 API 发送部署成功报告（仅生产环境）
6. **发送企业微信通知**: 发送包含预览链接的通知消息

**输入参数**:
- `env`: 环境类型 (prod/uat)，必需
- `version`: 版本号，必需
- `repository_name`: 仓库名称，可选
- `rm`: SCP 参数，是否删除目标目录，默认: true
- `strip_components`: SCP 参数，去除的目录层级，默认: 1
- `download_dist_name`: 构建产物名称，默认: "dist"

**必需密钥**:
- `SSH_HOST`: SSH 服务器地址
- `SSH_PORT`: SSH 端口
- `SSH_USERNAME`: SSH 用户名
- `SSH_KEY`: SSH 私钥
- `SERVER_HTML`: 服务器 HTML 根目录
- `QYWX_WEBHOOK`: 企业微信机器人 Webhook

**预览 URL 规则**:
- 生产环境: `https://www.xshuliner.online/{repository_name}`
- UAT 环境: `https://www.xshuliner.online/uat/{repository_name}`
- 特殊项目 `project-pages` 生产环境: `https://www.xshuliner.online`

## 使用示例

### 在其他仓库中调用工作流

```yaml
# 在 .github/workflows/your-workflow.yml 中
jobs:
  build:
    # ... 构建步骤 ...
    
  deploy:
    needs: build
    uses: Xshuliner/workflows/.github/workflows/auto-increment-tag.yml@main
    with:
      version_type: "feat"
      update_package_json_path: "package.json"
    
  release:
    needs: [build, deploy]
    uses: Xshuliner//workflows/.github/workflows/deploy-release.yml@main
    with:
      version: ${{ needs.deploy.outputs.new_version }}
      download_dist_name: "dist"
    
  upload:
    needs: [build, deploy]
    uses: Xshuliner/workflows/.github/workflows/upload-html.yml@main
    with:
      env: "prod"
      version: ${{ needs.deploy.outputs.new_version }}
      download_dist_name: "dist"
    secrets:
      SSH_HOST: ${{ secrets.SSH_HOST }}
      SSH_PORT: ${{ secrets.SSH_PORT }}
      SSH_USERNAME: ${{ secrets.SSH_USERNAME }}
      SSH_KEY: ${{ secrets.SSH_KEY }}
      SERVER_HTML: ${{ secrets.SERVER_HTML }}
      QYWX_WEBHOOK: ${{ secrets.QYWX_WEBHOOK }}
```

## 注意事项

1. **权限要求**: 确保工作流有足够的权限访问仓库和创建 Release
2. **密钥配置**: 使用 HTML 上传功能前需要配置相应的 SSH 密钥和企业微信 Webhook
3. **版本管理**: 建议使用语义化版本号，遵循 major.feat.fix 格式
4. **环境区分**: 生产环境和 UAT 环境使用不同的部署路径和通知策略
