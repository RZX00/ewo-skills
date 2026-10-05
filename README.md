<div align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="./assets/ewo-logo-dark.svg" />
    <img src="./assets/ewo-logo.svg" alt="ewo" width="220" />
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
| [`ewo-video-generate`](./ewo-video-generate/) | 文生视频、图生视频 | `wan3.0-video` |

## 安装

### 图像生成与编辑 · `ewo-image-generate`

**一键安装** —— 把下面这段发给你的 agent（Claude Code / Codex / Cursor / Cline …），它会自己放到该放的地方：

```text
帮我装一个 agent skill：获取 https://raw.githubusercontent.com/RZX00/ewo-skills/main/ewo-image-generate/SKILL.md，原样保存成一个名为 ewo-image-generate 的技能（放到你自己的 skills 目录），然后加载它并告诉我装好了。
```

**或手动装** —— 把 `SKILL.md` 下载到你的 agent 的 skills 目录（下面以 Claude Code 的 `~/.claude/skills/` 为例，其他 agent 换成各自的目录）：

```bash
curl -fsSL https://raw.githubusercontent.com/RZX00/ewo-skills/main/ewo-image-generate/SKILL.md --create-dirs -o ~/.claude/skills/ewo-image-generate/SKILL.md
```

### 文生视频、图生视频 · `ewo-video-generate`

**一键安装** —— 把下面这段发给你的 agent（Claude Code / Codex / Cursor / Cline …），它会自己放到该放的地方：

```text
帮我装一个 agent skill：获取 https://raw.githubusercontent.com/RZX00/ewo-skills/main/ewo-video-generate/SKILL.md，原样保存成一个名为 ewo-video-generate 的技能（放到你自己的 skills 目录），然后加载它并告诉我装好了。
```

**或手动装** —— 把 `SKILL.md` 下载到你的 agent 的 skills 目录（下面以 Claude Code 的 `~/.claude/skills/` 为例，其他 agent 换成各自的目录）：

```bash
curl -fsSL https://raw.githubusercontent.com/RZX00/ewo-skills/main/ewo-video-generate/SKILL.md --create-dirs -o ~/.claude/skills/ewo-video-generate/SKILL.md
```

## 配置你的 ewo Key（只需一次）

技能需要一个 `sk-eapi-` 开头的 key（以前发的 `sk-ewo-` key 也能用）。第一次用时 agent 会引导你，也可以现在就配好：

1. **注册**：https://api.ewo.so/sign-up —— 新账号自带免费试用额度。
2. **建 key**：https://api.ewo.so/keys （`sk-eapi-` 开头）。
3. **保存**：二选一
   - 把 key 单独一行写进 `~/.ewo/api_key`，或
   - shell 里 `export EWO_API_KEY=sk-eapi-...`。

装了 ewo 桌面端的电脑上，`~/.ewo/credentials` 是桌面端自己的文件夹，不要往里写 key。

额度用完了？在 https://api.ewo.so/console/topup 充值。

## 试一下

Key 配好后，直接对你的 agent 说：

> 画一只水彩风格的橘猫
>
> 把这张照片的背景换成海边
>
> 生成一段霓虹东京雨夜的 5 秒短视频
>
> 让这张图动起来，做成 5 秒视频

## 工作原理

每个技能都是一个自包含的 `SKILL.md`，只用 `curl`，用你的 key 调 ewo 的公开 OpenAI 兼容端点：
图片走 `https://api.ewo.so/v1/images/generations` 和 `/v1/images/edits`，提交成异步任务后轮询；
视频走 `https://api.ewo.so/v1/videos/generations`，提交后轮询到完成再下载。
只为生成成功的内容付费，从预付 ewo 钱包扣。

---

<sub>本目录由 ewo 能力清单自动生成，请勿手改 `SKILL.md`；维护者用 `pnpm gen:oss-skills` 重新生成。</sub>
