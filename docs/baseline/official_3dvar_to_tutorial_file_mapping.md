# Mapeamento dos arquivos oficiais 3DVar para dados do tutorial

Data da analise: 2026-06-04.

Diretorios procurados:

```text
/p/projetos/monan_das/joao.gerd/data/mpasjedi_tutorial2024_testdata
/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/data
/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/build/runtime
/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi-data
/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data
```

Status usados:

```text
found_exact
found_equivalent
missing
needs_generation
incompatible
unknown
```

## Limitacao de inspecao

`ncdump` nao esta disponivel no ambiente, e os modulos Python `netCDF4`, `h5py` e `scipy` tambem nao estao instalados. Assim, a compatibilidade de malha/celulas foi inferida por nome de malha (`x1.2562`, `x1.10242`, `x1.40962`, `x1.62691`), tamanhos de arquivo, caminhos e nomes. Onde seria necessario ler dimensoes internas NetCDF/HDF5, o campo fica `unknown`.

## Resumo tecnico

O teste oficial que passou usa malha global 480 km `x1.2562`. Os dados principais do tutorial encontrados usam:

| Conjunto | Evidencia | Compatibilidade com oficial 480 km |
| --- | --- | --- |
| tutorial 240 km | `x1.10242.invariant.nc`, `x1.10242.graph.info.part.36`, `x1.10242.graph.info.part.64`, background de 53.9 MB | malha diferente; nao e substituicao direta |
| tutorial 120 km | `x1.40962.invariant.nc`, `x1.40962.graph.info.part.36`, `x1.40962.graph.info.part.64`, background de 215 MB | malha diferente; nao e substituicao direta |
| tutorial conus15km | `x1.62691.invariant.nc`, `x1.62691.graph.info.part.128` | malha/regiao diferente; nao e substituicao direta |
| dados oficiais `mpas-jedi-data` | `testinput_tier_1/480km/bg/*` | exato para a rodada oficial |

Conclusao: trocar somente o background oficial pelo background tutorial provavelmente torna a geometria inconsistente, a menos que namelist, streams, invariant e graph sejam trocados de forma coordenada para a mesma malha.

## Tabela de mapeamento

| arquivo oficial | equivalente encontrado | status | exato/equivalente | malha bate | data/ciclo bate | celulas/resolucao compativel | rank/decomp | precisa gerar | observacao |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `testinput/3dvar.yaml` | nenhum YAML tutorial equivalente exato identificado | missing | n/a | n/a | n/a | n/a | nao | nao | deve ser copiado para area isolada e alterado progressivamente |
| `Data/480km/bg/mpasout.2018-04-15_00.00.00.nc` | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi-data/testinput_tier_1/480km/bg/mpasout.2018-04-15_00.00.00.nc` | found_exact | exato oficial | sim | sim | sim por evidencia de caminho 480km/x1.2562 | nao | nao | arquivo oficial da rodada |
| `Data/480km/bg/mpasout.2018-04-15_00.00.00.nc` | `/p/projetos/monan_das/joao.gerd/data/mpasjedi_tutorial2024_testdata/background/2018041418/mpasout.2018-04-15_00.00.00.nc` | found_equivalent | equivalente por nome/data | nao | data bate, diretorio ciclo `2018041418` | provavel 240 km; tamanho 53890680 bytes | depende de `x1.10242` | nao para np36/64 se usar graph tutorial | nao e drop-in para geometria 480 km |
| `Data/480km/bg/mpasout.2018-04-15_00.00.00.nc` | `/p/projetos/monan_das/joao.gerd/data/mpasjedi_tutorial2024_testdata/background_120km/2018041418/mpasout.2018-04-15_00.00.00.nc` | found_equivalent | equivalente por nome/data | nao | data bate, diretorio ciclo `2018041418` | provavel 120 km; tamanho 215477880 bytes | depende de `x1.40962` | nao para np36/64 se usar graph tutorial | ainda menos compativel com 480 km |
| `Data/480km/bg/x1.2562.invariant.nc` | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi-data/testinput_tier_1/480km/bg/x1.2562.invariant.nc` | found_exact | exato oficial | sim | static | sim | nao | nao | invariant oficial |
| `Data/480km/bg/x1.2562.invariant.nc` | `/p/projetos/monan_das/joao.gerd/data/mpasjedi_tutorial2024_testdata/MPAS_namelist_stream_physics_files/x1.10242.invariant.nc` | incompatible | equivalente funcional | nao | static | nao, `x1.10242` | depende de graph `x1.10242.*` | nao | tutorial 240 km |
| `Data/480km/bg/x1.2562.invariant.nc` | `/p/projetos/monan_das/joao.gerd/data/mpasjedi_tutorial2024_testdata/MPAS_namelist_stream_physics_files/x1.40962.invariant.nc` | incompatible | equivalente funcional | nao | static | nao, `x1.40962` | depende de graph `x1.40962.*` | nao | tutorial 120 km |
| `Data/480km/namelist.atmosphere_2018041500` | `/p/projetos/monan_das/joao.gerd/data/mpasjedi_tutorial2024_testdata/MPAS_namelist_stream_physics_files/namelist.atmosphere_240km` | found_equivalent | equivalente | nao | ciclo precisa conferir pelo conteudo | compativel com 240 km, nao 480 km | prefixo esperado deve ser `x1.10242...` | nao | candidato se migrar toda geometria para 240 km |
| `Data/480km/namelist.atmosphere_2018041500` | `/p/projetos/monan_das/joao.gerd/data/mpasjedi_tutorial2024_testdata/MPAS_namelist_stream_physics_files/namelist.atmosphere_120km` | found_equivalent | equivalente | nao | ciclo precisa conferir pelo conteudo | compativel com 120 km, nao 480 km | prefixo esperado deve ser `x1.40962...` | nao | candidato se migrar toda geometria para 120 km |
| `Data/480km/streams.atmosphere` | `/p/projetos/monan_das/joao.gerd/data/mpasjedi_tutorial2024_testdata/MPAS_namelist_stream_physics_files/streams.atmosphere_240km` | found_equivalent | equivalente | nao | datas via templates | compativel com 240 km, nao 480 km | nao | nao | candidato coordenado com namelist/invariant/background 240 km |
| `Data/480km/streams.atmosphere` | `/p/projetos/monan_das/joao.gerd/data/mpasjedi_tutorial2024_testdata/MPAS_namelist_stream_physics_files/streams.atmosphere_120km` | found_equivalent | equivalente | nao | datas via templates | compativel com 120 km, nao 480 km | nao | nao | candidato coordenado com namelist/invariant/background 120 km |
| `x1.2562.graph.info` | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi-data/testinput_tier_1/480km/bg/conus480km.graph.info` | found_equivalent | nao exato por nome; oficial tambem tem `x1.2562.graph.info` em testinput | sim para 480 km se usado com arquivos oficiais | static | provavel compativel | sim | nao | graph base oficial tambem existe em `mpas-jedi/test/testinput/namelists/480km` |
| `x1.2562.graph.info.part.16` | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/480km/x1.2562.graph.info.part.16` | found_exact | exato oficial | sim | static | sim | sim, `np16` | nao | passou conforme contexto |
| `x1.2562.graph.info.part.64` | nenhum `x1.2562.graph.info.part.64` encontrado | needs_generation | n/a | sim se gerado do graph oficial | static | sim se gerado do graph oficial | sim, `np64` | sim, por exemplo via `gpmetis` | causa raiz da falha `np64` reportada |
| `x1.10242.graph.info.part.64` | `/p/projetos/monan_das/joao.gerd/data/mpasjedi_tutorial2024_testdata/MPAS_namelist_stream_physics_files/x1.10242.graph.info.part.64` | found_equivalent | equivalente tutorial | nao | static | 240 km | sim, `np64` | nao | util apenas se toda geometria migrar para `x1.10242` |
| `x1.40962.graph.info.part.64` | `/p/projetos/monan_das/joao.gerd/data/mpasjedi_tutorial2024_testdata/MPAS_namelist_stream_physics_files/x1.40962.graph.info.part.64` | found_equivalent | equivalente tutorial | nao | static | 120 km | sim, `np64` | nao | util apenas se toda geometria migrar para `x1.40962` |
| `sondes_obs_2018041500_m.nc4` | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1/sondes_obs_2018041500_m.nc4` | found_exact | exato oficial | independente da malha | sim | n/a | nao | nao | usado pelo oficial |
| `sondes_obs_2018041500_m.nc4` | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/data/observations/ioda/2018041500/sondes_obs_2018041500.h5` | found_equivalent | equivalente tutorial/workflow | independente da malha, mas HofX depende da geometria | sim | n/a | nao | nao | formato/nome diferente; requer ajuste YAML |
| `aircraft_obs_2018041500_m.nc4` | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1/aircraft_obs_2018041500_m.nc4` | found_exact | exato oficial | independente da malha | sim | n/a | nao | nao | usado pelo oficial |
| `aircraft_obs_2018041500_m.nc4` | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/data/observations/ioda/2018041500/aircraft_obs_2018041500.h5` | found_equivalent | equivalente tutorial/workflow | independente da malha, mas HofX depende da geometria | sim | n/a | nao | nao | requer ajuste YAML |
| `sfc_obs_2018041500_m.nc4` | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1/sfc_obs_2018041500_m.nc4` | found_exact | exato oficial | independente da malha | sim | n/a | nao | nao | usado por dois observadores oficiais |
| `sfc_obs_2018041500_m.nc4` | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/data/observations/ioda/2018041500/sfc_obs_2018041500.h5` | found_equivalent | equivalente tutorial/workflow | independente da malha, mas HofX depende da geometria | sim | n/a | nao | nao | requer ajuste YAML |
| `gnssro_obs_2018041500_s.nc4` | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1/gnssro_obs_2018041500_s.nc4` | found_exact | exato oficial | independente da malha | sim | n/a | nao | nao | nenhum equivalente workflow tutorial especifico encontrado na busca |
| `amsua_n19_obs_2018041500_m.nc4` | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/ufo-data/testinput_tier_1/amsua_n19_obs_2018041500_m.nc4` | found_exact | exato oficial | independente da malha, mas CRTM usa geovals/geometry | sim | n/a | nao | nao | nenhum equivalente workflow tutorial especifico encontrado na busca |
| `obsop_name_map.yaml` | `/p/projetos/monan_das/joao.gerd/data/mpasjedi_tutorial2024_testdata/conus15km/obsop_name_map.yaml` | found_equivalent | equivalente | depende de operadores | n/a | n/a | nao | nao | conteudo nao comparado nesta rodada |
| `obsop_name_map.yaml` | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/data/static/obsop_name_map.yaml` | found_equivalent | equivalente reduzido | depende de operadores | n/a | n/a | nao | nao | tamanho 147 bytes contra 963 bytes oficial; pode ser incompleto para oficial |
| `Data/UFOCoeff/amsua_n19.*` e coeficientes CRTM | `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/test-data-release/crtm/fix_REL-3.1.2.0/fix/.../Little_Endian/*` | found_exact | exato por symlink oficial | n/a | n/a | n/a | nao | nao | nao estava na lista de diretorios tutorial, mas e dependencia oficial |
| `stream_list.atmosphere.*` | oficiais em `/p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/namelists/stream_list.atmosphere.*` | found_exact | exato oficial | sim para 480 km | n/a | sim | nao | nao | devem ser preservados enquanto usar `streams.atmosphere` oficial |
| `geovars.yaml` | `/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/data/static/geovars.yaml` e oficial em testinput/namelists | found_equivalent | equivalente | unknown | n/a | unknown | nao | nao | nao e referencia explicita do YAML oficial 3DVar |
| `keptvars.yaml` | oficial em testinput/namelists; runtime/workflow tambem contem | found_equivalent | equivalente | unknown | n/a | unknown | nao | nao | nao e referencia explicita do YAML oficial 3DVar |
| `templateFields.10242.nc` | tutorial/workflow usa background 240 km como template em logs anteriores | incompatible | equivalente de outro fluxo | nao | pode bater data | `x1.10242`, nao `x1.2562` | nao | nao | nao usado pelo YAML oficial 3DVar |

## Classificacao pratica para migracao

Arquivos seguros para trocar isoladamente primeiro:

| Classe | Motivo |
| --- | --- |
| YAML copiado para area isolada, ainda apontando para arquivos oficiais | nao altera dados nem workflow; valida caminhos |
| obs convencionais tutorial (`aircraft`, `sondes`, `sfc`) | independem de malha como arquivo, mas HofX ainda depende da geometria; devem ser trocados um por vez |

Arquivos que nao devem ser trocados isoladamente:

| Classe | Motivo |
| --- | --- |
| background 240 km/120 km do tutorial | incompativel com invariant/graph/namelist 480 km oficiais |
| invariant `x1.10242` ou `x1.40962` | precisa vir junto com background, namelist, streams e graph da mesma malha |
| graph/decomposicao | depende do numero de ranks e da mesma malha do background/invariant |

Ponto de decisao: para adaptar o oficial aos dados tutorial, primeiro escolher a malha tutorial alvo (`x1.10242` 240 km ou `x1.40962` 120 km). Enquanto essa escolha nao for feita, a unica migracao controlada recomendada e trocar observacoes uma por vez, mantendo geometria/background oficiais.
