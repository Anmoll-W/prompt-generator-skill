# Prompt Generator — Claude Code Skill

A Claude Code skill that transforms a role title or task description into a production-ready, XML-structured prompt — optimized for consistent, token-efficient outputs on first execution.

---

## The Problem with Naive Prompts

Most people prompt like this:

```
"You are a helpful code reviewer. Review this code and give me feedback."
```

It works. Sometimes. The output varies wildly depending on Claude's mood, the conversation history, and how the stars align. You get:

- Long preambles ("Sure! I'd be happy to review your code...")
- Inconsistent structure (bullet list today, prose tomorrow)
- Scope drift (explains things you didn't ask for)
- Wasted tokens on filler before the actual answer

The core issue: **vague prompts force the model to make dozens of implicit decisions** — about tone, structure, depth, format, what to include, what to skip. Every one of those decisions is a source of variance.

---

## What This Skill Does

Feed it a role or task description. It outputs a complete, self-contained prompt with 8 structured XML sections — ready to copy-paste into any Claude conversation, no editing required.

```
Input:  "code reviewer"
Output: A complete 200-line prompt covering persona, objective,
        step-by-step instructions, output schema, examples, and error handling
```

Every generated prompt includes explicit constraints that eliminate common failure modes — including a hard ban on preamble, postscripts, and trailing summaries.

---

## The 8-Section Structure

| Section | Purpose |
|---|---|
| `<role>` | Specific persona — expertise level, years of experience, tone, perspective |
| `<objective>` | Single sentence defining the primary goal |
| `<instructions>` | Numbered steps with imperative verbs (no hedging) |
| `<context_and_constraints>` | Scope, domain rules, what to avoid, assumptions |
| `<interaction_model>` | When to ask questions vs. proceed autonomously |
| `<output_format>` | Exact schema or template — never vague |
| `<examples>` | 1–2 input→output demonstrations |
| `<error_handling>` | How to handle ambiguous, incomplete, or out-of-scope input |

---

## Token Efficiency

This is where the structural approach pays off concretely.

### Why Naive Prompts Waste Tokens

When a prompt doesn't specify format, Claude fills the gap with what it was trained to produce: courteous, thorough, narrative responses. That's expensive.

| Pattern | Example | Token cost |
|---|---|---|
| Preamble | "Sure! I'd be happy to help you review this code. Let me take a look..." | ~25–40 tokens wasted before any content |
| Trailing summary | "In summary, here are the main issues I found: [repeats everything]" | ~50–150 tokens of duplication |
| Prose over schema | Three paragraphs describing a finding | 3–4x more tokens than a structured table |
| Scope drift | Explains language history when asked for a bug fix | Unbounded |

### What Generated Prompts Do Differently

Every prompt this skill produces includes:

```xml
<context_and_constraints>
  Do not output preamble, postscripts, or meta-commentary.
  Do not summarize what you just did.
  Begin your response with the first meaningful output token.
</context_and_constraints>

<output_format>
  [Most compact format sufficient for the task — table or structured skeleton preferred]
</output_format>
```

### Real Comparison: Code Reviewer

**Naive prompt:** `"Review this Python function and tell me what's wrong."`

**Response:**
```
Sure! I'd be happy to review this Python function for you. Let me go through it carefully.

Looking at your code, I've identified a few areas that could be improved:

Firstly, on line 3, you're using a mutable default argument (def process(items=[])).
This is a common Python gotcha — mutable defaults are shared across all calls to the
function, which can lead to unexpected behavior if items gets modified...

[continues for 8 more paragraphs]

Overall, the code is mostly solid but has a few issues to address. The mutable default
argument is the most critical fix. Let me know if you have any questions!
```

**Token count: ~380 tokens**

---

**Generated prompt output for the same function:**

```
🔴 Line 3 — Mutable default argument
Issue: `def process(items=[])` shares the list across all calls.
Fix: `def process(items=None): items = items or []`

🟡 Line 11 — Bare except clause
Issue: `except:` catches SystemExit and KeyboardInterrupt.
Fix: `except Exception as e:`

🟢 Line 7 — Variable name `l` is ambiguous
Fix: Rename to `lines` or `items_list`.

Verdict: 1 critical, 1 important, 1 suggestion. Functional once line 3 is fixed.
```

**Token count: ~95 tokens**

### Summary

| Metric | Naive prompt | Generated prompt | Improvement |
|---|---|---|---|
| Preamble tokens | ~30 | 0 | −100% |
| Trailing summary | ~80 | 0 | −100% |
| Output format | Prose paragraphs | Structured table/list | ~3–4x denser |
| Total response tokens | ~380 | ~95 | **~75% reduction** |
| Consistency across runs | Low | High | Deterministic schema |

Savings compound at scale. If you're running this in an agent loop, a batch job, or any system that calls Claude repeatedly — the difference between 380 and 95 tokens per call is significant.

---

## Installation

1. Clone or copy `SKILL.md` into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/prompt-generator
curl -o ~/.claude/skills/prompt-generator/SKILL.md \
  https://raw.githubusercontent.com/Anmoll-W/prompt-generator-skill/main/SKILL.md
```

2. Claude Code picks it up automatically — no config needed.

---

## Usage

The skill triggers automatically on prompt engineering intent. Just say:

```
"Generate a prompt for a data analyst"
"Create a system prompt for a bug triager"
"Make a prompt that extracts action items from meeting transcripts"
"Turn this into a proper Claude prompt"
"Improve this prompt: [paste existing prompt]"
```

If the input is critically ambiguous, the skill asks at most 2 clarifying questions. Otherwise it generates immediately.

**Output:** The generated prompt only — no preamble, no commentary. Copy it directly.

---

## Example Output

**Input:** `"data analyst"`

```xml
<role>
You are a Senior Data Analyst with 12 years of experience across SaaS, e-commerce,
and financial services. You specialize in translating raw datasets into actionable
business insights using SQL, Python (pandas, matplotlib), and BI tools (Tableau,
Looker). Your communication style is precise and executive-ready — you lead with
the insight, not the methodology. You know that a finding without a recommendation
is just a number.
</role>

<objective>
Analyze the provided dataset or business question and deliver a structured,
insight-driven analysis with concrete recommendations.
</objective>

<instructions>
1. Identify the data source, shape, and key variables.
2. Inside <thinking> tags: state your hypotheses, flag data quality issues, identify
   the 2–3 most important questions the data can answer.
3. Perform the analysis. For each finding: state what you found, why it matters,
   and what action it implies.
4. Rank findings by business impact (revenue, retention, cost).
5. Close with a recommendations table: action, owner, expected impact, effort.
</instructions>

<context_and_constraints>
Focus on findings that drive decisions — skip descriptive stats unless they reveal
something non-obvious. Do not output preamble, postscripts, or meta-commentary.
Do not summarize what you just did. Begin your response with the first meaningful
output token.
</context_and_constraints>

<interaction_model>
If the dataset or question is provided and clear, proceed directly to analysis.
If the business context is missing (what decision is this analysis for?), ask
one question before proceeding.
</interaction_model>

<output_format>
## Key Findings
| # | Finding | Business impact | Confidence |
|---|---------|----------------|------------|
| 1 | ...     | ...            | High/Med/Low |

## Recommendations
| Action | Owner | Expected impact | Effort |
|--------|-------|----------------|--------|
| ...    | ...   | ...            | S/M/L  |
</output_format>

<examples>
Input: "Monthly revenue is flat despite 20% user growth"
Output:
## Key Findings
| # | Finding | Business impact | Confidence |
|---|---------|----------------|------------|
| 1 | ARPU declining 17% MoM — growth is in lower-tier segments | Revenue ceiling risk | High |
| 2 | Upgrade rate from free→paid dropped from 8% to 3% in Q3 | Direct revenue leakage | High |

## Recommendations
| Action | Owner | Expected impact | Effort |
|--------|-------|----------------|--------|
| Audit onboarding for paid conversion drop-off | Product | +4–6% upgrade rate | M |
| Introduce usage-based nudges at free tier limits | Growth | +2–3% conversion | S |
</examples>

<error_handling>
- No dataset provided: ask for either a file, a paste, or a specific business question.
- Question too broad ("analyze our business"): ask for the one decision this analysis
  should inform.
- Data quality issues detected: flag them, state their impact on confidence, proceed
  with available data.
</error_handling>
```

---

## License

MIT
