# Neural DNA

Local ancestry inference for the historical source of successive blocks in spliced neural networks.

- **Small pretrained language-model pilot:** [lm_local/README.md](lm_local/README.md). Open [lm_local/00_run_pipeline.ipynb](lm_local/00_run_pipeline.ipynb) and run all cells. Defaults use SmolLM2-135M, short sequences, seed 63, and a small training budget. CPU and Apple MPS are supported through PyTorch device selection.
- **Original residual-MLP experiment:** [00_run_pipeline.ipynb](00_run_pipeline.ipynb). Its design and measured results are in [METHODS.md](METHODS.md) and [RESULTS.md](RESULTS.md).

Both experiments use lettered PDP tasks, each with `input`, `output`, and `src`, linked in pipeline order. The language-model pilot has its own dependencies and runtime cache under `lm_local/`.
