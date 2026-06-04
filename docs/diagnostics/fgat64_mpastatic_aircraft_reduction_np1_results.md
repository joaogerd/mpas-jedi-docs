# FGAT64 MPASstatic Aircraft reduction np1 results

## Objective

Reduce only the Aircraft observer in the isolated 3D-FGAT np1 test with MPASstatic covariance. The goal was to separate failures caused by Aircraft filters, Aircraft `obsdataout`, or the core Aircraft observation path.

No operational workflow files, staging templates, or main submission scripts were changed. No 64-rank tests and no Radiosonde/SfcCorrected tests were submitted in this step.

## Files created

YAMLs:

- `fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_aircraft_no_filters_np1.yaml`
- `fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_aircraft_no_obsdataout_np1.yaml`
- `fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_aircraft_minimal_np1.yaml`

PBS scripts:

- `run_fgat64_mpastatic_aircraft_no_filters_np1.pbs`
- `run_fgat64_mpastatic_aircraft_no_obsdataout_np1.pbs`
- `run_fgat64_mpastatic_aircraft_minimal_np1.pbs`

## Static validation

Validation completed before submission:

- all YAMLs parsed with `yaml.safe_load`;
- all PBS scripts passed `bash -n`;
- no PBS contains internal `qsub`;
- no PBS uses `#PBS -V`;
- all PBS scripts run with `NP=1`;
- all PBS scripts export `OMP_NUM_THREADS=1` and `GFORTRAN_CONVERT_UNIT=big_endian:101-200`;
- each YAML contains only `Aircraft`;
- each YAML uses `covariance model: MPASstatic`;
- none of the YAMLs contains `SABER`, `BUMP`, `NICAS`, or `VBAL`.

Variant checks:

| YAML | filters | obsdataout |
|---|---:|---:|
| `3dvar_fgat.fgat64.mpastatic_aircraft_no_filters_np1.yaml` | removed | retained |
| `3dvar_fgat.fgat64.mpastatic_aircraft_no_obsdataout_np1.yaml` | retained | removed |
| `3dvar_fgat.fgat64.mpastatic_aircraft_minimal_np1.yaml` | removed | removed |

## Common runtime configuration

- Working directory: `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test`
- Binary: `/p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x`
- MPI launcher: `/opt/cray/pals/1.6/bin/mpiexec`
- Ranks: `1`
- Covariance: `MPASstatic`
- Cost type: `3D-FGAT`
- Time window: `2018-04-14T21:00:00Z` to `2018-04-15T03:00:00Z`
- Background: `background/mpasout.2018-04-14_21.00.00.nc`
- Aircraft IODA input: `obs/aircraft_obs_2018041500.h5`
- Operator: `VertInterp`
- Simulated variables: `airTemperature`, `windEastward`, `windNorthward`, `specificHumidity`

## Results table

| case | job id | filters | obsdataout | result | failure point | analysis generated | feedback generated | conclusion |
|---|---|---:|---:|---|---|---|---|---|
| `aircraft_no_filters_np1` | `258353.pbs-ha` | no | yes | failed, status 139 | after `Model:forecast: forecast finished`; rank 0 `signal 11` | no new file | no | failure is not caused only by Aircraft filters |
| `aircraft_no_obsdataout_np1` | `258360.pbs-ha` | yes | no | failed, status 139 | after `Model:forecast: forecast finished`; rank 0 `signal 11` | no new file | no | failure is not caused only by feedback writing |
| `aircraft_minimal_np1` | `258363.pbs-ha` | no | no | failed, status 139 | after `Model:forecast: forecast finished`; rank 0 `signal 11` | no new file | no | core Aircraft path is enough to trigger the crash |

The only file in `analysis/` remains:

```text
2026-06-04 14:26:39 31716080 analysis/analysis.2018-04-15_00.00.00.nc
```

That file is from the earlier MPASstatic no-observers case. None of the three Aircraft reduction tests created a new analysis file. No feedback file was produced.

Runtime logs present after the last run:

- `log.atmosphere.0000.out`
- `log.atmosphere.0000.d0001.out`

Runtime logs not observed:

- `log.atmosphere.0000.err`
- `mpasjedi_variational.log`

## Key log evidence

All three variants show the same pattern. Representative lines from the minimal case:

```text
Aircraft: read database from obs/aircraft_obs_2018041500.h5 (io pool size: 1)
Aircraft processed vars: 5 Variables: airTemperature, specificHumidity, virtualTemperature, windEastward, windNorthward
Aircraft assimilated vars: 4 Variables: airTemperature, windEastward, windNorthward, specificHumidity
Model:forecast: forecast finished:
cn-0005.head.cm.cptec.inpe.br: rank 0 died from signal 11 and dumped core
[INFO] command exit status: 139
```

No clear `CRITICAL ERROR`, `MPI_Abort`, `GeoVaLs`, `HofX`, `Nonlinear Jo`, or successful IODA write message appears before the crash.

## Interpretation

The reduction matrix rules out the simplest explanations:

- The failure is not only from `PreQC` or `Background Check`, because removing all Aircraft filters still fails.
- The failure is not only from `obsdataout`/feedback writing, because removing `obsdataout` still fails.
- The failure remains in the minimal Aircraft path, so the active fault domain is now the core Aircraft observation path.

The current strongest candidates are:

- the generated Aircraft IODA file content/layout;
- metadata or dimensions required by `VertInterp`;
- a missing or incompatible vertical coordinate/pressure/height field for Aircraft interpolation;
- one of the simulated variables requested by Aircraft;
- GeoVaLs/HofX generation for Aircraft before the code emits a visible HofX/GeoVaLs log line;
- an incompatibility between this runtime's model/state variables and what the Aircraft operator expects.

The MPAS pressure warnings are still present, but they are not sufficient alone to explain this crash because the MPASstatic no-observers case passed.

## Recommended next diagnostics

Stay in np1 and keep MPASstatic. Do not return to 64 ranks yet.

Recommended next matrix, one case at a time:

1. Aircraft minimal with `airTemperature` only.
2. Aircraft minimal with winds only: `windEastward`, `windNorthward`.
3. Aircraft minimal with `specificHumidity` only.
4. Static HDF5 comparison between `obs/aircraft_obs_2018041500.h5` and an official MPAS-JEDI Aircraft IODA file from a passing reference test, checking groups, dimensions, variable names, missing values, time metadata, pressure/height coordinates, and required metadata for `VertInterp`.
5. If one variable-only case passes and another fails, focus on the corresponding GeoVaLs/operator dependency.
6. If every variable-only case fails, focus first on Aircraft IODA metadata/layout and `VertInterp` requirements rather than filters or feedback.
