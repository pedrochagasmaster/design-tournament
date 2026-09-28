# Design Tournament

An agent skill for running an end-to-end UI design tournament: divergent candidate generation, isolated independent review, evidence-based synthesis, and a second candidate round built from settled decisions.

## What it does

The workflow separates five roles:

1. Orchestrator
2. Candidate generator
3. Two or more independent judges
4. Synthesizer
5. Next-round generator

The core constraint is reviewer isolation: judges see the same immutable artifact and rubric, but not each other's conclusions. Their reports are synthesized against product truth rather than averaged as votes.

## Skill

The complete reusable skill is in [`SKILL.md`](./SKILL.md).

It supports both separate-session execution for strongest context isolation and a single orchestrator session using fresh isolated subagents.

## Origin

This workflow was reconstructed from a two-round UI design exploration used on Taurifer, then generalized into a reusable product-design skill.
