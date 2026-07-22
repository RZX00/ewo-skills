---
name: ewo-video-generate
description: Generate short videos from a text prompt. Use when the user asks 生成视频/做个视频/视频素材 or create/generate a video clip. Free to start — new ewo accounts get a trial credit; then pay-as-you-go with your own ewo key (sk-ewo-…), no subscription.
---

# Video Generate — ewo open skill

A drop-in, multi-model media API. This skill calls ewo's public OpenAI-compatible
endpoint; you pay only for what you generate, from a prepaid ewo wallet. **Free to
start** — a new ewo account comes with a small trial credit, enough to try it out
before you ever recharge.

- Endpoint: `POST https://api.ewo.so/v1/videos/generations`
- Default `model`: `seedance-2-fast` (also: seedance-2, seedance-2-mini). Pick a model by setting the `model` field.

## Step 1 — resolve ORIGIN and KEY (first hit wins)

`ORIGIN=https://api.ewo.so`. For the key:

1. Environment: `KEY=$EWO_API_KEY`.
2. Config file: `~/.ewo/credentials` (or `%USERPROFILE%\.ewo\credentials` on
   Windows) — a line `api_key=sk-ewo-...` or a JSON `{ "api_key": "sk-ewo-..." }`.
3. If the ewo desktop app is running, its managed key also works — set
   `EWO_API_KEY` from it.

**If you find no key, STOP — do not call the API.** Tell the user, in their
language:

> This skill uses ewo's image/video API. It's free to start:
> 1. Sign up at https://api.ewo.so/sign-up — new accounts get a free trial credit.
> 2. Create an API key at https://api.ewo.so/keys (it starts with `sk-ewo-`).
> 3. Save it: `export EWO_API_KEY=sk-ewo-...` (or put `api_key=sk-ewo-...` in
>    `~/.ewo/credentials`), then ask me again.

## Step 2 — call the endpoint

Write the JSON body to a temp file — never inline user text into shell quoting.

```bash
cat > /tmp/ewo-req.json <<'EOF'
{
  "model": "seedance-2-fast",
  "prompt": "a neon koi swimming through a rainy Tokyo alley at night",
  "resolution": "720p",
  "duration": 5
}
EOF
curl -sS --max-time 180 "$ORIGIN/v1/videos/generations" \
  -H "Authorization: Bearer $KEY" -H "Content-Type: application/json" \
  -d @/tmp/ewo-req.json -o /tmp/ewo-resp.json
```

### Fields

- `prompt` (string, required)
- `resolution` (string, optional) (one of: 480p, 720p, 1080p, 4k)
- `duration` (integer, optional)
- `aspect_ratio` (string, optional) (one of: 21:9, 16:9, 4:3, 1:1, 3:4, 9:16, adaptive)

## Step 3 — poll, then download the video

Video is asynchronous. The create call returns a task with an `id` (and
`status` `running`/`accepted`). Poll every 5 seconds until `status` is
`succeeded` or `failed`:

```bash
TASK=$(jq -r '.id // .task_id' /tmp/ewo-resp.json)
while :; do
  curl -sS -H "Authorization: Bearer $KEY" \
    "$ORIGIN/v1/videos/generations/$TASK" -o /tmp/ewo-video.json
  S=$(jq -r '.status' /tmp/ewo-video.json)
  [ "$S" = "succeeded" ] && break
  [ "$S" = "failed" ] && { echo "video failed"; cat /tmp/ewo-video.json; exit 1; }
  sleep 5
done
URL=$(jq -r '.data[0].url // .url // empty' /tmp/ewo-video.json)
OUT="ewo-video-$(date +%s).mp4"
curl -sS "$URL" -o "$OUT" && echo "saved: $OUT"
```

A clip takes ~1-2 minutes. Tell the user the saved file path when done.

## Failures (JSON `{ error: { code, message } }`) — do NOT blind-retry 4xx

- `401` (missing / invalid / expired key) → the user has no working ewo key.
  Walk them through **Step 1** signup (sign up at https://api.ewo.so/sign-up, create a key
  at https://api.ewo.so/keys), then stop.
- `402` `HABITAT_INSUFFICIENT_BALANCE` → the ewo wallet is empty. Tell the user,
  in their language, to recharge at https://api.ewo.so/console/topup, then stop. Do **not**
  retry — `402` is terminal.
- `400` `invalid_request_error` → fix the request body against the Fields list.
- `429` → real rate limit; honor `Retry-After` and retry once.
- `503` → temporary ewo-side capacity issue; wait a few seconds, retry once.
