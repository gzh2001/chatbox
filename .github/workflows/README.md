# GitHub Actions 工作流说明

本目录包含了Chatbox项目的GitHub Actions工作流配置文件，用于自动化构建、测试、代码质量检查和发布流程。

## 工作流文件说明

### 1. `ci.yml` - 持续集成
**触发条件：**
- 推送到 `main` 或 `develop` 分支
- 针对 `main` 或 `develop` 分支的Pull Request

**功能：**
- 运行代码检查（lint）
- 运行TypeScript类型检查
- 运行测试用例
- 基础构建验证
- 构建Web版本

### 2. `build-release.yml` - 构建和发布
**触发条件：**
- 推送带有版本标签的代码（如 `v1.0.0`）
- 手动触发（workflow_dispatch）

**功能：**
- 为Windows、macOS、Linux三个平台分别构建
- 支持跨平台构建（package:all）
- 自动创建GitHub Release
- 上传构建产物到Release

### 3. `dev-build.yml` - 开发构建
**触发条件：**
- 推送到 `develop` 分支
- 针对 `develop` 分支的Pull Request
- 手动触发

**功能：**
- 在多个操作系统上测试构建
- 验证开发模式启动
- 为当前平台打包
- 上传开发构建产物

### 4. `code-quality.yml` - 代码质量检查
**触发条件：**
- 推送到 `main` 或 `develop` 分支
- 针对 `main` 或 `develop` 分支的Pull Request

**功能：**
- ESLint代码检查
- TypeScript类型检查
- Prettier格式检查
- 测试覆盖率检查
- 安全漏洞扫描
- 依赖项审查（仅PR）

### 5. `web-deploy.yml` - Web版本构建和部署
**触发条件：**
- 推送到 `main` 分支（排除文档更改）
- 手动触发

**功能：**
- 构建Web版本
- 可选择部署到GitHub Pages
- 测试Web版本服务

### 6. `build-test.yml` - 构建测试工作流
**触发条件：**
- 手动触发
- 推送到 `main` 分支（仅当工作流文件或构建配置更改时）

**功能：**
- 在所有平台上测试构建过程
- 调试构建输出和文件路径
- 验证构建配置正确性
- 不进行打包，仅测试编译

## 使用说明

### 本地开发工作流
1. 在 `develop` 分支上开发
2. 创建Pull Request到 `develop` 分支
3. CI会自动运行测试和代码质量检查
4. 合并后会触发开发构建

### 发布工作流
1. 将 `develop` 分支合并到 `main` 分支
2. 创建版本标签：`git tag v1.0.0 && git push origin v1.0.0`
3. 自动触发构建和发布流程
4. 查看GitHub Releases页面获取构建产物

### 手动触发构建
可以在GitHub网页的Actions页面手动触发以下工作流：
- Build and Release（构建发布）
- Development Build（开发构建）
- Web Build and Deploy（Web构建部署）

## 环境要求

- Node.js版本：根据 `.node-version` 文件（当前为v22.7.0）
- 操作系统：支持Ubuntu、Windows、macOS
- 依赖：npm包管理器

## 注意事项

1. **安全性**：所有敏感信息都应使用GitHub Secrets存储
2. **权限**：发布工作流需要GITHUB_TOKEN权限
3. **缓存**：使用npm缓存加速构建过程
4. **产物保留**：构建产物保留7-30天不等
5. **错误处理**：部分步骤允许失败并继续执行
6. **代码签名**：CI环境中会自动跳过代码签名以避免构建错误

## 代码签名和构建问题解决

如果遇到以下构建错误，这些都是已知问题并已解决：

### Windows签名错误
```
Cannot find module 'custom_win_sign.js'
```

### macOS DMG构建错误
```  
Cannot find module 'dmg-license'
```

我们已经采取了以下措施：

1. 创建了智能的签名脚本 `custom_win_sign.js`
2. 在所有CI环境中设置 `CSC_IDENTITY_AUTO_DISCOVERY: false`
3. 移除了跨平台构建以避免平台特定依赖问题
4. 为macOS构建添加了依赖检查和修复
5. 添加了详细的调试信息以便排查问题

### GitHub Actions Artifact错误
```
Artifact not found for name: windows-build
```

我们已经采取了以下措施：

1. 修正了构建输出路径（从 `dist/` 改为 `release/build/`）
2. 为每个构建job添加了详细的输出调试
3. 实现了容错的artifact下载机制
4. 添加了 `build-test.yml` 工作流用于调试构建问题
5. 添加了 `if-no-files-found: warn` 选项以便更好地诊断问题

详细信息请参考项目根目录下的 `BUILD_SIGNING.md` 文件。

## 自定义配置

如需修改工作流行为，可以调整以下配置：
- Node.js版本：修改 `.node-version` 文件
- 构建平台：修改 `build-release.yml` 中的平台配置
- 部署域名：在 `web-deploy.yml` 中设置CNAME
- 代码质量规则：修改相应的配置文件（.eslintrc.js、.prettierrc等）
