# Analise da execucao CTest do `mpasjedi_3dvar`

Data da analise: 2026-06-03

Escopo: esta analise usa apenas arquivos ja existentes no repositorio, no build e nos logs PBS. Nenhum job PBS novo foi submetido e o executavel `mpasjedi_variational.x` nao foi executado nesta etapa.

## Resultado resumido

O teste oficial que passa e o CTest `mpasjedi_3dvar`, registrado no build em:

```text
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/CTestTestfile.cmake
```

O comando visivel registrado pelo CTest e:

```bash
/opt/cray/pals/1.6/bin/mpiexec -n 1 \
  /p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/bin/mpasjedi_variational.x \
  testinput/3dvar.yaml
```

As variaveis de ambiente explicitas do teste sao:

```bash
GFORTRAN_CONVERT_UNIT=big_endian:101-200
OMP_NUM_THREADS=1
```

O CTest foi executado com sucesso a partir do build:

```bash
cd /p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build
ctest --output-on-failure -R '^mpasjedi_3dvar$'
```

No PBS que funcionou, o log registrou:

```text
Start 2268: mpasjedi_3dvar
1/1 Test #2268: mpasjedi_3dvar ...................   Passed    5.91 sec
100% tests passed, 0 tests failed out of 1
```

## Registro CTest

Trecho relevante de `CTestTestfile.cmake`:

```cmake
add_test(mpasjedi_3dvar
  "/opt/cray/pals/1.6/bin/mpiexec"
  "-n" "1"
  "/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/bin/mpasjedi_variational.x"
  "testinput/3dvar.yaml")

set_tests_properties(mpasjedi_3dvar PROPERTIES
  ENVIRONMENT "GFORTRAN_CONVERT_UNIT=big_endian:101-200;OMP_NUM_THREADS=1"
  LABELS "mpasjedi;script;mpi")
```

Tambem existe um teste relacionado, mas diferente, com 2 ranks:

```cmake
add_test(mpasjedi_3dvar_2pe
  "/opt/cray/pals/1.6/bin/mpiexec"
  "-n" "2"
  "/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/bin/mpasjedi_variational.x"
  "testinput/3dvar.yaml")
```

O teste solicitado e `mpasjedi_3dvar`, portanto o caso oficial usa 1 rank, nao 2.

## Diretorio de trabalho

Nao ha propriedade `WORKING_DIRECTORY` explicita para `mpasjedi_3dvar` no `CTestTestfile.cmake`. Para testes registrados nesse arquivo, o diretorio de trabalho efetivo e o diretorio onde o arquivo de testes esta localizado:

```text
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test
```

Isso e importante porque o YAML usa caminhos relativos, por exemplo:

```yaml
geometry:
  nml_file: "./Data/480km/namelist.atmosphere_2018041500"
  streams_file: "./Data/480km/streams.atmosphere"
```

## YAML usado

O CTest passa este argumento ao executavel:

```text
testinput/3dvar.yaml
```

No diretorio de trabalho do CTest, esse arquivo e um link para o fonte:

```text
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/testinput/3dvar.yaml
  -> /p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/3dvar.yaml
```

Blocos principais do YAML:

- `test`: define tolerancias e arquivos de referencia/log.
- `cost function`: define `cost type: 3D-Var`.
- `time window`: janela de assimilacao de 2018-04-14T21:00:00Z por 6 horas.
- `geometry`: aponta para `./Data/480km/namelist.atmosphere_2018041500` e `./Data/480km/streams.atmosphere`.
- `background`: usa o estado `./Data/480km/bg/mpasout.2018-04-15_00.00.00.nc`.
- `background error`: usa `covariance model: MPASstatic`.
- `observations`: configura radiossonda, aircraft, GNSS-RO, superficie e outros conjuntos observacionais UFO/IODA.

Arquivos de teste definidos no bloco `test`:

```text
reference filename: testoutput/3dvar.ref
log output filename: testoutput/3dvar.run
test output filename: testoutput/3dvar.run.ref
```

## Arquivos e diretorios auxiliares

O diretorio CTest contem arquivos estaticos e links exigidos pelo MPAS-JEDI:

```text
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/Data
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/testinput
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/testoutput
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/geovars.yaml
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/keptvars.yaml
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/x1.2562.graph.info
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/x1.2562.graph.info.part.2
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/x1.2562.graph.info.part.4
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/x1.2562.graph.info.part.8
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test/x1.2562.graph.info.part.16
```

Tambem ha tabelas MPAS no diretorio de execucao, como `GENPARM.TBL`, `LANDUSE.TBL`, `VEGPARM.TBL`, `SOILPARM.TBL`, `RRTMG_*`, `OZONE_*` e `CAM_*`.

Exemplos de dados acessados via links:

```text
Data/480km/bg
  -> /p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi-data/testinput_tier_1/480km/bg

Data/ufo/testinput_tier_1
  -> /p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1
```

## Comparacao: CTest vs execucao direta

| Item | CTest `mpasjedi_3dvar` | Execucao direta testada |
| --- | --- | --- |
| Harness | `ctest` | shell/PBS chamando o binario diretamente |
| Comando visivel | `mpiexec -n 1 mpasjedi_variational.x testinput/3dvar.yaml` | mesmo padrao foi tentado |
| Launcher MPI | `/opt/cray/pals/1.6/bin/mpiexec` | ajustado para `/opt/cray/pals/1.6/bin/mpiexec` nas tentativas posteriores |
| Ranks | 1 | inicialmente houve variacao; depois foi ajustado para 1 |
| YAML | `testinput/3dvar.yaml` no build/test | mesmo YAML ou link equivalente |
| Workdir | inferido como `build/mpas-jedi/test` | tambem foi tentado dentro de `build/mpas-jedi/test` |
| Env explicito do teste | `GFORTRAN_CONVERT_UNIT=big_endian:101-200`, `OMP_NUM_THREADS=1` | tambem exportado nos scripts |
| Resultado | passa em ~5.91 s | para/trava apos `==> create geom` |

O executavel instalado e o executavel do build foram comparados anteriormente e tinham o mesmo hash SHA-256:

```text
6edd541aecec4aa42045317db99dde7e36e6a0672a0aa7b267f79368e794e59a
```

Portanto, a diferenca observada nao parece ser o binario.

## Evidencia do travamento

Nos logs diretos:

```text
Run: Starting oops::Variational<MPAS, UFO and IODA observations>
==> create geom
```

Nao ha finalizacao do `Variational` nem comparacao de referencia nesses logs.

No log `testoutput/3dvar.run` produzido pelo CTest, a execucao passa por `create geom` e termina com sucesso:

```text
Run: Finishing oops::Variational<MPAS, UFO and IODA observations> with status = 0
[TestReference] Comparison is done
OOPS Ending   2026-06-03 18:44:55 (UTC+0000)
```

## Hipotese principal

A diferenca principal encontrada nao esta no YAML, no numero de ranks, no launcher MPI visivel ou no executavel. A diferenca empiricamente relevante e o harness: o caminho que passa e o `ctest -R '^mpasjedi_3dvar$'`, enquanto a chamada direta do binario, mesmo tentando reproduzir os elementos visiveis do CTest, trava em `==> create geom`.

As hipoteses mais provaveis sao:

- o ambiente herdado pelos scripts diretos, especialmente por causa de `#PBS -V`, ainda nao e identico ao ambiente efetivo criado pelo CTest para o teste;
- o CTest aplica controle de processo, diretorio de teste, stdout/stderr, stdin, grupo de processo ou contexto de execucao que altera o comportamento do PALS/MPI;
- alguma variavel MPI/PALS/Cray ou OOPS herdada no shell direto interfere na inicializacao da geometria;
- a reproducao direta ainda nao capturou todas as propriedades reais do teste porque `CTestTestfile.cmake` mostra apenas as propriedades explicitas, nao todo o ambiente herdado pelo processo `ctest`.

## Comando manual equivalente inferido

O equivalente visivel ao CTest e:

```bash
cd /p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test

GFORTRAN_CONVERT_UNIT=big_endian:101-200 \
OMP_NUM_THREADS=1 \
/opt/cray/pals/1.6/bin/mpiexec -n 1 \
  /p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/bin/mpasjedi_variational.x \
  testinput/3dvar.yaml
```

Entretanto, esse comando deve ser tratado como equivalente apenas do ponto de vista do registro CTest. Pelas evidencias coletadas, a execucao direta desse padrao ainda nao reproduz de forma confiavel o comportamento do harness CTest no JACI.

## Recomendacao para PBS minimo reprodutivel

Para reproduzir o teste oficial agora, o caminho recomendado e submeter um PBS minimo que execute o CTest, nao o binario diretamente:

```bash
cd /p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build
ctest --output-on-failure -R '^mpasjedi_3dvar$'
```

Esse e o caminho comprovadamente funcional no job `257453.pbs-ha`.

Para investigar a causa raiz da diferenca, o proximo passo recomendado e criar uma instrumentacao temporaria que capture o ambiente efetivo imediatamente antes do `mpiexec` nos dois caminhos:

```bash
pwd
env | sort
ulimit -a
module list
which ctest
which mpiexec
/opt/cray/pals/1.6/bin/mpiexec --version
```

Idealmente, deve-se registrar um wrapper temporario no CTest ou usar uma variante temporaria do teste que faca dump de ambiente e depois execute exatamente o comando original. Esse dump deve ser comparado com o dump do PBS direto. Sem essa captura, a diferenca fica restrita ao que o `CTestTestfile.cmake` expoe, e isso nao explica sozinho o travamento.

## Conclusao

O comando real registrado para `mpasjedi_3dvar` e uma chamada direta via PALS `mpiexec` com 1 rank e `testinput/3dvar.yaml`, sob o harness CTest. O CTest define explicitamente apenas `GFORTRAN_CONVERT_UNIT=big_endian:101-200` e `OMP_NUM_THREADS=1`.

A principal diferenca encontrada e que o CTest passa sob controle do proprio `ctest`, enquanto a execucao direta do binario, mesmo aproximando comando, workdir, YAML, launcher e variaveis explicitas, trava em `==> create geom`. Assim, a recomendacao objetiva e usar o PBS baseado em `ctest -R '^mpasjedi_3dvar$'` para reproducao operacional e instrumentar ambiente/processo antes de tentar substituir o CTest por chamada direta do binario.
