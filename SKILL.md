---
name: unisoc-mocor-commit-skill
version: "1.0.0"
author: zqingquan
description: >
  展锐 8910 系 / MOCOR 提交边界：哪些该交、哪些不能交、
  产品 bat 会覆盖的文件如何落到可持久源再提交。
  触发：提交、commit、暂存、git add、该交什么、不要提交、覆盖文件、
  Prj_mk、产品 bat、USER_DEBUG.mk、编译产物、.pac。
---

# 8910 / MOCOR 提交规范

**本 skill 只管提交边界**（分类 dirty 文件、建议暂存列表；不主动 commit，除非用户明确要求）。  
不写具体客户/SKU 工程名，用 `<Product>` / `Prj_mk\<工程>` 占位；举例仅说明模式。

## 0. 先判仓型（强制）

在分类任何 dirty 文件之前，先定编译根与仓型：

| 仓型 | 特征 | 提交策略 |
|------|------|----------|
| **客户 bat 仓** | 编译根有 `<Product>.bat`（含 `xcopy`/`copy` + 调 make）且有 `Prj_mk\` | 只交源，勿交 bat 覆盖目标 |
| **Prj_mk 补丁仓** | 有 `Prj_mk\` + `hx_macro_option.mk` include 链，但**无** 上述产品 xcopy bat | 无文件覆盖链；树内源可交。`Prj_mk\<工程>\` / 共享基线（宏、`.mk`、NV、驱动修订）**必交**。勿套 [OVERWRITE-MAP.md](OVERWRITE-MAP.md) 拒树内源 |
| **平台仓** | 有 `make.bat`，**无** 产品 xcopy bat，**无** `Prj_mk\` | 树内路径多为真相源，可交 |

编译根常见：仓库根，或 `idh.code\MS_Code`（以含 `make.bat` 为准）。

- 客户 bat 仓 → 下文 + [OVERWRITE-MAP.md](OVERWRITE-MAP.md)。  
- 补丁仓 / 平台仓 → 不拦「覆盖目标」；仍禁止交 `build\*_builddir\`、`.pac`、本机噪音；有 `Prj_mk` 时优先交补丁源。  
- 吃不准 → 搜含 `xcopy` 且引用 `Prj_mk` 的 `*.bat`：有 → 客户 bat 仓；仅有 `Prj_mk` → 补丁仓；都无 → 平台仓。

## 单一真相源（仅客户 bat 仓）

产品 `<Product>.bat` 会用 `xcopy` / `copy` / `echo >>` 覆盖一批路径。  
**只改、只交源路径**；目标副本下次会被盖掉，单独提交无意义。

源→目标详见 [OVERWRITE-MAP.md](OVERWRITE-MAP.md)。条目因产品而异；**以当前 `<Product>.bat` 实际命令为准**。

## 提交前检查清单（强制）

完成准则：每个 dirty 文件已归类，建议暂存列表不含产物 /（客户仓下的）覆盖目标。

1. 定编译根 + **仓型**（§0）。
2. `git status` + `git diff`（含已暂存）列出候选。
3. 逐文件分类：`产物` / `生成物` / `覆盖目标` / `真相源` / `普通源码`。
4. **客户仓**：覆盖目标且源未改 → **拦住**，提示改 `Prj_mk\<工程>\` / 共享 `Prj_mk\<公共目录>\` / `*_in_use.h` / 客户 `config\<源目录>` / bat 变量；勿把目标加入暂存。
5. **补丁仓 / 平台仓**：勿把树内驱动/config/资源头误判为覆盖目标而拒交。
6. 给出建议暂存列表 + 一句提交说明方向。
7. **不主动 `git commit`**，除非用户明确要求。

## 该提交

| 类型 | 示例（占位） |
|------|----------------|
| 业务/驱动/MMI 源码 | 非覆盖目标下的 `.c`/`.h`（补丁仓/平台仓含 `lcd`/`dc` 等） |
| 工程补丁源 | `Prj_mk\<工程>\` 下 `hx_cust_config.h`、`hx_macro_switch.mk`、`tft*`/`sensor*`、`str_table.xls`、`CustomImages\`、NV、`key_text_com.h` 等 |
| 工程间共享补丁 | `Prj_mk\<公共目录>\*.h` / `*.mk`（bat 或 `include` 引用时） |
| 宏桥（非 bat 重写） | `Prj_mk\hx_macro_option.mk`（若有） |
| 模板源 | `*_in_use.h`（客户仓） |
| 客户 config 源 | `MS_Customize\...\config\<客户 refphone 源目录>\`（目标常为 `uis8910ff_refphone`） |
| 刻意改动的构建入口 | `project_*.mk`、产品 `*.bat` |
| 平台/树内公共源 | 确认无 bat 回写后再交 |

## 不要提交

| 类型 | 示例（占位） |
|------|----------------|
| 编译产物树 | `build\<工程>_builddir\`（`.pac` / `img` / `log` / `obj` / 模拟器） |
| 每次 bat 重写 | `Prj_mk\USER_DEBUG.mk`、`bt_default_name.h`、`version_*.h`、`custom_wallpaper_number.h`、`custom_ring_res.h`、`wallpaper_mdu_def.h` |
| 由模板生成的副本 | 客户仓：`common_mdu_def.h`、`mmi_res_prj_def.h`（交 `*_in_use.h`）。无 `*_in_use.h` 的仓则可交 `.h` 本体 |
| xcopy 目标侧 | 如只改了资源树里的 `str_table.xls`，真相在 `Prj_mk\...\str_table.xls` |
| 模拟器 lib 被盖脏 | `lib\<平台>_BT_DEBUG\` 等若由 `lib\simu_VS2010\` 覆盖 → 勿当业务改动交 |
| 本机噪音 | IDE 临时文件、无意改动的第三方子树 |

## 改到会被覆盖的文件时（客户 bat 仓）

1. 在 `<Product>.bat` 搜该文件名 / 目标目录的 `xcopy`/`copy`/`echo >>`。
2. 定源：`Prj_mk\<工程>\` → 共享 `Prj_mk\<公共目录>\` → `*_in_use.h` → 客户 `config\<源目录>` → bat 内变量。
3. **只暂存源**（及非覆盖类真实改动）；目标 dirty 也不暂存。
4. 说明里可写「目标为 bat 覆盖副本，以源为准」，仍只交源。

## 易漏交 / 易乱交

| 风险 | 典型场景 |
|------|----------|
| 漏交 | 只改了 `lcd`/`dc`/`uis8910ff_refphone`/资源树副本，未改 `Prj` / 客户 config 源 |
| 漏交 | 改了 `hx_macro_switch.mk` / 共享 `Prj_mk\*.mk` 却当成生成物跳过（真相源，**不是** `USER_DEBUG.mk`） |
| 漏交 | 客户仓改了 bat 会拷的 `CustomImages\gif` 等，只交了目标目录 |
| 乱交 | `USER_DEBUG.mk`、bt/version/wallpaper/ring 生成头、`build\*_builddir` |
| 乱交 | 补丁仓/平台仓硬套客户 map，拒交本可交的树内源 |
