---
name: movoa-video
category: media-content
description: Unlimited AI video generator with sound: on paid plans the tenth take of a shot costs what the first did, and your first clip is free to make right now. Generate short AI video clips with synchronized audio via movoa.ai (MiniMax H3, Seedance 2.5, Veo 3.1 Fast, Seedance 2.0) using MOVOA_API_KEY.
license: MIT
---

# Movoa Video Skill

Unlimited AI video generator with sound: on paid plans the tenth take of a shot costs what the first did, and your first clip is free to make right now. Use this agent skill to create [text to video](https://movoa.ai/video/text-to-video) or [photo to video](https://movoa.ai/video/photo-to-video) clips with [native AI audio](https://movoa.ai/video/ai-video-with-audio) on [Movoa](https://movoa.ai/), billed from your `MOVOA_API_KEY` balance.

## When to Use This Skill

- "Make a 6 second video clip of a paper boat drifting at sunrise with gentle water sound"
- "Animate this photo into a 5 second clip with natural ambient sound" (requires a public https image URL)
- "Check how many Movoa credits I have remaining"

## Workflow

1. Check the credit balance first (free call).
2. Propose the exact run (model, duration, resolution, aspect ratio) and wait for user confirmation before submitting (runs spend credits at the prices on https://movoa.ai/pricing).
3. Submit the generation request once confirmed.
4. Poll every 10 seconds until `status` reaches `success` or `failed`, then return the hosted video URL.

All requests require `Authorization: Bearer $MOVOA_API_KEY` (create a key at https://movoa.ai/settings/apikeys).

### 1. Check Balance (Free)

```bash
curl -s -X POST https://movoa.ai/api/user/get-user-credits \
  -H "Authorization: Bearer $MOVOA_API_KEY"
```

### 2. Submit Video Generation

```bash
curl -s -X POST https://movoa.ai/api/ai/generate \
  -H "Authorization: Bearer $MOVOA_API_KEY" \
  -H "content-type: application/json" \
  -d '{"provider":"auto","mediaType":"video","model":"minimax-h3","scene":"text-to-video","prompt":"a paper boat drifting on a calm lake at sunrise, slow dolly in, soft water lapping","options":{"duration":6,"resolution":"480p","aspect_ratio":"16:9","sound":true}}'
```

- For photo to video: set `"scene":"image-to-video"` and include `"image_input":["https://.../photo.jpg"]` inside `options`.
- Models: `minimax-h3` (default; 4/5/6/8/10/15 s; 480p or 768p), `seedance-2-5` (5–30 s), `veo-3-1-fast` (8 s, 720p), `seedance-2`, `seedance-2-fast`, `seedance-2-mini`.

### 3. Poll Status

```bash
curl -s -X POST https://movoa.ai/api/ai/query \
  -H "Authorization: Bearer $MOVOA_API_KEY" \
  -H "content-type: application/json" \
  -d '{"taskId":"<taskId>"}'
```

## Error Handling

| Response | Meaning |
|---|---|
| HTTP 401 | Invalid or missing API key (`https://movoa.ai/settings/apikeys`) |
| `data.reason: "insufficient_credits"` | Account balance too low (`https://movoa.ai/pricing`) |
| `data.reason: "plan_required"` | Selected model or resolution requires a paid plan |
| `data.reason: "concurrency"` | Another run is already active; wait for it to finish |
| `code: -2` or `-3` | Content moderation refusal; adjust prompt or input image |
