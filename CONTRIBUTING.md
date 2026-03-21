# Contributing

Thanks for taking the time to contribute. This repository is a field manual focused on operational deployability for tool-using LLM agents.

## What feedback is most valuable
- Missing failure modes that occur in production
- Readiness gates that are unrealistic in practice
- Control patterns that need stronger enforcement detail
- Metric definitions that are ambiguous or hard to operationalize
- Evaluation scenarios that do not reflect real attacker or operator behavior
- Tier 4 multi-agent constraints and verification patterns

## How to file an issue
Please include:
- Section or appendix reference (example: "Section 6.5" or "appendices/appendix-f-evaluation-scenarios.md")
- What is wrong or missing
- Why it matters operationally (impact, blast radius, auditability, recovery)
- A concrete suggestion if you have one

## How to submit a pull request
- Keep PRs narrow: one topic per PR.
- Reference the exact section or appendix being modified.
- If your change affects rollout gates, metrics, or audit requirements, say so explicitly.

## Style expectations
- Keep the tone operational and vendor-neutral.
- Avoid manifesto language.
- Prefer concrete requirements, detection signals, and recovery expectations.
- Use plain requirement language suitable for engineering teams.

## License for contributions
By submitting a contribution, you agree that it will be licensed under the license that applies to the directory you are modifying, as defined in the root `LICENSE` overview.
