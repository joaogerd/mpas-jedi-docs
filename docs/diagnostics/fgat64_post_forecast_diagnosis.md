# Diagnostico pos-forecast do 3D-FGAT isolado

## Resultado observado

A execucao isolada 3D-FGAT com 64 ranks nao falha mais em `create geom` nem na leitura de `templateFields.10242.nc`.

O PBS registrou:

```text
[CHECK] templateFields.10242.nc time metadata
initial_time = "2018-04-14_18:00:00"
xtime        = "2018-04-14_21:00:00"
```

O MPAS leu o estado inicial e completou a propagacao do modelo:

```text
Reading initial state from 'input' stream
----- done reading initial state -----
Model:forecast: forecast finished:
  Valid time: 2018-04-15T03:00:00Z
```

O termino ativo foi:

```text
rank 51 died from signal 11 and dumped core
rank 46 died from signal 15
command exit status: 143
```

Nao foram criados arquivos em `analysis/` nem em `feedback/`.

## Interpretacao

A falha esta depois da propagacao do modelo FGAT e antes da escrita de `analysis/`/`feedback/`. O ponto mais provavel e a transicao pos-forecast para calculo de HofX/GeoVaLs, filtros dos observers, escrita IODA ou algum passo OOPS associado ao primeiro uso das observacoes depois da trajetoria do modelo.

Os tres ObsSpaces foram lidos antes do forecast:

```text
Aircraft: read database from obs/aircraft_obs_2018041500.h5
Radiosonde: read database from obs/sondes_obs_2018041500.h5
SfcCorrected: read database from obs/sfc_obs_2018041500.h5
```

Os `dateTime` dos HDF5 estao dentro da janela FGAT `2018-04-14T21:00:00Z` a `2018-04-15T03:00:00Z`.

## Artefatos de diagnostico criados

YAMLs isolados adicionais:

```text
fgat64_runtime_test/3dvar_fgat.fgat64.aircraft_only.yaml
fgat64_runtime_test/3dvar_fgat.fgat64.sondes_only.yaml
fgat64_runtime_test/3dvar_fgat.fgat64.sfc_only.yaml
```

PBS adicionais, nao submetidos:

```text
run_fgat64_isolated_runtime_np1.pbs
run_fgat64_aircraft_only_np64.pbs
run_fgat64_sondes_only_np64.pbs
run_fgat64_sfc_only_np64.pbs
```

Todos usam o mesmo diretorio isolado e arquivam logs `log.atmosphere*` antigos antes da execucao.

## Matriz recomendada

Executar um por vez.

1. Full 3D-FGAT com 1 rank:

```bash
cd /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis
qsub run_fgat64_isolated_runtime_np1.pbs
```

Se passar, o problema e paralelo/64 ranks. Se falhar no mesmo ponto, o problema e configuracao cientifica/YAML/observers independente de MPI.

2. Aircraft-only com 64 ranks:

```bash
qsub run_fgat64_aircraft_only_np64.pbs
```

3. Radiosonde-only com 64 ranks:

```bash
qsub run_fgat64_sondes_only_np64.pbs
```

4. SfcCorrected-only com 64 ranks:

```bash
qsub run_fgat64_sfc_only_np64.pbs
```

Interpretacao dos observers isolados:

- se apenas um observer falhar, investigar esse operador/filtros/HDF5;
- se todos falharem em 64 ranks, investigar paralelismo no caminho FGAT pos-forecast, GetValues/GeoVaLs ou finalizacao MPI/OOPS;
- se nenhum falhar isolado, investigar interacao entre multiplos ObsSpaces e escrita simultanea de feedback.

## Evidencias principais

- `create geom`: passou.
- `templateFields.10242.nc`: tempo correto e leitura inicial passou.
- `x1.10242.graph.info.part.64`: existente e usado.
- Forecast MPAS: completou ate `2018-04-15T03:00:00Z`.
- Falha atual: `SIGSEGV` em rank MPI apos forecast, antes de `analysis/` e `feedback/`.
