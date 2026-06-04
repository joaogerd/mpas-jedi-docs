# Baseline oficial 3DVar

Esta seção documenta a rodada oficial do MPAS-JEDI 3DVar que já foi comprovadamente executada com sucesso na JACI.

O objetivo é partir de uma configuração confiável antes de adaptar arquivos do tutorial ou do MONAN-JEDI.

## Documentos

- [Inventário da rodada oficial](passing_3dvar_official_run_inventory.md)
- [Inventário dos arquivos necessários](passing_3dvar_required_files.md)
- [Mapeamento entre arquivos oficiais e arquivos do tutorial](official_3dvar_to_tutorial_file_mapping.md)
- [Plano de reconfiguração progressiva](reconfigure_passing_3dvar_to_tutorial_plan.md)

## Conclusão principal

A baseline que passou usa a malha oficial de 480 km, identificada como `x1.2562`.

Os dados do tutorial encontrados até agora usam majoritariamente outras malhas, como `x1.10242`, `x1.40962` e `x1.62691`. Portanto, arquivos como `background`, `invariant`, `graph`, `namelist` e `streams` devem ser tratados como um pacote coerente, e não substituídos individualmente.
