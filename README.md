# skills

- [bookkeeper-persona](./skills/bookkeeper-persona/SKILL.md) — 为[妙蛙记账](https://miaowa.sugarat.top/docs/personas/)生成自定义人设（名称、介绍、开场白、提示词）
- [weixin-shipinhao-download](./skills/weixin-shipinhao-download/SKILL.md) — **微信视频号下载**：把 `https://weixin.qq.com/sph/...` 分享链接存成本地 MP4

```bash
npx skills add ATQQ/skills -g
npx skills add ATQQ/skills@bookkeeper-persona -g
npx skills add ATQQ/skills@weixin-shipinhao-download -g
```

新增：`npx skills init skills/<name>`，约定见 [AGENTS.md](./AGENTS.md)。

---

## weixin-shipinhao-download · 怎么用

**一句话**：装好技能后，在对话里发一条（或多条）微信视频号分享链接，让助手下载即可。

### 1. 安装

```bash
npx skills add ATQQ/skills@weixin-shipinhao-download -g
```

或从本仓本地安装：

```bash
npx skills add . -g
```

### 2. 对话里怎么说

把链接丢给助手，例如：

- 「用微信视频号下载，把这个存本地：`https://weixin.qq.com/sph/xxxx`」
- 「下载这两条视频号」+ 多条链接

助手会走 `download`（单条）或 `batch`（多条），默认保存到 `~/Downloads/视频号一键存`，完成后回报 MP4 绝对路径。

### 3. 首次准备（只需一次）

- 系统：Windows / macOS，Python 3.10+
- 对话里可让助手先跑自检：`doctor`；缺 Playwright 时跑 `setup`
- **首次下载**会弹出专用浏览器窗口，需**本人用微信扫码登录元宝**；之后同一轮批量只扫一次。登录态过期后再扫即可

### 4. 注意

- 仅保存**本人发布或已获授权**的内容
- 链接须是 `https://weixin.qq.com/sph/...` 形式
- 细则（参数、错误码、清登录数据）见 [skills/weixin-shipinhao-download/references/usage.md](./skills/weixin-shipinhao-download/references/usage.md)
