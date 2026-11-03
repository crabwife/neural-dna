# Small language-model ancestry pilot

A standalone PDP pipeline using the pretrained **SmolLM2-135M** causal language model. It trains two source lineages, splices their transformer blocks, continues training the mosaics, and compares local ancestry recovery with simpler matching. The original MLP pipeline remains at the repository root.

## Run

From this directory, install the requirements into your notebook environment and open `00_run_pipeline.ipynb`. Run all cells in order. For a separate environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt jupyterlab
jupyter lab 00_run_pipeline.ipynb
```

The notebook's settings cell exposes `PROFILE` and `DEVICE`. The defaults are `local` and `auto`. Device selection tries CUDA, then Apple MPS, then CPU. Use `DEVICE = 'cpu'` to choose CPU explicitly. Python 3.10 or newer is needed; the installed PyTorch build must support your platform.

The first local run downloads a pinned model/tokenizer snapshot and TinyShakespeare. Later runs use the cache. No Hugging Face account, API key, hosted inference, or remote training service is required. Runtime checkpoints and downloads are ignored by Git.

For an offline integration check, set `PROFILE = 'smoke'`. That profile creates a small random Llama model and synthetic token data. Its reports explicitly identify it as a fixture. It never substitutes for an unavailable pretrained model. Local and smoke outputs occupy separate directories.

## Default budget

| Setting | Local default |
|---|---|
| Pretrained model | `HuggingFaceTB/SmolLM2-135M` |
| Pinned revision | `93efa2f097d58c2a74874c7e644dbc9b0cee75a2` |
| Transformer blocks | 30 |
| Source lineages | 2 |
| Children per lineage | One reference, one calibration donor, one test donor |
| Family / child training | 8 / 2 optimizer steps |
| Sequence length / batch size | 64 / 1 |
| Calibration mosaics | 2 long-run and 2 short-run |
| Test mosaics | 3 long-run and 3 short-run |
| Training after splicing | Snapshots at 0 and 4 steps |
| Sampled parameters per block | 4, 32, 256; nested samples |
| Random seed | 63 |

The default performs 68 training steps in total, plus short held-out evaluations. All transformer-block parameters are trainable. Embeddings, the tied output head, and the final normalization are frozen. This is direct block-weight fine-tuning, without LoRA or quantized training. The model runs in float32 for CPU/MPS portability.

Budget roughly 4 GB of free memory and 2 GB of disk for model cache and experiment artifacts, in addition to installed dependencies. These are estimates; device and package behavior affect the peak. Model loading and training are sequential, and only four donor update files are stored. The pipeline does not retain complete mosaic checkpoints. Runtime depends on the machine; use the offline profile to verify the environment first if needed.

Edit `a_import/input/config.json` to increase the number of mosaics or training steps. Larger sample counts are needed for a study. The defaults are deliberately a small local pilot.

## Pipeline and outputs

Each task has `input`, `output`, and `src`. After `a_import`, the task input is a relative symlink to the preceding output. Artifacts are written under `output/local` or `output/smoke`, and inherited files link through that chain. All task code is in code-only notebooks. The helper notebook is imported by the tasks.

| Task | Activity |
|---|---|
| `a_import` | Download/cache pinned inputs, tokenize, hold out evaluation text, record provenance |
| `b_train` | Train source families and independent children; save donor updates and reference fingerprints |
| `c_splice` | Copy known block runs, continue training, record fingerprints, perplexity, and sampled-coordinate drift |
| `d_fit` | Select copying penalty and temperature using calibration ancestry |
| `e_infer` | Infer test sources without reading test ancestry |
| `f_evaluate` | Score source labels, boundaries, probabilities, and paired method differences |
| `g_report` | Write comparison plots and a report |

Start with `g_report/output/local/RESULTS.md` and `g_report/output/local/source_recovery.png`. The fixed-example source painting is `ancestry_painting.png`. Full results are in `f_evaluate/output/local/summary.csv`, `trial_metrics.csv`, and `paired_gains.csv`. Timings are saved at `pipeline_timings_local.csv`.

The input corpus is split into training text and held-out evaluation text. Source lineages train on separate halves of the training portion. Reference, calibration-donor, and test-donor branches start from the same lineage ancestor but use independent minibatch histories and fresh optimizers. The same held-out language-model windows are used across models.

Donor files store FP16 **updates relative to the pretrained block weights**. Each donor is reconstructed before recording its exact-reference fingerprint. Mosaics copy those reconstructed weights, so the pre-training fingerprint check compares the actual source tensors used. Perplexity checks model behavior separately from historical source accuracy.

## Inference and interpretation

The copying method minimizes continuous reference mismatch plus a penalty for switching reference checkpoints along depth. Viterbi finds the minimum-cost path; forward-backward supplies source probabilities. Switch penalties and temperatures are fitted on calibration mosaics only. Exact panels contain the actual test donor fingerprints; related panels contain the reference sibling fingerprints.

Nearest matching uses each block independently. Global matching chooses one reference for the entire model. Two-source convex least-squares mixture coefficients are also saved. With exact endpoints their hard source labels equal nearest matching mathematically, so they are a diagnostic rather than an independent competing classifier. This is not a reproduction of the complete modelDNA system.

The marker budget measures coordinates used for matching. This training experiment loads model weights and constructs known splices; it is not an implementation of remote partial-weight downloading. Reported drift also uses the sampled coordinates, rather than all parameters.

One source panel and three test mosaics per tract regime support a pipeline and feasibility check. They do not establish population-level effect sizes or a novelty claim. Long runs may benefit from continuity while rapid true source changes are oversmoothed. Copied-block origin also differs from causal influence on generated text. Missing-source detection, reparameterization controls, multiple model families, and stronger provenance baselines remain outside this small run.

Model reference: [SmolLM2](https://huggingface.co/HuggingFaceTB/SmolLM2-135M), Apache 2.0. Corpus: [TinyShakespeare in char-rnn](https://github.com/karpathy/char-rnn/tree/370cbcd448eb7daf32f21a6be560b70e0b33c4e3/data/tinyshakespeare). Method context: [Loter](https://doi.org/10.1093/molbev/msy126) and [modelDNA](https://arxiv.org/abs/2607.10617).
