# Changelog

格式基于 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)，版本号遵循 [语义化版本](https://semver.org/lang/zh-CN/)。  
Skill 版本见 `SKILL.md` 头部 `version`；作者见 `author`。

## [Unreleased]

## [1.0.0] - 2026-11-19

### 新增

- 展锐 8910 / MOCOR 提交边界：仓型判定、该交 / 勿交、客户 bat 覆盖对照
- 文档：`SKILL.md`、`OVERWRITE-MAP.md`
- 补丁仓 **zxe / sprdbuildE** 覆盖链：优先交 `Prj_mk` 与 zxe 指向源，勿只交树内拷贝目标（`OVERWRITE-MAP` §F）

### 说明

- 仓型：客户 bat 仓 / Prj_mk 补丁仓 / 平台仓；zxe 入口为补丁仓上的正交编译/覆盖入口，非第四种仓型
- 脱敏：占位 `<Product>` / `Prj_mk\<工程>` / `<工程>.zxe`；勿写死客户/SKU/BOM
- 本 skill 只管提交边界；不主动 `git commit`（除非用户明确要求）；勿引用其它 skill 名
