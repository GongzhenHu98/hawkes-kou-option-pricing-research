# When Does Self-Excitation Matter for Option Pricing?

**Affine Hawkes–Kou pricing, approximation error, and economic significance**

> **Research status — work in progress.** This repository is a transparent research log, not a production pricer or a trading strategy. The weak-excitation approximation is currently **formal rather than fully rigorous**: no complete remainder bound has been established, and the numerical validation is still being built.

## Research question

When does Hawkes self-excitation add statistically identifiable, numerically reliable, and economically meaningful value beyond an ordinary Kou jump model?

The project compares the following model ladder:

1. Black / EWMA baseline;
2. Kou jump diffusion;
3. Hawkes–Kou jump diffusion;
4. formal weak-excitation approximation.

The goal is not to assume that the most complex model wins. The goal is to identify the regions in which additional complexity is justified by pricing accuracy, Greeks, calibration stability, and computational cost.

## Research path

The project started from a path-dependent volatility (PDV) model with an exponentially decaying memory state and a jump pulse. Rolling out-of-sample tests showed that the complex PDV model was weaker than EWMA in most calm regimes. The main problems were:

- the continuous-diffusion pricing operator did not explicitly price jump risk;
- memory and pulse parameters were strongly collinear;
- threshold-based jump detection was unstable;
- the model could not generate a robust skew or smile.

This motivated a move to an explicit Hawkes–Kou framework: the Hawkes intensity models jump clustering, while the double-exponential Kou distribution models asymmetric tails.

## Important rigor statement

The original exploratory approximation replaced the zero-order Kou model with a Black effective-variance proxy and expressed the correction through higher-order Greeks. It is retained only as a **legacy error-analysis case study**.

Two material issues have already been identified:

1. At zero self-excitation, the model remains a full Kou jump model. Replacing it by a Black variance proxy loses skewness and tail information.
2. The original first-order time kernel omitted an additional time integration and materially overstated the short-maturity self-excitation correction.

The current formal expansion is

\[
V^\delta = V_0 + \delta V_1 + O(\delta^2),
\]

but it should not yet be read as a theorem: the remainder bound and full convergence proof are not complete. The benchmark is being rebuilt using the affine characteristic function, COS/Fourier inversion, and event-driven Monte Carlo with confidence intervals.

See [`docs/known_issues.md`](docs/known_issues.md) for the full status.

## Model

Under the risk-neutral measure,

\[
\frac{dF_t}{F_{t^-}}
= \sigma\,dW_t - \lambda_t m\,dt
+ \int_{\mathbb R}(e^y-1)N(dt,dy),
\]

\[
d\lambda_t = \kappa(\lambda_\infty-\lambda_t)dt + \delta\,dN_t,
\qquad m=\mathbb E[e^Y-1].
\]

The jump size follows a double-exponential Kou distribution. The full model admits an affine characteristic-function representation

\[
\Phi^\delta(u)
= \exp\left(iu\log F_t + A^\delta(\tau,u) + B^\delta(\tau,u)\lambda_t\right),
\]

where \(A^\delta\) and \(B^\delta\) solve Riccati-type ODEs.

## Current implementation status

| Component | Status |
|---|---|
| Parameter validation and Kou moments | Implemented |
| Kou characteristic function | Implemented |
| Full affine characteristic function via ODE | Implemented as a research benchmark |
| Closed-form zero-self-excitation characteristic function | Implemented |
| Formal first-order characteristic-function correction | Implemented and explicitly marked formal |
| Sanity and limiting-case tests | Initial version implemented |
| COS pricer | Planned for v0.2 |
| Event-driven Monte Carlo with confidence intervals | Planned for v0.3 |
| Calibration and identifiability study | Planned |
| Production latency or trading PnL claims | **Not claimed** |

## Repository layout

```text
.
├── docs/                         # model, known issues, roadmap, reproducibility
├── src/hawkes_kou/
│   ├── model.py                  # parameter objects and validation
│   ├── pricing/
│   │   ├── characteristic.py     # Kou and affine characteristic functions
│   │   └── cos.py                # COS interface placeholder
│   ├── simulation/               # event-driven MC workstream
│   └── exploratory/              # legacy approximations, clearly isolated
├── scripts/                      # reproducible command-line checks
├── tests/                        # mathematical and limiting-case tests
├── data/                         # synthetic/public data only
└── figures/                      # generated outputs only
```

## Quick start

```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -e ".[dev]"
pytest
python scripts/run_sanity_checks.py
```

## Planned releases

- **v0.1 — Transparent research log:** model specification, affine CF benchmark, formal approximation, known issues, and sanity tests.
- **v0.2 — Affine pricing benchmark:** COS pricer and convergence tests.
- **v0.3 — Numerical validation:** event-driven MC, standard errors, confidence intervals, and error decomposition.
- **v1.0 — Research report:** reproducible conclusions on when self-excitation is economically meaningful.

## Data and confidentiality

Only synthetic data, public data, or data with explicit permission may be added. Company market data, internal code, internal slides, and confidential parameters must not be uploaded.

## License

MIT for original code in this repository. Data and third-party material may have separate terms.
