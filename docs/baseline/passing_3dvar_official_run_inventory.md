# Inventario da rodada oficial MPAS-JEDI 3DVar que passou

Data da analise: 2026-06-04.

## Escopo

Este documento inventaria apenas a rodada oficial MPAS-JEDI `mpasjedi_3dvar` que ja passou. Nenhum workflow operacional, YAML principal, PBS ou runtime foi alterado nesta analise.

## Baseline que passou

Teste oficial CTest:

```sh
cd /p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build
ctest --output-on-failure -R '^mpasjedi_3dvar$'
```

Comando registrado pelo CTest:

```sh
/opt/cray/pals/1.6/bin/mpiexec -n 1 \
  /p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/bin/mpasjedi_variational.x \
  testinput/3dvar.yaml
```

Chamada direta equivalente que tambem passou:

```sh
cd /p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test

/opt/cray/pals/1.6/bin/mpiexec -n 1 \
  /p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/bin/mpasjedi_variational.x \
  testinput/3dvar.yaml
```

Diretorio efetivo de execucao:

```text
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test
```

Binario usado:

```text
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/bin/mpasjedi_variational.x
```

YAML usado:

```text
testinput/3dvar.yaml
```

No build, esse arquivo e symlink para:

```text
/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/3dvar.yaml
```

## Ranks

| Caso | Status | Evidencia |
| --- | --- | --- |
| `np1` | passou | `CTestTestfile.cmake` define `mpasjedi_3dvar` com `mpiexec -n 1`; `LastTest.log` termina com `Run ... status = 0` e `Test Passed.` |
| chamada direta `np1` | passou | mesmo comando e diretorio do CTest, usando `testinput/3dvar.yaml` |
| `np2` oficial | definido pelo CTest | `CTestTestfile.cmake` define `mpasjedi_3dvar_2pe` com `mpiexec -n 2` |
| `np16` chamada direta | passou conforme contexto operacional informado | o diretorio de execucao contem `x1.2562.graph.info.part.16` |
| `np64` chamada direta | falhou por falta de decomposicao | a namelist exige prefixo `x1.2562.graph.info.part.`; existem `part.2`, `part.4`, `part.8`, `part.16`, mas nao `part.64` |

Arquivos de decomposicao disponiveis no diretorio oficial:

```text
x1.2562.graph.info
x1.2562.graph.info.part.2
x1.2562.graph.info.part.4
x1.2562.graph.info.part.8
x1.2562.graph.info.part.16
```

Arquivo ausente para `np64`:

```text
x1.2562.graph.info.part.64
```

## Variaveis de ambiente criticas

Do `CTestTestfile.cmake`, para `mpasjedi_3dvar`:

```text
GFORTRAN_CONVERT_UNIT=big_endian:101-200
OMP_NUM_THREADS=1
```

MPI launcher:

```text
/opt/cray/pals/1.6/bin/mpiexec
```

Variaveis e ambiente vistos em logs PBS/workflow anteriores, relevantes mas nao parte do CTest oficial:

| Variavel/componente | Evidencia observada | Observacao |
| --- | --- | --- |
| `cray-mpich/8.1.31/none/none/jedi-mpas-env/1.0.0` | logs PBS carregam esse modulo | relevante para chamadas PBS/workflow, nao necessario para reproduzir o CTest ja configurado no build |
| `MODULEPATH`, `LD_LIBRARY_PATH`, `MPICH_DIR`, `CRAY_MPICH_*` | logs PBS mostram purge/load de modulo | ambiente operacional; nao foi reconfigurado nesta analise |
| `PBS_*` | logs PBS anteriores existem | fora do baseline CTest oficial |

## Logs que comprovam sucesso

| Log | Evidencia |
| --- | --- |
| `/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/Testing/Temporary/LastTest.log` | registra o comando CTest, `Run: Finishing ... with status = 0`, `Test Passed.`, tempo `00:00:13` |
| `/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/testoutput/3dvar.run` | log salvo pela infraestrutura `TestReference` |
| `/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/testoutput/3dvar.run.ref` | saida de teste gerada para comparacao |
| `/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/log.atmosphere.0000.out` | log MPAS da execucao direta/recente no diretorio oficial |

Linhas-chave observadas:

```text
Command: "/opt/cray/pals/1.6/bin/mpiexec" "-n" "1" ".../mpasjedi_variational.x" "testinput/3dvar.yaml"
Run: Finishing oops::Variational<MPAS, UFO and IODA observations> with status = 0
Test Passed.
```

## Arquivos gerados pela rodada

Arquivos de saida gerados/atualizados no diretorio oficial:

| Arquivo | Funcao |
| --- | --- |
| `testoutput/3dvar.run` | log de execucao salvo pelo teste |
| `testoutput/3dvar.run.ref` | saida numerica/test output salvo pelo teste |
| `Data/states/mpas.3dvar.2018-04-15_00.00.00.nc` | estado de analise MPAS escrito por `output.filename` |
| `Data/os/obsout_3dvar_sondes.nc4` | feedback IODA para radiossonda |
| `Data/os/obsout_3dvar_aircraft.nc4` | feedback IODA para aircraft |
| `Data/os/obsout_3dvar_gnssroref.nc4` | feedback IODA para GNSSRO refractivity |
| `Data/os/obsout_3dvar_sfc.nc4` | feedback IODA para superficie T/Q/Ps |
| `Data/os/obsout_3dvar_sfc_wind.nc4` | feedback IODA para vento de superficie |
| `Data/os/obsout_3dvar_amsua_n19--nohydro.nc4` | feedback IODA AMSUA canais sem hidrometeoros |
| `Data/os/obsout_3dvar_amsua_n19--hydro.nc4` | feedback IODA AMSUA canais com hidrometeoros |
| `Data/os/geoval_out_*.nc` | saidas do filtro `GOMsaver`; no caso observado, particionadas por rank de uma rodada recente |
| `log.atmosphere.0000.out`, `log.atmosphere.0000.err`, `log.atmosphere.0000.d000*.out` | logs MPAS no diretorio de execucao |

## Observacoes sobre caminhos relativos

Todos os caminhos no YAML oficial sao resolvidos a partir do diretorio efetivo:

```text
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test
```

Exemplos:

| Referencia no YAML | Resolve para |
| --- | --- |
| `testinput/3dvar.yaml` | symlink para fonte oficial em `projects/MONAN-JEDI/mpas-jedi/test/testinput/3dvar.yaml` |
| `./Data/480km/namelist.atmosphere_2018041500` | symlink em `Data/480km` para namelist oficial |
| `./Data/480km/streams.atmosphere` | symlink em `Data/480km` para streams oficial |
| `./Data/480km/bg/mpasout.2018-04-15_00.00.00.nc` | arquivo em `projects/MONAN-JEDI/mpas-jedi-data/testinput_tier_1/480km/bg` |
| `Data/ufo/testinput_tier_1/*.nc4` | symlink `Data/ufo/testinput_tier_1` para `projects/MONAN-JEDI/ufo-data/testinput_tier_1` |
| `Data/UFOCoeff/` | symlinks para `projects/MONAN-JEDI/test-data-release/crtm/fix_REL-3.1.2.0/fix` |
| `testinput/obsop_name_map.yaml` | symlink/fonte oficial em `mpas-jedi/test/testinput` |

Conclusao operacional: copiar apenas o YAML para outro diretorio nao reproduz a rodada. A estrutura relativa `Data/`, `testinput/`, os symlinks de stream list e os arquivos MPAS no diretorio de execucao fazem parte do contrato da rodada.
