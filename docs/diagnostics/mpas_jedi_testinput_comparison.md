# MPAS-JEDI testinput comparison for MONAN 3DVar-FGAT

Date: 2026-06-02

Workspace: `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow`

Observed local branch from `git status`: `cleanup/3dvar-single-time-workflow`. The requested branch name was `docs/3dvar-fgat-provenance`; no checkout or branch mutation was performed.

## Files analyzed

Official MPAS-JEDI references:

- `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/3dfgat.yaml`
- `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/3dfgat_cda.yaml`
- `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/3dfgat_pseudo.yaml`
- `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/3dvar.yaml`
- `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/3dvar_bumpcov.yaml`
- `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/3dvar_bumpcov_nbam.yaml`
- `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/3dvar_bumpcov_ropp.yaml`
- `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/4dfgat.yaml`
- `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/hofx3d.yaml`
- `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/obsop_name_map.yaml`
- `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/parameters_bumpcov.yaml`
- `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/parameters_bumploc.yaml`
- `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/480km/namelist.atmosphere_2018041421`
- `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/480km/namelist.atmosphere_2018041500`
- `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/480km/streams.atmosphere`

Local MONAN files:

- `configs/jedi/applications/3dvar_fgat.yaml`
- `build/rendered/3dvar_fgat.yaml`
- `configs/experiments/3dvar_fgat/render_context.example.yaml`
- `configs/experiments/3dvar_fgat/render_context.single_time_3dvar.yaml`
- `configs/experiments/3dvar_fgat/variable_map.example.yaml`
- `configs/experiments/3dvar_fgat/observers.yaml`
- `configs/jedi/obs_plugs/variational/aircraft.yaml`
- `configs/jedi/obs_plugs/variational/sondes.yaml`
- `configs/jedi/obs_plugs/variational/sfc.yaml`
- `build/rendered/provenance/3dvar_fgat.trace`
- `build/rendered/provenance/variable_map.trace`

## Provenance of the rendered YAML

`build/rendered/3dvar_fgat.yaml` is not rendered from the FGAT example context. Its trace says it was generated with:

- template: `configs/jedi/applications/3dvar_fgat.yaml`
- context: `configs/experiments/3dvar_fgat/render_context.single_time_3dvar.yaml`
- variable profile: `tutorial_2024_mpas_8_2_minimal_3dvar`

That provenance explains two local differences:

- The rendered time window is a single-time smoke window: `begin: 2018-04-15T00:00:00Z`, `length: PT1H`.
- Variable names are resolved from canonical names to MPAS/internal names, for example `air_temperature -> temperature`, `eastward_wind -> uReconstructZonal`, and `air_pressure_at_surface -> surface_pressure`.

This is useful as a 3DVar smoke artifact, but it is not a faithful MPAS-JEDI 3D-FGAT configuration.

## Critical differences

### 1. 3D-FGAT structure

Official `3dfgat.yaml` uses:

- `cost function.cost type: 3D-FGAT`
- `time window.begin: 2018-04-14T21:00:00Z`
- `time window.length: PT6H`
- `geometry.nml_file: ./Data/480km/namelist.atmosphere_2018041421`
- `model:` inside `cost function`
- `background.filename: ./Data/480km/bg/mpasout.2018-04-14_21.00.00.nc`
- `background.date: 2018-04-14T21:00:00Z`
- one active outer loop; the second loop is commented with a warning that 3D-FGAT cannot run multiple outer loops without later loops becoming 3D-Var.

Local `configs/jedi/applications/3dvar_fgat.yaml` and current `build/rendered/3dvar_fgat.yaml` use:

- `cost function.cost type: 3D-Var`
- no `model:` block inside `cost function`
- rendered window `2018-04-15T00:00:00Z`/`PT1H`
- rendered background `mpasout.2018-04-15_00.00.00.nc`
- rendered background date `2018-04-15T00:00:00Z`
- one active outer loop, but as 3D-Var smoke configuration rather than 3D-FGAT.

Probable error: for a real 3D-FGAT run, the local template should render `cost type: 3D-FGAT` and include a `model:` block. If using MPAS model propagation, the background should be at the FGAT window start, not at the central analysis time. For the official 2018-04-15T00 cycle, that means `2018-04-14T21:00:00Z` and `mpasout.2018-04-14_21.00.00.nc`.

Acceptable difference only for smoke testing: `render_context.single_time_3dvar.yaml` deliberately uses a single 00Z state and `PT1H`. It should not be labeled or treated as validated 3D-FGAT.

### 2. Variables

Official 3D-FGAT analysis variables are canonical JEDI/VADER names:

- `air_temperature`
- `water_vapor_mixing_ratio_wrt_moist_air`
- `eastward_wind`
- `northward_wind`
- `air_pressure_at_surface`

Official 3D-FGAT `model variables`/`background.state variables` include the same canonical analysis variables plus MPAS direct/auxiliary fields:

- `air_potential_temperature`
- `dry_air_density`
- `u`
- `water_vapor_mixing_ratio_wrt_dry_air`
- `air_pressure`
- `landmask`
- `seaice_fraction`
- `snowc`
- `skin_temperature_at_surface`
- `ivgtyp`
- `isltyp`
- `cloud_liquid_water`
- `cloud_liquid_ice`
- `rain_water`
- `snow_water`
- `graupel`
- `pressure_p`
- `snowh`
- `vegetation_area_fraction`
- `eastward_wind_at_10m`
- `northward_wind_at_10m`
- `lai`
- `smois`
- `tslb`
- `w`

Official SABER/BUMP 3DVar files use the canonical five-variable analysis list and a reduced but still canonical state list including direct MPAS fields such as `u`, `landmask`, `ivgtyp`, `isltyp`, `snowc`, `smois`, `tslb`, and `pressure_p`.

Local rendered YAML uses MPAS/internal names in `analysis variables`, `background.state variables`, BUMP variables, and `Control2Analysis`:

- `spechum`
- `surface_pressure`
- `temperature`
- `uReconstructMeridional`
- `uReconstructZonal`
- plus `theta`, `rho`, `u`, `qv`, `pressure_p`

Probable error: final MPAS-JEDI YAML should keep canonical JEDI/VADER names for the analysis and state contract where official testinput does so. The current `variable_map.example.yaml` resolves canonical names into internal names before rendering. That is likely wrong for the final YAML, even if those internal names are useful for validating the contents of MPAS NetCDF files.

Recommended minimum for our 3D-FGAT/SABER analysis variables:

```yaml
analysis variables:
- air_temperature
- water_vapor_mixing_ratio_wrt_moist_air
- eastward_wind
- northward_wind
- air_pressure_at_surface
```

Recommended minimum for our state/model variables, matching the official 3DVar BUMP covariance pattern and avoiding optional hydrometeors unless observations/operators require them:

```yaml
state variables:
- air_temperature
- water_vapor_mixing_ratio_wrt_moist_air
- eastward_wind
- northward_wind
- air_pressure_at_surface
- air_potential_temperature
- dry_air_density
- u
- water_vapor_mixing_ratio_wrt_dry_air
- air_pressure
- landmask
- seaice_fraction
- snowc
- skin_temperature_at_surface
- ivgtyp
- isltyp
- snowh
- vegetation_area_fraction
- eastward_wind_at_10m
- northward_wind_at_10m
- lai
- smois
- tslb
- pressure_p
```

For MPAS-model 3D-FGAT, use the same list as `model.model variables` and `background.state variables`. If hydrometeors or vertical velocity are needed by active obs operators, extend toward the official `3dfgat.yaml` full `&modvars` list with `cloud_liquid_water`, `cloud_liquid_ice`, `rain_water`, `snow_water`, `graupel`, and `w`.

### 3. Background error / SABER / BUMP

Official `3dfgat.yaml` uses `MPASstatic`, but official `3dvar_bumpcov.yaml`, `3dvar_bumpcov_nbam.yaml`, and `3dvar_bumpcov_ropp.yaml` show the relevant SABER pattern:

- `covariance model: SABER`
- `saber central block.saber block name: BUMP_NICAS`
- `read.io.files prefix: Data/bump/mpas_bumpcov`
- `drivers.multivariate strategy: univariate`
- `drivers.read local nicas: true`
- two grids: 3D variables and 2D surface pressure
- `saber outer blocks` with `StdDev`
- `StdDev.read.model file.filename: Data/bump/mpas.stddev.$Y-$M-$D_$h.$m.$s.nc`
- `stream name: control`

Local SABER section has the same high-level structure and additionally has:

- `no outer loop update: true`
- `read.io.data directory` plus `files prefix`
- `active variables`
- `BUMP_VerticalBalance` outer block
- `linear variable change: Control2Analysis`

Probable error: the local SABER variable names are internal/BUMP tutorial names (`stream_function`, `velocity_potential`, `temperature`, `spechum`, `surface_pressure`) while official MPAS-JEDI BUMP covariance reads canonical variables (`air_temperature`, `water_vapor_mixing_ratio_wrt_moist_air`, `eastward_wind`, `northward_wind`, `air_pressure_at_surface`). This may be acceptable only if the external MONAN SABER files were generated with those internal variable names and the installed SABER variable-change chain expects them. That must be proven from the actual BUMP files, not inferred from the MPAS background file variable names.

Difference requiring confirmation, not immediate removal: the official `3dvar_bumpcov.yaml` does not use `BUMP_VerticalBalance` or `Control2Analysis`, but the local covariance data has `NICAS`, `StdDev`, and `VBAL` files. Keeping SABER/BUMP/NICAS/VBAL can be scientifically reasonable, but the structure should be matched to the installed JEDI/SABER examples that include VBAL. The provided official references do not prove that the local `BUMP_VerticalBalance` block is invalid; they only prove it is not part of the baseline `3dvar_bumpcov.yaml`.

Likely low-risk correction: keep SABER, but switch read-grid variables and analysis/state variables to canonical names unless BUMP file metadata shows the precomputed files require internal names.

### 4. Observers

Official names and patterns:

- Radiosonde obs space is named `Radiosonde`, not `sondes`.
- Aircraft obs space is named `Aircraft`, not `aircraft`.
- FGAT and BUMP tests use `SfcCorrected` for station pressure surface observations.
- Full `3dvar.yaml` additionally splits surface observations into `Surface T,Q,Ps` with `Composite` and `Surface U,V` with `VertInterp`.
- `obsop_name_map.yaml` maps IODA/UFO simulated variable names such as `airTemperature`, `windEastward`, `windNorthward`, `specificHumidity`, `virtualTemperature`, and `stationPressure` to canonical model variables.

Local observer skeletons:

- `aircraft`: `VertInterp`, simulated `[airTemperature, windEastward, windNorthward, specificHumidity]`, no filters.
- `sondes`: `VertInterp`, simulated `[airTemperature, virtualTemperature, windEastward, windNorthward, specificHumidity]`, no filters.
- `sfc`: `SfcCorrected`, simulated `[stationPressure]`, no filters.

Probable errors:

- Missing `PreQC` and `Background Check` filters for aircraft/sondes.
- Missing `PreQC`, `Difference Check`, and `Background Check` filters for surface pressure.
- Lowercase obs space names are not consistent with official testinput and may change diagnostics, output metadata, or test expectations.
- `sondes` includes `virtualTemperature`; official MPAS-JEDI `3dfgat.yaml`, `3dvar.yaml`, and `hofx3d.yaml` radiosonde simulated variables use `airTemperature`, not `virtualTemperature`. `obsop_name_map.yaml` supports `virtualTemperature`, but the provided official variational tests do not use it for the radiosonde block.

Acceptable difference with conditions: Keeping surface as only `SfcCorrected`/`stationPressure` is consistent with official `3dfgat.yaml`, `3dvar_bumpcov.yaml`, NBAM, ROPP, and 4DFGAT. Splitting into `Surface T,Q,Ps` and `Surface U,V` is supported by official `3dvar.yaml`, but it is a broader scientific/configuration change and should depend on the IODA surface variables we intend to assimilate.

Path issue: official YAML uses `testinput/obsop_name_map.yaml`, relative to the mpas-jedi test working directory. Local YAML uses `obsop_name_map.yaml`, which is only safe if the runtime directory contains that file and the executable starts in that runtime directory. The prepared runtime currently has `obsop_name_map.yaml`, so the local relative path is coherent if the PBS/runtime enters that directory before execution.

### 5. Minimizer and variational settings

Official variational tests use:

- `minimizer.algorithm: DRPCG`
- `ninner: '10'`
- `gradient norm reduction: 1e-10`
- `test: 'on'`
- one active loop for `3dfgat.yaml` and `3dfgat_pseudo.yaml`
- two loops for 3DVar BUMP tests

Local rendered YAML uses:

- `minimizer.algorithm: DRIPCG`
- `ninner: 60`
- `gradient norm reduction: 1e-3`
- no `test: 'on'`
- one active loop
- `final.diagnostics.departures: oman`

Probable error for reproducing official behavior: use `DRPCG`, `ninner: '10'`, `gradient norm reduction: 1e-10`, and `test: 'on'` for official-style comparison runs. `DRIPCG`/60/`1e-3` may be acceptable for a production-tuned MONAN experiment only after runtime validation.

Acceptable difference: `final diagnostics.departures: oman` is a local output/diagnostic choice. The official YAMLs do not require this exact block in the excerpts reviewed, but it is not obviously incompatible.

### 6. Runtime paths

Official MPAS-JEDI testinput assumes a test runtime layout with paths like:

- `./Data/480km/namelist.atmosphere_2018041421`
- `./Data/480km/streams.atmosphere`
- `testinput/obsop_name_map.yaml`
- `Data/bump/mpas_bumpcov`

Local rendered YAML mixes absolute MONAN paths for background/observations/covariance with relative files for MPAS runtime assets:

- `namelist.atmosphere.outer`
- `streams.atmosphere.outer`
- `obsop_name_map.yaml`

The prepared runtime directory `build/runtime/jaci_3dvar_fgat_tutorial_2018041500/2018041500` contains symlinked/runtime files:

- `namelist.atmosphere.inner`
- `namelist.atmosphere.outer`
- `streams.atmosphere.inner`
- `streams.atmosphere.outer`
- `obsop_name_map.yaml`
- `background/mpasout.2018-04-15_00.00.00.nc`
- `covariance/mpas.stddev.nc`
- `graph/graph.info.part.0064`
- `x1.10242.graph.info.part.64`
- physics/static tables

Probable runtime risk: the rendered YAML references absolute scratch/data paths for background and observations, so entering the runtime directory is not sufficient by itself. Conversely, the relative namelist/streams/obsop paths require execution from a directory that contains those files. The PBS/runtime wrapper must `cd` to the prepared runtime directory before running MPAS-JEDI. This report does not propose PBS changes per request.

FGAT-specific runtime gap: current staged runtime has only `mpasout.2018-04-15_00.00.00.nc`. Real MPAS-model 3D-FGAT, matching official `3dfgat.yaml`, needs the 21Z background and a namelist/streams pair consistent with that start time, for example `mpasout.2018-04-14_21.00.00.nc` and a 21Z namelist equivalent.

## Differences acceptable for now

- Keeping SABER/BUMP instead of official `MPASstatic` is acceptable because the stated MONAN goal is SABER/BUMP/NICAS/VBAL.
- Keeping `SfcCorrected` only for surface observations is acceptable for a conservative FGAT/BUMP pattern. Splitting surface observations should be a separate scientific decision.
- Preserving MONAN/JACI absolute paths is acceptable for staged datasets, provided relative runtime assets are resolved by changing into the runtime directory.
- One active outer loop is correct for 3D-FGAT.
- The single-time context is acceptable as an explicitly named 3DVar smoke context, but should not be used as the rendered artifact for a real 3D-FGAT comparison.

## Probable errors to fix

1. `cost type` is `3D-Var`; for the 3D-FGAT template it should render `3D-FGAT`.
2. `cost function.model` is missing; official 3D-FGAT uses it.
3. Real FGAT background date/file should be at the beginning of the FGAT window, e.g. `2018-04-14T21:00:00Z`, not the central analysis time.
4. Final YAML currently uses internal MPAS variable names where official YAML uses canonical JEDI/VADER names.
5. `variable_map.example.yaml` currently drives rendered YAML substitution to internal names. It should instead support validation/provenance unless a profile explicitly declares it is an internal-name smoke mode.
6. Aircraft/sondes/sfc observer filters are missing.
7. Official obs space capitalization/names are not followed.
8. Radiosonde simulated variables likely should not include `virtualTemperature` for the official-style variational comparison.
9. Minimizer should use `DRPCG` for official reproduction.
10. Runtime has only the 00Z background staged for the current smoke run; true 3D-FGAT needs 21Z background/runtime consistency.

## Recommendations

### Low-risk YAML corrections

- Change the 3DVar-FGAT application template to render `cost type: 3D-FGAT`.
- Add `cost function.model` for MPAS-model FGAT:

```yaml
model:
  name: MPAS
  tstep: PT45M
  model variables: &modvars
  - air_temperature
  - water_vapor_mixing_ratio_wrt_moist_air
  - eastward_wind
  - northward_wind
  - air_pressure_at_surface
  ...
```

- Use `background.state variables: *modvars` for MPAS-model 3D-FGAT.
- Keep a single active iteration block.
- Set the official-style minimizer defaults to `DRPCG`, `ninner: '10'`, `gradient norm reduction: 1e-10`, `test: 'on'`.
- Add official filters to `aircraft`, `sondes`, and `sfc`.
- Use official obs space names: `Aircraft`, `Radiosonde`, `SfcCorrected`.
- Use canonical variables in final YAML; preserve internal MPAS field lookup in `variable_map` only for validation/provenance.

### Medium-risk corrections requiring data confirmation

- Move the FGAT render context from 00Z/`PT1H` to 21Z/`PT6H` only when `mpasout.2018-04-14_21.00.00.nc` and matching namelist/streams are staged.
- Keep or adjust `BUMP_VerticalBalance` only after confirming the installed SABER configuration and BUMP file metadata expect the current control variable names and vertical balance relationships.
- Decide whether to split surface observations into `Surface T,Q,Ps` and `Surface U,V` after checking the actual `sfc_obs_2018041500.h5` variables and the intended assimilation set.

## Patch plan

No YAML patch has been applied in this analysis step. A separate patch should be staged in this order:

1. Add an official-style FGAT variable profile or change the FGAT profile so rendered `analysis_variables`, `state_variables`, BUMP grid variables, and `Control2Analysis` output variables remain canonical.
2. Update `configs/jedi/applications/3dvar_fgat.yaml` to use `cost type: 3D-FGAT`, add `model:`, and bind `background.state variables` to model variables for FGAT.
3. Add context fields for `jedi.model_tstep` and `jedi.model_variables`, preserving MONAN/JACI paths.
4. Update `configs/experiments/3dvar_fgat/render_context.example.yaml` to represent true FGAT timing: 21Z start, `PT6H`, and background file date `2018-04-14_21.00.00`, if that file is staged.
5. Leave `render_context.single_time_3dvar.yaml` as explicit smoke-only context or rename/output it so it cannot be confused with 3D-FGAT.
6. Update observer plugs with official filters and official obs space names.
7. Switch official reproduction minimizer to `DRPCG`, `ninner: '10'`, `gradient norm reduction: 1e-10`, `test: 'on'`.
8. Re-render to a fresh artifact and validate parseability.
9. Do not modify PBS or staging workflow in this patch.

## Patch note: limited YAML configuration correction

The follow-up limited patch should keep the single-time 3DVar smoke context separate from the official-style FGAT example context. The FGAT example context should render canonical JEDI/VADER analysis, state, and model variables, while retaining MPAS/internal aliases in the variable map for validation and provenance.

SABER/BUMP/NICAS/VBAL should remain enabled. The local VBAL/Control2Analysis variable names are still an item to confirm against the real pre-generated BUMP/SABER files; they should not be removed solely because the baseline official BUMP covariance YAML does not include VBAL.

The patch only corrects configuration and rendering. A real 3D-FGAT execution still requires a staged `mpasout.2018-04-14_21.00.00.nc` background plus namelist/streams files compatible with that 21Z background start.

## Commands to reproduce the comparison

List local and official files:

```bash
find configs/jedi configs/experiments build/rendered docs -type f | sort
find /p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput -type f | sort
```

Inspect key local artifacts:

```bash
sed -n '1,260p' configs/jedi/applications/3dvar_fgat.yaml
sed -n '1,340p' build/rendered/3dvar_fgat.yaml
sed -n '1,220p' build/rendered/provenance/3dvar_fgat.trace
sed -n '1,220p' build/rendered/provenance/variable_map.trace
```

Inspect key official artifacts:

```bash
sed -n '1,260p' /p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/3dfgat.yaml
sed -n '1,260p' /p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/3dvar_bumpcov.yaml
sed -n '1,260p' /p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/3dvar.yaml
sed -n '1,220p' /p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/obsop_name_map.yaml
```

Validate parseability of the rendered YAML and contexts:

```bash
python3 - <<'PY'
from pathlib import Path
import yaml

paths = [
    "build/rendered/3dvar_fgat.yaml",
    "configs/experiments/3dvar_fgat/render_context.example.yaml",
    "configs/experiments/3dvar_fgat/render_context.single_time_3dvar.yaml",
    "configs/experiments/3dvar_fgat/variable_map.example.yaml",
    "configs/experiments/3dvar_fgat/observers.yaml",
    "build/rendered/render_context.with_observers.yaml",
]

for path in paths:
    yaml.safe_load(Path(path).read_text(encoding="utf-8"))
    print(f"OK {path}")
PY
```

Re-render the current smoke artifact:

```bash
scripts/run/render_3dvar_fgat.sh \
  configs/experiments/3dvar_fgat/render_context.single_time_3dvar.yaml \
  build/rendered/3dvar_fgat.yaml
```

Re-render with the FGAT example context:

```bash
scripts/run/render_3dvar_fgat.sh \
  configs/experiments/3dvar_fgat/render_context.example.yaml \
  build/rendered/3dvar_fgat.example.yaml
```

Check runtime files required by relative paths:

```bash
find -L build/runtime/jaci_3dvar_fgat_tutorial_2018041500/2018041500 -maxdepth 2 -type f | sort
find -L data/covariance data/static data/graph -maxdepth 3 -type f | sort
```
