# Genetic Algorithm (GA) Optimized Finite-Difference (FD) Seismic Modeling

## 1. Purpose of this notebook
This project implements and tests high-order, GA-optimized staggered-grid finite-difference (FD) schemes for 2D acoustic wave propagation. 

The notebook:
* Computes and plots dispersion and stability curves (Figures 1–6).
* Builds layered and complex velocity models, including the BP salt-dome model (Figures 7–8, 13).
* Runs 2D wave simulations using four FD schemes and compares them:
  * 40th-order conventional (reference)
  * 12th-order conventional
  * GA-optimized 12th-order (single set)
  * GA-optimized 12th-order (velocity range table)
* Produces shot gathers, wavefield snapshots, and animations (Figures 9–12, 14).

The notebook is designed so that anyone can open it in Google Colab and reproduce all figures from 1 to 14 with minimal manual editing.

---

## 2. Software and package requirements
You can run this notebook either:
* In **Google Colab** or **Jupyter Notebook**
* Locally in a Python environment (e.g., `conda` or `venv`)

**Required Python packages:**
* `numpy`
* `matplotlib`
* `numba`
* `segyio` (for reading the BP velocity model SEGY file)
* `pathlib` (standard library, used for output folders)
* `IPython.display` (for animations in notebooks)

If you are using Google Colab, you will likely need to install `segyio` and `numba` explicitly. You can do this in the first cell of your notebook:
```bash
!pip install segyio numba
```

---
## 3. External data required (BP velocity model)

You can get the `vel_z6.25m_x12.5m_exact.segy` file from the Model folder.

Several figures (8, 10, 12, 14) rely on the BP salt-dome velocity model stored in this SEGY file. The expected path in the notebook is:

```python
BP_SEGY_PATH = "./Model/vel_z6.25m_x12.5m_exact.segy"
```

**To make this work in Colab:**

1. Upload the SEGY file `vel_z6.25m_x12.5m_exact.segy` to Colab, into the `/Model/` directory.
* *Tip: In Colab, use the left sidebar → "Files" → "Upload".*

2. Ensure the filename and path exactly match `BP_SEGY_PATH`.
* *Note: If you change the filename or location, update `BP_SEGY_PATH` in all relevant cells (Figures 8, 10, 12, 14).*

*If you do not have this file, the BP-related figures will not run.*

---
## 4. General notebook structure and naming

Across the notebook, code blocks follow a consistent pattern. You will typically see a header like:

```python
# -----------------------------
# FIGURE X: Brief description
# -----------------------------

```

Followed by definitions of:

* **Global controls:** (e.g., `DTYPE`, `COMPONENT`, `SPONGE_NB`, `T_MAX`, etc.)
* **Model building functions:** (e.g., `create_layers`, `build_layered_salt_lens_model`, `load_bp_subsection`)
* **Numerical kernels:** (`update_vx_*`, `update_vz_*`, `update_p_*` compiled with Numba)
* **Post-processing and plotting functions**
* **A driver function:** `run_figureX(...)` for that specific figure
* **Execution block:** A small block at the end that actually calls the driver and saves the figure.

Where possible, the same functions are reused between figures (e.g., FD kernels, Ricker wavelet, sponge boundaries, GA coefficients). Therefore, it is highly recommended to **run the notebook top-to-bottom** to ensure all definitions exist before executing later figures.

---












---


## References

- Finkelstein, B., and R. Kastner, 2006, *Finite-difference time domain dispersion reduction schemes*: Journal of Computational Physics, **221**(1), 422–438.

- Igel, H., B. Riollet, and P. Mora, 1992, *Accuracy of staggered 3-D finite-difference grids for anisotropic wave propagation*: SEG Technical Program Expanded Abstracts, 1244–1246.

- Kelly, K. R., R. W. Ward, S. Treitel, and R. M. Alford, 1976, *Synthetic seismograms: A finite-difference approach*: GEOPHYSICS, **41**(1), 2–27.

- Liu, Y., and M. K. Sen, 2011, *Scalar wave equation modeling with time-space domain dispersion relation-based staggered-grid finite-difference schemes*: Bulletin of the Seismological Society of America, **101**(1), 141–159.

- Vanga, M., and M. Ojha, 2025, *Modeling the seismic wave equation using a staggered grid finite-difference method optimized with a genetic algorithm*: Journal of Seismic Exploration, **34**(2), 1–13.













