# 🎯 MediaHunter (媒体猎手)

![Version](https://img.shields.io/badge/Version-Front_v8.3.7_|_Back_v1.9.17-blue.svg)
![Platform](https://img.shields.io/badge/Platform-Windows_10%2F11-lightgrey.svg)
![Architecture](https://img.shields.io/badge/Architecture-Local_API-success.svg)

**MediaHunter** 是一款高度优化的音视频嗅探与静默下载系统。它巧妙地将浏览器的资源解析能力与强大的 `yt-dlp` 下载引擎无缝结合，并通过 Windows 系统原生通知提供丝滑的下载进度反馈。

> **核心理念：** 拒绝臃肿的 GUI 客户端，拒绝浏览器的下载沙盒限制。让极客工具回归“静默、高效、安全、高度可配”的本质。

---

## 📸 运行效果预览 (Showcase)

以下是 MediaHunter 完整的工作流演示（悬浮按钮 ➔ 解析面板 ➔ 点击下载中 ➔ 后台下载进度条 ➔ 提示下载完成）：

> **💡 交互体验设计：**
> * **零打扰弱提醒**：点击“下载”时，仅网页端按钮状态变为“下载中”进行弱提醒，轻量无打扰。
> * **按需查看进度**：可手动点击 Windows 系统右下角的 **通知中心**（或按 `Win + N`）查看下载进度条。
> * **关键节点强提醒**：仅在**下载完成**或**下载失败**时，才会在桌面右下角弹窗通知。

![MediaHunter 完整下载流程演示](assets/workflow_demo.gif)

---

## 📌 目录 (Table of Contents)

1. [🚀 快速上手 (Quick Start)](#-快速上手-quick-start)
2. [💡 前端功能使用 (Usage)](#-前端功能使用-usage)
3. [⚙️ 核心配置调优指南 (Configuration)](#-核心配置调优指南-configuration)
4. [🛠️ 常见问题与日志诊断 (FAQ)](#-常见问题与日志诊断-faq)
5. [📜 免责声明 (Disclaimer)](#-免责声明-disclaimer)

---

## 🚀 快速上手 (Quick Start)

本项目发布提供两种压缩包，请根据您的需求下载：
* **📦 全能集成包 (`MediaHunter-Full.zip`)**：**推荐，开箱即用**。内置集成了 `yt-dlp.exe`、`ffmpeg.exe`/`ffprobe.exe` 以及免安装的解密引擎 `deno.exe`。
* **🪶 纯净轻量包 (`MediaHunter-Lite.zip`)**：**体积小**。仅包含核心服务脚本，需自行手动补全“运行依赖”。

---

### 📂 情况 A：如果您使用的是【全能集成包 (Full)】

1. **解压目录**：直接将 `MediaHunter-Full.zip` 解压至本地任意目录（例如 `D:\MediaHunter\`）。
2. **启动服务**：双击解压目录下的 **`StartServer.vbs`**。本地网关将以隐藏窗口形式在后台启动。
   > [!WARNING]
   > **Windows 11 24H2 及以上系统的兼容性提示：**
   > 微软从 **Windows 11 版本 24H2 (OS Build 26100 及以上)** 开始默认将 VBScript 设为按需可选功能且可能默认未预装。如果您双击 `.vbs` 脚本无反应或报错，请前往 Windows **“设置” -> “系统” -> “可选功能”** 中搜索并安装 **“VBScript”**。
3. **启用脚本**：
   * 确保您的浏览器安装有 [Tampermonkey (油猴)](https://www.tampermonkey.net/) 插件。
   * 在油猴中“添加新脚本”，复制粘贴项目中的 `MediaHunter-8.3.7.user.js` 全部代码并保存。
   * 刷新任意视频网站，若网页右下角显示紫色 **「媒体猎手」** 按钮，说明部署已圆满成功！

---

### 🪶 情况 B：如果您使用的是【纯净轻量包 (Lite)】

1. **解压目录**：将 `MediaHunter-Lite.zip` 解压至本地任意目录（如 `D:\MediaHunter\`）。
2. **手动补全运行依赖与环境变量**：
   * **`yt-dlp` 下载引擎**：前往 [yt-dlp Github Release](https://github.com/yt-dlp/yt-dlp/releases) 下载最新版 `yt-dlp.exe`，并**手动将其路径配置进系统环境变量 `Path`**。
   * **音视频合并器 (FFmpeg)**：前往 [gyan.dev](https://www.gyan.dev/ffmpeg/builds/) 下载并解压，同时将 `bin` 目录路径配置到系统环境变量 `Path` 中。
   * **JS 运行时 (可选)**：用于 YouTube 等站点的本地解密。直接从官方网站下载 Windows 安装包手动安装即可。
3. **启动服务与脚本**：同“情况 A”的 2、3 步，双击 **`StartServer.vbs`** 运行，并在浏览器油猴插件中安装前端脚本。

> [!TIP]
> **🚀 配置开机自启 (可选)：**
> 1. 右键点击 `StartServer.vbs` ➔ 创建快捷方式。
> 2. 按 `Win + R` 键，输入 `shell:startup` 回车打开 Windows 启动文件夹。
> 3. 将快捷方式剪切并粘贴至该文件夹下，后续每次开机便会自动在后台静默运行网关。

---

## 💡 前端功能使用 (Usage)

在网页中成功嗅探到媒体资源后，点击右下角 **「媒体猎手」** 展开资源卡片列表，每个资源卡片都支持以下三种操作：

1. **📋 复制直链**
   * **单击“复制”按钮**：将视频流的真实 URL 复制到剪贴板。适合复制到mpv、N_m3u8DL-RE中播放与下载。
2. **📥 静默下载**
   * **单击“下载”按钮**：将下载任务投递至本地下载器，并开始后台下载。
3. **💻 复制带参直链**
   * **Alt + 单击“复制”按钮**：复制一条携带Referer、Origin伪装头、文件名等参数的 `yt-dlp` 下载命令，方便您在终端中手动调试。

---

## ⚙️ 核心配置调优指南 (Configuration)

所有的下载行为、保存路径、网络代理及多浏览器适配参数，均统一存放在 **`yt-dlp.conf`** 配置文件中。您可以使用记事本打开该文件，自定义以下设置：

### 🟢 基础配置调整
1. **自定义下载保存路径**：
   找到并编辑 `--paths` 属性，将其修改为您电脑上真实存在的目录。
   ```text
   --paths "temp:D:\Cache"        # 临时缓存目录（绝对路径示例：D盘缓存文件夹）
   --paths "home:D:\Downloads"    # 最终下载完毕的保存目录（绝对路径示例：D盘下载文件夹）
   ```
2. **修改或关闭代理设置**：
   若无科学上网，或者代理端口不是 `10808`，请修改该配置，或者直接在行首添加 `#` 注释掉该行：
   ```text
   #--proxy "socks5://127.0.0.1:10808"
   ```

---

### 💻 适配不同的浏览器

本项目默认优先适配机制更为开放的 **Firefox**（已默认启用）：
* **优点**：无需手动导出任何文件，支持实时在线热提取 Cookie。只要您在 Firefox 中登录了目标视频网站，下载引擎便能自动共享登录凭证（包括 VIP 身份），即插即用。

如果您倾向于使用 **Chrome** 浏览器，由于 Chromium 内核的 DPAPI 本地安全加密限制，直接自动提取将无法正常工作。具体配置和解决办法请直接参考 **`yt-dlp.conf`** 配置文件中 **【方案 B：使用 Chrome 浏览器】** 对应的注释说明进行调整（需要配合手动导出 `cookies.txt`）。

---

## 🛠️ 常见问题与日志诊断 (FAQ)

### Q1：提示“下载失败”，日志含 `FFmpeg not found` 怎么办？
* **解决办法**：未配置音视频合并工具。请将 `ffmpeg.exe` 与 `ffprobe.exe` 直接放入部署同级目录中。

### Q2：提示“下载失败”，日志含 `Failed to establish a new connection` 怎么办？
* **解决办法**：本地网络代理不通。如果您不需要代理，请用记事本打开 `yt-dlp.conf` 并在 `--proxy` 行首添加 `#` 注释掉该行。

### Q3：提示“下载失败”，日志含 `403 Forbidden` 或 `Sign in` 怎么办？
* **解决办法**：视频网站的 Cookie 登录态过期。请先在对应浏览器中打开并登录该视频网站以刷新登录凭证。


---

## 📜 免责声明 (Disclaimer)

本项目仅供网络技术学习、防盗链机制研究及本地系统 API 调用测试使用。请遵守当地法律法规及目标网站的服务条款，严禁使用本工具下载或传播未经授权的版权内容。

---

*Created with ❤️ by yudong2ao & Gemini*