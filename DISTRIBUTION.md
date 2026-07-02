# 免费分发流程

本文档记录 XCopy 离开 Mac App Store 后的免费分发方案。目标是保持链路简单、透明、可重复：网站和安装包都托管在 GitHub Pages 对应的 仓库根目录 目录中，应用完整功能免费开放。

## 当前决策

- 不接入账号、订阅、购买、授权码或设备绑定。
- `index.html` 是 GitHub Pages 首页。
- 最新下载按钮固定指向：

```text
https://raw.githubusercontent.com/Kelvenbit/XCopy-Website/aebdef97fd9e8de8287d3f0dfd8ecf8e408bee67/assets/XCopy.dmg
```

- 每次发布前，必须重新生成 raw GitHub 固定文件 `XCopy.dmg`。
- DMG 应优先使用 Developer ID 签名并完成 Apple 公证，降低用户首次打开时的安全警告。

## GitHub Pages 配置

在 GitHub 仓库中开启 Pages：

1. 打开 `Settings`。
2. 进入 `Pages`。
3. Source 选择 `Deploy from a branch`。
4. Branch 选择 `main`，目录选择 `/`。
5. 保存后等待 GitHub 生成站点。

默认站点地址：

```text
https://kelvenbit.github.io/XCopy-Website/
```

如后续绑定独立域名，只需要在 GitHub Pages 配置 custom domain，不需要改应用代码。

## 打包 DMG

当前网站直接下载文件位于：

```text
assets/XCopy.dmg
```

生成最新 DMG：

```bash
./scripts/package_docs_dmg.sh
```

这个脚本会执行 Release 构建，并把最新安装包覆盖写入 raw GitHub 固定文件 `XCopy.dmg`。

## 正式公证包

推荐使用现有脚本生成签名和公证后的 DMG：

```bash
./scripts/build_notarize_dmg.sh --keychain-profile XCopy
```

如果只需要本地验证打包流程，可以跳过公证：

```bash
./scripts/build_notarize_dmg.sh --skip-notarize
```

跳过公证的 DMG 不适合正式公开分发。正式 Release 应使用已公证版本。

## 发布 GitHub Pages

1. 使用 `./scripts/package_docs_dmg.sh` 生成最新 DMG。
2. 确认 `index.html` 下载按钮指向 `https://raw.githubusercontent.com/Kelvenbit/XCopy-Website/aebdef97fd9e8de8287d3f0dfd8ecf8e408bee67/assets/XCopy.dmg`。
3. 提交并推送 `index.html`、raw GitHub 固定文件 `XCopy.dmg` 和相关文档。
4. GitHub Pages 更新后打开下载链接确认能直接下载：

```text
https://raw.githubusercontent.com/Kelvenbit/XCopy-Website/aebdef97fd9e8de8287d3f0dfd8ecf8e408bee67/assets/XCopy.dmg
```

## 发布前检查

- `xcodebuild` 构建通过。
- DMG 可以挂载。
- DMG 内包含 `XCopy.app` 和 `Applications` 快捷方式。
- 正式分发版本已签名、公证并 staple。
- raw GitHub 固定文件 `XCopy.dmg` 已更新。
- GitHub Pages 首页下载按钮可直接下载最新版本。
- 隐私政策页面可访问。

## 后续可选项

这些不是当前免费分发 MVP 的必要项：

- 接入 Sparkle 自动更新。
- 绑定独立域名。
- 增加英文首页。
- 增加 SHA256 校验值。
- 增加 Release 自动化工作流。
