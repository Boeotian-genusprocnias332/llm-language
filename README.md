<p align="center">
  <h1 align="center">llm-language</h1>
  <p align="center">
    <strong>The translation layer between human intent and machine execution</strong>
  </p>
  <p align="center">
    <em>Like the Rosetta Stone decoded hieroglyphs for the modern world,<br>
    llm-language decodes your intent for Claude — so you think less about how to ask<br>
    and more about what to build.</em>
  </p>
  <p align="center">
    <a href="#installation"><img alt="Claude Code Plugin" src="https://img.shields.io/badge/Claude_Code-Plugin-6B4FBB?style=flat-square&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZD0iTTEyIDJMMiAyMmgyMEwxMiAyeiIgZmlsbD0id2hpdGUiLz48L3N2Zz4="></a>
    <a href="LICENSE"><img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-green.svg?style=flat-square"></a>
    <img alt="Version" src="https://img.shields.io/badge/version-3.2.0-blue?style=flat-square">
    <img alt="Papers" src="https://img.shields.io/badge/papers-110+-orange?style=flat-square">
    <img alt="Dimensions" src="https://img.shields.io/badge/scoring_dims-8-red?style=flat-square">
    <img alt="Threshold" src="https://img.shields.io/badge/threshold-9.2%2F10-yellow?style=flat-square">
  </p>
  <p align="center">
    Proactive workflow intelligence with Jarvis mode, codebase-aware context engineering,<br>
    adaptive memory via ROSETTA.md, and 110+ scientific papers.<br>
    Your intent in, your project understood, maximum results out.
  </p>
</p>

---

<details>
<summary><strong>Table of Contents</strong></summary>

- [Why llm-language?](#why-llm-language)
- [Quick Start](#quick-start)
- [Architecture](#architecture)
- [Jarvis Mode](#jarvis-mode)
- [ROSETTA.md](#rosettamd--persistent-memory)
- [Scoring Rubric](#scoring-rubric)
- [Benchmark Results](#benchmark-results)
- [Sub-Skills](#sub-skills)
- [Scientific Foundation](#scientific-foundation)
- [Installation](#installation)
- [Usage](#usage)
- [File Structure](#file-structure)
- [Contributing](#contributing)
- [Citation](#citation)

</details>

---

## Why llm-language?

> *The Rosetta Stone (196 BC) bore the same decree in three scripts — hieroglyphic, demotic, and Greek — enabling scholars to finally decode a language that had been opaque for millennia. **ROSETTA.md** does the same for your intent: it translates between how you think (vague, contextual, implicit) and how Claude works best (structured, explicit, grounded in your actual project). The more you use it, the better the translation becomes.*

**This is not a prompt engineering tool.** Prompt engineering is a skill you learn. llm-language is an **intelligence layer** that sits between you and Claude Code, ensuring that every interaction leverages the full power of the model — your project context, your working patterns, your accumulated preferences — without you having to think about it.

You type 5 vague words. llm-language understands your project, your history, and your intent — then produces the structured, grounded execution plan that gets the best possible result.

```
Before:  "progetta un trading system"  (5 words, zero context)

After:   Structured XML prompt with:
         - 7 dependency-ordered sub-tasks
         - 10 domain-specific edge cases
         - 5 codebase invariants as hard constraints
         - 3 architecture alternatives (Tree of Thoughts)
         - Acceptance criteria, anti-patterns, risk assessment
         - All grounded in your ACTUAL project files
```

**Empirical results** (8 test cases, v3.0 evaluation):

| Metric | Baseline (no skill) | With llm-language | Delta |
|--------|:-------------------:|:-----------------:|:-----:|
| Task decomposition | 13% | 100% | **+87pp** |
| Acceptance criteria | 0% | 100% | **+100pp** |
| Edge case coverage | 25% | 100% | **+75pp** |
| Codebase grounding | 0% | 100% | **+100pp** |
| Mean quality score | 5.5/10 | 9.37/10 | **+64%** |

---

## Quick Start

```bash
# Install (one command)
claude plugins marketplace add https://github.com/Silence-view/llm-language
claude plugins install llm-language@llm-language --scope user

# Use
/llm-language            # Full pipeline on your next prompt
/llm-language:jarvis     # Proactive mode (learns your workflow)
/llm-language:update     # Check for new features & techniques
```

---

## Architecture

```
                         ┌─────────────────────────────────────┐
                         │          llm-language v3.2           │
                         └──────────────┬──────────────────────┘
                                        │
                    ┌───────────────────┬┴┬───────────────────┐
                    │                   │ │                    │
              ┌─────▼─────┐    ┌───────▼─▼──────┐    ┌───────▼───────┐
              │  ROSETTA   │    │ CODEBASE SCAN  │    │  CLAUDE.md    │
              │   Load     │    │  (mandatory)   │    │   Check       │
              │ Phase 0    │    │  Phase 0.5     │    │  Phase 0.6    │
              └─────┬──────┘    └───────┬────────┘    └───────┬───────┘
                    │                   │                      │
                    └───────────┬───────┘──────────────────────┘
                                │
                         ┌──────▼──────┐
                         │   INTAKE    │  Classify complexity
                         │  Phase 1    │  Discover skills
                         └──────┬──────┘  Assess uncertainty
                                │
                         ┌──────▼──────┐
                         │  PRODUCER   │  Opus agent + WebSearch
                         │  Phase 2    │  XML prompt generation
                         └──────┬──────┘  Codebase-grounded
                                │
                         ┌──────▼──────┐
                         │   CRITIC    │  8 dimensions
                    ┌────│  Phase 3    │  Anchor at 5
                    │    └──────┬──────┘  Threshold: 9.2
                    │           │
                    │    ┌──────▼──────┐
              < 9.2 │    │  Score ≥    │
              max 4 │    │   9.2?      │────── Yes ──┐
              rounds│    └─────────────┘              │
                    │                          ┌──────▼──────┐
                    └──── REVISE (Phase 4) ◄───│  EXECUTE    │
                                               │  Phase 5    │  ultrathink
                                               └──────┬──────┘
                                                      │
                                               ┌──────▼──────┐
                                               │   ROSETTA   │
                                               │   Update    │  Silent
                                               │  Phase 6    │  + Jarvis
                                               └─────────────┘  passive
```

| Phase | Name | What it does |
|:-----:|------|-------------|
| 0 | ROSETTA Load | Read persistent memory — preferences, effective patterns, anti-patterns |
| 0.5 | **Codebase Scan** | **MANDATORY.** Read CLAUDE.md + key files. Detect context mismatch. Build `<codebase-context>` |
| 0.6 | CLAUDE.md Check | Generate or update CLAUDE.md if missing/stale (WHY-WHAT-HOW structure) |
| 1 | Intake | Classify complexity, discover skills, assess uncertainty, select techniques |
| 2 | Generate | Producer Agent (Opus) creates XML prompt grounded in codebase + ROSETTA |
| 3 | Critique | Critic Agent scores on **8 dimensions**, anchor at **5**, threshold **9.2** |
| 4 | Revise | If score < 9.2: revise with feedback. Max **4 rounds**, delta < 0.2 stops |
| 5 | Execute | Summary banner + execute with **ultrathink**. Full tool + skill access |
| 6 | ROSETTA Update | Silent update with learnings + Jarvis passive workflow observation |

---

## Jarvis Mode

> **Jarvis learns ALWAYS, acts only when asked.**

Passive observation runs in every session (via Phase 6), building a personalized workflow model. Active mode (anticipation + execution) activates only with `/llm-language:jarvis`.

```
Phase 1: OBSERVATION          Phase 2: ANTICIPATION         Phase 3: AUTONOMOUS
(sessions 1-10)               (sessions 11-30)              (sessions 30+, opt-in)

  Watches silently               Proposes next step            Executes automatically
  Records task→next_action       Confidence ≥ 60%              Confidence ≥ 90%
  Builds pattern library         SUCCESS: +5%                  Safety rails enforced
  Zero interruption              MISS: -10%                    Never auto-commits
```

**Example anticipation:**
```
🔮 Jarvis anticipa:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Ho completato: implementazione modulo auth

Basandomi su 12 osservazioni, il prossimo passo piu' probabile e':
  → Eseguire i test (confidence: 85%)

Vuoi che proceda, o preferisci altro?
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Safety rails:** never auto-commits, never auto-deletes, never auto-pushes. `jarvis stop` to deactivate.

---

## ROSETTA.md — Persistent Memory

> *Named after the [Rosetta Stone](https://en.wikipedia.org/wiki/Rosetta_Stone) — the artifact that bridged the gap between human understanding and an otherwise opaque writing system. ROSETTA.md bridges the gap between your cognitive patterns and Claude's execution model. Each interaction adds another line to the translation — your preferences, your corrections, your workflows — until the system speaks your language natively.*

ROSETTA.md (`~/.claude/ROSETTA.md`) is a self-evolving document that captures what works for **you specifically**.

| Feature | How it works |
|---------|-------------|
| **Read** at start | Phase 0 loads your preferences, patterns, anti-patterns |
| **Updated** after every task | Phase 6 logs what worked, what didn't |
| **Cold-start bootstrap** | No history? Infers profile from CLAUDE.md + git log |
| **Jarvis integration** | § Jarvis Patterns tracks workflow sequences |
| **Auto-consolidation** | Merges similar patterns at 300 lines |
| **Scored by Critic** | Dimension 7 (User-Fit) + Dimension 8 (Codebase) |

Inspired by [PersonalLLM](https://openreview.net/forum?id=2R7498e2Tx) (ICLR 2025), [PromptWizard](https://microsoft.github.io/PromptWizard/) (Microsoft 2024), and Reflective Memory Management research.

---

## Scoring Rubric

The Critic evaluates every generated prompt on **8 dimensions**. Anchor at **5** (not 7) to prevent inflation — empirically validated (v2.0 showed 2.51-point inflation at anchor 7).

| # | Dimension | Weight | Anchors |
|:-:|-----------|:------:|---------|
| 1 | Intent Preservation | 0.18 | Does the optimized prompt preserve the user's original goal? |
| 2 | Precision | 0.16 | Is every instruction specific and unambiguous? |
| 3 | Completeness | 0.16 | Are all sub-tasks, edge cases, and acceptance criteria covered? |
| 4 | Structure | 0.12 | Is the XML well-formed with clear hierarchy? |
| 5 | Opus 4.6 Optimization | 0.08 | Are model-specific capabilities leveraged? |
| 6 | Scientific Grounding | 0.08 | Are appropriate techniques applied per task type? |
| 7 | User-Fit Alignment | 0.10 | Does the prompt leverage ROSETTA.md insights? |
| 8 | **Codebase Grounding** | **0.12** | **Does the prompt reference real project state?** |

**Pass threshold: 9.2/10.** Most prompts require 2-3 revision rounds.

---

## Benchmark Results

Evaluated across 8 test cases with deliberately vague Italian prompts for heavy tasks:

### v3.0 Results (threshold 9.2, 8 dimensions, anchor at 5)

| Test Case | Prompt | Score | Rounds |
|:---------:|--------|:-----:|:------:|
| TC-01 | "fammi un sistema di autenticazione" | 9.28 | 3 |
| TC-02 | "analizza questi dati e dimmi cosa trovi" | 9.60 | 3 |
| TC-03 | "scrivi l'introduzione del paper" | 9.35 | 4 |
| TC-04 | "questo codice non funziona, fixalo" | 9.27 | 4 |
| TC-05 | "progetta un trading system" | 9.40 | 4 |
| TC-06 | "pulisci questo codice" | 9.60 | 3 |
| TC-07 | "fammi un riassunto della letteratura" | 9.25 | 3 |
| TC-08 | "crea un dashboard per le performance" | 9.21 | 2 |

**Mean: 9.37** | **Min: 9.21** | **Max: 9.60** | **Pass rate: 8/8 (100%)**

### Score Evolution Across Versions

| Version | Threshold | Mean Score | Pass Rate | Avg Rounds |
|:-------:|:---------:|:----------:|:---------:|:----------:|
| v2.0 (self-assessed) | 8.5 | 8.94 | 100% | 0 |
| v2.0 (external triple-critique) | 8.5 | 6.43 | 0% | — |
| v2.0 + ROSETTA + codebase | 8.5 | 9.01 | 100% | 0 |
| **v3.0** | **9.2** | **9.37** | **100%** | **3.0** |

### Comparison with Literature

| Framework | Approach | Our Comparison |
|-----------|----------|---------------|
| [PromptWizard](https://microsoft.github.io/PromptWizard/) (Microsoft) | Generate-Critique-Revise, 45 tasks | Same architecture. We add codebase awareness + ROSETTA |
| [OPRO](https://arxiv.org/abs/2309.03409) (Google DeepMind) | LLM-as-optimizer, +8-50% on benchmarks | Our +64% quality uplift is in range |
| [DSPy](https://github.com/stanfordnlp/dspy) (Stanford) | Programmatic prompt compilation | Complementary — DSPy compiles modules, we optimize single prompts |
| [PEEM](https://arxiv.org/abs/2603.10477) | 9-axis joint evaluation, rho=0.97 | Our 8-dimension rubric is comparable in scope |

---

## Sub-Skills

| Skill | Command | Description |
|-------|---------|-------------|
| **llm-language** | `/llm-language` | Main pipeline — prompt meta-compilation with 8-phase cycle |
| **jarvis** | `/llm-language:jarvis` | Proactive assistant — observe, anticipate, act. Patterns learned from behavior |
| **update** | `/llm-language:update` | Self-evolution — searches for new Claude Code features and prompting papers |

---

## Scientific Foundation

Built on **110+ papers** across 8 categories. Full bibliography in [`docs/BIBLIOGRAPHY.md`](docs/BIBLIOGRAPHY.md).

<details>
<summary><strong>A. Reasoning Enhancement</strong></summary>

| Technique | Citation | Application |
|---|---|---|
| Chain-of-Thought | Wei et al., NeurIPS 2022 | Step-by-step decomposition in `<methodology>` |
| Zero-Shot CoT | Kojima et al., NeurIPS 2022 | "Think step by step" for simple tasks |
| Tree-of-Thought | Yao et al., NeurIPS 2023 | Multiple reasoning paths for ambiguous problems |
| Self-Consistency | Wang et al., ICLR 2023 | Multiple paths, majority vote for high-stakes |
| Least-to-Most | Zhou et al., ICLR 2023 | Simple-to-complex sub-task ordering |
</details>

<details>
<summary><strong>B. Self-Improvement & Reflection</strong></summary>

| Technique | Citation | Application |
|---|---|---|
| Self-Refine | Madaan et al., NeurIPS 2023 | Core Generate-Critique-Revise cycle |
| Reflexion | Shinn et al., NeurIPS 2023 | Learning from prior critique feedback |
| Constitutional AI | Bai et al., arXiv 2022 | Scoring rubric as "constitution" |
</details>

<details>
<summary><strong>C. Structural Optimization</strong></summary>

| Technique | Citation | Application |
|---|---|---|
| XML Structured Prompting | Xu et al., arXiv 2025; Anthropic 2025 | Entire output is XML-structured |
| Instruction Hierarchy | Wallace et al., 2024 | MUST/SHOULD/MAY priority ordering |
| Few-Shot Exemplars | Brown et al., NeurIPS 2020 | Thinking traces in examples |
</details>

<details>
<summary><strong>D. Meta-Techniques</strong></summary>

| Technique | Citation | Application |
|---|---|---|
| Meta-Prompting | Fernando et al., 2023; Suzgun & Kalai, 2024 | The skill IS meta-prompting |
| Automatic Prompt Engineering | Zhou et al., ICLR 2023 | Automated prompt optimization |
| Decomposed Prompting | Khot et al., ICLR 2023 | Modular sub-task delegation |
</details>

<details>
<summary><strong>E. Multi-Agent Techniques</strong></summary>

| Technique | Citation | Application |
|---|---|---|
| Multi-Agent Debate (MAD) | Du et al., ICML 2024 | Producer-Critic debate cycle |
| Diverse MAD (DMAD) | Liang et al., 2024 | Heterogeneous agent roles |
</details>

<details>
<summary><strong>F. Model-Specific (Claude Opus 4.6)</strong></summary>

| Technique | Citation | Application |
|---|---|---|
| Extended Thinking | Anthropic, 2025-2026 | Ultrathink with ~32K thinking tokens |
| Claude XML Best Practices | Anthropic, 2025 | Consistent tags, primacy/recency |
| Context Window Optimization | Anthropic, 2025 | 1M context exploitation |
</details>

<details>
<summary><strong>G. Adaptive Memory & User Modeling</strong></summary>

| Technique | Citation | Application |
|---|---|---|
| PersonalLLM | Hannah et al., ICLR 2025 | User preference learning via ROSETTA.md |
| PromptWizard | Agarwal et al., Microsoft 2024 | Self-evolving feedback-driven optimization |
| PROMST | Chen et al., EMNLP 2024 (Oral) | Human feedback for multi-step tasks |
| PLUM | ACL 2025 | Cross-session personalization |
| Reflective Memory Management | Survey 2025 | Adaptive memory granularity |
</details>

<details>
<summary><strong>H. Context Engineering</strong></summary>

| Technique | Citation | Application |
|---|---|---|
| Codified Context | arXiv:2602.20478, 2026 | 3-tier context infrastructure for Phase 0.5/0.6 |
| Agentic Context Engineering | arXiv:2510.04618, 2025 | Evolving playbooks (ROSETTA concept) |
| Context Engineering for Multi-Agent | arXiv:2508.08322, 2025 | Multi-agent context management |
</details>

### Key Design Principles

1. **General instructions > prescriptive steps** in ultrathink mode (Anthropic, 2026)
2. **DMAD > standard MAD** — heterogeneous roles outperform identical debaters (Liang et al., 2024)
3. **Codebase awareness is the #1 quality driver** — +2.5 points in empirical testing
4. **Score inflation is structural** — anchor at 5 reduces inflation by ~2.5 points (validated empirically)
5. **Personalization requires patience** — meaningful patterns emerge after 10+ interactions (PersonalLLM, ICLR 2025)

### Decision Matrix

| Task Type | Primary Technique | Secondary | Thinking |
|-----------|:-----------------:|:---------:|:--------:|
| Simple factual | Direct instruction | — | quick |
| Multi-step reasoning | CoT | Self-Consistency | extended |
| Ambiguous/creative | ToT | DMAD | ultrathink |
| Code generation | Decomposition | Self-Refine | extended |
| Architecture/design | ToT + Decomposition | Reflexion | ultrathink |
| Analysis/research | CoT + Retrieval-Aug | Calibration | ultrathink |
| Writing/editing | Role + Self-Refine | Few-shot | extended |
| Debugging | Reflexion + CoT | Adversarial | extended |
| Complex multi-part | Least-to-Most + DecomP | Meta-Prompting | ultrathink |

---

## Installation

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- Claude Opus 4.6 model access (Max, Team, or Enterprise plan for 1M context)

### Method 1: Marketplace (recommended)

```bash
claude plugins marketplace add https://github.com/Silence-view/llm-language
claude plugins install llm-language@llm-language --scope user
claude plugins list  # Verify: llm-language@llm-language v3.2.0
```

### Method 2: Local Clone (for development)

```bash
git clone https://github.com/Silence-view/llm-language.git ~/.claude/plugins/local/llm-language
claude plugins marketplace add ~/.claude/plugins/local/llm-language
claude plugins install llm-language@llm-language --scope user
```

### Update

```bash
claude plugins install llm-language@llm-language --scope user  # Re-install pulls latest
```

---

## Usage

### Commands

| Command | What it does |
|---------|-------------|
| `/llm-language` | Run full pipeline on your next prompt |
| `/llm-language:jarvis` | Activate proactive mode (observe → anticipate → act) |
| `/llm-language:update` | Search for new features, papers, and techniques |
| `skip llm-language` | Bypass the pipeline for one message |
| `jarvis stop` | Deactivate Jarvis active mode |
| `jarvis status` | Show Jarvis phase, patterns, confidence scores |

### Example Output

```
★ llm-language v3.2 ────────────────────────
Applied: ToT + Self-Refine | Role: Quant Systems Architect
Complexity: critical | Sub-tasks: 7 | Thinking: ultrathink
Score: 9.4/10 | Rounds: 3 | Threshold: 9.2
Codebase: grounded | Research: yes
Skills matched: paper, simplify | ROSETTA patterns: 5
──────────────────────────────────────────────────
```

### Token Cost

| Complexity | Overhead | Cost (Opus) | Time |
|:----------:|:--------:|:-----------:|:----:|
| simple | 8-15K | ~$0.20 | ~10s |
| moderate | 30-60K | ~$1.00 | ~30s |
| complex | 60-100K | ~$1.80 | ~60s |
| critical | 100-150K | ~$2.50 | ~90s |

---

## File Structure

```
llm-language/
├── .claude-plugin/
│   ├── plugin.json                  # Plugin manifest (v3.2.0)
│   └── marketplace.json             # Registration metadata
├── skills/
│   ├── llm-language/
│   │   ├── SKILL.md                 # Main pipeline orchestrator (v3.2)
│   │   └── references/
│   │       ├── scientific-principles.md   # 20+ techniques, 110+ papers
│   │       ├── xml-prompt-template.md     # XML template with field docs
│   │       ├── scoring-rubric.md          # 8-dimension rubric (v3.0)
│   │       └── rosetta-bootstrap.md       # ROSETTA.md bootstrap template
│   ├── jarvis/
│   │   └── SKILL.md                 # Proactive assistant (3-phase)
│   └── update/
│       └── SKILL.md                 # Self-evolution research agent
├── docs/
│   └── BIBLIOGRAPHY.md              # Full academic bibliography
├── README.md
└── LICENSE
```

---

## Contributing

Contributions welcome. Priority areas:

- **Jarvis pattern library** — workflow templates for common development patterns
- **Cross-model Critic** — using a different model as evaluator to reduce inflation
- **Task-accuracy benchmarks** — GSM8K/BBH comparison vs raw prompts
- **Domain-specific XML templates** — code review, academic writing, system design

---

## Citation

```bibtex
@software{llm_language_2026,
  title     = {llm-language: A Scientifically-Grounded Prompt Meta-Compiler
               with Proactive Workflow Anticipation for Claude Code},
  author    = {Andrea Nardello},
  year      = {2026},
  url       = {https://github.com/Silence-view/llm-language},
  version   = {3.2.0},
  note      = {Multi-agent prompt optimization with Jarvis proactive mode,
               mandatory codebase awareness, ROSETTA.md adaptive memory,
               and 8-dimension scoring rubric (threshold 9.2)},
  license   = {MIT}
}
```

---

## Acknowledgments

Built on foundational research by Wei et al. (CoT), Yao et al. (ToT), Madaan et al. (Self-Refine), Du et al. (MAD), Hannah et al. (PersonalLLM), Agarwal et al. (PromptWizard), and Anthropic's Claude prompting guidelines. Context engineering methodology from arXiv:2602.20478 (Codified Context). Full bibliography in [`docs/BIBLIOGRAPHY.md`](docs/BIBLIOGRAPHY.md).

<p align="center">
  <sub>Made with ultrathink by Andrea Nardello</sub>
</p>
