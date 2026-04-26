# VSCode 插件开发闭环

## 触发语

`/vscode-extension`

## 用途

用于 VSCode API 兼容插件的开发、测试、打包和真实安装验证。

## 执行要求

1. 先读取项目的 `package.json`、构建配置、测试配置和 manifest/contributes。
2. 按改动类型选择测试：编译、单测、Extension Host、Webview E2E、视觉截图。
3. 涉及 Webview 或 UI 时必须截图 review。
4. 打包 VSIX 后安装到目标编辑器：VS Code、Cursor、Windsurf 中项目要求的目标。
5. 安装后 reload window，必要时关闭旧 tab 重新打开。
6. 最终汇报测试结果、安装结果、剩余风险。

详细流程见：`../../workflows/extension-dev-loop-插件开发循环.md`。
