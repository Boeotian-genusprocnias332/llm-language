# llm-language

**A scientifically-grounded prompt meta-compiler for Claude Code (v3.2 -- Jarvis proactive mode + mandatory codebase awareness)**

> Automatic prompt re-engineering through multi-agent debate, proactive workflow anticipation via Jarvis mode, and mandatory codebase awareness — grounded in 100+ prompt engineering papers, specialized for Claude Opus 4.6 with ultrathink.

---

## Overview

`llm-language` is a [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that intercepts every user message and re-engineers it through a multi-agent Generate-Critique-Revise pipeline before execution. The system produces an optimized XML-structured prompt that is ephemeral (never saved to disk) and executed with maximum reasoning depth.

The pipeline implements a **Diverse Multi-Agent Debate** (DMAD) architecture where a Producer agent generates an optimized prompt and an independent Critic agent evaluates it against an 8-dimension scoring rubric. If the score falls below 9.2/10, the prompt is revised and re-evaluated, up to 4 rounds.

**v3.0** introduces **mandatory codebase awareness** — the pipeline reads CLAUDE.md and key project files before generating any prompt, ensuring every output is grounded in the real project state. ROSETTA.md now supports **cold-start bootstrap**, inferring a user profile from CLAUDE.md and git log when no prior history exists. Agents have access to **all user skills**, **web deep research**, and can **ask the user for clarification** when uncertain.

### Key Features

- **Jarvis proactive mode** (`/llm-language:jarvis`): 3-phase assistant that observes user workflows (sessions 1-10), anticipates next steps (11-30), and acts autonomously (30+, opt-in). Patterns are LEARNED from behavior, never hardcoded.
- **Skill auto-invocation**: Jarvis invokes /brainstorming before creative tasks, /code-review after implementation, /paper after LaTeX edits — adapting to each user's actual workflow
- **Workflow pattern learning**: ROSETTA.md § Jarvis Patterns tracks post-task sequences, building a personalized workflow model with confidence scoring
- **Mandatory codebase awareness**: reads CLAUDE.md + key files before generating (Phase 0.5)
- **Context mismatch detection**: flags when request doesn't match project
- **File content awareness**: reads target files to detect existing work
- **ROSETTA cold-start bootstrap**: infers profile from CLAUDE.md + git log when no prior history exists
- **8-dimension scoring rubric**: added Codebase Grounding (0.12 weight)
- **Threshold 9.2**: raised from 8.5 based on empirical evaluation
- **Critic anchor at 5**: starts at 5/10 (was 7), reduces inflation by ~2.5 points
- **ROSETTA.md persistent memory**: self-evolving user profile that improves output quality over time
- **Automatic interception** of every user message (skippable with "skip llm-language")
- **Multi-agent debate**: Producer (generator) + Critic (evaluator) with heterogeneous roles
- **Deep web research**: agents autonomously research unfamiliar domains before generating prompts
- **Full skill access**: agents can use ALL installed skills, tools, and capabilities
- **User clarification**: when uncertain, agents ask targeted questions instead of guessing
- **Scientific grounding**: 20+ techniques from 110+ papers mapped to task types
- **XML-structured output**: leverages Claude's native XML parsing capabilities
- **Complexity-adaptive**: scales pipeline depth (simple=1 agent, critical=4 agents)
- **Ephemeral prompts**: generated XML lives only in conversation context
- **Opus 4.6 optimized**: ultrathink activation, extended context exploitation

## Architecture

```
User Message
    |
    v
[Phase 0: ROSETTA LOAD]   Read ~/.claude/ROSETTA.md — user preferences,
    |                       effective patterns, anti-patterns, domain context
    v
[Phase 0.5: CODEBASE SCAN] Read CLAUDE.md + key files, check relevance,
    |                        detect existing content, build codebase-context
    v
[Phase 1: INTAKE]          Classify complexity, discover skills,
    |                       assess uncertainty, select principles
    |                       Ask user if uncertainty is HIGH
    v
[Phase 2: GENERATE]        Producer Agent (Opus) with ALL tools + WebSearch
    |                       generates XML prompt using template + ROSETTA context
    v
[Phase 3: CRITIQUE]        Critic Agent (Opus) scores on 8 dimensions
    |                       including User-Fit Alignment (ROSETTA)
    v
[Phase 4: REVISE]          If score < 9.2: revise + re-critique
    |                       Max 4 rounds, delta convergence < 0.3
    v
[Phase 5: EXECUTE]         Summary banner + execute with ultrathink
    |                       Full tool access + all skills available
    v
[Phase 6: ROSETTA UPDATE]  Update ROSETTA.md with learnings from this
                            interaction — effective patterns, user signals
```

### Jarvis Mode (Proactive Layer)

When activated via `/llm-language:jarvis`, a proactive layer wraps around the standard pipeline:

```
[Standard Pipeline] → Task Complete
    |
    v
[Observe] Record what user does next (ROSETTA § Jarvis Patterns)
    |
    v
[Anticipate] If pattern confidence ≥ 60%: propose next step
    |
    v
[Act] If confidence ≥ 90% AND autonomous mode: execute without asking
```

**Three phases:**
- **Observation** (sessions 1-10): watches and records, never proposes
- **Anticipation** (sessions 11-30): proposes next steps, learns from responses
- **Autonomous** (sessions 30+, opt-in): executes high-confidence patterns automatically

Safety rails: never auto-commits, never auto-deletes, never auto-pushes. User can say `jarvis stop` at any time.

### ROSETTA.md — Persistent Memory

ROSETTA.md (`~/.claude/ROSETTA.md`) is a self-evolving document that captures what works for each specific user. It is:

- **Read at the start** of every pipeline invocation (Phase 0)
- **Updated silently** after every execution (Phase 6)
- **Passed to all agents** as `<rosetta-context>` for personalized prompt generation
- **Scored by the Critic** via Dimension 7 (User-Fit Alignment) and Dimension 8 (Codebase Grounding)
- **Cold-start bootstrap**: when no ROSETTA.md exists, the system infers an initial profile from CLAUDE.md project instructions and recent git log activity, seeding preferences before any user interaction
- **Consolidated automatically** when approaching 300 lines (merges similar patterns, removes outdated entries)

After 10+ interactions, the system noticeably adapts to the user's preferences, communication style, and typical task patterns. Inspired by PersonalLLM (ICLR 2025), PromptWizard (Microsoft 2024), and Reflective Memory Management research.

### Scoring Rubric

| Dimension | Weight | What it measures |
|---|---|---|
| Intent Preservation | 0.18 | User's original goal unchanged |
| Precision | 0.16 | Every instruction specific, unambiguous |
| Completeness | 0.16 | Sub-tasks + edge cases covered |
| Structure | 0.12 | XML well-formed, clear hierarchy |
| Opus 4.6 Optimization | 0.08 | Model-specific capabilities leveraged |
| Scientific Grounding | 0.08 | Appropriate techniques applied |
| User-Fit Alignment | 0.10 | ROSETTA.md insights leveraged |
| **Codebase Grounding** | **0.12** | **Real project state referenced** |

### Token Cost

| Complexity | Subagents | Research | Estimated Overhead |
|---|---|---|---|
| simple | 1 | no | ~8K-15K tokens |
| moderate | 2-3 | optional | ~30K-60K tokens |
| complex | 3-4 | yes | ~60K-100K tokens |
| critical | 4 | deep | ~100K-150K tokens |

## Installation

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- Claude Opus 4.6 model access

### Method 1: Install via Marketplace (recommended)

This registers the GitHub repository as a Claude Code marketplace, making it installable with a single command.

```bash
# Step 1: Add the GitHub repository as a marketplace source
claude plugins marketplace add https://github.com/Silence-view/llm-language

# Step 2: Install the plugin globally (user scope = works in any directory)
claude plugins install llm-language@llm-language --scope user

# Step 3: Verify installation
claude plugins list
# Expected output:
#   llm-language@llm-language
#   Version: 1.0.0
#   Scope: user
#   Status: enabled
```

To update to a newer version later:
```bash
claude plugins install llm-language@llm-language --scope user
```

To uninstall:
```bash
claude plugins uninstall llm-language@llm-language
claude plugins marketplace remove llm-language
```

### Method 2: Install from Local Clone

If you want to modify the skill or develop locally:

```bash
# Step 1: Clone the repository
git clone https://github.com/Silence-view/llm-language.git ~/.claude/plugins/local/llm-language

# Step 2: Register the local folder as a marketplace
claude plugins marketplace add ~/.claude/plugins/local/llm-language

# Step 3: Install at user scope
claude plugins install llm-language@llm-language --scope user

# Step 4: Verify
claude plugins list
```

With a local clone, any changes you make to the skill files take effect immediately.

### Method 3: Install as Standalone Skill (minimal)

If you don't need the full plugin infrastructure:

```bash
# Copy just the skill directory
git clone https://github.com/Silence-view/llm-language.git /tmp/sl-temp
cp -r /tmp/sl-temp/skills/llm-language ~/.claude/skills/llm-language
rm -rf /tmp/sl-temp
```

> **Note:** Standalone skills only work when Claude Code loads the `~/.claude/skills/` directory. The plugin method (Methods 1-2) is more reliable and works globally.

## Usage

### Automatic Mode (default)

Once installed, the skill triggers automatically on every user message. You'll see a summary banner before each response:

```
★ llm-language v3.0 ────────────────────────
Applied: CoT + Self-Refine | Role: Senior Backend Engineer
Complexity: moderate | Sub-tasks: 3 | Thinking: ultrathink
Score: 9.3/10 | Rounds: 2 | Threshold: 9.2
Codebase: grounded | Research: no
Skills matched: paper, humanizer | ROSETTA patterns: 3
──────────────────────────────────────────────────
```

### Manual Invocation

```
/llm-language
```

### Skip for a Message

Include "skip llm-language" or "raw mode" in your message to bypass the pipeline.

## File Structure

```
llm-language/
├── .claude-plugin/
│   ├── plugin.json              # Plugin identity
│   └── marketplace.json         # Registration metadata
├── skills/
│   └── llm-language/
│       ├── SKILL.md             # Main skill (pipeline orchestrator, v3.2)
│       ├── jarvis/
│       │   └── SKILL.md             # Proactive assistant (v3.2, 3-phase)
│       └── references/
│           ├── scientific-principles.md   # 20+ techniques, 110+ papers
│           ├── xml-prompt-template.md     # XML template with field docs
│           ├── scoring-rubric.md          # 8-dimension quality rubric (v3.0)
│           └── rosetta-bootstrap.md       # ROSETTA.md initial template (v3.0 with cold-start)
├── docs/
│   └── BIBLIOGRAPHY.md          # Full academic bibliography
├── README.md
└── LICENSE
```

## Scientific Foundation

The system synthesizes findings from the following research areas. For the full bibliography with 100+ papers, see [`docs/BIBLIOGRAPHY.md`](docs/BIBLIOGRAPHY.md).

### Core Techniques Implemented

#### A. Reasoning Enhancement

| Technique | Citation | Application in llm-language |
|---|---|---|
| Chain-of-Thought (CoT) | Wei et al., NeurIPS 2022 | Step-by-step decomposition in `<methodology>` |
| Zero-Shot CoT | Kojima et al., NeurIPS 2022 | "Think step by step" triggers for simple tasks |
| Tree-of-Thought (ToT) | Yao et al., NeurIPS 2023 | Multiple reasoning paths for ambiguous problems |
| Self-Consistency | Wang et al., ICLR 2023 | Multiple paths, majority vote for high-stakes tasks |
| Least-to-Most | Zhou et al., ICLR 2023 | Simple-to-complex sub-task ordering |

#### B. Self-Improvement & Reflection

| Technique | Citation | Application in llm-language |
|---|---|---|
| Self-Refine | Madaan et al., NeurIPS 2023 | Core Generate-Critique-Revise cycle |
| Reflexion | Shinn et al., NeurIPS 2023 | Learning from prior critique feedback |
| Constitutional AI | Bai et al., arXiv 2022 | Scoring rubric as "constitution" |

#### C. Structural Optimization

| Technique | Citation | Application in llm-language |
|---|---|---|
| XML Structured Prompting | Xu et al., arXiv 2025; Anthropic 2025 | Entire output is XML-structured |
| Instruction Hierarchy | Wallace et al., 2024 | MUST/SHOULD/MAY priority ordering |
| Few-Shot Exemplars | Brown et al., NeurIPS 2020 | Thinking traces in examples |

#### D. Meta-Techniques

| Technique | Citation | Application in llm-language |
|---|---|---|
| Meta-Prompting | Fernando et al., 2023; Suzgun & Kalai, 2024 | The skill IS meta-prompting |
| Automatic Prompt Engineering | Zhou et al., ICLR 2023 | Automated prompt optimization |
| Decomposed Prompting | Khot et al., ICLR 2023 | Modular sub-task delegation |

#### E. Multi-Agent Techniques

| Technique | Citation | Application in llm-language |
|---|---|---|
| Multi-Agent Debate (MAD) | Du et al., ICML 2024 | Producer-Critic debate cycle |
| Diverse MAD (DMAD) | Liang et al., 2024 | Heterogeneous agent roles |

#### F. Model-Specific (Claude Opus 4.6)

| Technique | Citation | Application in llm-language |
|---|---|---|
| Extended Thinking | Anthropic, 2025-2026 | Ultrathink with ~32K thinking tokens |
| Claude XML Best Practices | Anthropic, 2025 | Consistent tags, primacy/recency |
| Context Window Optimization | Anthropic, 2025 | 1M context exploitation |

#### G. Adaptive Memory & User Modeling (v2.0)

| Technique | Citation | Application in llm-language |
|---|---|---|
| PersonalLLM | Hannah et al., ICLR 2025 | User preference learning via ROSETTA.md |
| PromptWizard | Agarwal et al., Microsoft 2024 | Self-evolving feedback-driven optimization |
| PROMST | Chen et al., EMNLP 2024 (Oral) | Human feedback integration for multi-step tasks |
| PLUM | ACL 2025 | Cross-session personalization via conversation memory |
| Reflective Memory Management | Survey 2025 | Adaptive memory granularity with consolidation |
| Nemori | 2025 | Self-organizing episodic memory |

### Key Findings Informing Design

1. **General instructions outperform prescriptive steps** in extended thinking mode (Anthropic, 2026). The Producer generates high-level methodology rather than step-by-step plans.

2. **DMAD consistently outperforms standard MAD** (Liang et al., 2024). We use heterogeneous roles (Producer vs Critic) rather than identical debaters.

3. **Role prompting improves style/format but not factual accuracy** (Meincke et al., 2025; multiple studies). The `<role>` field is used for output alignment, not knowledge claims.

4. **XML reduces hallucination** in structured outputs (Xu et al., 2025; Anthropic, 2025). The entire prompt is XML-structured.

5. **Score inflation is a known failure mode** in multi-agent evaluation (Smit et al., 2024). The rubric uses anti-inflation rules: start at 5, justify upward -- empirically validated (v2.0 showed 2.51-point inflation at anchor 7).

6. **Personalized models provide maximal benefits as interactions accumulate** (PersonalLLM, ICLR 2025). ROSETTA.md enables this by persisting preference signals across sessions.

7. **Self-evolving feedback loops achieve superior performance across 45 tasks** (PromptWizard, Microsoft 2024). Our Generate-Critique-Revise cycle with ROSETTA persistence extends this pattern.

8. **Human feedback integration improves multi-step task performance by 10-29%** (PROMST, EMNLP 2024). The user clarification mechanism in Phase 1.4 implements this.

9. **Codebase awareness is the single largest quality driver** (+2.5 points in empirical testing). v3.0 makes it mandatory via Phase 0.5.

## Decision Matrix

Used in Phase 1.4 to select appropriate scientific techniques:

| Task Type | Primary Technique | Secondary | Thinking Level |
|---|---|---|---|
| Simple factual | Direct instruction | -- | quick/default |
| Multi-step reasoning | CoT | Self-Consistency | extended |
| Ambiguous/creative | ToT | DMAD | ultrathink |
| Code generation | Decomposition | Self-Refine | extended |
| Architecture/design | ToT + Decomposition | Reflexion | ultrathink |
| Analysis/research | CoT + Retrieval-Aug | Calibration | ultrathink |
| Writing/editing | Role + Self-Refine | Few-shot | extended |
| Debugging | Reflexion + CoT | Adversarial | extended |
| Complex multi-part | Least-to-Most + DecomP | Meta-Prompting | ultrathink |

## Contributing

Contributions are welcome. Areas of interest:

- **New scoring dimensions** for the rubric
- **Technique-specific optimizations** for different Claude models
- **Benchmark results** comparing llm-language outputs vs raw prompts
- **Domain-specific XML templates** (e.g., for code review, academic writing)

## License

MIT License. See [LICENSE](LICENSE).

## Citation

If you use this work in academic research, please cite:

```bibtex
@software{llm_language_2026,
  title     = {llm-language: A Scientifically-Grounded Prompt Meta-Compiler for Claude Code},
  author    = {Andre},
  year      = {2026},
  url       = {https://github.com/Silence-view/llm-language},
  note      = {v3.0: Multi-agent prompt optimization with mandatory codebase awareness,
               DMAD, Self-Refine, and XML-structured prompting for Claude Opus 4.6},
  license   = {MIT}
}
```

## Acknowledgments

This work builds on the foundational research of Wei et al. (Chain-of-Thought), Yao et al. (Tree-of-Thought), Madaan et al. (Self-Refine), Du et al. (Multi-Agent Debate), and Anthropic's Claude prompting guidelines. Full bibliography in [`docs/BIBLIOGRAPHY.md`](docs/BIBLIOGRAPHY.md).
