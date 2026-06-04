# Documentação e diagnósticos do MPAS-JEDI

Este repositório organiza a documentação produzida durante a investigação de experimentos MPAS-JEDI e MONAN-JEDI na máquina JACI.

O objetivo imediato é documentar um caminho confiável para executar e adaptar experimentos variacionais do MPAS-JEDI, partindo de uma configuração que já foi comprovadamente executada com sucesso.

## Foco atual

Estamos investigando a configuração e a execução de casos 3DVar e 3D-FGAT do MPAS-JEDI, com o objetivo de apoiar a construção futura de workflows MONAN-JEDI.

A estratégia atual é:

1. Identificar uma rodada oficial do MPAS-JEDI que passa.
2. Documentar exatamente qual YAML, executável, diretório de execução e arquivos de entrada foram usados.
3. Documentar a função de cada arquivo necessário.
4. Mapear os arquivos oficiais para os arquivos disponíveis no tutorial/MONAN.
5. Reconfigurar progressivamente o YAML que passou, alterando apenas um componente por vez.
6. Somente depois que o caso manual/isolado estiver compreendido e reproduzido, atualizar o workflow operacional.

## Seções principais

### Baseline oficial

Comece por aqui:

- [Baseline oficial 3DVar](docs/baseline/README.md)

Documentos principais:

- [Inventário da rodada oficial](docs/baseline/passing_3dvar_official_run_inventory.md)
- [Inventário dos arquivos necessários](docs/baseline/passing_3dvar_required_files.md)
- [Mapeamento entre arquivos oficiais e arquivos do tutorial](docs/baseline/official_3dvar_to_tutorial_file_mapping.md)
- [Plano de reconfiguração progressiva](docs/baseline/reconfigure_passing_3dvar_to_tutorial_plan.md)

### Diagnósticos

Relatórios históricos da investigação do 3DVar/3D-FGAT:

- [Índice dos relatórios de diagnóstico](docs/diagnostics/README.md)

### YAMLs de referência

Cópias selecionadas dos YAMLs oficiais do MPAS-JEDI usados para comparação:

- [YAMLs oficiais de referência](docs/reference-yamls/README.md)

## Conclusões principais até aqui

A rodada oficial 3DVar do MPAS-JEDI passa usando a malha de 480 km `x1.2562`.

Ela também funciona com números de ranks para os quais existe um arquivo de particionamento compatível, por exemplo `x1.2562.graph.info.part.16`.

A execução oficial com 64 ranks falha quando o arquivo `x1.2562.graph.info.part.64` não existe. Isso caracteriza um problema de decomposição da malha, e não uma falha genérica de MPI, CTest ou do executável.

Os dados do tutorial encontrados até agora usam majoritariamente outras malhas, como `x1.10242`, `x1.40962` e `x1.62691`. Portanto, os seguintes arquivos precisam ser tratados como um pacote coerente:

- background;
- invariant/static mesh;
- graph partition;
- namelist;
- streams.

Eles não devem ser substituídos individualmente sem verificar compatibilidade de malha, data, resolução e decomposição.

Os diagnósticos anteriores indicaram que o caminho FGAT isolado funciona sem observers quando usa `MPASstatic`, mas falha quando o observer `Aircraft` é ativado, mesmo com IODA oficial. Assim, a falha não é explicada apenas pelo arquivo HDF5 de Aircraft gerado pelo workflow.

A abordagem atual é retornar para a baseline oficial 3DVar que passa e adaptá-la passo a passo.

## Política do repositório

Este repositório é destinado apenas à documentação, análise e rastreabilidade técnica.

Não armazene aqui arquivos grandes, como NetCDF ou HDF5. Use caminhos, inventários, relatórios e instruções reprodutíveis.
