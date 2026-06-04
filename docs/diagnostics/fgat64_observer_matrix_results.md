# Resultados da matriz FGAT64 isolada

## Escopo

Esta etapa continuou o teste isolado FGAT64 no JACI sem alterar o workflow operacional e sem mexer nos scripts principais do `monan-jedi-workflow`.

A regra aplicada foi executar primeiro o caso full 3D-FGAT com 1 rank. Como esse caso falhou, os observers isolados em 64 ranks nao foram submetidos, conforme a interpretacao planejada: os testes `aircraft_only`, `sondes_only` e `sfc_only` so devem ser executados se o `np1` passar.

## Tabela consolidada

| caso | ranks | observers | resultado | ponto de falha | arquivos gerados | conclusao |
|---|---:|---|---|---|---|---|
| full_fgat_np1 | 1 | Aircraft, Radiosonde, SfcCorrected | falhou | depois de `Model:forecast: forecast finished`, com `rank 0 died from signal 11` e `command exit status: 139` | `log.atmosphere.0000.out`, `log.atmosphere.0000.d0001.out`; nenhum arquivo em `analysis/` ou `feedback/` | falha nao e exclusiva de 64 ranks; ha problema estrutural no caso/YAML/modelo/physics/observers ou na integracao pos-forecast mesmo em serial |
| aircraft_only_np64 | 64 | Aircraft | nao executado | nao aplicavel | nao aplicavel | pendente; nao submeter ate entender o `np1` |
| sondes_only_np64 | 64 | Radiosonde | nao executado | nao aplicavel | nao aplicavel | pendente; nao submeter ate entender o `np1` |
| sfc_only_np64 | 64 | SfcCorrected | nao executado | nao aplicavel | nao aplicavel | pendente; nao submeter ate entender o `np1` |

## Caso full_fgat_np1

PBS usado:

```text
/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/run_fgat64_isolated_runtime_np1.pbs
```

Job ID:

```text
257806.pbs-ha
```

YAML usado:

```text
/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.yaml
```

Diretorio de execucao:

```text
/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test
```

Log PBS:

```text
/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/logs/fgat64_isolated_runtime_np1.pbs.out
```

Logs MPAS encontrados:

```text
/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/log.atmosphere.0000.out
/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/log.atmosphere.0000.d0001.out
```

Nao foi encontrado `mpasjedi_variational.log` nem `log.atmosphere.0000.err` para esta rodada.

Comando registrado pelo PBS:

```text
/opt/cray/pals/1.6/bin/mpiexec -n 1 \
  /p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x \
  /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.yaml
```

Status final:

```text
[INFO] command exit status: 139
```

## Evidencias do log

Os tres ObsSpaces foram lidos antes da assimilacao incremental:

```text
Aircraft: read database from obs/aircraft_obs_2018041500.h5 (io pool size: 1)
Radiosonde: read database from obs/sondes_obs_2018041500.h5 (io pool size: 1)
SfcCorrected: read database from obs/sfc_obs_2018041500.h5 (io pool size: 1)
No bias-correction is performed for this ObsSpace.
No bias-correction is performed for this ObsSpace.
No bias-correction is performed for this ObsSpace.
```

O OOPS iniciou a assimilacao incremental:

```text
OOPS_STATS IncrementalAssimilation start            - Runtime:     3.95 sec,  Local Memory:     2.15 GB
OOPS_STATS IncrementalAssimilation iteration 0      - Runtime:     3.95 sec,  Local Memory:     2.15 GB
```

O forecast completou:

```text
Model:forecast: forecast finished:
  Valid time: 2018-04-15T03:00:00Z
```

A falha ativa ocorreu imediatamente depois:

```text
cn-0001.head.cm.cptec.inpe.br: rank 0 died from signal 11 and dumped core
[INFO] command exit status: 139
```

Nao ha mensagens `GeoVaLs`, `HofX`, `obsdataout`, `writing`, `IODA`, `CRITICAL`, `MPI_Abort` ou `ERROR` depois do forecast. Isso sugere uma falha nativa/segfault antes de o OOPS registrar a proxima etapa de alto nivel.

## Arquivos gerados

Nao houve arquivos em:

```text
fgat64_runtime_test/analysis/
fgat64_runtime_test/feedback/
```

Foram gerados apenas logs MPAS/OOPS no diretorio de execucao e no log PBS.

## Observacao importante sobre o forecast em 1 rank

Durante o forecast em 1 rank, o MPAS escreveu mensagens diagnosticas do tipo:

```text
--- subroutine MPAS_to_phys - pressure(1) < pressure(2):
```

com dumps de colunas verticais. A execucao continuou e chegou a `forecast finished`, mas esse comportamento e relevante: o estado/modelo apresenta colunas com ordenamento vertical de pressao inconsistente durante a fisica. Isso pode estar relacionado ao segfault posterior ou pode apenas expor um problema numerico preexistente.

## Interpretacao

A comparacao agora e decisiva:

- `np64 full`: falha depois do `forecast finished`, com `signal 11/15` e status 143.
- `np1 full`: falha depois do `forecast finished`, com `signal 11` e status 139.

Portanto, o crash nao e causado apenas por paralelismo/MPI em 64 ranks. O problema existe tambem em 1 rank, com o mesmo ponto logico de falha: apos a propagacao FGAT e antes da criacao de `analysis/` ou `feedback/`.

Neste momento a hipotese mais forte e problema estrutural do caso isolado/renderizado:

1. configuracao FGAT/YAML ainda nao validada para este conjunto de observers;
2. incompatibilidade entre background/model variables/physics e os operadores observacionais;
3. problema no caminho pos-forecast antes de registrar GeoVaLs/HofX;
4. instabilidade numerica exposta pela mensagem `pressure(1) < pressure(2)`;
5. possivel problema no estado x1.10242 usado como `templateFields`/background para iniciar a trajetoria.

Como `np1` falhou, os testes observers-only em 64 ranks nao isolariam corretamente um problema de observer paralelo neste momento. A prioridade deve ser reduzir o caso em 1 rank primeiro.

## Proximo diagnostico recomendado

Criar uma segunda matriz em 1 rank, ainda no teste isolado:

1. `np1_aircraft_only`
2. `np1_sondes_only`
3. `np1_sfc_only`
4. opcional: `np1_no_observers` ou um teste de forecast/model-only, se houver YAML JEDI apropriado

A pergunta agora deve mudar para:

```text
Qual observer, filtro ou componente do YAML faz o caso serial falhar depois do forecast?
```

Se todos os observers isolados em `np1` falharem no mesmo ponto, investigar primeiro o modelo/estado/physics/GeoVaLs comum. Se apenas um observer falhar, investigar esse ObsSpace/operator/filtros/HDF5. Se nenhum observer isolado falhar, investigar interacao entre observers ou memoria no conjunto completo.

Nao e recomendado reconstruir o workflow operacional ainda.
