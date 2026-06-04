# Diagnostic reports

This section contains diagnostic reports generated while investigating MONAN-JEDI / MPAS-JEDI 3DVar-FGAT execution on JACI.

These reports are historical and should be read as evidence trails. The current strategy is to return to the official passing 3DVar baseline and adapt it progressively.

## Main diagnostic thread

- [MPAS-JEDI testinput comparison](mpas_jedi_testinput_comparison.md)
- [CTest 3DVar execution analysis](ctest_3dvar_execution_analysis.md)
- [3DVar rank/YAML matrix](mpasjedi_3dvar_rank_yaml_matrix.md)
- [Decomposition rank diagnosis](decomposition_rank_diagnosis.md)
- [FGAT64 post-forecast diagnosis](fgat64_post_forecast_diagnosis.md)
- [FGAT64 observer matrix results](fgat64_observer_matrix_results.md)
- [FGAT64 pass vs fail comparison](fgat64_pass_vs_fail_comparison.md)

## Aircraft / observer investigation

- [Aircraft IODA static comparison](aircraft_ioda_static_comparison.md)
- [MPASstatic no-observers result](fgat64_mpastatic_no_observers_np1_result.md)
- [MPASstatic observer isolation results](fgat64_mpastatic_observer_isolation_np1_results.md)
- [Aircraft reduction results](fgat64_mpastatic_aircraft_reduction_np1_results.md)
- [Aircraft variable isolation results](fgat64_mpastatic_aircraft_variable_isolation_np1_results.md)
- [Aircraft official IODA result](fgat64_mpastatic_aircraft_official_ioda_np1_result.md)

## Current interpretation

The isolated FGAT path works without observers when using MPASstatic, but fails when Aircraft is activated, even with official IODA. This suggests the active issue is not only the workflow-generated Aircraft HDF5. The next reliable path is to start from the official 3DVar configuration that passes and adapt it one component at a time.
