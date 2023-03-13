# campeonato-brasileiro-tableau-sql

Este projeto foi realizado como um exercício para praticar consultas SQL e criação de gráficos com Tableu. Os dados foram coletados do site [Base dos Dados](https://basedosdados.org) e a ferramenta utilizada foi o Big Query do Google Cloud.

O dashboard Tablaeu se encontra nesse link: https://public.tableau.com/app/profile/bruno.perdig.o.de.oliveira/viz/CampeonatoBrasileiro2020-v2/Painel1

A [base de dados](https://basedosdados.org/dataset/mundo-transfermarkt-competicoes?bdm_table=brasileirao_serie_a), possui todos os registros em uma única tabela. Os dados são coletados para cada jogo, sendo necessário algumas manipulações para se obter informações gerais sobre o campeonato.

Muitas colunas estavam com dados incompletos, especialmente em anos menos recentes. Isso limitou a possibilidade de fazer análises históricas. Portanto, resolvi focar em apenas uma edição do campeonato, a de 2020. A seguir está uma explicação de algumas cada consulta realizada.

---

## Média de quantos chutes a gol um time tenta até fazer um gol
```sql

SELECT time_vis,
    (chutes_total_vis + mandante.chutes_total_man) AS chutes_total,
    (gols_total_vis + mandante.gols_total_man) AS gols_total,
    ROUND((chutes_total_vis + mandante.chutes_total_man) / (gols_total_vis + mandante.gols_total_man),2) AS chutes_por_gol
FROM (
  SELECT time_vis, SUM(chutes_vis) AS chutes_total_vis, SUM(gols_vis) AS gols_total_vis
  FROM `basedosdados.mundo_transfermarkt_competicoes.brasileirao_serie_a`
  WHERE ano_campeonato = 2020
  GROUP BY time_vis
) JOIN (
    SELECT time_man, SUM(chutes_man) AS chutes_total_man, SUM(gols_man) AS gols_total_man
      FROM `basedosdados.mundo_transfermarkt_competicoes.brasileirao_serie_a`
      WHERE ano_campeonato = 2020
      GROUP BY time_man  
  ) mandante ON time_vis = time_man
ORDER BY chutes_por_gol
```
### Resultado
|time_vis     |chutes_total|gols_total|chutes_por_gol|
|-------------|------------|----------|--------------|
|Internacional|450         |61        |7.38          |
|Fluminense   |437         |55        |7.95          |
|Ceará SC     |438         |54        |8.11          |
|Santos FC    |439         |52        |8.44          |
|Flamengo     |577         |68        |8.49          |
|São Paulo    |524         |59        |8.88          |
|Grêmio       |498         |53        |9.4           |
|Atlético-MG  |610         |64        |9.53          |
|Goiás EC     |392         |41        |9.56          |
|Palmeiras    |511         |51        |10.02         |
|Vasco da Gama|388         |37        |10.49         |
|Corinthians  |474         |45        |10.53         |
|EC Bahia     |510         |48        |10.63         |
|Athletico-PR |420         |38        |11.05         |
|Atlético-GO  |443         |40        |11.07         |
|Sport Recife |361         |31        |11.65         |
|RB Bragantino|588         |50        |11.76         |
|Fortaleza    |412         |34        |12.12         |
|Coritiba FC  |388         |31        |12.52         |
|Botafogo     |521         |32        |16.28         |

## Média de quantos chutes a gol um time leva até sofrer um gol
```sql

SELECT time_vis,
    (chutes_total_man + mandante.chutes_total_vis) AS chutes_total,
    (gols_total_man + mandante.gols_total_vis) AS gols_total,
    ROUND((chutes_total_man + mandante.chutes_total_vis) / (gols_total_man + mandante.gols_total_vis),2) AS chutes_por_gol
FROM (
  SELECT time_vis, SUM(chutes_man) AS chutes_total_man, SUM(gols_man) AS gols_total_man
  FROM `basedosdados.mundo_transfermarkt_competicoes.brasileirao_serie_a`
  WHERE ano_campeonato = 2020
  GROUP BY time_vis
) JOIN (
    SELECT time_man, SUM(chutes_vis) AS chutes_total_vis, SUM(gols_vis) AS gols_total_vis
      FROM `basedosdados.mundo_transfermarkt_competicoes.brasileirao_serie_a`
      WHERE ano_campeonato = 2020
      GROUP BY time_man  
  ) mandante ON time_vis = time_man
ORDER BY chutes_por_gol DESC
```
### Resultado
|time_vis     |chutes_total|gols_total|chutes_por_gol|
|-------------|------------|----------|--------------|
|Athletico-PR |473         |36        |13.14         |
|Palmeiras    |440         |37        |11.89         |
|Sport Recife |584         |50        |11.68         |
|Corinthians  |511         |45        |11.36         |
|Internacional|395         |35        |11.29         |
|Atlético-GO  |484         |45        |10.76         |
|Coritiba FC  |574         |54        |10.63         |
|RB Bragantino|410         |40        |10.25         |
|Fluminense   |424         |42        |10.1          |
|Grêmio       |403         |40        |10.07         |
|Fortaleza    |440         |44        |10.0          |
|Santos FC    |486         |51        |9.53          |
|Vasco da Gama|533         |56        |9.52          |
|São Paulo    |381         |41        |9.29          |
|Ceará SC     |465         |51        |9.12          |
|Goiás EC     |557         |63        |8.84          |
|Botafogo     |538         |62        |8.68          |
|EC Bahia     |510         |59        |8.64          |
|Flamengo     |406         |48        |8.46          |
|Atlético-MG  |367         |45        |8.16          |

## Relação por time da quantidade de impedimentos criado e gols sofridos

```sql
/*
Ajuda a compreender se times que costumam deixar o adversário em impedimento sofrem mais ou menos gols
Uma quantidade grande de impedimentos criados pode indicar dois fatores distintos:
  1 - O time tem como estratégia jogar com uma defesa que constantemente cria o impedimento de maneira consciente
  2 - A defesa do time é muito exposta e muitos impedimentos são criados por consequência disso
*/

SELECT time_vis, (impedimentos_criados_vis+man.impedimentos_criados_man) AS impedimentos_criados, (gols_sofridos_vis + man.gols_sofridos_man) AS gols_sofridos
FROM (
  SELECT time_vis, SUM(impedimentos_man) AS impedimentos_criados_vis, SUM(gols_man) AS gols_sofridos_vis
    FROM `basedosdados.mundo_transfermarkt_competicoes.brasileirao_serie_a`
    WHERE ano_campeonato = 2020
    GROUP BY time_vis
    LIMIT 500
) JOIN (
    SELECT time_man, SUM(impedimentos_vis) impedimentos_criados_man, SUM(gols_vis) AS gols_sofridos_man
      FROM `basedosdados.mundo_transfermarkt_competicoes.brasileirao_serie_a`
      WHERE ano_campeonato = 2020
      GROUP BY time_man
      LIMIT 500
) man ON time_vis = man.time_man
ORDER BY impedimentos_criados DESC
```
### Resultado
|time_vis     |impedimentos_criados|gols_sofridos|
|-------------|--------------------|-------------|
|Goiás EC     |85                  |63           |
|Athletico-PR |84                  |36           |
|Corinthians  |80                  |45           |
|RB Bragantino|65                  |40           |
|Flamengo     |62                  |48           |
|Atlético-GO  |61                  |45           |
|Santos FC    |59                  |51           |
|Atlético-MG  |58                  |45           |
|EC Bahia     |58                  |59           |
|Ceará SC     |55                  |51           |
|Fortaleza    |54                  |44           |
|Internacional|51                  |35           |
|Sport Recife |50                  |50           |
|São Paulo    |49                  |41           |
|Vasco da Gama|46                  |56           |
|Grêmio       |40                  |40           |
|Botafogo     |39                  |62           |
|Palmeiras    |38                  |37           |
|Coritiba FC  |37                  |54           |
|Fluminense   |35                  |42           |

## Colocação dos times a cada rodada
```sql
SELECT time_geral, colocacao_geral, rodada
FROM(
  SELECT time_man AS time_geral, colocacao_man AS colocacao_geral, rodada
  FROM `basedosdados.mundo_transfermarkt_competicoes.brasileirao_serie_a`
  WHERE ano_campeonato = 2020
)
UNION DISTINCT
SELECT time_geral, colocacao_geral, rodada
FROM (
  SELECT time_vis AS time_geral, colocacao_vis As colocacao_geral, rodada
  FROM `basedosdados.mundo_transfermarkt_competicoes.brasileirao_serie_a`
  WHERE ano_campeonato = 2020
)
ORDER BY rodada, colocacao_geral

```
### Resultado
|time_geral   |colocacao_geral|rodada|
|-------------|---------------|------|
|São Paulo    |1              |1     |
|Athletico-PR |2              |1     |
|Sport Recife |3              |1     |
|EC Bahia     |4              |1     |
|Grêmio       |5              |1     |
|Internacional|6              |1     |
|Atlético-MG  |7              |1     |
|Santos FC    |8              |1     |
|Palmeiras    |9              |1     |
|Vasco da Gama|10             |1     |
|RB Bragantino|11             |1     |
|Corinthians  |12             |1     |
|Atlético-GO  |13             |1     |
|Ceará SC     |14             |1     |
|Botafogo     |15             |1     |
|Flamengo     |16             |1     |
|Fluminense   |17             |1     |
|Coritiba FC  |18             |1     |
|Fortaleza    |19             |1     |
|Goiás EC     |20             |1     |
|São Paulo    |1              |2     |
|Athletico-PR |2              |2     |
|Sport Recife |3              |2     |
|EC Bahia     |4              |2     |
|Grêmio       |5              |2     |
|Internacional|6              |2     |
|...          |...            |...   |

## Média de gols categorizado pela quantidade de faltas em cada partida
```sql
/*
Hipótese: partidas muito faltosas tendem a ter menos gols
*/

SELECT category, ROUND(AVG(total_gols_partida),2) AS media_gols
FROM (

SELECT total_gols_partida, faltas_total_partida,
  CASE
    WHEN faltas_total_partida >= 40 THEN "40 ou mais faltas"
    WHEN faltas_total_partida >= 30 THEN "30 ou mais faltas"
    WHEN faltas_total_partida >= 20 THEN "20 ou mais faltas"
    WHEN faltas_total_partida < 20 THEN "abaixo de 20"
    ELSE "- sem dados suficientes - "
    END AS category
FROM (

SELECT time_man, time_vis, (gols_man + gols_vis) AS total_gols_partida, (faltas_man + faltas_vis) AS faltas_total_partida,
  FROM `basedosdados.mundo_transfermarkt_competicoes.brasileirao_serie_a`
  WHERE ano_campeonato = 2020
  ORDER BY faltas_total_partida DESC
  LIMIT 500

)) 
GROUP BY category
```
### Resultado
|category                  |media_gols|
|--------------------------|----------|
|40 ou mais faltas         |2.0       |
|30 ou mais faltas         |2.56      |
|20 ou mais faltas         |2.56      |
|abaixo de 20              |2.33      |
|- sem dados suficientes - |2.17      |


## Quantas vezes cada time venceu uma partida de outro time que estava melhor colocado
```sql
/*
Ajuda a compreender times que tivera uma recuperação no campeonato
*/
SELECT ganhador, SUM(pior_ganhou_melhor) AS vitorias_contra_melhor
FROM (
SELECT time_man, time_vis,
CASE
  WHEN colocacao_man > colocacao_vis AND gols_man > gols_vis THEN 1
  ELSE 0
  END AS pior_ganhou_melhor,
CASE
  WHEN gols_man > gols_vis THEN time_man
  WHEN gols_man > gols_vis THEN time_vis
  ELSE "Empate"
  END AS ganhador,
FROM `basedosdados.mundo_transfermarkt_competicoes.brasileirao_serie_a`
WHERE ano_campeonato = 2020
ORDER BY ganhador
)
GROUP BY ganhador
ORDER BY vitorias_contra_melhor DESC
```
### Resultado
|ganhador     |vitorias_contra_melhor|
|-------------|----------------------|
|RB Bragantino|8                     |
|EC Bahia     |6                     |
|Fortaleza    |6                     |
|Corinthians  |5                     |
|Goiás EC     |5                     |
|Vasco da Gama|4                     |
|Santos FC    |4                     |
|Ceará SC     |4                     |
|Coritiba FC  |4                     |
|Sport Recife |3                     |
|Athletico-PR |3                     |
|Fluminense   |3                     |
|Botafogo     |3                     |
|Palmeiras    |3                     |
|Atlético-MG  |2                     |
|Atlético-GO  |2                     |
|São Paulo    |1                     |
|Internacional|1                     |
|Flamengo     |1                     |
|Empate       |0                     |
|Grêmio       |0                     |


## Quantidade de Faltas por árbitro
```sql
SELECT arbitro, 
        SUM(faltas_man + faltas_vis) AS total_faltas, 
        COUNT(arbitro) AS partidas_apitadas,
        ROUND(SUM(faltas_man + faltas_vis) / COUNT(arbitro), 2) AS faltas_por_partida
        
  FROM `basedosdados.mundo_transfermarkt_competicoes.brasileirao_serie_a`
  WHERE ano_campeonato = 2020
  GROUP BY arbitro
  HAVING partidas_apitadas >= 10
  ORDER BY faltas_por_partida DESC
```
### Resultado
|arbitro                       |total_faltas|partidas_apitadas|faltas_por_partida|
|------------------------------|------------|-----------------|------------------|
|Felipe Fernandes              |423         |12               |35.25             |
|Rodolpho Toski Marques        |415         |12               |34.58             |
|Héber Lopes                   |412         |12               |34.33             |
|Rafael Traci                  |473         |14               |33.79             |
|Rodrigo D                     |470         |14               |33.57             |
|Ramon Abatti                  |368         |11               |33.45             |
|Savio Pereira Sampaio         |633         |19               |33.32             |
|Marcelo de Lima Henrique      |463         |14               |33.07             |
|Caio Max Augusto Vieira       |392         |12               |32.67             |
|Anderson Daronco              |543         |17               |31.94             |
|Jean Pierre Goncalves Lima    |383         |12               |31.92             |
|Luiz Flávio de Oliveira       |507         |16               |31.69             |
|Ricardo Marques Ribeiro       |467         |15               |31.13             |
|Bruno Arleu de Araújo         |396         |13               |30.46             |
|Flavio Rodrigues de Souza     |437         |15               |29.13             |
|Leandro Pedro Vuaden          |552         |20               |27.6              |
|Braulio da Silva Machado      |492         |18               |27.33             |
|Wilton Sampaio                |459         |17               |27.0              |
|Edina Alves Batista           |259         |10               |25.9              |
|Raphael Claus                 |476         |19               |25.05             |
|Wagner do Nascimento Magalhães|297         |12               |24.75             |
|Paulo Roberto Alves Júnior    |319         |13               |24.54             |
