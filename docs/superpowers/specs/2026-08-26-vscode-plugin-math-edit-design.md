# VS Code 插件公式与预览编辑修复设计

## 目标

修复 VS Code 插件中的两个问题，并发布补丁版本 0.3.4：

1. MathJax 无法识别 `\\boldsymbol`，导致公式源码以红色文本显示。
2. 预览 Webview 获得焦点后，点击“编辑”被误判为 Markdown 文档未激活。

## 设计

### MathJax

继续使用插件内置的 `mathjax/es5/tex-svg.js`，不依赖外部网络资源。在加载 MathJax bundle 之前配置 `window.MathJax`，显式声明 TeX package `boldsymbol`，并保留当前的 `noerrors`、`cancel`、`color` 和安全配置。MathJax 加载完成后，Cherry Markdown 通过 `externals.MathJax` 使用同一个实例。

配置必须在动态 import 之前完成，以便 MathJax startup 读取配置；同时将 `boldsymbol` 放入 Cherry Markdown 的 MathJax loader 配置，确保构建后的 Webview 使用完整扩展集合。

### 预览编辑状态

`targetEditor` 表示当前预览绑定的 Markdown 文档，不应在预览面板获得焦点时丢失。活动编辑器变为 `undefined` 只代表焦点可能转移到 Webview，不能立即发送 `disable-edit`。

保留 Markdown 编辑器为目标文档时，`isEditEnabled()` 只检查目标文档是否为 Markdown，并允许活动编辑器暂时为空；当活动编辑器切换到其他文档时，更新目标编辑器并发送 `disable-edit`。切回 Markdown 文档时重新绑定目标编辑器并发送 `enable-edit` 与最新状态。

### 测试

增加以下回归覆盖：

- Webview MathJax 配置包含 `boldsymbol` package 和宏配置。
- 活动编辑器为空时，目标 Markdown 文档仍保持可编辑。
- 切换到非 Markdown 编辑器后预览编辑被禁用。
- 切回 Markdown 编辑器后预览编辑恢复。

使用现有 Vitest 单元测试和 VS Code 集成测试，保持无网络依赖。

### 发布

将 [packages/vscodePlugin/package.json](../../packages/vscodePlugin/package.json) 版本更新为 `0.3.4`，运行插件构建、类型检查、单元测试、包内容测试和 VSIX 测试，生成 `cherry-markdown-vscode-plugin-0.3.4.vsix`。
