# Validation

Both the offline `smoke` profile and the default pretrained `local` profile were executed through all seven PDP tasks on CPU on November 3, 2026. The pretrained run used four PyTorch threads and the pinned model revision in `a_import/input/config.json`.

- Pretrained model parameters: 134,515,008; trainable block parameters: 106,202,880.
- Pretrained pipeline runtime: 131.5 seconds with model weights already cached. Runtime on another machine may differ.
- Source-mosaic fingerprints matched the actual reconstructed donor blocks exactly before further training.
- Viterbi matched exhaustive enumeration; maximum forward-backward posterior discrepancy was 1.11e-16.
- Source-state aggregation and equivalence of two-source convex-mixture hard calls to nearest matching passed their checks.
- Prediction probabilities were finite and summed to one. Both plots and the generated report were inspected.
- The CPU path was exercised. Apple MPS and CUDA selection are implemented but were not exercised in this validation.

For the default related-reference case with four sampled parameters per block after four further-training steps, nearest and copying both recovered 96.67% of long-run labels and 90.00% of short-run labels. There were only three test mosaics in each regime. The default therefore establishes a runnable small-model example, not an advantage for the copying method or a novelty claim.

Tested package versions: torch 2.15.0+cpu, transformers 4.57.6, numpy 2.3.5, pandas 2.2.3, scipy 1.17.0, safetensors 0.8.0. Generated data, model downloads, donor updates, and notebook execution outputs are excluded from this commit. Running the notebook regenerates local results under `g_report/output/local/`.
