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

The notebook is designed so that anyone can open it in Colab or Jupyter NB and reproduce all figures from 1 to 14 with minimal manual editing.

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

## 5. High-order FD coefficients and GA tables

Several figures rely on the same coefficient functions and GA tables:

* `fornberg_weights(...)`
* `conventional_coefficients_fornberg(M)`
* `M = 20` → 40th-order half-operator (reference)
* `M = 6` → 12th-order half-operator (conventional)


* `GA12_COEFFS` → GA-optimized single 12th-order set
* `VEL_LEVELS` and `GA_RANGE_TABLE` → GA range-table as a function of velocity.

These definitions appear near the top of the modeling figures (e.g., Figures 9–12) and are reused in downstream cells (Figures 11, 12, 14).
**Important:** Do not delete or re-run these cells out of order; keep this block executed before running downstream figures.

---

## 6. Numba kernels

For performance, spatial updates are accelerated using Numba JIT compilation:

* `update_vx_uniform`, `update_vz_uniform`, `update_p_uniform`
* `update_vx_var`, `update_vz_var`, `update_p_var`

Each has two modes:

* **uniform:** Applies the same coefficient set everywhere (used for 40th-order, conventional 12th-order, and the single-set GA 12th-order).
* **var:** Coefficients are dynamically selected per grid cell using a velocity class (`velcls`) cross-referenced with the `GA_RANGE_TABLE`.

*Note: On the first run, Numba will compile the kernels, which can take a moment. Subsequent runs will be significantly faster.*

---
## 7. Caching shot gathers to NPZ

For the computationally heavy modeling figures, simulation runs are cached as compressed `.npz` files (NumPy’s compressed format) to avoid rerunning expensive operations.

* **Example output directories:** `fig9_outputs/`, `fig10_outputs/`, `fig11_outputs/`, `fig12_outputs/`, `fig14_outputs/`
* **Toggle variable:** `LOAD_PREVIOUS = False` (or `True`)

**Workflow:**

1. **First time:** Set `LOAD_PREVIOUS = False`. The code runs the simulation, saves `*.npz` files in the output directory, and then plots the results.
2. **Later runs:** Set `LOAD_PREVIOUS = True`. The code will load the existing `.npz` files and skip the expensive modeling stage.

If you want to regenerate everything from scratch (e.g., after changing model parameters), you must either delete the corresponding `.npz` files or set `LOAD_PREVIOUS = False` again.

---

## 8. Per-figure usage guide

### Figures 1–7 (Dispersion / Stability / 1D tests)

These early figures define wavenumber/angle ranges, compute numerical vs. analytical dispersion or stability curves, and plot phase-velocity error and CFL stability.

* **Usage:** Run the definition cells, then run the plotting cells sequentially. Because these are 1D or low-cost computations, they run quickly and do not require the SEGY file. Each cell ends with `plt.savefig("figureX.png")` and `plt.show()`.

### Figure 8 – Velocity models used in the study

* **Mechanism:** Loads the BP SEGY file, extracts the 33–41 km window, applies piecewise depth remapping, and constructs both the synthetic layered cartoon (Left/Fig. 8A) and the remapped BP salt-dome velocity model (Right/Fig. 8B).
* **Usage:** Ensure `file_path` correctly points to the uploaded SEGY file. Run the cell and verify that `figure8.png` correctly displays both models with matching colorbars.

### Figure 9 – Shot gathers in the horizontal layered model

* **Mechanism:** Uses the layered model from Figure 8A. Simulates the four FD schemes and records a single shot gather (`p` or `vz`) at the source depth. Preprocesses the data with `preprocess_time_gain_gamma(...)` and plots it in a 3×4 grid.
* **Usage:** Ensure common functions are defined. Set `COMPONENT = "p"` or `"vz"`. For the first run, set `LOAD_PREVIOUS = False` and execute `gathers_raw_9, gathers_proc_9 = run_figure9()`.

### Figure 10 – Shot gathers in the BP velocity model

* **Mechanism:** Extracts the 33–41 km window from the BP SEGY and resamples it to 10 m × 10 m. Simulates the four schemes through the complex salt-dome model, plotting in a 3×4 layout.
* **Usage:** Verify `BP_SEGY_PATH`. Set the desired component, manage the `LOAD_PREVIOUS` toggle, and execute `gathers_raw_10, gathers_proc_10 = run_figure10()`.

### Figure 11 – Layered model: wavefields + single receiver trace

* **Mechanism:** Runs a full simulation for all 4 schemes on the layered velocity model. Stores a wavefield snapshot at a chosen time (e.g., `SNAPSHOT_TIMES = [2.6]`) and a single receiver trace. Plots a 4×2 layout (snapshots on the left, normalized traces on the right).
* **Usage:** Execute `run_figure11_for_snapshot(2.6)`. For the animation (which creates a 1×3 movie excluding the conventional 12th-order to save memory), run `anim = animate_figure11_layered_3panels(...)` followed by `HTML(anim.to_jshtml())`.

### Figure 12 – Wavefields + trace in BP model

* **Mechanism:** Same concept as Figure 11, applied to the resampled 10 m × 10 m BP salt-dome model.
* **Usage:** Execute `run_figure12_for_snapshot(2.6)`. To generate the animation showing wave propagation for the 40th-order and GA 12th range-table schemes, run `anim12 = animate_figure12_bp(...)`.

### Figure 13 – Layered + salt-lens velocity model

* **Mechanism:** Builds a synthetic background model matching Figure 8A but embeds an elliptical salt lens (4500 m/s) centered around (4 km, 3.8 km).
* **Usage:** Call `plot_figure13_velocity()` and verify the salt lens is visible in the resulting `figure13.png`.

### Figure 14 – Layered + salt-lens wavefields + trace

* **Mechanism:** Uses the synthetic model from Figure 13 to run wave propagation for the four schemes, outputting a 4×2 layout similar to Figures 11 and 12.
* **Usage:** Set `COMPONENT_A1` and `SNAPSHOT_TIMES_A1`, then run `run_figure14_for_snapshot(2.6)`.

---
## 9. Working with animations in Colab

For Figures 11, 12, and 14, animations are built using `matplotlib.animation.FuncAnimation` and displayed via `IPython.display.HTML(anim.to_jshtml())`.

**Important Memory Notes:**

* Colab has a strict inline animation size limit (`rcParams["animation.embed_limit"]`). The code automatically sets this slightly lower (100–120 MB) to prevent session crashes.
* If Colab crashes or runs out of memory while animating, try the following:
* Animate three or fewer schemes at a time.
* Increase `frame_stride` to skip more time steps (resulting in fewer frames).
* Reduce `t_end` to shorten the simulation time.

---
## 10. Recommended workflow in Colab

1. Open the notebook in Google Colab.
2. Install necessary packages in the first cell (`!pip install segyio numba`).
3. Upload the SEGY file `vel_z6.25m_x12.5m_exact.segy` directly to `/content/`.
4. Run all setup cells sequentially (coefficients, GA tables, Numba kernels, helper functions).
5. Run the figures in order.
* *If you only care about the wavefield modeling, you can skip the dispersion/stability charts (Figures 1–7) and start directly from Figure 8.*


6. For heavy figures (9–12, 14), start with `LOAD_PREVIOUS = False` for the first run, then switch to `True` for subsequent adjustments to the plots.
7. Download the saved PNGs (`figureX*.png`) and animations for your final report or paper.

---
## 11. Troubleshooting

* **`NameError: name '...' is not defined`**
* **Cause:** You attempted to run a figure cell before executing the cell that defines its dependencies (e.g., `build_layered_salt_lens_model`, `sponge_masks`).
* **Fix:** Run the notebook from the top, or at least re-run the specific definition cells.


* **`FileNotFoundError` for SEGY**
* **Cause:** The SEGY path is incorrect, or the file has not finished uploading to Colab.
* **Fix:** Ensure the file is completely uploaded to `/content/` or manually adjust `BP_SEGY_PATH` to point to the correct directory.


* **Colab crashes / "Out of Memory"**
* **Cause:** The simulations are too large, or the HTML animations have exceeded browser memory constraints.
* **Fix:** Increase `frame_stride`, reduce `t_end`, temporarily disable some schemes (e.g., skip the conventional 12th-order), or set `LOAD_PREVIOUS = True` to avoid holding too much simulation data in active memory.


---

## References

- Finkelstein, B., and R. Kastner, 2006, *Finite-difference time domain dispersion reduction schemes*: Journal of Computational Physics, **221**(1), 422–438.

- Igel, H., B. Riollet, and P. Mora, 1992, *Accuracy of staggered 3-D finite-difference grids for anisotropic wave propagation*: SEG Technical Program Expanded Abstracts, 1244–1246.

- Kelly, K. R., R. W. Ward, S. Treitel, and R. M. Alford, 1976, *Synthetic seismograms: A finite-difference approach*: GEOPHYSICS, **41**(1), 2–27.

- Liu, Y., and M. K. Sen, 2011, *Scalar wave equation modeling with time-space domain dispersion relation-based staggered-grid finite-difference schemes*: Bulletin of the Seismological Society of America, **101**(1), 141–159.

- Vanga, M., and M. Ojha, 2025, *Modeling the seismic wave equation using a staggered grid finite-difference method optimized with a genetic algorithm*: Journal of Seismic Exploration, **34**(2), 1–13.












