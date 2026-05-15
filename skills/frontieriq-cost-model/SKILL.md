---
name: frontieriq-cost-model
description: >-
  Use the FrontierIQ AI Workflow API to build cost models for agentic and
  genAI projects. Covers querying model economics data (API pricing, self-hosted
  GPU costs, throughput, break-even analysis), projecting token budgets across
  agent loops, and comparing deployment options. Use whenever the user asks
  about LLM cost estimation, self-hosted vs API trade-offs, token budgeting,
  break-even analysis, or wants to pull live model pricing data from FrontierIQ.
---

# FrontierIQ Cost Modelling

Build cost models for agentic/genAI projects using live model economics data
from the FrontierIQ AI Workflow API.

**Base URL:** `https://frontieriq.airefinery.accenture.com`  
**Auth:** `Authorization: Bearer <FRONTIERIQ_API_KEY>`  
(Generate a key on the FrontierIQ API Keys page.)

---

## API Reference

### 1. List models — `GET /api/ai/v1/models`

Browse the model catalogue. Returns compact records suitable for selection.

| Param | Type | Notes |
|-------|------|-------|
| `q` | string | Free-text search |
| `organization` | string | Filter by org name (e.g. `meta`, `google`) |
| `benchmark` | string | Filter to models with results on this benchmark |
| `weightAvailability` | `open`\|`closed`\|`unknown` | Open = self-hostable |
| `limit` | int | 1–100, default 25 |

**Response shape:**
```json
{
  "models": [
    {
      "slug": "llama-3.3-70b-instruct",
      "displayName": "Llama 3.3 70B Instruct",
      "modelVersion": "llama-3.3-70b-instruct",
      "canonicalName": "Llama 3.3 70B Instruct",
      "organizations": ["meta"],
      "countries": ["US"],
      "weightAvailability": "open",
      "parameterCountEstimate": 70000000000,
      "activeParameterCountEstimate": null,
      "contextLengthTokens": 131072,
      "benchmarkCount": 8,
      "benchmarks": ["swe_bench_verified", "gpqa_diamond", "..."],
      "latestReleaseDate": "2024-12-06",
      "firstReleaseDate": "2024-12-06",
      "huggingfaceRepoUrl": "https://huggingface.co/meta-llama/..."
    }
  ],
  "count": 25,
  "totalAvailable": 312,
  "filteredAvailable": 312,
  "query": "",
  "organization": "all",
  "benchmark": "all",
  "weightAvailability": "all"
}
```

---

### 2. Get model detail — `GET /api/ai/v1/models/{slug}`

Returns benchmark scorecards and all observed provider pricing for one model.

**Response shape:**
```json
{
  "model": { /* compact model record, same fields as above */ },
  "benchmarkScorecards": [
    {
      "benchmarkSlug": "swe_bench_verified",
      "score": 0.493,
      "releaseDate": "2024-11-12"
    }
  ],
  "providerObservations": [
    {
      "providerSlug": "together",
      "providerDisplayName": "Together AI",
      "providerModelName": "meta-llama/Llama-3.3-70B-Instruct-Turbo",
      "creatorName": "Meta",
      "promptPerMillionTokensUsd": 0.6,
      "completionPerMillionTokensUsd": 0.6,
      "blendedPerMillionTokensUsd": 0.6,
      "medianOutputTokensPerSecond": 85.4,
      "medianTimeToFirstTokenSeconds": 0.32
    }
  ],
  "dashboardUrl": "/models/llama-3.3-70b-instruct"
}
```

Key fields for cost modelling:
- `blendedPerMillionTokensUsd` — blended price (assumes 3:1 input/output ratio) used as the primary cost signal
- `promptPerMillionTokensUsd` / `completionPerMillionTokensUsd` — split prices for precise modelling
- `medianOutputTokensPerSecond` — throughput signal for latency and capacity estimates

---

### 3. Get full cost context — `GET /api/ai/v1/context?model={slug}`

**The primary endpoint for cost modelling.** Returns a pre-computed, AI-ready
bundle: best API offer, best self-hosted fit, adjusted cost summary with
break-even and capacity numbers, benchmark summary, and data freshness.

**Response shape:**
```json
{
  "model": { /* compact model record */ },
  "costAnalysis": {
    "bestApiOffer": {
      "providerSlug": "together",
      "providerDisplayName": "Together AI",
      "providerModelName": "meta-llama/Llama-3.3-70B-Instruct-Turbo",
      "promptPerMillionTokensUsd": 0.6,
      "completionPerMillionTokensUsd": 0.6,
      "blendedPerMillionTokensUsd": 0.6,
      "throughputTokensPerSecond": 85.4
    },
    "bestSelfHostedOffer": {
      "providerSlug": "aws",
      "providerDisplayName": "AWS",
      "skuName": "p4d.24xlarge",
      "region": "us-east-1",
      "gpuFamily": "A100",
      "gpuCount": 8,
      "gpuMemoryGb": 40,
      "purchaseModel": "Reserved",
      "reservationTerm": "1yr",
      "precision": "fp16",
      "monthlyCostUsd": 14400,
      "hourlyRateUsd": 20.0,
      "observedThroughputTokensPerSecond": 85.4,
      "requiredVramGib": 131.8,
      "usableGpuMemoryGb": 288.0
    },
    "selfHostedProviderOptions": [ /* same shape, one per aws/gcp/azure */ ],
    "adjusted": {
      "assumptions": {
        "inputTokensPerOutputToken": 3,
        "utilizationPercent": 70,
        "batchConcurrency": 16,
        "batchingEfficiencyPercent": 60
      },
      "apiCostPerMillionOutputTokens": 2.4,
      "selfHostedCostPerMillionOutputTokens": 1.1,
      "breakEvenMillionOutputTokensPerMonth": 6000,
      "monthlyCapacityMillionOutputTokens": 13056,
      "observedThroughputTokensPerSecond": 85.4,
      "effectiveThroughputTokensPerSecond": 819.84,
      "calculationNotes": [
        "API cost per 1M output tokens = prompt price x input/output ratio + completion price.",
        "Effective self-hosted TPS = observed provider TPS x concurrency x batching efficiency.",
        "Monthly capacity = effective TPS x 3,600 x 720 x utilization."
      ]
    }
  },
  "providerObservations": [ /* same as /models/{slug} */ ],
  "benchmarkSummary": [
    {
      "benchmarkSlug": "swe_bench_verified",
      "score": 0.493,
      "releaseDate": "2024-11-12"
    }
  ],
  "dataFreshness": [
    {
      "slug": "provider-inference",
      "name": "Provider Inference Pricing",
      "lastUpdatedAt": "2025-05-05T12:00:00Z",
      "refreshedAt": "2025-05-05T14:00:00Z",
      "lastUpdatedLabel": "1 day ago"
    }
  ],
  "dashboardUrls": {
    "model": "/models/llama-3.3-70b-instruct",
    "costAnalysis": "/cost-analysis?model=llama-3.3-70b-instruct"
  }
}
```

---

## Cost Modelling Workflow

### Step 1 — Discover candidate models

```bash
# Find open-weight models (self-hostable)
curl -H "Authorization: Bearer $FRONTIERIQ_API_KEY" \
  "https://frontieriq.airefinery.accenture.com/api/ai/v1/models?weightAvailability=open&limit=50"

# Search by name
curl -H "Authorization: Bearer $FRONTIERIQ_API_KEY" \
  "https://frontieriq.airefinery.accenture.com/api/ai/v1/models?q=llama+70b"

# Filter by benchmark (only models benchmarked on SWE-bench)
curl -H "Authorization: Bearer $FRONTIERIQ_API_KEY" \
  "https://frontieriq.airefinery.accenture.com/api/ai/v1/models?benchmark=swe_bench_verified"
```

### Step 2 — Pull full cost context for shortlisted models

```bash
curl -H "Authorization: Bearer $FRONTIERIQ_API_KEY" \
  "https://frontieriq.airefinery.accenture.com/api/ai/v1/context?model=llama-3.3-70b-instruct"
```

### Step 3 — Apply your project's token budget to the numbers

The `adjusted` block uses these default assumptions:
- **3:1 input/output token ratio** — adjust manually if your agent loop is more input-heavy
- **70% utilization** — fraction of calendar time the GPU is actively inferring
- **16 batch concurrency × 60% batching efficiency** — parallelism headroom for batch workloads

To override the input/output ratio when projecting API costs:

```python
# Custom cost projection
prompt_price = best_api_offer["promptPerMillionTokensUsd"]
completion_price = best_api_offer["completionPerMillionTokensUsd"]

# Your actual ratio from profiling agent traces
input_tokens_per_output = 8  # e.g. tool results + system prompts are input-heavy

api_cost_per_million_output = (
    prompt_price * input_tokens_per_output + completion_price
)

monthly_output_tokens = estimated_monthly_calls * avg_output_tokens_per_call
monthly_cost = api_cost_per_million_output * monthly_output_tokens / 1_000_000
```

### Step 4 — Self-hosted break-even check

`monthly_output_tokens` is computed in the Step 3 projection snippet above.

```python
context = ctx["costAnalysis"]["adjusted"]  # ctx from Step 2 get_context() call

break_even = context["breakEvenMillionOutputTokensPerMonth"]
projected   = monthly_output_tokens / 1_000_000  # from Step 3 projection

if break_even and projected >= break_even:
    print(f"Self-hosting is cheaper above {break_even:.0f}M output tokens/month")
    print(f"Your projection ({projected:.0f}M) exceeds break-even — consider self-hosting")
else:
    print(f"API is cheaper at your projected volume")
```

---

## Agentic Workflow Cost Patterns

### Multi-step agent loop

Each agent turn typically has:
- A large **system prompt + tool definitions** (input, repeated every call)
- A growing **conversation history** (input, grows per turn)
- A short **model response / tool call** (output)

This pushes the real input:output ratio well above the default 3:1. Profile a
representative trace first:

```python
# Profile a sample trace
turns = [
  {"input_tokens": 4200, "output_tokens": 180},
  {"input_tokens": 6100, "output_tokens": 250},
  {"input_tokens": 8800, "output_tokens": 90},
]
total_input  = sum(t["input_tokens"] for t in turns)
total_output = sum(t["output_tokens"] for t in turns)
real_ratio   = total_input / total_output  # often 20–50x for agentic loops
```

Then apply `real_ratio` in place of `inputTokensPerOutputToken` when you
override the `adjusted` calculation.

### Batch vs. interactive throughput

`effectiveThroughputTokensPerSecond` already accounts for `batchConcurrency`
and `batchingEfficiencyPercent`. Use it for **offline/batch** workloads
(nightly report generation, document processing).

For **interactive** use cases (real-time agent responses), use
`observedThroughputTokensPerSecond` directly and model latency from
`medianTimeToFirstTokenSeconds` in the `providerObservations`.

### Multi-model pipeline

If your pipeline uses different models per step (e.g. a cheap model for
routing + a frontier model for synthesis), call `/context` for each model
and sum costs:

```python
steps = [
    {"model": "llama-3.2-3b-instruct", "avg_output_tokens": 50,  "calls_per_day": 100_000},
    {"model": "llama-3.3-70b-instruct", "avg_output_tokens": 400, "calls_per_day": 20_000},
]

total_monthly_cost = 0
for step in steps:
    ctx = get_context(step["model"])  # GET /api/ai/v1/context
    price = ctx["costAnalysis"]["bestApiOffer"]["blendedPerMillionTokensUsd"]
    monthly_tokens = step["avg_output_tokens"] * step["calls_per_day"] * 30
    total_monthly_cost += price * monthly_tokens / 1_000_000
```

---

## Key Fields Quick Reference

| Field | Where | Use for |
|-------|-------|---------|
| `blendedPerMillionTokensUsd` | `bestApiOffer` | Quick cost estimate at 3:1 I/O ratio |
| `promptPerMillionTokensUsd` | `bestApiOffer` | Precise cost when you know input volume |
| `completionPerMillionTokensUsd` | `bestApiOffer` | Precise cost when you know output volume |
| `apiCostPerMillionOutputTokens` | `adjusted` | Pre-computed blended output cost |
| `selfHostedCostPerMillionOutputTokens` | `adjusted` | Self-host cost at default assumptions |
| `breakEvenMillionOutputTokensPerMonth` | `adjusted` | Monthly volume above which self-hosting wins |
| `monthlyCapacityMillionOutputTokens` | `adjusted` | Max output a single SKU can serve per month |
| `effectiveThroughputTokensPerSecond` | `adjusted` | Batch capacity (concurrency + efficiency applied) |
| `observedThroughputTokensPerSecond` | `adjusted` | Raw provider TPS (use for interactive latency) |
| `medianTimeToFirstTokenSeconds` | `providerObservations` | TTFT for latency-sensitive flows |
| `monthlyCostUsd` | `bestSelfHostedOffer` | Fixed monthly SKU cost before amortising |
| `contextLengthTokens` | `model` | Max context window — affects per-call input ceiling |

---

## Error Responses

| Status | Error key | Meaning |
|--------|-----------|---------|
| 400 | `bad_request` | Missing required param (e.g. `model`) or invalid `weightAvailability` |
| 401 | `unauthorized` | Missing or invalid bearer token |
| 404 | `not_found` | Model slug not found in catalogue |
| 503 | `data_unavailable` | Upstream database unavailable — retry |

---

## Python Helper Snippet

```python
import os
import httpx

BASE_URL = "https://frontieriq.airefinery.accenture.com"
HEADERS  = {"Authorization": f"Bearer {os.environ['FRONTIERIQ_API_KEY']}"}

def list_models(**params) -> dict:
    r = httpx.get(f"{BASE_URL}/api/ai/v1/models", params=params, headers=HEADERS)
    r.raise_for_status()
    return r.json()

def get_model(slug: str) -> dict:
    r = httpx.get(f"{BASE_URL}/api/ai/v1/models/{slug}", headers=HEADERS)
    r.raise_for_status()
    return r.json()

def get_context(model_slug: str) -> dict:
    r = httpx.get(f"{BASE_URL}/api/ai/v1/context", params={"model": model_slug}, headers=HEADERS)
    r.raise_for_status()
    return r.json()

def project_monthly_cost(model_slug: str, monthly_calls: int,
                          avg_input_tokens: int, avg_output_tokens: int) -> dict:
    ctx = get_context(model_slug)
    offer = ctx["costAnalysis"]["bestApiOffer"]
    prompt_price     = offer["promptPerMillionTokensUsd"]
    completion_price = offer["completionPerMillionTokensUsd"]

    monthly_input  = monthly_calls * avg_input_tokens
    monthly_output = monthly_calls * avg_output_tokens
    monthly_cost   = (
        prompt_price     * monthly_input  / 1_000_000 +
        completion_price * monthly_output / 1_000_000
    )
    adjusted = ctx["costAnalysis"]["adjusted"]
    break_even = adjusted["breakEvenMillionOutputTokensPerMonth"]

    return {
        "model": ctx["model"]["displayName"],
        "provider": offer["providerDisplayName"],
        "monthly_cost_usd": round(monthly_cost, 2),
        "break_even_million_output_tokens": break_even,
        "recommendation": (
            "Consider self-hosting"
            if break_even and monthly_output / 1_000_000 >= break_even
            else "API is cost-effective at this volume"
        ),
    }
```
