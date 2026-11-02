# GRD---continous

GRD v7: primary semi-blinded interim efficacy monitoring

1. Extract the ZIP to a writable folder.
2. In RStudio, set that folder as the working directory, then run:
   source("GRD_semiblinded_v7.R")
   This defaults to SMOKE mode (36 panels); implementation checks only.
3. For manuscript-scale simulation, use Terminal:
   Rscript GRD_semiblinded_v7.R full 300000 50000 500000
   Full mode has 72 panels plus calibration and fixed-look diagnostics.
   Do not launch the full run by pasting a shell command into the R console.
   Alternatively, change RUN_MODE <- "full" in the script and source it.
4. Check run_status.csv, completion_audit.csv and COMPLETE.txt.

Main changes
- Primary rule RG is a separate function accepting no original-label data.
- The synthetic DMC dry run stops at the first valid GRD crossing.
- No original-label efficacy statistic is computed in that dry run.
- Original-label monitoring remains a paired simulation reference.
- RC remains secondary, with its own stopping and confirmation workload.
- GRD boundary fitting continues to use GRD first crossings alone.
- GRD numerical failures and unresolved final analyses are reported separately.
- Added invariance tests: changing original-label results cannot change RG.
- New checksum-based v7 folders prevent reuse of incompatible v6 checkpoints.
- Existing calibration seed blocks and data generation are retained.

Primary outputs
primary_GRD_metrics.csv: RG Type I error/power, Wilson CI, MCSE, ESS,
ESS MCSE, first-look and early stopping, reached failures, unresolved analyses.
primary_GRD_null_sensitivity.csv: primary null results with the existing
all-policy Bonferroni family intervals retained (216 null rows in full mode).
GRD_reference_comparison.csv: paired decision disagreement and capped sample
size differences; these do not represent original-label operational access.
example_*_DMC_report.csv: GRD-only synthetic report.
secondary_confirmation_metrics.csv: RC results only.
confirmation_and_delay.csv: SECONDARY RC operations, not RG stopping delays.
policy_metrics.csv: all procedures, with explicit analysis_role column.

Scope and validation
This script covers the additional IBS study, not all initial 24 configurations.
No new simulation results have been generated here. No R runtime was available
in the editing environment. Static delimiter/string and primary-path isolation
checks passed; embedded R self-tests and a smoke run must pass in R before use.
No external packages are required. The full run can take substantial time.
Existing v6 GRD results may remain applicable if independently verified, but
v7 deliberately does not import old checkpoints automatically.
Finite-grid validation does not establish uniform nuisance-robust control.
Fixed-look confidence intervals are not post-stopping intervals.
Synthetic routing service records contain original assignments and must be
kept separate from routed releases. Technical timings are not DMC feasibility.
