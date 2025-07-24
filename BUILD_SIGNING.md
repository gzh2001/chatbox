# 构建和代码签名说明

## 问题背景

在构建过程中，可能会遇到以下错误：

### 1. Windows代码签名错误
```
Cannot find module 'D:\a\chatbox\chatbox\custom_win_sign.js'
```

这是因为electron-builder配置文件中指定了自定义的Windows代码签名脚本，但该文件在CI环境中不存在。

### 3. GitHub Actions Artifact错误
```
Artifact not found for name: windows-build
```

这通常发生在以下情况：
- 构建job失败，没有生成artifact
- 构建输出路径配置错误
- GitHub Actions之间的权限问题
- 构建文件生成在错误的目录中

## 解决方案

### 1. 自定义签名脚本 (custom_win_sign.js)

我们创建了一个智能的签名脚本，它会：
- 在CI环境中跳过代码签名
- 在没有签名证书时跳过代码签名
- 只在有适当证书和环境时执行实际签名

### 2. 跨平台构建优化

为了避免平台特定的依赖问题：
- 移除了在单一环境中的跨平台构建 (`npm run package:all`)
- 改为使用各平台专用的构建环境
- 在macOS环境中添加了 `dmg-license` 依赖检查和安装

### 3. 构建输出路径修正

根据 `electron-builder.yml` 配置，构建输出目录是 `release/build/`，不是 `dist/`：
- 修正了所有artifact上传路径
- 添加了构建输出调试信息
- 实现了容错的artifact下载机制

### 4. 环境变量配置

在GitHub Actions中，我们使用以下环境变量来控制签名行为：
- `CSC_IDENTITY_AUTO_DISCOVERY: false` - 禁用自动证书发现
- `CI` - CI环境标识（自动设置）

### 3. 平台特定的签名设置

#### Windows签名
- 需要代码签名证书 (.p12文件或证书存储中的证书)
- 在生产环境中需要配置 `CSC_LINK` 和 `CSC_KEY_PASSWORD` 环境变量
- CI环境中会自动跳过签名

#### macOS签名和公证
- 需要Apple开发者证书
- 需要配置 `APPLE_ID`, `APPLE_ID_PASS`, `APPLE_TEAM_ID` 环境变量
- 如果环境变量不存在，会自动跳过公证步骤

## 本地开发

### 不进行代码签名的构建
```bash
# 设置环境变量跳过签名
export CSC_IDENTITY_AUTO_DISCOVERY=false
npm run package
```

### 进行代码签名的构建
如果你有代码签名证书，可以配置相应的环境变量：

#### Windows
```bash
# 设置证书路径和密码
export CSC_LINK=path/to/certificate.p12
export CSC_KEY_PASSWORD=your_certificate_password
npm run package
```

#### macOS
```bash
# 设置Apple开发者账户信息
export APPLE_ID=your_apple_id
export APPLE_ID_PASS=your_app_specific_password
export APPLE_TEAM_ID=your_team_id
npm run package
```

## GitHub Actions配置

所有的GitHub Actions工作流都已经配置为在CI环境中跳过代码签名，这样可以：
1. 避免签名相关的构建错误
2. 加快构建速度
3. 允许没有签名证书的贡献者也能运行构建

## 安全性说明

- 签名证书和密码等敏感信息应该存储在GitHub Secrets中
- `custom_win_sign.js` 文件会自动检查环境并跳过签名，不会暴露任何敏感信息
- 在生产发布时，应该使用有效的代码签名证书来确保应用的安全性和用户信任

## 故障排除

如果仍然遇到构建相关的错误：

### Windows签名问题
1. 确认 `custom_win_sign.js` 文件存在
2. 检查环境变量设置
3. 查看构建日志中的调试信息
4. 确认electron-builder.yml配置正确

### GitHub Actions Artifact问题
1. 检查构建job是否成功完成
2. 确认构建输出路径正确（应该是 `release/build/`）
3. 查看构建日志中的文件列表
4. 确认artifact上传步骤没有失败
5. 检查GitHub Actions的运行权限

### 构建输出路径问题
- 根据 `electron-builder.yml`，输出目录是 `release/build/`
- 确认 `directories.output` 配置正确
- 使用 `build-test.yml` 工作流来调试构建输出

### 通用构建问题
1. 检查Node.js版本是否与 `.node-version` 文件匹配
2. 确认所有依赖都已正确安装：`npm install`
3. 清理构建目录：`npm run clean` 或手动删除 `dist/` 和 `release/` 目录
4. 检查系统资源（磁盘空间、内存等）

## 生产发布

在生产环境中发布时，建议：
1. 使用有效的代码签名证书
2. 配置适当的环境变量
3. 测试签名和公证流程
4. 验证最终构建的安装包
