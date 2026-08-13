# AE 2026.2+ 插件中文汉化乱码修复工具

一个用于修复 **Adobe After Effects 2026.2 及更高版本** 中第三方插件（尤其是经过中文汉化的插件）出现界面中文乱码问题的补丁工具。

## 问题背景

从 After Effects 2026.2 版本开始，Adobe 调整了内部的字符串编码处理逻辑。许多使用 **GBK 编码** 制作的中文汉化插件，在新版本 AE 中加载时会把 GBK 文本错误地当作 UTF-8 解析，导致插件界面上的中文显示为乱码。

本工具通过 **API Hook（钩子）** 的方式在运行时拦截并修正字符串编码转换过程，从而让这些汉化插件在新版 AE 中正常显示中文。

## 工作原理

该工具本身是一个 AE 插件（`.aex` 文件），被 AE 加载后会自动挂载编码转换钩子：

- 拦截 Windows 系统 API：`MultiByteToWideChar` / `WideCharToMultiByte`，纠正代码页（GBK ↔ UTF-8）的转换。
- 拦截 Adobe 内部字符串库 `dvacore` 的转换函数（如 `UTF8ToWstring`、`WcharToUTF16`、`UTF8to16`），修正 UTF-8 与宽字符之间的转换。
- 通过 `LoadLibraryW/ExW`、`GetProcAddress`、`VirtualProtect` 等实现对目标函数的动态挂钩。
- 插件入口函数为 `EntryPointFunc`，内部标识为 `GBK UTF8 Hook`。

> 简单来说：它在 AE 读取汉化插件文本时，"翻译"回正确的编码，避免中文变成乱码。

## 文件说明

| 文件 | 说明 |
| --- | --- |
| `!!GBK_UTF8_HOOK.aex` | 核心补丁插件（x64 / 64 位）。文件名前缀 `!!` 用于保证其在插件目录中优先加载。 |
| `安装方法.txt` | 简要安装说明。 |
| `README.md` | 本说明文档。 |

### 技术信息

- **类型**：After Effects 插件（`.aex`，本质为 Windows DLL）
- **架构**：x64（64 位），依赖 `VCRUNTIME140.dll` / `MSVCP140.dll` 等 Visual C++ 运行库
- **平台**：Windows
- **适用版本**：After Effects 2026.2 及更高版本

## 安装方法

1. 将 `!!GBK_UTF8_HOOK.aex` 复制到 AE 的插件目录，例如：

   ```
   C:\Program Files\Adobe\Adobe After Effects 2026\Support Files\Plug-ins
   ```

2. 重新启动 After Effects，钩子会在启动时自动生效。

> 提示：若 AE 安装在其他路径，请对应修改上面的目录。

## 注意事项

- 本工具仅适用于 **Windows 平台** 的 **64 位** After Effects。
- 仅针对因 GBK/UTF-8 编码不一致导致的中文乱码问题；其他原因引起的显示问题不在修复范围内。
- 由于工具通过 API Hook 方式工作，若某些安全软件误报，请自行判断并添加信任。

## 来源

更多软件 / 插件 / 模板 / 素材下载，请访问
