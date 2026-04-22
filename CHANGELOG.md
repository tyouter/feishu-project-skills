# Changelog

## 0.2.0 (2026-04-22)

- **fix**: feishu-sync 文档创建流程从两步改为一步（--wiki-node --wiki-space）
- **new**: 新增 sync-mapping.yaml 配置文件（静态映射，与 sync-state.yaml 职责分离）
- **new**: 新增 `/update` 技能 — 版本感知的增量更新机制
- **new**: 新增 VERSION 文件和 CHANGELOG.md
- **change**: 会议目录从 `Meetings/Video/` 重构为 `3-Meetings/<slug>/`
- **change**: meeting-sync 新增 summary.md 作为 Wiki 推送源

## 0.1.0 (2026-04-20)

- 初始版本：5 个技能 + 模板 + setup 向导
