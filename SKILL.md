---
name: design-tournament
description: "Run an end-to-end UI design tournament: generate genuinely different candidates in a shared review harness, have multiple isolated judges evaluate them independently, synthesize a normative design spec, and use that spec to generate the next round. Use for major UI exploration or iterative candidate rounds where judge contamination would bias the result."
disable-model-invocation: true
---

# Design Tournament

A divergence → independent review → synthesis → renewed divergence loop for product UI.

The point is not to generate many mockups. The point is to make competing product-level design hypotheses comparable, judge them without cross-contaminating reviewer roles, convert the strongest evidence into settled rules, then reopen only the design space that remains genuinely undecided.

## Roles

- **Orchestrator:** owns the workflow, artifact references, gates, and handoffs. It does not pre-judge candidates.
- **Generator:** builds the candidate set in one shared comparison harness.
- **Judges:** two or more independent reviewers. Each gets the same artifact bundle and rubric, and none sees another judge's report before submitting its own.
- **Synthesizer:** receives all completed judge reports together and resolves disagreements into a normative spec.
- **Next-round generator:** starts from the normative spec and source of truth, not from the raw debate.

For consequential design work, default to **2 independent judges**. Add more only when the product decision is unusually broad or the user explicitly asks.

## Non-negotiable isolation rule

A model that generated a candidate must not be the sole judge of that candidate.

### Separate-session mode

This is the strongest isolation mode and the one the recovered Taurifer workflow used:

1. Run candidate generation in its own session.
2. Start Judge 1 from a fresh session with the common judge brief.
3. Start Judge 2 from another fresh session with the identical brief.
4. Do not give either judge the other's output.
5. Start a synthesis session only after every judge has finished.
6. Save the synthesis as a durable spec/decision log.
7. Start the next generator in a fresh session with that spec.

### Single-session subagent mode

The full workflow may stay under one orchestrator if subagents provide isolated contexts:

1. The orchestrator prepares immutable artifact references and one canonical judge brief.
2. Spawn judges in parallel from that brief before exposing any judge output to siblings.
3. Each judge writes to a separate report artifact.
4. Wait for all judges to finish. Do not relay partial verdicts between them.
5. Spawn a fresh synthesizer with all reports at once.
6. Spawn the next-round generator with only the consolidated spec plus product/source artifacts.
7. Do not pass raw judge chains, rankings, or identities to the next generator unless needed to resolve a documented ambiguity.

If the subagent system inherits conversational context, explicitly tell each judge to ignore prior opinions and inspect the artifacts independently. Prefer truly fresh subagents when available.

## Phase 0 — Freeze the comparison contract

Before generating anything, inspect the current product and define the common ground all candidates must share.

Record:

- the product/user/job being designed for;
- authoritative product, design, domain, brand, accessibility, and implementation documents;
- current production UI and behavior to compare against;
- target devices, widths, languages, themes, input modes, and important states;
- real domain data and calculations that must remain semantically identical across candidates;
- implementation boundaries: what prototype code may change and what production code must remain untouched;
- the acceptance matrix every candidate must pass.

Do not let candidates compete using different data, easier states, shorter copy, or different feature coverage. That makes the comparison invalid.

For Taurifer, re-inspect the repository every run. The observed tournament used `PRODUCT.md`, `DESIGN.md`, `CONTEXT.md`, `docs/brand-guide.md`, the current shell, UI captures, and the real progression engine as authority. Verify those paths still exist and use newer governing documents when the repo has changed.

Taurifer defaults from the recovered rounds were mobile-first, PT-BR first, 360/390/430px, then EN, light and dark. Treat these as defaults to verify, not timeless facts.

## Phase 1 — Build the candidate round

Default to **3 genuinely different directions** when starting from an open design question.

Every direction must have:
- a one-line thesis explaining the direction;
- a named axis of divergence such as hierarchy, interaction model, information density, evidence presentation, navigation, or spatial strategy;
- the exact same task/state/screen coverage as its siblings;
- production-shaped copy and realistic data;
- working interactions needed to judge the concept;
- the project's real tokens and visual language unless breaking one is an explicit design hypothesis.

Three cosmetic variations of one composition do not count. A direction should be different enough that choosing it would produce a meaningfully different product experience.

Build all directions inside one **shared review harness**. The harness is part of the experiment: one control surface must let a reviewer switch candidate, screen/state, language, theme, and target width without changing the underlying data.

For the recovered Taurifer round, the common surface covered Today, focus workout, Why this weight, rest timer, session summary with and without PRs, Progress overview, exercise chart, History list, History session, Program, and bottom navigation. Rebuild the screen/state inventory from the current app before a future run rather than assuming this list is complete forever.

Use real product logic when the UI depends on computed output. A prototype that invents recommendations, outcomes, or domain states may look convincing while testing the wrong product.

Keep exploratory UI isolated from production code. Prefer a worktree and a review-only route/directory. Do not modify the live shell merely to make candidate review easier.

## Generator prompt structure

Use this structure rather than a free-form design request:

1. **Goal:** what product decision this candidate round is meant to resolve.
2. **Source of truth:** exact docs, code, captures, and domain logic to inspect before designing.
3. **Reference user and environment:** who is using it, where, and under what physical/cognitive constraints.
4. **Invariant contract:** screens/states/data/logic/accessibility/languages/themes/widths shared by every direction.
5. **Divergence instruction:** require genuinely different design hypotheses, not variants of one idea.
6. **Design freedom:** explicitly name what may change: copy, hierarchy, layout, components, interaction model, motion, density, etc.
7. **Implementation boundary:** name production files/surfaces that must not be touched.
8. **Delivery:** shared review harness, local/public preview, branch/PR, one-line thesis per direction.
9. **Verification:** acceptance matrix, responsive checks, localization, theme, interaction, console/network errors, semantic consistency.
10. **Quality bar:** real copy, real calculations, no unfinished placeholders.

### First round

With no prior normative spec, instruct the generator to explore the design space broadly while preserving the invariant contract. The generator may choose the direction theses itself.

### Later rounds

After synthesis, do not merely ask for 'improvements'. Use the consolidated spec as a boundary:

- first implement the synthesized direction exactly enough to serve as a control/baseline candidate;
- treat correctness fixes, domain semantics, accessibility floors, and explicitly settled system rules as fixed;
- then generate fresh candidates that obey those fixed rules while diverging on the remaining design decisions;
- state that nothing from the earlier candidates is sacred beyond the settled rules;
- keep the same harness so old and new directions can be compared on identical screens and data.

This is how the recovered second Taurifer round was framed: build the synthesized Direction D exactly, then create new candidates based on D's settled correctness/system rules while reopening copy, layout, hierarchy, and components.

## Phase 2 — Publish a judgeable artifact

Do not ask judges to reason from code alone if the design is runnable.
Give every judge the same bundle:

- live preview URL or reproducible local serving instructions;
- PR/branch/commit or immutable artifact reference;
- candidate names and one-line theses;
- exact files that implement the candidates and harness;
- governing product/design/domain documents;
- current production UI/captures for comparison;
- the reference user/environment;
- the invariant test matrix.

Verify the preview yourself before handing it to judges. A broken tunnel, stale branch, or mismatched commit invalidates the review.

## Phase 3 — Independent judging

Run at least two judges with the **same prompt structure** and isolated contexts.

Each judge must inspect the governing docs, current product, candidate code, and rendered candidates before scoring. It should test or render the relevant matrix rather than infer responsive/localization quality from source alone when browser tooling is available.

Use a rubric tied to the actual product job. For the recovered Taurifer rounds the dimensions were:

- in-gym usability and one-thumb ergonomics;
- next-target clarity;
- trust and intelligibility of recommendations;
- data legibility;
- brand/design-system fit;
- PT-first robustness and text expansion;
- cross-screen coherence;
- polish across widths/themes/states;
- implementation feasibility against real architecture/domain logic;
- actual improvement over the current product.

Update these dimensions when the product problem changes. Do not import generic SaaS criteria that are unrelated to the decision.

### Judge output contract

Require every judge to return:

1. a candidate-by-dimension comparison with concise evidence for each assessment;
2. a per-screen/per-state verdict identifying the strongest treatment and why;
3. each direction's single biggest strength and weakness;
4. correctness or semantic defects that apply regardless of taste;
5. implementation conflicts with the real product/domain;
6. a concrete overall recommendation: one direction or an explicit synthesis;
7. file/line or artifact citations for specific claims.

Scores may help force explicit comparison, but **scores are evidence annotations, not votes**. Never choose a winner by averaging judge totals.

Tell judges to be critical, not diplomatic. A judge may conclude that none of the candidates is good enough.

### Judge contamination guard

Judges must not:

- read sibling judge reports before submitting;
- receive a provisional favorite from the orchestrator;
- inherit the generator's rationale as authoritative;
- optimize for agreement with an earlier review;
- change the candidate implementation while judging it.

A judge can inspect commit history and product docs, but should evaluate the supplied candidate artifact independently.

## Phase 4 — Synthesize, do not vote

Only begin synthesis after all judge reports are complete.

The synthesizer receives the reports simultaneously plus the source-of-truth artifacts. Its job is to resolve disagreement, not find the numerical midpoint.

Classify every material finding into:

- **correctness/domain:** factual or semantic errors that must be fixed;
- **system constraint:** accessibility, localization, architecture, design-system, or interaction rules that should become invariant;
- **task usability:** evidence about speed, reach, glanceability, cognitive load, error recovery, or other job-specific behavior;
- **information architecture:** hierarchy, density, navigation, discoverability, and cross-screen coherence;
- **visual/craft:** typography, spacing, composition, material, motion, and polish;
- **implementation cost/risk:** complexity or conflict with the current product;
- **open taste decision:** legitimate alternatives where evidence does not settle the choice.

When judges disagree, inspect the underlying artifact and source of truth. Prefer demonstrated product correctness and task evidence over reviewer confidence or score totals.

Do not silently convert exploratory ideas into product decisions. Mark each synthesis item as one of:

- **settled** — required in the next round;
- **rejected** — explicitly excluded;
- **open** — intentionally left for new candidate exploration.

## Synthesis output contract

Write a durable Markdown spec that a fresh implementation agent can execute without reading the raw review debate.

It should contain:

1. the chosen base system or synthesized direction;
2. a per-screen/per-component decision map;
3. cross-cutting system rules;
4. mandatory correctness/domain fixes;
5. interaction and accessibility floors;
6. design-system and copy constraints;
7. patterns explicitly rejected and why;
8. an explicit decision log of settled vs open questions;
9. implementation boundaries;
10. acceptance checks covering the full comparison matrix.

Whenever possible, cite the actual product/code evidence behind a settled rule. The spec is the compression boundary between review and the next generation round.

For Taurifer, preserve the distinction between product truth and prototype truth. Prototype-derived calculations, outcome labels, or invented engine semantics do not become canonical merely because multiple judges liked their presentation.

## Phase 5 — Generate the next round

Start a fresh generator context.

Give it:

- the consolidated spec;
- current repository/source-of-truth docs;
- the existing review harness and candidate artifacts;
- the same invariant test matrix;
- explicit freedom over decisions still marked open.

Do **not** give the next generator the judges' raw score tables by default. The consolidated spec exists to remove anchoring and reviewer-personality effects.

The next round should contain:

- one faithful implementation of the synthesized direction, serving as the control;
- additional candidates that preserve every settled correctness/system rule;
- meaningful divergence only on open decisions;
- identical data, screens/states, widths, languages, and themes;
- the same production quality bar and verification.

Then repeat independent judging and synthesis.

## Stop condition

Stop iterating when all of the following are true:

- correctness/domain checks pass;
- the candidate behaves coherently across the full state matrix;
- remaining judge disagreement is primarily taste rather than unresolved product risk;
- the preferred direction or synthesis can be specified without hidden assumptions;
- another round is unlikely to reveal a materially different useful design direction.

The tournament narrows design uncertainty; it does not remove human product ownership. Unless the user explicitly delegated the final choice, present the finalist(s), synthesis, evidence, and tradeoffs for the user to choose before production integration.

## Anti-patterns

- Generator judges its own work and becomes the only reviewer.
- Judge 2 sees Judge 1's verdict and converges around it.
- The synthesizer averages scores instead of resolving why judges disagree.
- Candidate A uses rich data while Candidate B gets an easier empty state.
- Candidates cover different screens or hide hard states.
- Different candidates use different engine outputs or domain calculations.
- Later rounds merely polish the prior winner instead of reopening documented open decisions.
- 'Different' candidates vary only color, radius, or minor copy.
- Generic SaaS conventions override the actual product job or established visual identity.
- Prototype shortcuts quietly become production semantics.
- The review page itself becomes production dependency.
- Responsive/localization/dark-mode checks happen only after a favorite is chosen.

## Prompt skeleton — generator

Use and adapt this structure:

> Build a candidate UI round for **{decision}** in **{product}**.
>
> Before designing, inspect **{source-of-truth docs/code/captures}**. Extract the current design language and actual behavior. The candidates must belong to this product, not a generic redesign.
>
> Reference user/environment: **{user, context, physical/cognitive constraints}**.
>
> Shared contract: **{screens/states}**, **{data/domain logic}**, **{widths/devices}**, **{languages}**, **{themes}**, **{accessibility floors}**.
>
> Create **{N}** meaningfully different directions. State each direction's one-line thesis. They may differ in **{open design axes}**, but every direction must preserve the shared contract.
>
> Use real product copy and real computed output. Keep exploration isolated from production code. Build one comparison harness that switches among all directions on the exact same states.
>
> Validate every direction against **{acceptance matrix}**. Serve/publish the harness and return the immutable artifact/PR plus preview URL and verification results.

For a later round, insert before the divergence instruction:

> Read **{consolidated spec}** in full. First build its synthesized direction faithfully as the control. Treat all items marked settled as fixed and all correctness fixes as mandatory. Then generate new directions that obey those rules while exploring only decisions marked open. Earlier candidates are not sacred beyond the settled rules.

## Prompt skeleton — judge

> Independently evaluate **{candidate set}** for **{product/user job}**.
>
> Artifact bundle: **{preview}**, **{PR/commit}**, **{candidate files}**.
>
> Before judging, read **{source-of-truth docs}** and compare the candidates with the current product. Inspect real domain/engine behavior where the UI presents computed claims.
>
> Judge **{primary locale}** first, then **{other locales}**, at **{widths/devices}**, in **{themes}**. Exercise the important interactions and hard states rather than reading source only.
>
> Evaluate: **{rubric dimensions}**.
>
> Return: (1) candidate-by-dimension comparison with evidence, (2) per-screen/state verdict, (3) biggest strength and weakness of each direction, (4) correctness/system defects, and (5) a concrete overall direction or synthesis.
>
> Cite file/line or artifact evidence for specific claims. Be critical. Do not grade on a curve and do not split the difference merely to be diplomatic.

Each judge receives this brief independently and receives no sibling report.

## Prompt skeleton — synthesizer

> You are consolidating **{N}** independent design reviews of the same candidate set.
>
> Read all reports together with **{source-of-truth artifacts}**. Do not select by score totals or majority vote.
>
> For every disagreement, inspect the underlying evidence and classify it as correctness/domain, system constraint, task usability, information architecture, visual craft, implementation risk, or open taste.
>
> Produce one normative design spec for a fresh implementation agent. Separate **settled**, **rejected**, and **open** decisions.
>
> The spec must include: base/synthesis direction, per-screen/component decisions, cross-cutting rules, correctness fixes, accessibility/localization constraints, rejected patterns, implementation boundaries, decision log, and acceptance checks.
>
> Preserve real product semantics. Do not turn a prototype shortcut or reviewer preference into a settled product rule without evidence.

## Recovered Taurifer tournament shape

The session history shows this concrete sequence:

1. **Round 1 generator:** produced A · Folha de treino, B · Um polegar, and C · Evidência in one review page.
2. **Shared harness:** the same eleven main-screen surfaces, coherent training data, PT/EN, light/dark, 360/390/430 widths, one/all-screen modes, and working review interactions.
3. **Independent reviews:** separate design-review contexts evaluated the same candidate artifact using the same product-grounded brief, without seeing one another's conclusions.
4. **Synthesis:** a separate context received the completed reports together and converted their disagreement into a definitive Direction D specification and decision log.
5. **Round 2 generator:** a fresh context was instructed to build D exactly, then create additional candidates that preserved D's correctness and system rules while reopening presentation choices.
6. **Repeat:** the new set returns to the same independent review and synthesis loop.

This architecture is intentional. Preserve the **role boundaries**, not necessarily the number of chat windows.

## Final report from a tournament run

Return only the information needed to steer the next decision:

- candidate artifact/PR and preview URL;
- candidate theses;
- verification matrix/result;
- independent judge report references;
- consolidated spec reference;
- what is settled, rejected, and still open;
- whether the next action is another round or human selection/promotion.