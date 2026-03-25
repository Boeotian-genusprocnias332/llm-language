---
name: llm-language
version: "3.0"
user-invocable: true
description: >
  Use when user explicitly invokes "/llm-language" or says "llm-language",
  "optimize prompt", "re-engineer prompt", "massima precisione", "maximum
  quality", "ottimizza il prompt". This skill re-engineers the user's
  prompt through multi-agent debate with scientific optimization,
  ROSETTA.md adaptive memory, mandatory codebase awareness, web deep
  research, and full skill access. Opus 4.6 + ultrathink. NOTE:
  ROSETTA.md is updated passively in the background to learn user style
  even when the skill is NOT invoked.
---

# llm-language v3.0

## Overview

A self-evolving prompt meta-compiler that re-engineers user messages through a multi-agent debate process grounded in 100+ scientific papers, producing optimized XML-structured prompts for Claude Opus 4.6 with ultrathink. The system maintains persistent memory via **ROSETTA.md** and **mandatory codebase awareness** — reading the actual project before generating prompts, ensuring every output is grounded in reality.

**Core cycle:** ROSETTA Load → **Codebase Scan** → Intake → Generate → Critique → Revise → Execute → ROSETTA Update

**New in v3.0:**
- **MANDATORY codebase awareness** — Producer reads CLAUDE.md + key files before generating (P0)
- **ROSETTA cold-start bootstrap** — infers user profile from CLAUDE.md + git log when no ROSETTA exists
- **Threshold raised to 9.2** — Critic must score ≥9.2 to accept (was 8.5)
- **4 revision rounds max** — more iterations to reach higher quality bar (was 2)
- **Critic anchor at 5** — starts at 5/10 and justifies upward (was 7), reducing inflation
- **Context mismatch detection** — flags when the request doesn't match the current project
- **File content awareness** — reads target file content, not just paths, to detect existing work
- Deep research via WebSearch, full skill access, ROSETTA.md adaptive memory (from v2.0)

## When to Use

**Only when explicitly invoked.** The full pipeline runs on:
- `/llm-language` command
- User says "llm-language", "optimize prompt", "massima precisione", "maximum quality"

**ROSETTA.md updates happen passively** — even when the skill is NOT invoked, ROSETTA.md should be updated at session boundaries to capture user style signals (language preference, communication patterns, domain context). This is handled by the ROSETTA Passive Update mechanism (see Phase 6b).

---

## Pipeline Execution (7 Phases)

### Phase 0: ROSETTA LOAD (inline, first thing)

**Before anything else**, read the ROSETTA.md file:

1. Check if `~/.claude/ROSETTA.md` exists
2. If it exists: read it fully — this is the user's preference profile, accumulated learnings, and effective patterns from all prior interactions
3. If it does NOT exist: **COLD-START BOOTSTRAP** — before creating from template:
   a. Read CLAUDE.md (if exists) to extract: project description, commands, architecture, conventions
   b. Run `git log --oneline -20` to extract: recent activity, author name, commit style
   c. Infer from conversation language: Italian/English/mixed preference
   d. Pre-populate ROSETTA.md user profile from these signals BEFORE creating from template
   e. Then create from `references/rosetta-bootstrap.md` with the pre-populated fields filled in
4. Extract from ROSETTA.md:
   - **User preferences** (communication style, preferred output formats, domain expertise)
   - **Effective patterns** (techniques that scored high, approaches the user responded well to)
   - **Anti-patterns** (things that failed or the user corrected)
   - **Domain context** (projects, tech stack, common themes)

This extracted context is passed to ALL subsequent agents as `<rosetta-context>`.

### Phase 0.5: CODEBASE SCAN (MANDATORY — inline, no subagent)

**This phase is NON-OPTIONAL. Skip it = skill failure.**

Scan the current project to build a `<codebase-context>` block:

1. **Read CLAUDE.md** (or AGENTS.md, GEMINI.md if present) — extract:
   - Project description and purpose
   - Architecture overview and module roles
   - Critical invariants and conventions
   - Test commands and build commands
   - Any explicit "do NOT" rules

2. **Read key project files** — use `Glob` to find entry points, then `Read` the first 50-80 lines of:
   - Main entry point (main.py, index.ts, etc.)
   - Config files (params.yaml, package.json, etc.)
   - Up to 3 most-recently-modified source files (from git status)

3. **Project structure inventory** — run `ls` or `Glob` on the root to understand directory layout

4. **Relevance check** — compare the user's request against the project context:
   - If the request is **RELEVANT** to the project: anchor all subsequent prompts to the codebase
   - If the request is **IRRELEVANT** (e.g., auth system for a backtest): flag with `<context-mismatch>` and offer clarification options
   - If **AMBIGUOUS**: use codebase to disambiguate (e.g., "questi dati" → the Parquet files in data/)

5. **File content check** — if the request targets a specific file/section that may already exist:
   - Read the target file to check if the content is already present
   - If content exists: frame the task as "revise/extend" not "create from scratch"

The `<codebase-context>` block is passed to ALL subsequent agents alongside `<rosetta-context>`.

### Phase 1: INTAKE (inline, no subagent)

**1.1 Capture Raw Input**

Store the user's exact message verbatim. This is IMMUTABLE.

**1.2 Discover Available Skills**

Run `Glob` on `~/.claude/skills/*/SKILL.md`, `~/.claude/agents/*.md`, and `~/.claude/plugins/*/skills/*/SKILL.md` to enumerate all available skills and agents. For each, extract `name` and `description`. Build a compact registry.

**CRITICAL: Exclude `llm-language` from the registry.**

**1.3 Classify Complexity**

- **simple** → single-step, clear output, no ambiguity
- **moderate** → multi-step, clear dependencies
- **complex** → multiple approaches, architecture decisions
- **critical** → high-stakes, irreversible, maximum depth

**1.4 Assess Uncertainty**

Before proceeding, evaluate: is the user's intent clear enough to generate a high-quality prompt? Check:
- Is the primary objective unambiguous?
- Are there critical missing details that would change the approach?
- Does ROSETTA.md context resolve the ambiguity?

**If uncertainty is HIGH and cannot be resolved by ROSETTA.md context:**

1. First, calculate HOW MANY questions you'd need for an excellent result (N). Consider:
   - Number of ambiguous aspects in the user's message
   - Number of critical decisions that depend on user preference
   - How much ROSETTA.md already covers
2. Use `AskUserQuestion` to present the user with a recommendation:
   - "Per un risultato ottimale servirebbero N domande di chiarimento. Quante vuoi che te ne faccia?"
   - Options: "Tutte (N) — massima precisione", "Solo le top 2-3 — bilanciato", "Zero — procedi con le tue assunzioni"
3. Ask ONLY the number of questions the user approved, ordered by impact (highest-impact first)
4. Do NOT ask obvious questions — only ask when the answer would materially change the generated prompt

**If uncertainty is LOW or ROSETTA.md resolves it:** proceed without interrupting the user. ROSETTA.md preferences have priority over asking — if the answer is already in ROSETTA, don't ask again.

**1.5 Select Scientific Principles**

Use the Quick Decision Matrix in `references/scientific-principles.md`. Cross-reference with ROSETTA.md effective patterns — if the user historically responds well to certain techniques, prefer those.

**1.6 Read XML Template**

Read `references/xml-prompt-template.md` for the Producer agent.

### Phase 2: GENERATE (Producer Subagent)

Dispatch a **general-purpose Agent** (model=opus) with FULL tool access:

```
You are an expert prompt engineer specializing in Claude Opus 4.6 optimization.
You have access to ALL tools: WebSearch, Read, Grep, Glob, Bash, and all
available skills. Use them when needed.

Your task: transform the user's raw input into a maximally optimized XML prompt.
You MUST ground the prompt in the actual codebase — never generate in a vacuum.

RAW USER INPUT:
<raw-input>
{user's exact message}
</raw-input>

USER PROFILE (from ROSETTA.md):
<rosetta-context>
{extracted ROSETTA.md context from Phase 0}
</rosetta-context>

PROJECT CONTEXT (from Phase 0.5 codebase scan):
<codebase-context>
{project description, architecture, invariants, key file contents, relevance assessment}
</codebase-context>

AVAILABLE SKILLS AND AGENTS:
<skill-registry>
{skill/agent registry from Phase 1}
</skill-registry>

COMPLEXITY LEVEL: {classified complexity}

SELECTED SCIENTIFIC PRINCIPLES:
<principles>
{selected principle IDs and brief descriptions}
</principles>

XML TEMPLATE TO FOLLOW:
<template>
{content of references/xml-prompt-template.md}
</template>

INSTRUCTIONS:
1. Analyze the raw input to extract the user's TRUE intent (not just surface request)
2. USE THE ROSETTA CONTEXT to align with known user preferences and effective patterns
3. If the topic requires domain knowledge you lack, USE WebSearch to research it
   before generating the prompt — deep research is encouraged for complex/critical tasks
4. Fill every section of the XML template with specific, precise content
5. Apply the selected scientific principles to the methodology section
6. Match skills/agents from the registry that are relevant to this task
7. Set thinking="ultrathink" in the root element
8. Ensure the <role> field assigns a specific expert persona matched to the domain
9. Decompose into sub-tasks ordered by dependency
10. Add edge cases specific to this task
11. Define concrete acceptance criteria
12. Specify anti-patterns to avoid (cross-reference ROSETTA anti-patterns)

RESEARCH PROTOCOL:
- For complex/critical tasks: WebSearch for domain-specific best practices BEFORE writing the prompt
- For moderate tasks: WebSearch only if the domain is unfamiliar
- For simple tasks: no research needed
- Always cite findings in the <context><background> section

OUTPUT: Return ONLY the complete XML prompt. No commentary, just the XML.

CRITICAL RULES:
- Preserve the user's original intent EXACTLY
- Be SPECIFIC — every instruction must be unambiguous
- Favor GENERAL instructions over prescriptive steps in methodology
- Include available skills in <tools-and-skills> only if genuinely relevant
- The XML prompt must be immediately executable — no placeholders or TODOs
- LEVERAGE the ROSETTA context — it contains what works for THIS user
- GROUND every claim in codebase-context — reference real files, real invariants, real conventions
- If the request is IRRELEVANT to the project, flag with <context-mismatch> and offer alternatives
- If the target content ALREADY EXISTS (e.g., intro already written), frame as revise/extend
- NEVER recommend technologies/scale beyond what the project actually uses
- Match architecture recommendations to the ACTUAL team size, budget, and infrastructure
```

### Phase 3: CRITIQUE (Critic Subagent)

Dispatch a **separate general-purpose Agent** (model=opus) with FULL tool access:

```
You are an independent prompt quality critic. You have access to ALL tools
including WebSearch — use them to verify claims or research standards.

ORIGINAL USER MESSAGE:
<raw-input>
{user's exact message}
</raw-input>

USER PROFILE (from ROSETTA.md):
<rosetta-context>
{extracted ROSETTA.md context}
</rosetta-context>

GENERATED XML PROMPT:
<generated-prompt>
{Producer's output from Phase 2}
</generated-prompt>

SCORING RUBRIC:
<rubric>
{content of references/scoring-rubric.md}
</rubric>

CODEBASE CONTEXT (from Phase 0.5):
<codebase-context>
{project description, architecture, invariants — use this to verify the prompt is grounded}
</codebase-context>

INSTRUCTIONS:
1. Score each of the 7 dimensions (1-10) using the anchored scale
2. For each score, cite SPECIFIC evidence from the generated prompt
3. For any score < 8, explain the issue and suggest a concrete fix
4. PAY SPECIAL ATTENTION to Dimension 7 (User-Fit Alignment):
   does the prompt leverage ROSETTA.md insights effectively?
5. **NEW — Dimension 8: Codebase Grounding (weight 0.12)**:
   does the prompt reference real files, real invariants, real project state?
   Does it match the project's actual scale (team, budget, tech stack)?
   Does it detect context mismatch or existing content?
6. Calculate the weighted score
7. Set verdict to "accept" if weighted score >= **9.2**, "revise" otherwise
8. If you identify a CRITICAL ambiguity that neither the prompt nor ROSETTA
   resolves, flag it with <needs-clarification>question</needs-clarification>

OUTPUT: Return evaluation in the XML format from the rubric.

ANTI-INFLATION RULES:
- **Start at 5 and justify UPWARD** — do not start at 7
- "Adequate" = 6. "Good" = 7. "Strong" = 8. "Excellent" = 9. 10 = literally perfect.
- EVERY score must cite specific content from the prompt
- Missing required sections = automatic 4 or below for structure
- Be genuinely critical — your job is to FIND WEAKNESSES, not validate
- Cross-check EVERY claim against codebase-context: if the prompt says "use kdb+"
  but the project uses Parquet files, that's a Codebase Grounding score of 3
- If the prompt over-engineers beyond the project's actual scale, penalize heavily
```

**If the Critic flags `<needs-clarification>`:** pause the pipeline and use `AskUserQuestion` to ask the user. Incorporate the answer, then re-run the Critic.

### Phase 4: REVISE + FINAL CRITIQUE

**If verdict is "accept" (score >= 9.2):** Skip to Phase 5.

**If verdict is "revise" (score < 9.2):**

Dispatch Producer again with critic feedback, ROSETTA context, codebase context, and full tool access. The revision prompt includes:
- The original message
- Previous XML prompt
- Critic feedback (ALL dimension scores + specific suggestions)
- ROSETTA context
- Codebase context
- Instruction: "Fix ONLY the dimensions scored below 9. Do not regress on dimensions already at 9+."

After revision, dispatch Critic one more time for scoring.

**Convergence rules:**
- Maximum **4** revision rounds total (was 2 in v2.0)
- Stop if weighted score >= **9.2**
- Stop if score improvement < 0.2 between rounds (tighter convergence check)
- If after 4 rounds the score is still < 9.2, proceed with the best-scoring version and log the gap in ROSETTA anti-patterns

### Phase 5: EXECUTE

**5.1 Summary Banner**

```
★ llm-language v3.0 ────────────────────────
Applied: {techniques} | Role: {persona}
Complexity: {level} | Sub-tasks: {N} | Thinking: ultrathink
Score: {score}/10 | Rounds: {N} | Threshold: 9.2
Codebase: {grounded/mismatch} | Research: {yes/no}
Skills matched: {list} | ROSETTA patterns: {count applied}
──────────────────────────────────────────────────
```

**5.2 Execute the Prompt**

Follow the XML prompt as your execution blueprint. Use ultrathink thinking mode. The executing agent has access to ALL tools and ALL skills — it can invoke any discovered skill, run any command, search the web, read files, write code.

**5.3 Ephemeral XML**

The generated XML prompt is NOT saved to any file. It lives only in conversation context.

### Phase 6: ROSETTA UPDATE (after execution)

**After the execution is complete**, update ROSETTA.md:

1. Read the current `~/.claude/ROSETTA.md`
2. Analyze the interaction that just occurred:
   - What complexity level was assigned?
   - What techniques were applied?
   - What was the final score?
   - Did the user seem satisfied? (infer from: did they correct the output? ask for changes? move on without complaint?)
   - Were there any clarification questions? What did the user answer?
   - Did the user's response reveal preferences not yet captured?
3. Update ROSETTA.md with new learnings:
   - Add effective patterns that scored >= 8.5
   - Add anti-patterns from corrections or low-scoring dimensions
   - Update user preference profile if new signals detected
   - Update domain context if working on a new project/topic
   - Increment the interaction counter
4. Keep ROSETTA.md under 300 lines — if approaching limit, consolidate older entries by merging similar patterns and removing outdated ones
5. NEVER delete the core structure (sections must persist)

**ROSETTA update is SILENT** — do not tell the user you are updating it. Just do it after every execution.

### Phase 6b: ROSETTA PASSIVE UPDATE (runs even WITHOUT skill invocation)

Even when llm-language is NOT explicitly invoked, the system should update ROSETTA.md to passively learn the user's style. This happens at natural session boundaries or when significant user signals are detected:

1. **Language preference**: detect from user messages (Italian, English, mixed)
2. **Communication style**: terse vs detailed, formal vs casual
3. **Skills frequently invoked**: track which `/skill-name` commands the user calls most often
4. **Agents frequently dispatched**: track which agents are used most
5. **Domain patterns**: what projects/topics recur across sessions
6. **Correction patterns**: when the user corrects Claude's output, capture what was wrong
7. **Tool preferences**: which tools the user asks to use (Bash, Read, WebSearch, etc.)

Update the relevant ROSETTA.md sections silently. This is lightweight — just a few lines appended to existing tables.

---

## ROSETTA.md System

**Location:** `~/.claude/ROSETTA.md`

**Purpose:** Persistent episodic memory that enables the skill to evolve and personalize over time. Inspired by PersonalLLM (ICLR 2025), PromptWizard (Microsoft 2024), and Reflective Memory Management research.

**Bootstrap:** On first invocation, ROSETTA.md is created from `references/rosetta-bootstrap.md`.

**Structure:** See `references/rosetta-bootstrap.md` for the canonical structure.

**Key principle:** ROSETTA is a LIVING document. It grows smarter with every interaction. After 10+ interactions, the system should noticeably better understand the user's preferences, communication style, and typical task patterns.

---

## Optimization for Simple Tasks

For **simple** tasks:
- Skip Phase 3 (Critique), Phase 4 (Revise) — but STILL read ROSETTA (Phase 0)
- Producer generates minimal XML (required fields only)
- Execute with `default` or `extended` thinking
- STILL update ROSETTA (Phase 6)

---

## Token Cost Estimate

| Complexity | Subagents | Research | Estimated Overhead |
|---|---|---|---|
| simple | 1 | no | ~8K-15K tokens |
| moderate | 2-3 | optional | ~30K-60K tokens |
| complex | 3-4 | yes | ~60K-100K tokens |
| critical | 4 | deep | ~100K-150K tokens |

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Changing user's intent | Critic dim 1 (Intent Preservation) catches this |
| Over-engineering simple requests | Complexity classification scales the pipeline |
| Prescriptive steps in ultrathink | Use general instructions — Claude exceeds hand-written plans |
| Saving prompt to disk | Never. Ephemeral only (XML prompt, not ROSETTA). |
| Ignoring ROSETTA context | Always pass rosetta-context to agents. Score via Dimension 7. |
| Not researching unfamiliar topics | Producer MUST WebSearch for complex/critical unfamiliar domains |
| Asking too many clarifying questions | Only ask when uncertainty is HIGH and ROSETTA can't resolve it |
| ROSETTA growing too large | Consolidate at 300 lines — merge similar, remove outdated |
| Recursive self-invocation | EXCLUDE llm-language from skill registry |
| Not updating ROSETTA after execution | Phase 6 is MANDATORY, even for simple tasks |

---

## Scientific Grounding

Built on 100+ papers. Core references documented in `references/scientific-principles.md`.

**Key v2.0 additions:**
- **PersonalLLM** (Hannah et al., ICLR 2025): Adapting LLMs to individual user preferences
- **PromptWizard** (Agarwal et al., Microsoft 2024): Self-evolving prompt optimization through feedback
- **PROMST** (Chen et al., EMNLP 2024): Human feedback integration for multi-step prompt optimization
- **PLUM** (ACL 2025): Cross-session personalization via conversation memory
- **Reflective Memory Management**: Adaptive granularity memory with RL-based relevance ranking

Full bibliography: `references/scientific-principles.md` § Source Bibliography.
