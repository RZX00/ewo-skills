---
name: ewo-image-generate
description: Generate or edit images. Use whenever the user asks to draw, create, or modify any picture, illustration, logo, poster, icon, or photo — 画图/画一张/生成图片/改图/做个logo/海报/插画/头像. Free to start — new ewo accounts get a trial credit; then pay-as-you-go with your own ewo key (sk-ewo-…), no subscription.
---

# Image Generate & Edit — ewo open skill

A drop-in, multi-model media API. This skill calls ewo's public OpenAI-compatible
endpoint; you pay only for what you generate, from a prepaid ewo wallet. **Free to
start** — a new ewo account comes with a small trial credit, enough to try it out
before you ever recharge.

- Endpoint: `POST https://api.ewo.so/v1/images/generations`
- Default `model`: `gpt-image-2`. Pick a model by setting the `model` field.

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
  "model": "gpt-image-2",
  "prompt": "a fluffy orange cat, watercolor style",
  "size": "1024x1024"
}
EOF
curl -sS --max-time 180 "$ORIGIN/v1/images/generations" \
  -H "Authorization: Bearer $KEY" -H "Content-Type: application/json" \
  -d @/tmp/ewo-req.json -o /tmp/ewo-resp.json
```

### Fields

- `prompt` (string, required) — The image prompt. When editing (image_url provided), describe the desired edit.
- `model` (string, optional) (one of: gpt-image-2) — Optional image model. Defaults to `gpt-image-2`.
- `image_url` (string, optional) — Optional. URL or data URL of a source image to edit; when provided the invocation runs in edit mode against an edit-capable model.
- `mask_url` (string, optional) — Optional mask URL or data URL. Only valid together with image_url.
- `size` (string, optional) (one of: 1024x1024, 1024x1536, 1536x1024, auto)
- `quality` (string, optional) (one of: standard, medium, high, auto) — Requested quality tier (OpenAI-image models only). Non-OpenAI image models (Nano Banana, Doubao Seedream) ignore this and render at their native quality; it is dropped before upstream so it never causes a 4xx.
- `background` (string, optional) (one of: opaque, transparent, auto)
- `output_format` (string, optional) (one of: png, jpeg, webp)

## Step 3 — save the image

The synchronous response is the OpenAI images shape: `.data[0]` holds either
`b64_json` (a multi-megabyte base64 payload — **NEVER print it into the chat, it
destroys your context**) or `url` (a short-lived link — download it immediately).

```bash
B64=$(jq -r '.data[0].b64_json // empty' /tmp/ewo-resp.json)
URL=$(jq -r '.data[0].url // empty' /tmp/ewo-resp.json)
OUT="ewo-image-$(date +%s).png"   # or the path the user asked for
if [ -n "$B64" ]; then printf %s "$B64" | base64 -d > "$OUT"; else curl -sS "$URL" -o "$OUT"; fi
echo "saved: $OUT"
```

Finish by telling the user the saved file path.

### Editing an existing image

To edit instead of generate, POST the same body plus `image` (a `data:` URL of
the source picture) to `$ORIGIN/v1/images/edits`. `gpt-image-2` supports editing.

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
