# Official 3DVar baseline

This section documents the MPAS-JEDI official 3DVar test that is known to pass on JACI.

The goal is to start from a proven baseline before adapting any MONAN/tutorial data.

## Documents

- [Official run inventory](passing_3dvar_official_run_inventory.md)
- [Required files inventory](passing_3dvar_required_files.md)
- [Official-to-tutorial file mapping](official_3dvar_to_tutorial_file_mapping.md)
- [Reconfiguration plan](reconfigure_passing_3dvar_to_tutorial_plan.md)

## Key conclusion

The passing baseline uses the official 480 km `x1.2562` mesh. Tutorial data found so far mostly use other meshes, such as `x1.10242` and `x1.40962`. Therefore, background, invariant, graph, namelist and streams must be treated as a consistent package, not replaced one by one.
