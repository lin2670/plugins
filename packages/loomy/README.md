# @magpie-community/opencode-loomy-auth

[OpenCode](https://opencode.ai) / [magpie](https://usemagpie.ai) provider 插件：**Loomy（讯飞，`loomyad.xunfei.cn`）**——登录、模型清单、聊天转发、双积分钱包与每日赠送积分刷新（签到）。

请求形态移植自 [aimod-cc/agent2api](https://github.com/aimod-cc/agent2api)（MIT）的 `providers/loomy`，按 magpie 插件 API 重写。

## 安装

```
magpie plugin add @magpie-community/opencode-loomy-auth
magpie plugin login loomy     # 选「短信验证码登录」或「手动粘贴 session」
```

## 能力

- **登录**：`短信验证码登录`（发码 → 输入 6 位码完成，session **14 天**）或 `手动粘贴 session`。
- **无续期接口**：session 过期（业务码 `020002`/`100002`）需重新登录（短信或粘贴）。
- **聊天**：`POST {modelBase}/chat/completions`，OpenAI 形态基本透传，两处适配（参照实现 `reasoning_patch` / `images.rs`）：思考档折进 low/medium/high（超档删除）、DeepSeek 系模型的 data-URI 图片转 `<image_base64>` 标记；SSE；请求头必须带 `charset=utf-8`（上游对无 charset 的 body 按 GBK 解码）。
- **模型**：`GET /api/v1/models`（OpenAI 形态；能力位读 `capabilities`；展示名尾部倍率 `(x3.0)` 拆为 rate 字段；无静态兜底——Loomy 官方网关没有内置清单）。
- **余额（usage）**：`GET /api/v2/points/records?...` → 永久积分 + 每日赠送积分两个窗口。
- **每日签到（checkin）**：`POST /api/v1/points/first-login`（body `{}`）——服务端按"当天是否第一次"把每日赠送刷回 5000；按天幂等，重复调=成功（`claimed`，附当前每日余额）。

## 账号域签名说明

登录两步（`POST https://account.xfinfr.com/login/phone/{sendMsgCode,checkCode}`）带 HMAC-SHA1 签名头（9 段待签串，AK/SK 为客户端内置常量），插件按参照实现同款实现（见 `index.mjs` 的 `signHeaders`）。

## 环境变量（测试 / 镜像用，与 agent2api 同名）

| 变量 | 默认 |
|---|---|
| `LOOMY_XFYUN_BASE_URL` | `https://account.xfinfr.com`（登录域，HMAC 签名） |
| `LOOMY_POINTS_BASE_URL` | `https://loomyad.xunfei.cn`（积分/签到/模型网关） |

## 测试

```
bun test packages/loomy
```

测试打到进程内假上游（`index.test.mjs` 内联 mock），不触网、不使用真实凭据。

## 许可

MIT。请求逻辑移植自 aimod-cc/agent2api（MIT）。
