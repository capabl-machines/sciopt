# sciopt — Scientific Optimization Skill for Claude Code

A reusable [Claude Code](https://claude.ai/code) skill that runs autonomous optimization loops for any scientific problem with a quantitative scoring function.

Born from optimizing drug-like molecules (automol), battle-tested across 500K+ candidates, multi-agent parallel search, and mathematical ceiling proofs. Now generalized for any domain.

## What it does

`/sciopt /path/to/project 5` launches an autonomous optimization loop:

1. **Orients** — reads your project's CLAUDE.md, scoring function, and history
2. **Analyzes** — decomposes the scoring landscape using mathematical thinking models (Fourier, Fibonacci, entropy, Bayesian)
3. **Generates** — progresses from manual design → enumeration → 100K+ random search → database mining → multi-agent parallel search
4. **Evaluates** — runs your scoring function, logs results
5. **Iterates** — different strategy each round, commits with hypotheses

## Install

Copy the skill into your Claude Code skills directory:

```bash
# Personal (available in all projects)
cp -r sciopt ~/.claude/skills/sciopt

# Or project-specific
cp -r sciopt your-project/.claude/skills/sciopt
```

Then use it in Claude Code:

```
/sciopt /path/to/your/project 3
```

## Your project needs

The skill works with any project that has:

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Describes the problem, scoring function, current state |
| `results.tsv` | Tab-separated experiment log (commit, score, candidate, status, description) |
| A design file | The file the agent modifies (e.g., `design.py`, `config.yaml`) |
| An eval command | How to score candidates (e.g., `uv run design.py`, `python evaluate.py`) |

## Mathematical thinking models

The skill applies structured mathematical reasoning — not just brute force:

| Model | What it reveals | When to use |
|-------|----------------|-------------|
| **Fourier / Harmonic** | Theoretical ceiling, component peaks | Always — first thing to do |
| **Fibonacci / Golden Ratio** | Natural resonances in optimal values | When properties cluster near 1,2,3,5,8,13,21... |
| **Entropy / Information** | Most probable good candidate composition | When choosing what to generate |
| **Bayesian / Probabilistic** | Where to search next | When allocating compute across strategies |
| **Topological / Graph** | Landscape shape (smooth vs rugged) | When deciding hill-climb vs random |
| **Dimensional / Scaling** | Which properties actually matter | When too many knobs to turn |

See [thinking-models.md](thinking-models.md) for full details with worked examples.

## Strategy progression

The skill automatically escalates across rounds:

| Round | Strategy | Scale |
|-------|----------|-------|
| 1 | Manual domain-expert design | 5–10 candidates |
| 2 | Systematic enumeration | 100–1K |
| 3 | Random generation + mutation | 10K–500K |
| 4 | Database mining (ChEMBL, PubChem, etc.) | 100–1K real compounds |
| 5 | Multi-agent parallel search (SCOUT/MUTANT/MINER) | 500K+ total |
| 6 | Mathematical ceiling analysis | Proves optimality |

## Multi-agent architecture

For large search spaces, the skill deploys parallel agents in isolated git worktrees:

```
┌─────────────────────────────────────┐
│         ORCHESTRATOR (main)         │
│  Reads results, dispatches agents,  │
│  merges findings, commits best      │
└──────┬──────────┬───────────────────┘
       │          │           │
  ┌────▼───┐ ┌───▼────┐ ┌───▼────┐
  │ SCOUT  │ │ MUTANT │ │ MINER  │
  │ Random │ │ Mutate │ │ Query  │
  │ 500K+  │ │ best   │ │ real   │
  │ mols   │ │ seeds  │ │ data   │
  └────────┘ └────────┘ └────────┘
```

## Example: molecular optimization

```bash
# Setup
mkdir automol && cd automol
# ... create prepare.py (scoring), design.py (candidates), CLAUDE.md, results.tsv

# Run 5 rounds of optimization
/sciopt /Users/you/automol 5
```

**Results from automol** (drug-like molecule optimization):
- Started: benzamide, score 1.027
- Finished: THP-fluorophenyl-thiazole, score 0.0516
- 20x improvement across ~40 experiments
- Reached 99.997% of mathematically proven theoretical maximum
- Explored 500K+ molecules via multi-agent parallel search

## Example: antibiotic optimization

```bash
/sciopt /Users/you/automol-antibiotic 3
```

Different scoring (property ranges for Gram-negative penetration + similarity to known antibiotics + synthetic accessibility penalty), same skill.

## Example: ML hyperparameter tuning

```bash
/sciopt /Users/you/autoresearch 5
```

Works with any project that has a scoring function. The skill reads your CLAUDE.md to understand the domain.

## Key principles encoded

1. **Zero penalty first** — satisfy all constraints before optimizing the objective
2. **Scale matters** — 100K random candidates often beats 100 carefully designed ones
3. **Real data wins** — database mining finds decades of human optimization for free
4. **Prove the ceiling** — mathematical analysis prevents wasted effort on impossible improvements
5. **Diverse exploration** — different strategies find different solutions
6. **Document everything** — commit messages are your lab notebook

## Files

```
sciopt/
├── SKILL.md              # Main skill definition (the optimization loop)
├── thinking-models.md    # 6 mathematical reasoning frameworks
├── reference.md          # Domain strategies, API references, logging format
└── README.md             # This file
```

## License

MIT — use it however you want.
