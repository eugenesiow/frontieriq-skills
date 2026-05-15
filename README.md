# FrontierIQ Skills

Agent skills for working with the [FrontierIQ](https://frontieriq.airefinery.accenture.com) AI Workflow API — live model economics, benchmark data, and cost modelling for agentic and genAI projects.

## Install

```bash
# Install all skills
npx skills add eugenesiow/frontieriq-skills

# Install a specific skill
npx skills add eugenesiow/frontieriq-skills --skill frontieriq-cost-model
npx skills add eugenesiow/frontieriq-skills --skill frontieriq-benchmarks
```

## Skills

| Skill | Description |
|-------|-------------|
| [`frontieriq-cost-model`](#frontieriq-cost-model) | Build cost models — API pricing, self-hosted GPU break-even, token budgeting |
| [`frontieriq-benchmarks`](#frontieriq-benchmarks) | Select models by task type using benchmark data and the Strongest / Cheapest / Fastest lenses |

---

### `frontieriq-cost-model`

Build cost models for agentic and genAI projects using live model economics data from the FrontierIQ AI Workflow API.

**Covers:** API pricing · self-hosted GPU costs · break-even analysis · token budget projection · multi-model pipeline costing · agentic I/O ratio profiling

**Auth required:** Yes — generate a workflow API key on the [FrontierIQ API Keys page](https://frontieriq.airefinery.accenture.com).

---

### `frontieriq-benchmarks`

Select and compare LLMs by task type using live benchmark data from FrontierIQ.

**Covers:** 11-benchmark taxonomy mapped to agentic use cases · Strongest / Cheapest / Fastest selection lenses · public frontier/cost/speed endpoints · per-model scorecard data · model selection matrices and decision reports

**Auth required:** Public endpoints need no auth; scorecard endpoints require a workflow API key.

---

## API key

Both skills call `https://frontieriq.airefinery.accenture.com`. Set your key as an environment variable:

```bash
export FRONTIERIQ_API_KEY=your_key_here
```

## Contributing

Skills live in `skills/<skill-name>/SKILL.md`. To contribute:

1. Fork this repo
2. Create `skills/<your-skill-name>/SKILL.md` with `name` and `description` frontmatter
3. Open a pull request

## License

[MIT](LICENSE)
