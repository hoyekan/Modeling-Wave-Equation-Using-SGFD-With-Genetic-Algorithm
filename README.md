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
