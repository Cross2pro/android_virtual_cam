# GitHub Actions 工作流说明 / Workflows Documentation

## Android Build 工作流 / Android Build Workflow

此工作流自动构建、签名并发布 Android 虚拟摄像头 APK。

This workflow automatically builds, signs, and publishes the Android Virtual Cam APK.

### 触发条件 / Triggers

- **Push**: 任何分支的推送 / Any push to any branch
- **Pull Request**: 任何 Pull Request / Any pull request
- **Tag Push**: 推送标签以创建 Release / Push tags to create releases

### 前置要求 / Prerequisites

工作流需要以下 GitHub Secrets 来签名 APK：

The workflow requires the following GitHub Secrets to sign the APK:

1. `ANDROID_KEYSTORE_BASE64` - Base64 编码的密钥库文件 / Base64-encoded keystore file
2. `ANDROID_KEYSTORE_PASSWORD` - 密钥库密码 / Keystore password
3. `ANDROID_KEY_ALIAS` - 密钥别名 / Key alias
4. `ANDROID_KEY_PASSWORD` - 密钥密码 / Key password

### 配置 Secrets / Setting up Secrets

1. 生成或使用现有的 Android 密钥库 / Generate or use an existing Android keystore
2. 将密钥库文件转换为 Base64：
   ```bash
   base64 -i your-keystore.jks | tr -d '\n'
   ```
3. 在 GitHub 仓库设置中添加 Secrets：
   - Settings → Secrets and variables → Actions → New repository secret

### 工作流程 / Workflow Steps

1. **检出代码** / Checkout code
2. **设置 JDK 11** / Setup JDK 11
3. **验证签名密钥** / Validate signing secrets
   - 检查所有必需的密钥是否配置 / Check if all required secrets are configured
4. **解码密钥库** / Decode keystore (if secrets available)
5. **构建签名 APK** / Build signed APK (if secrets available)
   ```bash
   ./gradlew clean assembleRelease
   ```
6. **验证 APK** / Verify APK was built
7. **提取版本信息** / Extract version information
8. **上传 Artifact** / Upload artifact
   - 名称格式 / Name format: `android-virtual-cam-v{version}-signed`
   - 在每次成功构建后都会上传 / Uploaded after every successful build
9. **创建 GitHub Release** / Create GitHub Release (only on tag push)
   - 仅在推送标签时触发 / Only triggered when pushing tags
   - 包含详细的发布说明 / Contains detailed release notes

### Artifacts

每次成功构建后，签名的 APK 会作为 Artifact 上传：

After every successful build, the signed APK is uploaded as an Artifact:

- **名称** / Name: `android-virtual-cam-v{version}-signed`
- **文件** / File: `app-release.apk`
- **保留时间** / Retention: 90 天（GitHub 默认）/ 90 days (GitHub default)

要下载 Artifact：
1. 进入 Actions 标签页 / Go to Actions tab
2. 选择工作流运行 / Select a workflow run
3. 在 "Artifacts" 部分下载 / Download from "Artifacts" section

### Releases

要创建 GitHub Release：

To create a GitHub Release:

1. 创建并推送标签 / Create and push a tag:
   ```bash
   git tag v4.4
   git push origin v4.4
   ```

2. GitHub Actions 会自动：
   - 构建签名 APK / Build signed APK
   - 创建 Release / Create release
   - 上传 APK 到 Release / Upload APK to release
   - 添加详细的发布说明 / Add detailed release notes

### 故障排除 / Troubleshooting

#### 未生成 Artifacts / No Artifacts Generated

**问题** / Problem: 工作流运行成功但没有 Artifacts

**原因** / Cause:
- 签名密钥未配置 / Signing secrets not configured
- APK 构建失败 / APK build failed

**解决方案** / Solution:
1. 检查工作流日志中的 "Validate signing secrets" 步骤
2. 确保所有 4 个密钥都已配置 / Ensure all 4 secrets are configured
3. 检查构建日志是否有错误 / Check build logs for errors

#### 未创建 Release / No Release Created

**问题** / Problem: 推送代码但未创建 Release

**原因** / Cause: Release 仅在推送标签时创建

**解决方案** / Solution:
1. 创建标签 / Create a tag: `git tag v4.5`
2. 推送标签 / Push tag: `git push origin v4.5`
3. 检查 Actions 标签页确认工作流运行

#### APK 签名失败 / APK Signing Failed

**问题** / Problem: 构建失败并显示签名错误

**解决方案** / Solution:
1. 验证密钥库密码正确 / Verify keystore password is correct
2. 验证密钥别名存在 / Verify key alias exists
3. 确保 Base64 编码正确（无换行）/ Ensure Base64 encoding is correct (no newlines)

### 版本管理 / Version Management

版本在 `app/build.gradle` 中定义：

Version is defined in `app/build.gradle`:

```gradle
versionCode 28
versionName "4.4"
```

发布新版本时：
1. 更新 `app/build.gradle` 中的版本号 / Update version in `app/build.gradle`
2. 提交更改 / Commit changes
3. 创建对应的标签 / Create corresponding tag
4. 推送代码和标签 / Push code and tag

### 手动运行 / Manual Trigger

目前工作流不支持手动触发。如需手动触发，可以在工作流文件中添加：

Currently, manual triggering is not supported. To enable it, add to the workflow file:

```yaml
on:
  push:
  pull_request:
  workflow_dispatch:  # 添加此行 / Add this line
```

## 相关链接 / Related Links

- [GitHub Actions 文档](https://docs.github.com/actions)
- [Android Signing 文档](https://developer.android.com/studio/publish/app-signing)
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release)
