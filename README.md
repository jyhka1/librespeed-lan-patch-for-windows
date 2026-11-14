# LibreSpeed 局域网测速 · 配置补丁包

> 同内容另有 `README.txt` 纯文本版，方便用记事本阅读

## 获取完整文件

本仓库首页只放说明文档。完整补丁包请到本仓库右侧的 **Releases** 下载 `librespeed-lan-patch.zip`（选最新版本），解压后得到 `settings.toml`、`assets/`、`start.bat`、`LICENSE` 以及这两份说明文档，再按第二节操作。

这是一份**非官方**的配置与前端文件补丁，用于把官方 `speedtest-go` 的 Windows release 包变成「局域网内手机测速到电脑」可直接使用的形态。

解决的问题：官方 1.1.5 release 包内没有前端文件，后端会回退使用 exe 内嵌的「多服务器示例页」，页面上的两个测试服务器并不存在，因此手机打开会显示 `No servers available`；只补首页又会因为缺少 `speedtest_worker.js` 而出现点 Start 数值全为 0。

**适用版本：官方 `speedtest-go` v1.1.5 Windows amd64 release**（Go 静态编译，无需安装 Go 或运行库）。本包**不含** `speedtest-backend.exe`，需要你自行从官方 Releases 下载。

---

## 一、补丁包里有什么（从 Releases 下载）

| 文件 | 作用 | 放进官方目录后 |
| --- | --- | --- |
| `settings.toml` | 后端配置 | **覆盖**同名文件 |
| `assets/` | 前端页面目录（5 个文件） | **新建**目录（官方 1.1.5 release 包里没有） |
| `start.bat` | Windows 启动脚本 | **新增** |
| `LICENSE` | LGPL-3.0 许可证全文 | **覆盖**同名文件（内容与官方一致） |
| `README.md` | Markdown 说明（本文件） | 给 GitHub 首页渲染用，**不需要**拖进官方目录 |
| `README.txt` | 同内容纯文本版 | 给记事本阅读用，**不需要**拖进官方目录 |
| `.gitignore` | 忽略规则 | 给 GitHub 仓库用，**不需要**拖进官方目录 |

`assets/` 内：`index.html`（仪表盘版首页）、`pretty.html`（文字版首页）、`speedtest.js`、`speedtest_worker.js`、`favicon.ico`。

关键配置差异：`listen_port=1119`、`assets_path="./assets"`。

---

## 二、使用步骤

> 前置：先按上文从 Releases 下载并解压 `librespeed-lan-patch.zip` 得到补丁包。

1. 从官方 Releases 下载并解压 **`speedtest-go_1.1.5_windows_amd64`**（解压后目录里应有 `speedtest-backend.exe`、`settings.toml`、`LICENSE`、`README.md`）。
2. 把本包的 `settings.toml`、`assets/`、`start.bat`、`LICENSE` **这四项拖进该目录**，提示覆盖时选「是」。
   - 若你之前改过 `settings.toml`，先自行备份一份再覆盖。
   - `assets/` 是**目录**不是单个文件；官方 1.1.5 release 包里原本没有它，拖进去等于新建。
   - 补丁包自带的 `README.md`、`README.txt` 和 `.gitignore` 是给 GitHub 仓库用的，**不需要**拖进官方目录，尤其别用本文件覆盖官方那份 `README.md`。
3. **双击 `start.bat`**（不要直接双击 exe；脚本会先切到自身目录，否则配置会因工作目录不对而失效）。窗口保持打开即代表服务在运行。
4. 首次运行若弹出 Windows 防火墙提示，**勾选「专用网络」并允许**。
5. 看到 `Starting backend server on :1119` 即启动成功：
   - 电脑本机：`http://127.0.0.1:1119/`
   - 手机：连**同一个 WiFi**，打开 `http://<电脑局域网IP>:1119/`，在电脑上用 `ipconfig` 查「IPv4 地址」（形如 `192.168.x.x`）。
6. 点 Start 开始测速。测完在窗口按 `Ctrl+C` 或直接关闭窗口即停止服务。

想切回文字版界面，访问 `http://<电脑IP>:1119/pretty.html`。

---

## 三、常见问题

- **`No servers available`**：说明配置没生效。多半是没用 `start.bat` 启动（工作目录不对），或后端用的是旧进程。关掉所有 `speedtest-backend.exe` 后重新双击 `start.bat`；日志出现 `using default assets` 就是没读到 `assets/`。
- **点 Start 数值全是 0**：`assets/speedtest_worker.js` 缺失或浏览器缓存了旧页面。确认文件存在，然后强制刷新 / 清缓存。
- **手机打不开页面**：确认电脑与手机在同一网段、防火墙已放行、路由器没开「AP 隔离」，以及电脑 IP 没变（重连 WiFi 后 IP 可能更换）。
- **`address already in use`**：端口 1119 被占用，或上次的后端没关干净。用 `netstat -ano | findstr 1119` 和 `tasklist | findstr speedtest-backend` 检查；也可把 `listen_port` 改成别的空闲端口。
- **下载测试时终端刷 `connection reset by peer`**：浏览器在测试阶段主动掐断未传完的并发请求，属正常现象，不影响结果。
- **想还原**：删除 `assets/` 和 `start.bat`，把 `settings.toml` 的 `listen_port` 改回 `8989`、`assets_path` 改回 `""` 即可。

---

## 四、隐私与安全提示

- 程序运行时会在同目录生成 `speedtest.db`（BoltDB 统计库），里面可能记录客户端 IP、User-Agent 和测速结果。**不要把这个文件提交到 GitHub 或分享给别人**。本包已用 `.gitignore` 排除。
- `settings.toml` 中 `statistics_password="PASSWORD"` 是官方占位默认值（等于未启用统计页）。若要启用统计功能，务必先改成自己的强密码。
- `ipinfo_api_key=""` 为空；若填入真实 API key，请勿公开提交。

---

## 五、许可与声明

本项目基于 LibreSpeed 的开源组件，采用 **GNU Lesser General Public License v3.0 or later (LGPL-3.0-or-later)**。你可以自由使用、修改和再分发，但需保留许可证全文与版权声明，并对修改作出说明。

- 后端 `speedtest-go`：Copyright (C) 2016-2020 Federico Dossena；Copyright (C) 2020 Maddie Zhan — <https://github.com/librespeed/speedtest-go>
- 前端 `librespeed/speedtest`：Copyright (C) 2016-2024 Federico Dossena — <https://github.com/librespeed/speedtest>
- `LICENSE` 为 LGPL v3 全文；`speedtest.js` 与 `speedtest_worker.js` 文件头保留了原作者版权注释，请勿删除。

**本包为第三方配置补丁，与 LibreSpeed 官方无关。** 不含任何后端二进制；`speedtest-backend.exe` 为官方 release v1.1.5 原样文件，其对应源码请见上方官方仓库的相应版本 tag。

改动记录（2026-11-14）：
- `settings.toml`：`listen_port` 8989 → 1119；`assets_path` 空 → `./assets`
- 新增 `assets/`：取自 `speedtest-go` v1.1.6 源码 `web/assets/` 的成套前端文件，其中 `index.html` 采用 `example-singleServer-gauges.html`（仪表盘版），另存文字版为 `pretty.html`
- 新增 `start.bat`：确保以自身目录为工作目录启动
