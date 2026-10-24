# 微信视频号下载 · 使用细则

## 给用户看的步骤

把本人发布或已获授权的微信视频号链接发出来，助手直接开始保存，没有次数限制。
首次（以及登录态过期后）仍需本人在弹出的专用浏览器窗口里扫码登录元宝。

## 准备环境

支持 Windows 10/11 与 macOS，其他系统未适配。需要 Python 3.10+。Chromium 系浏览器可选 ——
找不到系统浏览器时用 Playwright 自带 Chromium。

```bash
sh "<Skill目录>/scripts/run.sh" doctor          # 自检，不登录、不访问链接
sh "<Skill目录>/scripts/setup.sh"               # 装 Playwright（Windows 用 setup.ps1）
```

安装脚本只在本机专用目录建虚拟环境并装 `requirements.txt` 里固定版本的 Playwright，
**不安装 Playwright 自带浏览器、不改根证书或系统代理**。运行目录：
Windows `%LOCALAPPDATA%\wxvideo-local\runtime`，macOS `~/Library/Application Support/wxvideo-local/runtime`。

需要自带浏览器时：

```bash
"$HOME/Library/Application Support/wxvideo-local/runtime/bin/python" -m playwright install chromium
```

## 浏览器选择

启动顺序：

1. `WXVIDEO_BROWSER`（若设置，则**只用**它，不回退）；
2. 自动探测到的系统浏览器：Chrome → Edge → Brave → Vivaldi → Chromium；
3. 探测到的 Playwright 自带 Chromium（`~/Library/Caches/ms-playwright/chromium-*/...`）；
4. Playwright 默认解析（`executable_path=None`）；
5. **Arc**（兜底）。

每次启动限时 20 秒（`LAUNCH_TIMEOUT_MS`），失败自动换下一个；全失败报 `BROWSER_START_FAILED`
并列出各候选的失败类型。

> **Arc 实测结论（macOS 1.159.0 + Playwright 1.63.0）**：Arc 能被拉起（命令行带上
> `--user-data-dir` 与 `--remote-debugging-pipe`），但 `launch_persistent_context` **永不返回**，
> 卡到手写看门狗超时 —— Arc 的 CDP 握手不兼容。因此 Arc 不参与自动探测，只在最后兜底；
> 要用只能显式指定 `WXVIDEO_BROWSER`。

## 下载一条

```bash
sh "<Skill目录>/scripts/run.sh" download "https://weixin.qq.com/sph/示例ID"
```

成功返回 MP4 绝对路径、字节数、标题、SHA-256；失败返回错误码并清理临时 `.part` 文件，
不会把未完成文件伪装成 MP4。SHA-256 是本地文件指纹，不是平台原件校验值。

## 一次保存多条

每批最多 50 条，重复传 `--url`，**不要把多条拼进一个参数**。

```bash
sh "<Skill目录>/scripts/run.sh" batch --url "https://weixin.qq.com/sph/第一条" --url "https://weixin.qq.com/sph/第二条"
```

也支持 `--links-file "<UTF-8 清单>"`（每行一条，≤64 KiB）与 `--workers 1..4`（默认 3）。

整批共用一次专用浏览器登录；解析逐条进行，文件下载并行；重复链接自动跳过；单条失效不影响其他链接；
遇到平台限流停止解析剩余链接并对后续条目标 `RATE_LIMITED`，已开始的下载会收尾。

结束时返回成功 / 失败 / 跳过数量、各 MP4 路径，并在保存目录生成 `wxvideo_batch_*.json` 清单
（含分享链接和本地路径，留在本机；不含 Cookie、票据或媒体地址）。
`partial_success` 表示只有部分保存成功，**不能说成整批成功**。

## 常见失败

| 错误码 | 含义与处理 |
|---|---|
| `PLATFORM_UNSUPPORTED` | 只支持 Windows 与 macOS |
| `PLAYWRIGHT_MISSING` | 先跑对应 `setup` 脚本 |
| `BROWSER_START_FAILED` | 所有候选都启动失败；关掉旧窗口重试，或用 `WXVIDEO_BROWSER` 指定 |
| `BROWSER_MISSING` | `WXVIDEO_BROWSER` 指向的文件不存在 |
| `LOGIN_REQUIRED` | 重新运行，在可见的专用窗口里由用户本人扫码 |
| `INVALID_LINK` | 输入必须是一条 `https://weixin.qq.com/sph/...` |
| `NO_LINKS` / `TOO_MANY_LINKS` | 批量没给链接，或超过每批 50 条 |
| `RATE_LIMITED` / `FEED_HTTP_429` | 平台限流；**不得换账号或代理规避** |
| `PARSE_REJECTED` / `NO_PLAYABLE_URL` / `FEED_REJECTED` | 检查链接可访问性与保存权；也可能是腾讯网页接口变化 |
| `UNTRUSTED_MEDIA_URL` | 媒体域名不在白名单内，人工核对来源，不要关闭校验 |
| `DOWNLOAD_UNAVAILABLE` / `INCOMPLETE_DOWNLOAD` | 检查网络与磁盘空间后重试一次 |

## 退出登录 / 清掉本机数据

先关闭专用浏览器窗口，再删除本机目录：
macOS `~/Library/Application Support/wxvideo-local`；Windows `%LOCALAPPDATA%\wxvideo-local`。
这不会影响日常浏览器配置。

## 验证状态（macOS / Python 3.14.0 / Playwright 1.63.0 实测）

| 项 | 结果 |
|---|---|
| `doctor` | `status: ready`，`missing: []` |
| 自带 Chromium 启动 | 成功，1.9 秒；成功打开 `yuanbao.tencent.com` 登录页 |
| Arc 启动 | **失败**：`launch_persistent_context` 挂住直至看门狗超时 |
| `download` 端到端 | 成功，扫码登录后产出 MP4，SHA-256 复核一致 |
| `batch` | 未端到端测试（同一套解析链路） |
| 登录态持久性 | 隔约 7 天后失效，需重新扫码 |

## 限制与已知问题

- **适用范围**：仅用于本人发布或明确获授权保存的视频。仍需遵守著作权、内容授权与平台规则。
  不提供无授权批量搬运、私密/付费内容绕过、风控规避、去水印承诺、云端代存。
- **依赖网页内部行为**：不是官方公开下载 API。接口变化会导致失败，不承诺永久可用、免登录或零配置。
- **敏感目录**：专用浏览器资料目录（`wxvideo-local/browser-profile`）含登录状态，
  不要打包、同步到公开云盘或分享。
- **不改系统**：不安装根证书、不修改系统代理、不调用外部模型或模型 API Key。
- **Arc 不可用于自动化**（macOS 1.159.0 实测）：进程能拉起，但 CDP 握手不完成，
  `launch_persistent_context` 永不返回。已排在候选最后，需用只能显式设 `WXVIDEO_BROWSER`。
- **选自带 Chromium 时要按 revision 取**：`~/Library/Caches/ms-playwright` 里可能同时存在多个
  `chromium-*`，必须取当前 Playwright 的 `browsers.json` 里声明的那个（脚本已自动处理）。
  Playwright ≥1.5x 在 macOS 上自带的浏览器是 `chrome-mac-arm64/Google Chrome for Testing.app/...`，
  不再是 `Chromium.app`。
- **装浏览器可能被安全软件拦**：`playwright install chromium` 会尝试清理一个空的
  `ms-playwright/__dirlock`，某些沙箱会拒绝；所需 revision 已存在时无影响，真要重装先手动删该目录。
