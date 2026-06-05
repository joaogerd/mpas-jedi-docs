# 3DFGAT x1.10242 np64 Runtime Path Resolution

## Context

This note documents the diagnosis that produced a passing 3DFGAT
x1.10242 `np64` run from the official MPAS-JEDI 3DFGAT baseline, using
workflow/tutorial runtime geometry and background files.

The passing YAML was:

```text
/p/projetos/monan_das/joao.gerd/manual-tests/official-3dvar-baseline-x1.2562/variants/3dfgat.workflow_geometry_background.yaml
```

The passing PBS was:

```text
/p/projetos/monan_das/joao.gerd/manual-tests/official-3dvar-baseline-x1.2562/run_3dfgat_workflow_geometry_background_np64.pbs
```

The run directory was:

```text
/p/projetos/monan_das/joao.gerd/manual-tests/official-3dvar-baseline-x1.2562
```

Source report:

```text
/p/projetos/monan_das/joao.gerd/manual-tests/official-3dvar-baseline-x1.2562/baseline_3dfgat_x1.10242_np64_success.md
```

## Incremental Diagnostic Method

The diagnosis started from a known-good official 3DVar/3DFGAT test
environment and changed one thing at a time. The workflow geometry and
background were tested in an isolated manual area without changing the
operational workflow.

The key rule was to keep the official YAML structure and only redirect
specific inputs through controlled variants and symlinks. Each failure
was diagnosed from MPAS/OOPS logs before making the next change.

## Test Summary

| Step | Result | Finding |
|---|---|---|
| Official 3DFGAT baseline | Passed | Official x1.2562 paths and relative files were internally consistent. |
| Workflow geometry/background, `np1` | Failed in `create geom` | Runtime only had `x1.10242.graph.info.part.64`, not an `np1` decomposition. |
| Workflow geometry/background, `np64` | Failed in `create geom` | The graph existed, so the failure was not only rank decomposition. |
| Static x1.10242 package inspection | Consistent | `invariant`, `templateFields`, background, and graph dimensions were compatible. |
| Log inspection before symlinks | Failed | MPAS could not open `x1.10242.invariant.nc`. |
| Add required relative symlinks | Advanced | Geometry bootstrap succeeded and reached the input stream. |
| Fix `templateFields.10242.nc` to 21Z | Passed | OOPS finished with `status = 0`. |

## Root Cause

The main issue was runtime path resolution, not a failing observer or a
3DFGAT algorithm problem.

The YAML used absolute paths for the workflow namelist and streams files,
but MPAS resolves filenames inside `streams.atmosphere.*`, such as
`filename_template` and `stream_list.*`, relative to the process working
directory, not relative to the directory containing the streams file.

Because the job ran in the isolated official baseline directory, MPAS
looked for x1.10242 runtime files in that directory. Those relative files
were initially missing.

A second issue appeared after the missing paths were fixed:
`templateFields.10242.nc` pointed to a 00Z state, while the run needed
`2018-04-14_21:00:00`.

## Error 1: Missing Invariant File

First real MPAS error:

```text
CRITICAL ERROR: Could not open input file 'x1.10242.invariant.nc' to read mesh fields
```

Cause: `streams.atmosphere.outer` referenced `x1.10242.invariant.nc`
as a relative filename, but that file did not exist in the job execution
directory.

Resolution: create the required x1.10242 relative symlinks in the run
directory.

## Error 2: Wrong Template Time

After adding the symlinks, MPAS advanced to the input stream and failed
with:

```text
ERROR: File templateFields.10242.nc does not contain the time 2018-04-14_21:00:00
```

Cause: `templateFields.10242.nc` resolved to:

```text
/p/projetos/monan_das/joao.gerd/data/mpasjedi_tutorial2024_testdata/background/2018041418/mpasout.2018-04-15_00.00.00.nc
```

but the geometry/forecast needed `2018-04-14_21:00:00`.

Resolution: point `templateFields.10242.nc` at the 21Z file:

```text
/p/projetos/monan_das/joao.gerd/data/mpasjedi_tutorial2024_testdata/background/2018041418/mpasout.2018-04-14_21.00.00.nc
```

## Passing Baseline

Final status from the passing log:

```text
Run: Finishing oops::Variational<MPAS, UFO and IODA observations> with status = 0
OOPS Ending   2026-06-05 00:08:20 (UTC+0000)
Job finished at 2026-06-05T00:08:20+00:00
```

Produced outputs:

| Output | Size |
|---|---:|
| `Data/states/mpas.3dfgat.2018-04-15_00.00.00.nc` | 1872944 |
| `Data/os/obsout_3dfgat_sondes.nc4` | 750708 |
| `Data/os/obsout_3dfgat_gnssroref.nc4` | 99227 |
| `Data/os/obsout_3dfgat_sfc.nc4` | 389488 |

## Required Relative Files

The passing run required these relative files in the execution directory:

| Relative file | Observed target |
|---|---|
| `x1.10242.invariant.nc` | `/p/projetos/monan_das/joao.gerd/data/mpasjedi_tutorial2024_testdata/MPAS_namelist_stream_physics_files/x1.10242.invariant.nc` |
| `templateFields.10242.nc` | `/p/projetos/monan_das/joao.gerd/data/mpasjedi_tutorial2024_testdata/background/2018041418/mpasout.2018-04-14_21.00.00.nc` |
| `x1.10242.graph.info.part.64` | `/p/projetos/monan_das/joao.gerd/data/mpasjedi_tutorial2024_testdata/MPAS_namelist_stream_physics_files/x1.10242.graph.info.part.64` |
| `stream_list.atmosphere.background` | `stream_list.atmosphere.background` for the active MPAS stream package |
| `stream_list.atmosphere.analysis` | `stream_list.atmosphere.analysis` for the active MPAS stream package |
| `stream_list.atmosphere.ensemble` | `stream_list.atmosphere.ensemble` for the active MPAS stream package |
| `stream_list.atmosphere.control` | `stream_list.atmosphere.control` for the active MPAS stream package |

The `stream_list.*` files should come from the same coherent runtime
package as the active x1.10242 streams/static inputs. Mixing official
x1.2562 stream lists with x1.10242 runtime files is a risk and should be
avoided in the workflow.

## Recommendations for monan-jedi-workflow

- Ensure `prepare_runtime` creates every relative file expected by
  `streams.atmosphere.*` in the job execution directory.
- Ensure `templateFields.*` points to the same initial time as the
  background used by geometry and forecast.
- Avoid mixing official x1.2562 `stream_list.*` files with x1.10242
  runtime files.
- Validate `readlink -f` for `x1.*.invariant.nc`, `templateFields.*`,
  `x1.*.graph.info.part.N`, and `stream_list.atmosphere.*` before job
  submission.
- Validate the `xtime` in `templateFields.*` against the run initial time
  before job submission.
- Add a pre-submit validation for all relative files referenced by
  `streams.atmosphere.*`.

## Lessons Learned

- Absolute YAML paths to namelist and streams files do not make paths
  inside `streams.atmosphere.*` absolute.
- MPAS stream filenames are resolved relative to the execution directory.
- Rank decomposition problems should be separated from stream path
  problems; the `np64` graph existed, but the run still failed until
  relative stream inputs were present.
- A static dimension check can rule out mesh-size mismatches, but it does
  not validate time consistency.
- `templateFields.*` is not just a generic template in this path; its
  `xtime` must match the initial time requested by the MPAS run.
