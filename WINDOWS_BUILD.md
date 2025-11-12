# Windows 打包指南

## 📦 打包方式

### 方式 1：在 Windows 系统上打包（推荐）

这是最可靠的方式，可以确保所有原生模块（如 robotjs）正确编译。

#### 前置要求

1. **Windows 10/11 系统**
2. **Node.js 16+**
   - 下载：https://nodejs.org/
3. **Visual Studio Build Tools**
   - 用于编译原生模块（robotjs）
4. **Git**（可选）

#### 安装构建工具

```powershell
# 方式 1: 使用 npm 安装（推荐）
npm install --global windows-build-tools

# 方式 2: 手动安装 Visual Studio Build Tools
# 下载: https://visualstudio.microsoft.com/downloads/
# 选择: "Desktop development with C++"
```

#### 打包步骤

```bash
# 1. 克隆项目（如果还没有）
git clone https://github.com/crispvibe/Windsurf-Tool.git
cd Windsurf-Tool

# 2. 安装依赖
npm install

# 3. 打包 Windows 版本
npm run build:win
```

#### 输出文件

打包完成后，在 `dist/` 目录下会生成：

```
dist/
├── Windsurf-Tool 1.0-1.0.0-x64.exe          # NSIS 安装程序
├── Windsurf-Tool-1.0.0-portable.exe         # 便携版（免安装）
└── win-unpacked/                             # 未打包的文件（用于调试）
```

### 方式 2：在 macOS/Linux 上交叉编译

可以在 macOS 或 Linux 上交叉编译 Windows 版本，但 **robotjs 等原生模块可能无法正常工作**。

#### 使用脚本打包

```bash
# 给脚本执行权限
chmod +x build-windows.sh

# 运行打包脚本
./build-windows.sh
```

#### 手动打包

```bash
npm run build:win
```

#### ⚠️ 注意事项

1. **robotjs 问题**
   - 在 macOS/Linux 上打包的 Windows 版本，robotjs 可能无法工作
   - 需要在 Windows 上重新安装：`npm rebuild robotjs --runtime=electron --target=27.1.0`

2. **测试**
   - 必须在 Windows 系统上测试打包后的应用
   - 确保所有功能正常

## 🔧 打包配置说明

### package.json 配置

```json
{
  "build": {
    "win": {
      "target": [
        { "target": "nsis", "arch": ["x64"] },      // 安装程序
        { "target": "portable", "arch": ["x64"] }   // 便携版
      ],
      "icon": "build/icon.ico",
      "requestedExecutionLevel": "requireAdministrator"
    },
    "nsis": {
      "oneClick": false,                            // 允许自定义安装路径
      "allowToChangeInstallationDirectory": true,
      "perMachine": true,                           // 为所有用户安装
      "createDesktopShortcut": true,
      "createStartMenuShortcut": true,
      "shortcutName": "Windsurf-Tool"
    }
  }
}
```

### 打包目标

1. **NSIS 安装程序**
   - 标准的 Windows 安装程序
   - 支持自定义安装路径
   - 创建桌面和开始菜单快捷方式
   - 支持卸载

2. **便携版 (Portable)**
   - 单个 EXE 文件
   - 无需安装，直接运行
   - 适合 U 盘使用
   - 配置保存在程序目录

## 🐛 常见问题

### 1. robotjs 编译失败

**问题：**
```
Error: Could not locate the bindings file
```

**解决方案：**

```bash
# 方式 1: 重新安装 robotjs
npm uninstall robotjs
npm install robotjs

# 方式 2: 使用 electron-rebuild
npm install --save-dev electron-rebuild
npx electron-rebuild

# 方式 3: 手动重建
npm rebuild robotjs --runtime=electron --target=27.1.0 --disturl=https://electronjs.org/headers
```

### 2. 缺少 Visual Studio Build Tools

**问题：**
```
error MSB8036: The Windows SDK version 10.0 was not found
```

**解决方案：**

```powershell
# 安装构建工具
npm install --global windows-build-tools

# 或手动安装
# https://visualstudio.microsoft.com/downloads/
# 选择 "Desktop development with C++"
```

### 3. 打包后应用无法启动

**可能原因：**
- robotjs 未正确编译
- 缺少运行时依赖
- 权限不足

**解决方案：**

1. **检查 robotjs**
   ```bash
   # 在 Windows 上重新编译
   npm rebuild robotjs --runtime=electron --target=27.1.0
   ```

2. **以管理员身份运行**
   - 右键点击应用
   - 选择"以管理员身份运行"

3. **查看日志**
   - 打开开发者工具：`Ctrl+Shift+I`
   - 查看控制台错误信息

### 4. 防病毒软件误报

**问题：**
- Windows Defender 阻止运行
- 其他杀毒软件报毒

**解决方案：**

1. **添加到白名单**
   - Windows Defender: 设置 → 病毒和威胁防护 → 管理设置 → 添加排除项

2. **代码签名**（可选）
   - 购买代码签名证书
   - 使用 electron-builder 签名

### 5. 打包体积过大

**优化方案：**

1. **启用 asar 打包**
   ```json
   {
     "build": {
       "asar": true
     }
   }
   ```

2. **排除不必要的文件**
   ```json
   {
     "build": {
       "files": [
         "!**/*.map",
         "!**/node_modules/*/{CHANGELOG.md,README.md,README,readme.md,readme}",
         "!**/node_modules/.bin"
       ]
     }
   }
   ```

## 📋 打包检查清单

### 打包前

- [ ] 所有依赖已安装：`npm install`
- [ ] 代码已测试通过
- [ ] 版本号已更新：`package.json` 中的 `version`
- [ ] 图标文件已准备：`build/icon.ico`
- [ ] LICENSE 文件存在

### 打包中

- [ ] 构建工具已安装
- [ ] robotjs 编译成功
- [ ] 无编译错误
- [ ] 无警告信息

### 打包后

- [ ] 安装程序可以正常安装
- [ ] 便携版可以直接运行
- [ ] 应用界面正常显示
- [ ] 所有功能正常工作
- [ ] robotjs 键盘模拟正常
- [ ] 窗口检测正常
- [ ] 进程管理正常
- [ ] 配置文件读写正常

## 🚀 发布流程

### 1. 准备发布

```bash
# 更新版本号
npm version patch  # 1.0.0 -> 1.0.1
# 或
npm version minor  # 1.0.0 -> 1.1.0
# 或
npm version major  # 1.0.0 -> 2.0.0

# 提交更改
git add .
git commit -m "Release v1.0.1"
git push origin main
```

### 2. 打包

```bash
# 在 Windows 上打包
npm run build:win
```

### 3. 测试

- 在 Windows 10 上测试
- 在 Windows 11 上测试
- 测试所有核心功能
- 测试边缘情况

### 4. 创建 GitHub Release

```bash
# 创建 tag
git tag v1.0.1
git push origin v1.0.1
```

然后在 GitHub 上：
1. 访问：https://github.com/crispvibe/Windsurf-Tool/releases/new
2. 选择 tag: `v1.0.1`
3. 填写 Release 标题和说明
4. 上传打包文件：
   - `Windsurf-Tool 1.0-1.0.1-x64.exe` (NSIS 安装程序)
   - `Windsurf-Tool-1.0.1-portable.exe` (便携版)
5. 发布 Release

### 5. 更新 README

更新 README.md 中的下载链接：

```markdown
| Windows | x64 | [Windsurf-Tool-1.0.1.exe](https://github.com/crispvibe/Windsurf-Tool/releases/download/v1.0.1/Windsurf-Tool-1.0.1-x64.exe) |
```

## 📝 打包命令速查

```bash
# 安装依赖
npm install

# 安装构建工具（Windows）
npm install --global windows-build-tools

# 打包 Windows 版本
npm run build:win

# 重新编译 robotjs
npm rebuild robotjs --runtime=electron --target=27.1.0

# 清理并重新打包
rm -rf dist node_modules
npm install
npm run build:win

# 查看打包输出
ls -lh dist/
```

## 🔗 相关资源

- [electron-builder 文档](https://www.electron.build/)
- [robotjs 文档](http://robotjs.io/)
- [Windows Build Tools](https://github.com/felixrieseberg/windows-build-tools)
- [Electron 文档](https://www.electronjs.org/docs/latest/)

## 💡 最佳实践

1. **始终在目标平台上打包**
   - Windows 应用在 Windows 上打包
   - 避免交叉编译问题

2. **测试多个 Windows 版本**
   - Windows 10 (1909+)
   - Windows 11

3. **提供多种安装方式**
   - NSIS 安装程序（推荐）
   - 便携版（高级用户）

4. **签名你的应用**
   - 减少防病毒软件误报
   - 提升用户信任度

5. **保持依赖更新**
   - 定期更新 Electron
   - 更新安全补丁

## ⚠️ 重要提示

1. **robotjs 是关键依赖**
   - 必须在 Windows 上正确编译
   - 打包前务必测试

2. **管理员权限**
   - 某些功能需要管理员权限
   - 安装程序会请求权限

3. **防病毒软件**
   - 可能误报为病毒
   - 建议代码签名

4. **测试充分**
   - 在真实 Windows 环境测试
   - 测试所有功能

## 📞 获取帮助

如果遇到问题：
1. 查看本文档的"常见问题"部分
2. 查看 GitHub Issues
3. 加入 QQ 群交流
