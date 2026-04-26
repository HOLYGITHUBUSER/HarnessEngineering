# 视觉审查

- 布局、换行、行高、选中态、hover、focus、颜色、webview 改动都属于视觉改动。
- 视觉行为不能只靠 DOM 断言判断。
- 重要状态要生成截图，并在汇报成功前人工 review。
- Webview 要用 Playwright 测真实 bundle，或尽量接近真实环境的 harness。
- 如果改动和主题或布局有关，要覆盖深色/浅色主题、小窗口/大窗口。
- 生成的截图目录默认不要提交，除非项目明确需要 baseline 图片入库。
