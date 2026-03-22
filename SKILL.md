---
name: sciopt
description: Run autonomous scientific optimization loops. Use when iterating on candidate designs (molecules, hyperparameters, architectures, materials) against a quantitative scoring function. Handles the full loop — analyze, design, evaluate, log, commit, repeat.
argument-hint: "<project-path> <rounds>"
---

# Scientific Optimization Loop

You are an autonomous optimization agent. Your job is to iteratively improve
candidates against a quantitative scoring function.

## Arguments

- `$0` — project path (required). Must contain a scoring script and CLAUDE.md.
- `$1` — number of rounds to run (default: 3)

## Phase 0: Orient

1. Read `$0/CLAUDE.md` to understand:
   - What's being optimized (molecules, hyperparameters, architecture, etc.)
   - The scoring function and how to evaluate
   - Current best score and constraints
   - Domain-specific tips

2. Read `$0/results.tsv` for experiment history

3. Read recent git log: `git -C $0 log --oneline -10`

4. Identify the **evaluation command** (usually `uv run design.py`, `python evaluate.py`, etc.)

5. Identify the **design file** (the only file you modify — usually `design.py` or `config.yaml`)

## Phase 1: Analyze the landscape (first round only)

Before generating candidates, understand the scoring function through
**mathematical thinking models** (see [thinking-models.md](thinking-models.md)):

### Fourier / Harmonic Analysis (always do this)
- **Decompose** the score into component terms (penalties, bonuses, costs)
- **Find each component's peak** numerically — scan property ranges
- **Compute the theoretical ceiling** — what if all peaks align?
- **Identify destructive interference** — which components fight each other?

### Fibonacci / Golden Ratio (check for resonances)
- Do optimal property values land on Fibonacci numbers (1,2,3,5,8,13,21...)?
- Are property ratios near φ=1.618? This reveals natural harmonics.
- Use golden-angle sampling for uniform coverage of multi-dimensional space.

### Entropy Analysis (find the likely winner)
- What composition/formula has the most possible arrangements (max Shannon entropy)?
- Which properties carry the most mutual information with the score?
- Are your candidates clustered (low search entropy) or diverse (high)?

### Bayesian / Probabilistic (guide the search)
- Given what you've evaluated, where is expected improvement highest?
- Use multi-armed bandit logic to allocate compute across strategies.

### Topological (understand the landscape shape)
- Is the landscape smooth (hill climbing works) or rugged (random search)?
- Are all good candidates connected or isolated?

Apply **at least 2-3 models** here. Write the analysis in the commit message.

## Phase 2: Generate candidates

Use a **progression of strategies** across rounds:

### Round 1: Domain knowledge (manual design)
- Use domain expertise from CLAUDE.md/program.md
- Design 5-10 candidates testing clear hypotheses
- Focus on getting all constraints satisfied first

### Round 2: Systematic enumeration (scale up)
- Write code in the design file to enumerate 100-1000+ candidates
- Fragment combination, parameter sweeps, template substitution
- Filter by constraints, sort by score, keep top 10

### Round 3+: Diverse search (multi-strategy)
Pick from these strategies based on what's worked:

- **Random generation**: combine fragments from pools (100K+ scale)
- **Mutation**: swap/insert/delete on best candidates (SMILES, params, etc.)
- **Database mining**: query real-world databases via WebFetch
  - Chemistry: ChEMBL API (`https://www.ebi.ac.uk/chembl/api/data/molecule.json?...`)
  - ML: papers with code, model zoo
- **Genetic algorithm**: crossover + mutation + selection
- **Mathematical analysis**: Fourier decomposition, entropy maximization

### Multi-agent deployment (for large searches)
When the search space is huge, deploy parallel agents:

```
Agent 1 (SCOUT):  Random generation at scale (100K+)
Agent 2 (MUTANT): Mutation hill-climbing from best seeds
Agent 3 (MINER):  Database queries for real-world candidates
```

Use `isolation: "worktree"` and `run_in_background: true` for each agent.
Merge results when all complete.

## Phase 3: Evaluate

1. Modify the design file with new candidates
2. Run the evaluation command
3. Record the score

## Phase 4: Log and commit

1. Append best result to `$0/results.tsv`:
   ```
   <commit>	<score>	<candidate>	<status>	<description>
   ```
   Use `pending` as commit hash before committing.

2. Git commit:
   - Improved: `git commit -m "keep: <description with hypothesis and result>"`
   - Not improved: revert design file, then `git commit -m "discard: <description>"`

3. Always include `Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>` in commits.

## Phase 5: Iterate

Repeat Phases 2-4 for `$1` rounds (default 3).

Each round should use a **different strategy** or **refine** based on learnings.

After all rounds, summarize:
- Best score achieved
- Best candidate
- What strategies worked / didn't
- Suggested next steps

## Key principles

1. **Zero penalty first**: get all constraints satisfied before optimizing the objective
2. **Diverse exploration**: don't just tweak one design — try fundamentally different approaches
3. **Scale matters**: 100K random candidates often beats 100 carefully designed ones
4. **Real data wins**: database mining finds what decades of domain expertise produced
5. **Prove the ceiling**: mathematical analysis prevents wasted effort on impossible improvements
6. **Document everything**: commit messages are your lab notebook
