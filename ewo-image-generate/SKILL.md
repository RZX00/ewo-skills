---
name: ewo-image-generate
description: Generate or edit images. Use whenever the user asks to draw, create, or modify any picture, illustration, logo, poster, icon, or photo — 画图/画一张/生成图片/改图/做个logo/海报/插画/头像. Free to start — new ewo accounts get a trial credit; then pay-as-you-go with your own ewo key (sk-eapi-…), no subscription.
---

# Image Generate & Edit — ewo open skill

Generates and edits images through ewo's public, OpenAI-compatible Images API.
Each image is paid from a prepaid ewo wallet; a new account comes with a free
trial credit, so the first images cost nothing.

- Generate: `POST https://api.ewo.so/v1/images/generations` (JSON body)
- Edit or use reference pictures: `POST https://api.ewo.so/v1/images/edits` (multipart upload)
- Poll: `GET https://api.ewo.so/v1/image-jobs/<id>`
- Default model: `gpt-image-2`. Every image model your key can use is listed live by Step 2.

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
"2.5" or "nano banana" to an id below), asks what is available, or the default
keeps failing. This lists every image model the key can use right now (free):

```bash
curl -sS --max-time 30 "$ORIGIN/v1/models" -H "Authorization: Bearer $KEY" \
  | tr '{' '\n' | grep '"x_habitat_capabilities":\[[^]]*"image"' | cut -d'"' -f4
```

## Step 3 — submit an image job

A render takes 30 seconds to a few minutes — longer than the API edge keeps a
request open — so always submit an **async job** (`async: true`) and poll it.
Ask for `response_format: "url"` so the result is a short link, never megabytes
of base64.

**Generate.** Write the body with your file tool (JSON-escape the prompt; never
paste user text into shell quotes), e.g. `/tmp/ewo-image-req.json`:

```json
{
  "model": "gpt-image-2",
  "prompt": "a fluffy orange cat, watercolor style",
  "size": "1024x1024",
  "response_format": "url",
  "async": true
}
```

```bash
curl -sS --max-time 60 "$ORIGIN/v1/images/generations" \
  -H "Authorization: Bearer $KEY" -H "Content-Type: application/json" \
  -d @/tmp/ewo-image-req.json -o /tmp/ewo-image-job.json
```

**Edit a picture, or draw from reference pictures.** Upload the files as multipart.
Write the instruction to `/tmp/ewo-prompt.txt` first — `prompt=<file` makes curl
read it from there:

```bash
curl -sS --max-time 120 "$ORIGIN/v1/images/edits" -H "Authorization: Bearer $KEY" \
  -F model=gpt-image-2 -F response_format=url -F async=true \
  -F "prompt=</tmp/ewo-prompt.txt" -F "image=@photo.png" \
  -o /tmp/ewo-image-job.json
```

Repeat `-F "image=@other.png"` for more references. To repaint only part of the
picture with `gpt-image-2`, add `-F "mask=@mask.png"`: a PNG the size of the source
whose transparent pixels mark the area to change.

### Fields

- `prompt` (required) — what to draw, or the change to make.
- `model` — default `gpt-image-2`, or any id from Step 2. Gemini models render at
  about 1K whatever `size` says; use `gpt-image-2` for 2K or 4K.
- `size` — pixels `WIDTHxHEIGHT`: `1024x1024` square, `1024x1536` portrait,
  `1536x1024` landscape, `2048x1152` 2K 16:9, `3840x2160` 4K 16:9. Do not send
  aspect ratios such as `16:9`.
- `quality` — leave it out; ewo asks each model for its best tier.
- `background` — `transparent`, `opaque` or `auto`.
- `output_format` — `png`, `jpeg` or `webp`.
- Do not send `resolution`: it does not change the pixels, but it changes the price tier.

A good submit answers HTTP 202 with a job:
`{"object":"image.job","id":"imgjob_…","status":"queued",…}`. Anything else is an
error — see Failures.

## Step 4 — wait for the job, then save the image

```bash
field() { tr -d '\n' < "$2" | grep -o "\"$1\": *\"[^\"]*\"" | head -n 1 | cut -d'"' -f4; }
JOB=$(field id /tmp/ewo-image-job.json)
S=$(field status /tmp/ewo-image-job.json); i=0
while [ "$S" != succeeded ] && [ "$S" != failed ] && [ "$S" != cancelled ] && [ "$S" != expired ] && [ $i -lt 120 ]; do
  sleep 5; i=$((i+1))
  curl -sS --max-time 30 "$ORIGIN/v1/image-jobs/$JOB" \
    -H "Authorization: Bearer $KEY" -o /tmp/ewo-image-job.json
  S=$(field status /tmp/ewo-image-job.json)
done
echo "job $JOB: $S"
```

When the status is `succeeded`, fetch the result and download it right away:

```bash
curl -sS --max-time 60 "$ORIGIN/v1/image-jobs/$JOB/result" \
  -H "Authorization: Bearer $KEY" -o /tmp/ewo-image-result.json
URL=$(field url /tmp/ewo-image-result.json)
if [ -n "$URL" ]; then
  P=${URL%%\?*}; OUT="ewo-image-$(date +%s).${P##*.}"   # or the path the user asked for
  curl -sS --max-time 120 "$URL" -o "$OUT" && echo "saved: $OUT"
else
  grep -o '"message": *"[^"]*"' /tmp/ewo-image-result.json | head -n 1   # no link: this prints why
fi
```

`field` reads one value from the API's compact JSON. If it prints nothing where a
value should be, open the JSON file and read it yourself.

Tell the user the saved path. If the status is `failed`, `cancelled` or
`expired`, read `error.message` in `/tmp/ewo-image-job.json`; failed images are
not charged. A temporary failure (the message says overloaded, timeout or retry)
is worth one more submit; if that fails too, tell the user and offer another model
from Step 2. If the loop gave up while the job was still running, poll the same
job id again later — a new submit is billed again.

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
