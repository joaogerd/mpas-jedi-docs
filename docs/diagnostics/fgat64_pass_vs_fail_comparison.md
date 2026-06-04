# Comparacao: casos que passaram vs FGAT isolado que falhou

## Escopo

Esta analise compara estaticamente:

1. CTest/direct 3DVar oficial que passou, com `testinput/3dvar.yaml`.
2. YAMLs oficiais FGAT de referencia: `3dfgat.yaml` e `3dfgat_cda.yaml`.
3. YAML isolado FGAT que falhou: `3dvar_fgat.fgat64.yaml`.
4. YAML renderizado operacional: `build/rendered/3dvar_fgat.yaml`.

Nao foram submetidos jobs novos e `mpasjedi_variational.x` nao foi executado nesta etapa.

## Execucao e ambiente

| item | 3DVar oficial passou | FGAT isolado falhou |
|---|---|---|
| binario | `/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/bin/mpasjedi_variational.x` | `/p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x` |
| comando confirmado | `/opt/cray/pals/1.6/bin/mpiexec -n 1 ... testinput/3dvar.yaml`; tambem passou com `-n 16` | `/opt/cray/pals/1.6/bin/mpiexec -n 1 ... 3dvar_fgat.fgat64.yaml`; tambem falhou com `-n 64` |
| working directory | `/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test` | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/mpas-jedi/test/isolated_variational/analysis/fgat64_runtime_test` |
| ranks | passou com 1 e 16; 64 falhou apenas por falta de `x1.2562.graph.info.part.64` | falhou com 1 e 64, depois do forecast |
| OMP | `OMP_NUM_THREADS=1` nos casos confirmados equivalentes ao CTest | `OMP_NUM_THREADS=1` |
| endian | `GFORTRAN_CONVERT_UNIT=big_endian:101-200` | `GFORTRAN_CONVERT_UNIT=big_endian:101-200`; tambem `F_UFMTENDIAN=big:101-200` |
| rede/MPI extra | sem `FI_CXI_RX_MATCH_MODE` nos PBS oficiais diretos | `FI_CXI_RX_MATCH_MODE=hybrid` |
| status | `command exit status: 0` | `rank 0 died from signal 11`, `command exit status: 139` em np1; `signal 11/15`, status 143 em np64 |

Diferenca material: ha diferenca de binario (`work/.../build` vs `builds/...`) e de ambiente carregado pelo workflow. Como o mesmo ponto de falha aparece em `np1`, isso nao parece ser apenas problema de MPI/64 ranks, mas ainda vale controlar binario/ambiente numa etapa posterior.

## Comparacao dos YAMLs

| item | 3DVar oficial | 3DFGAT oficial | 3DFGAT CDA oficial | FGAT isolado falho |
|---|---|---|---|---|
| cost type | `3D-Var` | `3D-FGAT` | `3D-FGAT` | `3D-FGAT` |
| bloco `model` | ausente | presente | presente | presente |
| tstep | nao aplicavel | `PT45M` | `PT45M` | `PT45M` |
| janela | `2018-04-14T21:00:00Z`, `PT6H`; background em `2018-04-15T00:00:00Z` | `2018-04-14T21:00:00Z`, `PT6H` | `2018-04-14T21:00:00Z`, `PT3H`; segundo OL/CDA com janela adicional | `2018-04-14T21:00:00Z`, `PT6H` |
| background | `Data/480km/bg/mpasout.2018-04-15_00.00.00.nc`, date `2018-04-15T00:00:00Z` | `Data/480km/bg/mpasout.2018-04-14_21.00.00.nc`, date `2018-04-14T21:00:00Z` | mesmo inicio `2018-04-14T21:00:00Z` | `background/mpasout.2018-04-14_21.00.00.nc`, date `2018-04-14T21:00:00Z` |
| geometry | `Data/480km/namelist.atmosphere_2018041500`, `streams.atmosphere` | `Data/480km/namelist.atmosphere_2018041421`, `streams.atmosphere` | mesmo | `namelist.atmosphere.outer/inner`, `streams.atmosphere.outer/inner` |
| graph prefix | `x1.2562.graph.info.part.` | `x1.2562.graph.info.part.` | `x1.2562.graph.info.part.` | `x1.10242.graph.info.part.` |
| malha | x1.2562, 480 km | x1.2562, 480 km | x1.2562, 480 km | x1.10242, nominalmente 240 km |
| `config_len_disp` | `480000.0`, consistente com 480 km | `480000.0`, consistente com 480 km | `480000.0`, consistente com 480 km | `480000.0`, mas o arquivo x1.10242 reporta `nominalMinDc = 240000.0` |
| background error | `MPASstatic` | `MPASstatic` | `MPASstatic` | `SABER` com `BUMP_NICAS`, `StdDev`, `BUMP_VerticalBalance`, `Control2Analysis` |
| minimizer | DRPCG | DRPCG | DRPCG | DRPCG |
| output | `Data/states` ou `Data/os` conforme obs | `Data/states/mpas.3dfgat...` | `Data/states/mpas.3dfgat_cda...` | `analysis/analysis...` e `feedback/*.h5`; nada foi escrito antes do crash |

## Variaveis do modelo e do estado

Diferenca critica: o FGAT oficial inclui variaveis de microfisica e velocidade vertical no `model variables`; o FGAT isolado falho nao inclui esses campos.

FGAT oficial `model variables` inclui, alem das variaveis dinamicas principais:

```text
cloud_liquid_water, cloud_liquid_ice, rain_water, snow_water, graupel, w
```

O FGAT isolado falho omite esses nomes e usa:

```text
air_temperature, water_vapor_mixing_ratio_wrt_moist_air, eastward_wind,
northward_wind, air_pressure_at_surface, air_potential_temperature,
dry_air_density, u, water_vapor_mixing_ratio_wrt_dry_air, air_pressure,
landmask, seaice_fraction, snowc, skin_temperature_at_surface, ivgtyp,
isltyp, snowh, vegetation_area_fraction, eastward_wind_at_10m,
northward_wind_at_10m, lai, smois, tslb, pressure_p
```

O arquivo background isolado contem equivalentes MPAS nativos para varios campos omitidos (`qc`, `qi`, `qr`, `qs`, `qg`, `w`), mas o YAML nao os inclui no estado propagado. Isso e material porque o modelo MPAS com fisica `mesoscale_reference`/WSM6 usa microfisica durante o forecast. O log do `np1` tambem mostrou dumps de `MPAS_to_phys - pressure(1) < pressure(2)`, indicando instabilidade/inconsistencia numerica durante a fisica.

O 3DVar oficial tambem tem uma lista de `analysis variables` mais ampla que inclui hidrometeoros:

```text
cloud_liquid_water, cloud_liquid_ice, rain_water, snow_water, graupel
```

O FGAT oficial usa analysis variables de cinco campos, mas preserva hidrometeoros e `w` no `model variables`/`state variables`.

## SABER/BUMP/NICAS/VBAL

Os YAMLs oficiais analisados usam:

```text
background error:
  covariance model: MPASstatic
```

O FGAT isolado usa uma configuracao substancialmente diferente:

```text
background error:
  covariance model: SABER
  saber central block: BUMP_NICAS
  saber outer blocks: StdDev, BUMP_VerticalBalance
  linear variable change: Control2Analysis
```

Essa diferenca ainda nao explica sozinha o segfault imediatamente pos-forecast, mas muda profundamente a proxima etapa da variacional. A configuracao SABER tambem usa variaveis de controle:

```text
stream_function, velocity_potential, temperature, spechum, surface_pressure
```

com saida para:

```text
air_temperature, water_vapor_mixing_ratio_wrt_moist_air,
eastward_wind, northward_wind, air_pressure_at_surface
```

## Observers, operadores e filtros

| item | oficiais 3DVar/3DFGAT | FGAT isolado falho |
|---|---|---|
| Radiosonde | `VertInterp`, `airTemperature`, `windEastward`, `windNorthward`, `specificHumidity` | mesmo conjunto basico |
| Aircraft | presente no 3DVar oficial; ausente no 3DFGAT oficial | presente |
| GNSSRO | presente no 3DVar e 3DFGAT oficiais | ausente |
| Surface | oficial 3DVar usa blocos `Surface T,Q,Ps` e `Surface U,V`; FGAT oficial usa `SfcCorrected` para `stationPressure` | apenas `SfcCorrected` para `stationPressure` |
| filtros | PreQC, Background Check; surface inclui Difference Check; 3DVar tem filtros extras como Domain Check, ROobserror, Perform Action, GOMsaver | PreQC, Background Check; SfcCorrected tem Difference Check |
| obsdataout | presente nos oficiais e no isolado | presente, mas nenhum feedback foi escrito antes do crash |

Campos esperados por operadores/filtros:

- `VertInterp`: GeoVaLs verticais para temperatura, umidade, vento e coordenadas verticais/pressao/altura.
- `SfcCorrected`: `stationPressure` e campos de superficie/orografia/pressao de superficie.
- `Difference Check` de superficie: `MetaData/stationElevation` e `GeoVaLs/height_above_mean_sea_level_at_surface`.
- `Background Check`: HofX/departures para as variaveis simuladas.

O log do caso falho nao chegou a registrar `GeoVaLs`, `HofX`, `obsdataout`, `writing` ou `IODA` depois do forecast. A falha ocorre antes de uma mensagem OOPS de alto nivel da etapa seguinte.

## Estrutura IODA

Os IODA oficiais sao pequenos e contem grupos historicos GSI adicionais:

| arquivo oficial | nlocs aproximado | grupos relevantes |
|---|---:|---|
| `aircraft_obs_2018041500_m.nc4` | 2103 | `ObsValue`, `ObsError`, `PreQC`, `MetaData`, `GsiHofX`, `GsiUseFlag`, `GsiQCWeight`, etc. |
| `sondes_obs_2018041500_m.nc4` | 974 | mesmos grupos GSI adicionais |
| `sfc_obs_2018041500_m.nc4` | 282 | mesmos grupos GSI adicionais |
| `gnssro_obs_2018041500_s.nc4` | 20 | `ObsValue`, `ObsError`, `MetaData` para refratividade/bending angle |

Os IODA isolados sao muito maiores e nao contem os grupos GSI adicionais:

| arquivo isolado | nlocs | grupos relevantes |
|---|---:|---|
| `aircraft_obs_2018041500.h5` | 348901 | `ObsValue`, `ObsError`, `ObsType`, `PreQC`, `MetaData`, `nvars`, `nstring` |
| `sondes_obs_2018041500.h5` | 73235 | idem |
| `sfc_obs_2018041500.h5` | 150334 | idem, com `stationPressure` |

Essa diferenca e material: o caso isolado processa ordens de magnitude mais observacoes. Mesmo em `np1`, os ObsSpaces foram lidos, mas o crash ocorreu depois do forecast, antes de escrita de feedback.

## Passed vs failed: diferencas que podem explicar o crash pos-forecast

| diferenca | passado/oficial | falho | risco |
|---|---|---|---|
| `model variables` FGAT | inclui hidrometeoros e `w` | omite hidrometeoros e `w` | alto; pode corromper/invalidar estado propagado com fisica ativa |
| malha vs `config_len_disp` | x1.2562 e `480000.0` coerentes | x1.10242, mas `config_len_disp=480000.0`; log indica nominalMinDc 240000 | alto; inconsistencia de escala/physics/dinamica |
| covariance | `MPASstatic` | `SABER/BUMP/NICAS/VBAL` | medio/alto; muda etapa variacional depois da trajetoria |
| IODA | pequeno, oficial, grupos GSI | grande, renderizado, sem GSI groups | medio/alto; aumenta memoria e muda caminho de ObsSpace/HofX |
| observers | FGAT oficial: Radiosonde, GNSSRO, SfcCorrected | Aircraft, Radiosonde, SfcCorrected | medio; Aircraft nao esta no FGAT oficial de referencia |
| binario | build CTest `/work/...` | bundle operacional `/builds/...` | medio; precisa controle se a reducao YAML nao resolver |
| ambiente | CTest/direct minimo | workflow env completo, FI_CXI, F_UFMTENDIAN | baixo/medio; np1 tambem falha, mas ambiente ainda difere |
| logs MPAS | 3DVar passa | `MPAS_to_phys - pressure(1) < pressure(2)` durante forecast | alto; indica problema numerico antes do segfault |

Hipotese principal nesta comparacao: o caso isolado FGAT esta propagando MPAS com uma configuracao de estado incompleta/inconsistente para a malha x1.10242. Os candidatos mais fortes sao a omissao de hidrometeoros/`w` em `model variables` e o `config_len_disp` de 480 km aplicado a uma malha cujo arquivo reporta 240 km. A falha aparece antes de mensagens de `GeoVaLs/HofX`, entao a causa pode estar no estado/modelo propagado, nao necessariamente nos filtros IODA.

## Matriz minima np1 preparada, sem submissao

Foram criados YAMLs e PBSs apenas na area isolada. Nenhum job foi submetido.

| caso | YAML | PBS | o que testa |
|---|---|---|---|
| A. sem observers | `fgat64_runtime_test/3dvar_fgat.fgat64.no_observers.yaml` | `run_fgat64_no_observers_np1.pbs` | se OOPS aceita FGAT sem observers; separa modelo/forecast/SABER de ObsSpaces |
| B. aircraft only | `fgat64_runtime_test/3dvar_fgat.fgat64.aircraft_only.yaml` | `run_fgat64_aircraft_only_np1.pbs` | observer Aircraft isolado em serial |
| C. sondes only | `fgat64_runtime_test/3dvar_fgat.fgat64.sondes_only.yaml` | `run_fgat64_sondes_only_np1.pbs` | observer Radiosonde isolado em serial |
| D. sfc only | `fgat64_runtime_test/3dvar_fgat.fgat64.sfc_only.yaml` | `run_fgat64_sfc_only_np1.pbs` | observer SfcCorrected/Difference Check isolado em serial |
| E. sem filters | `fgat64_runtime_test/3dvar_fgat.fgat64.no_filters.yaml` | `run_fgat64_no_filters_np1.pbs` | separa filtros (`PreQC`, `Background Check`, `Difference Check`) do restante |
| F. sem obsdataout | `fgat64_runtime_test/3dvar_fgat.fgat64.no_obsdataout.yaml` | `run_fgat64_no_obsdataout_np1.pbs` | separa escrita IODA/feedback do restante |
| G. modvars oficiais | `fgat64_runtime_test/3dvar_fgat.fgat64.official_modvars.yaml` | `run_fgat64_official_modvars_np1.pbs` | aproxima o `model variables/state variables` do `3dfgat.yaml` oficial, adicionando hidrometeoros e `w` |

Todos os PBSs usam:

```text
/opt/cray/pals/1.6/bin/mpiexec -n 1
/p/projetos/monan_das/joao.gerd/builds/monan-jedi-mpas/bin/mpasjedi_variational.x
```

Validados estaticamente:

```text
bash -n: passou em todos os PBSs novos
qsub interno: ausente
```

## Ordem recomendada para execucao futura

Nao executar todos de uma vez. A ordem mais informativa e:

1. `run_fgat64_official_modvars_np1.pbs`
2. `run_fgat64_no_observers_np1.pbs` se o caso sem observers for aceito pelo executavel
3. `run_fgat64_no_filters_np1.pbs`
4. `run_fgat64_no_obsdataout_np1.pbs`
5. observers isolados `aircraft`, `sondes`, `sfc`

Se `official_modvars_np1` passar do ponto do segfault, o primeiro conserto real deve ser alinhar `model variables/state variables` do workflow com o FGAT oficial. Se ainda falhar, ajustar/diagnosticar `config_len_disp` para x1.10242 antes de culpar observers. Se os casos sem filtros/sem obsdataout mudarem o ponto de falha, entao voltar aos filtros/escrita IODA.
