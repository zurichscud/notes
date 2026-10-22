# Tauri 图标生成指南

Tauri 官方 CLI 提供了内置的 `icon` 命令工具，能够从**一张高分辨率源图**一次性批量生成适配桌面端（macOS、Windows、Linux）与移动端（Android、iOS）的全套多规格图标。

---

## 1. 源图准备规范

为了保证在所有设备和分辨率下均有最佳显示效果，源图建议满足以下规范：

- **分辨率**：建议 **1024 × 1024 px**（最低不小于 512 × 512 px）。
- **格式**：推荐无损 **PNG**（带 Alpha 透明通道）或 **SVG** 矢量图。
- **设计细节**：
  - **不要自带圆角和阴影**：macOS 和 iOS 等系统会自动应用裁切遮罩和系统级圆角/阴影，自带圆角会导致多重边框或裁切黑边。
  - **底色处理**：iOS 规范要求 App 图标不应包含透明通道。若源图为透明底，生成时应指定背景填充色（详见命令行参数）。

---

## 2. 命令行使用方法

进入 Tauri 项目根目录，根据所用的包管理器执行命令：

### 基础生成命令
```bash

# pnpm
pnpm tauri icon [源图路径.png|svg]

# cargo
cargo tauri icon [源图路径.png|svg]
```
产物将直接输出到当前项目的 `src-tauri/icons` 目录下，并自动覆盖同名文件。

### 常用参数选项

| 参数 | 缩写 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `--output [DIR]` | `-o` | `src-tauri/icons` | 自定义生成图标的存放路径 |
| `--ios-color [HEX]` | - | `#ffffff` | 为 iOS 图标的透明背景指定填充纯色（如 `#ffffff` 或 `#000000`） |
| `--png [SIZES]` | `-p` | - | 指定额外导出的 PNG 尺寸（多个尺寸用英文逗号分隔） |

**示例**：
```bash
# 生成图标并为 iOS 自动填充白色背景
npx tauri icon ./app-logo.png --ios-color "#ffffff"

# 输出到指定目录
npx tauri icon ./app-logo.png -o ./custom-icons
```

---

## 3. 生成产物与平台规格明细

执行一次生成命令后，通常会输出约 **50+** 个文件，涵盖以下平台：

| 平台 / 分组 | 文件构成 | 数量 | 用途与系统规范 |
| --- | --- | :---: | --- |
| **通用 PNG** | `icon.png` (512×512)、`32x32.png`、`64x64.png`、`128x128.png`、`128x128@2x.png` | 5 | Linux 桌面应用图标、Linux AppImage/deb 包、通用的回退显示图 |
| **macOS** | `icon.icns` 以及 4 个 `.icns` 尺寸变体 | 5 | macOS 应用程序图标，供 Dock 栏、Finder、启动台及系统关于弹窗展示 |
| **Windows 主程序** | `icon.ico` | 1 | 整合 16~256px 多尺寸的多图层 ICO 格式，用于 `.exe` 快捷方式与 NSIS 安装向导 |
| **Windows 磁贴/应用商店** | `Square*Logo.png` 系列 (9 个)、`StoreLogo.png` | 10 | 适配 Windows 磁贴、开始菜单应用列表及 UWP/MSIX 打包规范 |
| **Android** | `android/mipmap-*` (5 档屏幕密度) | 15 | Android 各分辨率目录（mdpi 到 xxxhdpi），包含常规图标、圆形图标及自适应前景图 |
| **iOS** | `ios/AppIcon-*.png` | 18 | 覆盖 iPhone/iPad 设置、通知、Spotlight 搜索、主屏图标及 App Store 1024 预览图 |

### 输出目录结构参考
```text
src-tauri/icons/
├── 32x32.png
├── 64x64.png
├── 128x128.png
├── 128x128@2x.png
├── icon.png                 # 512×512 主图
├── icon.icns                # macOS 图标
├── icon.ico                 # Windows 可执行文件图标
├── Square30x30Logo.png ...  # Windows 磁贴图标
├── StoreLogo.png
├── android/                 # Android 平台资源
│   ├── mipmap-hdpi/
│   ├── mipmap-mdpi/
│   ├── mipmap-xhdpi/
│   ├── mipmap-xxhdpi/
│   └── mipmap-xxxhdpi/
└── ios/                     # iOS 平台资源
    ├── AppIcon-60x60@2x.png
    ├── AppIcon-1024x1024@1x.png
    └── ...
```

---

## 4. 在项目中引用配置

生成完毕后，确保在 `tauri.conf.json` 中配置了图标路径（默认已预设）：

```json
{
  "bundle": {
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ]
  }
}
```

> **系统托盘（System Tray）提示**：
> 如果应用使用了系统托盘功能，托盘通常需要一张独立的单色/模板小图标（如 32×32 或 16×16，透明底黑白线稿），不建议直接使用色彩复杂的 512×512 主应用图标。

---

## 5. 常见问题排查（FAQ）

### Q1: iOS 图标在真机或打包后显示为黑色背景？
- **原因**：iOS 严格禁止 App 桌面图标包含透明通道（Alpha）。如果源图包含透明部分，系统可能会默认填充为黑色。
- **解决**：在运行生成命令时附带 `--ios-color` 参数，如 `--ios-color "#ffffff"`，自动将透明部分融合为纯色背景。

### Q2: Windows 下修改了图标，打包或运行后图标仍然没变？
- **原因**：Windows 资源管理器有极其强烈的图标缓存机制（IconCache）。
- **解决**：
  1. 重启 Windows 资源管理器进程（Explorer.exe）。
  2. 或者将新生成的 `.exe` 文件重命名/移动到新目录查看效果。

### Q3: 导出的 macOS 图标在 Retina 屏幕上有些模糊？
- **原因**：源图尺寸不足。macOS 在 Retina 屏幕下需要最大到 1024×1024 的高精度纹理。
- **解决**：请确保输入给 `tauri icon` 的源图至少达到 1024×1024 像素，推荐使用矢量 SVG 文件作为输入源。
