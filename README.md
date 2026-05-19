<div align="center">

<img src="https://capzy.ai/capzy-logo.svg" alt="Capzy" width="220" />

# AWS WAF Captcha Solver

**Two modes: full token flow (aws-waf-token cookie) or image-only grid classifier.**

[![Solve cost](https://img.shields.io/badge/from-%240.001%20%2F%20solve-%23ff5d2a)](https://capzy.ai/pricing)
[![Speed](https://img.shields.io/badge/avg%20solve-~8s%20token%20flow%20/%20%3C3s%20classification-%2322c55e)](https://capzy.ai/products/aws-waf)
[![Uptime](https://img.shields.io/badge/uptime-99.9%25-%2322c55e)](https://capzy.ai/status)
[![License: MIT](https://img.shields.io/badge/license-MIT-%23ff5d2a)](LICENSE)

[Live Demo](https://capzy.ai/products/aws-waf/demo) ·
[Get Free $0.10 Credit](https://capzy.ai/auth/register) ·
[Dashboard](https://capzy.ai/dashboard) ·
[Full Docs](https://capzy.ai/docs) ·
[Pricing](https://capzy.ai/pricing)

</div>

---

## What this repo is

Copy-pasteable examples for solving **AWS WAF Captcha** through the
[Capzy](https://capzy.ai) HTTP API — no SDK required. Pure curl, Python,
and Node.js using the raw API. Easy to read, easy to port, easy to audit.

## What is AWS WAF Captcha?

AWS WAF Captcha is Amazon's bot detection on sites using AWS Web Application Firewall. Two integration modes: full token flow (Capzy returns the aws-waf-token cookie hands-off), or image-only classification (you keep your browser, Capzy just classifies the 9-tile grid).

## Why Capzy

- **From $0.001 per solve.** Flat pricing — no tiers, no retainer, no monthly minimum.
- **~8s token flow / <3s classification average solve.** Production-grade speed.
- **Drop-in compatible.** `createTask` / `getTaskResult` protocol. If your code already speaks the standard solver shape, swap the host to `https://api.capzy.ai`.
- **$0.10 in real credits on sign-up.** No card. 100 free test solves.

## Pricing

| Task type | When to use | Cost / solve |
|-----------|-------------|-------------:|
| `AntiAwsWafTaskProxyLess`             | Proxyless (Capzy supplies the IP) | **$0.001**   |
| `AntiAwsWafTask`                       | You supply the proxy              | **$0.001**   |

For consistency across the target site, use the proxy variant with the
**same proxy your session is already running through** — the solver
mints the token from that IP, so when you submit it back through the
same proxy everything looks consistent.

## 60-second quickstart

```bash
# 1. Sign up — gets you $0.10 in free credits (100 solves)
open https://capzy.ai/auth/register

# 2. Copy your API key from the dashboard
#    https://capzy.ai/dashboard/api-keys

# 3. Run any example
export CAPZY_KEY="capzy_..."
bash examples/curl/basic.sh
```

Minimal Python:

```python
import requests, time

KEY = "capzy_xxxxxxxxxxxxxxxxxxxxxxxx"

# 1) Create the task
created = requests.post("https://api.capzy.ai/createTask", json={
    "clientKey": KEY,
    "task": {
        "type": "AntiAwsWafTaskProxyLess",
        "websiteURL": "https://example.com/protected"
    },
}).json()
task_id = created["taskId"]

# 2) Poll until ready
while True:
    result = requests.post("https://api.capzy.ai/getTaskResult", json={
        "clientKey": KEY, "taskId": task_id,
    }).json()
    if result["status"] == "ready":
        break
    time.sleep(2)

print(result["solution"])
```

That's the whole protocol. The rest of this repo is just that, in every
language we could think of.

## Pick your language

| Language        | Example                                       |
|-----------------|-----------------------------------------------|
| **curl / bash** | [`examples/curl/basic.sh`](examples/curl/basic.sh)    |
| **Python**      | [`examples/python/basic.py`](examples/python/basic.py) |
| **Node.js**     | [`examples/nodejs/basic.js`](examples/nodejs/basic.js) |

See [`examples/README.md`](examples/README.md) for setup details.

## Request envelope

```json
{
  "clientKey": "capzy_xxxxxxxxxxxxxxxxxxxxxxxx",
  "task": {
    "type": "AntiAwsWafTaskProxyLess",
    "websiteURL": "https://example.com/protected"
  }
}
```

| Field | Type | Required | Notes |
|-------|------|:--------:|-------|
| `type` | `string` | yes | AntiAwsWafTaskProxyLess / AntiAwsWafTask (token flow), or AwsWafClassification (image-only) |
| `websiteURL` | `string` | no | Token flow only — the protected page URL |
| `images` | `string[]` | no | AwsWafClassification only — array of base64 PNG/JPEG tile images |
| `question` | `string` | no | AwsWafClassification only — target label, e.g. `aws:grid:chair`, `aws:grid:bag` |
| `proxyType` | `string` | no  | http | https | socks4 | socks5 (only for `AntiAwsWafTask`) |
| `proxyAddress` | `string` | no  | IP or hostname of your proxy (only for `AntiAwsWafTask`) |
| `proxyPort` | `integer` | no  | Port number of your proxy (only for `AntiAwsWafTask`) |
| `proxyLogin` | `string` | no  | Optional — omit if your proxy doesn't require auth (only for `AntiAwsWafTask`) |
| `proxyPassword` | `string` | no  | Optional — omit if your proxy doesn't require auth (only for `AntiAwsWafTask`) |

Full reference in [`docs/parameters.md`](docs/parameters.md).

## Response shape

When the task is ready (`status: "ready"`), `solution` contains:

| Field | Type | Notes |
|-------|------|-------|
| `cookie` | `string` | Token flow — the aws-waf-token cookie value |
| `objects` | `number[]` | Classification — 0-based indices of tiles that matched the target label |
| `scores` | `number[]` | Classification — per-tile confidence (parallel to input order) |

### How to use the result

Token flow: set the aws-waf-token cookie on your HTTP client. Classification: click the tile indices we return in your own browser session.

## Features

- Two task types: full token flow + image-only classification
- Image-only path is sub-3s with no browser, no proxy bandwidth
- Covers all standard AWS WAF grid labels

## FAQ

**Which mode should I use?** Already running your own browser (Selenium / Puppeteer / Playwright)? Use AwsWafClassification — cheaper and faster. Otherwise AntiAwsWafTask for the full handled flow.

**What images do I send to classification?** The 9 grid tiles as base64 PNG/JPEG, in the order they appear (top-left to bottom-right). We return 0-based indices that matched.

## What you'll need

- A Capzy API key — [sign up](https://capzy.ai/auth/register) (free, $0.10 credit).
- Network access to `https://api.capzy.ai`.

## Other captcha types

Capzy solves 25+ captcha types. Full catalog at
[capzy.ai/pricing](https://capzy.ai/pricing). Each type has its own
solver repo on [github.com/capzy-ai](https://github.com/capzy-ai).

## License

[MIT](LICENSE).

---

<div align="center">

**[Sign up for free credits →](https://capzy.ai/auth/register)**

Built by [Capzy](https://capzy.ai). Issues + PRs welcome.

</div>
