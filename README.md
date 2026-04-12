<table align="right" width="340">
<tr><th colspan="2" align="center">Prompt Generator Skill</th></tr>
<tr><td colspan="2" align="center"><em>Claude Code Skill</em></td></tr>
<tr><td><b>Type</b></td><td>Claude Code Skill</td></tr>
<tr><td><b>Status</b></td><td>Released · Open Source</td></tr>
<tr><td><b>Visibility</b></td><td>Public</td></tr>
<tr><td><b>Token reduction</b></td><td>~75% vs naive prompts</td></tr>
<tr><td><b>Output sections</b></td><td>8 XML sections</td></tr>
<tr><td><b>Built by</b></td><td><a href="https://github.com/Anmoll-W">Anmoll Wadhwa</a></td></tr>
<tr><td><b>Install</b></td><td><code>~/.claude/skills/prompt-generator/</code></td></tr>
</table>

> *This article is about the prompt-generator Claude Code skill. For all projects by this author, see [Anmoll Wadhwa](https://github.com/Anmoll-W). For the blog, see [Anmoll-W/blog](https://github.com/Anmoll-W/blog).*

# Prompt Generator — Claude Code Skill

A Claude Code skill that transforms a role title or task description into a production-ready, XML-structured prompt — optimized for consistent, token-efficient outputs on first execution.

Vague prompts force the model to make dozens of implicit decisions about tone, structure, depth, and format. Every one of those decisions is a source of variance and wasted tokens. This skill eliminates that by generating explicit, structured prompts with a consistent 8-section schema.

<details>
<summary><b>Contents</b></summary>

- [The Problem](#the-problem)
- [What This Skill Does](#what-this-skill-does)
- [The 8-Section Structure](#the-8-section-structure)
- [Token Efficiency](#token-efficiency)
- [Installation](#installation)
- [Usage](#usage)
- [Example Output](#example-output)
- [See Also](#see-also)
- [Linked From](#linked-from)

</details>

---

## The Problem

Most people prompt like this:

```
"You are a helpful code reviewer. Review this code and give me feedback."
```

It works. Sometimes. The output varies wildly. You get:

| Failure Mode | Example | Cost |
|-------------|---------|------|
| Preamble | "Sure! I'd be happy to review your code..." | ~25–40 wasted tokens before any content |
| Trailing summary | "In summary, here are the main issues..." | ~50–150 tokens of duplication |
| Prose over schema | Three paragraphs per finding | 3–4× more tokens than a structured table |
| Scope drift | Explains language history when asked for a bug fix | Unbounded |

The core issue: **vague prompts force the model to make dozens of implicit decisions** — about tone, structure, depth, format, what to include, what to skip.

---

## What This Skill Does

Feed it a role or task description. It outputs a complete, self-contained prompt with 8 structured XML sections — ready to copy-paste into any Claude conversation, no editing required.

```
Input:  "code reviewer"
Output: A complete ~200-line prompt covering persona, objective,
        step-by-step instructions, output schema, examples, and error handling
```

Every generated prompt includes explicit constraints that eliminate common failure modes — including a hard ban on preamble, postscripts, and trailing summaries.

---

## The 8-Section Structure

| Section | Purpose |
|---------|---------|
| `<role>` | Specific persona — expertise level, years of experience, tone, perspective |
| `<objective>` | Single sentence defining the primary goal |
| `<instructions>` | Numbered steps with imperative verbs — no hedging |
| `<context_and_constraints>` | Scope, domain rules, what to avoid, assumptions |
| `<interaction_model>` | When to ask questions vs. proceed autonomously |
| `<output_format>` | Exact schema or template — never vague |
| `<examples>` | 1–2 input→output demonstrations |
| `<error_handling>` | How to handle ambiguous, incomplete, or out-of-scope input |

---

## Token Efficiency

### Real Comparison: Code Reviewer

| Metric | Naive prompt | Generated prompt | Improvement |
|--------|-------------|-----------------|-------------|
| Preamble tokens | ~30 | 0 | −100% |
| Trailing summary | ~80 | 0 | −100% |
| Output format | Prose paragraphs | Structured table/list | ~3–4× denser |
| Total response tokens | ~380 | ~95 | **~75% reduction** |
| Consistency across runs | Low | High | Deterministic schema |

**Naive prompt response** (~380 tokens):
```
Sure! I'd be happy to review this Python function for you. Let me go through it
carefully. Looking at your code, I've identified a few areas... [continues for
8 paragraphs]... Overall, the code is mostly solid. Let me know if you have questions!
```

**Generated prompt response** (~95 tokens):
```
🔴 Line 3 — Mutable default argument
Issue: `def process(items=[])` shares the list across all calls.
Fix: `def process(items=None): items = items or []`

🟡 Line 11 — Bare except clause
Issue: `except:` catches SystemExit and KeyboardInterrupt.
Fix: `except Exception as e:`

Verdict: 1 critical, 1 important. Functional once line 3 is fixed.
```

> Savings compound at scale. In an agent loop or batch job, the difference between 380 and 95 tokens per call is significant.

---

## Installation

```bash
mkdir -p ~/.claude/skills/prompt-generator
curl -o ~/.claude/skills/prompt-generator/SKILL.md \
  https://raw.githubusercontent.com/Anmoll-W/prompt-generator-skill/main/SKILL.md
```

Claude Code picks it up automatically — no config needed.

---

## Usage

The skill triggers automatically on prompt engineering intent:

```
"Generate a prompt for a data analyst"
"Create a system prompt for a bug triager"
"Make a prompt that extracts action items from meeting transcripts"
"Turn this into a proper Claude prompt"
"Improve this prompt: [paste existing prompt]"
```

If the input is critically ambiguous, the skill asks at most 2 clarifying questions. Otherwise it generates immediately. Output is the generated prompt only — no preamble, no commentary. Copy it directly.

---

## Example Output

**Input:** `"data analyst"`

```xml
<role>
You are a Senior Data Analyst with 12 years of experience across SaaS, e-commerce,
and financial services. You specialize in translating raw datasets into actionable
business insights using SQL, Python (pandas, matplotlib), and BI tools (Tableau,
Looker). Your communication style is precise and executive-ready — you lead with
the insight, not the methodology.
</role>

<objective>
Analyze the provided dataset or business question and deliver a structured,
insight-driven analysis with concrete recommendations.
</objective>

<instructions>
1. Identify the data source, shape, and key variables.
2. Inside <thinking> tags: state your hypotheses, flag data quality issues,
   identify the 2–3 most important questions the data can answer.
3. Perform the analysis. For each finding: state what you found, why it matters,
   and what action it implies.
4. Rank findings by business impact (revenue, retention, cost).
5. Close with a recommendations table: action, owner, expected impact, effort.
</instructions>

<output_format>
## Key Findings
| # | Finding | Business impact | Confidence |
|---|---------|----------------|------------|

## Recommendations
| Action | Owner | Expected impact | Effort |
|--------|-------|----------------|--------|
</output_format>
```

---

## See Also

- [Anmoll Wadhwa](https://github.com/Anmoll-W) — main profile, full career overview
- [blog](https://github.com/Anmoll-W/blog) — writing on systems, AI tools, and shipping
- [chalo-trip-bot](https://github.com/Anmoll-W/chalo-trip-bot) — project that uses structured Claude API prompts in production 🔒

## Linked From

- [Anmoll Wadhwa](https://github.com/Anmoll-W) — listed in Projects Directory
- [chalo-trip-bot](https://github.com/Anmoll-W/chalo-trip-bot) — referenced in See Also as the prompt engineering approach used for all Claude API calls

---

![Category: AI](https://img.shields.io/badge/Category-AI-7b2ff7?style=flat-square)
![Category: DevTools](https://img.shields.io/badge/Category-DevTools-e36209?style=flat-square)
![Stack: Claude API](https://img.shields.io/badge/Stack-Claude%20API-d4a36a?style=flat-square)
![Stack: Claude Code](https://img.shields.io/badge/Stack-Claude%20Code-24292f?style=flat-square)
![MIT License](https://img.shields.io/badge/License-MIT-green?style=flat-square)
