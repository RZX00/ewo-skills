---
name: ewo-video-generate
description: Generate short videos from a text prompt or a starting image. Use when the user asks 生成视频/做个视频/视频素材/图生视频/让图片动起来 or create/generate/animate a video clip. Free to start — new ewo accounts get a trial credit; then pay-as-you-go with your own ewo key (sk-eapi-…), no subscription.
---

# Video Generate — ewo open skill

Generates short clips through ewo's public video API, from text or from a
starting picture. Video is paid per second, by resolution, from a prepaid ewo
wallet; a new account comes with a free trial credit.

- Submit: `POST https://api.ewo.so/v1/videos/generations` (JSON body)
- Poll: `GET https://api.ewo.so/v1/videos/generations/<id>`
- Default model: `wan3.0-video`. Every video model your key can use is listed live by Step 2.

The commands are POSIX shell (macOS, Linux, Git Bash on Windows) and need only
`curl`. In PowerShell, call `curl.exe` and translate the loops.

## Step 1 — find the API key

`ORIGIN=https://api.ewo.so`. Use the first key you find, and never print it:

1. The environment variable `EWO_API_KEY`: `KEY=$EWO_API_KEY`.
2. The file `~/.ewo/api_key` (Windows: `%USERPROFILE%\.ewo\api_key`), which holds
   only the key: `KEY=$(tr -d ' \r\n' < ~/.ewo/api_key)`.
3. Older setups: a **file** `~/.ewo/credentials` with a line `api_key=sk-...`. When the
   ewo desktop app is installed, `~/.ewo/credentials` is its directory — never write there.

Keys start with `sk-eapi-`; older `sk-ewo-` keys still work.

**If there is no key, STOP — do not call the API.** Tell the user, in their
language:

> This skill uses ewo's image/video API. It's free to start:
> 1. Sign up at https://api.ewo.so/sign-up — new accounts get a free trial credit.
> 2. Create an API key at https://api.ewo.so/keys (it starts with `sk-eapi-`).
> 3. Save it as the only line of `~/.ewo/api_key` (or `export EWO_API_KEY=sk-eapi-...`),
>    then ask me again.

## Step 2 — pick another model (when needed)

Use the default unless the user names a model (match a loose name such as
"wan" to an id below), asks what is available, or the default
keeps failing. This lists every video model the key can use right now (free):

```bash
curl -sS --max-time 30 "$ORIGIN/v1/models" -H "Authorization: Bearer $KEY" \
  | tr '{' '\n' | grep '"x_habitat_capabilities":\[[^]]*"video"' | cut -d'"' -f4
```

## Step 3 — submit a video task

Write the body with your file tool (JSON-escape the prompt; never paste user text
into shell quotes), e.g. `/tmp/ewo-video-req.json`:

```json
{
  "model": "wan3.0-video",
  "prompt": "a neon koi swimming through a rainy Tokyo alley at night",
  "resolution": "720p",
  "duration": 5,
  "aspect_ratio": "16:9"
}
```

```bash
curl -sS --max-time 60 "$ORIGIN/v1/videos/generations" \
  -H "Authorization: Bearer $KEY" -H "Content-Type: application/json" \
  -d @/tmp/ewo-video-req.json -o /tmp/ewo-video-task.json
```

A good submit answers HTTP 200 with a task:
`{"id":"evt_…","status":"queued",…}`. The wallet is charged the estimated
maximum at submit; the difference comes back when the clip is done, and all of it
comes back if the task fails.

**Start from a picture (image-to-video).** Add
`"media": [{ "type": "first_frame", "url": "https://…/start.png" }]`. For a local
file (up to 20 MB) use a `data:` URL. The script below pipes the picture into the
body file, so its base64 never enters the chat or a command line (escape any `"`
or `\` in the prompt; write `image/jpeg` for a JPEG):

```bash
{ printf '{"model":"wan3.0-video","prompt":"%s","resolution":"720p","duration":5,"media":[{"type":"first_frame","url":"data:image/png;base64,' \
    "the cat slowly turns its head and blinks"
  base64 < start.png | tr -d '\n'
  printf '"}]}'
} > /tmp/ewo-video-req.json
```

### Fields

- `model` (string, required) — `wan3.0-video` unless the user picked another video model in Step 2.
- `prompt` (string, optional) — What the clip shows. Required unless media is given. Wan 3.0 accepts up to 20,000 characters.
- `media` (array, optional) — Pictures, clips or sounds to start from, each { type, url }; url is public HTTPS or a data: URL (images up to 20 MB). For image-to-video send first_frame, optionally with last_frame. Or send references instead: up to 10 reference_image, up to 5 reference_video and 5 reference_audio (each kind at most 15 s in total). file and link take at most one each. first_frame/last_frame cannot be mixed with reference_* items.
- `resolution` (string, optional) (one of: 480p, 720p, 1080p, 2k, 4k) — Wan 3.0 takes 480p, 720p or 1080p and defaults to 1080p, the most expensive tier; use 720p or 480p for drafts. Other models may also take 2k or 4k.
- `duration` (integer, optional) — Clip length in seconds. Wan 3.0: 2–30, default 5; -1 lets the model choose and reserves 30 seconds of balance.
- `aspect_ratio` (string, optional) (one of: 21:9, 16:9, 4:3, 1:1, 3:4, 9:16, adaptive)
- `generate_audio` (boolean, optional) — Also generate a soundtrack (Wan 3.0 `audio`).
- `seed` (integer, optional) — -1 for random, or 0 through 2147483647 to make a result repeatable.
- `prompt_extend` (boolean, optional) — Let the model expand the prompt with detail. Default true; file and link media require true.
- `watermark` (boolean, optional) — Add the provider watermark. Default false.

## Step 4 — wait for the clip, then download it

A 5-second clip usually takes 1–5 minutes.

```bash
field() { tr -d '\n' < "$2" | grep -o "\"$1\": *\"[^\"]*\"" | head -n 1 | cut -d'"' -f4; }
TASK=$(field id /tmp/ewo-video-task.json)
S=$(field status /tmp/ewo-video-task.json); i=0
while [ "$S" != completed ] && [ "$S" != failed ] && [ "$S" != cancelled ] && [ "$S" != expired ] && [ $i -lt 90 ]; do
  sleep 10; i=$((i+1))
  curl -sS --max-time 30 "$ORIGIN/v1/videos/generations/$TASK" \
    -H "Authorization: Bearer $KEY" -o /tmp/ewo-video-task.json
  S=$(field status /tmp/ewo-video-task.json)
done
echo "task $TASK: $S"
```

When the status is `completed`, the link is `result.data[0].url`. It stays valid
for at least a day, and polling the task again returns a fresh one. Tasks are kept
for 7 days; an older `completed` task has no link, only the error
`EWO_VIDEO_TASK_RESULT_EXPIRED`, so check before downloading:

```bash
URL=$(field url /tmp/ewo-video-task.json)
if [ -n "$URL" ]; then
  OUT="ewo-video-$(date +%s).mp4"   # or the path the user asked for
  curl -sSL --max-time 300 "$URL" -o "$OUT" && echo "saved: $OUT"
else
  grep -o '"message": *"[^"]*"' /tmp/ewo-video-task.json | head -n 1   # no link: this prints why
fi
```

`field` reads one value from the API's compact JSON. If it prints nothing where a
value should be, open the JSON file and read it yourself.

Tell the user the saved path. If the status is `failed`, `cancelled` or
`expired`, show them `error.message` from `/tmp/ewo-video-task.json`; the
reserved balance is refunded. If the loop gave up while the task was still
running, poll the same task id again later — a new submit is billed again.

## Failures (JSON `{ "error": { "code", "message" } }`) — do NOT blind-retry 4xx

- `401` (missing / invalid / expired key) → the user has no working ewo key.
  Walk them through **Step 1** signup (sign up at https://api.ewo.so/sign-up, create a key
  at https://api.ewo.so/keys), then stop.
- `402` `HABITAT_INSUFFICIENT_BALANCE` → the ewo wallet is empty or cannot cover
  this request. Tell the user, in their language, to recharge at https://api.ewo.so/console/topup,
  then stop. Do **not** retry — `402` is terminal.
- `400` → fix the request body; `error.message` names the field. An unknown model
  also lands here or on `404`: run Step 2 and pick a listed model.
- `429` → rate limit; wait the `Retry-After` seconds, then retry once.
- `503` or `524` → temporary capacity or edge timeout; wait a few seconds and retry
  once.
