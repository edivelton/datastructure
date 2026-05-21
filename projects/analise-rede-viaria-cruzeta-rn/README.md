# Análise Estrutural da Rede Viária de Cruzeta/RN

Este projeto analisa a malha viária de **Cruzeta/RN** como um grafo, usando dados do OpenStreetMap extraídos com OSMnx, métricas estruturais calculadas com NetworkX e visualização complementar no Gephi.

A proposta é responder, com base em evidências quantitativas e visuais, quais elementos da rede urbana se comportam como hubs, quais pontos concentram intermediação, como o k-core caracteriza a estrutura da rede e como a leitura geográfica difere da leitura estrutural por layout de força.

> Situação atual: o notebook, as tabelas, os gráficos quantitativos e o arquivo `.graphml` já foram gerados. As visualizações finais feitas diretamente no Gephi ainda serão incorporadas ao projeto.

## Autores

| Nome | Curso | Matrícula | Perfil |
|---|---|---:|---|
| EDIVELTON RAFAETT SILVA DE ARAÚJO | Engenharia de Computação/CT | 20230094613 | [edivelton](https://github.com/edivelton) |
| FRANCISCO MICARLOS TEIXEIRA PINTO | Engenharia de Computação/CT | 20260000734 | [micarlio](https://github.com/micarlio) |

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

- **Python**: linguagem principal da análise.
- **OSMnx**: extração da rede viária a partir do OpenStreetMap.
- **NetworkX**: cálculo das métricas de grafos.
- **Pandas/Numpy**: organização tabular e estatística dos resultados.
- **Matplotlib/Seaborn**: gráficos quantitativos no notebook.
- **Gephi**: visualizações geográfica e estrutural da rede.

## Metodologia

### 1. Coleta da rede viária

A rede foi obtida com `network_type="drive"`, de modo que o grafo represente ruas e rodovias destinadas ao tráfego de veículos.

```python
PLACE = "Cruzeta, Rio Grande do Norte, Brazil"
NETWORK_TYPE = "drive"

G_drive = ox.graph_from_place(
    PLACE,
    network_type=NETWORK_TYPE,
    simplify=True,
)
```

No grafo original do OSMnx, cada nó representa uma interseção ou um ponto relevante da geometria viária, e cada aresta representa um segmento de via. Os atributos `x` e `y` correspondem à longitude e à latitude, sendo fundamentais para o Geo Layout no Gephi.

### 2. Conversão para grafo simples não direcionado

O OSMnx retorna um `MultiDiGraph`, que pode conter direção e arestas paralelas. Para as métricas estruturais do trabalho, o grafo foi convertido para uma versão simples e não direcionada:

```python
simple = nx.Graph()

# Arestas paralelas são colapsadas mantendo o menor comprimento.
if simple.has_edge(u, v):
    if length < simple[u][v]["length"]:
        simple[u][v].update(attrs)
else:
    simple.add_edge(u, v, **attrs)
```

Essa decisão evita duplicidade de arestas e permite o cálculo direto de grau, core number, closeness e betweenness. O comprimento das vias (`length`) foi preservado para ponderar métricas baseadas em distância.

### 3. Métricas calculadas

Foram calculadas as métricas exigidas no enunciado:

```python
degree = dict(G.degree())
degree_centrality = nx.degree_centrality(G)
closeness = nx.closeness_centrality(G, distance="length")
betweenness = nx.betweenness_centrality(G, normalized=True, weight="length")
core_number = nx.core_number(G)
eigenvector = nx.eigenvector_centrality(G, max_iter=2000, tol=1e-6)
```

A betweenness e a closeness foram ponderadas pelo comprimento das vias em metros. Isso torna a análise mais próxima do deslocamento real na rede urbana, em vez de considerar apenas a quantidade de arestas percorridas.

## Resumo da rede

| Indicador | Valor |
|---|---:|
| Região | Cruzeta/RN, Brasil |
| Consulta OSMnx | `Cruzeta, Rio Grande do Norte, Brazil` |
| Tipo de rede | `drive` |
| Nós | 607 |
| Arestas | 840 |
| Grafo conectado | Sim |
| Comprimento total aproximado | 211.44 km |
| Densidade | 0.004567 |
| Grau médio | 2.768 |
| Grau máximo | 4 |
| Maior core number | 2 |
| k escolhido | 2 |
| Nós no k-core escolhido | 480 (79.08%) |
| Diâmetro não ponderado | 43 |
| Raio não ponderado | 29 |

A rede analisada possui **607 nós** e **840 arestas**, com grau médio de **2.77**. O grau máximo observado foi **4**, valor típico de redes viárias, nas quais cruzamentos raramente possuem muitas conexões diretas.

## Distribuição de grau

| Grau | Nós | Percentual |
| --- | ---: | ---: |
| 1 | 107 | 17.63% |
| 2 | 11 | 1.81% |
| 3 | 405 | 66.72% |
| 4 | 84 | 13.84% |

![Distribuição de grau e CDF](outputs/figures/01_distribuicao_grau_cdf.png)

A distribuição mostra que a maior parte dos nós possui grau 3. Isso indica uma rede predominantemente formada por interseções simples, com poucos nós de grau 4 e uma parcela relevante de nós de grau 1, que representam extremidades, ruas sem continuidade ou acessos periféricos.

A linha vermelha representa a **CDF** (*Cumulative Distribution Function*), isto é, a probabilidade acumulada de um nó ter grau menor ou igual a determinado valor. Por isso, o valor em grau 2 inclui os nós de grau 1 e de grau 2.

## Distribuição por core number

| Core number | Nós | Percentual |
| --- | ---: | ---: |
| 1 | 127 | 20.92% |
| 2 | 480 | 79.08% |

![Distribuição por core number](outputs/figures/03_distribuicao_core.png)

O maior core number encontrado foi **2**. O k-core escolhido, portanto, foi **k = 2**, contendo **480 nós**, aproximadamente **79.08%** da rede.

Esse resultado indica que o k-core principal retira nós periféricos ou pouco conectados, mas ainda preserva uma parcela ampla da rede. Assim, o k-core ajuda a separar a estrutura persistente da malha, mas não deve ser usado isoladamente para identificar os principais hubs.

## Resumo estatístico das métricas

| Métrica | Média | Desvio | Mínimo | Mediana | Máximo |
| --- | ---: | ---: | ---: | ---: | ---: |
| degree | 2.767710 | 0.899438 | 1.000000 | 3.000000 | 4.000000 |
| degree_centrality | 0.004567 | 0.001484 | 0.001650 | 0.004950 | 0.006601 |
| betweenness | 0.044947 | 0.075399 | 0.000000 | 0.015122 | 0.387677 |
| closeness | 0.000232 | 0.000078 | 0.000066 | 0.000259 | 0.000321 |
| core_number | 1.790774 | 0.407091 | 1.000000 | 2.000000 | 2.000000 |

O desvio padrão da betweenness é maior que sua média, indicando concentração dessa métrica em poucos nós. Isso sugere que alguns pontos funcionam como passagens estruturais importantes, mesmo quando não são os nós de maior grau.

## Hubs por grau

A tabela abaixo lista os 10 maiores hubs por grau. Como há muitos empates em redes viárias, os empates foram ordenados por betweenness.

| Nó | Grau | Betweenness | Closeness | Core | Vias incidentes |
| --- | ---: | ---: | ---: | ---: | --- |
| 2098173324 | 4 | 0.387677 | 0.000311 | 2 | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / Rua Cipriano Lopes Galvão / residential / residential |
| 2098133818 | 4 | 0.285247 | 0.000320 | 2 | Avenida Doutor Sílvio Bezerra de Melo / secondary / Rua Emílio Vale / residential |
| 2098918029 | 4 | 0.277970 | 0.000320 | 2 | Praça Celso Azevedo / residential / Rua Félix Pereira de Araújo / secondary / Rua Sebastião Araújo / residential |
| 2098943368 | 4 | 0.267354 | 0.000320 | 2 | Avenida Doutor Sílvio Bezerra de Melo; Rua Félix Pereira de Araújo / secondary / Praça Celso Azevedo / residential / Rua Félix Pereira de Araújo / secondary / Rua João Lopes de Araújo / residential |
| 2098144101 | 4 | 0.266808 | 0.000320 | 2 | Avenida Doutor Sílvio Bezerra de Melo / secondary / Avenida Doutor Sílvio Bezerra de Melo; Rua Félix Pereira de Araújo / secondary / Rua Miguel Laurentino / residential / residential |
| 2098144124 | 4 | 0.266170 | 0.000318 | 2 | Rua Félix Pereira de Araújo / secondary / Rua João Gomes / residential / Rua Raimundo Hermes Dantas / residential |
| 7375819476 | 4 | 0.265996 | 0.000321 | 2 | Avenida Doutor Sílvio Bezerra de Melo / secondary / Rua João XXIII / tertiary |
| 2098144097 | 4 | 0.265843 | 0.000320 | 2 | Avenida Doutor Sílvio Bezerra de Melo / secondary / Rua Geraldo Lopes de Araújo / residential |
| 2098144128 | 4 | 0.264206 | 0.000319 | 2 | Rua Félix Pereira de Araújo / secondary / Rua Leôncio Pires Galvão / residential |
| 7375826292 | 4 | 0.237373 | 0.000319 | 2 | Avenida Doutor Sílvio Bezerra de Melo / secondary / residential |

## Nós com maior betweenness

A betweenness identifica nós que aparecem com frequência nos menores caminhos da rede. Esses nós podem ser críticos para mobilidade porque conectam regiões distintas.

| Nó | Grau | Betweenness | Closeness | Core | Vias incidentes |
| --- | ---: | ---: | ---: | ---: | --- |
| 2098173324 | 4 | 0.387677 | 0.000311 | 2 | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / Rua Cipriano Lopes Galvão / residential / residential |
| 2340784260 | 3 | 0.380520 | 0.000280 | 2 | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / RN-288 / secondary / residential |
| 314979940 | 3 | 0.372648 | 0.000277 | 2 | RN-288 / secondary / Rua Maria Izaura de Araújo / residential |
| 7374755393 | 3 | 0.368791 | 0.000277 | 2 | RN-288 / secondary / residential |
| 2340784276 | 3 | 0.358236 | 0.000275 | 2 | RN-288 / secondary / residential |
| 2340784271 | 3 | 0.357767 | 0.000265 | 2 | RN-288 / secondary / unclassified |
| 7375628953 | 3 | 0.344227 | 0.000271 | 2 | RN-288 / secondary / residential |
| 2340028998 | 3 | 0.343774 | 0.000272 | 2 | RN-288 / secondary / Rua Cipriana Bezerra de Medeiros / residential |
| 2098144099 | 3 | 0.335957 | 0.000313 | 2 | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / Rua Félix Pereira de Araújo / secondary |
| 2340078036 | 3 | 0.334943 | 0.000312 | 2 | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / Rua Francisco Raimundo de Araújo / residential |

Observa-se que vários nós de maior betweenness estão associados à **RN-288** e à **Avenida Carmelita Monteiro da Silva**, sugerindo um eixo de circulação importante na estrutura da rede. Essa interpretação ainda deve ser confirmada visualmente no Gephi.

## Comparação: grau x betweenness

![Grau local x intermediação](outputs/figures/02_grau_vs_betweenness.png)

O gráfico compara conectividade local (`degree`) com intermediação global (`betweenness`). A cor representa o `core_number`, e os círculos destacados indicam os 10 maiores valores de betweenness.

Apenas **1** nó do top 10 por betweenness também aparece no top 10% por grau. Isso mostra que grau e betweenness capturam dimensões diferentes da importância estrutural.

| Nó | Grau | Betweenness | Core | Top grau | Top betweenness | Vias incidentes |
| --- | ---: | ---: | ---: | --- | --- | --- |
| 2098173324 | 4 | 0.387677 | 2 | True | True | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / Rua Cipriano Lopes Galvão / residential / residential |
| 2340784260 | 3 | 0.380520 | 2 | False | True | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / RN-288 / secondary / residential |
| 314979940 | 3 | 0.372648 | 2 | False | True | RN-288 / secondary / Rua Maria Izaura de Araújo / residential |
| 7374755393 | 3 | 0.368791 | 2 | False | True | RN-288 / secondary / residential |
| 2340784276 | 3 | 0.358236 | 2 | False | True | RN-288 / secondary / residential |
| 2340784271 | 3 | 0.357767 | 2 | False | True | RN-288 / secondary / unclassified |
| 7375628953 | 3 | 0.344227 | 2 | False | True | RN-288 / secondary / residential |
| 2340028998 | 3 | 0.343774 | 2 | False | True | RN-288 / secondary / Rua Cipriana Bezerra de Medeiros / residential |
| 2098144099 | 3 | 0.335957 | 2 | False | True | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / Rua Félix Pereira de Araújo / secondary |
| 2340078036 | 3 | 0.334943 | 2 | False | True | Avenida Carmelita Monteiro da Silva / RN-288 / secondary / Rua Francisco Raimundo de Araújo / residential |
| 2098133818 | 4 | 0.285247 | 2 | True | False | Avenida Doutor Sílvio Bezerra de Melo / secondary / Rua Emílio Vale / residential |
| 2098918029 | 4 | 0.277970 | 2 | True | False | Praça Celso Azevedo / residential / Rua Félix Pereira de Araújo / secondary / Rua Sebastião Araújo / residential |

## Nós com maior closeness

A closeness centrality mede quão próximo um nó está dos demais, considerando as distâncias ponderadas pelo comprimento das vias. Valores maiores indicam nós com menor distância média até o restante da rede.

| Nó | Grau | Betweenness | Closeness | Core | Vias incidentes |
| --- | ---: | ---: | ---: | ---: | --- |
| 7375819476 | 4 | 0.265996 | 0.000321 | 2 | Avenida Doutor Sílvio Bezerra de Melo / secondary / Rua João XXIII / tertiary |
| 7375826296 | 3 | 0.144718 | 0.000321 | 2 | Avenida Doutor Sílvio Bezerra de Melo / secondary / Rua João XXIII / tertiary |
| 2098144081 | 3 | 0.278941 | 0.000321 | 2 | Avenida Doutor Sílvio Bezerra de Melo / secondary |
| 7375819471 | 3 | 0.220402 | 0.000321 | 2 | Avenida Doutor Sílvio Bezerra de Melo / secondary / Rua Manoel Martiniano de Medeiros / tertiary |
| 7375819470 | 4 | 0.152268 | 0.000321 | 2 | Avenida Doutor Sílvio Bezerra de Melo / secondary / Rua Manoel Martiniano de Medeiros / tertiary |
| 2098144093 | 3 | 0.279639 | 0.000321 | 2 | Avenida Doutor Sílvio Bezerra de Melo / secondary / residential |
| 2098136084 | 3 | 0.092966 | 0.000320 | 2 | Rua João XXIII / tertiary / residential |
| 2098133818 | 4 | 0.285247 | 0.000320 | 2 | Avenida Doutor Sílvio Bezerra de Melo / secondary / Rua Emílio Vale / residential |
| 2340015648 | 3 | 0.091307 | 0.000320 | 2 | Rua Antônio Apolinário / residential / Rua João XXIII / tertiary |
| 2098144097 | 4 | 0.265843 | 0.000320 | 2 | Avenida Doutor Sílvio Bezerra de Melo / secondary / Rua Geraldo Lopes de Araújo / residential |

## Arquivo para Gephi

O arquivo abaixo contém o grafo enriquecido com as métricas calculadas:

```text
outputs/gephi/cruzeta_rn_rede_urbana.graphml
```

Atributos importantes presentes nos nós:

| Atributo | Uso no Gephi |
|---|---|
| `x` | Longitude para Geo Layout |
| `y` | Latitude para Geo Layout |
| `degree` | Tamanho dos nós |
| `betweenness` | Destaque de pontos de intermediação |
| `closeness` | Análise de proximidade na rede |
| `core_number` | Cor dos nós e filtro por k-core |
| `top_10pct_degree` | Filtro do top 10% por grau |
| `top_10_betweenness` | Destaque dos 10 maiores nós por betweenness |
| `selected_k_core` | Filtro do k-core escolhido |

## Procedimento previsto no Gephi

A etapa de Gephi ainda deve ser finalizada. O procedimento planejado é:

1. Importar `outputs/gephi/cruzeta_rn_rede_urbana.graphml`.
2. Aplicar **Geo Layout** usando:
   - longitude = `x`
   - latitude = `y`
3. Codificar visualmente:
   - tamanho do nó proporcional a `degree`;
   - cor do nó por `core_number`;
   - destaque para `top_10_betweenness == 1`.
4. Criar uma visualização estrutural com **ForceAtlas2**.
5. Aplicar os filtros obrigatórios:
   - `top_10pct_degree == 1`;
   - `selected_k_core == 1` ou `core_number >= 2`.

## Respostas preliminares às questões do professor

### 1. Os nós com maior grau coincidem com os nós de maior betweenness?

Parcialmente, mas a coincidência é pequena. Apenas **1** nó entre os 10 maiores por betweenness também aparece no top 10% por grau. Isso indica que os nós com mais conexões locais não são necessariamente os principais pontos de passagem da rede.

### 2. O núcleo identificado pelo k-core coincide com os principais hubs?

Os principais nós listados nas tabelas aparecem com `core_number = 2`, isto é, pertencem ao k-core principal. No entanto, esse k-core contém **79.08%** dos nós da rede, portanto ele é amplo. O k-core ajuda a remover extremidades, mas não é seletivo o suficiente para identificar sozinho os hubs mais importantes.

### 3. O que a betweenness revela que o grau não revela?

A betweenness revela pontos de intermediação. Vários nós com grau 3 aparecem entre os maiores valores de betweenness, especialmente em trechos associados à RN-288. Isso mostra que um nó pode não ter o maior número de conexões diretas, mas ainda ser estruturalmente importante por estar no caminho entre diferentes partes da rede.

### 4. O que muda entre a posição geográfica real e o layout estrutural?

Essa pergunta ainda depende das visualizações no Gephi. A expectativa metodológica é que o Geo Layout preserve a forma real da cidade, enquanto o ForceAtlas2 reorganize a rede pela conectividade, evidenciando agrupamentos, pontes e corredores estruturais.

### 5. Existem regiões críticas para mobilidade urbana na área analisada?

Os dados de betweenness sugerem pontos críticos associados à RN-288 e à Avenida Carmelita Monteiro da Silva. Essa conclusão ainda deve ser validada no Gephi, observando a posição geográfica desses nós destacados e sua relação com a malha urbana.

### 6. A rede parece homogênea ou apresenta concentração estrutural?

A rede não parece completamente homogênea. Embora a maioria dos nós tenha grau 3, a betweenness se concentra em poucos pontos. Isso sugere que a conectividade local é relativamente padronizada, mas a função estrutural dos nós é desigual.

### 7. Os resultados fazem sentido considerando o conhecimento urbano da região escolhida?

A resposta final depende da conferência visual no Gephi e do conhecimento local sobre Cruzeta/RN. Até o momento, a recorrência da RN-288 entre os nós de maior betweenness é coerente com a interpretação de uma via de ligação importante, mas a apresentação final deve confirmar essa leitura na visualização geográfica.

## Como reproduzir

1. Abrir o notebook `analise_rede_urbana_cruzeta_rn.ipynb` no Google Colab.
2. Executar as células em ordem.
3. Ao final, verificar os arquivos em:
   - `outputs/figures/`
   - `outputs/tables/`
   - `outputs/gephi/`
4. Importar o GraphML no Gephi para gerar as visualizações finais.

## Status do projeto

- [x] Coleta da rede viária com OSMnx.
- [x] Cálculo das métricas estruturais com NetworkX.
- [x] Geração de tabelas e gráficos essenciais.
- [x] Exportação do GraphML para o Gephi.
- [ ] Produção das visualizações finais no Gephi.
- [ ] Inclusão das imagens finais do Gephi neste README.
- [ ] Inserção do link do vídeo Loom após a gravação.
