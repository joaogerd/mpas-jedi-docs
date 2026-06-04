# Arquivos requeridos pela rodada oficial MPAS-JEDI 3DVar

Data da analise: 2026-06-04.

YAML oficial analisado:

```text
/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/3dvar.yaml
```

Diretorio efetivo de execucao:

```text
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test
```

## Observacoes

Os caminhos relativos abaixo sao relativos ao diretorio efetivo de execucao. A coluna "obrigatorio" indica necessidade para a configuracao oficial completa do `3dvar.yaml`, nao para variantes reduzidas.

`background error: covariance model: MPASstatic` nao referencia um arquivo externo explicito no YAML. A evidencia disponivel indica que a covariancia estatica e construida pelo componente MPASstatic a partir da geometria/estado, sem arquivo BUMP/SABER explicito neste YAML.

`geovars.yaml`, `keptvars.yaml` e `templateFields` nao sao referenciados explicitamente pelo `3dvar.yaml`. `geovars.yaml` e `keptvars.yaml` existem como symlinks no diretorio de execucao por infraestrutura comum de testes MPAS-JEDI; nao foram identificados como entradas diretas desta rodada 3DVar pelo YAML. Nenhum `templateFields` e usado pelo YAML oficial 3DVar.

## Tabela de arquivos

| arquivo | tipo | caminho oficial | caminho relativo | quem usa | funcao | obrigatorio | observacao |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `3dvar.yaml` | YAML principal | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/3dvar.yaml` | `testinput/3dvar.yaml` | `mpasjedi_variational.x` | configuracao 3D-Var, observadores, output, iteracoes | sim | no build e symlink para a fonte oficial |
| `mpasout.2018-04-15_00.00.00.nc` | background MPAS | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi-data/testinput_tier_1/480km/bg/mpasout.2018-04-15_00.00.00.nc` | `Data/480km/bg/mpasout.2018-04-15_00.00.00.nc` | `cost function.background.filename` e stream `input` | estado inicial/background na data `2018-04-15T00:00:00Z` | sim | tamanho observado: 2180664 bytes |
| `mpas.3dvar.2018-04-15_00.00.00.nc` | analysis output | gerado em `.../build/mpas-jedi/test/Data/states/mpas.3dvar.2018-04-15_00.00.00.nc` | `Data/states/mpas.3dvar.$Y-$M-$D_$h.$m.$s.nc` | `output.filename` | estado de analise | saida obrigatoria | gerado pela rodada |
| `sondes_obs_2018041500_m.nc4` | obs input IODA | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1/sondes_obs_2018041500_m.nc4` | `Data/ufo/testinput_tier_1/sondes_obs_2018041500_m.nc4` | observador `Radiosonde` | radiossondas: T, vento, umidade | sim | `Data/ufo/testinput_tier_1` e symlink para `ufo-data` |
| `aircraft_obs_2018041500_m.nc4` | obs input IODA | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1/aircraft_obs_2018041500_m.nc4` | `Data/ufo/testinput_tier_1/aircraft_obs_2018041500_m.nc4` | observador `Aircraft` | aircraft: T, vento, umidade | sim | arquivo oficial exato existe em `ufo-data` |
| `gnssro_obs_2018041500_s.nc4` | obs input IODA | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1/gnssro_obs_2018041500_s.nc4` | `Data/ufo/testinput_tier_1/gnssro_obs_2018041500_s.nc4` | observador `GnssroRefNCEP` | refratividade atmosferica | sim | arquivo oficial exato existe em `ufo-data` |
| `sfc_obs_2018041500_m.nc4` | obs input IODA | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1/sfc_obs_2018041500_m.nc4` | `Data/ufo/testinput_tier_1/sfc_obs_2018041500_m.nc4` | observadores `Surface T,Q,Ps` e `Surface U,V` | obs de superficie para T/Q/Ps e vento | sim | mesmo arquivo usado por dois observadores |
| `amsua_n19_obs_2018041500_m.nc4` | obs input IODA | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1/amsua_n19_obs_2018041500_m.nc4` | `Data/ufo/testinput_tier_1/amsua_n19_obs_2018041500_m.nc4` | observadores `AMSUA-NOAA19--nohydro` e `AMSUA-NOAA19--hydro` | radiancias AMSUA NOAA-19 | sim | mesmo arquivo usado por dois blocos de canais |
| `obsout_3dvar_sondes.nc4` | obs output/feedback | gerado em `.../build/mpas-jedi/test/Data/os/obsout_3dvar_sondes.nc4` | `Data/os/obsout_3dvar_sondes.nc4` | observador `Radiosonde` | feedback/HofX/QC | saida | gerado pela rodada |
| `obsout_3dvar_aircraft.nc4` | obs output/feedback | gerado em `.../build/mpas-jedi/test/Data/os/obsout_3dvar_aircraft.nc4` | `Data/os/obsout_3dvar_aircraft.nc4` | observador `Aircraft` | feedback/HofX/QC | saida | gerado pela rodada |
| `obsout_3dvar_gnssroref.nc4` | obs output/feedback | gerado em `.../build/mpas-jedi/test/Data/os/obsout_3dvar_gnssroref.nc4` | `Data/os/obsout_3dvar_gnssroref.nc4` | observador `GnssroRefNCEP` | feedback/HofX/QC | saida | gerado pela rodada |
| `obsout_3dvar_sfc.nc4` | obs output/feedback | gerado em `.../build/mpas-jedi/test/Data/os/obsout_3dvar_sfc.nc4` | `Data/os/obsout_3dvar_sfc.nc4` | observador `Surface T,Q,Ps` | feedback/HofX/QC | saida | gerado pela rodada |
| `obsout_3dvar_sfc_wind.nc4` | obs output/feedback | gerado em `.../build/mpas-jedi/test/Data/os/obsout_3dvar_sfc_wind.nc4` | `Data/os/obsout_3dvar_sfc_wind.nc4` | observador `Surface U,V` | feedback/HofX/QC | saida | gerado pela rodada |
| `obsout_3dvar_amsua_n19--nohydro.nc4` | obs output/feedback | gerado em `.../build/mpas-jedi/test/Data/os/obsout_3dvar_amsua_n19--nohydro.nc4` | `Data/os/obsout_3dvar_amsua_n19--nohydro.nc4` | observador AMSUA sem hidrometeoros | feedback/HofX/QC | saida | gerado pela rodada |
| `obsout_3dvar_amsua_n19--hydro.nc4` | obs output/feedback | gerado em `.../build/mpas-jedi/test/Data/os/obsout_3dvar_amsua_n19--hydro.nc4` | `Data/os/obsout_3dvar_amsua_n19--hydro.nc4` | observador AMSUA com hidrometeoros | feedback/HofX/QC | saida | gerado pela rodada |
| `geoval_out.nc` / `geoval_out_*.nc` | obs output auxiliar | gerado em `.../build/mpas-jedi/test/Data/os` | `Data/os/geoval_out.nc` | filtro `GOMsaver` do observador `Surface U,V` | salva GeoVaLs | saida | em execucao paralela pode aparecer particionado |
| `namelist.atmosphere_2018041500` | MPAS namelist | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/480km/namelist.atmosphere_2018041500` | `Data/480km/namelist.atmosphere_2018041500` | `geometry.nml_file` e iteracoes | controla MPAS, decomposicao, fisica e DA | sim | define `config_block_decomp_file_prefix = 'x1.2562.graph.info.part.'` |
| `streams.atmosphere` | MPAS streams | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/480km/streams.atmosphere` | `Data/480km/streams.atmosphere` | `geometry.streams_file` e iteracoes | define streams `input`, `invariant`, `background`, `analysis`, etc. | sim | referencia stream lists por nome relativo ao diretorio de execucao |
| `x1.2562.graph.info` | graph mesh | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/480km/x1.2562.graph.info` | `x1.2562.graph.info` | MPAS/decomposicao | grafo base da malha 480 km | sim para gerar decomposicoes | disponivel como symlink no diretorio de execucao |
| `x1.2562.graph.info.part.2` | decomposicao MPI | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/480km/x1.2562.graph.info.part.2` | `x1.2562.graph.info.part.2` | MPAS quando `np=2` | particionamento da malha | sim para `np2` | CTest tem `mpasjedi_3dvar_2pe` |
| `x1.2562.graph.info.part.4` | decomposicao MPI | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/480km/x1.2562.graph.info.part.4` | `x1.2562.graph.info.part.4` | MPAS quando `np=4` | particionamento da malha | sim para `np4` | disponivel |
| `x1.2562.graph.info.part.8` | decomposicao MPI | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/480km/x1.2562.graph.info.part.8` | `x1.2562.graph.info.part.8` | MPAS quando `np=8` | particionamento da malha | sim para `np8` | disponivel |
| `x1.2562.graph.info.part.16` | decomposicao MPI | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/480km/x1.2562.graph.info.part.16` | `x1.2562.graph.info.part.16` | MPAS quando `np=16` | particionamento da malha | sim para `np16` | disponivel |
| `x1.2562.graph.info.part.64` | decomposicao MPI | nao encontrado | `x1.2562.graph.info.part.64` | MPAS quando `np=64` | particionamento da malha | sim para `np64` | ausente; precisa ser gerado se `np64` for exigido |
| `x1.2562.invariant.nc` | static mesh/invariant | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi-data/testinput_tier_1/480km/bg/x1.2562.invariant.nc` | `Data/480km/bg/x1.2562.invariant.nc` | stream `invariant` | constantes da malha global 480 km | sim | tamanho observado: 6758628 bytes |
| `stream_list.atmosphere.background` | stream list | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/stream_list.atmosphere.background` | `stream_list.atmosphere.background` | stream `background` | variaveis do stream background | sim | referenciado por `streams.atmosphere` |
| `stream_list.atmosphere.analysis` | stream list | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/stream_list.atmosphere.analysis` | `stream_list.atmosphere.analysis` | stream `analysis` | variaveis escritas na analise | sim | referenciado por `streams.atmosphere` |
| `stream_list.atmosphere.ensemble` | stream list | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/stream_list.atmosphere.ensemble` | `stream_list.atmosphere.ensemble` | stream `ensemble` | variaveis de ensemble | condicional | stream existe, mas nao e entrada principal do 3DVar |
| `stream_list.atmosphere.control` | stream list | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/stream_list.atmosphere.control` | `stream_list.atmosphere.control` | stream `control` | variaveis de controle | condicional | stream existe no arquivo streams |
| `stream_list.atmosphere.dastate` | stream list | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/stream_list.atmosphere.dastate` | `stream_list.atmosphere.dastate` | stream `dastate` | variaveis de DA state | condicional | stream existe no arquivo streams |
| `obsop_name_map.yaml` | alias obs/model | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/obsop_name_map.yaml` | `testinput/obsop_name_map.yaml` | operadores `VertInterp` de radiossonda/aircraft | mapeia nomes UFO para variaveis modelo | sim para esses operadores | caminho e relativo a execucao, nao ao YAML |
| `Data/UFOCoeff/amsua_n19.SpcCoeff.bin` | CRTM coeficiente | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/test-data-release/crtm/fix_REL-3.1.2.0/fix/SpcCoeff/Little_Endian/amsua_n19.SpcCoeff.bin` | `Data/UFOCoeff/amsua_n19.SpcCoeff.bin` | operador CRTM AMSUA | coeficiente espectral | sim para AMSUA | symlink criado no diretorio de teste |
| `Data/UFOCoeff/amsua_n19.TauCoeff.bin` | CRTM coeficiente | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/test-data-release/crtm/fix_REL-3.1.2.0/fix/TauCoeff/ODPS/Little_Endian/amsua_n19.TauCoeff.bin` | `Data/UFOCoeff/amsua_n19.TauCoeff.bin` | operador CRTM AMSUA | coeficiente transmittance | sim para AMSUA | symlink criado no diretorio de teste |
| `Data/UFOCoeff/CloudCoeff.bin` | CRTM coeficiente | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/test-data-release/crtm/fix_REL-3.1.2.0/fix/CloudCoeff/Little_Endian/CloudCoeff.bin` | `Data/UFOCoeff/CloudCoeff.bin` | operador CRTM hidro | coeficiente de nuvens | sim para bloco hidro | symlink criado no diretorio de teste |
| `Data/UFOCoeff/AerosolCoeff.bin` | CRTM coeficiente | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/test-data-release/crtm/fix_REL-3.1.2.0/fix/AerosolCoeff/Little_Endian/AerosolCoeff.bin` | `Data/UFOCoeff/AerosolCoeff.bin` | CRTM | coeficiente aerosol | provavelmente requerido pelo CRTM | presente por infraestrutura |
| `Data/UFOCoeff/FASTEM6.MWwater.EmisCoeff.bin` | CRTM coeficiente | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/test-data-release/crtm/fix_REL-3.1.2.0/fix/EmisCoeff/MW_Water/Little_Endian/FASTEM6.MWwater.EmisCoeff.bin` | `Data/UFOCoeff/FASTEM6.MWwater.EmisCoeff.bin` | CRTM MW | emissividade agua | sim para radiancias MW | presente por infraestrutura |
| `CAM_ABS_DATA.DBL`, `CAM_AEROPT_DATA.DBL`, `GENPARM.TBL`, `LANDUSE.TBL`, `OZONE_DAT.TBL`, `OZONE_LAT.TBL`, `OZONE_PLEV.TBL`, `RRTMG_LW_DATA`, `RRTMG_LW_DATA.DBL`, `RRTMG_SW_DATA`, `RRTMG_SW_DATA.DBL`, `SOILPARM.TBL`, `VEGPARM.TBL`, `COMPATIBILITY` | tabelas/fisica MPAS | symlinks para `/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/MPAS/core_atmosphere/*` | nomes no diretorio de execucao | MPAS physics/core | tabelas de fisica e constantes MPAS | sim para geometria/modelo MPAS completo | nao aparecem no YAML, mas estao no diretorio de execucao oficial |

## Arquivos sob `Data/` explicitamente referenciados

```text
Data/480km/namelist.atmosphere_2018041500
Data/480km/streams.atmosphere
Data/480km/bg/mpasout.2018-04-15_00.00.00.nc
Data/480km/bg/x1.2562.invariant.nc
Data/ufo/testinput_tier_1/sondes_obs_2018041500_m.nc4
Data/ufo/testinput_tier_1/aircraft_obs_2018041500_m.nc4
Data/ufo/testinput_tier_1/gnssro_obs_2018041500_s.nc4
Data/ufo/testinput_tier_1/sfc_obs_2018041500_m.nc4
Data/ufo/testinput_tier_1/amsua_n19_obs_2018041500_m.nc4
Data/UFOCoeff/
Data/os/*.nc4
Data/os/geoval_out.nc
Data/states/mpas.3dvar.$Y-$M-$D_$h.$m.$s.nc
```

## Arquivos referenciados por namelist/streams

Da namelist:

```text
config_block_decomp_file_prefix = 'x1.2562.graph.info.part.'
```

Dos streams:

```text
./Data/480km/bg/mpasout.$Y-$M-$D_$h.$m.$s.nc
./Data/480km/bg/x1.2562.invariant.nc
stream_list.atmosphere.background
stream_list.atmosphere.analysis
stream_list.atmosphere.ensemble
stream_list.atmosphere.control
stream_list.atmosphere.dastate
```
