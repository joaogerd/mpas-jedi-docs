# FGAT64 MPASstatic observer isolation np1 results

## Objective

Test the isolated 3D-FGAT runtime in one MPI rank with MPASstatic covariance and one observer at a time, after the control case with MPASstatic and no observers passed. This separates the basic FGAT/model/runtime path from the observation/HofX/GeoVaLs/IODA path.

Operational workflow files and main submission scripts were not changed. No 64-rank tests were run in this step.

## Files prepared

YAMLs created under:

`/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/`

- `3dvar_fgat.fgat64.mpastatic_aircraft_only_np1.yaml`
- `3dvar_fgat.fgat64.mpastatic_sondes_only_np1.yaml`
- `3dvar_fgat.fgat64.mpastatic_sfc_only_np1.yaml`

PBS scripts created under:

`/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/`

- `run_fgat64_mpastatic_aircraft_only_np1.pbs`
- `run_fgat64_mpastatic_sondes_only_np1.pbs`
- `run_fgat64_mpastatic_sfc_only_np1.pbs`

All three YAMLs are derived from the MPASstatic FGAT np1 case. They keep the same runtime, same FGAT window, same model, same MPASstatic covariance, and retain filters and obsdataout for the selected observer.

## Static validation

Checks performed before submission:

- YAML parsing with `yaml.safe_load`: OK for all three YAMLs.
- PBS syntax with `bash -n`: OK for all three PBS scripts.
- No PBS contains internal `qsub`.
- No PBS uses `#PBS -V`.
- All PBS scripts set `OMP_NUM_THREADS=1` and `GFORTRAN_CONVERT_UNIT=big_endian:101-200` explicitly.
- Each YAML contains only its expected observer.
- Each YAML uses `covariance model: MPASstatic`.
- No created YAML contains `SABER`, `BUMP`, `NICAS`, or `VBAL` in the covariance block.

## Submitted case

Only the first case was submitted, as requested.

- Case: MPASstatic Aircraft-only np1
- Job id: `258336.pbs-ha`
- PBS: `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/run_fgat64_mpastatic_aircraft_only_np1.pbs`
- YAML: `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_aircraft_only_np1.yaml`
- Working directory: `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test`
- Binary: `/p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x`
- MPI launcher: `/opt/cray/pals/1.6/bin/mpiexec`
- Ranks: `1`
- Command:

```bash
/opt/cray/pals/1.6/bin/mpiexec -n 1 \
  /p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x \
  /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_aircraft_only_np1.yaml
```

## Aircraft-only result

PBS log:

`/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/logs/fgat64_mpastatic_aircraft_only_np1.pbs.out`

Runtime logs present after the run:

- `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/log.atmosphere.0000.out`
- `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/log.atmosphere.0000.d0001.out`

Runtime logs not present:

- `log.atmosphere.0000.err`
- `mpasjedi_variational.log`

Relevant log evidence:

```text
Aircraft: read database from obs/aircraft_obs_2018041500.h5 (io pool size: 1)
Aircraft processed vars: 5 Variables: airTemperature, specificHumidity, virtualTemperature, windEastward, windNorthward
Aircraft assimilated vars: 4 Variables: airTemperature, windEastward, windNorthward, specificHumidity
No bias-correction is performed for this ObsSpace.
Model:forecast: forecast finished:
cn-0006.head.cm.cptec.inpe.br: rank 0 died from signal 11 and dumped core
[INFO] command exit status: 139
```

The crash occurs after the MPAS forecast finishes and before any successful final analysis/feedback write from this Aircraft-only case. There is no explicit `CRITICAL ERROR`, `MPI_Abort`, or Fortran diagnostic in the captured logs.

The MPAS runtime warnings about `pressure(1) < pressure(2)` still appear. These warnings also appeared in previous cases and are not sufficient by themselves to explain the crash, because the MPASstatic no-observers np1 case completed successfully.

## Output files

Existing file in `analysis/`:

```text
2026-06-04 14:26:39 31716080 analysis/analysis.2018-04-15_00.00.00.nc
```

This timestamp corresponds to the previous MPASstatic no-observers successful run, not to the Aircraft-only run. Therefore, the Aircraft-only run did not create a new analysis file.

No feedback file was produced by the Aircraft-only run.

## Matrix status

| case | observer | ranks | covariance | result | failure point | analysis generated | feedback generated | conclusion |
|---|---:|---:|---|---|---|---|---|---|
| `mpastatic_aircraft_only_np1` | Aircraft | 1 | MPASstatic | Failed, status 139 | After forecast, rank 0 `signal 11`; before clear GeoVaLs/HofX/IODA completion messages | No new file; only previous no-observers analysis remains | No | Aircraft observer path is enough to trigger the active crash. |
| `mpastatic_sondes_only_np1` | Radiosonde | 1 | MPASstatic | Not submitted | Stopped because Aircraft failed first | Not applicable | Not applicable | Pending manual decision. |
| `mpastatic_sfc_only_np1` | SfcCorrected | 1 | MPASstatic | Not submitted | Stopped because Aircraft failed first | Not applicable | Not applicable | Pending manual decision. |

## Interpretation

The previous MPASstatic no-observers np1 case passed, created analysis output, and ended normally. The Aircraft-only MPASstatic np1 case fails after the same forecast stage with `signal 11`.

This proves that the basic FGAT/model/runtime/MPASstatic path is functional in np1, and that enabling the Aircraft ObsSpace is sufficient to trigger the crash. At this stage the most likely fault domain is not MPI rank count and not SABER/BUMP/VBAL. It is the Aircraft observation path, including one or more of:

- Aircraft IODA content or layout;
- Aircraft `VertInterp` operator inputs;
- GeoVaLs/HofX generation for Aircraft variables;
- Aircraft filters, especially `Background Check` after forecast;
- Aircraft `obsdataout` feedback writing;
- mismatch between required GeoVaLs and available model/state variables.

## Recommended next step

Do not move directly back to 64 ranks yet. The next diagnostic should stay in np1 and reduce only the Aircraft observer path, because Aircraft alone already reproduces the crash.

Recommended sequence, one job at a time:

1. Aircraft-only MPASstatic np1 with filters removed.
2. Aircraft-only MPASstatic np1 with `obsdataout` removed.
3. Aircraft-only MPASstatic np1 with one simulated variable group at a time, for example temperature only, winds only, humidity only.
4. Static comparison of `obs/aircraft_obs_2018041500.h5` against the official MPAS-JEDI Aircraft IODA used by passing reference tests, including groups, variable names, dimensions, missing values, time window, pressure/height coordinates, and metadata required by `VertInterp`.

Only after the Aircraft np1 path is understood should Radiosonde, SfcCorrected, or 64-rank tests be resumed.
