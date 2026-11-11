GRD v12.5: Independent computational verification

1. Extract the ZIP into a new folder.
2. In RStudio open GRD_v12_5_RUN_INDEPENDENT_VERIFICATION.R.
3. Click Source (not Run line). No 300,000-trial calibration is rerun.
4. Inspect GRD_v12_5_independent_results/independent_verification_summary.csv
   and independent_verification_per_trial.csv.
5. Default: 100 synthetic trial datasets x 9 scenarios x 4 eta = 3600 trials.
   To smoke test first, run options(grd_verifier_replicates=2L)
   in the Console before pressing Source. For a larger study use 200L.

The primary Wald function is extracted from the bundled exact v12.4
source by parsing, without running the whole script. The separately
implemented verifier receives routed labels and outcomes, NOT original
assignments. Frozen boundaries come from the user-uploaded v12.4 audit.
This is a numerical agreement test, not independent replication of the
nuisance-robust Type I simulation, not independent data-generation code,
and not evidence of prospective DMC turnaround. No agreement is claimed
until this script runs successfully in RStudio.

v12.5.1 HOTFIX: Replaced tail(looks,1) with looks[length(looks)] and
changed function environments to inherit from the R session search path,
so stats functions (e.g. cov, eigen) are available. The original isolated
baseenv() did not expose stats package functions. No calibration rerun.
