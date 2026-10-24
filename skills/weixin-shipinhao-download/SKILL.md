---
name: weixin-shipinhao-download
display_name: 微信视频号下载
display_name_en: WeChat Channels Downloader
description: 微信视频号下载。把一条或多条 https://weixin.qq.com/sph/... 微信视频号分享链接保存成本地 MP4。当用户发来微信视频号/视频号链接，或说"下载/保存/备份微信视频号""存一下这个视频号视频""把视频号弄到本地"时使用。用法：本技能 + 视频号链接。仅用于本人发布或已获授权保存的内容。
description_zh: 微信视频号下载。把一条或多条 https://weixin.qq.com/sph/... 分享链接保存成本地 MP4。用法：本技能 + 视频号链接。
description_en: Download WeChat Channels videos to local MP4 from one or more https://weixin.qq.com/sph/... share links.
version: 1.0.0
---

# 微信视频号下载

**用法：本技能 + 一条（或多条）微信视频号链接。** 一条用 `download`，多条用 `batch`，保存完把 MP4 绝对路径报给用户。默认存到 `~/Downloads/视频号一键存`。

## 1. 命令

macOS：

```bash
sh "<Skill目录>/scripts/run.sh" download "https://weixin.qq.com/sph/示例ID"
sh "<Skill目录>/scripts/run.sh" batch --url "https://weixin.qq.com/sph/第一条" --url "https://weixin.qq.com/sph/第二条"
```

Windows（PowerShell）：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "<Skill目录>\scripts\run.ps1" download "https://weixin.qq.com/sph/示例ID"
powershell -NoProfile -ExecutionPolicy Bypass -File "<Skill目录>\scripts\run.ps1" batch --url "https://weixin.qq.com/sph/第一条" --url "https://weixin.qq.com/sph/第二条"
```

常用参数：`--output-dir "<绝对路径>"` 改保存位置；`--codec h264|h265`（默认 h264）；
`--workers 1..4`（批量并行数，默认 3）；`--login-wait 30..600`（默认 240 秒）。

## 2. 环境

只支持 Windows / macOS，需要 Python 3.10+。一条命令自检：

```bash
sh "<Skill目录>/scripts/run.sh" doctor
```

`status: ready` 就可以下载。缺 Playwright 时先跑一次安装脚本（Windows 用 `setup.ps1`）：

```bash
sh "<Skill目录>/scripts/setup.sh"
```

本机如果没有 Chrome / Edge 等 Chromium 系浏览器，用装好的隔离环境再装一份自带 Chromium：

```bash
"$HOME/Library/Application Support/wxvideo-local/runtime/bin/python" -m playwright install chromium
```

浏览器按 `Chrome → Edge → Brave → Vivaldi → Chromium → Playwright 自带 Chromium → Arc` 的顺序尝试，
每次启动限时 20 秒，失败自动换下一个；也可设 `WXVIDEO_BROWSER` 指定可执行文件（此时只用它）。

## 3. 登录

首次（以及登录态过期后）会弹出专用浏览器窗口，需**用户本人扫码登录元宝**。助手不代登录、不读取现有浏览器会话。
登录状态存在本机的 `wxvideo-local` 资料目录里，**同一轮里的批量任务只扫一次码**。

## 4. 边界

- 单次批量最多 50 条链接，自动去重。
- **只保存本人发布或明确获授权的内容**；不下载私密、付费、删除或无授权内容；平台限流就停止解析，不换号绕限流。
- 专用浏览器资料目录含登录状态，是本机敏感数据，不要打包、同步到公开云盘或分享。
- 底层依赖网页的内部行为，不是官方公开下载 API，接口变化会导致失败。

细则见 [references/usage.md](references/usage.md)：常见错误码、清单文件、并行说明、
浏览器选择、限制与已知问题。
