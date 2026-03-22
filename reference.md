# Scientific Optimization Reference

## Strategy selection by round

| Round | Strategy | Scale | When to use |
|-------|----------|-------|-------------|
| 1 | Manual domain-expert design | 5-10 | Always start here |
| 2 | Systematic enumeration | 100-1K | After understanding constraints |
| 3 | Random generation + filter | 10K-500K | When design space is large |
| 4 | Database mining (ChEMBL, etc.) | 100-1K real | When real-world data exists |
| 5 | Multi-agent parallel search | 500K+ total | When approaching ceiling |
| 6 | Mathematical ceiling analysis | N/A | When progress stalls |

## Domain-specific strategies

### Chemistry (Molecular Design)
- **Fragment pools**: aromatic rings, saturated rings, linkers, substituents
- **SMILES mutation**: character swap (C↔N, c↔n, O↔S), insertion, deletion
- **Scaffold hopping**: replace core ring while keeping pharmacophore
- **Database mining**: ChEMBL API for bioactive compounds by property range
- **Similarity search**: Tanimoto fingerprints to known actives
- **QED analysis**: Fourier decomposition of desirability functions
- **BRICS**: retrosynthetic fragment decomposition and recombination

### Machine Learning (Hyperparameter Tuning)
- **Grid search**: exhaustive over discrete params
- **Random search**: sample continuous params (often beats grid)
- **Bayesian optimization**: Gaussian process surrogate model
- **Population-based training**: evolving configs during training
- **Learning rate schedules**: cosine, warmup, cyclical

### Materials Science
- **Composition enumeration**: vary elemental ratios
- **Crystal structure prediction**: prototype + relaxation
- **Phase diagram exploration**: convex hull analysis
- **Property regression**: ML models on Materials Project data

### Architecture Search
- **Cell enumeration**: combine primitives (conv, pool, skip, attention)
- **Evolutionary NAS**: mutation + crossover of architectures
- **One-shot**: weight sharing supernet
- **Hardware-aware**: latency/FLOPS constraints

## Scoring function analysis toolkit

### Decomposition
Break the score into additive/multiplicative components.
Optimize each independently, then jointly.

### Ceiling computation
For weighted geometric means (like QED):
```
max_score = exp(Σ w_i × log(peak_i) / Σ w_i)
```
Find each component's peak numerically by scanning.

### Sensitivity analysis
Which component has the most room for improvement?
```
gap_i = peak_i - current_i
impact_i = gap_i × weight_i
```
Focus on highest-impact components first.

## ChEMBL API reference

Base URL: `https://www.ebi.ac.uk/chembl/api/data/molecule.json`

Useful filters:
```
molecule_properties__mw_freebase__gte=X
molecule_properties__mw_freebase__lte=Y
molecule_properties__alogp__gte=X
molecule_properties__alogp__lte=Y
molecule_properties__hbd=X
molecule_properties__hba=X
molecule_properties__psa__gte=X
molecule_properties__psa__lte=Y
molecule_properties__aromatic_rings=X
molecule_properties__num_ro5_violations=0
max_phase=4          # approved drugs only
max_phase__gte=2     # phase 2+
limit=100
offset=0
```

Extract SMILES from: `molecules[].molecule_structures.canonical_smiles`

## PubChem API reference

Single compound: `https://pubchem.ncbi.nlm.nih.gov/rest/pug/compound/name/DRUGNAME/property/MolecularWeight,XLogP,IsomericSMILES/JSON`

## Results logging format

Tab-separated in results.tsv:
```
commit	score	candidate	status	description
abc1234	0.051	SMILES_OR_PARAMS	keep	hypothesis tested and result
```

Status values: `baseline`, `keep`, `discard`
