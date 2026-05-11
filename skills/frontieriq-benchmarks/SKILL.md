---
name: frontieriq-benchmarks
description: >-
  Use the FrontierIQ benchmark APIs to select and compare LLMs by task type for
  agentic and genAI projects. Covers the full benchmark taxonomy (what each
  benchmark measures and which agentic use case it maps to), the three selection
  lenses (Strongest / Cheapest / Fastest), querying frontier leaders and
  leaderboard data, and using benchmark scores to make evidence-based model
  selection decisions. Use whenever the user asks which model to use for a
  specific task, wants to compare models on a benchmark, or needs to justify a
  model choice with benchmark data.
---

# FrontierIQ Benchmarks

Query live benchmark data from FrontierIQ to select and compare LLMs for
specific agentic tasks.

**Base URL:** `https://frontier-iq.vercel.app`

The benchmark endpoints come in two tiers:
- **Public** (`/api/frontier/`, `/api/costs/`, `/api/speeds/`, `/api/home-summary`) — no auth required, return single best-model summaries
- **AI Workflow API** (`/api/ai/v1/`) — requires `Authorization: Bearer <FRONTIERIQ_API_KEY>`, returns full scorecard data per model

---

## Benchmark Taxonomy

Every benchmark maps to one or more **agentic use cases**. Use this table to
pick the right benchmark slug for the task at hand.

| Slug | Name | Measures | Agentic use case |
|------|------|----------|-----------------|
| `gpqa_diamond` | GPQA Diamond | Graduate-level scientific reasoning across biology, physics, chemistry | Reasoning |
| `simpleqa_verified` | SimpleQA Verified | Short-form factual QA on verified knowledge prompts | World Knowledge |
| `deepresearchbench` | DeepResearchBench | Research-style synthesis, browsing, and knowledge work over long horizons | Research |
| `terminal_bench_2` | TerminalBench 2 | Real terminal tasks: CLI execution, debugging, environment interaction | Terminal Use |
| `aider_polyglot` | Aider Polyglot | Code editing and programming across many languages and repos | General Programming |
| `swe_bench_verified` | SWE-bench Verified | Resolving real GitHub issues with validated patches | Bug Fixing |
| `gso` | GSO | Code optimisation under engineering constraints | Code Optimisation |
| `apex_agents` | Apex Agents | Collaborative multi-step agent tasks with tool use and orchestration | Co-Work / Multi-Agent |
| `metr_time_horizons` | METR Time Horizons | Long-horizon task planning, persistence, and sequential execution | Long Horizon Agents |
| `video_mme` | Video-MME | Multimodal understanding over video inputs with temporal reasoning | Video Analysis |
| `vpct` | VPCT | Visual physics reasoning — inferring physical behaviour from scenes | Visual Physics |

---

## The Three Lenses

FrontierIQ surfaces models through three lenses. Choose the right one for your
project's priority.

| Lens | Picks | Algorithm |
|------|-------|-----------|
| **Strongest** | Highest raw score | Sort by score descending, take top |
| **Cheapest** | Best score-to-cost ratio | Models scoring ≥ score midpoint AND cost ≤ geometric mean of min/max cost; then cheapest among those |
| **Fastest** | Fastest total response time | Sort by `total_response_time_for_100_output_tokens_seconds` ascending |

Use **Strongest** when accuracy is the bottleneck (e.g. reasoning agents, research synthesis).  
Use **Cheapest** when you're processing high token volumes (e.g. batch pipelines, document processing).  
Use **Fastest** when latency drives UX (e.g. interactive agents, streaming responses).

---

## API Reference

### Public endpoints — no auth required

#### Frontier leader — `GET /api/frontier/{benchmark}`

Returns the single highest-scoring model on the given benchmark.

```bash
curl "https://frontier-iq.vercel.app/api/frontier/swe_bench_verified"
```

```json
{
  "topModel": "Claude Sonnet 4.5",
  "modelVersion": "claude-sonnet-4-5",
  "score": 0.727
}
```

---

#### Cheapest leader — `GET /api/costs/{benchmark}`

Returns the model with the best score-to-cost ratio: above the score midpoint
and below the geometric mean of the cost range.

```bash
curl "https://frontier-iq.vercel.app/api/costs/aider_polyglot"
```

```json
{
  "topModel": "Llama 3.3 70B Instruct",
  "modelVersion": "llama-3.3-70b-instruct",
  "cost": 0.6,
  "score": 0.583
}
```

`cost` is blended price in USD per million tokens (3:1 input/output ratio).

---

#### Fastest leader — `GET /api/speeds/{benchmark}`

Returns the model with the lowest total response time for 100 output tokens,
as measured across managed API providers.

```bash
curl "https://frontier-iq.vercel.app/api/speeds/gpqa_diamond"
```

```json
{
  "topModel": "Llama 3.1 8B Instruct",
  "modelVersion": "llama-3.1-8b-instruct",
  "responseTime": 1.24,
  "score": 0.318
}
```

`responseTime` is in seconds for 100 output tokens (lower is better).

---

#### Home summary — `GET /api/home-summary`

Returns all three lens leaders across every benchmark in one call. Good for
building a comparison dashboard or populating a model selection matrix.

```bash
curl "https://frontier-iq.vercel.app/api/home-summary"
```

```json
{
  "generatedAt": "2025-05-11T10:00:00.000Z",
  "stats": {
    "modelCount": 312,
    "providerCount": 18,
    "infrastructureSkuCount": 940
  },
  "leaders": {
    "Strongest": {
      "swe_bench_verified": "Claude Sonnet 4.5",
      "gpqa_diamond": "Gemini 2.5 Pro",
      "aider_polyglot": "Claude Sonnet 4.5"
    },
    "Cheapest": {
      "swe_bench_verified": "Llama 3.3 70B Instruct",
      "gpqa_diamond": "Gemini 2.0 Flash",
      "aider_polyglot": "Llama 3.3 70B Instruct"
    },
    "Fastest": {
      "swe_bench_verified": "Llama 3.1 70B Instruct",
      "gpqa_diamond": "Llama 3.1 8B Instruct",
      "aider_polyglot": "Llama 3.1 70B Instruct"
    }
  }
}
```

Cached for 5 minutes server-side. `generatedAt` tells you data freshness.

---

### AI Workflow API — requires Bearer token

#### Filter models by benchmark — `GET /api/ai/v1/models?benchmark={slug}`

Returns only models that have results on the given benchmark. Combine with
`weightAvailability=open` to find self-hostable options with coverage.

```bash
curl -H "Authorization: Bearer $FRONTIERIQ_API_KEY" \
  "https://frontier-iq.vercel.app/api/ai/v1/models?benchmark=swe_bench_verified&weightAvailability=open&limit=20"
```

Response includes compact model records — see the `frontieriq-cost-model` skill
for the full model shape.

---

#### Model benchmark scorecards — `GET /api/ai/v1/models/{slug}`

Returns all benchmark scorecards for a specific model alongside provider
pricing observations.

```bash
curl -H "Authorization: Bearer $FRONTIERIQ_API_KEY" \
  "https://frontier-iq.vercel.app/api/ai/v1/models/llama-3.3-70b-instruct"
```

```json
{
  "model": { "slug": "llama-3.3-70b-instruct", "displayName": "Llama 3.3 70B Instruct", "..." : "..." },
  "benchmarkScorecards": [
    { "benchmarkSlug": "swe_bench_verified", "score": 0.493, "releaseDate": "2024-11-12" },
    { "benchmarkSlug": "gpqa_diamond",       "score": 0.502, "releaseDate": "2024-11-12" },
    { "benchmarkSlug": "aider_polyglot",     "score": 0.583, "releaseDate": "2024-11-12" }
  ],
  "providerObservations": [ "..." ],
  "dashboardUrl": "/models/llama-3.3-70b-instruct"
}
```

`score` is a 0–1 fraction (multiply by 100 for a percentage).

---

#### Full context with benchmark summary — `GET /api/ai/v1/context?model={slug}`

Returns the top 5 benchmark scorecards combined with cost analysis. Use this
when you need to justify a model choice by both capability and cost in one
response.

```bash
curl -H "Authorization: Bearer $FRONTIERIQ_API_KEY" \
  "https://frontier-iq.vercel.app/api/ai/v1/context?model=llama-3.3-70b-instruct"
```

The `benchmarkSummary` field contains up to 5 scorecards with scores:

```json
{
  "benchmarkSummary": [
    { "benchmarkSlug": "swe_bench_verified", "score": 0.493, "releaseDate": "2024-11-12" },
    { "benchmarkSlug": "gpqa_diamond",       "score": 0.502, "releaseDate": "2024-11-12" }
  ]
}
```

---

## Model Selection Patterns

### Pattern 1 — Pick the best model for a specific task type

Map the user's task to a benchmark slug, then query the appropriate lens.

```python
import httpx

BASE_URL = "https://frontier-iq.vercel.app"

# Task → benchmark mapping
TASK_BENCHMARKS = {
    "reasoning":          "gpqa_diamond",
    "world_knowledge":    "simpleqa_verified",
    "research":           "deepresearchbench",
    "terminal":           "terminal_bench_2",
    "general_coding":     "aider_polyglot",
    "bug_fixing":         "swe_bench_verified",
    "code_optimisation":  "gso",
    "multi_agent":        "apex_agents",
    "long_horizon":       "metr_time_horizons",
    "video":              "video_mme",
    "visual_physics":     "vpct",
}

def recommend_model(task: str, priority: str = "strongest") -> dict:
    """priority: 'strongest' | 'cheapest' | 'fastest'"""
    benchmark = TASK_BENCHMARKS.get(task)
    if not benchmark:
        raise ValueError(f"Unknown task: {task}")

    endpoint_map = {
        "strongest": f"/api/frontier/{benchmark}",
        "cheapest":  f"/api/costs/{benchmark}",
        "fastest":   f"/api/speeds/{benchmark}",
    }
    r = httpx.get(BASE_URL + endpoint_map[priority])
    r.raise_for_status()
    data = r.json()

    return {
        "task": task,
        "benchmark": benchmark,
        "priority": priority,
        "recommended_model": data.get("topModel"),
        "model_version": data.get("modelVersion"),
        "score": data.get("score"),
        "cost_per_million_tokens": data.get("cost"),       # cheapest only
        "response_time_100_tokens": data.get("responseTime"),  # fastest only
    }

# Example
print(recommend_model("bug_fixing", priority="cheapest"))
```

---

### Pattern 2 — Build a model selection matrix

Compare multiple models across multiple task-relevant benchmarks.

```python
import httpx
import os

HEADERS = {"Authorization": f"Bearer {os.environ['FRONTIERIQ_API_KEY']}"}

def build_selection_matrix(model_slugs: list[str], benchmark_slugs: list[str]) -> list[dict]:
    rows = []
    for slug in model_slugs:
        r = httpx.get(
            f"{BASE_URL}/api/ai/v1/models/{slug}",
            headers=HEADERS,
        )
        if r.status_code == 404:
            continue
        r.raise_for_status()
        data = r.json()

        scores = {
            sc["benchmarkSlug"]: sc["score"]
            for sc in data["benchmarkScorecards"]
            if sc["benchmarkSlug"] in benchmark_slugs
        }
        rows.append({
            "model": data["model"]["displayName"],
            "slug": slug,
            **{b: scores.get(b) for b in benchmark_slugs},
        })
    return rows

# Compare coding-focused models on the three coding benchmarks
matrix = build_selection_matrix(
    model_slugs=["llama-3.3-70b-instruct", "qwen2.5-72b-instruct", "deepseek-v3"],
    benchmark_slugs=["aider_polyglot", "swe_bench_verified", "gso"],
)
for row in matrix:
    print(row)
```

---

### Pattern 3 — Justify model choice with capability + cost evidence

Pull both benchmark scores and cost context together for a decision report.

```python
def model_decision_report(model_slug: str, task: str) -> dict:
    benchmark = TASK_BENCHMARKS[task]
    r = httpx.get(
        f"{BASE_URL}/api/ai/v1/context",
        params={"model": model_slug},
        headers=HEADERS,
    )
    r.raise_for_status()
    ctx = r.json()

    task_score = next(
        (s["score"] for s in ctx["benchmarkSummary"] if s["benchmarkSlug"] == benchmark),
        None,
    )
    api_offer = ctx["costAnalysis"]["bestApiOffer"]
    adjusted  = ctx["costAnalysis"]["adjusted"]

    return {
        "model":           ctx["model"]["displayName"],
        "task":            task,
        "benchmark":       benchmark,
        "score":           task_score,
        "score_pct":       f"{task_score * 100:.1f}%" if task_score else None,
        "provider":        api_offer["providerDisplayName"] if api_offer else None,
        "blended_$/1m":    api_offer["blendedPerMillionTokensUsd"] if api_offer else None,
        "break_even_1m_output_tokens": adjusted["breakEvenMillionOutputTokensPerMonth"],
        "dashboard":       ctx["dashboardUrls"]["model"],
    }
```

---

### Pattern 4 — Scan all benchmarks for a multi-step agent pipeline

When your pipeline has distinct steps (routing → reasoning → coding → synthesis),
fetch the full home-summary and pick the right model per step.

```python
r = httpx.get(f"{BASE_URL}/api/home-summary")
summary = r.json()["leaders"]

pipeline = {
    "routing":    summary["Fastest"].get("simpleqa_verified"),  # latency-sensitive
    "reasoning":  summary["Strongest"].get("gpqa_diamond"),     # accuracy-critical
    "coding":     summary["Cheapest"].get("aider_polyglot"),    # high-volume
    "synthesis":  summary["Strongest"].get("deepresearchbench"),
}
print(pipeline)
```

---

## Score Interpretation Guide

Scores are benchmark-specific fractions (0–1). Do **not** compare raw scores
across benchmarks — they have different baselines and difficulty distributions.
Use them only for **within-benchmark** comparisons.

| Score (approx.) | Interpretation |
|-----------------|----------------|
| > 0.85 | Near-frontier or frontier capability |
| 0.60–0.85 | Strong; suitable for most production agentic tasks |
| 0.40–0.60 | Moderate; consider for cost-sensitive or less complex steps |
| < 0.40 | Weak on this task; avoid for this use case |

For `responseTime` (speeds endpoint): lower is better, measured in seconds for
100 output tokens. Values below 2s are generally suitable for interactive use.

---

## Quick Benchmark → Use Case Cheat Sheet

```
Need a reasoning/science agent?     → gpqa_diamond    (Strongest)
Need a factual Q&A / RAG layer?     → simpleqa_verified (Strongest or Cheapest)
Need a research synthesis agent?    → deepresearchbench (Strongest)
Need a terminal/DevOps agent?       → terminal_bench_2 (Strongest)
Need a code-generation assistant?   → aider_polyglot   (Cheapest or Strongest)
Need a bug-fixing agent?            → swe_bench_verified (Strongest or Cheapest)
Need a code optimisation step?      → gso             (Strongest)
Need a multi-agent orchestrator?    → apex_agents     (Strongest)
Need a long-running background job? → metr_time_horizons (Strongest)
Need a video understanding step?    → video_mme       (Strongest)
Need a visual scene analyser?       → vpct            (Strongest)
```
