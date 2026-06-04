# Diagnostico de decomposicao por ranks: MPAS-JEDI 3DVar

Data: 2026-06-03

Escopo: analise estatica dos arquivos oficiais MPAS-JEDI e do workflow operacional. Nesta etapa nao foi submetido nenhum job, nao foi executado `mpasjedi_variational.x`, nao foi alterado YAML operacional e nao foi alterado o workflow.

## Resumo dos casos A/B/C

| Caso | Comando essencial | Resultado | O que provou |
| --- | --- | --- | --- |
| A | `ctest --output-on-failure -R '^mpasjedi_3dvar$'` | Passou | O teste oficial CTest esta funcional no JACI. |
| B | `mpiexec -n 1 .../build/bin/mpasjedi_variational.x testinput/3dvar.yaml` | Passou | A chamada direta tambem funciona com 1 rank quando usa o layout oficial do CTest. |
| C | `mpiexec -n 64 .../build/bin/mpasjedi_variational.x testinput/3dvar.yaml` | Falhou | O YAML/layout oficial 480 km nao funciona com 64 ranks porque falta a particao `x1.2562.graph.info.part.64`. |

Conclusao: a falha observada em C nao e causada por CTest versus chamada direta. Ela aparece ao aumentar o numero de ranks para 64 usando a malha oficial 480 km `x1.2562`.

## Falha objetiva do caso C

O erro decisivo veio de:

```text
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/log.atmosphere.0000.err
```

Conteudo relevante:

```text
Beginning MPAS-atmosphere Error Log File for task       0 of      64
ERROR: Could not open block decomposition file for 64 blocks.
CRITICAL ERROR: Filename: x1.2562.graph.info.part.64
```

O MPAS usa o numero de tarefas MPI para montar o nome do arquivo de decomposicao. Com prefixo:

```text
x1.2562.graph.info.part.
```

e 64 ranks, ele procura:

```text
x1.2562.graph.info.part.64
```

Como esse arquivo nao existe no layout oficial do build, a geometria aborta durante `create geom`.

## Onde o prefixo e definido

Nos namelists oficiais 480 km:

```text
/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/480km/namelist.atmosphere_2018041500
/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/480km/namelist.atmosphere_2018041421
```

o bloco `decomposition` define:

```fortran
&decomposition
    config_block_decomp_file_prefix = 'x1.2562.graph.info.part.'
/
```

O stream oficial correspondente usa a malha `x1.2562`:

```text
filename_template="./Data/480km/bg/x1.2562.invariant.nc"
```

Portanto, para a malha oficial 480 km, o prefixo `x1.2562.graph.info.part.` esta coerente. O problema do caso C e apenas a ausencia da particao para 64 ranks.

## Arquivos de decomposicao x1.2562

No source oficial existem:

| Arquivo source | Existe no source? | Linkado no build atual? |
| --- | --- | --- |
| `x1.2562.graph.info.part.2` | sim | sim |
| `x1.2562.graph.info.part.4` | sim | sim |
| `x1.2562.graph.info.part.6` | sim | nao |
| `x1.2562.graph.info.part.8` | sim | sim |
| `x1.2562.graph.info.part.12` | sim | nao |
| `x1.2562.graph.info.part.16` | sim | sim |
| `x1.2562.graph.info.part.64` | nao encontrado | nao |

O CMake do MPAS-JEDI registra as particoes 480 km assim:

```cmake
# multi_pe_standard_480: list of processor counts for which to run non-enkf multi-PE 480km tests
# options: 2, 4, 6, 8, 12, 16
list(APPEND multi_pe_standard_480 2)
list(APPEND multi_pe_other_480 4 8 16)
list(APPEND mpas_test_partition_480km x1.2562.graph.info)
foreach(THIS_NPE ${multi_pe_standard_480} ${multi_pe_other_480})
    if( THIS_NPE GREATER 1 )
        list( APPEND mpas_test_partition_480km x1.2562.graph.info.part.${THIS_NPE})
    endif()
endforeach()
```

Arquivo:

```text
/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/CMakeLists.txt
```

Nao foi encontrado, no MPAS-JEDI, um script oficial usado pelos testes para gerar `x1.2562.graph.info.part.64`. A logica do teste e linkar particoes precomputadas; o build atual linka 2, 4, 8 e 16 para `x1.2562`.

## PBS C16 criado

Foi criado um novo PBS minimo:

```text
/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/isolated_variational/analysis/run_matrix_C16_direct_3dvar_ctest_yaml_np16.pbs
```

Log configurado:

```text
/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/isolated_variational/logs/direct_3dvar_ctest_yaml_np16.pbs.out
```

Ele nao usa `#PBS -V`, carrega o stack MONAN-JEDI dentro do script, e exporta explicitamente:

```bash
OMP_NUM_THREADS=1
GFORTRAN_CONVERT_UNIT=big_endian:101-200
```

Observacao sobre o diretorio: para a chamada direta funcionar com `testinput/3dvar.yaml`, `Data/` e `x1.2562.graph.info.part.16`, o diretorio efetivo precisa ser:

```text
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test
```

Esse e o layout efetivo usado pelos testes do MPAS-JEDI. Rodar a mesma chamada a partir do topo do build com o argumento relativo `testinput/3dvar.yaml` nao preserva os caminhos relativos do YAML oficial.

Comando principal do C16:

```bash
cd /p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test
/opt/cray/pals/1.6/bin/mpiexec -n 16 \
  /p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/bin/mpasjedi_variational.x \
  testinput/3dvar.yaml
```

## Como submeter manualmente o C16

Nao foi submetido automaticamente. Para submeter manualmente:

```bash
cd /p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/isolated_variational/analysis
qsub run_matrix_C16_direct_3dvar_ctest_yaml_np16.pbs
```

Monitorar:

```bash
qstat -u joao.gerd @pbs-ha
tail -n 160 /p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/isolated_variational/logs/direct_3dvar_ctest_yaml_np16.pbs.out
```

Recomendacao: executar C16 antes de qualquer D/E. Se C16 passar, fica confirmado que o caso oficial escala ao menos ate uma particao disponivel. Se C16 falhar, investigar paralelismo 480 km antes de olhar workflow.

## Diagnostico preliminar do workflow operacional

Workflow analisado:

```text
/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow
```

Runtime analisado:

```text
/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/build/runtime/jaci_3dvar_fgat_tutorial_2018041500/2018041500
```

O runtime contem a malha `x1.10242`:

```text
x1.10242.invariant.nc -> /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/data/static/x1.10242.invariant.nc
x1.10242.graph.info.part.64 -> /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/data/static/x1.10242.graph.info.part.64
```

Tambem existe:

```text
graph/graph.info.part.0064 -> /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/data/graph/graph.info.part.0064
```

Os streams do workflow apontam para a malha `x1.10242`:

```xml
filename_template="x1.10242.invariant.nc"
filename_template="templateFields.10242.nc"
```

Mas os dois namelists renderizados no runtime apontam para o prefixo errado:

```text
namelist.atmosphere.outer: config_block_decomp_file_prefix = 'x1.2562.graph.info.part.'
namelist.atmosphere.inner: config_block_decomp_file_prefix = 'x1.2562.graph.info.part.'
```

Isso explica o erro ja presente no runtime operacional:

```text
CRITICAL ERROR: Filename: x1.2562.graph.info.part.64
```

Diagnostico preliminar: o workflow operacional parece estar usando arquivos de malha `x1.10242`, e o arquivo `x1.10242.graph.info.part.64` esta presente no local onde o MPAS poderia encontra-lo. Porem, os namelists `inner` e `outer` ainda instruem o MPAS a procurar `x1.2562.graph.info.part.64`. Antes de rodar D/E, e necessario corrigir ou regenerar o runtime para que o prefixo de decomposicao combine com a malha `x1.10242`.

## Proximos passos recomendados

1. Submeter manualmente o C16 para confirmar que o caso oficial 480 km funciona com uma particao disponivel.
2. Nao submeter D/E ainda, porque o workflow renderizado tem mismatch entre `streams`/malha `x1.10242` e `config_block_decomp_file_prefix` `x1.2562`.
3. Investigar no workflow por que `namelist.atmosphere.inner` e `namelist.atmosphere.outer` sao links para namelists 480 km do MPAS-JEDI oficial, enquanto o runtime usa arquivos `x1.10242`.
4. Regenerar ou ajustar o staging/render do workflow para que o namelist use um prefixo coerente, provavelmente `x1.10242.graph.info.part.`, se esse for o padrao esperado pela malha operacional.
5. Depois de corrigir/regenerar o runtime, repetir primeiro D com 1 rank e depois E com 64 ranks.
