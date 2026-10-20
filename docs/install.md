# 给 Agent 的安装说明

本文件写给 Agent。用户把本文件的链接发给你，是请你把 ewo-skills 里的技能装进你自己的技能目录。按下面的顺序做，做完向用户汇报。

## 1 要装什么

仓库 `RZX00/ewo-skills` 里有两个技能，各自只有一个 `SKILL.md`：

| 技能 | 用途 | 原始文件 |
|---|---|---|
| `ewo-image-generate` | 生成和编辑图片 | `https://raw.githubusercontent.com/RZX00/ewo-skills/main/ewo-image-generate/SKILL.md` |
| `ewo-video-generate` | 文生视频、图生视频 | `https://raw.githubusercontent.com/RZX00/ewo-skills/main/ewo-video-generate/SKILL.md` |

用户没有说明时，两个都装；用户只提到画图或只提到视频时，只装对应的一个。

## 2 装到哪里

放进你自己的技能目录，路径为 `<技能目录>/<技能名>/SKILL.md`。

- 你应当知道自己的技能目录在哪。例如 Claude Code 的用户级技能目录是 `~/.claude/skills/`（Windows 为 `%USERPROFILE%\.claude\skills\`）。
- 不确定时问用户，不要猜。

## 3 下载并保存

1. 下载上表中的原始文件，原样保存，不要改动内容。
2. 保存后确认文件开头是 `---`，且含 `name:` 和 `description:`，名字与目录名一致。

以 Claude Code 为例，装图像技能的一行命令：

```bash
curl -fsSL https://raw.githubusercontent.com/RZX00/ewo-skills/main/ewo-image-generate/SKILL.md --create-dirs -o ~/.claude/skills/ewo-image-generate/SKILL.md
```

## 4 确认已加载

重新加载技能（具体做法以你所在 Agent 的文档为准，多数需要新开一个会话）后，确认技能列表里能看到 `ewo-image-generate` 或 `ewo-video-generate`。看不到时如实告诉用户，不要说已经装好。

## 5 检查 key

两个技能都需要用户自己的 ewo API key，按下列顺序查找，**只检查是否存在，不要把 key 打印出来**：

1. 环境变量 `EWO_API_KEY`；
2. 文件 `~/.ewo/api_key`（Windows 为 `%USERPROFILE%\.ewo\api_key`），内容只有 key 一行。

两处都没有时，把下面三步告诉用户，让用户自己完成，然后停下：

1. 在 https://api.ewo.so/sign-up 注册。
2. 在 https://api.ewo.so/keys 创建 key（以 `sk-eapi-` 开头）。
3. 把 key 单独一行写进 `~/.ewo/api_key`，或在 shell 里 `export EWO_API_KEY=sk-eapi-...`。

## 6 不要做

- 不要替用户注册账号或创建 key，不要让用户把 key 贴进对话。
- 不要往 `~/.ewo/credentials` 写任何东西：装了 ewo 桌面端的电脑上，它是桌面端自己的目录。
- 没有 key 时不要调用 API；有 key 也不要为了测试安装而生成图片或视频，每次生成都会从用户的钱包扣费。
- 不要修改 `SKILL.md`。它由 ewo 的能力清单生成，改了会在下次更新时被覆盖。

## 7 汇报

装完后告诉用户：装了哪些技能、保存在哪个路径、是否确认已加载、key 是否就绪，以及下一步可以对你说的第一句话，例如：

> 用 ewo-image-generate 画一只水彩风格的橘猫
