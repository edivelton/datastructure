# RideSmart AED2

Este projeto apresenta uma modelagem em grafos para comparar algoritmos de menor caminho em um cenário de deslocamento urbano.

A proposta considera:

- **A**: ponto de origem do passageiro;
- **B**: destino final;
- **P**: ponto de embarque escolhido dentro de um raio máximo de caminhada;
- **X**: distância máxima que o passageiro aceita caminhar.

O trecho **A -> P** é calculado sobre a malha de pedestres do OpenStreetMap, enquanto o trecho **P -> B** é calculado sobre a malha de direção para veículos. O notebook também compara o caso sem caminhada, isto é, **A = P**.

## Notebook

Arquivo principal:

```text
RideSmart_AED2.ipynb
```

O notebook pode ser executado localmente ou no Google Colab. Durante a execução, ele baixa os grafos da cidade escolhida com OSMnx, calcula os candidatos a ponto de embarque e gera tabelas e mapas interativos para análise.

## Algoritmos comparados

O projeto compara quatro algoritmos:

- Dijkstra simples;
- Dijkstra com heap;
- A*;
- Dijkstra bidirecional.

Cada algoritmo é avaliado em três critérios:

- menor rota em distância;
- rota mais rápida sem trânsito;
- rota mais rápida com trânsito sintético.

Além disso, cada critério é comparado em dois cenários:

- com caminhada até um ponto **P**;
- sem caminhada, usando **A = P**.

## Saídas geradas

Ao executar o notebook, são gerados:

- mapa inicial com **A**, **B**, raio de caminhada e candidatos **P**;
- mapa consolidado com filtros por algoritmo, critério e cenário;
- tabela de resultados consolidados;
- tabela de ganho ao caminhar.

As saídas são geradas localmente durante a execução do notebook e não precisam estar versionadas no repositório.

## Bibliotecas principais

- OSMnx;
- NetworkX;
- Pandas;
- NumPy;
- Folium;
- Matplotlib.

## Observação de modelagem

O ponto **P** nasce na malha de pedestres e é validado contra a malha de carros. Como os grafos do OSMnx são discretos, o ponto real de embarque é aproximado pelo nó viário mais próximo de **P**. Essa decisão mantém o problema alinhado à modelagem em grafos e aos algoritmos de menor caminho.
