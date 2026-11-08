# L-RRM vs Parametric Synthetic Data — v8

## Scope

This is a **base-R** extension of the supplied v7 program. It retains its Gaussian repeated-measures simulation, oracle and plug-in L-RRM, oracle and plug-in parametric synthetic generation, DP-outcome and DP-score *reference* mechanisms, Monte Carlo inference, batch restart, and privacy attacks. v8 adds:

1. Synthetic covariance-shrinkage **sensitivity generators** (`shrink=0,0.25,0.5` by default). Shrinkage is a deliberately altered generator, **not** exact covariance preservation.
2. Same-replication paired **empirical membership-AUC matching** between each L-RRM gamma and each synthetic generator; a configurable AUC tolerance and a stricter confidence-interval-within-tolerance flag.
3. A **point-estimate Pareto frontier** minimizing RMSE and practical membership-inference AUC, excluding gamma=1.
4. Correct scenario-aware grouping and a multi-page privacy–utility PDF.
5. Unit/sanity checks for gamma=1, covariance, clipping, and AUC ranking.

**Not implemented:** matched formal-DP L-RRM versus formal-DP synthetic data. Neither plug-in method is DP. A matched DP claim requires separately developed and verified private parameter fitting, release mechanisms, and privacy composition. The existing DP outcome/score mechanisms remain **reference benchmarks**, not proof of DP for L-RRM or synthetic data.

## Requirements

- R 4.2+ recommended; **base R only**, no external packages.
- Substantial compute for 5,000 replications, bootstrap, and privacy attacks; start with smoke test.
- Script has **not been executed in this delivery environment** (Rscript unavailable). Check smoke-test outputs before a production run.

## Run

Save `LRRM_v8_full.R` and run from a terminal:

```bash
# Linux/macOS smoke test (4 scenarios x 10 replications)
LRRM_REPS=10 LRRM_BOOT=0 LRRM_PRIV_EVERY=2 LRRM_DP_SENS=false \
  LRRM_OUT=LRRM_v8_smoke Rscript LRRM_v8_full.R

# Standard run: 4 scenarios x 5000 replications
LRRM_REPS=5000 LRRM_BOOT=200 LRRM_BOOT_EVERY=10 \
  LRRM_PRIV_EVERY=10 LRRM_OUT=LRRM_v8_results \
  Rscript LRRM_v8_full.R

# Expanded grid: 4 sample sizes x 3 visit counts x 2 effects x 2 covariances
LRRM_FULL=true LRRM_REPS=5000 LRRM_OUT=LRRM_v8_full_results \
  Rscript LRRM_v8_full.R
```

On Windows PowerShell:

```powershell
$env:LRRM_REPS='10'
$env:LRRM_BOOT='0'
$env:LRRM_PRIV_EVERY='2'
$env:LRRM_DP_SENS='false'
$env:LRRM_OUT='LRRM_v8_smoke'
Rscript .\LRRM_v8_full.R
```

Alternatively in RStudio, set environment variables with `Sys.setenv(...)` and then `source('LRRM_v8_full.R')`.

## Configuration

| Variable | Default | Meaning |
|---|---|---|
| `LRRM_SEED` | `20261105` | Reproducible base seed |
| `LRRM_REPS` | `5000` | Monte Carlo replications per scenario |
| `LRRM_FULL` | `false` | Expanded sample size, visit, covariance grid |
| `LRRM_OUT` | `LRRM_v8_results` | Output directory |
| `LRRM_BATCH` | `25` | Batch size for restart |
| `LRRM_RESUME` | `true` | Reuse completed batches with identical configuration |
| `LRRM_BOOT` | `200` | Bootstrap draws per selected replication; `0` disables |
| `LRRM_BOOT_EVERY` | `10` | Bootstrap every kth replication |
| `LRRM_PRIV_EVERY` | `10` | Run attacks every kth replication |
| `LRRM_SYN_SHRINK` | `0,0.25,0.5` | Synthetic covariance shrinkage sensitivity grid; must include 0 |
| `LRRM_AUC_TOL` | `0.02` | Maximum absolute difference in empirical AUC for point matching |
| `LRRM_DP_EPS` | `0.9` | Reference DP epsilon (single alternative mechanism) |
| `LRRM_DP_DELTA` | `1e-5` | Reference DP delta |
| `LRRM_DP_CLIP` | `10` | DP outcome whole-trajectory L2 clipping bound |
| `LRRM_SCORE_CLIP` | `10` | DP score whole-patient L2 clipping bound |
| `LRRM_DP_SENS` | `true` | Generate DP reference sensitivity grid |
| `LRRM_DP_EVERY` | `10` | DP sensitivity every kth replication |
| `LRRM_EPS_GRID` | `0.3,0.5,0.9` | Reference DP epsilon alternatives |
| `LRRM_CLIP_GRID` | `5,10,20` | Reference DP clipping alternatives |

**Important:** changing any configuration requires a **new output directory**. The script rejects mismatched saved configurations.

## Primary outputs

- `utility_privacy_summary.csv`: scenario/method-level RMSE, bias, empirical SD, rejection rate, coverage, analytic/boot SE ratios, and privacy metrics.
- `paired_mse_comparison.csv`: paired squared-error differences versus the **baseline** plug-in synthetic generator.
- `lrrm_vs_synthetic_joint_comparison.csv`: baseline paired utility and privacy differences.
- `matched_privacy_comparison.csv`: matched-risk candidates for **all** synthetic sensitivity generators. `matched_point` indicates |mean AUC difference| <= tolerance. `matched_ci` requires the **entire approximate 95% CI** for the AUC difference to lie inside that tolerance. A point match alone is not evidence of equivalent privacy.
- `privacy_utility_frontier.csv`: point-estimate nondominated RMSE–AUC configurations. No uncertainty-aware dominance inference is made.
- `privacy_utility_tradeoff.pdf`: separate two-panel page per scenario (RMSE versus AUC; RMSE versus nearest-neighbor reconstruction MSE).
- `simulation_validation.csv`: sanity checks; all `pass` should be `TRUE`.
- `trial_level.csv`: per-replication raw simulation diagnostics.
- `dp_sensitivity_summary.csv`, `dp_sensitivity_trial_level.csv`, `dp_reference_diagnostics.pdf`: existing alternative DP benchmark diagnostics (if enabled).
- `configuration.txt`, `scenario_manifest.csv`, `session_info.txt`, `LIMITATIONS.txt`: reproducibility and limitations.

## Interpreting empirical privacy

- Practical membership-inference AUC near **0.5** is chance-level; higher is more successful attack discrimination. Compare methods under the **same attack**, same scenario, and same evaluated replications.
- Nearest-neighbor reconstruction MSE is an **attack error**: higher typically means harder reconstruction, unlike estimator RMSE, for which lower is better.
- Patient row-aligned linkage is defined only for L-RRM; do not assign synthetic data an artificial zero linkage rate.
- Matching AUC does **not** match every privacy risk, and is **not** differential privacy.
- AUCs based on relatively few privacy evaluations can be noisy; increase `LRRM_REPS` and/or decrease `LRRM_PRIV_EVERY`, and inspect `privacy_pairs` and AUC CIs.
- No AUC matches may exist. In that case, report **no overlap**, not a fabricated matched comparison.

## Statistical caveats

- Oracle preservation holds only under specified Gaussian model and known parameters; plug-in release and model fitting alter repeated-sampling behavior.
- Analytic plug-in SE multipliers are working approximations. Validate `mean_se`, `se_ratio`, Type I error, and coverage; use the selected parametric bootstrap as an additional diagnostic, not as a universal correction.
- Synthetic shrinkage variants deliberately change within-patient covariance. They are sensitivity generators, not automatically valid replacements for the target longitudinal data law.
- The practical attack uses a public zero-interaction baseline and public covariance; oracle attack uses true parameters. Realistic attack performance depends on adversary knowledge.
- DP outcome and score benchmarks assume fixed public design and public covariance. The clipped score variance is a working approximation and clipping can bias estimation.
- The `trial_level.csv` and other diagnostics are computed on simulated confidential trajectories. **Do not publish analogous diagnostics on real confidential patient data without a separate privacy assessment.**
- The full grid may be computationally expensive. Batch files help restart, but do not guarantee transactional integrity of all aggregated output files.

## Manuscript reporting suggestions

1. Report paired utility RMSE and Monte Carlo confidence intervals.
2. Report both privacy AUC difference and its confidence interval, plus attack evaluation counts.
3. Restrict the main matched-risk table to `matched_ci=TRUE` if adequate sample size and overlap exist; otherwise explicitly report insufficient evidence of matched risk.
4. Show point-estimate frontier plots with uncertainty caveats; do not claim statistically established dominance from point estimates.
5. Keep DP reference comparisons in a distinct section; do **not** label the plug-in L-RRM or plug-in synthetic generators as DP.

## Version notes

v8 is derived from the supplied v7 code. Its simulation, DP reference and inference components are largely retained; matched empirical risk, synthetic sensitivity generators, plotting, and checks are additions. Before manuscript submission, perform a smoke test, independently verify inference and attack calculations, and inspect complete outputs.
