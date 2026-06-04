# Resultado FGAT np1 com MPASstatic sem observers

Data da análise: 2026-06-04

## Resumo

O teste `MPASstatic + no observers + np1` passou com sucesso.

Isso separa claramente o problema ativo:

- `FGAT/modelo/runtime/MPASstatic` funciona em np1 quando não há observers;
- o crash pós-forecast observado no caso `MPASstatic` com observers ativos entra pela pilha de observações, isto é, observers/IODA/filtros/HofX/GeoVaLs ou interação associada;
- a falha `BUMP_VerticalBalance` observada antes é um problema separado do bloco SABER/BUMP/VBAL.

## Arquivos criados

| Arquivo | Função |
|---|---|
| `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_no_observers_np1.yaml` | YAML isolado com `MPASstatic` e `observers: []` |
| `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/run_fgat64_mpastatic_no_observers_np1.pbs` | PBS np1 para executar o YAML acima |

## Validações antes da submissão

Foram confirmados:

- `yaml.safe_load`: passou;
- `background error: covariance model: MPASstatic`;
- `observers` é lista vazia, `len = 0`;
- ausência de `SABER`, `BUMP`, `NICAS` e `VBAL` no YAML;
- `bash -n run_fgat64_mpastatic_no_observers_np1.pbs`: passou;
- ausência de `qsub` interno no PBS.

## Job executado

| Item | Valor |
|---|---|
| Job ID | `258320.pbs-ha` |
| PBS | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/run_fgat64_mpastatic_no_observers_np1.pbs` |
| YAML | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_no_observers_np1.yaml` |
| Diretório de execução | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test` |
| Binário | `/p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x` |
| Launcher MPI | `/opt/cray/pals/1.6/bin/mpiexec` |
| Ranks | `1` |
| `OMP_NUM_THREADS` | `1` |
| `GFORTRAN_CONVERT_UNIT` | `big_endian:101-200` |
| `F_UFMTENDIAN` | `big:101-200` |
| `FI_CXI_RX_MATCH_MODE` | `hybrid` |
| Log PBS | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/logs/fgat64_mpastatic_no_observers_np1.pbs.out` |
| `log.atmosphere.0000.out` | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/log.atmosphere.0000.out` |
| `log.atmosphere.0000.err` | Não criado |
| `mpasjedi_variational.log` | Não criado |
| Status final | `0` |

Comando efetivo:

```bash
/opt/cray/pals/1.6/bin/mpiexec -n 1 \
  /p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x \
  /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_no_observers_np1.yaml
```

## Evidência principal

Trechos relevantes do log PBS:

```text
ObsSpaces<MODEL, OBS>::ObsSpaces: no obs spaces created
Model:forecast: forecast finished:
CostJo   : Nonlinear Jo = 0
CostJo   : Nonlinear Jo = 0
OOPS_STATS oops::State::write : 123.15 ...
OOPS_STATS Run end - Runtime: 93.62 sec, Memory: total: 4.39 GB
OOPS Ending 2026-06-04 14:26:39 (UTC+0000)
[INFO] command exit status: 0
```

O MPAS também terminou sem erro crítico:

```text
Finished running the atmosphere core
Total log messages printed:
  Output messages = 917
  Warning messages = 15
  Error messages = 0
  Critical error messages = 0
```

Persistem avisos já conhecidos no `log.atmosphere.0000.out`:

```text
Setting nominalMinDc to 480000.000000000 based on namelist option config_len_disp
WARNING: nominalMinDc was read from input file as a positive value (240000.000000000) that differs
--- subroutine MPAS_to_phys - pressure(1) < pressure(2):
```

Esses avisos não impediram o caso sem observers de concluir.

## Arquivos gerados

Foi criado o arquivo de análise:

```text
/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/analysis/analysis.2018-04-15_00.00.00.nc
```

Tamanho:

```text
31716080 bytes
```

Não houve arquivos novos em `feedback/`, como esperado, pois não havia observers.

Logs criados/atualizados:

- `fgat64_runtime_test/log.atmosphere.0000.out`
- `fgat64_runtime_test/log.atmosphere.0000.d0001.out`
- `logs/fgat64_mpastatic_no_observers_np1.pbs.out`

## Interpretação

Este é o primeiro caso isolado FGAT np1 da matriz que conclui com sucesso no runtime `x1.10242`.

A matriz agora indica:

| Caso | Observers | Covariância | Resultado | Diagnóstico |
|---|---:|---|---|---|
| FGAT base np1 | sim | SABER/BUMP/VBAL | falha `139` | crash pós-forecast |
| official_modvars_np1 | sim | SABER/BUMP/VBAL | falha `139` | não era só lista de model variables |
| no_observers_np1 | não | SABER/BUMP/VBAL | falha `139` | erro explícito em `BUMP_VerticalBalance`/variável 2D |
| mpastatic_np1 | sim | MPASstatic | falha `139` | crash pós-forecast com observers ativos |
| mpastatic_no_observers_np1 | não | MPASstatic | passa `0` | modelo/FGAT/runtime/MPASstatic básico está funcional |

Conclusão objetiva:

1. O runtime isolado, o forecast FGAT e a escrita de análise funcionam com `MPASstatic` quando não há observers.
2. O crash pós-forecast restante depende da presença dos observers.
3. O bloco SABER/BUMP/VBAL tem uma falha própria, mas não é a única causa do crash operacional completo.

## Próximos testes recomendados

A próxima etapa deve isolar os observers, ainda em np1 e ainda com `MPASstatic`, um por vez:

1. `MPASstatic + aircraft only + np1`
2. `MPASstatic + sondes only + np1`
3. `MPASstatic + sfc only + np1`

Interpretação esperada:

- Se apenas um observer falhar, investigar aquele ObsSpace/ObsOperator/filtros/HDF5.
- Se todos falharem individualmente, investigar caminho comum GeoVaLs/HofX/VertInterp/SfcCorrected ou incompatibilidade comum de variáveis/geometria.
- Se todos passarem individualmente, mas juntos falharem, investigar interação entre ObsSpaces, memória, filtros combinados ou escrita simultânea de feedback.

Não foi submetido nenhum próximo teste automaticamente.
