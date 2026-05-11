# FrontierIQ Skills

Agent skills for working with the [FrontierIQ](https://frontier-iq.vercel.app) AI Workflow API.

## Install

```bash
# Install all skills
npx skills add accenture-dx/frontieriq-skills

# Install a specific skill
npx skills add accenture-dx/frontieriq-skills --skill frontieriq-cost-model
```

## Skills

### `frontieriq-cost-model`

Build cost models for agentic and genAI projects using live model economics
data from the FrontierIQ AI Workflow API.

**Covers:**
- Querying API pricing, self-hosted GPU costs, and throughput data
- Projecting token budgets across agent loops
- Break-even analysis (self-hosted vs. managed API)
- Multi-model pipeline cost summation
- Agentic I/O ratio profiling

**Requires:** A FrontierIQ workflow API key — generate one on the
[FrontierIQ API Keys page](https://frontier-iq.vercel.app).

---

### `frontieriq-benchmarks`

Select and compare LLMs by task type using live benchmark data from FrontierIQ.

**Covers:**
- Full benchmark taxonomy: 11 benchmarks mapped to agentic use cases
  (reasoning, coding, bug-fixing, research, long-horizon, multi-agent, and more)
- Three selection lenses: Strongest / Cheapest / Fastest — what each means and
  when to use it
- Public endpoints (no auth) for frontier, cost, and speed leaders per benchmark
- Authenticated endpoints for per-model scorecard data and full context
- Model selection patterns: task → benchmark mapping, selection matrices,
  capability + cost decision reports, multi-step pipeline model assignment

## Contributing

Skills live under `skills/<skill-name>/SKILL.md`. To add a new skill:

1. Create a new directory under `skills/`
2. Add a `SKILL.md` with the standard frontmatter (`name`, `description`)
3. Open a PR

## License

MIT
