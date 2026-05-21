# Analise Estrutural da Rede Viaria de Cruzeta/RN

Este projeto analisa a malha viaria de **Cruzeta/RN** como um grafo, usando dados do OpenStreetMap extraidos com OSMnx, metricas estruturais calculadas com NetworkX e visualizacao complementar no Gephi.

A proposta e responder, com base em evidencias quantitativas e visuais, quais elementos da rede urbana se comportam como hubs, quais pontos concentram intermediacao, como o k-core caracteriza a estrutura da rede e como a leitura geografica difere da leitura estrutural por layout de forca.

> Situacao atual: o notebook, as tabelas, os graficos quantitativos e o arquivo `.graphml` ja foram gerados. As visualizacoes finais feitas diretamente no Gephi ainda serao incorporadas ao projeto.

## Estrutura do projeto

```text
analise-rede-viaria-cruzeta-rn/
|-- analise_rede_urbana_cruzeta_rn.ipynb
|-- README.md
|-- requirements.txt
`-- outputs/
    |-- figures/
    |   |-- 01_distribuicao_grau_cdf.png
    |   |-- 02_grau_vs_betweenness.png
    |   `-- 03_distribuicao_core.png
    |-- gephi/
    |   `-- cruzeta_rn_rede_urbana.graphml
    `-- tables/
        |-- cruzeta_nodes_metrics.csv
        |-- cruzeta_summary.json
        |-- cruzeta_top_hubs_by_degree.csv
        |-- cruzeta_top_betweenness.csv
        |-- cruzeta_top_closeness.csv
        `-- demais tabelas auxiliares
```

## Tecnologias utilizadas

- **Python**: linguagem principal da analise.
- **OSMnx**: extracao da rede viaria a partir do OpenStreetMap.
- **NetworkX**: calculo das metricas de grafos.
- **Pandas/Numpy**: organizacao tabular e estatistica dos resultados.
- **Matplotlib/Seaborn**: graficos quantitativos no notebook.
- **Gephi**: visualizacoes geografica e estrutural da rede.

## Metodologia

### 1. Coleta da rede viaria

A rede foi obtida com `network_type="drive"`, de modo que o grafo represente ruas e rodovias destinadas ao trafego de veiculos.

```python
PLACE = "Cruzeta, Rio Grande do Norte, Brazil"
NETWORK_TYPE = "drive"

G_drive = ox.graph_from_place(
    PLACE,
    network_type=NETWORK_TYPE,
    simplify=True,
)
```

No grafo original do OSMnx, cada no representa uma intersecao ou ponto relevante da geometria viaria, e cada aresta representa um segmento de via. Os atributos `x` e `y` correspondem a longitude e latitude, sendo fundamentais para o Geo Layout no Gephi.

### 2. Conversao para grafo simples nao direcionado

O OSMnx retorna um `MultiDiGraph`, que pode conter direcao e arestas paralelas. Para as metricas estruturais do trabalho, o grafo foi convertido para uma versao simples e nao direcionada:

```python
simple = nx.Graph()

# Arestas paralelas sao colapsadas mantendo o menor comprimento.
if simple.has_edge(u, v):
    if length < simple[u][v]["length"]:
        simple[u][v].update(attrs)
else:
    simple.add_edge(u, v, **attrs)
```

Essa decisao evita duplicidade de arestas e permite o calculo direto de grau, core number, closeness e betweenness. O comprimento das vias (`length`) foi preservado para ponderar metricas baseadas em distancia.

### 3. Metricas calculadas

Foram calculadas as metricas exigidas no enunciado:

```python
degree = dict(G.degree())
degree_centrality = nx.degree_centrality(G)
closeness = nx.closeness_centrality(G, distance="length")
betweenness = nx.betweenness_centrality(G, normalized=True, weight="length")
core_number = nx.core_number(G)
eigenvector = nx.eigenvector_centrality(G, max_iter=2000, tol=1e-6)
```

A betweenness e a closeness foram ponderadas pelo comprimento das vias em metros. Isso torna a analise mais proxima do deslocamento real na rede urbana, em vez de considerar apenas a quantidade de arestas percorridas.

## Resumo da rede

| Indicador | Valor |
|---|---:|
| Regiao | Cruzeta/RN, Brasil |
| Consulta OSMnx | `Cruzeta, Rio Grande do Norte, Brazil` |
| Tipo de rede | `drive` |
| Nos | 607 |
| Arestas | 840 |
| Grafo conectado | True |
| Comprimento total aproximado | 211.44 km |
| Densidade | 0.004567 |
| Grau medio | 2.768 |
| Grau maximo | 4 |
| Maior core number | 2 |
| k escolhido | 2 |
| Nos no k-core escolhido | 480 (79.08%) |
| Diametro nao ponderado | 43 |
| Raio nao ponderado | 29 |

A rede analisada possui **607 nos** e **840 arestas**, com grau medio de **2.77**. O grau maximo observado foi **4**, valor tipico de redes viarias, nas quais cruzamentos raramente possuem muitas conexoes diretas.

## Distribuicao de grau

| Grau | Nos | Percentual |
| --- | --- | --- |
| 1 | 107 | 17.63% |
| 2 | 11 | 1.81% |
| 3 | 405 | 66.72% |
| 4 | 84 | 13.84% |

![Distribuicao de grau e CDF](outputs/figures/01_distribuicao_grau_cdf.png)

A distribuicao mostra que a maior parte dos nos possui grau 3. Isso indica uma rede predominantemente formada por intersecoes simples, com poucos nos de grau 4 e uma parcela relevante de nos de grau 1, que representam extremidades, ruas sem continuidade ou acessos perifericos.

A linha vermelha representa a **CDF** (*Cumulative Distribution Function*), isto e, a probabilidade acumulada de um no ter grau menor ou igual a determinado valor. Por isso, o valor em grau 2 inclui os nos de grau 1 e de grau 2.

## Distribuicao por core number

| Core number | Nos | Percentual |
| --- | --- | --- |
| 1 | 127 | 20.92% |
| 2 | 480 | 79.08% |

![Distribuicao por core number](outputs/figures/03_distribuicao_core.png)

O maior core number encontrado foi **2**. O k-core escolhido, portanto, foi **k = 2**, contendo **480 nos**, aproximadamente **79.08%** da rede.

Esse resultado indica que o k-core principal retira nos perifericos ou pouco conectados, mas ainda preserva uma parcela ampla da rede. Assim, o k-core ajuda a separar a estrutura persistente da malha, mas nao deve ser usado isoladamente para identificar os principais hubs.

## Resumo estatistico das metricas

| Metrica | Media | Desvio | Minimo | Mediana | Maximo |
| --- | --- | --- | --- | --- | --- |
| degree | 2.767710 | 0.899438 | 1.000000 | 3.000000 | 4.000000 |
| degree_centrality | 0.004567 | 0.001484 | 0.001650 | 0.004950 | 0.006601 |
| betweenness | 0.044947 | 0.075399 | 0.000000 | 0.015122 | 0.387677 |
| closeness | 0.000232 | 0.000078 | 0.000066 | 0.000259 | 0.000321 |
| core_number | 1.790774 | 0.407091 | 1.000000 | 2.000000 | 2.000000 |

O desvio padrao da betweenness e maior que sua media, indicando concentracao dessa metrica em poucos nos. Isso sugere que alguns pontos funcionam como passagens estruturais importantes, mesmo quando nao sao os nos de maior grau.

## Hubs por grau

A tabela abaixo lista os 10 maiores hubs por grau. Como ha muitos empates em redes viarias, os empates foram ordenados por betweenness.

| No | Grau | Betweenness | Closeness | Core | Vias incidentes |
| --- | --- | --- | --- | --- | --- |
| 2098173324 | 4 | 0.387677 | 0.000311 | 2 | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / Rua Cipriano Lopes Galvao / residential / residential |
| 2098133818 | 4 | 0.285247 | 0.000320 | 2 | Avenida Doutor Silvio Bezerra de Melo / secondary / Rua Emilio Vale / residential |
| 2098918029 | 4 | 0.277970 | 0.000320 | 2 | Praca Celso Azevedo / residential / Rua Felix Pereira de Araujo / secondary / Rua Sebastiao Araujo / residential |
| 2098943368 | 4 | 0.267354 | 0.000320 | 2 | Avenida Doutor Silvio Bezerra de Melo; Rua Felix Pereira de Araujo / secondary / Praca Celso Azevedo / residential / Rua Felix Pereira de Araujo / secondary / Rua Joao Lopes de Araujo / residential |
| 2098144101 | 4 | 0.266808 | 0.000320 | 2 | Avenida Doutor Silvio Bezerra de Melo / secondary / Avenida Doutor Silvio Bezerra de Melo; Rua Felix Pereira de Araujo / secondary / Rua Miguel Laurentino / residential / residential |
| 2098144124 | 4 | 0.266170 | 0.000318 | 2 | Rua Felix Pereira de Araujo / secondary / Rua Joao Gomes / residential / Rua Raimundo Hermes Dantas / residential |
| 7375819476 | 4 | 0.265996 | 0.000321 | 2 | Avenida Doutor Silvio Bezerra de Melo / secondary / Rua Joao XXIII / tertiary |
| 2098144097 | 4 | 0.265843 | 0.000320 | 2 | Avenida Doutor Silvio Bezerra de Melo / secondary / Rua Geraldo Lopes de Araujo / residential |
| 2098144128 | 4 | 0.264206 | 0.000319 | 2 | Rua Felix Pereira de Araujo / secondary / Rua Leoncio Pires Galvao / residential |
| 7375826292 | 4 | 0.237373 | 0.000319 | 2 | Avenida Doutor Silvio Bezerra de Melo / secondary / residential |

## Nos com maior betweenness

A betweenness identifica nos que aparecem com frequencia nos menores caminhos da rede. Esses nos podem ser criticos para mobilidade porque conectam regioes distintas.

| No | Grau | Betweenness | Closeness | Core | Vias incidentes |
| --- | --- | --- | --- | --- | --- |
| 2098173324 | 4 | 0.387677 | 0.000311 | 2 | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / Rua Cipriano Lopes Galvao / residential / residential |
| 2340784260 | 3 | 0.380520 | 0.000280 | 2 | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / RN-288 / secondary / residential |
| 314979940 | 3 | 0.372648 | 0.000277 | 2 | RN-288 / secondary / Rua Maria Izaura de Araujo / residential |
| 7374755393 | 3 | 0.368791 | 0.000277 | 2 | RN-288 / secondary / residential |
| 2340784276 | 3 | 0.358236 | 0.000275 | 2 | RN-288 / secondary / residential |
| 2340784271 | 3 | 0.357767 | 0.000265 | 2 | RN-288 / secondary / unclassified |
| 7375628953 | 3 | 0.344227 | 0.000271 | 2 | RN-288 / secondary / residential |
| 2340028998 | 3 | 0.343774 | 0.000272 | 2 | RN-288 / secondary / Rua Cipriana Bezerra de Medeiros / residential |
| 2098144099 | 3 | 0.335957 | 0.000313 | 2 | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / Rua Felix Pereira de Araujo / secondary |
| 2340078036 | 3 | 0.334943 | 0.000312 | 2 | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / Rua Francisco Raimundo de Araujo / residential |

Observa-se que varios nos de maior betweenness estao associados a **RN-288** e a **Avenida Carmelita Monteiro da Silva**, sugerindo um eixo de circulacao importante na estrutura da rede. Essa interpretacao ainda deve ser confirmada visualmente no Gephi.

## Comparacao: grau x betweenness

![Grau local x intermediacao](outputs/figures/02_grau_vs_betweenness.png)

O grafico compara conectividade local (`degree`) com intermediacao global (`betweenness`). A cor representa o `core_number`, e os circulos destacados indicam os 10 maiores valores de betweenness.

Apenas **1** no do top 10 por betweenness tambem aparece no top 10% por grau. Isso mostra que grau e betweenness capturam dimensoes diferentes da importancia estrutural.

| No | Grau | Betweenness | Core | Top grau | Top betweenness | Vias incidentes |
| --- | --- | --- | --- | --- | --- | --- |
| 2098173324 | 4 | 0.387677 | 2 | True | True | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / Rua Cipriano Lopes Galvao / residential / residential |
| 2340784260 | 3 | 0.380520 | 2 | False | True | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / RN-288 / secondary / residential |
| 314979940 | 3 | 0.372648 | 2 | False | True | RN-288 / secondary / Rua Maria Izaura de Araujo / residential |
| 7374755393 | 3 | 0.368791 | 2 | False | True | RN-288 / secondary / residential |
| 2340784276 | 3 | 0.358236 | 2 | False | True | RN-288 / secondary / residential |
| 2340784271 | 3 | 0.357767 | 2 | False | True | RN-288 / secondary / unclassified |
| 7375628953 | 3 | 0.344227 | 2 | False | True | RN-288 / secondary / residential |
| 2340028998 | 3 | 0.343774 | 2 | False | True | RN-288 / secondary / Rua Cipriana Bezerra de Medeiros / residential |
| 2098144099 | 3 | 0.335957 | 2 | False | True | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / Rua Felix Pereira de Araujo / secondary |
| 2340078036 | 3 | 0.334943 | 2 | False | True | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / Rua Francisco Raimundo de Araujo / residential |
| 2098133818 | 4 | 0.285247 | 2 | True | False | Avenida Doutor Silvio Bezerra de Melo / secondary / Rua Emilio Vale / residential |
| 2098918029 | 4 | 0.277970 | 2 | True | False | Praca Celso Azevedo / residential / Rua Felix Pereira de Araujo / secondary / Rua Sebastiao Araujo / residential |

## Nos com maior closeness

A closeness centrality mede quao proximo um no esta dos demais, considerando as distancias ponderadas pelo comprimento das vias. Valores maiores indicam nos com menor distancia media ate o restante da rede.

| No | Grau | Betweenness | Closeness | Core | Vias incidentes |
| --- | --- | --- | --- | --- | --- |
| 7375819476 | 4 | 0.265996 | 0.000321 | 2 | Avenida Doutor Silvio Bezerra de Melo / secondary / Rua Joao XXIII / tertiary |
| 7375826296 | 3 | 0.144718 | 0.000321 | 2 | Avenida Doutor Silvio Bezerra de Melo / secondary / Rua Joao XXIII / tertiary |
| 2098144081 | 3 | 0.278941 | 0.000321 | 2 | Avenida Doutor Silvio Bezerra de Melo / secondary |
| 7375819471 | 3 | 0.220402 | 0.000321 | 2 | Avenida Doutor Silvio Bezerra de Melo / secondary / Rua Manoel Martiniano de Medeiros / tertiary |
| 7375819470 | 4 | 0.152268 | 0.000321 | 2 | Avenida Doutor Silvio Bezerra de Melo / secondary / Rua Manoel Martiniano de Medeiros / tertiary |
| 2098144093 | 3 | 0.279639 | 0.000321 | 2 | Avenida Doutor Silvio Bezerra de Melo / secondary / residential |
| 2098136084 | 3 | 0.092966 | 0.000320 | 2 | Rua Joao XXIII / tertiary / residential |
| 2098133818 | 4 | 0.285247 | 0.000320 | 2 | Avenida Doutor Silvio Bezerra de Melo / secondary / Rua Emilio Vale / residential |
| 2340015648 | 3 | 0.091307 | 0.000320 | 2 | Rua Antonio Apolinario / residential / Rua Joao XXIII / tertiary |
| 2098144097 | 4 | 0.265843 | 0.000320 | 2 | Avenida Doutor Silvio Bezerra de Melo / secondary / Rua Geraldo Lopes de Araujo / residential |

## Arquivo para Gephi

O arquivo abaixo contem o grafo enriquecido com as metricas calculadas:

```text
outputs/gephi/cruzeta_rn_rede_urbana.graphml
```

Atributos importantes presentes nos nos:

| Atributo | Uso no Gephi |
|---|---|
| `x` | Longitude para Geo Layout |
| `y` | Latitude para Geo Layout |
| `degree` | Tamanho dos nos |
| `betweenness` | Destaque de pontos de intermediacao |
| `closeness` | Analise de proximidade na rede |
| `core_number` | Cor dos nos e filtro por k-core |
| `top_10pct_degree` | Filtro do top 10% por grau |
| `top_10_betweenness` | Destaque dos 10 maiores nos por betweenness |
| `selected_k_core` | Filtro do k-core escolhido |

## Procedimento previsto no Gephi

A etapa de Gephi ainda deve ser finalizada. O procedimento planejado e:

1. Importar `outputs/gephi/cruzeta_rn_rede_urbana.graphml`.
2. Aplicar **Geo Layout** usando:
   - longitude = `x`
   - latitude = `y`
3. Codificar visualmente:
   - tamanho do no proporcional a `degree`;
   - cor do no por `core_number`;
   - destaque para `top_10_betweenness == 1`.
4. Criar uma visualizacao estrutural com **ForceAtlas2**.
5. Aplicar os filtros obrigatorios:
   - `top_10pct_degree == 1`;
   - `selected_k_core == 1` ou `core_number >= 2`.

## Respostas preliminares as questoes do professor

### 1. Os nos com maior grau coincidem com os nos de maior betweenness?

Parcialmente, mas a coincidencia e pequena. Apenas **1** no entre os 10 maiores por betweenness tambem aparece no top 10% por grau. Isso indica que os nos com mais conexoes locais nao sao necessariamente os principais pontos de passagem da rede.

### 2. O nucleo identificado pelo k-core coincide com os principais hubs?

Os principais nos listados nas tabelas aparecem com `core_number = 2`, isto e, pertencem ao k-core principal. No entanto, esse k-core contem **79.08%** dos nos da rede, portanto ele e amplo. O k-core ajuda a remover extremidades, mas nao e seletivo o suficiente para identificar sozinho os hubs mais importantes.

### 3. O que a betweenness revela que o grau nao revela?

A betweenness revela pontos de intermediacao. Varios nos com grau 3 aparecem entre os maiores valores de betweenness, especialmente em trechos associados a RN-288. Isso mostra que um no pode nao ter o maior numero de conexoes diretas, mas ainda ser estruturalmente importante por estar no caminho entre diferentes partes da rede.

### 4. O que muda entre a posicao geografica real e o layout estrutural?

Essa pergunta ainda depende das visualizacoes no Gephi. A expectativa metodologica e que o Geo Layout preserve a forma real da cidade, enquanto o ForceAtlas2 reorganize a rede pela conectividade, evidenciando agrupamentos, pontes e corredores estruturais.

### 5. Existem regioes criticas para mobilidade urbana na area analisada?

Os dados de betweenness sugerem pontos criticos associados a RN-288 e a Avenida Carmelita Monteiro da Silva. Essa conclusao ainda deve ser validada no Gephi, observando a posicao geografica desses nos destacados e sua relacao com a malha urbana.

### 6. A rede parece homogenea ou apresenta concentracao estrutural?

A rede nao parece completamente homogenea. Embora a maioria dos nos tenha grau 3, a betweenness se concentra em poucos pontos. Isso sugere que a conectividade local e relativamente padronizada, mas a funcao estrutural dos nos e desigual.

### 7. Os resultados fazem sentido considerando o conhecimento urbano da regiao escolhida?

A resposta final depende da conferencia visual no Gephi e do conhecimento local sobre Cruzeta/RN. Ate o momento, a recorrencia da RN-288 entre os nos de maior betweenness e coerente com a interpretacao de uma via de ligacao importante, mas a apresentacao final deve confirmar essa leitura na visualizacao geografica.

## Como reproduzir

1. Abrir o notebook `analise_rede_urbana_cruzeta_rn.ipynb` no Google Colab.
2. Executar as celulas em ordem.
3. Ao final, verificar os arquivos em:
   - `outputs/figures/`
   - `outputs/tables/`
   - `outputs/gephi/`
4. Importar o GraphML no Gephi para gerar as visualizacoes finais.

## Status do projeto

- [x] Coleta da rede viaria com OSMnx.
- [x] Calculo das metricas estruturais com NetworkX.
- [x] Geracao de tabelas e graficos essenciais.
- [x] Exportacao do GraphML para o Gephi.
- [ ] Producao das visualizacoes finais no Gephi.
- [ ] Inclusao das imagens finais do Gephi neste README.
- [ ] Insercao do link do video Loom apos a gravacao.
