# Plano para reconfigurar o 3DVar oficial que passou para dados do tutorial

Data da analise: 2026-06-04.

## Principios

Nao alterar workflow operacional. Nao alterar YAMLs principais. Nao submeter jobs. Nao rodar `mpasjedi_variational.x` nesta etapa de documentacao.

A estrategia e partir da rodada oficial que passou e alterar uma coisa por vez em uma area isolada. O aprendizado so deve ser migrado para o workflow depois que a combinacao de arquivos estiver comprovada.

## Step 0: confirmar baseline oficial

Objetivo: confirmar que a base confiavel continua passando antes de qualquer mudanca.

Arquivos alterados: nenhum.

Arquivos preservados: todos os oficiais em:

```text
/p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build/mpas-jedi/test
```

Comando futuro sugerido, nao executado nesta analise:

```sh
cd /p/projetos/monan_das/joao.gerd/work/monan-jedi-mpas/build
ctest --output-on-failure -R '^mpasjedi_3dvar$'
```

Criterio de sucesso: `LastTest.log` termina com `Run ... status = 0` e `Test Passed.`.

Criterio de falha: qualquer erro antes do `Test Passed.`.

Conclusao a tirar: se falhar, parar; a baseline deixou de ser confiavel e a migracao nao deve comecar.

## Step 1: copiar YAML oficial para area isolada e manter todos os arquivos oficiais

Objetivo: provar que uma copia isolada do YAML ainda consegue consumir exatamente os arquivos oficiais.

Arquivos alterados: apenas uma copia do YAML, por exemplo:

```text
scratch/3dvar_official_to_tutorial/step01/3dvar.yaml
```

Arquivos preservados: background, namelist, streams, graph, invariant, obs, CRTM coefficients e stream lists oficiais.

Criterio de sucesso: resultado numericamente compativel com o oficial e logs equivalentes.

Criterio de falha: erro de caminho relativo.

Conclusao a tirar: se falhar, o problema e staging/caminho relativo, nao dado cientifico.

Observacao: como o YAML oficial usa varios caminhos relativos (`Data/...`, `testinput/...`), a area isolada precisa reproduzir a estrutura relativa ou apontar explicitamente para a arvore oficial.

## Step 2: trocar somente uma observacao convencional

Objetivo: testar compatibilidade de obs tutorial sem mexer em malha/background.

Primeiro candidato: `Aircraft`, por ser um bloco unico e ja ter equivalente workflow:

```text
/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/data/observations/ioda/2018041500/aircraft_obs_2018041500.h5
```

Arquivos alterados: somente `obsfile` do bloco `Aircraft`.

Arquivos preservados: background 480 km oficial, invariant `x1.2562`, namelist/streams oficiais, obs restantes oficiais, CRTM oficial.

Criterio de sucesso: execucao termina com status 0 e apenas estatisticas do bloco `Aircraft` mudam.

Criterio de falha: erro de leitura IODA, variavel ausente, dimensao de obs inconsistente ou falha no operador.

Conclusao a tirar: se passar, o arquivo tutorial de aircraft e consumivel pelo YAML oficial com geometria oficial.

## Step 3: trocar somente `sondes`

Objetivo: testar radiossonda tutorial isoladamente.

Arquivos alterados:

```text
Data/ufo/testinput_tier_1/sondes_obs_2018041500_m.nc4
```

para equivalente:

```text
/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/data/observations/ioda/2018041500/sondes_obs_2018041500.h5
```

Arquivos preservados: tudo exceto o `obsfile` do bloco `Radiosonde`.

Criterio de sucesso: status 0; mudanca localizada nas estatisticas `Radiosonde`.

Criterio de falha: variaveis observadas incompativeis, alias insuficiente, erro de PreQC/Background Check.

Conclusao a tirar: se passar, `sondes` tutorial e compativel com o operador/alias usado no oficial.

## Step 4: trocar somente `sfc`

Objetivo: testar o arquivo de superficie tutorial, que alimenta dois blocos no YAML oficial.

Arquivos alterados: `obsfile` em ambos:

```text
Surface T,Q,Ps
Surface U,V
```

Equivalente:

```text
/p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/data/observations/ioda/2018041500/sfc_obs_2018041500.h5
```

Arquivos preservados: background/geometria oficiais e obs restantes oficiais.

Criterio de sucesso: status 0; mudancas localizadas nos blocos de superficie.

Criterio de falha: falta de `stationElevation`, `height`, `stationPressure`, `windEastward`, `windNorthward`, `airTemperature` ou `specificHumidity`; falha nos filtros `Difference Check`.

Conclusao a tirar: se falhar, separar os blocos `Surface T,Q,Ps` e `Surface U,V` em testes independentes.

## Step 5: decidir sobre GNSSRO e AMSUA

Objetivo: evitar misturar dificuldade de obs especializada com migracao de malha.

Arquivos alterados: nenhum inicialmente.

Arquivos preservados: `gnssro_obs_2018041500_s.nc4`, `amsua_n19_obs_2018041500_m.nc4` e `Data/UFOCoeff/` oficiais.

Criterio de sucesso: rodada com obs convencionais tutorial e obs especializadas oficiais passa.

Criterio de falha: falha em blocos que nao foram alterados indica efeito colateral de staging/caminho, nao dos dados tutorial.

Conclusao a tirar: so procurar equivalentes GNSSRO/AMSUA do tutorial depois de estabilizar conventional obs.

## Step 6: escolher uma malha tutorial alvo

Objetivo: preparar a troca coordenada de geometria.

Opcoes encontradas:

| Malha | Arquivos tutorial | Observacao |
| --- | --- | --- |
| `x1.10242` 240 km | `x1.10242.invariant.nc`, `x1.10242.graph.info.part.36`, `x1.10242.graph.info.part.64`, `namelist.atmosphere_240km`, `streams.atmosphere_240km`, background de 53.9 MB | candidato mais proximo dos testes FGAT anteriores |
| `x1.40962` 120 km | `x1.40962.invariant.nc`, `x1.40962.graph.info.part.36`, `x1.40962.graph.info.part.64`, `namelist.atmosphere_120km`, `streams.atmosphere_120km`, background de 215 MB | custo maior; nao misturar com 480 km |
| `x1.62691` conus15km | `conus15km/*` | regional; nao e substituto direto do global 480 km |

Arquivos alterados: nenhum neste passo; apenas decisao documentada.

Criterio de sucesso: uma unica malha alvo escolhida.

Criterio de falha: tentar usar background de uma malha com invariant/graph/namelist de outra.

Conclusao a tirar: background, invariant, graph, namelist e streams formam um pacote indivisivel.

## Step 7: trocar geometria como pacote minimo, nao arquivo por arquivo

Objetivo: migrar do pacote 480 km oficial para o pacote tutorial escolhido.

Arquivos alterados juntos:

```text
background mpasout.*.nc
invariant x1.*.invariant.nc
namelist.atmosphere_*
streams.atmosphere_*
graph/decomposition x1.*.graph.info.part.N
```

Arquivos preservados: YAML estrutural, obs ja validadas nos passos anteriores, CRTM oficial se AMSUA permanecer ativa.

Criterio de sucesso: geometria inicializa, background e invariant sao aceitos, e a execucao chega aos observadores.

Criterio de falha: erro de dimensao/celulas, erro de stream `invariant`, erro de decomposicao, erro de arquivo `graph.info.part.N`.

Conclusao a tirar: se falhar aqui, a falha e de pacote de malha/geometria, nao de obs.

## Step 8: tratar decomposicao por rank

Objetivo: garantir que o numero de ranks escolhido tem decomposicao existente para a malha escolhida.

Arquivos alterados: nenhum YAML, se usar rank ja suportado; gerar apenas decomposicao se necessario.

Arquivos preservados: pacote de malha escolhido.

Criterio de sucesso: arquivo `x1.*.graph.info.part.N` existe para o `N` usado.

Criterio de falha: erro por arquivo `graph.info.part.N` ausente.

Conclusao a tirar:

| Caso | Acao |
| --- | --- |
| 480 km oficial `np64` | gerar `x1.2562.graph.info.part.64` a partir de `x1.2562.graph.info`, por exemplo via `gpmetis`, antes de rodar `np64` |
| tutorial 240 km `np64` | usar `x1.10242.graph.info.part.64`, ja encontrado |
| tutorial 120 km `np64` | usar `x1.40962.graph.info.part.64`, ja encontrado |

## Step 9: so depois migrar aprendizado para o workflow

Objetivo: evitar que o workflow operacional absorva uma combinacao de arquivos ainda nao provada.

Arquivos alterados: nenhum ate a matriz isolada estar documentada.

Arquivos preservados: workflow operacional.

Criterio de sucesso: ha uma receita isolada com lista de arquivos, YAML e resultado.

Criterio de falha: tentar corrigir workflow antes de saber exatamente qual combinacao de dados passa.

Conclusao a tirar: a migracao para workflow deve ser mecanica, baseada no inventario aprovado.

## Proxima unica acao manual recomendada

Nao executar ainda o binario. A proxima acao manual recomendada e criar uma area isolada de staging e copiar o YAML oficial para ela, mantendo referencias oficiais intactas:

```sh
mkdir -p /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/build/analysis/passing_3dvar_to_tutorial/step01
cp /p/projetos/monan_das/joao.gerd/projects/MONAN-JEDI/mpas-jedi/test/testinput/3dvar.yaml \
  /p/projetos/monan_das/joao.gerd/projects/monan-jedi-workflow/build/analysis/passing_3dvar_to_tutorial/step01/3dvar.yaml
```

Esse comando apenas prepara a copia do YAML. Ele nao submete job e nao roda `mpasjedi_variational.x`.
