<div align="center">
  <img src="./assets/ewo-logo.png" alt="ewo" width="220" />

  <h1>ewo skills</h1>

  <p><strong>Open-source image &amp; video generation skills for your AI agent.</strong></p>
  <p>Free to start · multi-model · pay-as-you-go · no subscription</p>
</div>

---

Drop-in [agent skills](https://docs.claude.com/en/docs/agents/skills) for image and
video generation, powered by [ewo](https://api.ewo.so)'s public,
OpenAI-compatible media API. Install into Claude Code, Codex, Cursor, Cline, or any
agent that reads `SKILL.md`.

**Free to start** — a new ewo account comes with a trial credit, so your first few
generations are on us. After that it's pay-as-you-go from a prepaid wallet.

| Skill | What it does | Models |
|---|---|---|
| [`ewo-image-generate`](./ewo-image-generate/) | Image Generate & Edit | `gpt-image-2` + 3 more |
| [`ewo-video-generate`](./ewo-video-generate/) | Video Generate | `seedance-2-fast` + 2 more |

## Install

### Image Generate & Edit — `ewo-image-generate`

**One-click install** — paste this to your agent (Claude Code, Codex, Cursor, Cline, …):

```text
Install an agent skill for me. Create the folder `~/.claude/skills/ewo-image-generate/` (or your agent's skills directory), download https://raw.githubusercontent.com/RZX00/ewo-skills/main/ewo-image-generate/SKILL.md into it as `SKILL.md`, then load it and tell me it's ready. Do not change its contents.
```

Or install manually:

```bash
mkdir -p ~/.claude/skills/ewo-image-generate
curl -fsSL https://raw.githubusercontent.com/RZX00/ewo-skills/main/ewo-image-generate/SKILL.md \
  -o ~/.claude/skills/ewo-image-generate/SKILL.md
```

### Video Generate — `ewo-video-generate`

**One-click install** — paste this to your agent (Claude Code, Codex, Cursor, Cline, …):

```text
Install an agent skill for me. Create the folder `~/.claude/skills/ewo-video-generate/` (or your agent's skills directory), download https://raw.githubusercontent.com/RZX00/ewo-skills/main/ewo-video-generate/SKILL.md into it as `SKILL.md`, then load it and tell me it's ready. Do not change its contents.
```

Or install manually:

```bash
mkdir -p ~/.claude/skills/ewo-video-generate
curl -fsSL https://raw.githubusercontent.com/RZX00/ewo-skills/main/ewo-video-generate/SKILL.md \
  -o ~/.claude/skills/ewo-video-generate/SKILL.md
```

## Set up your ewo key (once)

The skill needs a `sk-ewo-` key. Your agent will walk you through this the first
time, or do it now:

1. **Sign up** at https://api.ewo.so/sign-up — new accounts get a free trial credit.
2. **Create a key** at https://api.ewo.so/keys (it starts with `sk-ewo-`).
3. **Save it** — either:
   - `export EWO_API_KEY=sk-ewo-...` in your shell, or
   - put `api_key=sk-ewo-...` in `~/.ewo/credentials`.

Out of credit? Recharge at https://api.ewo.so/console/topup.

## Try it

Once your key is set, just ask your agent:

> 画一只水彩风格的橘猫 · draw a watercolor orange cat
>
> 生成一段霓虹东京雨夜的短视频 · make a short neon-Tokyo-rain video

## How it works

Each skill is a self-contained `SKILL.md` that calls ewo's public
OpenAI-compatible endpoints (`POST https://api.ewo.so/v1/images/generations`,
`POST https://api.ewo.so/v1/videos/generations`) with your key. You pay only
for what you generate, billed to your prepaid ewo wallet.

---

<sub>Generated from the ewo capability manifests — do not edit `SKILL.md` by hand.
Maintainers regenerate with `pnpm gen:oss-skills`.</sub>
