# 🎯 MediaHunter (媒体猎手)

![Version](https://img.shields.io/badge/Version-Front_v8.3.7_|_Back_v1.9.17-blue.svg)
![Platform](https://img.shields.io/badge/Platform-Windows_10%2F11-lightgrey.svg)
![Architecture](https://img.shields.io/badge/Architecture-Local_API-success.svg)

**MediaHunter** 是一个高度优化的“浏览器-本地桌面”联动音视频嗅探与下载系统。它巧妙地将前端浏览器的资源嗅探能力与后端强大的 `yt-dlp` 命令行工具无缝结合，并通过底层的 Windows 原生 Toast API 提供丝滑的下载进度反馈。

> **核心理念：** 拒绝臃肿的 GUI 界面，拒绝浏览器的下载沙盒限制。让极客工具回归“静默、高效、安全、高度可配”的本质。

---

## 📌 目录 (Table of Contents)

1. [⚙️ 系统架构 (System Architecture)](#-系统架构-system-architecture)
2. [✨ 核心特性 (Key Features)](#-核心特性-key-features)
3. [📊 优势对比 (Feature Comparison)](#-优势对比-feature-comparison)
4. [🚀 安装与部署 (Installation & Deployment)](#-安装与部署-installation--deployment)
5. [💻 服务运行与管理 (Service Management)](#-服务运行与管理-service-management)
6. [⚙️ 配置文件指南 (Configuration Guide)](#-配置文件指南-configuration-guide)
7. [🛠️ 故障排除与日志 (FAQ & Troubleshooting)](#-故障排除与日志-faq--troubleshooting)
8. [📜 免责声明 (Disclaimer)](#-免责声明-disclaimer)

---

## ⚙️ 系统架构 (System Architecture)

MediaHunter 采用典型的 **Local API** 架构，由以下五大组件构成完美闭环：

```mermaid
graph TD
    subgraph Browser["🌐 浏览器端 (Frontend)"]
        Tab["网页 (任意视频网站)"] -->|拦截/解析媒体流| Script["MediaHunter-8.3.7.user.js (前端探针)"]
        Script -->|GM_xmlhttpRequest (绕过浏览器CORS)| Server["127.0.0.1:23333 (安全网关)"]
    end

    subgraph Windows["💻 Windows 桌面端 (Backend)"]
        Server -->|静默拉起子进程| Worker["Worker.ps1 (任务执行与UI)"]
        Worker -->|载入配置参数| Conf["yt-dlp.conf (引擎配置文件)"]
        Worker -->|执行下载命令| Engine["yt-dlp.exe (核心引擎)"]
        Engine -->|拉取并发分片流| Web["网络视频源 (M3U8 / MP4 等)"]
        Worker -->|反射 WinRT 字典| Toast["Windows 原生 Toast 通知 (无闪烁进度条)"]
    end

    classDef default fill:#f9f9f9,stroke:#e0e0e0,stroke-width:1px;
    classDef highlight fill:#e1f5fe,stroke:#0288d1,stroke-width:2px;
    class Script,Server,Worker,Toast highlight;
```

1. 🌐 **`MediaHunter-8.3.7.user.js` (前端探针)**
   - 运行在浏览器的 Tampermonkey 环境下，负责网络请求拦截（XHR/Fetch重写）、DOM 扫描与媒体解析，并将获取到的资源地址加密发送至本地网关。
2. 🚪 **`Server.ps1` (本地安全网关)**
   - 常驻后台的微型 HTTP 服务器（`127.0.0.1:23333`），只接受特定的 POST 任务，拒绝所有外界探针。
3. ⚙️ **`Worker.ps1` (下载与 UI 执行器)**
   - 隐藏进程，负责接管 `yt-dlp` 命令，挂载输出流以渲染 Windows Toast 通知卡片。
4. 🚀 **`StartServer.vbs` (静默启动器)**
   - 动态获取目录，以无窗口模式拉起 Server 网关，不阻塞前台。
5. 📝 **`yt-dlp.conf` (引擎配置文件)**
   - 集中管理 `yt-dlp` 的核心运行参数，包括 Cookie 提取、请求头伪装、代理及分片下载策略。

---

## ✨ 核心特性 (Key Features)

### 🛡️ 极致安全与解耦 (Security & Decoupling)
* **无跨域本地网关 (No-CORS API)**：后端网关彻底关闭了跨域（CORS）许可，配合 Tampermonkey 的 `GM_xmlhttpRequest` 特权 API，完美切断了常规恶意网页的端口探测与 CSRF 盲打攻击。
* **配置外置化**：所有的底层下载引擎参数（代理、伪装、并发数、保存路径）全部收纳于独立的 `yt-dlp.conf` 配置文件中，实现代码与配置的完全解耦。

### ⚡ 极限性能 (Extreme Performance)
* **VBS 守护进程启动**：原生提供 `StartServer.vbs` 脚本，以最高隐蔽级别彻底隐藏 PowerShell 黑框，实现真正的无感常驻运行与开机自启。
* **纯 PS 反射原生通知**：摒弃 C# 动态编译，通过纯 PowerShell 内存反射直接操作底层 WinRT 字典，实现通知卡片“秒弹”。
* **OOM 防溢出环形缓冲区**：采用队列（Queue）机制，长达数小时的直播流下载也不会造成后台内存泄漏。

### 🎨 优雅体验 (Elegant UX)
* **无闪烁原生进度条**：调用 Windows 10/11 原生通知中心，实现真正的局部进度条刷新，告别频闪与假死。
* **智能日志回收 (Log Rotation)**：任务报错时自动在 `logs` 独立子目录下生成带 UUID 的诊断日志，并静默清理多余旧文件（仅保留最近 9 份），保持目录绝对整洁。

---

## 📊 优势对比 (Feature Comparison)

| 维度 / 特性 | 🌐 传统浏览器扩展 | 🖥️ 传统 GUI 下载器 (如 IDM) | 🎯 MediaHunter (本系统) |
| :--- | :--- | :--- | :--- |
| **反爬突破能力** | 🔴 弱（受限于扩展沙盒，难以突破高强度反爬） | 🟡 中（频繁更新规则，部分网站易失效） | 🟢 极强（基于 `yt-dlp` 强大的 TLS 指纹伪装与请求模拟） |
| **CORS 安全隔离** | 🟡 一般（若暴露本地接口，极易遭受本地网页 CSRF） | 🟡 一般（部分客户端提供公开 API 易被滥用） | 🟢 极高（零跨域网关配合 TM 脚本限制，拒绝盲打） |
| **前台交互打扰** | 🟢 低（一般静默在扩展栏内） | 🔴 高（频繁弹出下载确认框，打断操作） | 🟢 极低（纯后台静默运行，Windows 原生 Toast 轻量反馈） |
| **Cookie 提取能力** | 🟢 强（在浏览器沙盒内直接提取） | 🔴 弱（常因独占锁/加密数据库导致热读取失败） | 🟢 极强（Firefox 热提取，支持 Chrome/Edge 协作） |
| **长视频/直播录制** | 🔴 极弱（浏览器内存易溢出崩溃，无法长久录制） | 🟡 中（部分支持，但解析和合并效率较差） | 🟢 极强（基于 `yt-dlp` 分片并发下载与流式安全写入） |

---

## 🚀 安装与部署 (Installation & Deployment)

### 1. 环境准备
- **操作系统**：Windows 10 或 Windows 11（必须，依赖 Windows 原生 Toast 机制）。
- **依赖核心**：下载最新版的 [yt-dlp](https://github.com/yt-dlp/yt-dlp/releases) 并重命名为 `yt-dlp.exe`。
- **浏览器扩展**：[Tampermonkey](https://www.tampermonkey.net/)（油猴）。

### 2. 本地文件部署
在电脑中新建一个专属文件夹（例如 `D:\MediaHunter\`），将以下文件放入该目录中：
- `Server.ps1`
- `Worker.ps1`
- `StartServer.vbs`
- `yt-dlp.conf`
- `yt-dlp.exe`
- `icon.png` (可选，用于通知栏显示的图标)

### 3. 设置开机自启 (推荐)
为了获得最极致的体验，建议让网关在开机时自动静默运行：
1. 右键点击 `StartServer.vbs` -> **创建快捷方式**。
2. 按下 `Win + R` 键，输入 `shell:startup` 并回车，打开 Windows 启动文件夹。
3. 将刚才创建的快捷方式剪切到该文件夹中即可。
*(现在你可以双击 `StartServer.vbs` 直接启动服务，没有任何黑框打扰！)*

### 4. 浏览器端部署
在 Tampermonkey 中新建脚本，将 `MediaHunter-8.3.7.user.js` 的代码复制粘贴并保存。打开任意视频网站，右下角出现“媒体猎手”悬浮按钮即代表前端部署成功。

---

## 💻 服务运行与管理 (Service Management)

为了保证本系统能够实现极致纯净运行，后台服务未采用复杂的 GUI 管理工具，用户可以使用以下方式管理后台网关服务：

### 🟢 启动服务
- **手动启动**：直接双击项目目录中的 `StartServer.vbs` 即可在后台静默拉起 API 网关。
- **开机自动启动**：参考 [设置开机自启](#3-设置开机自启-推荐) 一节。

### 🔍 验证运行状态
如果您想确认本地网关是否已在后台成功监听，请打开 **PowerShell** 窗口，并执行以下命令：
```powershell
Get-Process -Name powershell | Where-Object { $_.CommandLine -like "*Server.ps1*" }
```
*若终端输出带有 `powershell` 的进程信息，说明网关已常驻后台监听中。*

### 🔴 一键停止服务
如需停用本系统，可随时打开 **PowerShell** 并执行以下命令：
```powershell
Get-Process -Name powershell | Where-Object { $_.CommandLine -like "*Server.ps1*" } | Stop-Process -Force
```
*提示：此命令只会精准停止本系统的 Server 网关进程，不会对您其他的正常 PowerShell 工作窗口造成任何影响。*

---

## ⚙️ 配置文件指南 (Configuration Guide)

本项目强烈推荐通过编辑 `yt-dlp.conf` 来调优和配置下载引擎的行为。

### 🦊 为什么默认推荐使用 Firefox？
由于现代浏览器的安全机制，Chrome / Edge 等 Chromium 系浏览器在运行时会通过操作系统级的 DPAPI 对 Cookie 数据库进行强加密和文件独占锁。这导致 `yt-dlp` 经常无法在浏览器未关闭的情况下成功提取 Cookie。

**Firefox** 的机制更为开放，`yt-dlp` 可以随时流畅地热读取 Firefox 的 Cookie，这也是为什么配置中默认采用 `--cookies-from-browser firefox` 且配合 `--impersonate "firefox"` 来绕过强力反爬校验的原因。

### 🟢 核心配置说明 (小白用户必看)
建议普通用户使用记事本打开 `yt-dlp.conf` 根据自身情况修改以下三处：

1. **修改下载路径**：
   找到 `--paths`，将路径修改为你电脑上实际存在的文件夹。
   ```text
   --paths "temp:D:\你的缓存文件夹"
   --paths "home:D:\你的最终下载文件夹"
   ```
2. **修改或关闭代理**：
   如果你没有使用科学上网，或者代理端口不是 `10808`，请修改代理端口或在前面加 `#` 注释掉该行：
   ```text
   #--proxy "socks5://127.0.0.1:10808"
   ```
3. **调整 Cookie 来源和 User-Agent**（如果不使用 Firefox）：
   将 `--cookies-from-browser firefox` 修改为 `--cookies-from-browser chrome` 或 `--cookies-from-browser edge`，同时 `--user-agent` 也请调整为对应匹配的 Chrome 或 Edge 浏览器的 User-Agent。

### 🔴 高级参数定制 (极客专用区)
针对有经验的用户，配置文件中预留了强大的反爬与性能参数，建议按需微调：
* `--concurrent-fragments 6`：开启 6 线程并发下载 M3U8 分片，极大提升下载速度。
* `--impersonate "firefox"`：底层 TLS 指纹伪装，用于突破 Cloudflare 等高防 CDN。
* `--add-header` 参数：注入了完整的原生浏览器 `Sec-Fetch-*` 跨域请求头，完美模拟人类真实访问。

> [!WARNING]
> `--cookies-from-browser`、`--impersonate`、`--user-agent`、`--add-header` 等参数具有高度关联性。若不清楚具体配置规则，请勿随意修改或添加 `--add-header`，以免由于请求头冲突被目标网站拦截。

---

## 🛠️ 故障排除与日志 (FAQ & Troubleshooting)

### Q1：点击下载后，Windows 右下角弹出“下载失败”的提示，该如何定位？
1. 打开本项目的部署目录，进入自动生成的 `logs/` 文件夹。
2. 找到最新的 `yt-dlp_error_[时间戳]_[UUID].log` 文件并用记事本打开。
3. 查看日志底部的错误日志并进行排查：
   - **报错包含 `Errno 2: No such file or directory`**：说明 `yt-dlp.conf` 中配置的缓存路径或下载路径在您的电脑中并不存在。请前往配置文件检查并修正 `--paths` 选项，或手动在电脑中创建对应文件夹。
   - **报错包含 `Failed to establish a new connection`**：说明网络代理无法接通。请确认您的代理工具运行正常，或在 `yt-dlp.conf` 中通过添加 `#` 注释掉 `--proxy` 选项。
   - **报错包含 `Sign in to confirm your age` 或 `HTTP Error 403: Forbidden`**：说明网站的 Cookie 提取失败或已过期。请确保您的浏览器已成功登录目标网站，并检查 `yt-dlp.conf` 中的 `--cookies-from-browser` 配置与您当前使用的浏览器是否匹配。

### Q2：启动时终端或日志提示“Server failed to start. Port might be in use.”？
* **原因**：本地端口 `23333` 已经被占用，或之前已经有一个网关进程在后台未正常退出。
* **解决办法**：请按照 [一键停止服务](#-一键停止服务) 部分的命令清理已有的后台进程，然后再重新双击 `StartServer.vbs` 运行。

---

## 📜 免责声明 (Disclaimer)

本项目仅供网络技术学习、防盗链机制研究及本地系统 API 调用测试使用。请遵守当地法律法规及目标网站的服务条款，严禁使用本工具下载或传播未经授权的版权内容。

---

*Created with ❤️ by yudong2ao & Gemini*