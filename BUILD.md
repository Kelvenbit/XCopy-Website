# 构建和运行指南

本文档解释如何构建、运行和测试 XCopy。

## 目录

- [前置要求](#前置要求)
- [构建项目](#构建项目)
- [运行应用](#运行应用)
- [免费分发](#免费分发)
- [清理构建](#清理构建)
- [故障排除](#故障排除)
- [开发工作流](#开发工作流)

---

## 前置要求

### 必需软件

| 软件 | 最低版本 | 推荐 |
|----------|-----------------|-------------|
| **Xcode** | 15.0+ | 26.2（最新） |
| **macOS** | 13.5 Ventura | 最新（Sonoma/Sequoia） |
| **Swift** | 5.9+ | Xcode 捆绑 |

### Xcode 组件

安装 Xcode 时，确保安装了以下组件：
- Xcode IDE
- 命令行工具
- Simulator（用于 iOS 测试，如需要）
- Platforms（macOS SDK）

### 验证安装

```bash
# 检查 Xcode 版本
xcodebuild -version

# 检查 Swift 版本
swift --version

# 验证 Xcode 工具
xcode-select -p
```

**预期输出**：
```
Xcode 26.2
Build version 26...

Swift version 6.0...
```

---

## 构建项目

### 在 Xcode 中打开（推荐）

```bash
# 导航到项目目录
cd /path/to/XCopy

# 在 Xcode 中打开
open XCopy.xcodeproj
# 或
open XCopy.xcworkspace
```

**步骤**：
1. 等待 Xcode 索引项目
2. 选择"XCopy" scheme
3. 选择目标设备（My Mac）
4. 按 ??R 或 Product ??? Run

### 从命令行构建

#### Debug 构建

```bash
xcodebuild -project XCopy.xcodeproj \
           -scheme XCopy \
           -configuration Debug \
           build
```

**输出位置**：`~/Library/Developer/Xcode/DerivedData/XCopy-*/Build/Products/Debug/`

#### Release 构建

```bash
xcodebuild -project XCopy.xcodeproj \
           -scheme XCopy \
           -configuration Release \
           build
```

**输出位置**：`~/Library/Developer/Xcode/DerivedData/XCopy-*/Build/Products/Release/`

#### 自定义输出构建

```bash
# 构建到特定目录
xcodebuild -project XCopy.xcodeproj \
           -scheme XCopy \
           -configuration Release \
           -archiveBuild ./build/XCopy.app
```

### 构建选项

| 选项 | 描述 |
|--------|-------------|
| `-project` | `.xcodeproj` 文件路径 |
| `-scheme` | Build scheme 名称 |
| `-configuration` | Debug 或 Release |
| `-sdk` | 目标 SDK（macosx、macosx13.5 等） |
| `-jobs` | 并行作业数 |
| `-quiet` | 抑制输出 |
| `-verbose` | 详细输出 |

---

## 运行应用

### 从 Xcode

1. 选择 **My Mac** 作为目标
2. 按 **??R** 或点击 **Run** 按钮
3. 在提示时授予必要权限：
   - Accessibility（用于全局快捷键）
   - Files and Folders（用于文件剪贴板）

### 从命令行

#### 运行 Debug 构建

```bash
# 导航到构建输出
cd ~/Library/Developer/Xcode/DerivedData/XCopy-*/Build/Products/Debug

# 运行应用
open XCopy.app
```

#### 运行 Release 构建

```bash
# 导航到构建输出
cd ~/Library/Developer/Xcode/DerivedData/XCopy-*/Build/Products/Release

# 运行应用
open XCopy.app
```

#### 带参数运行

```bash
# 使用特定参数打开
./XCopy.app/Contents/MacOS/XCopy --argument=value
```

### 从 Archive 运行

```bash
# 构建 archive
xcodebuild -project XCopy.xcodeproj \
           -scheme XCopy \
           -archiveArchive ./build/XCopy.xcarchive

# 导出 archive
xcodebuild -exportArchive \
           -archivePath ./build/XCopy.xcarchive \
           -exportPath ./build/ \
           -exportOptionsPlist ExportOptions.plist
```

---

## 免费分发

XCopy 当前采用免费完整功能分发：GitHub Pages 承载下载网站，GitHub Releases 承载 DMG 文件。

### 生成网站直接下载 DMG

```bash
./scripts/package_docs_dmg.sh
```

### 发布要求

网站下载按钮会固定访问：

```text
https://raw.githubusercontent.com/Kelvenbit/XCopy-Website/aebdef97fd9e8de8287d3f0dfd8ecf8e408bee67/assets/XCopy.dmg
```

对应仓库文件为 raw GitHub 固定文件 `XCopy.dmg`。

完整流程见 [DISTRIBUTION.md](DISTRIBUTION.md)。

---

## 清理构建

### 清理构建文件夹

```bash
# 通过 xcodebuild 清理
xcodebuild -project XCopy.xcodeproj \
           -scheme XCopy \
           clean
```

### 清理构建文件夹（Shift + ??K 等效）

```bash
# 清理构建文件夹（更彻底）
xcodebuild -project XCopy.xcodeproj \
           -scheme XCopy \
           clean build-folder
```

### 手动清理

```bash
# 删除 DerivedData
rm -rf ~/Library/Developer/Xcode/DerivedData/XCopy-*

# 删除构建目录（如果存在）
rm -rf build/

# 清理 Xcode 缓存
rm -rf ~/Library/Caches/com.apple.dt.Xcode
```

---

## 运行自动化测试

### 最基础 UI Smoke Test

项目现在支持一个可执行的 macOS UI 自动化流程：

- 应用以 `--uitesting --reset-app` 启动
- 可按场景打开 Settings 或预置带样例数据的 `FloatPanel`
- UI 测试验证关键设置页元素是否出现
- UI 测试验证可以切换到 Hotkeys 页面
- UI 测试验证 `FloatPanel` 快捷键拉起、卡片快捷键操作、卡片右键收藏、收藏夹新增/重命名/删除

运行命令：

```bash
./scripts/run_basic_ui_tests.sh
```

或直接执行：

```bash
xcodebuild -project XCopy.xcodeproj \
           -scheme XCopy \
           -destination 'platform=macOS' \
           -derivedDataPath .derivedData \
           -only-testing:XCopyUITests/XCopyUITests \
           test
```

---

## 故障排除

### 常见构建错误

#### 1. "No such module 'ModuleName'"

**原因**：缺少依赖或 Swift Package Manager 缓存问题。

**解决方案**：
```bash
# 重置包缓存
rm -rf ~/Library/Developer/Xcode/DerivedData/*/SourcePackages

# 再次解析包
xcodebuild -resolvePackageDependencies -project XCopy.xcodeproj

# 清理并重新构建
xcodebuild clean build -project XCopy.xcodeproj -scheme XCopy
```

#### 2. "Code signing failed"

**原因**：证书或配置文件问题。

**解决方案**：
```bash
# 检查可用的签名证书
security find-identity -v -p codesigning

# 在 Xcode 中更新代码签名设置：
# Project ??? Target ??? Signing & Capabilities
# 选择"Automatically manage signing"
# 选择您的开发团队
```

#### 3. "Command PhaseScriptExecution failed"

**原因**：构建阶段脚本错误或权限。

**解决方案**：
```bash
# 使构建脚本可执行
find . -name "*.sh" -exec chmod +x {} \;

# 清理并重新构建
xcodebuild clean build -project XCopy.xcodeproj -scheme XCopy
```

#### 4. "Cannot find interface declaration"

**原因**：过时的构建缓存或 DerivedData。

**解决方案**：
```bash
# 清理 DerivedData
rm -rf ~/Library/Developer/Xcode/DerivedData/*

# 重新构建
xcodebuild build -project XCopy.xcodeproj -scheme XCopy
```

### 运行时问题

#### 1. 应用启动时崩溃

**调试**：
```bash
# 使用崩溃日志运行
Console.app ??? User Reports ??? 选择崩溃日志

# 使用 LLDB 调试器运行
lldb ./XCopy.app/Contents/MacOS/XCopy
(lldb) run
(lldb) bt  # 崩溃时的 backtrace
```

#### 2. 剪贴板未检测到

**原因**：缺少 accessibility 权限。

**解决方案**：
1. 打开系统设置 ??? 隐私与安全性
2. 找到"Accessibility"
3. 添加 XCopy.app 并启用它
4. 重启应用

#### 3. 全局快捷键不工作

**原因**：缺少 accessibility 权限或快捷键冲突。

**解决方案**：
1. 检查 Accessibility 权限
2. 在系统设置 ??? 键盘中检查快捷键冲突
3. 尝试不同的快捷键组合

---

## 开发工作流

### 典型开发周期

```
1. 编辑源代码
2. 构建（??B）或构建并运行（??R）
3. 测试更改
4. 如有必要进行调试
5. 提交更改
6. 重复
```

### Xcode 中的键盘快捷键

| 快捷键 | 操作 |
|----------|--------|
| ??B | 构建 |
| ??R | 运行 |
| ??Shift+K | 清理构建文件夹 |
| ??U | 测试 |
| ??G | 使用 Instruments 运行 |
| ??+ | 增大字体大小 |
| ??- | 减小字体大小 |
| ??0 | 显示标准编辑器 |
| ??+ ??+ + Left | 显示左侧助手 |
| ??+ ??+ + Right | 显示右侧助手 |

### 调试

#### 断点

```swift
// 通过点击行号设置断点
// 或使用 lldb 命令：
(lldb) breakpoint set --file ClipboardViewModel.swift --line 42
```

#### LLDB 命令

| 命令 | 描述 |
|---------|-------------|
| `run` | 开始执行 |
| `continue` | 继续到下一个断点 |
| `step` | 进入函数 |
| `next` | 跳过函数 |
| `print variable` | 打印变量值 |
| `po object` | 打印对象描述 |
| `bt` | 打印 backtrace |
| `frame variable` | 显示所有局部变量 |

#### 日志记录

```swift
import os.log

let log = OSLog(subsystem: "com.kt.xcopy", category: "Clipboard")

os_log("剪贴板已更改：%@", log: log, type: .info, item.content)

// 或使用 OSLogLogger
Logger(subsystem: "com.kt.xcopy", category: "Clipboard")
    .info("剪贴板已更改：\(item.content)")
```

### 测试

#### 运行单元测试

```bash
# 运行所有测试
xcodebuild test -project XCopy.xcodeproj -scheme XCopy

# 运行特定测试
xcodebuild test -project XCopy.xcodeproj \
                -scheme XCopy \
                -only-testing:XCopyTests/ClipboardViewModelTests

# 运行带代码覆盖率的测试
xcodebuild test -project XCopy.xcodeproj \
                -scheme XCopy \
                -enableCodeCoverage YES
```

#### 查看测试结果

```bash
# 在 Xcode 中打开测试结果
open ~/Library/Developer/Xcode/DerivedData/*/Logs/Test/*.xcresult
```

---

## 构建配置

### Debug 与 Release

| 设置 | Debug | Release |
|---------|-------|---------|
| **优化** | 无 (-Onone) | 优化速度 (-O) |
| **调试信息** | 完整 | Dwarf with dSYM |
| **断言** | 启用 | 禁用 |
| **剥离符号** | 否 | 是 |
| **大小** | 较大 | 较小 |
| **性能** | 较慢 | 较快 |

### 自定义构建设置

您可以在 Xcode 中修改构建设置：
1. 在导航器中选择项目
2. 选择目标
3. 转到"Build Settings"选项卡
4. 根据需要修改设置

**常用设置**：

| 设置 | 描述 | 默认 |
|---------|-------------|---------|
| `SWIFT_VERSION` | Swift 语言版本 | 6.0 |
| `MACOSX_DEPLOYMENT_TARGET` | 最低 macOS 版本 | 13.5 |
| `ENABLE_PREVIEWS` | 启用 SwiftUI 预览 | YES（Debug） |
| `SWIFT_ACTIVE_COMPILATION_CONDITIONS` | 条件编译标志 | DEBUG |
| `PRODUCT_BUNDLE_IDENTIFIER` | App bundle ID | KT.XCopy |

---

## 签名和分发

### 代码签名

XCopy 使用自动代码签名：

```bash
# 查看当前签名
codesign -dvvv ./XCopy.app

# 验证签名
codesign -v ./XCopy.app

# 如需要重新签名
codesign --force --deep --sign "Developer ID Application: Your Name" ./XCopy.app
```

### 公证（用于分发）

```bash
# 公证应用
xcrun notarytool submit ./XCopy.app \
    --apple-id "your@email.com" \
    --password "app-specific-password" \
    --team-id "YOUR_TEAM_ID" \
    --wait

# 装订票据
xcrun stapler staple ./XCopy.app
```

### 创建 DMG

```bash
# 创建 DMG 分发
hdiutil create -volname "XCopy" \
               -srcfolder ./XCopy.app \
               -ov -format UDZO \
               ./XCopy.dmg
```

---

## 性能分析

### Instruments

```bash
# 使用 Instruments 运行
instruments -t "Time Profiler" \
           ./XCopy.app/Contents/MacOS/XCopy

# 或从 Xcode 启动：
# Product ??? Profile (??I)
```

### 常用 Instruments 模板

| 模板 | 用例 |
|----------|----------|
| **Time Profiler** | CPU 使用、调用栈 |
| **Allocations** | 内存分配、泄漏 |
| **Leaks** | 内存泄漏检测 |
| **Core Data** | 数据库性能 |
| **System Trace** | 系统级性能 |

---

## 提示和最佳实践

1. **定期清理构建**：特别是在切换分支时
2. **使用 DerivedData**：不要修改 DerivedData 中的文件
3. **Gitignore DerivedData**：添加到 `.gitignore`
4. **发布前 Archive**：创建可分发的应用
5. **在真实硬件上测试**：macOS 没有模拟器
6. **保持 Xcode 更新**：最新版本有更好的 Swift 支持
7. **监控 Console**：检查运行时错误和警告

---

## 相关文档

- [ARCHITECTURE.md](ARCHITECTURE.md) - 架构概述
- [DATA_MODELS.md](DATA_MODELS.md) - 数据模型文档
- [UI_DESIGN.md](UI_DESIGN.md) - UI 规范
- [ROADMAP.md](ROADMAP.md) - 开发路线图
- [DEPENDENCIES.md](DEPENDENCIES.md) - 外部依赖
