# GPU machine follow-up

This repo was started on a host with **no NVIDIA driver** (`nvidia-smi` missing) and **no PyTorch install**. Notebooks are written, not executed. Finish the items below on a CUDA box.

## Environment

- [ ] `nvidia-smi` shows a driver new enough for **CUDA 13.2** (PyTorch 2.14 default wheel).
- [ ] `uv sync` (or `uv sync --extra cpu` only if there is still no GPU).
- [ ] Confirm:
  ```text
  torch.__version__          # >= 2.14
  torch.version.cuda         # 13.2
  torch.cuda.is_available()  # True
  torch.cuda.get_device_name(0)
  ```
- [ ] Register a Jupyter kernel from the uv env and open the notebooks there.

If the driver is too old for 13.2, fall back to a published older CUDA index (`cu130` / `cu126`) in `pyproject.toml` `[tool.uv.index]` / `[tool.uv.sources]` — do not stay on CPU for the checks below.

## 00 — `00_fundamental_signals.ipynb`

- [ ] Run all cells with tensors on `cuda`.
- [ ] Device banner prints GPU name and compute capability (not the “No GPU” fallback).
- [ ] `math` vs SciPy vs Torch overlays still match (float32 GPU vs float64 CPU ~1e-6).
- [ ] Bench cell: `torch.cuda.synchronize()` timings; GPU column is a real `cuda` device, not CPU.
- [ ] Save executed outputs in the notebook (File → Save) so the next machine can see figures.

## 10 — `10_low_pass_filters.ipynb`

- [ ] Run all cells on GPU.
- [ ] python-control Bode + `forced_response` remains the reference.
- [ ] Sequential Euler IIR on `cuda` matches the `math` list IIR (float32).
- [ ] `conv1d` FIR low-pass actually runs on GPU (`y_th.device` is `cuda:0`).
- [ ] Compare wall time: Python-loop IIR on GPU vs `conv1d` FIR (FIR should win).

## 11 — `11_band_pass_filters.ipynb`

- [ ] Run all cells on GPU.
- [ ] In-band 220 Hz RMS ≫ out-of-band 880 Hz RMS on **all four** stacks.
- [ ] Torch Euler state lives on `cuda`; FIR `conv1d` band-pass on `cuda`.
- [ ] Save executed outputs.

## After first GPU run

- [ ] Commit the executed `.ipynb` outputs (plots + printed CUDA banner).
- [ ] Note driver / GPU / `torch.version.cuda` in that commit message.
- [ ] Optional: `uv sync --extra slycot` if you want faster python-control numerics (needs Fortran).
