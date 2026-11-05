# Language-model ancestry: local

This is a small pretrained causal-language-model pilot.

Model: HuggingFaceTB/SmolLM2-135M.
Seed: 63. Transformer blocks: 30. Model parameters: 134,515,008.

The experiment trains two source lineages and independent reference/calibration/test children, copies known block runs, and continues training the mosaics. All transformer-block parameters are trainable; embeddings, tied output head, and final norm are frozen. Source checkpoints store FP16 updates and are reconstructed before exact fingerprints are recorded.

Calibration and test donors are different branches. Settings are fitted on calibration ancestry only. The inference notebook never reads test truth. The language-model evaluation uses held-out corpus windows, reused across checkpoints.

## Four sampled parameters per block

| panel | steps | kind | method | n | block_accuracy | boundary_f1 | perplexity |
| --- | --- | --- | --- | --- | --- | --- | --- |
| exact | 0 | long | copying | 3 | 1.0000 | 1.0000 | 32.7584 |
| exact | 0 | long | global | 3 | 0.6556 | 0.0000 | 32.7584 |
| exact | 0 | long | nearest | 3 | 1.0000 | 1.0000 | 32.7584 |
| exact | 0 | short | copying | 3 | 1.0000 | 1.0000 | 31.9893 |
| exact | 0 | short | global | 3 | 0.5333 | 0.0000 | 31.9893 |
| exact | 0 | short | nearest | 3 | 1.0000 | 1.0000 | 31.9893 |
| exact | 4 | long | copying | 3 | 1.0000 | 1.0000 | 33.0541 |
| exact | 4 | long | global | 3 | 0.6556 | 0.0000 | 33.0541 |
| exact | 4 | long | nearest | 3 | 0.9889 | 0.8333 | 33.0541 |
| exact | 4 | short | copying | 3 | 0.9556 | 0.9361 | 34.0737 |
| exact | 4 | short | global | 3 | 0.5333 | 0.0000 | 34.0737 |
| exact | 4 | short | nearest | 3 | 0.9667 | 0.9537 | 34.0737 |
| proxy | 0 | long | copying | 3 | 0.9444 | 0.6714 | 32.7584 |
| proxy | 0 | long | global | 3 | 0.6556 | 0.0000 | 32.7584 |
| proxy | 0 | long | nearest | 3 | 0.9444 | 0.6714 | 32.7584 |
| proxy | 0 | short | copying | 3 | 0.9444 | 0.9129 | 31.9893 |
| proxy | 0 | short | global | 3 | 0.5333 | 0.0000 | 31.9893 |
| proxy | 0 | short | nearest | 3 | 0.9444 | 0.9129 | 31.9893 |
| proxy | 4 | long | copying | 3 | 0.9667 | 0.7744 | 33.0541 |
| proxy | 4 | long | global | 3 | 0.6556 | 0.0000 | 33.0541 |
| proxy | 4 | long | nearest | 3 | 0.9667 | 0.7744 | 33.0541 |
| proxy | 4 | short | copying | 3 | 0.9000 | 0.8483 | 34.0737 |
| proxy | 4 | short | global | 3 | 0.5333 | 0.0000 | 34.0737 |
| proxy | 4 | short | nearest | 3 | 0.9000 | 0.8483 | 34.0737 |

All marker counts and per-mosaic differences are in `summary.csv` and `paired_gains.csv`. Nearest, global, and copying comparisons share the same observations. The saved two-source convex least-squares mixture coefficients are a diagnostic: with exact reference endpoints, their hard source decisions equal nearest matching mathematically; they are not an independent competing classifier.

These small sample counts and one source panel support a local pilot only. A benefit on long runs may come with errors on short runs. Copied source labels describe historical block origin, not causal influence on generated tokens. Marker drift is measured on sampled coordinates; perplexity uses the saved held-out windows. No hypothesis-test or population-level generalization claim is made.

The local pilot does not include absent-source rejection, invariance to hidden-coordinate changes, multiple model families, or the full modelDNA implementation. The copying objective is a standard mismatch-plus-switching model, so the genetics interpretation alone does not establish methodological novelty.
