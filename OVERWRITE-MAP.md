# 常见覆盖对照（源 → 目标）

仅用于 **客户 bat 仓**（有含 `xcopy`+`Prj_mk` 的 `<Product>.bat`）。  
路径相对编译根（含 `make.bat` / `Prj_mk` 的那一层；也可能在 `idh.code\MS_Code`）。

**平台仓** / **Prj_mk 补丁仓** 不要用本表拦截树内源；补丁仓仍应提交 `Prj_mk\` 真相源。见 `SKILL.md` §0。

不同产品 bat 条目会增减；**以当前 `<Product>.bat` 实际 `xcopy`/`copy`/`echo` 为准**。

记号：`Prj` = `Prj_mk\<工程>\`

## A. Prj_mk 补丁 → 树内目标

| 源（交这个） | 目标（勿单独交） |
|-------------|------------------|
| `Prj\tft*` | `MS_Customize\source\product\driver\lcd\` |
| `Prj\sensor*` | `MS_Customize\source\product\driver\dc\` |
| `Prj\hx_cust_config.h` | `RTOS\export\inc\hx_cust_config.h` |
| `Prj_mk\<公共目录>\*.h`（工程间共享客户头） | `RTOS\export\inc\` |
| `Prj\hx_nv_cus_config\nvitem\` | `lib\modem\...\nvitem\`（USER/DEBUG 常各一份） |
| `Prj\Fota_adup\abup_define.h` | `Third-party\abup\hal\include\abup_define.h` |
| `Prj\str_table.xls` | `MS_MMI_Main\source\resource\Common\str_table.xls` |
| `Prj\color_table.xls` | `MS_MMI_Main\source\resource\<资源工程>\` |
| `Prj\CustomImages\Mainmenu_user\mmimenu_icon_xy.h` | `MS_MMI_Main\...\mainmenu\h\` |
| `Prj\CustomImages\Mainmenu_user\mmi_mainmenu_data_user.c` | `MS_MMI_Main\...\mainmenu\c\` |
| `Prj\CustomImages\Mainmenu_user\mmi_menutable_user.c` | `MS_MMI_Main\...\common\c\` |
| `Prj\CustomImages\wallpaper\` | `...\staticwallpaper\`（部分 bat 另有第二资源树目标） |
| `Prj\CustomImages\POWER_ON_OFF\` | `...\Power_on_off\`（可能双目录） |
| `Prj\CustomImages\Mainmenu_user\`（图） | `...\Mainmenu_user\` |
| `Prj\CustomImages\gif\` | 视 bat（如 ai_box 图库或 preload 图库） |
| `Prj\CustomImages\ring\*` | `MS_MMI_Main\source\resource\common\RING\` |
| `Prj\key_text_com.h` | `MS_MMI_Main\...\eng\h\test_key_text.h` |

Keymap / PinMap：部分 bat 从 `MS_Customize\...\config\<客户 refphone 源目录>\` 拷到 `...\config\uis8910ff_refphone\`。  
**交客户 config 源目录**，勿只交 `uis8910ff_refphone` 副本。

## A2. 非 xcopy、但是真相源（易漏交）

| 路径 | 说明 |
|------|------|
| `Prj\hx_macro_switch.mk` | 经 `Prj_mk\hx_macro_option.mk` `include`，不被 bat 覆盖；改宏必须交 |
| `Prj_mk\hx_macro_option.mk` | 公共宏桥；刻意改动时可交 |
| `Prj_mk\<公共目录>\*.mk` | 若被 `hx_macro_switch.mk` `include`；交 mk（对应 `.h` 或另有 xcopy） |

勿与 `Prj_mk\USER_DEBUG.mk`（bat 重写）混淆。

## B. 模板 → 生成头

| 源（交这个） | 目标（勿交） |
|-------------|--------------|
| `...\common_mdu_def_in_use.h` | `...\common_mdu_def.h` |
| `...\mmi_res_prj_def_in_use.h` | `...\mmi_res_prj_def.h` |

追加段的资源真相在 `Prj\CustomImages\`；固定模板段在 `*_in_use.h`。无 `*_in_use.h` 的仓可交 `.h` 本体。

## C. bat 变量 / 扫描 → 纯生成物（通常不交）

| 生成物 | 应改 / 应交的源 |
|--------|-----------------|
| `Prj_mk\USER_DEBUG.mk` | bat 的 USER/DEBUG 参数（不交此文件） |
| `MS_Customize\export\inc\bt_default_name.h` | bat 内蓝牙名变量 |
| `version\version_sw_release.h` | bat 内版本变量 |
| `version\version_resource_package.h` | version 源文件或 bat（视仓约定）；不交生成头 |
| `...\wallpaper_mdu_def.h` | `Prj\CustomImages\wallpaper\` |
| `CustomFiles\custom_wallpaper_number.h` | 同上 |
| `CustomFiles\custom_ring_res.h` | `Prj\CustomImages\ring\` |

## D. 整树产物（永不交）

- `build\<工程>_builddir\`（`img\*.pac`、`log\`、`obj\`、`win\` 等）
- `img\` 下打包临时目录 / `package_info.txt`
- 由 `lib\simu_VS2010\` 盖脏的 `lib\<平台>_BT_DEBUG\` 等

## E. 判定口诀（commit）

1. 无产品 xcopy bat：有 `Prj_mk` → 补丁仓；无 → 平台仓。勿用本表拒树内源；只挡 `build\` 等产物。  
2. `build\` 下 → 产物，跳过。  
3. 出现在 `<Product>.bat` 的 `xcopy` **右侧** → 找左侧源；只交源。  
4. `hx_macro_switch.mk` / 被 include 的 `Prj_mk\*.mk` → 真相源（排除 `USER_DEBUG.mk`）。  
5. 有同名 `*_in_use.h` → 交模板，不交生成副本。  
6. `copy nul` + `echo >>` 写出 → 生成物；交驱动它的源或 bat 变量。  
7. 吃不准 → 打开当前 `<Product>.bat` 搜文件名再分类；禁止默认「全 add」。
