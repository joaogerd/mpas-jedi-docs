# Resultado FGAT np1 com model variables oficiais

Data da análise: 2026-06-04

## Resumo

O teste `official_modvars_np1` foi submetido isoladamente e falhou no mesmo ponto lógico do FGAT np1 base: depois do forecast MPAS terminar em `2018-04-15T03:00:00Z`, antes de mensagens de GeoVaLs, HofX, filtros, IODA ou escrita de feedback/análise.

A inclusão das variáveis oficiais adicionais do `3dfgat.yaml` não resolveu o crash. Portanto, a falha não é explicada apenas pela omissão de `cloud_liquid_water`, `cloud_liquid_ice`, `rain_water`, `snow_water`, `graupel` e `w` no bloco `model variables`.

## Job executado

| Item | Valor |
|---|---|
| Job ID | `258267.pbs-ha` |
| PBS | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/run_fgat64_official_modvars_np1.pbs` |
| YAML | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.official_modvars.yaml` |
| Diretório de execução | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test` |
| Binário | `/p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x` |
| Launcher MPI | `/opt/cray/pals/1.6/bin/mpiexec` |
| Ranks | `1` |
| `OMP_NUM_THREADS` | `1` |
| `GFORTRAN_CONVERT_UNIT` | `big_endian:101-200` |
| `F_UFMTENDIAN` | `big:101-200` |
| `FI_CXI_RX_MATCH_MODE` | `hybrid` |
| Log PBS | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/logs/fgat64_official_modvars_np1.pbs.out` |
| `log.atmosphere.0000.out` | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/log.atmosphere.0000.out` |
| `log.atmosphere.0000.err` | Não criado |
| `mpasjedi_variational.log` | Não criado |
| Status final | `139` |

Comando efetivo registrado no PBS:

```bash
/opt/cray/pals/1.6/bin/mpiexec -n 1 \
  /p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x \
  /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test/3dvar_fgat.fgat64.official_modvars.yaml
```

## Configuração testada

O YAML mantém:

- `cost type: 3D-FGAT`
- janela: `2018-04-14T21:00:00Z` por `PT6H`
- `model tstep: PT45M`
- geometria local: `namelist.atmosphere.outer` e `streams.atmosphere.outer`
- background: `background/mpasout.2018-04-14_21.00.00.nc`
- covariância: `SABER` com `BUMP_NICAS`, `StdDev` e `BUMP_VerticalBalance`
- observers: `Aircraft`, `Radiosonde`, `SfcCorrected`
- `obsdataout` em `feedback/*.h5`

A mudança principal deste teste foi usar uma lista expandida de 30 `model variables`, incluindo os campos oficiais que faltavam no YAML base:

```text
cloud_liquid_water, cloud_liquid_ice, rain_water, snow_water, graupel, w
```

O início do forecast confirmou `nFields = 30`, incluindo esses campos:

```text
Model:forecast: forecast starting:
  Valid time: 2018-04-14T21:00:00Z
  Resolution: nCellsGlobal = 10242, nFields = 30
...
Fld=17 ... cloud_liquid_water
Fld=18 ... cloud_liquid_ice
Fld=19 ... rain_water
Fld=20 ... snow_water
Fld=21 ... graupel
...
Fld=30 ... w
```

## Ponto de falha

Trecho decisivo do log PBS:

```text
Model:forecast: forecast finished:
  Valid time: 2018-04-15T03:00:00Z
  Resolution: nCellsGlobal = 10242, nFields = 30
...
Fld=30  Min=-0.136063, Max=0.240016, RMS=0.0113863 : w
cn-0083.head.cm.cptec.inpe.br: rank 0 died from signal 11 and dumped core
[INFO] command exit status: 139
```

Não aparecem mensagens posteriores indicando entrada em GeoVaLs, HofX, filtros, `obsdataout`, escrita IODA ou produção de análise. O diretório `analysis/` não recebeu arquivos e o diretório `feedback/` também não recebeu arquivos.

Também apareceram mensagens MPAS durante o forecast:

```text
--- subroutine MPAS_to_phys - pressure(1) < pressure(2):
```

Essas mensagens ocorreram antes do segfault e indicam inconsistência física/vertical detectada no caminho de física MPAS. Elas ainda precisam ser correlacionadas com a configuração de malha/namelist/estado inicial.

## Arquivos gerados

Nenhum arquivo novo de produto foi criado em:

- `fgat64_runtime_test/analysis/`
- `fgat64_runtime_test/feedback/`

Arquivos de log relevantes criados/atualizados:

- `fgat64_runtime_test/log.atmosphere.0000.out`
- `fgat64_runtime_test/log.atmosphere.0000.d0001.out`
- `logs/fgat64_official_modvars_np1.pbs.out`

## Interpretação

Como `official_modvars_np1` falhou no mesmo ponto do FGAT np1 base, a causa provável não é apenas a lista incompleta de `model variables`.

A falha continua ocorrendo no caminho pós-forecast imediato, antes de evidência textual de GeoVaLs/HofX/IODA. Isso mantém como hipóteses mais fortes:

1. inconsistência do estado inicial/namelist/physics para a malha `x1.10242`;
2. efeito do bloco `model` do FGAT com forecast MPAS de 6 horas sobre esse background;
3. interação ainda anterior aos observers, a ser testada removendo observers;
4. possível incompatibilidade da configuração SABER/BUMP/VBAL com este caso, mas isso deve ser testado depois de saber se o crash ocorre sem observers.

Uma diferença material ainda presente é que os namelists isolados usam malha `x1.10242`, mas anteriormente foi observado `config_len_disp = 480000.0`, enquanto o arquivo de malha reporta `nominalMinDc = 240000.0`. Isso não foi corrigido neste teste e segue como suspeita de configuração física/geométrica.

## Próximo teste recomendado

O próximo teste recomendado é `no_observers_np1`, não `no_obsdataout_np1` ainda.

Justificativa: como o crash acontece antes de qualquer escrita de `obsdataout` e antes de mensagens GeoVaLs/HofX/filtros, remover só `obsdataout` provavelmente testa tarde demais no fluxo. Remover todos os observers separa primeiro se o problema já ocorre com `3D-FGAT + model forecast + background error` sem nenhuma observação.

Comando manual recomendado, quando for decidido rodar o próximo caso:

```bash
cd /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis
qsub run_fgat64_no_observers_np1.pbs
```

Interpretação esperada:

- Se `no_observers_np1` falhar no mesmo ponto, o problema é estrutural no modelo/estado/namelist/FGAT/SABER, não nos observers/IODA.
- Se `no_observers_np1` passar, o problema entra com a presença dos observers; aí o próximo isolamento deve ser `no_obsdataout_np1` ou observers individuais em np1.
