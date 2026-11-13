# FSD adaptive substep — research evidence (2026-11-14 JST)

## State: PARK / NOT END-TO-END REPRODUCED

Upstream Icepack main SHA observed: `9a1f3e4687f298f87f14a5eea5a57aca05faaf85`.
The CI workflow fetches upstream HEAD at runtime and saves the exact fetched SHA as an artifact; the observed SHA above was independently checked after runs.

### Observation

In `columnphysics/icepack_therm_itd.F90`, `lateral_melt` calculates:
```fortran
subdt = get_subdt_fsd(afsd_tmp(:), d_afsd_tmp(:))
subdt = MIN(subdt, dt)
...
elapsed_t = elapsed_t + subdt
```
Therefore, if a later subcycle exists with `elapsed_t > 0` and `subdt > dt-elapsed_t`, the integration exceeds the requested interval. The one-line candidate is `MIN(subdt, dt-elapsed_t)`.

This is a valid *conditional* source-level claim, **not** a demonstrated error under ordinary sea-ice forcing.

### Real upstream driver tests

The research CI clones and builds the **official standalone Icepack driver** with the Consortium's `icepack.setup`, `icepack.build`, and `icepack.submit` scripts, using the upstream `fsd12` option. The only changes to Icepack source in the CI jobs are a diagnostic inside the target loop and, for the corrected job, the proposed one-line change.

- [Run 43492038321](https://github.com/Johnny-Kao/icepack-fsd-timestep-research/actions/runs/43492038321): regular FSD12 smoke test; baseline and corrected build/run/test PASS; no observed `FSD_LATM_OVERSHOOT`.
- [Run 43492818833](https://github.com/Johnny-Kao/icepack-fsd-timestep-research/actions/runs/43492818833): stress experiment with 24-hour model step and summer-season start; both jobs PASS, no observed `FSD_LATM_OVERSHOOT`; its attempted namelist display used missing `rg` and was not trustworthy.
- [Run 43493195805](https://github.com/Johnny-Kao/icepack-fsd-timestep-research/actions/runs/43493195805): corrected verification of actual case namelist and branch entry. `dt=86400`, `npt=24`, `istep0=180`, `tr_fsd=.true.` verified in the **generated case namelist**. Both builds/runs/tests PASS. **206** `FSD_LATM_ACTIVE` events in the unmodified baseline, **206** in the one-line-corrected version; **zero** `FSD_LATM_OVERSHOOT` events across both.

### Interpretation and limitations

The genuine full-driver FSD lateral-melt branch is reachable and executed, but **the full driver has not reproduced the hypothesized overrun**. Normal rates in the tested cases may not require multiple substeps. Absence of a diagnostic in these finite cases does not prove impossibility for every configuration.

An earlier standalone mathematical/Fortran demonstration used manually constructed FSD states and high melt forcing, yielding an overshoot, but it is **not** sufficient to establish Icepack driver reachability or ordinary production impact. Do not report it as a verified production bug.

CI pass means a smoke test completed successfully, **not** proof of bit-for-bit equivalence in non-triggering configurations or full formal regression coverage.

### Decision

**PARK — do not submit an issue, PR, or move code to Staging.**

Reopen only upon (1) a defensible, physically/operationally reachable forcing case that produces `FSD_LATM_OVERSHOOT` inside the official driver, or (2) evidence from an established full-driver workload with multiple FSD substeps. Then run exact Red/Green plus conservation and model result comparisons.

CI was changed to `workflow_dispatch` only to avoid automatic future research compute.
