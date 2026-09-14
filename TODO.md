# GPU machine follow-up

This repo was started on a host with **no NVIDIA driver** (`nvidia-smi` missing) and **no PyTorch install**. Notebooks are written, not executed. Finish the items below on a CUDA box.

## Environment

- [x] `nvidia-smi` shows a driver new enough for **CUDA 13.2** (PyTorch 2.14 default wheel).
- [x] `uv sync --extra gpu` (or `uv sync --extra cpu` only if there is still no GPU).
- [x] Run `50_CrossCheck.ipynb` — it must print `PASS` (driver + CUDA wheel + live `cuda:0` kernel).
- [x] Confirm:
    ```text
    torch.__version__          # 2.14.0+cu132
    torch.version.cuda         # 13.2
    torch.cuda.is_available()  # True
    torch.cuda.get_device_name(0)  # NVIDIA GeForce RTX 3060
    ```
- [x] Register a Jupyter kernel from the uv env and open the notebooks there.

If the driver is too old for 13.2, fall back to a published older CUDA index (`cu130` / `cu126`) in `pyproject.toml` `[tool.uv.index]` / `[tool.uv.sources]` — do not stay on CPU for the checks below.

## 00 — `00_fundamental_signals.ipynb`

- [x] Run all cells with tensors on `cuda`.
- [x] Device banner prints GPU name and compute capability (not the “No GPU” fallback).
- [x] `math` vs SciPy vs Torch overlays still match (float32 GPU vs float64 CPU ~1e-6).
- [x] Bench cell: `torch.cuda.synchronize()` timings; GPU column is a real `cuda` device, not CPU.
- [x] Save executed outputs in the notebook (File → Save) so the next machine can see figures.

## 10 — `10_low_pass_filters.ipynb`

- [x] Run all cells on GPU.
- [x] python-control Bode + `forced_response` remains the reference.
- [x] Sequential Euler IIR on `cuda` matches the `math` list IIR (float32).
- [x] `conv1d` FIR low-pass actually runs on GPU (`y_th.device` is `cuda:0`).
- [ ] Compare wall time: Python-loop IIR on GPU vs `conv1d` FIR (FIR should win).

## 11 — `11_band_pass_filters.ipynb`

- [x] Run all cells on GPU.
- [x] In-band 220 Hz RMS ≫ out-of-band 880 Hz RMS on **all four** stacks.
- [x] Torch Euler state lives on `cuda`; FIR `conv1d` band-pass on `cuda`.
- [x] Save executed outputs.

## 12 — `12_high_pass_filters.ipynb`

- [x] Run all cells on GPU.
- [x] DC of the offset square wave is rejected; 220 Hz edges remain on all four stacks.
- [x] FIR `conv1d` high-pass (`δ −` low-pass) lives on `cuda`.

## 13 — `13_notch_filters.ipynb`

- [x] Run all cells on GPU.
- [x] 220 Hz RMS ≫ 880 Hz RMS on control / math / SciPy / torch.
- [x] Tustin notch + FIR (`δ −` band-pass) on `cuda` (forward Euler is unstable at \(f_n/f_s \approx 0.11\)).

## 14 — `14_second_order_plants.ipynb`

- [x] Step family for several `ζ`; python-control is the spec.
- [x] Batched Euler over `ζ` on `cuda` (shape `(B, T)`).

## 15 — `15_pid_control.ipynb`

- [x] Closed-loop `T(s)` margins print; step tracks the reference.
- [x] GPU `Kp` sweep runs on `cuda`.

## 16 — `16_lead_lag.ipynb`

- [x] Lead raises PM and cuts overshoot vs unity feedback.
- [x] GPU sweep over lead pole `p` on `cuda`.

## 17 — `17_state_space.ipynb`

- [x] `ctrb` / `obsv` full rank; `place` damps the plant.
- [x] Batched ICs integrate on `cuda`.

## 18 — `18_discrete_time.ipynb`

- [x] ZOH / Tustin / Euler `c2d` overlay at coarse vs 8 kHz.
- [x] Batched ZOH over sample rates on `cuda`.

## 19 — `19_frequency_response_fft.ipynb`

- [x] python-control Bode, SciPy `bode`, `cmath` resolvent, and `torch.fft.rfft` overlay.
- [x] Batched rFFT of impulse responses on `cuda`.

## After first GPU run

- [x] Commit the executed `.ipynb` outputs (plots + printed CUDA banner).
- [x] Note driver / GPU / `torch.version.cuda` in that commit message.
- [ ] Optional: `uv sync --extra slycot` if you want faster python-control numerics (needs Fortran).
