# CLT Experiments for KL_inf and Stopping Times

Simulation code for the central limit theorem (CLT) of the empirical
KL-divergence infimum and of the associated sequential stopping time
τ<sub>α</sub>. Two notebooks reproduce the synthetic and real-data
figures.

## Background

For a distribution `q` on `[0, 1]` and a threshold `m₀`, the
KL-divergence infimum is

```
KL_inf(q, m₀) = inf { KL(q, q')  :  E_{q'}[X] ≥ m₀ }
              = sup { E_q[log(1 - λ(X - m₀))]  :  λ ∈ [0, 1/(1 - m₀)) }   (dual)
```

The estimator plugs the empirical measure `q̂_n` of `X₁,…,X_n` into
the dual and solves the one-dimensional concave program. Writing
`L = KL_inf(q, m₀)` and `L̂_n = KL_inf(q̂_n, m₀)`, the CLTs studied
here are

```
√n  (L̂_n − L)                                  ⇒  N(0, σ²(q, m₀))
√log(1/α) · (τ_α / log(1/α) − 1/L)              ⇒  N(0, σ²_bd(q, m₀))
```

where `τ_α = inf { n ≥ 1 : n · L̂_n ≥ β(n, α) }` and
`σ²_bd = σ² / L³`.

## Files

| File | What it does |
| --- | --- |
| `kl_inf.ipynb` | Simulates `W_n = √n (L̂_n − L)` for Bernoulli(0.6) and Beta(3, 2). Plots histograms with the `N(0, σ²)` overlay for `n ∈ {1000, 2000}`. |
| `stopping_time.ipynb` | Four experiments: (1) τ<sub>α</sub> CLT under the relaxed threshold `β(n, α) = log(1/α)`; (2) same under the refined threshold `β(n, α) = 1 + log(2(1+n)/α)`; (3) real-data version that bootstraps from `icml.xlsx`; (4) two 95% confidence intervals for τ<sub>α</sub> — empirical and single-path (Lemma 4.4). |

## Computing KL∞ in practice

Both notebooks share the same numerical routine for the dual problem:

1. Compute `Ξ = mean((1 − m₀) / (1 − X))` on clipped data.
2. If `Ξ < 1`, the optimum sits on the boundary `λ* ≈ 1/(1 − m₀)`.
3. Otherwise, solve `Ψ(λ) = −mean((X − m₀) / (1 − λ(X − m₀))) = 0` —
   first with Brent's method on a sign-checked bracket, falling back
   to `fsolve` with a midpoint warm-start.
4. Plug `λ̂` back into the dual to get `L̂ = mean(log(1 − λ̂(X − m₀)))`.

For Bernoulli inputs the analytic form `d(p, q) = p log(p/q) + (1−p)
log((1−p)/(1−q))` and its derivative are used directly, which is
faster than the generic dual solver.

## Experiments at a glance

**`kl_inf.ipynb`** — `m₀ = 0.75`, `n ∈ {1000, 2000}`, 5000 paths per
setting, 400 000 Monte-Carlo samples for the population quantities.
Distributions: Bernoulli(0.6) and Beta(3, 2). Output figures:
`bernoulli_clt_klinf.pdf`, `beta_clt_klinf.pdf`.

**`stopping_time.ipynb`** — Bernoulli setting uses `p = 0.6`,
`m₀ = 0.2`, `α ∈ {10⁻⁴, 10⁻⁸}`, 5000–10 000 paths, `n_max =
2 000 000`. Real-data setting uses column `Arm_1` of `icml.xlsx`
(rescaled to `[0, 1]`), `m₀ = 0.5`, `α = 10⁻⁴`, 3000 bootstrap paths,
`n_max = 10 000`. Output figures: `clt_stopping_relaxed_beta.pdf`,
`clt_stopping_beta.pdf`, `real_data_stopping_time.png`.

The CI cell prints, for each α, both confidence intervals on the
τ-scale and overlays them as shaded bands on the rescaled histogram.

## Requirements

```
python >= 3.10
numpy
scipy
matplotlib
pandas       # only for the real-data cell
openpyxl     # only for the real-data cell (reads .xlsx)
```

The real-data cell in `stopping_time.ipynb` includes a
`google.colab.files.upload()` call. Outside Colab, delete that line
and point `pd.read_excel(...)` at a local copy of `icml.xlsx`.

## Running

```bash
pip install numpy scipy matplotlib pandas openpyxl
jupyter notebook kl_inf.ipynb         # ~1–2 min
jupyter notebook stopping_time.ipynb  # synthetic cells: minutes;
                                      # real-data cell: longer,
                                      # scales with n_paths × n_max
```

The stopping-time simulations recompute `KL_inf` at every `n` along
each path, so wall-clock time is dominated by `n_paths × E[τ_α]`.
Reduce `n_paths` or `n_max` for a faster pass.

## Reproducibility

Every randomized routine takes an explicit `seed` / `base_seed`. Path
`r` of a simulation uses `base_seed + r`, so individual paths can be
replayed in isolation. The population-quantity Monte Carlo uses a
separate seed (default `123`) to keep `L`, `λ*`, `σ²` fixed across
runs that vary only the path seeds.
