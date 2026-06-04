# Resultado FGAT np1 sem observers

Data da análise: 2026-06-04

## Resumo

O teste `no_observers_np1` foi executado como próximo caso da matriz FGAT np1. Ele não passou, mas avançou além do ponto dos casos anteriores com observers.

Diferença importante:

- com observers, o caso falhava logo após `Model:forecast: forecast finished`, antes de qualquer mensagem clara posterior;
- sem observers, o forecast terminou, `Jo = 0` foi reportado corretamente, e o código entrou na criação dos blocos SABER;
- a falha explícita ocorreu em `BUMP_VerticalBalance` com a mensagem:

```text
!!! ABORT in nam_from_conf on task #000001: level for 2d variables not specified but 2d variables present
```

Portanto, este teste mostra que o problema não é exclusivamente dos observers, filtros, GeoVaLs, HofX ou escrita IODA. Há uma falha material no caminho de covariância SABER/BUMP/VBAL do YAML isolado.

## Job executado

| Item | Valor |
|---|---|
| Job ID | `258288.pbs-ha` |
| PBS | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/run_fgat64_no_observers_np1.pbs` |
| YAML | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.no_observers.yaml` |
| Diretório de execução | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test` |
| Binário | `/p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x` |
| Launcher MPI | `/opt/cray/pals/1.6/bin/mpiexec` |
| Ranks | `1` |
| `OMP_NUM_THREADS` | `1` |
| `GFORTRAN_CONVERT_UNIT` | `big_endian:101-200` |
| `F_UFMTENDIAN` | `big:101-200` |
| `FI_CXI_RX_MATCH_MODE` | `hybrid` |
| Log PBS | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/logs/fgat64_no_observers_np1.pbs.out` |
| `log.atmosphere.0000.out` | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/log.atmosphere.0000.out` |
| `log.atmosphere.0000.err` | Não criado |
| `mpasjedi_variational.log` | Não criado |
| Status final | `139` |

Comando efetivo registrado no PBS:

```bash
/opt/cray/pals/1.6/bin/mpiexec -n 1 \
  /p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x \
  /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.no_observers.yaml
```

## Configuração testada

O YAML mantém:

- `cost type: 3D-FGAT`
- janela: `2018-04-14T21:00:00Z` por `PT6H`
- `model tstep: PT45M`
- geometria local: `namelist.atmosphere.outer` e `streams.atmosphere.outer`
- background: `background/mpasout.2018-04-14_21.00.00.nc`
- covariância: `SABER` com `BUMP_NICAS`, `StdDev` e `BUMP_VerticalBalance`
- `linear variable change: Control2Analysis`
- `observations.observers: []`

O log confirma que nenhum ObsSpace foi criado:

```text
ObsSpaces<MODEL, OBS>::ObsSpaces: no obs spaces created
```

## Ponto de falha

Trecho decisivo do log PBS:

```text
Model:forecast: forecast finished:
  Valid time: 2018-04-15T03:00:00Z
  Resolution: nCellsGlobal = 10242, nFields = 24
...
CostJo Observations:
Jo Observations:
Jo Observations Equivalent:
Jo Bias Corrected Departures:
Jo Observations Errors:

CostJo   : Nonlinear Jo = 0
Info     : Prepare ensemble configuration
Info     : Read full ensemble
Info     : Creating outer blocks
Info     : Creating outer block: BUMP_VerticalBalance
...
Info     :        Model:{"variables":["stream_function","velocity_potential","temperature","spechum","surface_pressure"],"2d variables":["surface_pressure"],"do not cross mask boundaries":false,"nl0":55,"nearest 3d level":"","levels direction":true}
!!! ABORT in nam_from_conf on task #000001: level for 2d variables not specified but 2d variables present
MPICH ERROR [Rank 0] ... Abort(1) ... application called MPI_Abort(MPI_COMM_WORLD, 1)
cn-0017.head.cm.cptec.inpe.br: rank 0 died from signal 11 and dumped core
[INFO] command exit status: 139
```

Esse erro é diferente e mais informativo que o segfault pós-forecast dos casos com observers. O abort vem do BUMP ao configurar variáveis 2D, especificamente `surface_pressure`, sem um nível associado para variáveis 2D.

## Arquivos gerados

Nenhum arquivo de produto foi criado em:

- `fgat64_runtime_test/analysis/`
- `fgat64_runtime_test/feedback/`

Logs criados/atualizados no runtime:

- `fgat64_runtime_test/log.atmosphere.0000.out`
- `fgat64_runtime_test/log.atmosphere.0000.d0001.out`

Log PBS criado:

- `logs/fgat64_no_observers_np1.pbs.out`

## Interpretação

Este teste muda a conclusão anterior:

1. O forecast/modelo consegue terminar também sem observers.
2. A remoção dos observers permitiu revelar uma falha explícita no bloco SABER/BUMP/VBAL.
3. O problema não está apenas em IODA, `obsdataout`, filtros ou observers.
4. A configuração de `BUMP_VerticalBalance` no YAML isolado está incompleta ou incompatível para variáveis 2D, porque `surface_pressure` aparece em `2d variables`, mas não há especificação de nível para variáveis 2D.

A presença de `signal 11` continua aparecendo no encerramento do MPI, mas aqui há uma causa textual anterior e mais útil: o `ABORT in nam_from_conf`.

## Próximo teste recomendado

O próximo teste mais informativo é trocar temporariamente a covariância `SABER/BUMP/NICAS/VBAL` por `MPASstatic`, aproximando o YAML isolado dos YAMLs oficiais `3dvar.yaml`/`3dfgat.yaml` que já passaram no ambiente MPAS-JEDI.

Justificativa:

- `no_observers_np1` removeu a pilha UFO/IODA e ainda falhou;
- a falha explícita está em `BUMP_VerticalBalance`;
- testar observers individuais agora seria menos informativo, porque existe uma falha de covariância independente dos observers;
- aproximar a background error covariance de `MPASstatic` separa o problema do FGAT/modelo contra o problema SABER/BUMP/VBAL.

Recomendação objetiva para a próxima etapa, sem executar automaticamente:

1. Criar um YAML isolado np1 com os mesmos dados/runtime, mas substituindo `background error: covariance model: SABER` por o bloco `MPASstatic` do `3dfgat.yaml` oficial, ajustando caminhos para o diretório isolado.
2. Criar um PBS np1 correspondente.
3. Submeter apenas esse caso depois, em nova solicitação explícita.

Se o caso com `MPASstatic` passar, a causa provável fica concentrada em SABER/BUMP/VBAL. Se também falhar após o forecast, a investigação deve voltar para estado inicial, namelist/physics e consistência de malha `x1.10242`.
