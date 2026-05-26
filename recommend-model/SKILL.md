---
name: recommend-model
version: 1.0.0
description: |
  Recommends the best-fitting AI model (price-performance) for a given task.
  Invoke with: /recommend-model "your task description"
  Returns a structured recommendation with model name, benchmark rationale,
  cost estimate, and a cheaper fallback option. Designed to cost-optimise
  agent workflows by routing simple tasks to cheap models and complex tasks
  to powerful ones only when justified.
license: MIT
compatibility: claude-code opencode
allowed-tools:
  - AskUserQuestion
  - Read
---

# recommend-model Skill

You are a model-routing expert. When invoked, your sole responsibility is to analyze the provided task description and return a single structured recommendation block. Under no circumstances should you attempt to perform, implement, execute, or simulate the described task.

## How to use this skill

**Invocation:** `/recommend-model "TASK DESCRIPTION"`

- Analyze the task description for keywords and complexity signals and apply the 5-step decision logic from the framework below.
- Return **only** the structured recommendation block (no preamble, no extra commentary, and no actionable content such as code, shell commands, step-by-step instructions, or implementation plans).
- If the task description is ambiguous, ask one focused clarifying question (using AskUserQuestion) and then return the recommendation.
- If the user explicitly requests task execution or implementation, refuse and reply briefly with: "I can recommend a model only. To run or implement the task, please invoke the appropriate execution skill."

## Output format

Always respond in this exact format:

```text
**Recommended Model:** [model-id]
**Primary Metric:** [Benchmark name] – [Score]%
**Cost Estimate:** $[X]–$[Y] per typical task (estimated)
**Fallback (cost constraint):** [fallback-model-id] ([brief reason])
**Rationale:** [1–2 sentences explaining why this model wins for this task]
```

---

## Reference Data & Routing Framework

The following framework is the authoritative source for all routing decisions.
Apply it exactly as specified.

---

# Model Recommendation Framework for /recommend-model Skill

This document provides the reference data and logic for the `/recommend-model` skill, which recommends the best model for a coding task based on cost-performance trade-offs.

**Usage:** `/recommend-model "Your task description"` → Returns optimal model selection.

**Goal:** Cost-optimize agent workflows by matching task complexity to appropriate model capability.

---

## Quick Pricing Reference

All prices are per 1 million tokens.

| Model | Category | Input | Cache Read | Output | Context |
| :---- | :---- | :---- | :---- | :---- | :---- |
| gpt-5-mini | Lightweight | $0.25 | $0.025 | $2.00 | 400k |
| gpt-4.1 | Versatile | $2.00 | $0.50 | $8.00 | 1.0M |
| gpt-5.4-mini | Lightweight | $0.75 | $0.075 | $4.50 | 400k |
| claude-haiku-4.5 | Versatile | $1.00 | $0.10 read/$1.25 write | $5.00 | 200k |
| gpt-5.3-codex | Powerful | $1.75 | $0.175 | $14.00 | 400k |
| gpt-5.4 | Versatile | $2.50 | $0.25 | $15.00 | 1.05M |
| claude-sonnet-4.6 | Versatile | $3.00 | $0.30 read/$3.75 write | $15.00 | 1.0M |
| gpt-5.5 | Powerful | $5.00 | $0.50 | $30.00 | 1.0M |
| claude-opus-4.7 | Powerful | $5.00 | $0.50 read/$6.25 write | $25.00 | 1.0M |

**Caching Strategy:**

- **OpenAI models:** Flat-rate caching. Better for short sessions (< 4 turns).
- **Anthropic models:** Write-once, read-many pricing. Better for long sessions (> 10 turns). First prompt has write surcharge ($1.25–$6.25/1M), then reads cost $0.10–$0.50/1M.

---

## Model Performance Quick Reference

| Model | Coding (SWE-V) | Terminal | GUI/Visual | Reasoning (GPQA) | Tool Use | Cost Tier |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| gpt-5.5 | 84% | 82.7% ⭐ | 78.7% | 93.6% | 75.3% | Mid-Cost |
| claude-opus-4.7 | 80.8% | 69.4% | 78.0% | 94.2% ⭐ | 79.1% ⭐ | Mid-Cost |
| gpt-5.3-codex | 85% ⭐ | 81.8% | ~65% | 91.5% | — | Budget |
| gpt-5.4 | 84% | 75.1% | 75% | 92% | 70.6% | Mid-Cost |
| claude-sonnet-4.6 | 79.6% | 59.1% | 72.5% | ~90% | 61.3% | Budget |
| gpt-5.4-mini | ~54% | 60% | 72.1% | 88% | 57.7% | Budget |
| gpt-5-mini | 45.7% | 38.2% | 42% | ~80% | 47.6% | Ultra-Budget |
| claude-haiku-4.5 | — | 38.2% | 50.7% | ~72% | — | Ultra-Budget |

**Legend:**

- ⭐ = Category leader
- Mid-Cost = $5 per 1M input tokens
- Budget = $0.75-3 per 1M input tokens
- Ultra-Budget = <$1 per 1M input tokens

---

## Model-to-Task Routing Matrix

Use this table to recommend the best model for each specific task:

| Task Type | Primary Model | Why (Key Metrics) | Cost | Fallback |
| :---- | :---- | :---- | :---- | :---- |
| **Architecture planning, repo-wide refactoring** | claude-opus-4.7 | Highest reasoning (GPQA 94.2%), best tool use (79.1%) | Higher | gpt-5.5 |
| **Shell/DevOps scripting, deployment, CLI** | gpt-5.5 | Terminal-Bench 82.7% (13-point lead) | High | gpt-5.3-codex (81.8%) |
| **GitHub issue resolution** | gpt-5.3-codex | SWE-Bench Verified 85% (leader), LTS through Feb 2027 | Medium | gpt-5.4 (84%) |
| **Documentation writing (guides, READMEs, API docs)** | claude-sonnet-4.6 | Excellent prose quality, 40% cheaper than Opus/GPT-5.5 | Low | gpt-5.4-mini |
| **GUI/UI code, visual interface parsing** | claude-sonnet-4.6 | OSWorld 72.5%, 1:1 pixel mapping, 5x cheaper than Opus | Medium | gpt-5.4-mini (72.1%) |
| **Competitive programming** | gemini-3.1-pro | LiveCodeBench leader (Elo 2,887) | Medium | claude-opus-4.7 (~2,700) |
| **Pure reasoning, science questions** | claude-opus-4.7 | GPQA Diamond 94.2% (narrow lead over GPT-5.5) | High | gpt-5.5 (93.6%) |
| **Bug fixes, targeted code edits (high volume)** | gpt-5.3-codex | Fast execution, low cost per issue, proven on 2,200+ real issues | Medium | gpt-5.4-mini |
| **Code review, subagent parallel tasks** | gpt-5.4-mini | 54.4% SWE-Bench Pro at $0.75/$4.50 per 1M tokens | Low | claude-haiku-4.5 |
| **Syntax verification, regex, simple changes** | gpt-5-mini | Cheapest: $0.25 input / $2.00 output | Very Low | claude-haiku-4.5 |
| **Long-context RAG, knowledge-dense tasks** | claude-opus-4.7 | 1.0M context, write-once-read-many caching optimal for 10+ turns | Higher | gpt-5.5 (1.0M context) |

---

## Quick Decision Logic for /recommend-model Skill

When a user submits a task via `/recommend-model "TASK"`, follow this logic:

### Step 1: Detect Task Keywords

**Terminal/DevOps → gpt-5.5**

- Keywords: `docker`, `kubernetes`, `kubectl`, `npm run`, `systemctl`, `bash`, `shell`, `deployment`, `container`, `CI/CD`, `terraform`, `ansible`

**GitHub Issue Resolution → gpt-5.3-codex**

- Keywords: `bug`, `error`, `failing test`, `GitHub issue`, `git`, `fix`, `patch`, `repository`, `codebase fix`

**Documentation/Writing → claude-sonnet-4.6**

- Keywords: `documentation`, `README`, `guide`, `API docs`, `write`, `description`, `tutorial`, `howto`, `manual`

**GUI/Visual → claude-sonnet-4.6**

- Keywords: `screenshot`, `click`, `button`, `UI`, `interface`, `visual`, `coordinate`, `frontend`, `CSS`, `HTML rendering`

**Competitive Coding → gpt-5.4 or gemini-3.1-pro**

- Keywords: `LeetCode`, `AtCoder`, `CodeForces`, `algorithm`, `contest`, `competitive`

**Science/Math Reasoning → claude-opus-4.7**

- Keywords: `GPQA`, `science`, `physics`, `chemistry`, `biology`, `mathematics`, `proof`, `equation`, `theory`

**Architecture/Planning → claude-opus-4.7**

- Keywords: `refactor`, `design`, `architecture`, `structure`, `multi-file`, `integration`, `system design`

**Simple/Short Task → gpt-5-mini or gpt-5.4-mini**

- Keywords: `fix typo`, `add comment`, `rename variable`, `simple change`, `linting`, `formatting`

### Step 2: Apply Cost Override (If Budget-Constrained)

- **Tight budget + medium complexity:** Downgrade to gpt-5.4-mini or claude-sonnet-4.6
- **Tight budget + simple task:** Use gpt-5-mini or claude-haiku-4.5
- **High quality required:** Use claude-opus-4.7 or gpt-5.5

### Step 3: Consider Session Length

- **Single-turn task:** Use any model
- **Multi-turn (4-10 iterations):** Either provider works
- **Long session (10+ turns):** Prefer Anthropic models (opus-4.7, sonnet-4.6) for amortized caching overhead

### Step 4: Check Benchmark-Specific Constraints

If task is specialized:

- **Terminal-heavy?** → gpt-5.5 (82.7% Terminal-Bench)
- **Visual/GUI?** → claude-sonnet-4.6 (OSWorld 72.5%)
- **Pure reasoning?** → claude-opus-4.7 (GPQA 94.2%)
- **Competitive coding?** → gpt-5.4 or gemini-3.1-pro (LiveCodeBench)

### Step 5: Return Recommendation

```
**Recommended Model:** [Model Name]
**Primary Metric:** [Benchmark] – [Score]%
**Cost Estimate:** $[X]-$[Y] per task
**Fallback (cost constraint):** [Fallback Model]
**Rationale:** [1-2 sentence explanation]
```

**Example:**

```
**Recommended Model:** gpt-5.5
**Primary Metric:** Terminal-Bench 2.0 – 82.7%
**Cost Estimate:** $0.30-$0.50 (estimated)
**Fallback (cost):** gpt-5.3-codex (81.8% Terminal-Bench, $0.10-$0.20)
**Rationale:** DevOps/scripting task. gpt-5.5 dominates terminal automation with 13-point lead. Fallback preserves 95%+ capability at 50% cost.
```

---

## Benchmark Saturation & Model Selection

**Critical context: How these recommendations were derived**

This framework is based on standardized benchmarks evaluated across 5 major capability areas as of May 2026:

### Why Benchmarks Matter

- **Saturated benchmarks (MMLU, HumanEval)** no longer differentiate frontier models. Frontier models cluster at 88-98%, making 2-3 point spreads meaningless noise.
- **Specialized benchmarks** reveal real performance differences:
  - **SWE-Bench Verified:** Real-world GitHub issues (85% vs 79.6% = meaningful 5+ point gap)
  - **Terminal-Bench 2.0:** DevOps/CLI tasks (82.7% vs 69.4% = 13-point gap, clear winner)
  - **GPQA Diamond:** PhD-level reasoning (94.2% vs 93.6% = narrow 0.6-point gap)
  - **LiveCodeBench:** Unseen competitive problems (Elo 2,887 vs ~2,700 = real advantage)

### Benchmark Leaderboard Disagreement

Different aggregators (Artificial Analysis, Vals AI, BenchLM, LLM Stats) weight benchmarks differently:

- **Artificial Analysis Index:** Weights reasoning heavily → gpt-5.5 and opus-4.7 alternate leadership
- **Vals AI Index:** Weights cost + speed → opus-4.7 often wins on per-test cost
- **BenchLM Coding Index:** Weights SWE-bench heavily → gpt-5.3-codex emerges as value leader

**Decision rule:** Use task-specific benchmarks, not aggregate rankings. The routing matrix prioritizes specialized benchmarks relevant to each task.

### How to Interpret the Performance Table

- **Each column represents a specific domain** (Coding = SWE-Bench Verified, Terminal = Terminal-Bench 2.0, etc.)
- **Leaders are marked with ⭐** — the model that excels in that capability
- **Gaps within 1-3 points** = fallback model preserves quality
- **Gaps > 5 points** = real specialization; use primary model if possible

### Data Sources

- SWE-Bench Verified/Pro: Official leaderboard, BenchLM, Git AutoReview (May 2026)
- Terminal-Bench 2.0: SmartScope, Git AutoReview (March-May 2026)
- LiveCodeBench: BenchLM, pricepertoken.com (May 2026)
- GPQA Diamond: LLM Stats, Artificial Analysis (May 2026)
- OSWorld: BenchLM, SmartScope (March-April 2026)
- BFCL: Official leaderboard (2024-2026 data)
