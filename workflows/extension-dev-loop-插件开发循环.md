---
description: VSCode/Cursor/Windsurf 插件开发、测试、打包、真实安装验证闭环
---

# VSCode 插件开发闭环工作流

## 适用范围
用于 VSCode API 兼容插件的完整开发闭环，包括 VS Code、Cursor、Windsurf 等编辑器。适合命令型插件、Webview 插件、自定义编辑器、语言服务、调试器、TreeView、状态栏、Notebook、文件系统 Provider 等。

这个流程的目标不是“跑几个测试就结束”，而是确认改动在真实编辑器、真实 VSIX、真实用户设置下仍然工作。

## 触发方式
使用斜杠命令：`/vscode-extension`

也可在任意插件项目中按本文件作为检查清单执行。

## 0. 项目识别
先识别项目实际技术栈，不要套模板。

- 读取 `package.json`，确认 `main`、`activationEvents`、`contributes`、`scripts`、`engines.vscode`。
- 读取构建配置：`tsconfig.json`、webpack/esbuild/vite 配置、`.vscodeignore`。
- 读取测试配置：Mocha、Vitest、Node test runner、`@vscode/test-electron`、Playwright、Jest 等。
- 读取调试配置：`.vscode/launch.json`、`.vscode/tasks.json`。
- 识别产物目录：`out/`、`dist/`、`dist-产物/`、`.vsix`、coverage、test-results。
- 识别目标编辑器：VS Code、Cursor、Windsurf，必要时都要安装验证。

## 1. 需求与风险分析
每次改动前先分类，决定测试深度。

- Extension host 逻辑：命令、配置、状态、文件读写、workspace API。
- Webview UI：DOM、样式、消息协议、快捷键、截图 review。
- Manifest/contributes：命令、菜单、语言、custom editor、configuration、activationEvents。
- 打包发布：`.vscodeignore`、README、icon、依赖、VSIX 内容。
- 状态与配置：globalState、workspaceState、用户设置、工作区设置、远程设置。
- 兼容性：VS Code/Cursor/Windsurf、不同主题、不同窗口大小、已有旧 tab。

输出本轮测试计划：最小必跑测试、风险扩展测试、是否需要真实安装。

## 2. 基线检查
修改前尽量建立基线，尤其是修 bug。

- 运行现有测试中最相关的 subset。
- 记录当前用户可见行为。
- 对 Webview/视觉问题，先截一张失败前截图。
- 对配置问题，先记录有效配置来源：默认值、用户设置、工作区设置、remote 设置。
- 对生命周期问题，记录 activate/deactivate 日志和打开/关闭/重载步骤。

## 3. 开发实施
按项目既有结构改，避免引入不必要新框架。

- Extension host 代码修改后，同步更新测试。
- Webview 协议变化要同时改两端：扩展端 `postMessage` 处理和 Webview 脚本。
- 新命令要同步 `contributes.commands`、activationEvents、命令注册、测试。
- 新配置要同步 `contributes.configuration`、读取逻辑、配置变更监听、文档。
- 新资源要检查 `webview.asWebviewUri`、CSP、`.vscodeignore`。

## 4. 分层测试
按改动类型选择测试，不要只跑一个总命令。

### 4.1 静态检查
- TypeScript 编译：`npm run compile` 或项目等价命令。
- Lint：`npm run lint`，如果仓库没有 lint 配置，要明确记录。
- Manifest 基础检查：`package.json` JSON 合法、contributes ID 唯一、命令 ID 一致。

### 4.2 单元测试
- 运行项目实际单元测试命令，例如 `npm test`。
- 如果项目使用 `node --test`、Mocha、Vitest、Jest，不要误写成固定框架。
- 对核心函数新增 focused test，避免只靠 E2E。

### 4.3 Extension Host 集成测试
适用于命令、配置、文件系统、语言服务、custom editor 激活等。

- 使用 `@vscode/test-electron` 或项目已有 harness。
- 验证命令可执行、配置变更生效、workspace 文件读写正确。
- 验证 activationEvents：打开目标文件或执行命令时才激活。
- 验证 reload window 后状态恢复。

### 4.4 Webview 自动化测试
适用于按钮、表格、输入框、拖拽、快捷键、消息协议。

- 用 Playwright 加载真实 Webview bundle 或等价 harness。
- 覆盖点击、双击、右键、拖拽、键盘快捷键、focus/blur。
- 验证 `postMessage` payload，不只验证 DOM 有变化。
- 对所有关键按钮保留“按钮全量验证”spec，防止浮动面板、排序、查找、替换等失效。

### 4.5 视觉截图 Review
只要涉及布局、换行、行高、颜色、选中态、浮动面板、主题适配，就必须截图。

- 生成稳定截图目录，例如 `e2e/visual-review/` 或 `e2e-端到端/visual-review/`。
- 覆盖至少：默认态、目标交互态、错误态或空态。
- Webview 表格类插件至少覆盖：紧凑/单行/换行、选中、编辑、排序、过滤。
- 同时看 light theme 和 dark theme，除非项目明确只支持一种。
- 视觉截图通过不等于自动正确，必须人工 review 截图内容。

### 4.6 配置覆盖测试
默认值不生效时，优先查配置覆盖。

- 用户设置：VS Code/Cursor/Windsurf 各自的 User `settings.json`。
- 工作区设置：项目 `.vscode/settings.json`。
- Remote/Profiles 设置：远程开发或 Profile 可能覆盖默认。
- 插件状态：globalState、workspaceState、memento、webview state。
- 旧 Webview 实例：reload 后仍不对时，关闭旧 tab 再重开。

## 5. 打包检查
真实插件问题很多发生在 VSIX 后，不在源码运行时暴露。

- 运行项目打包命令：`npm run package`、`vsce package`、`npx @vscode/vsce package` 或项目脚本。
- 检查 VSIX 内容：`vsce ls --tree` 或解压检查 `extension/package.json`、`extension/out/`、资源文件。
- 确认 `out/` 是最新编译产物。
- 确认 `.vscodeignore` 没误排除运行必需文件，也没带入明显无关文件。
- 检查 README 图片路径、icon、license、publisher、version。
- 如果项目要求产物入库，保留并提交 `dist/` 或 `dist-产物/`；如果不要求，加入 ignore。

## 6. 真实安装验证
打包后必须装到目标编辑器，不只在 Extension Development Host 里试。

### VS Code
```bash
code --install-extension path/to/extension.vsix --force
```

### Cursor
```bash
/Applications/Cursor.app/Contents/Resources/app/bin/cursor \
  --install-extension path/to/extension.vsix --force
```

### Windsurf
```bash
/Applications/Windsurf.app/Contents/Resources/app/bin/windsurf \
  --install-extension path/to/extension.vsix --force
```

安装后执行：

- `Developer: Reload Window`。
- 关闭旧的目标文件 tab，再重新打开。
- 在真实目标文件/真实 workspace 中验证核心行为。
- 如果涉及配置默认值，确认用户设置没有覆盖。
- 如果涉及 Webview，打开 Developer Tools 看 console error。

## 7. 跨编辑器/跨环境矩阵
根据风险选择矩阵，不必每次全跑，但发布前要覆盖。

- VS Code Stable。
- Cursor。
- Windsurf。
- macOS、Windows、Linux 中至少覆盖目标用户主平台。
- clean profile 和常用用户 profile。
- light/dark/high contrast theme。
- 小窗口、大窗口、多编辑器组。
- 本地文件、远程文件、只读文件、大文件。

## 8. 回归与失败处理
失败时不要直接猜。

- 先缩小复现：最小文件、最小命令、最小设置。
- 保存失败截图、trace、console、extension host logs。
- 对比修改前后 `git diff`。
- 判断是产品 bug、测试 harness 过期、环境设置覆盖，还是打包遗漏。
- 修复后只先跑 focused test，再跑必要全量测试。

## 9. 提交与推送
提交前确认工作区。

- `git status --short --branch`
- `git diff` 查看 unstaged/staged。
- 不要误提交 secrets、私有 token、`.env`。
- 产物是否提交按项目规则执行；如果项目要求“产物都推送”，`dist/`、`dist-产物/` 不要漏。
- 不用但要保留的文件移入 `backup/`。
- commit message 用项目既有风格，例如 Conventional Commits。
- push 前确认当前分支和 remote。

## 10. 输出报告格式
每次完成后输出简短但完整的验证报告。

```markdown
## 插件开发验证报告

### 改动目标
[本轮要解决的问题]

### 主要改动
- [改动 1]
- [改动 2]

### 测试结果
- 编译：[通过/失败/未跑，原因]
- 单元测试：[通过/失败/未跑，命令]
- Extension Host 集成：[通过/失败/未跑，命令]
- Webview E2E：[通过/失败/未跑，命令]
- 视觉截图：[已 review/未 review，截图路径]
- 打包：[通过/失败，VSIX 路径]
- 真实安装：[VS Code/Cursor/Windsurf 结果]

### 配置与状态检查
- 用户设置覆盖：[有/无]
- 工作区设置覆盖：[有/无]
- reload/reopen 后表现：[正常/异常]

### 剩余风险
- [仍需人工确认的点]

### Git
- commit：[hash + message]
- push：[remote/branch 或未 push 原因]
```

## 11. 常见坑
- 源码改了但 VSIX 里还是旧 `out/`。
- VSIX 装了但窗口没 reload。
- 旧 Webview tab 没重建，仍在跑旧脚本。
- 默认配置被 User/Workspace settings 覆盖。
- `.vscodeignore` 把运行资源排除了。
- Webview 在 harness 里能跑，真实 CSP 下失败。
- DOM 断言通过，但真实视觉错了，必须截图 review。
- 只测 VS Code，Cursor/Windsurf 里 API 行为或路径不同。
- 打包产物和源码版本不一致。
- 生成产物被忽略或误删，导致用户拿不到最新 VSIX。
