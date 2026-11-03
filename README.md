# Neural local ancestry: a PDP experiment

Recover the historical source of successive residual blocks in a spliced neural network. This is a controlled CPU experiment with trained networks and known splice histories. It is not an LLM benchmark or a claim of a new ancestry algorithm.

Open `00_run_pipeline.ipynb` from this project directory and run all cells. Install `requirements.txt` into the notebook environment first; `requirements-lock.txt` records the versions used for this run. All analysis code is in code-only notebooks; the runner executes the tasks in order, gives each a fresh namespace, and saves their outputs. It also restores directory symlinks when an archive extractor writes them as text files. The experiment needs no network access after dependencies are installed: scikit-learn supplies the digit images.

The random seed is 63. Each stage's `input` after `a_import` is a relative symlink to the previous stage's `output`. Inherited output files point through `../input/`, so every dependency remains traceable. The archive contains the executed task notebooks, checkpoints, predictions, metrics, and figures. The helper notebook is loaded by the tasks as a function library.

| Task | Activity | Main new output |
|---|---|---|
| `a_import` | Import digit images, configuration, sources, and helper notebook | `digits.npz`, `environment.json` |
| `b_train` | Train common ancestor, source families, reference siblings, and independent donor branches | `trained_networks.npz`, `network_metrics.csv` |
| `c_splice` | Copy known source tracts and continue training; construct permutation controls | `query_blocks.npz`, `truth.npz`, `queries.csv` |
| `d_fit` | Build reference panels and choose inference settings using calibration mosaics | `reference_panels.npz`, `fitted_parameters.csv` |
| `e_infer` | Infer sources without reading test ancestry | `predictions.npz`, `prediction_index.csv` |
| `f_evaluate` | Score ancestry, boundaries, uncertainty, missing sources, and paired gains | `summary.csv`, `paired_gains.csv`, `open_set_metrics.csv` |
| `g_report` | Make comparison figures and headline tables | `marker_recovery.png`, `training_recovery.png`, `ancestry_painting.png` |

## Experimental design

The model has a tanh input stem, 24 tanh residual blocks of width 24, and a ten-class linear output. Each block contains 576 weights and 24 biases. The stem and output head come from the common ancestor when constructing a mosaic; the experiment infers ancestry only for the 24 residual blocks.

The bundled dataset has 1,797 handwritten digit images. A stratified 75/25 train/test split is fixed at seed 63. The common model is trained first. Four source families branch from it under different minibatch histories and class prevalences, while solving the same digit classification task. Each family then produces a calibration donor, an independent test donor, and three reference siblings. No ancestry watermarks or random identifying tags are inserted into weights.

Sources A–C are present in the inference reference panel. Source D is used to construct a missing-source test and is withheld from that panel. Exact-reference inference has the actual source donor checkpoints. Proxy-reference inference has only the three sibling checkpoints for each known source, so its labels identify source lineages rather than establish the unique identity of an unseen donor.

Calibration uses 18 mosaics: 12 with expected tract length four blocks, six with expected tract length 1.5. Test data contain 24 long-tract mosaics, 12 short-tract mosaics, eight single-source controls, and 12 missing-source mosaics. Every mosaic is evaluated after 0, 25, 100, and 300 further Adam steps. The optimizer is retained between saved checkpoints. The same underlying mosaics are reused across these settings, so they are paired observations.

Twelve extra permutation cases and their oracle reversals test coordinate dependence: six fixed long-tract test mosaics at 0 and 300 steps. A common hidden-unit permutation is applied consistently to the stem, every residual block, and the head. It preserves logits while changing the weight coordinates supplied to inference. Reversal uses the known inverse permutation; it is a diagnostic, not an implemented alignment method.

The main resolution is one complete residual block. Within-block splices, inserted or reordered blocks, different widths, unrelated initializations, and continuous weight mixtures are outside this pilot.

## What the ancestry label means

The target is the donor from which a block was copied before further training. After training, the block's numerical weights have changed, but its historical label remains fixed. Recovering that label does not measure how much the source controls the final network's behavior. Digit test accuracy is reported separately as a check that the experiment uses functioning networks.

## Genetics correspondence and prior work

| Local ancestry inference | Neural experiment |
|---|---|
| Ordered genomic positions | Ordered residual blocks |
| Homologous genetic markers | Fixed weight/bias coordinates in matching blocks |
| Reference haplotypes | Exact donor or related checkpoint panel |
| Admixed haplotype | Block-spliced network |
| Mutation or population mismatch | Continued training and proxy-reference mismatch |
| Recombination boundary | Change of source between adjacent blocks |

The implemented method is a custom continuous-weight adaptation of a copying model. It is informed by [Loter](https://doi.org/10.1093/molbev/msy126), which minimizes mismatch plus a penalty for copying-source switches. No native Loter or RFMix package is invoked. [RFMix](https://doi.org/10.1016/j.ajhg.2013.06.020) is a contextual reference for local assignment from panels and structured inference; its random forests, CRF details, EM, and phase correction are not implemented here.

[modelDNA](https://arxiv.org/abs/2607.10617) already studies model lineage and merge decomposition from sampled weight fingerprints. The specific question evaluated here is ordered block-source reconstruction after discrete splicing and continued training. A broader novelty assessment would require reviewing that work and other model provenance methods in detail.

## Interpretation limits

The test mosaics share a small, fixed source panel and a common trained ancestor. Mosaic bootstrap intervals describe variation over splice realizations conditional on that panel; they do not establish generalization over independent network populations. Marker counts, training checkpoints, and methods share the same mosaics. Treat the full set of comparisons as an exploratory pilot, not independent confirmatory tests.

All source candidates must initially occupy aligned coordinates. The permutation control tests this assumption directly. Posterior probabilities come from a fitted energy model and can be miscalibrated under distribution shifts. Missing-source rejection uses a calibration residual threshold and is evaluated separately from the forced closed-set source labels.

See `RESULTS.md` for measured results and `METHODS.md` for the inference objective and validation checks.
