# Matriz diagnostica MPAS-JEDI 3DVar/3DVar-FGAT

Data: 2026-06-03

Objetivo: separar, com testes PBS pequenos e rastreaveis, se o travamento em `==> create geom` esta associado ao harness CTest, ao numero de ranks MPI, ao YAML oficial versus YAML renderizado do workflow, ou ao layout/diretorio de execucao.

Nenhum job foi submetido durante a criacao desta matriz.

## Artefatos criados

Os scripts PBS temporarios ficam em:

```text
/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/isolated_variational/analysis
```

Logs esperados:

```text
/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/isolated_variational/logs/ctest_3dvar_harness.pbs.out
/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/isolated_variational/logs/direct_3dvar_ctest_yaml_np1.pbs.out
/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/isolated_variational/logs/direct_3dvar_ctest_yaml_np64.pbs.out
/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/isolated_variational/logs/workflow_3dfgat_np1.pbs.out
/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/isolated_variational/logs/workflow_3dfgat_np64.pbs.out
```

Cada PBS imprime diagnosticos no inicio:

- data UTC, host, `PBS_JOBID`, `PBS_O_WORKDIR` e `pwd`;
- `PATH`, `LD_LIBRARY_PATH`, `MPASJEDI_VARIATIONAL_EXE`;
- `OMP_NUM_THREADS`, `GFORTRAN_CONVERT_UNIT`, `F_UFMTENDIAN`, `FI_CXI_RX_MATCH_MODE`;
- `ulimit -a`;
- `which mpiexec`, `which mpirun`;
- `module list`.

Os scripts carregam, se existir, o setup comum ja usado nos scripts isolados:

```text
/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/isolated_variational/scripts/common_setup.sh
```

Depois disso, definem explicitamente:

```bash
GFORTRAN_CONVERT_UNIT=big_endian:101-200
OMP_NUM_THREADS=1
```

## Matriz de testes

| Caso | Script PBS | Harness | Ranks | Workdir | YAML | Executavel | O que isola |
| --- | --- | --- | ---: | --- | --- | --- | --- |
| A | `run_matrix_A_ctest_3dvar_harness.pbs` | CTest | conforme CTest, registrado como 1 | `/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build` | `testinput/3dvar.yaml`, resolvido pelo CTest | registrado no CTest | Controle positivo: caminho oficial que ja passou |
| B | `run_matrix_B_direct_3dvar_ctest_yaml_np1.pbs` | chamada direta | 1 | `/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test` | `testinput/3dvar.yaml` | build MPAS-JEDI | Efeito CTest versus chamada direta com mesmo rank/YAML/layout oficial |
| C | `run_matrix_C_direct_3dvar_ctest_yaml_np64.pbs` | chamada direta | 64 | `/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test` | `testinput/3dvar.yaml` | build MPAS-JEDI | Efeito do numero de ranks usando YAML/layout oficial |
| D | `run_matrix_D_workflow_3dfgat_np1.pbs` | chamada direta | 1 | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/build/runtime/jaci_3dvar_fgat_tutorial_2018041500/2018041500` | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/build/rendered/3dvar_fgat.yaml` | instalacao MONAN-JEDI MPAS | Efeito do YAML/layout do workflow com rank unico |
| E | `run_matrix_E_workflow_3dfgat_np64.pbs` | chamada direta | 64 | mesmo do caso D | mesmo do caso D | instalacao MONAN-JEDI MPAS | Efeito combinado de workflow renderizado e 64 ranks |

## Comandos executados por cada caso

### A. CTest oficial

```bash
cd /p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build
ctest --output-on-failure -R '^mpasjedi_3dvar$'
```

Este caso valida o caminho que ja foi confirmado como funcional. Se ele falhar, o problema nao esta no workflow isolado, mas no ambiente PBS/modulos/dados naquele momento.

### B. Direto, YAML oficial, 1 rank

```bash
cd /p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test
/opt/cray/pals/1.6/bin/mpiexec -n 1 \
  /p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/bin/mpasjedi_variational.x \
  testinput/3dvar.yaml
```

Este caso compara diretamente com A. Se A passa e B trava, a diferenca principal e o harness CTest versus chamada direta, porque rank, YAML, workdir e binario visivel sao equivalentes.

### C. Direto, YAML oficial, 64 ranks

```bash
cd /p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test
/opt/cray/pals/1.6/bin/mpiexec -n 64 \
  /p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/bin/mpasjedi_variational.x \
  testinput/3dvar.yaml
```

Este caso compara com B. Se B passa e C falha, o fator dominante e a paralelizacao/ranks. Se B e C falham mas A passa, o fator dominante continua sendo CTest versus chamada direta.

### D. Workflow renderizado, 1 rank

```bash
cd /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/build/runtime/jaci_3dvar_fgat_tutorial_2018041500/2018041500
/opt/cray/pals/1.6/bin/mpiexec -n 1 \
  /p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x \
  /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/build/rendered/3dvar_fgat.yaml
```

Este caso testa o layout e YAML renderizado do workflow sem introduzir 64 ranks. Se B passa e D falha, a diferenca provavel esta no YAML renderizado, nos caminhos relativos/absolutos ou no layout do runtime.

### E. Workflow renderizado, 64 ranks

```bash
cd /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/build/runtime/jaci_3dvar_fgat_tutorial_2018041500/2018041500
/opt/cray/pals/1.6/bin/mpiexec -n 64 \
  /p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x \
  /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/build/rendered/3dvar_fgat.yaml
```

Este caso representa a pressao mais proxima do workflow paralelo. Deve ser interpretado depois dos casos B, C e D.

## Como submeter manualmente

Submeter a partir do diretorio `analysis` facilita rastrear quais scripts foram usados:

```bash
cd /p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/isolated_variational/analysis
```

Submissao caso a caso:

```bash
qsub run_matrix_A_ctest_3dvar_harness.pbs
qsub run_matrix_B_direct_3dvar_ctest_yaml_np1.pbs
qsub run_matrix_C_direct_3dvar_ctest_yaml_np64.pbs
qsub run_matrix_D_workflow_3dfgat_np1.pbs
qsub run_matrix_E_workflow_3dfgat_np64.pbs
```

Recomendacao: submeter primeiro A e B. So depois submeter C, D e E. Isso evita misturar falha de harness com falha de ranks ou de YAML.

## Como interpretar resultados

| Resultado | Interpretacao provavel |
| --- | --- |
| A passa, B trava | O problema esta associado ao harness CTest versus chamada direta. Comparar ambiente e controle de processo e a prioridade. |
| A passa, B passa, C falha | O problema esta associado a 64 ranks no YAML/layout oficial. Verificar particionamento, `x1.*.graph.info.part.*`, `mpiprocs` e compatibilidade do caso. |
| A passa, B passa, D falha | O problema esta associado ao YAML renderizado do workflow ou ao layout runtime do workflow. |
| D passa, E falha | O YAML/layout do workflow funciona com 1 rank, mas falha com 64. Investigar particionamento, arquivos `graph`, decomposicao e recursos PBS. |
| A falha | O controle positivo que ja havia passado deixou de passar. Revisar ambiente, modulos, disponibilidade de dados e estado do build antes de interpretar os outros casos. |
| B, C, D e E travam em `==> create geom`, mas A passa | A evidencia aponta fortemente para diferenca do harness CTest/processo/ambiente efetivo, nao para YAML isoladamente. |

## Observacoes sobre logs

Os scripts usam `#PBS -o` para gravar o stdout/stderr do script no log esperado. O comando principal tambem e redirecionado para o mesmo arquivo em modo append para preservar o cabecalho de diagnostico impresso antes da chamada MPI/CTest.

Ao analisar cada log, procurar:

```text
[INFO] command=
[INFO] workdir=
==> create geom
Run: Finishing oops::Variational
[TestReference] Comparison is done
[INFO] command exit status:
```

Se o log parar em `==> create geom` sem `command exit status`, o processo provavelmente ficou preso ou foi encerrado externamente sem retornar ao shell. Se houver `command exit status` diferente de zero, verificar as linhas imediatamente anteriores ao status.

## Proximo passo recomendado

Executar manualmente, nesta ordem:

1. `run_matrix_A_ctest_3dvar_harness.pbs`
2. `run_matrix_B_direct_3dvar_ctest_yaml_np1.pbs`
3. `run_matrix_C_direct_3dvar_ctest_yaml_np64.pbs`
4. `run_matrix_D_workflow_3dfgat_np1.pbs`
5. `run_matrix_E_workflow_3dfgat_np64.pbs`

Depois comparar os cinco logs com foco em ambiente, launcher MPI efetivo, workdir, YAML e ponto exato de parada.
