# Icepack FSD timestep research

Independent reproducibility work for the FSD adaptive substep bound in `columnphysics/icepack_therm_itd.F90` (and the analogous wave-fracture loop).

Scope: Research only; nothing is submitted to CICE-Consortium/Icepack.

Hypothesis: `min(subdt, dt)` can step past the remaining integration interval after the first FSD substep. Proposed candidate: `min(subdt, dt-elapsed_t)`.

The GitHub Actions workflow clones the upstream Icepack repository, builds the **official standalone Icepack driver** using the upstream `icepack.setup` and `icepack.build` scripts, then runs baseline and candidate configurations with FSD enabled. It records the exact upstream commit and log output.

An ordinary smoke-test pass is NOT proof of reachability: the diagnostic records whether an actual end-to-end run triggers substep overshoot. No claims of production reachability are made before observing that log.
