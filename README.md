# MPAS-JEDI documentation and diagnostics

This repository organizes documentation generated while investigating MPAS-JEDI and MONAN-JEDI experiments on the JACI HPC system.

The immediate goal is to document a reliable path for running and adapting MPAS-JEDI variational experiments, starting from a configuration that is already known to pass.

## Current focus

We are investigating the MPAS-JEDI 3DVar and 3D-FGAT workflow used as a basis for MONAN-JEDI experiments.

The current working strategy is:

1. Identify an official MPAS-JEDI test that passes.
2. Document exactly which YAML, executable, runtime directory and input files are used.
3. Document the role of every required file.
4. Map official files to available tutorial/MONAN files.
5. Reconfigure the passing YAML progressively, changing one component at a time.
6. Only after the manual/isolated case is understood and reproduced should the operational workflow be updated.

## Main sections

### Baseline

Start here:

- [Official 3DVar baseline documentation](docs/baseline/README.md)

Important documents:

- [Official run inventory](docs/baseline/passing_3dvar_official_run_inventory.md)
- [Required files inventory](docs/baseline/passing_3dvar_required_files.md)
- [Official-to-tutorial file mapping](docs/baseline/official_3dvar_to_tutorial_file_mapping.md)
- [Reconfiguration plan](docs/baseline/reconfigure_passing_3dvar_to_tutorial_plan.md)

### Diagnostics

Historical diagnostic reports from the FGAT investigation:

- [Diagnostic reports index](docs/diagnostics/README.md)

### Reference YAMLs

Selected official MPAS-JEDI YAMLs used for comparison:

- [Reference YAMLs](docs/reference-yamls/README.md)

## Key conclusions so far

The official MPAS-JEDI 3DVar test passes using the 480 km `x1.2562` mesh. It also works with MPI ranks for which a matching graph partition exists, for example `x1.2562.graph.info.part.16`.

The 64-rank official test fails if `x1.2562.graph.info.part.64` is missing. This is a decomposition-file issue, not a generic MPI or CTest failure.

Tutorial data found so far mostly use different meshes, such as `x1.10242` and `x1.40962`. Therefore, these files must be handled as a consistent package:

- background;
- invariant/static mesh;
- graph partition;
- namelist;
- streams.

They should not be replaced independently.

Earlier FGAT diagnostics showed that the isolated FGAT path works without observers when using MPASstatic, but fails when the Aircraft observer is activated, even with official IODA. This means the problem is not explained only by the workflow-generated Aircraft HDF5. The next reliable path is to start from the official 3DVar baseline that passes and adapt it progressively.

## Repository policy

This repository is for documentation and analysis only.

Do not store large NetCDF/HDF5 data files here. Use paths, inventories and reproducible instructions instead.
