# Resultado FGAT np1 com MPASstatic

Data da análise: 2026-06-04

## Resumo

Foi criado e testado um caso isolado FGAT np1 usando `MPASstatic` no lugar de `SABER/BUMP/NICAS/VBAL`.

O primeiro envio (`258303.pbs-ha`) falhou antes do diagnóstico porque o YAML gerado via `safe_dump` havia convertido datas para o formato `2018-04-14 21:00:00+00:00`, rejeitado pelo OOPS. O YAML foi corrigido na área isolada para manter datas como strings ISO `2018-04-14T21:00:00Z`, revalidado, e o mesmo caso foi reenviado como `258304.pbs-ha`.

Resultado da rodada válida: falhou com status `139` logo após `Model:forecast: forecast finished`, com `rank 0 died from signal 11 and dumped core`. Não houve arquivos novos em `analysis/` ou `feedback/`.

Conclusão principal: trocar `SABER/BUMP/VBAL` por `MPASstatic` remove a falha explícita do BUMP observada no caso `no_observers_np1`, mas não resolve o crash do FGAT com observers ativos. Portanto, há pelo menos dois problemas separados:

1. configuração SABER/BUMP/VBAL incompleta/incompatível para variável 2D `surface_pressure`;
2. crash pós-forecast quando observers estão ativos, mesmo com `MPASstatic`.

## Arquivos criados

| Arquivo | Função |
|---|---|
| `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_np1.yaml` | YAML isolado FGAT np1 com `background error: MPASstatic` |
| `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/run_fgat64_mpastatic_np1.pbs` | PBS np1 para executar o YAML acima |

## Validações estáticas

Executadas antes da rodada válida:

```bash
python3 - <<'PY'
from pathlib import Path
import yaml
p = Path('/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_np1.yaml')
with p.open() as f:
    cfg = yaml.safe_load(f)
print(cfg['cost function']['background error'])
PY
```

Resultado relevante:

```text
{'covariance model': 'MPASstatic', 'date': '2018-04-14T21:00:00Z'}
```

Também foram verificados:

- `bash -n run_fgat64_mpastatic_np1.pbs`: passou;
- ausência de `qsub` interno no PBS: confirmada;
- ausência de `SABER`, `BUMP`, `NICAS` e `VBAL` no YAML MPASstatic: confirmada;
- datas do YAML como strings ISO com `T` e `Z`: confirmado.

## Job executado

| Item | Valor |
|---|---|
| Job inicial malformado | `258303.pbs-ha` |
| Job válido analisado | `258304.pbs-ha` |
| PBS | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/run_fgat64_mpastatic_np1.pbs` |
| YAML | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_np1.yaml` |
| Diretório de execução | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test` |
| Binário | `/p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x` |
| Launcher MPI | `/opt/cray/pals/1.6/bin/mpiexec` |
| Ranks | `1` |
| `OMP_NUM_THREADS` | `1` |
| `GFORTRAN_CONVERT_UNIT` | `big_endian:101-200` |
| `F_UFMTENDIAN` | `big:101-200` |
| `FI_CXI_RX_MATCH_MODE` | `hybrid` |
| Log PBS | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/logs/fgat64_mpastatic_np1.pbs.out` |
| `log.atmosphere.0000.out` | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/log.atmosphere.0000.out` |
| `log.atmosphere.0000.err` | Não criado |
| `mpasjedi_variational.log` | Não criado |
| Status final | `139` |

Comando efetivo:

```bash
/opt/cray/pals/1.6/bin/mpiexec -n 1 \
  /p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x \
  /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.mpastatic_np1.yaml
```

## Configuração testada

O YAML válido usou:

```yaml
cost function:
  cost type: 3D-FGAT
  time window:
    begin: '2018-04-14T21:00:00Z'
    length: PT6H
  background error:
    covariance model: MPASstatic
    date: '2018-04-14T21:00:00Z'
```

Os três observers permaneceram ativos:

- `Aircraft`
- `Radiosonde`
- `SfcCorrected`

O log confirmou leitura dos três ObsSpaces antes do forecast.

## Ponto de falha

Trecho decisivo do log PBS:

```text
Aircraft: read database from obs/aircraft_obs_2018041500.h5 (io pool size: 1)
Radiosonde: read database from obs/sondes_obs_2018041500.h5 (io pool size: 1)
SfcCorrected: read database from obs/sfc_obs_2018041500.h5 (io pool size: 1)
Minimizer algorithm=DRPCG
Model:forecast: forecast finished:
  Valid time: 2018-04-15T03:00:00Z
...
cn-0014.head.cm.cptec.inpe.br: rank 0 died from signal 11 and dumped core
[INFO] command exit status: 139
```

Não há mensagem de `BUMP_VerticalBalance` nesta rodada válida, porque a covariância foi `MPASstatic`. Também não há mensagem clara posterior de GeoVaLs/HofX/IODA antes do `signal 11`.

O `log.atmosphere.0000.out` manteve os avisos já observados:

```text
Setting nominalMinDc to 480000.000000000 based on namelist option config_len_disp
WARNING: nominalMinDc was read from input file as a positive value (240000.000000000) that differs
...
--- subroutine MPAS_to_phys - pressure(1) < pressure(2):
```

## Arquivos gerados

Nenhum arquivo de produto foi criado em:

- `fgat64_runtime_test/analysis/`
- `fgat64_runtime_test/feedback/`

Logs criados/atualizados:

- `fgat64_runtime_test/log.atmosphere.0000.out`
- `fgat64_runtime_test/log.atmosphere.0000.d0001.out`
- `logs/fgat64_mpastatic_np1.pbs.out`

## Interpretação

A troca para `MPASstatic` respondeu parcialmente à pergunta diagnóstica:

- O erro explícito do `no_observers_np1` era de fato específico de `SABER/BUMP/VBAL`.
- Porém, o FGAT com observers ativos ainda falha após o forecast mesmo sem SABER/BUMP/VBAL.
- Isso enfraquece a hipótese de que a falha operacional completa seja causada apenas pela covariância SABER/BUMP.

Comparação dos pontos de falha:

| Caso | Observers | Covariância | Resultado | Ponto de falha |
|---|---:|---|---|---|
| FGAT base np1 | sim | SABER/BUMP/VBAL | falha `139` | logo após forecast |
| official_modvars_np1 | sim | SABER/BUMP/VBAL | falha `139` | logo após forecast |
| no_observers_np1 | não | SABER/BUMP/VBAL | falha `139` | `BUMP_VerticalBalance`: variável 2D sem nível |
| mpastatic_np1 | sim | MPASstatic | falha `139` | logo após forecast |

## Próximo teste recomendado

O próximo teste mais informativo é `mpastatic_no_observers_np1`.

Justificativa:

- `no_observers_np1` mostrou que sem observers o código passa do forecast e chega à covariância, mas falha no BUMP;
- `mpastatic_np1` removeu BUMP, mas com observers voltou ao crash pós-forecast;
- falta testar a combinação `sem observers + MPASstatic` para separar se o par `FGAT/modelo/runtime + MPASstatic` é capaz de concluir sem UFO/IODA.

Interpretação esperada do próximo caso:

- Se `mpastatic_no_observers_np1` passar, o caminho modelo/FGAT/MPASstatic básico está funcional, e o crash restante entra com observers ativos.
- Se `mpastatic_no_observers_np1` falhar após o forecast, o problema não depende nem de observers nem de SABER/BUMP, e a investigação deve focar em namelist/physics/estado inicial, especialmente `config_len_disp = 480000.0` contra `nominalMinDc = 240000.0` e as mensagens `pressure(1) < pressure(2)`.

Não foi submetido nenhum próximo teste automaticamente.
