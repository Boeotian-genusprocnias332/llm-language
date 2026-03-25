# Scoring Rubric — llm-language Critic Agent

## Overview

The Critic agent evaluates generated XML prompts on 7 dimensions. Each dimension is scored 1-10 using the anchored scale below. The weighted score determines whether revision is needed.

## Convergence Criteria

- **Pass threshold:** weighted score >= 8.5/10
- **Max revision rounds:** 2 (after initial generation)
- **Delta convergence:** stop if improvement < 0.3 between rounds
- **If score >= 8.5 on first pass:** skip directly to execution

## Scoring Dimensions

### 1. Intent Preservation (Weight: 0.25)

Does the optimized prompt preserve the user's original intent completely?

| Score | Anchor |
|---|---|
| 2 | Core intent changed or distorted. User would not recognize their request. |
| 4 | Intent partially preserved but important aspects missing or altered. |
| 6 | Intent preserved but some implicit needs not captured. |
| 8 | Intent preserved exactly, including implicit requirements inferred from context. |
| 10 | Intent + all implicit needs + edge cases anticipated. User's "ideal request" articulated better than they could. |

**Red flags:** Adding objectives the user didn't request. Changing the scope (narrowing or expanding without justification). Interpreting ambiguity without flagging the assumption.

---

### 2. Precision (Weight: 0.20)

Is every instruction specific and unambiguous?

| Score | Anchor |
|---|---|
| 2 | Vague, abstract instructions. "Make it good." Multiple interpretations possible. |
| 4 | Some specific instructions mixed with vague ones. |
| 6 | Most instructions are specific. 1-2 ambiguous areas remain. |
| 8 | All instructions are specific and unambiguous. Every word carries meaning. |
| 10 | Every instruction is load-bearing. Zero ambiguity. Constraints are tight but not over-constrained. |

**Red flags:** Adjectives without concrete definition ("clean code", "good performance"). Instructions that could be interpreted multiple ways. Missing output format specification.

---

### 3. Completeness (Weight: 0.20)

Are all sub-tasks addressed? Are edge cases covered?

| Score | Anchor |
|---|---|
| 2 | Major components of the task missing. Only surface-level coverage. |
| 4 | Core task addressed but no sub-task decomposition. No edge cases. |
| 6 | Sub-tasks identified and addressed. 1-2 edge cases noted. |
| 8 | All sub-tasks decomposed and sequenced. Key edge cases covered. Acceptance criteria defined. |
| 10 | Comprehensive decomposition with dependencies. Edge cases + anti-patterns + acceptance criteria + verification strategy. |

**Red flags:** Missing acceptance criteria for complex tasks. No edge case consideration for critical tasks. Sub-tasks not ordered by dependency.

---

### 4. Structure (Weight: 0.15)

Is the XML well-formed? Is the hierarchy clear and semantic?

| Score | Anchor |
|---|---|
| 2 | Malformed XML. Flat structure without meaningful hierarchy. |
| 4 | Valid XML but poor tag naming or flat organization. |
| 6 | Well-formed XML with correct section structure. Some tags could be more semantic. |
| 8 | Clean XML with semantic tags, proper nesting, and clear hierarchy. Follows the template exactly. |
| 10 | Optimal XML structure. Perfect use of attributes. Content placement exploits primacy/recency effects. Easy to parse and modify. |

**Red flags:** Missing required sections (meta, role, task, methodology). Incorrect nesting. Non-semantic tag names. Content in wrong section.

---

### 5. Opus 4.6 Optimization (Weight: 0.10)

Does the prompt leverage Opus 4.6's specific capabilities?

| Score | Anchor |
|---|---|
| 2 | Generic prompt that would work on any model. No model-specific optimization. |
| 4 | Thinking level set but not matched to complexity. |
| 6 | Correct thinking level. Some tool awareness. |
| 8 | Thinking level matched to complexity. Tool guidance included. Extended context exploited. General instructions favored over prescriptive steps. |
| 10 | Full model-specific optimization: thinking level, tool guidance, context window management, primacy/recency placement, thinking trace examples if few-shot. |

**Red flags:** Ultrathink for simple tasks (wastes tokens). Prescriptive step-by-step for ultrathink (limits Claude's reasoning). Missing tool guidance when tools are relevant.

---

### 6. Scientific Grounding (Weight: 0.10)

Are appropriate prompt engineering techniques applied to the task type?

| Score | Anchor |
|---|---|
| 2 | No techniques applied. Raw instruction dump. |
| 4 | CoT added generically regardless of task type. |
| 6 | 1-2 appropriate techniques selected and applied. |
| 8 | Techniques matched to task type using the Decision Matrix. Multiple complementary techniques combined. |
| 10 | Optimal technique selection. Techniques interact synergistically. Novel technique combination justified by task requirements. |

**Red flags:** ToT for simple tasks (overkill). Missing CoT for multi-step reasoning. Self-Consistency not used for high-stakes decisions. Role prompting for factual accuracy (doesn't work — use for style/format only).

---

### 7. User-Fit Alignment (Weight: 0.10) — NEW in v2.0

Does the prompt leverage ROSETTA.md insights to align with this specific user's preferences?

| Score | Anchor |
|---|---|
| 2 | ROSETTA.md context completely ignored. Generic prompt with no personalization. |
| 4 | ROSETTA.md was referenced but insights not applied. |
| 6 | Some ROSETTA patterns applied. User preferences partially reflected. |
| 8 | Key ROSETTA patterns applied. Output format, style, and approach match known user preferences. Prior effective techniques reused where applicable. |
| 10 | Deep ROSETTA integration. Preferences, anti-patterns, domain context, and prior effective techniques all reflected. Prompt anticipates user needs based on accumulated history. |

**Red flags:** Using techniques ROSETTA lists as anti-patterns. Ignoring known output format preferences. Not leveraging domain context from ROSETTA when the topic matches. If ROSETTA.md is empty (new user), score 7 by default — no penalty for lack of history.

**Special case:** If ROSETTA.md does not exist or has zero interactions, default score is 7 (neutral). The dimension becomes meaningful after 3+ interactions.

---

### 8. Codebase Grounding (Weight: 0.12) — NEW in v3.0

Does the prompt reference the ACTUAL project state, not an imagined one?

| Score | Anchor |
|---|---|
| 2 | Prompt ignores the codebase entirely. Recommends technologies/scale incompatible with the project. |
| 4 | Prompt mentions the project domain but does not reference specific files, invariants, or conventions. |
| 6 | Some codebase references present but superficial. Scale mostly appropriate but with mismatches. |
| 8 | Prompt references real files, real invariants, real conventions. Scale matches actual team/budget/tech. Existing content detected and acknowledged. |
| 10 | Deep codebase integration. Every recommendation grounded in actual project state. Context mismatches flagged. Existing work acknowledged and built upon. Technology choices match exactly what's installed/used. |

**Red flags:** Recommending technologies not in the project's stack. Ignoring existing implementations. Treating a working system as greenfield. Over-engineering beyond the project's actual scale. Not detecting that target content already exists.

**Special case:** If no codebase context is available (no CLAUDE.md, no git repo), default score is 6 (neutral). The dimension becomes meaningful when a project context exists.

---

## Weighted Score Calculation

```
weighted_score = (intent * 0.18) + (precision * 0.16) + (completeness * 0.16)
               + (structure * 0.12) + (opus_opt * 0.08) + (scientific * 0.08)
               + (user_fit * 0.10) + (codebase_grounding * 0.12)
```

**Weight changes from v2.0:** Intent 0.20→0.18, Precision 0.18→0.16, Completeness 0.18→0.16, Structure 0.14→0.12, Opus 0.10→0.08, Scientific 0.10→0.08. This creates room for Codebase Grounding (0.12) — the most impactful new dimension based on empirical evaluation data (contributed +2.5 points in Round 3 testing). Total = 1.00.

**Pass threshold: 9.2/10** (raised from 8.5 in v2.0, based on evaluation showing 8.5 was too easily met).

## Critic Output Format

The Critic agent MUST output its evaluation in this exact structure:

```xml
<critique>
  <scores>
    <dimension name="intent-preservation" score="8" weight="0.25">
      Evidence: [specific evidence from the prompt]
      Issue: [if score < 8, what's wrong]
    </dimension>
    <dimension name="precision" score="7" weight="0.20">
      Evidence: [...]
      Issue: [...]
    </dimension>
    <dimension name="completeness" score="9" weight="0.20">
      Evidence: [...]
    </dimension>
    <dimension name="structure" score="8" weight="0.15">
      Evidence: [...]
    </dimension>
    <dimension name="opus-optimization" score="7" weight="0.10">
      Evidence: [...]
      Issue: [...]
    </dimension>
    <dimension name="scientific-grounding" score="8" weight="0.10">
      Evidence: [...]
    </dimension>
    <dimension name="user-fit-alignment" score="7" weight="0.10">
      Evidence: [ROSETTA patterns applied or not]
      Issue: [if score < 8, what ROSETTA insights were missed]
    </dimension>
    <dimension name="codebase-grounding" score="8" weight="0.12">
      Evidence: [real files referenced, invariants preserved, scale match, existing content detected]
      Issue: [if score < 8, what codebase context was missed or misrepresented]
    </dimension>
  </scores>

  <needs-clarification>
    <!-- Only if a CRITICAL ambiguity exists that neither prompt nor ROSETTA resolves -->
  </needs-clarification>

  <weighted-score>7.85</weighted-score>
  <verdict>revise|accept</verdict>

  <suggestions priority="high">
    <suggestion dimension="precision">
      [Specific, actionable suggestion for improvement]
    </suggestion>
    <suggestion dimension="opus-optimization">
      [Specific, actionable suggestion for improvement]
    </suggestion>
  </suggestions>
</critique>
```

## Anti-Inflation Rules (v3.0 — STRICT)

To prevent score inflation (empirical finding: v2.0 self-assessment inflated by 2.51 points):

1. **Start at 5, justify UPWARD:** Do NOT start at 7. 5 is mediocre. Justify every point above 5.
2. **Anchor to examples:** Always compare against the rubric anchors, not gut feeling
3. **Require evidence:** Every score must cite specific content from the prompt
4. **Penalize missing sections:** Absent required sections = automatic 4 or below for structure
5. **Check Decision Matrix:** If techniques don't match the task type from the Quick Decision Matrix in scientific-principles.md, score scientific-grounding accordingly
6. **Cross-check codebase:** If the prompt recommends X but the project uses Y, score codebase-grounding accordingly. Over-engineering beyond actual project scale = automatic 5 or below.
7. **"Adequate" = 6, "Good" = 7:** Reserve 8+ for genuinely strong work. 9 = near-perfect. 10 = cannot find a single improvement.
8. **Pass threshold is 9.2:** This is deliberately hard to reach. Most prompts should require 1-2 revision rounds.
