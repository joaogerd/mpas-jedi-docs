# FGAT64 MPASstatic Aircraft airTemperature official-IODA np1 result

## Objective

Test the same isolated Aircraft minimal `airTemperature` np1 case with MPASstatic covariance, replacing only the Aircraft input IODA file with the official MPAS-JEDI Aircraft IODA used by passing tests.

This was intended to determine whether the workflow-generated `aircraft_obs_2018041500.h5` was the sole cause of the Aircraft crash.

No operational workflow files, templates, or main submission scripts were changed. No 64-rank tests and no other observer or variable tests were submitted.

## Official IODA selected

Located candidates:

- `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1/aircraft_obs_2018041500_m.nc4`
- `/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas-only-20260516T170436Z/jedi-bundle/ufo-data/testinput_tier_1/aircraft_obs_2018041500_m.nc4`

The source-tree file was used:

`/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1/aircraft_obs_2018041500_m.nc4`

This is the same Aircraft IODA path referenced by official MPAS-JEDI tests such as `mpas-jedi/test/testinput/3dvar.yaml` through `Data/ufo/testinput_tier_1/aircraft_obs_2018041500_m.nc4`.

## Files created

YAML:

`/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_aircraft_airTemperature_only_official_ioda_np1.yaml`

PBS:

`/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/run_fgat64_mpastatic_aircraft_airTemperature_only_official_ioda_np1.pbs`

## Validation

Checks before submission:

- YAML parsed with `yaml.safe_load`.
- PBS passed `bash -n`.
- PBS does not contain internal `qsub`.
- PBS does not use `#PBS -V`.
- YAML contains only `Aircraft`.
- YAML uses `covariance model: MPASstatic`.
- YAML contains no `SABER`, `BUMP`, `NICAS`, or `VBAL`.
- YAML contains no filters and no `obsdataout`.
- `simulated variables` contains only `airTemperature`.
- `obsdatain` points to the official file:

```text
/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1/aircraft_obs_2018041500_m.nc4
```

## Submitted job

- Job id: `258385.pbs-ha`
- PBS: `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/run_fgat64_mpastatic_aircraft_airTemperature_only_official_ioda_np1.pbs`
- YAML: `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_aircraft_airTemperature_only_official_ioda_np1.yaml`
- IODA: `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1/aircraft_obs_2018041500_m.nc4`
- Working directory: `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test`
- Binary: `/p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x`
- MPI launcher: `/opt/cray/pals/1.6/bin/mpiexec`
- Ranks: `1`
- PBS log: `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/logs/fgat64_mpastatic_aircraft_airTemperature_only_official_ioda_np1.pbs.out`

Command:

```bash
/opt/cray/pals/1.6/bin/mpiexec -n 1 \
  /p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x \
  /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_aircraft_airTemperature_only_official_ioda_np1.yaml
```

## Result

- Final status: `139`
- Failure: `rank 0 died from signal 11 and dumped core`
- Failure point: after `Model:forecast: forecast finished`
- New analysis file: no
- Feedback file: no
- `log.atmosphere.0000.err`: not present
- `mpasjedi_variational.log`: not present

Relevant log evidence:

```text
Aircraft: read database from /p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1/aircraft_obs_2018041500_m.nc4 (io pool size: 1)
Aircraft processed vars: 4 Variables: airTemperature, specificHumidity, windEastward, windNorthward
Aircraft assimilated vars: 1 Variables: airTemperature
Model:forecast: forecast finished:
cn-0001.head.cm.cptec.inpe.br: rank 0 died from signal 11 and dumped core
[INFO] command exit status: 139
```

Runtime logs after the run:

- `log.atmosphere.0000.out`
- `log.atmosphere.0000.d0001.out`

The only file in `analysis/` remains the older no-observers output:

```text
2026-06-04 14:26:39 31716080 analysis/analysis.2018-04-15_00.00.00.nc
```

No feedback file was produced.

## Interpretation

The official Aircraft IODA did not fix the crash. Therefore, the failure is not explained solely by the workflow-generated Aircraft HDF5 content or structure.

This does not clear the workflow IODA of all issues, because the static comparison still showed large structural differences. However, this test proves that even a known official Aircraft IODA fails in the isolated FGAT runtime/YAML when used as:

- `3D-FGAT`;
- `MPASstatic`;
- Aircraft only;
- `VertInterp`;
- `airTemperature` only;
- no filters;
- no `obsdataout`;
- x1.10242 runtime and current isolated namelists/streams.

The current fault domain shifts toward the runtime/YAML/operator configuration rather than only the IODA file.

Most likely remaining candidates:

- mismatch between the isolated x1.10242 runtime and the official test's x1.2562 geometry/runtime assumptions;
- `obs operator: VertInterp` configuration missing details that official tests rely on indirectly;
- `obsop_name_map.yaml` incompatibility with this runtime/YAML;
- model/state variables or GeoVaLs required by Aircraft `airTemperature` not being available/consistent in this isolated FGAT setup;
- MPAS runtime/namelist issue that only becomes fatal when an ObsSpace is active after forecast;
- difference between the operational binary `/p/projetos/.../builds/monan-jedi-mpas/bin/mpasjedi_variational.x` and the official CTest binary/environment.

## Recommended next diagnostics

Do not return to 64 ranks yet.

Recommended next steps, one at a time:

1. Run the official MPAS-JEDI `hofx3d.yaml` or `3dvar.yaml` Aircraft-only path in the official build/test directory with the same binary currently used here, to separate binary/runtime differences.
2. Create an isolated case that uses the official `3dvar.yaml` geometry/runtime as much as possible but only Aircraft `airTemperature`, then compare with the x1.10242 runtime.
3. Compare `obsop_name_map.yaml` in the isolated runtime with the one used by the passing official test.
4. Compare model variables and geovars/keptvars between the isolated FGAT YAML/runtime and the official passing `3dvar.yaml`/`hofx3d.yaml` setup.
5. If possible, rerun the isolated official-IODA case with the official CTest binary `/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/bin/mpasjedi_variational.x` before changing workflow templates.
