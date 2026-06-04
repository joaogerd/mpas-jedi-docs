# FGAT64 MPASstatic Aircraft variable isolation np1 results

## Objective

Continue the Aircraft-core diagnosis in the isolated 3D-FGAT np1 runtime with MPASstatic covariance. This step had two parts:

1. Static comparison of the workflow Aircraft IODA file against an official MPAS-JEDI Aircraft IODA file used by passing tests.
2. Preparation of Aircraft-minimal YAMLs that isolate simulated variables, then submission of only the first case: `airTemperature` only.

No operational workflow files, staging templates, or main submission scripts were changed. No 64-rank tests were run. Radiosonde and SfcCorrected were not run.

## Static IODA comparison

Report created:

`/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/aircraft_ioda_static_comparison.md`

Files compared:

- Workflow Aircraft link: `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/obs/aircraft_obs_2018041500.h5`
- Workflow Aircraft real path: `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/build/runtime/jaci_3dvar_fgat_tutorial_2018041500/2018041500/obs/aircraft_obs_2018041500.h5`
- Official MPAS-JEDI Aircraft source: `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1/aircraft_obs_2018041500_m.nc4`
- Official MPAS-JEDI Aircraft build copy: `/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/Data/ufo/testinput_tier_1/aircraft_obs_2018041500_m.nc4`

Key static differences:

| item | workflow IODA | official IODA |
|---|---:|---:|
| file size | 57,243,706 bytes | 343,800 bytes |
| locations | 348,901 | 2,103 |
| dateTime window | 2018-04-14T21:00:00Z to 2018-04-15T03:00:00Z | same |
| ObsValue variables | airTemperature, specificHumidity, virtualTemperature, windEastward, windNorthward | airTemperature, specificHumidity, windEastward, windNorthward |
| extra workflow ObsValue | virtualTemperature | none |
| GSI auxiliary groups | absent | present: GsiHofX, GsiHofXBc, GsiFinalObsError, GsiAdjustObsError, etc. |
| MetaData difference | workflow has variable_names | official has sequenceNumber |
| pressure attrs | workflow `_FillValue=-999`, no units captured | official `_FillValue=9.969e36`, units Pa |
| height attrs | workflow `_FillValue=-999`, no units captured | official `_FillValue=9.969e36`, units m |

The time window matches, but the workflow file is much larger, includes `virtualTemperature`, lacks GSI auxiliary groups, and differs in metadata/attributes. These differences are material for `VertInterp`/HofX diagnostics.

## YAMLs and PBS scripts created

YAMLs:

- `fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_aircraft_airTemperature_only_np1.yaml`
- `fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_aircraft_winds_only_np1.yaml`
- `fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_aircraft_specificHumidity_only_np1.yaml`

PBS scripts:

- `run_fgat64_mpastatic_aircraft_airTemperature_only_np1.pbs`
- `run_fgat64_mpastatic_aircraft_winds_only_np1.pbs`
- `run_fgat64_mpastatic_aircraft_specificHumidity_only_np1.pbs`

Validation before submission:

- all YAMLs parsed with `yaml.safe_load`;
- all PBS scripts passed `bash -n`;
- no PBS contains internal `qsub`;
- no PBS uses `#PBS -V`;
- each YAML contains only `Aircraft`;
- each YAML uses `covariance model: MPASstatic`;
- each YAML has no filters;
- each YAML has no `obsdataout`;
- none of the YAMLs contains `SABER`, `BUMP`, `NICAS`, or `VBAL`;
- simulated variables were confirmed exactly:
  - `airTemperature_only`: `airTemperature`
  - `winds_only`: `windEastward`, `windNorthward`
  - `specificHumidity_only`: `specificHumidity`

## Submitted test

Only the first variable-isolation case was submitted, as requested.

- Job id: `258374.pbs-ha`
- PBS: `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/run_fgat64_mpastatic_aircraft_airTemperature_only_np1.pbs`
- YAML: `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_aircraft_airTemperature_only_np1.yaml`
- Simulated variables: `airTemperature`
- Working directory: `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test`
- Binary: `/p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x`
- MPI launcher: `/opt/cray/pals/1.6/bin/mpiexec`
- Ranks: `1`
- PBS log: `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/logs/fgat64_mpastatic_aircraft_airTemperature_only_np1.pbs.out`

Command:

```bash
/opt/cray/pals/1.6/bin/mpiexec -n 1 \
  /p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x \
  /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_aircraft_airTemperature_only_np1.yaml
```

## Result

| case | job id | simulated variables | result | failure point | analysis generated | feedback generated | conclusion |
|---|---|---|---|---|---|---|---|
| `airTemperature_only_np1` | `258374.pbs-ha` | `airTemperature` | failed, status 139 | after `Model:forecast: forecast finished`; rank 0 `signal 11` | no new file | no | even one Aircraft variable is enough to trigger the crash |
| `winds_only_np1` | not submitted | `windEastward`, `windNorthward` | not run | stopped because first variable case failed | n/a | n/a | pending manual decision |
| `specificHumidity_only_np1` | not submitted | `specificHumidity` | not run | stopped because first variable case failed | n/a | n/a | pending manual decision |

Relevant log evidence:

```text
Aircraft: read database from obs/aircraft_obs_2018041500.h5 (io pool size: 1)
Aircraft processed vars: 5 Variables: airTemperature, specificHumidity, virtualTemperature, windEastward, windNorthward
Aircraft assimilated vars: 1 Variables: airTemperature
Model:forecast: forecast finished:
cn-0005.head.cm.cptec.inpe.br: rank 0 died from signal 11 and dumped core
[INFO] command exit status: 139
```

No clear `CRITICAL ERROR`, `MPI_Abort`, `GeoVaLs`, `HofX`, `Nonlinear Jo`, or successful IODA write message appears before the crash.

The only file in `analysis/` remains the older no-observers output:

```text
2026-06-04 14:26:39 31716080 analysis/analysis.2018-04-15_00.00.00.nc
```

No feedback file was produced.

Runtime logs present after the run:

- `log.atmosphere.0000.out`
- `log.atmosphere.0000.d0001.out`

Runtime logs not observed:

- `log.atmosphere.0000.err`
- `mpasjedi_variational.log`

## Interpretation

Because `airTemperature` alone reproduces the crash, the problem is not caused by a combination of Aircraft simulated variables. It is also not caused only by filters or `obsdataout`, which were removed in earlier tests.

The active fault domain is now narrower:

- Aircraft IODA structure/content;
- `MetaData` required by `VertInterp`, especially pressure/height/location metadata and attributes;
- the Aircraft `VertInterp` operator itself with this IODA/runtime combination;
- common Aircraft/HofX/GeoVaLs setup before any visible HofX/GeoVaLs log line.

The static comparison reinforces the IODA-structure hypothesis: the workflow Aircraft file is much larger than the official passing file, includes `virtualTemperature`, lacks GSI auxiliary groups, and differs in metadata/attributes for pressure/height.

## Recommended next step

Do not return to 64 ranks yet.

Most informative next diagnostics:

1. Run the same Aircraft minimal `airTemperature` np1 case using the official passing IODA file `aircraft_obs_2018041500_m.nc4`, copied or symlinked into the isolated runtime, without changing the operational workflow.
2. If official IODA passes, the generated workflow Aircraft HDF5 is the problem.
3. If official IODA also fails in this runtime, compare the isolated YAML/operator/runtime against official `3dvar.yaml`/`hofx3d.yaml`, especially `obs operator`, model variables, geometry, and `obsop_name_map.yaml`.
4. Inspect the workflow Aircraft HDF5 for invalid pressure/height values, missing units, fill values equal to valid ranges, and per-variable location consistency.
