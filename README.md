# image2.5 API Gateway on APIMart — GPT Image 2.5 (gpt-image-2.5 / gptimage2.5) Flare & Sunburst

A working map of the **image2.5** / **gpt image2.5** (GPT Image 2.5) API gateway on APIMart: which model ID to call, how the flash-cheap `ext` route differs from the token-billed official route, and what the output actually looks like. Includes a 12-image prompt gallery, copy-paste cURL / Python / JavaScript calls, and the async task-polling loop.

**Name variants you can search for:** `image2.5` · `image 2.5` · `gpt image2.5` · `gpt image 2.5` · `gpt-image-2.5` · `gptimage2.5` · `gpt-image2.5` · `GPT Image 2.5` — all refer to the same OpenAI image family served here as `gpt-image-2.5-flare`, `gpt-image-2.5-sunburst` and `gpt-image-2.5-ext`.

**Attributed entry points:** [Open GPT Image 2.5 on APIMart](https://go.apimart.ai/k-6479cd) · [Current pricing](https://go.apimart.ai/k-07e826) · [Get an API key](https://go.apimart.ai/k-c6f7b6)

## Contents

- [Model routes and IDs](#model-routes-and-ids)
- [Observed pricing](#observed-pricing)
- [What the output looks like](#what-the-output-looks-like)
- [Quickstart](#quickstart)
- [Request and response reference](#request-and-response-reference)
- [What you can call today](#what-you-can-call-today)
- [FAQ](#faq)
- [Attributed links](#attributed-links-how-this-repository-is-measured)
- [Repository map](#repository-map)

## Why an image2.5 gateway instead of a direct vendor call

An **AI API gateway** keeps one base URL, one key and one request shape in front of several image models. For teams
shipping image features, the practical wins are boring but real:

1. **One credential.** A single `Authorization: Bearer` header reaches image2.5 plus the rest of the catalog, so keys rotate in one place.
2. **Flat, predictable unit cost.** The relayed `gpt-image-2.5-ext` route is billed per delivered image, so a content
   pipeline can estimate cost before it runs. Token-billed routes cannot do that without measuring output size first.
3. **Async by default.** `POST /v1/images/generations` returns a `task_id`; the poll URL is part of the response, which
   makes queueing and retry logic straightforward.
4. **Model fallback.** If one variant is saturated, switch `version` (or the whole model ID) without touching the client.

This repository documents route selection, price expectations and the request/response shapes that were actually observed.

## Model routes and IDs

| Route | `model` value | Selector | Billing style | Best for |
| --- | --- | --- | --- | --- |
| Official (token) | `gpt-image-2.5-flare` | n/a | token usage, `quality` low → max | everyday generation, batch drafts |
| Official (token) | `gpt-image-2.5-sunburst` | n/a | token usage, `quality` low → max | editing precision, production assets |
| Relayed (per image) | `gpt-image-2.5-ext` | `version: "flare"` | per delivered image (`n` ≤ 4) | high-volume generation at a flat price |
| Relayed (per image) | `gpt-image-2.5-ext` | `version: "sunburst"` | per delivered image (`n` ≤ 4) | edits and reference-driven work at a flat price |

Both relayed variants accept `resolution` `1K` / `2K` / `4K`, ten aspect ratios plus `auto`, and up to 16 reference images in `image_urls`. The official route adds exact pixel dimensions and the `low / medium / high / xhigh / max` quality ladder.

- Family: **GPT Image 2.5** — the OpenAI image generation and editing series served through APIMart
- Model IDs: `gpt-image-2.5-flare`, `gpt-image-2.5-sunburst` (official route); `gpt-image-2.5-ext` with `version: flare|sunburst` (per-image relay route)
- Base URL: `https://api.apimart.ai/v1` — OpenAI-compatible `POST /v1/images/generations`
- Async tasks: submit, then poll `GET /v1/tasks/{task_id}` until `status: completed`
- Output tiers: `1K`, `2K`, `4K`; up to 16 reference images for image-to-image; `n` ≤ 4
- Observed 1K price on the relayed route: **$0.0085 per delivered image** (checked 2026-09-16)

## Observed pricing

| version | 1K | 2K | 4K | billing unit |
| --- | --- | --- | --- | --- |
| `flare` | $0.0085 | $0.014 | $0.021 | per delivered image |
| `sunburst` | $0.0085 | $0.014 | check live pricing | per delivered image |

Per-image billing on the relayed route is charged for delivered images, and the task response reports the exact amount in `cost` / `credits_cost`, so the table above can be re-verified after a single paid call. The official `gpt-image-2.5-flare` / `gpt-image-2.5-sunburst` route is token-billed with a `low → medium → high → xhigh → max` quality ladder, which is why this repository keeps both the flat per-image expectation and the token-billed option side by side. Snapshot date: 2026-09-16.

## What the output looks like

Every render below came from a single `POST /v1/images/generations` call on the relayed route, at the aspect ratio shown.
| Output | Recipe | Use case | Version | Ratio | Prompt |
| --- | --- | --- | --- | --- | --- |
| <img src="assets/02-rainy-tokyo-alley.jpg" width="220" alt="Cinematic night street generated with GPT Image 2.5"> | Cinematic night street | Cinematic still | `flare` | 16:9 | `Rain-slicked Tokyo alley at night, neon sign reflections on wet asphalt, a lone cyclist with an umbrella, cinematic 35mm film still, shallow depth of field` |
| <img src="assets/05-ivory-trench-portrait.jpg" width="220" alt="Fashion editorial portrait generated with GPT Image 2.5"> | Fashion editorial portrait | Fashion | `flare` | 3:4 | `Fashion editorial portrait of a model wearing an oversized ivory trench coat, seamless light grey studio backdrop, crisp high key lighting, medium format detail` |
| <img src="assets/04-gradient-paper-plane-icon.jpg" width="220" alt="App icon / 3D asset generated with GPT Image 2.5"> | App icon / 3D asset | UI asset | `flare` | 1:1 | `Glossy 3D app icon of a paper plane folded from a coral to violet gradient material, floating on a soft neutral grey backdrop, subtle contact shadow, centered composition` |
| <img src="assets/06-cliffside-villa-golden-hour.jpg" width="220" alt="Architectural render generated with GPT Image 2.5"> | Architectural render | Architecture | `flare` | 16:9 | `Modern cliffside villa at golden hour, infinity pool reflecting the sky, warm interior lights, architectural photography, 24mm perspective, ultra sharp detail` |
| <img src="assets/08-api-latency-dashboard-panel.jpg" width="220" alt="Flat vector dashboard panel generated with GPT Image 2.5"> | Flat vector dashboard panel | Design / infographic | `sunburst` | 16:9 | `Flat vector dashboard panel with three gauge dials, abstract latency curves and clean geometric cards, muted blue and sand palette, generous white space, crisp vector edges` |
| <img src="assets/07-isometric-smart-home.jpg" width="220" alt="Isometric 3D cutaway generated with GPT Image 2.5"> | Isometric 3D cutaway | 3D illustration | `sunburst` | 1:1 | `Isometric cutaway of a compact smart home control room, tiny mid century furniture, pastel palette, clay render finish, clean soft shadows, 3D illustration` |
| <img src="assets/12-paper-cut-mountain-lake.jpg" width="220" alt="Layered paper-cut illustration generated with GPT Image 2.5"> | Layered paper-cut illustration | Editorial illustration | `sunburst` | 4:3 | `Layered paper cut illustration of a mountain lake sunrise, five depth layers, soft pastel palette, subtle drop shadows, art print composition` |
| <img src="assets/10-amber-serum-packshot.jpg" width="220" alt="Cosmetics packshot generated with GPT Image 2.5"> | Cosmetics packshot | Product / cosmetics | `sunburst` | 1:1 | `Cosmetics packshot of a frosted glass bottle with amber serum, water droplets on the surface, seamless pale pink backdrop, high detail commercial product photography` |
| <img src="assets/01-product-hero-espresso-cup.jpg" width="220" alt="Studio product hero generated with GPT Image 2.5"> | Studio product hero | Product / e-commerce | `flare` | 1:1 | `Studio product hero of a matte ceramic espresso cup on a warm stone pedestal, soft window light, 85mm lens look, minimal beige backdrop, crisp shadow` |
| <img src="assets/11-floating-ruin-keyart.jpg" width="220" alt="Game key art generated with GPT Image 2.5"> | Game key art | Game concept art | `sunburst` | 16:9 | `Fantasy game key art, an armored knight standing on a floating stone ruin above a sea of clouds, dramatic backlight, painterly detail, wide cinematic composition` |
| <img src="assets/03-ramen-flatlay.jpg" width="220" alt="Menu food photography generated with GPT Image 2.5"> | Menu food photography | Food photography | `flare` | 4:3 | `Overhead flat lay of a spicy ramen bowl with a soft boiled egg, chopsticks resting on the rim, dark slate table, natural side light, editorial food photography` |
| <img src="assets/09-monarch-wing-macro.jpg" width="220" alt="Macro nature detail generated with GPT Image 2.5"> | Macro nature detail | Nature macro | `sunburst` | 3:2 | `Macro photograph of a dew covered monarch butterfly wing, iridescent orange scales, deep black background, focus stacked detail, studio lighting` |

Every recipe ships with the exact JSON body in [`examples/`](examples).

## Quickstart

The relayed route is asynchronous: submit, then poll the task ID.

```bash
# text to image on the per-image route
IDEMPOTENCY_KEY="$(uuidgen)"
curl --request POST \
  --url https://api.apimart.ai/v1/images/generations \
  --header "Authorization: Bearer $APIMART_API_KEY" \
  --header 'Content-Type: application/json' \
  --header 'X-APIMart-Response-Version: 2026-07-27' \
  --header "Idempotency-Key: $IDEMPOTENCY_KEY" \
  --data '{
    "model": "gpt-image-2.5-ext",
    "version": "flare",
    "prompt": "A cozy reading nook beside a window on a rainy day, warm table lamp, cinematic lighting",
    "size": "1:1",
    "resolution": "1K",
    "n": 1
  }'
```

```python
import os, time, uuid, requests

BASE = "https://api.apimart.ai/v1"
HEADERS = {
    "Authorization": f"Bearer {os.environ['APIMART_API_KEY']}",
    "Content-Type": "application/json",
    "X-APIMart-Response-Version": "2026-07-27",
    "Idempotency-Key": str(uuid.uuid4()),   # reuse on retry, not on a new image
}

def generate(prompt: str, version: str = "flare", resolution: str = "1K", size: str = "1:1") -> str:
    r = requests.post(f"{BASE}/images/generations", headers=HEADERS, timeout=60, json={
        "model": "gpt-image-2.5-ext", "version": version, "prompt": prompt,
        "size": size, "resolution": resolution, "n": 1,
    })
    r.raise_for_status()
    task_id = r.json()["data"]["id"]
    while True:
        t = requests.get(f"{BASE}/tasks/{task_id}", headers=HEADERS, timeout=60).json()["data"]
        if t["status"] in ("completed", "failed"):
            return t
        time.sleep(5)
```

```javascript
const headers = {
  Authorization: `Bearer ${process.env.APIMART_API_KEY}`,
  "Content-Type": "application/json",
  "X-APIMart-Response-Version": "2026-07-27",
  "Idempotency-Key": crypto.randomUUID(),
};
const res = await fetch("https://api.apimart.ai/v1/images/generations", {
  method: "POST",
  headers,
  body: JSON.stringify({
    model: "gpt-image-2.5-ext", version: "flare", prompt: "A sky garden at dawn, architectural photography",
    size: "16:9", resolution: "1K", n: 1,
  }),
});
const { data } = await res.json();          // data.id === task id
// poll GET https://api.apimart.ai/v1/tasks/${data.id} until data.status === "completed"
```

Official (token-billed) route, for comparison — same path, no `version`, quality ladder instead:

```bash
curl --request POST --url https://api.apimart.ai/v1/images/generations \
  --header "Authorization: Bearer $APIMART_API_KEY" --header 'Content-Type: application/json' \
  --data '{"model":"gpt-image-2.5-sunburst","prompt":"Preserve the product label, replace the background with soft off-white, add a natural cast shadow","size":"1:1","resolution":"1k","quality":"high","n":1}'
```

Full field reference: [official route docs](https://docs.apimart.ai/en/api-reference/images/gpt-image-2.5/generation) and [ext route docs](https://docs.apimart.ai/en/api-reference/images/gpt-image-2.5-ext/generation). Get a key at [apimart.ai/keys](https://go.apimart.ai/k-c6f7b6).

## Request and response reference (ext route)

| Field | Type | Default | Notes |
| --- | --- | --- | --- |
| `model` | string | required | `gpt-image-2.5-ext` |
| `version` | string | `flare` | `flare` or `sunburst` |
| `prompt` | string | required | must not be empty after trimming |
| `resolution` | string | `1K` | `1K`, `2K`, `4K` |
| `size` | string | `auto` | `auto` or 1:1, 16:9, 9:16, 4:3, 3:4, 3:2, 2:3, 5:4, 4:5, 21:9 |
| `n` | integer | `1` | 1–4 per request on the relayed route |
| `image_urls` | string[] | — | up to 16 references, URL or data URL, no extra charge |

Submission returns `202` with `data.id` (the task ID) and `data.poll_url`; `GET /v1/tasks/{task_id}` then reports
`status` (`pending` → `processing` → `completed` / `failed`), `progress`, `cost`, `credits_cost` and, when finished,
`result.images[].url` with an `expires_at` timestamp. Download outputs before that timestamp — the URLs are temporary.

## What you can call today

| Capability | How it is reached |
| --- | --- |
| Text to image | `model: gpt-image-2.5-ext`, `version: flare` or `sunburst` |
| Image to image / edit | same call plus `image_urls: [...]` (up to 16 references) |
| Batch of up to 4 | `n: 4` on the relayed route |
| Resolution tiers | `resolution: 1K \| 2K \| 4K` |
| Aspect ratio | ten ratios plus `auto` |
| Progress reporting | poll `GET /v1/tasks/{task_id}` for `progress` and `status` |



## FAQ

**What is the image2.5 API model ID?**

The official route exposes `gpt-image-2.5-flare` and `gpt-image-2.5-sunburst`. The relayed per-image route uses `gpt-image-2.5-ext` and selects the variant with `version: "flare"` or `version: "sunburst"`.

**How much does one image2.5 image cost on the cheap route?**

Observed 1K output on `gpt-image-2.5-ext` billed **$0.0085 per delivered image** (checked 2026-09-16). 2K and 4K steps are listed on the [pricing page](https://go.apimart.ai/k-07e826); re-check before a large run.

**Is the image2.5 API OpenAI-compatible?**

The endpoint path and JSON body follow the OpenAI images shape, but this is an APIMart-hosted gateway: the base URL is `https://api.apimart.ai/v1`, `n` is capped at 4 on the relayed route, and long jobs return a task ID instead of an inline image.

**Do reference images cost extra?**

On the relayed route, reference images are documented as incurring no additional charge; billing tracks version, resolution and the number of images actually delivered.

**How do I poll a task?**

Use the `poll_url` returned on submission, e.g. `GET /v1/tasks/task_01EXAMPLE`. A completed task carries `result.images[].url` plus `expires_at`, so download results before the window closes.

## Related searches

- `image2.5 api`
- `image 2.5 api`
- `image2.5 api gateway`
- `image2-5 api`
- `gpt-image-2.5 api`
- `image2.5 api pricing`
- `ai api relay`
- `ai api gateway`
- `ai api aggregator`
- `apimart image2.5`
- `image2.5 api documentation`
- `openai compatible image api`
- `image2 5`
- `image2 5 api`
- `image 2 5 api`
- `apimart`
- `api relay`

## Attributed links (how this repository is measured)

Every outbound link in this repository points at APIMart through a short link, so visits coming from this page are attributed instead of arriving as anonymous traffic.

| Purpose | Attributed link | Target |
| --- | --- | --- |
| Open GPT Image 2.5 on APIMart | <https://go.apimart.ai/k-6479cd> | `apimart.ai/model/gpt-image-2-5` |
| Current APIMart pricing | <https://go.apimart.ai/k-07e826> | `apimart.ai/pricing` |
| Get an API key on APIMart | <https://go.apimart.ai/k-c6f7b6> | `apimart.ai/keys` |

- [ ] Attribution target: the three `go.apimart.ai` short links above, all minted through the promo link API (302 with `utm_source=kol_sponsor&utm_medium=sponsor&sclid=...`). The endpoint docs on `docs.apimart.ai` are referenced as plain links: the link service only accepts the `apimart.ai` main domain, so no attributed short link exists for them.
- [ ] Re-check the price on the pricing page before a production run: promotional routing can change.

## Disclosure

APIMart is the service described in this repository; this page is published to document it, not to claim official status. The `ext` route is a third-party relay endpoint billed per delivered image, while the `gpt-image-2.5-flare` / `gpt-image-2.5-sunburst` models are the token-billed route. Model names, prices and limits belong to their respective owners, and everything here is observation-dated (2026-09-16). Verify with a single paid request before scaling volume.


## Repository map

```text
image2.5-api-gateway-apimart/
  PROMPTS.md           every recipe with its output
  README.md            overview, pricing, quickstart and FAQ
  examples/
    curl.sh            submit + poll with curl
    python_generate.py end-to-end Python client
    javascript.mjs     Node 18+ equivalent
  tools/check_links.py attribution + prompt-data validator
  .github/workflows/validate.yml  CI for the validator
  assets/              example renders (JPEG, resized for the README)
  LICENSE              MIT
```

## License

MIT — see [LICENSE](LICENSE). Model names and vendor documentation remain the property of their owners.
