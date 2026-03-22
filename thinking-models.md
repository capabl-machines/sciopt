# Mathematical Thinking Models for Optimization

These are structured reasoning frameworks drawn from different branches of
mathematics. Apply them to any scoring function to discover structure that
brute-force search misses.

---

## 1. Fourier / Harmonic Analysis

**Core idea**: Any scoring function is a superposition of component "waves."
Decompose it to find where all waves constructively interfere (the global peak).

### How to apply

1. **Identify the components**: If `score = f1 + f2 + f3 - g1`, each term is
   a "frequency" in property space.

2. **Find each component's peak numerically**:
   ```python
   # Scan each component independently
   for property_value in range(min, max, step):
       component_score = evaluate_component(property_value)
       if component_score > best:
           best = component_score
           peak = property_value
   ```

3. **Compute the theoretical optimum**: if all components peak simultaneously,
   what's the combined score? This is the **ceiling**.

4. **Measure the gap**: `gap = ceiling - current_best`. If the gap is tiny
   (< 0.1% of ceiling), you're essentially done.

5. **Find destructive interference**: which components fight each other?
   (e.g., increasing MW helps component A but hurts component B).
   These tradeoffs define the Pareto frontier.

### When it worked (automol)
- QED is a weighted geometric mean of 8 asymmetric double-sigmoid functions
- Each has a peak: MW=306, LogP=2.81, PSA=53, ROTB=3, AROM=2, etc.
- Theoretical max QED = 0.948449
- Our best QED = 0.948420 → gap of 0.000029 → proven near-optimal
- The ALERTS desirability (0.842 at 0 alerts) was identified as the
  fixed "damping frequency" that caps the entire system

### Domain-general application
- ML loss functions: decompose into data term + regularization + penalty
- Architecture search: decompose latency + accuracy + memory
- Materials: decompose formation energy + bandgap + stability

---

## 2. Fibonacci / Golden Ratio

**Core idea**: The golden ratio φ = 1.618... appears as nature's optimization
constant. Systems that exhibit Fibonacci structure often sit at energy minima
or information-theoretic optima.

### How to apply

1. **Check if optimal values are Fibonacci numbers**:
   Fibonacci sequence: 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144...

   Do the best candidates have property values that cluster near Fibonacci
   numbers? This indicates the scoring function has natural resonances.

2. **Check golden ratio in property ratios**:
   ```
   φ = 1.618034

   ratio_1 = property_A / property_B
   ratio_2 = property_C / property_D

   if abs(ratio - φ) < 0.1:  # near golden ratio
       # This pair of properties has a harmonic relationship
   ```

3. **Fibonacci search** (optimization algorithm):
   Instead of binary search or grid search, use Fibonacci spacing to narrow
   the optimum. Each step eliminates 1/φ of the remaining interval —
   provably optimal for unimodal functions with bounded evaluation count.

4. **Golden angle sampling**:
   When generating candidates in multi-dimensional space, use the golden
   angle (137.508°) to space samples. This produces the most uniform
   coverage with no clustering — better than random or grid.
   ```python
   golden_angle = math.pi * (3 - math.sqrt(5))  # ~2.4 radians
   for i in range(n_samples):
       theta = i * golden_angle
       r = math.sqrt(i / n_samples)  # sunflower distribution
       x = r * math.cos(theta)
       y = r * math.sin(theta)
   ```

### When it worked (automol)
- Best molecule had Fibonacci property values:
  - Heavy atoms = 21 (F₈)
  - Ring count = 3 (F₄)
  - Aromatic rings = 2 (F₃)
  - HBD = 1 (F₂)
  - HBA = 3 (F₄)
  - ROTB = 3 (F₄)
- LogP ≈ 2.87 ≈ φ + 1 = 2.618 (near golden)
- PSA/HBA ≈ 50/3 ≈ 16.7 ≈ 10φ (golden scaled)
- These are not coincidences — QED's desirability peaks align with
  Fibonacci because they model natural drug-like distributions

### Domain-general application
- Neural network layer sizes: Fibonacci-scaled widths (e.g., 8→13→21→34)
- Learning rate schedules: φ-ratio decay steps
- Batch size selection: Fibonacci numbers often work well
- Feature selection: golden-ratio-based importance thresholds

---

## 3. Entropy / Information Theory

**Core idea**: Among all candidates that satisfy the constraints, the one with
maximum entropy (most "disordered" / most "probable") is the best default
guess. Conversely, candidates with unusual (low-entropy) property distributions
are more likely to be special.

### How to apply

1. **Shannon entropy of candidate composition**:
   ```python
   def composition_entropy(candidate):
       """How diverse are the building blocks?"""
       counts = count_elements(candidate)  # atoms, params, features
       total = sum(counts.values())
       probs = [c/total for c in counts.values()]
       return -sum(p * log2(p) for p in probs if p > 0)
   ```
   Higher entropy = more diverse composition = more possible arrangements.

2. **Maximum entropy principle**:
   Given constraints (property ranges), generate the candidate with maximum
   compositional entropy. This is the "most likely" good candidate because
   it has the most structural isomers.

3. **Mutual information between properties**:
   ```
   I(score; property) = H(score) - H(score | property)
   ```
   Which properties carry the most information about the score?
   Focus optimization on high-MI properties — they're the control knobs.

4. **Entropy of the search distribution**:
   If all your candidates cluster in one region, search entropy is low.
   Maximize search entropy to ensure coverage:
   - Use diverse fragment pools
   - Enforce minimum Tanimoto distance between candidates
   - Sample from multiple scaffold families

5. **Boltzmann selection**:
   Instead of always keeping the best, select candidates with probability
   proportional to exp(-score / temperature). High temperature explores,
   low temperature exploits. Anneal temperature over rounds.
   ```python
   import math, random
   temperature = 1.0  # decrease each round
   prob = math.exp(-score / temperature)
   if random.random() < prob:
       keep(candidate)
   ```

### When it worked (automol)
- Molecular formulas with highest Shannon entropy (~1.8 bits across atom
  types: C, N, O, F/Cl) consistently produced the best QED scores
- The best molecules had ~15C + 3N + 2O + 1halogen (4 distinct atom types)
- Too few types (all carbon) = low entropy = limited QED
- Too many types (C,N,O,S,F,Cl,Br) = high entropy but fragmented = poor QED
- Sweet spot is 3-4 atom types at moderate proportions

### Domain-general application
- Feature engineering: maximum entropy feature distributions
- Ensemble methods: diverse models (high ensemble entropy) generalize better
- Data augmentation: maximize entropy of augmented distribution
- Hyperparameter search: uniform (max-entropy) priors on parameters

---

## 4. Probabilistic / Bayesian Thinking

**Core idea**: Treat the scoring function as a black box with uncertain output.
Build a probabilistic model of what regions of the design space are likely to
contain the optimum, and sample where expected improvement is highest.

### How to apply

1. **Prior from domain knowledge**: start with a belief about which
   parameter ranges are likely good (from CLAUDE.md, literature, etc.)

2. **Likelihood from evaluations**: each scored candidate updates the model.
   Regions near good scores are more likely to contain the optimum.

3. **Expected improvement (EI)**:
   ```
   EI(x) = E[max(0, f_best - f(x))]
   ```
   Sample where EI is highest — balances exploration (uncertain regions)
   and exploitation (near known good points).

4. **Thompson sampling**: for each candidate position, draw from the
   posterior distribution and pick the one with the best sample.
   Simple and works well in practice.

5. **Multi-armed bandit for strategy selection**:
   Treat each generation strategy (random, mutation, DB mining, GA) as an
   "arm." Use UCB1 or Thompson sampling to allocate compute:
   ```
   UCB(strategy) = mean_improvement + sqrt(2 * log(total_rounds) / n_uses)
   ```
   Strategies that find improvements get more rounds.

### When it worked (automol)
- After proving the QED ceiling via Fourier analysis, Bayesian reasoning
  told us: the probability of finding a molecule with QED > 0.9485 is
  essentially zero. This saved compute.
- Multi-agent deployment used implicit bandit: SCOUT got the most compute
  (500K) because random generation had the best track record.

---

## 5. Topological / Graph Theory

**Core idea**: The design space has a graph structure. Candidates are nodes,
transformations (mutations) are edges. The scoring function defines a landscape
on this graph. Find the topology of high-scoring regions.

### How to apply

1. **Neighborhood structure**: define what "one step" means in your space.
   - Chemistry: one atom swap, one bond change, one substituent addition
   - ML: one hyperparameter change
   - Architecture: one layer added/removed/modified

2. **Local vs global optima**: if all neighbors of a candidate score worse,
   it's a local optimum. Escape via:
   - Large jumps (random restart)
   - Tunneling (change multiple things at once)
   - Scaffold hopping (change the underlying topology)

3. **Fitness landscape ruggedness**: how correlated are neighbors' scores?
   - Smooth landscape → hill climbing works → use mutations
   - Rugged landscape → random search works → use diverse generation
   Compute: `correlation(score(x), score(neighbor(x)))` over many x.

4. **Connected components**: are all good candidates connected by short
   paths? If yes, hill climbing will find them. If no, you need multiple
   starting points.

### When it worked (automol)
- The QED landscape around score ~0.052 turned out to be very smooth —
  many diverse scaffolds (oxetane, azacyclononene, piperidine, etc.)
  all clustered near the same score, indicating a broad flat optimum
- This meant hill climbing (MUTANT) and random search (SCOUT) found
  similar-quality solutions — the landscape topology was a gentle basin

---

## 6. Dimensional Analysis / Scaling Laws

**Core idea**: how does the scoring function scale with the "size" of the
candidate? Are there power laws? Do certain ratios matter more than absolutes?

### How to apply

1. **Normalize properties**: convert to dimensionless ratios before comparing.
   ```
   normalized = (value - range_min) / (range_max - range_min)
   ```

2. **Scaling exponents**: plot score vs. each property on log-log scale.
   Linear relationship → power law → the exponent tells you sensitivity.

3. **Intensive vs extensive properties**:
   - Extensive (scale with size): MW, heavy atoms, ring count
   - Intensive (independent of size): LogP, Fsp3, QED
   Focus on intensive properties — they're the true "quality" measures.

4. **Dimensional reduction**: if 10 properties matter but only 3 are
   independent (others are correlated), work in the 3D subspace.
   Use PCA or correlation analysis to find the true degrees of freedom.

---

## Integration: When to use which model

| Situation | Best thinking model |
|-----------|-------------------|
| "What's the theoretical best?" | **Fourier** — decompose and find peaks |
| "Which properties should I target?" | **Fibonacci** — check for natural resonances |
| "What kind of candidate is most likely good?" | **Entropy** — maximum entropy composition |
| "Where should I search next?" | **Bayesian** — expected improvement |
| "Am I stuck in a local optimum?" | **Topological** — landscape analysis |
| "Which properties actually matter?" | **Dimensional** — scaling laws |
| "Should I explore or exploit?" | **Boltzmann/Bandit** — temperature annealing |

Apply **at least 2-3 of these models** during Phase 1 (landscape analysis).
The insights compound — Fourier finds the ceiling, entropy finds the
composition, Fibonacci validates the target, Bayesian guides the search.
