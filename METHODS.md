# Continuous block-copying inference

## Reference states and observations

A query supplies an ordered sequence of 24 vectors. Each vector concatenates the 576 weights and 24 biases of one residual block. All candidates have the same architecture, block order, hidden-unit coordinates, and trained common ancestor.

An exact panel has three checkpoint states, one actual donor per source. A proxy panel has nine checkpoint states: three independently trained siblings for each of three source lineages. The calibration and test donors are different sibling branches. The proxy references are shared between calibration and test.

For marker count `m`, use the first `m` coordinates of one seeded random permutation of the 600 coordinates. This gives nested marker sets of sizes 1, 4, 16, 64, and 600. A coordinate has the same meaning in every query and reference block. No markers are selected using test labels.

Let `q[l, j]` and `r[s, l, j]` be query and reference weights at block `l`, marker `j`, and reference checkpoint `s`. The mismatch cost is

\[
d_{l,s}=\frac{1}{m a_l}\sum_{j\in M_m}(q_{l,j}-r_{s,l,j})^2.
\]

The scale `a[l]` is the average squared distance between pairs of known source-lineage reference centroids, averaged over all 600 coordinates. It is computed entirely from proxy references and floored at `1e-12`. It weights blocks comparably and does not whiten coordinates or assume independent markers.

## Copying objective

The checkpoint-copying path minimizes

\[
E(s_{1:L})=\sum_{l=1}^{L}d_{l,s_l}
 +\lambda\sum_{l=2}^{L}\mathbf 1(s_l\ne s_{l-1}).
\]

Viterbi dynamic programming finds the exact minimizer. Checkpoint labels are then mapped to their source lineage. Switching between two references in the same lineage also pays the copying penalty, following the parental-haplotype copying interpretation. This is a continuous mismatch adaptation of the Loter-style objective, not an execution of the native genetic-data implementation and not a full reproduction of its ensemble procedure.

The probabilistic version assigns paths probability proportional to `exp(-E / T)`. A uniform initial distribution and symmetric checkpoint transitions produce an equivalent HMM. Forward-backward returns checkpoint-state marginals, which are summed over checkpoints belonging to each source. These marginals supply negative log likelihood, Brier scores, and confidence curves. Viterbi source labels supply ancestry accuracy and breakpoint metrics; Viterbi and marginal modes need not coincide.

The genetic interpretation requires an ordered, homologous sequence and a copying process. Here those assumptions are supplied by the splice construction. Network depth is an ordering, not a biological genetic map, and `lambda` is not an estimated recombination rate.

## Baselines

- **Nearest:** choose the closest reference independently at each block, then map it to its source.
- **Global:** choose the single reference with the smallest total mismatch across all blocks and assign its source everywhere.
- **Copying:** infer the ordered reference-copying path and map it to source labels.

Nearest is the primary comparator. Exact copied blocks with exact references are expected to match trivially before training; success there alone is not evidence for an advantage from local ancestry inference.

## Calibration and test separation

Each combination of reference panel, marker count, and further-training checkpoint gets its own fit. The switch penalty is selected from `[0, .05, .2, .5, 1, 2, 4]`; temperature is selected from `[.025, .05, .1, .25, .5, 1, 2, 4]`. The selected pair minimizes mean source-label negative log likelihood on calibration mosaics. Ties within `1e-12` favor the first pair in the grid. In particular, `lambda = 0` is allowed, so continuity is retained only when the fitted criterion supports it.

The calibration donor branches and splice realizations are separate from the test donors and mosaics. No test ancestry is used to fit the scale, select markers, choose settings, or set rejection thresholds. The `e_infer` notebook never loads `truth.npz`. `d_fit` selects calibration rows only; `f_evaluate` performs test scoring.

There are only 18 calibration mosaics. A selected temperature does not certify calibration under the short-tract, missing-source, or permutation shifts.

## Missing-source rejection

Closed-set calls always select A, B, or C. Separately, a block is rejected when its distance to the nearest reference exceeds the 99th percentile of minimum distances on the matching calibration setting, with threshold floored at `1e-10`. Thus a rejected block is not assigned a positively identified fourth ancestry.

AUROC and average precision distinguish actual D blocks from known-source blocks. The scored pool combines the 24 long-tract test mosaics and the 12 missing-source mosaics. Reported recall and false rejection use the fixed calibration threshold. This simple residual-based detector can fail after training; posterior confidence is not substituted for an unknown-source model.

## Metrics and uncertainty

Block accuracy is the fraction of the 24 historical donor labels recovered. Exact breakpoint F1 matches inferred and true source changes at the same inter-block position. A second F1 permits a one-block tolerance with one-to-one boundary matching. Results average trial metrics over mosaics, so boundary F1 is a macro average. A single-source path with no inferred or true breaks gets F1 of one.

Paired gains resample whole mosaics, not individual blocks, using seed 63 and 2,000 bootstrap replicates. They compare copying minus nearest block accuracy. Their 95% intervals condition on this fixed source panel; checkpoints and marker counts are paired repeated measurements. No population-level inference across independently trained panels or multiplicity-adjusted claim is made.

## Validation

- Analytic gradients are checked by finite differences for all six parameter groups before training.
- The decoder and posterior marginals are checked against exhaustive enumeration of all 81 paths in a four-position, three-state example.
- Every pre-training mosaic block is checked against the tensor actually copied from its recorded source.
- The hidden permutation is checked to preserve logits within `1e-10`; applying its known inverse must restore every tensor exactly.
- Boundary tolerance uses a one-to-one matching check.
- The pipeline runs in order, with recorded environment versions, timings, and fixed seed. Notebook cells contain code only.

These checks establish implementation consistency for the controlled experiment. They do not validate model-scale generalization, coordinate alignment, or historical identifiability after arbitrary transformations.
