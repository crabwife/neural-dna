# Results: local ancestry in block-spliced neural networks

This run used seed 63, a 24-block residual MLP, 18 calibration mosaics, and 56 test mosaics. Calibration and test donors were different branches. The main comparison below uses the 24 long-tract test mosaics: 576 block labels per setting. All percentages are measured on these constructed networks.

## What was implemented

- Trained one common ancestor, four source-family ancestors, eight donor checkpoints, and twelve reference siblings on digit classification. No ancestry tags were inserted into the weights.
- Constructed known source paths by copying actual residual-block tensors; saved further-training checkpoints at 0, 25, 100, and 300 Adam steps.
- Implemented a continuous-weight copying objective with a penalty for changing reference checkpoints, Viterbi decoding, and forward-backward posterior marginals. Proxy checkpoint marginals were aggregated to source lineages.
- Compared copying with independent nearest-reference matching and one-source global matching. Fit switch penalties and temperatures on calibration mosaics only.
- Tested sparse markers, short tracts, single-source controls, an absent source, and a function-preserving hidden-coordinate permutation.

## All 600 weights and biases per block

| Panel | Training steps | Copying | Global | Nearest | Copying boundary F1 |
| --- | --- | --- | --- | --- | --- |
| exact | 0 | 100.00% | 55.90% | 100.00% | 1.000 |
| exact | 25 | 100.00% | 53.30% | 100.00% | 1.000 |
| exact | 100 | 100.00% | 35.42% | 100.00% | 1.000 |
| exact | 300 | 52.60% | 28.65% | 52.60% | 0.409 |
| proxy | 0 | 100.00% | 54.34% | 100.00% | 1.000 |
| proxy | 25 | 100.00% | 53.12% | 100.00% | 1.000 |
| proxy | 100 | 99.48% | 40.10% | 99.31% | 0.982 |
| proxy | 300 | 30.56% | 28.65% | 30.56% | 0.061 |

Exact donor references recover all historical source labels and all boundaries through 100 further-training steps in this test. This success is also achieved by the nearest-reference baseline. Proxy panels recover source lineages rather than uniquely identify an unseen donor checkpoint.

After 300 steps, recovery degrades sharply even though mean digit test accuracy on long-tract mosaics is 97.32% (versus 97.32% before training). Thus a working classifier need not retain an easily recoverable donor history under this weight-distance method. These step counts describe this optimizer, learning rate, architecture, and training task; they are not general thresholds.

## Four markers per block

| Panel | Training steps | Copying | Global | Nearest | Copying boundary F1 |
| --- | --- | --- | --- | --- | --- |
| exact | 0 | 100.00% | 54.51% | 100.00% | 1.000 |
| exact | 25 | 94.97% | 54.34% | 92.71% | 0.860 |
| exact | 100 | 85.59% | 42.19% | 79.69% | 0.670 |
| exact | 300 | 51.74% | 28.65% | 50.52% | 0.319 |
| proxy | 0 | 86.11% | 51.91% | 76.91% | 0.610 |
| proxy | 25 | 79.51% | 48.78% | 73.44% | 0.439 |
| proxy | 100 | 74.31% | 48.96% | 63.72% | 0.499 |
| proxy | 300 | 43.40% | 28.65% | 43.40% | 0.297 |

At 100 training steps with proxy references and four markers per block, copying changes recovery by 10.59 percentage points relative to nearest matching. The paired whole-mosaic bootstrap 95% interval is [5.72, 14.93] percentage points, conditional on this fixed source panel. All marker counts, checkpoints, and tract regimes are included in `f_evaluate/output/summary.csv`; this highlighted comparison is exploratory.

On short tracts at the same checkpoint, copying recovers 57.29% versus 67.01% for nearest matching. The difference is -9.72 percentage points, with fixed-panel bootstrap interval [-14.58, -4.51]. A continuity prior can therefore erase real rapid source changes; improvement on long tracts is not a general improvement across splice patterns.

The useful role of continuity in this pilot is to pool weak local evidence when only a few coordinates are observed. It does not establish an advantage over full-weight nearest matching in the easy conditions, or a remedy for extensive training drift.

## Coordinate permutation

| Coordinates | Mosaics | Block accuracy | Boundary F1 |
| --- | --- | --- | --- |
| aligned | 24 | 100.00% | 1.000 |
| oracle_restored | 6 | 100.00% | 1.000 |
| permuted | 6 | 31.25% | 0.123 |

The permuted and oracle-restored rows use the same six fixed mosaics. The aligned row summarizes all 24 long-tract mosaics; their aligned recovery is perfect before training. The largest observed logit change under the permutation was 1.07e-14; applying the known inverse restored every tensor exactly. Raw-coordinate inference therefore fails on some functionally identical parameterizations. No automatic permutation alignment is implemented.

## Missing source

| Panel | Training steps | Unknown AUROC | Unknown recall | Known false rejection |
| --- | --- | --- | --- | --- |
| exact | 0 | 1.000 | 1.000 | 0.000 |
| exact | 100 | 0.923 | 0.647 | 0.060 |
| exact | 300 | 0.360 | 0.000 | 0.140 |
| proxy | 0 | 1.000 | 1.000 | 0.020 |
| proxy | 100 | 0.883 | 0.235 | 0.020 |
| proxy | 300 | 0.339 | 0.000 | 0.050 |

The fourth source was absent from the inference panel. Rejection used a fixed calibration residual threshold, independently of the forced A/B/C calls. It works best before substantial drift and deteriorates with further training. In particular, high source-assignment confidence must not be interpreted as evidence that all true sources are present in the reference panel.

## Checks and scope

- Maximum finite-difference gradient discrepancy: 3.07e-11 across the six parameter groups checked.
- Viterbi and forward-backward matched exhaustive enumeration on a four-block, three-state case; duplicate-reference posterior aggregation was checked separately.
- Every initial splice was verified against its recorded donor tensors. Permutation reversibility and one-to-one boundary matching were checked.
- The delivered pipeline contains code-only notebooks, seed 63, a traceable symlink chain, trained weights, truth, predictions, calibration settings, and plots.

The supported result is that ancestry continuity can improve sparse-coordinate reconstruction in aligned, shared-ancestor networks, while direct weight matching already solves many well-observed cases. This pilot does not establish robust recovery after extensive training, invariance to equivalent parameterizations, unique donor identification from proxy relatives, or performance on LLMs. Repeating the study over independent source panels is needed before treating the effect sizes as population-level results.

The method adapts the copying formulation used in [Loter](https://doi.org/10.1093/molbev/msy126). [modelDNA](https://arxiv.org/abs/2607.10617) is relevant prior work on model provenance and merge decomposition; this pilot does not establish novelty relative to that literature. See `METHODS.md` for the exact objective and scoring definitions.
