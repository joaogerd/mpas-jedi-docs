# Relatórios de diagnóstico

Esta seção reúne os relatórios de diagnóstico produzidos durante a investigação da execução 3DVar/3D-FGAT do MPAS-JEDI/MONAN-JEDI na JACI.

Estes documentos devem ser lidos como trilha de evidências. A estratégia atual é retornar para uma baseline oficial que passa e adaptá-la progressivamente, mudando apenas um componente por vez.

## Diagnóstico principal

- [Comparação com os arquivos oficiais de testinput do MPAS-JEDI](mpas_jedi_testinput_comparison.md)
- [Análise da execução CTest 3DVar](ctest_3dvar_execution_analysis.md)
- [Matriz rank/YAML para 3DVar](mpasjedi_3dvar_rank_yaml_matrix.md)
- [Diagnóstico de decomposição por número de ranks](decomposition_rank_diagnosis.md)
- [Diagnóstico pós-forecast do FGAT64](fgat64_post_forecast_diagnosis.md)
- [Resultados da matriz de observers do FGAT64](fgat64_observer_matrix_results.md)
- [Comparação entre casos que passaram e casos que falharam](fgat64_pass_vs_fail_comparison.md)

## Investigação dos observers e do Aircraft

- [Comparação estática dos arquivos IODA de Aircraft](aircraft_ioda_static_comparison.md)
- [Resultado MPASstatic sem observers](fgat64_mpastatic_no_observers_np1_result.md)
- [Isolamento de observers com MPASstatic](fgat64_mpastatic_observer_isolation_np1_results.md)
- [Redução do caso Aircraft](fgat64_mpastatic_aircraft_reduction_np1_results.md)
- [Isolamento por variável no Aircraft](fgat64_mpastatic_aircraft_variable_isolation_np1_results.md)
- [Teste Aircraft com IODA oficial](fgat64_mpastatic_aircraft_official_ioda_np1_result.md)

## Interpretação atual

O caminho FGAT isolado funciona sem observers quando usa `MPASstatic`, mas falha quando o observer `Aircraft` é ativado, mesmo usando IODA oficial.

Isso indica que a falha não é explicada apenas pelo HDF5 de Aircraft gerado pelo workflow. O caminho mais confiável agora é partir da baseline oficial 3DVar que passa e adaptá-la progressivamente.
