<div align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="./assets/ewo-logo-dark.svg" />
    <img src="./assets/ewo-logo.png" alt="ewo" width="220" />
  </picture>

  <h1>ewo skills</h1>

  <p><strong>给你的 AI agent 用的开源「图像 / 视频生成」技能。</strong></p>
  <p>免费起步 · 多模型 · 按量付费 · 无订阅</p>
</div>

---

即插即用的 [agent skill](https://docs.claude.com/en/docs/agents/skills)，由
[ewo](https://api.ewo.so) 的公开、OpenAI 兼容媒体 API 驱动。装进 Claude Code、
Codex、Cursor、Cline，或任何能读 `SKILL.md` 的 agent 即可用。

**免费起步** —— 新注册的 ewo 账号自带一笔试用额度，前几次生成不花钱；之后按量从预付钱包扣费，无订阅。

| 技能 | 用途 | 默认模型 |
|---|---|---|
| [`ewo-image-generate`](./ewo-image-generate/) | 图像生成与编辑 | `gpt-image-2` |
| [`ewo-video-generate`](./ewo-video-generate/) | 文生视频 | `seedance-2-fast` |

## 安装

### 图像生成与编辑 · `ewo-image-generate`

**一键安装** —— 把下面这段发给你的 agent（Claude Code / Codex / Cursor / Cline …）：

```text
帮我装一个 agent skill：把 https://raw.githubusercontent.com/RZX00/ewo-skills/main/ewo-image-generate/SKILL.md 下载到 ~/.claude/skills/ewo-image-generate/SKILL.md（没有目录就创建），保持内容不变，然后加载它并告诉我装好了。
```

**或一行命令装：**

```bash
curl -fsSL https://raw.githubusercontent.com/RZX00/ewo-skills/main/ewo-image-generate/SKILL.md --create-dirs -o ~/.claude/skills/ewo-image-generate/SKILL.md
```

### 文生视频 · `ewo-video-generate`

**一键安装** —— 把下面这段发给你的 agent（Claude Code / Codex / Cursor / Cline …）：

```text
帮我装一个 agent skill：把 https://raw.githubusercontent.com/RZX00/ewo-skills/main/ewo-video-generate/SKILL.md 下载到 ~/.claude/skills/ewo-video-generate/SKILL.md（没有目录就创建），保持内容不变，然后加载它并告诉我装好了。
```

**或一行命令装：**

```bash
curl -fsSL https://raw.githubusercontent.com/RZX00/ewo-skills/main/ewo-video-generate/SKILL.md --create-dirs -o ~/.claude/skills/ewo-video-generate/SKILL.md
```

## 配置你的 ewo Key（只需一次）

技能需要一个 `sk-ewo-` 开头的 key。第一次用时 agent 会引导你，也可以现在就配好：

1. **注册**：https://api.ewo.so/sign-up —— 新账号自带免费试用额度。
2. **建 key**：https://api.ewo.so/keys （`sk-ewo-` 开头）。
3. **保存**：二选一
   - shell 里 `export EWO_API_KEY=sk-ewo-...`，或
   - 写进 `~/.ewo/credentials`：`api_key=sk-ewo-...`。

额度用完了？在 https://api.ewo.so/console/topup 充值。

## 试一下

Key 配好后，直接对你的 agent 说：

> 画一只水彩风格的橘猫
>
> 生成一段霓虹东京雨夜的短视频

## 工作原理

每个技能都是一个自包含的 `SKILL.md`，用你的 key 调 ewo 的公开 OpenAI 兼容端点
（`POST https://api.ewo.so/v1/images/generations`、
`POST https://api.ewo.so/v1/videos/generations`）。只为你生成的内容付费，从预付
ewo 钱包扣。

---

<sub>本目录由 ewo 能力清单自动生成，请勿手改 `SKILL.md`；维护者用 `pnpm gen:oss-skills` 重新生成。</sub>
