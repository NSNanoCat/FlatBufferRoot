<!-- 按 .github/RELEASE-TEMPLATE.md 维护当前版本的发布说明。 -->
<!-- Maintain the current release notes based on .github/RELEASE-TEMPLATE.md. -->

### New Features

- 新增由 FlatBuffers JavaScript 根表模型配置的逐 slot 编解码处理器。
- 支持独立 patch 已注册的产品表，并透明保留未修改、未注册及未来 schema 的 slot。

### Bug Fixes

- none

### Dependencies

- 使用 `flatbuffers` peer dependency，并通过 `@nsnanocat/util` 输出诊断日志。
- 使用 Biome 2.4.6 统一格式化、lint 规则和发布前检查。

### Breaking Changes

- none

### Other Changes

- 提供 ESM 入口、TypeScript 类型、运行时测试、类型契约检查和 tag 发布工作流。
