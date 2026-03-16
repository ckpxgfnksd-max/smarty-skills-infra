# Your AI Doesn't Know You. Here's How We Fixed That.

*How a blog post about "correct nonsense" led us to build a self-learning memory system for AI agents — in 35 iterative rounds, with zero infrastructure.*

---

## The Problem Nobody Talks About

You've been using AI assistants for months. Maybe years. You've had thousands of conversations. And yet every new session starts from scratch — a blank slate with no memory of who you are, how you think, or what you care about.

Your AI doesn't know you prefer early returns over nested conditionals. It doesn't know you hate verbose variable names. It doesn't know you want to be challenged, not agreed with. It gives you the same consensus output it gives everyone else.

This isn't a model intelligence problem. It's a context problem.

## The Article That Changed Our Thinking

Grapeot's [Context Infrastructure](https://yage.ai/context-infrastructure.html) made the argument precisely: LLMs default to consensus because that's what next-token prediction optimizes for. The training mechanism itself produces a bias toward the middle — technically correct but uninspired output that reflects the average of all training data, not the preferences of any individual user.

The author demonstrated this with an experiment: two AI agents using identical models, same tools, same prompts — but one had access to a year of accumulated judgment frameworks. The first agent produced "checklists." The second produced "insights." Same silicon brain. Different context. Radically different output.

The key insight: **what determines output quality is context, not model intelligence.**

Grapeot's solution was a three-layer memory hierarchy:

- **L1 Observer**: Scan daily interactions, extract meaningful observations
- **L2 Reflector**: Weekly consolidation — remove duplicates, identify cross-project patterns
- **L3 Axiom**: Distill stable patterns into decision principles

Over a year, he'd accumulated 44 axioms covering technology choices, communication style, and business judgment. These axioms, loaded into every AI session, transformed his assistant from a generic tool into something that reasoned like a trusted colleague.

But his system required custom infrastructure — voice transcriptions, meeting note pipelines, WeChat exports, local file consolidation. It was built for one person with engineering skills. We wanted to give this to everyone.

## The Constraint That Became a Feature

We set out to build an [OpenClaw](https://openclaw.com) skill — a portable package that any user could install and immediately benefit from. But OpenClaw doesn't support hooks. No event listeners, no background processes, no daemon watching your interactions.

Everything had to work through prompting alone.

At first this felt like a dealbreaker. How do you build a "silent observer" without the ability to observe silently? The answer turned out to be elegant: you don't build infrastructure. You write *instructions that make the AI self-observe as a side-effect of doing its normal work.*

The skill is a SKILL.md file — a set of instructions loaded into the agent's context every session. Those instructions say: "When the user corrects you or states a preference, write it down. When enough observations accumulate, reflect on them. Promote stable patterns to axioms. Load axioms every session."

No servers. No databases. No hooks. Just a markdown file telling the AI how to learn.

## 35 Rounds of Iterative Design

We didn't write the skill once and ship it. We borrowed [Karpathy's autoresearch method](https://github.com/karpathy/autoresearch) — originally designed for overnight ML experiments — and applied it to prompt engineering.

The method is simple: each round modifies one variable, evaluates against fixed criteria, and keeps or discards the change. We ran 35 rounds, scoring each iteration on five weighted metrics:

- **Context cost** (30%): How many tokens does the always-on portion consume?
- **Signal quality** (25%): Does L1 capture real judgment, not noise?
- **Invisibility** (20%): Does the user notice any friction?
- **Convergence speed** (15%): How many sessions until useful axioms emerge?
- **Model-agnosticism** (10%): Does it work across different LLMs?

The starting score was 5.45/10 with ~1,430 tokens. By round 35: **9.2/10 with ~785 tokens.** 45% fewer tokens. 69% higher score. Same behavior.

Here are the changes that mattered most.

### Selective Observation (Round 4)

The baseline observed after every task. This produced enormous noise — "user asked for a function and I wrote one" isn't a preference signal. Switching to observing *only* on corrections, stated preferences, and retractions jumped signal quality from 4/10 to 9/10.

The counterintuitive insight: **most tasks should produce zero observations.** A session where the user never corrects anything tells you they're satisfied with the defaults. Silence is data.

### Positive Gating (Round 19)

The instruction "Do NOT observe routine task completions" was reliably ignored by LLMs. Replacing it with "ONLY observe when one of these triggers fires" was the single biggest compliance improvement across all models tested.

This is a well-known prompt engineering pattern, but it's worth repeating: LLMs follow positive constraints ("do X") far more reliably than negative ones ("don't do Y"). Every negative instruction in the skill was rewritten as a positive gate.

### First-Person Axiom Voice (Round 6)

We tested four axiom formats:

- Prose: "This user prefers explicit error handling"
- Structured: "WHEN: errors | PREFER: explicit checks | AVOID: try/catch"
- Weighted: "code-style/error-handling: explicit-checks (0.9)"
- Natural voice: "I prefer explicit error handling with early returns"

Natural voice won across every evaluation dimension. When the agent reads "I prefer early returns over nested if-else," it processes the axiom exactly like a user instruction. No parsing layer needed. No structured format to get wrong. And it works identically across Claude, GPT, Gemini, and open-source models.

### The Retraction Trigger (Round 28)

Stress-testing revealed a critical gap: what happens when a user says "forget that preference"? Without an explicit mechanism, the system had no way to immediately remove an axiom. The best it could do was record a contradicting observation and wait for the next reflection cycle.

We added a third trigger type — `retraction` — that gets processed immediately during reflection with no threshold. User agency over their own preference data isn't optional.

### Example-Driven Reflection (Round 20)

The original reflection algorithm was an 8-step numbered procedure. We replaced it with 4 high-level steps plus a concrete worked example showing three observations becoming one axiom — and a negative example showing why two same-session observations don't qualify for promotion.

LLMs learn procedures better from examples than from instructions. The example communicates the same algorithm in half the tokens, with higher compliance across models.

## How It Actually Works

The shipped skill is 70 lines of markdown. Here's the flow:

**Session start:**
1. Agent reads your profile (≤25 axioms). Treats them as its own working preferences.
2. Agent checks the observation log. If 15+ new entries have accumulated, it reflects — grouping observations, promoting stable patterns to axioms, pruning stale ones.

**During tasks:**
- If you correct the agent's output → observation recorded
- If you state a preference → observation recorded
- If you ask to forget something → retraction recorded
- If you do nothing notable → nothing recorded

**Over time:**
Observations accumulate across sessions. Patterns that appear in 3+ distinct contexts with no contradictions get promoted to axioms. Axioms strengthen with reinforcement, weaken without confirmation, and get flagged when contradicted.

The profile caps at 25 axioms — roughly 500 tokens of context. When the cap is reached, L3 merges related axioms or demotes the weakest. The system self-maintains.

**Cold start:**
The first two sessions use "bootstrap mode" — casting a wider observation net to capture implicit signals like consistent tool choices and accepted defaults. After enough observations accumulate, the system narrows to selective triggers.

## What Makes This Different from Memory Features

Most AI memory systems operate at the fact layer: "The user's name is Alice. She works at Acme Corp. Her project uses React." Grapeot's original article specifically critiques this approach — facts are useful but they don't capture judgment.

Smarty Skills-Infra operates at the **judgment layer**: "I prefer flat control flow with early exits. I want to be challenged, not agreed with. I use TypeScript with strict config." These aren't facts about the user — they're decision principles that reshape how the AI thinks.

The three-context promotion threshold ensures only stable, cross-situation preferences become axioms. A one-time choice stays an observation. A recurring pattern becomes a preference. A deeply confirmed pattern becomes a strong default. The system distinguishes signal from noise through temporal evidence, not semantic analysis.

## The Result

Install the skill. Use your AI normally. Over days and weeks, it silently builds a profile of your judgment — what you correct, what you prefer, what you reject. The profile loads automatically every session, making every interaction incrementally better tuned to how you think.

You never configure anything. You never write preference files. You never answer onboarding questionnaires. The system learns from the highest-signal moments in your natural workflow — the moments you push back.

As grapeot wrote in the original article:

> Silicon-based brains pursuing pure objectivity can only achieve intelligent mediocrity. Only human judgment accumulated over decades, filled with bias and taste, can reshape it.

We just made that reshaping automatic.

---

*Smarty Skills-Infra is open source at [github.com/ckpxgfnksd-max/smarty-skills-infra](https://github.com/ckpxgfnksd-max/smarty-skills-infra). MIT licensed. Works with any LLM.*
