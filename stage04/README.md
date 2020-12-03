Estrutura de pastas:

~~~
├── README.md  <- arquivo apresentando a proposta
│
├── data
│   ├── external       <- dados de terceiros em formato usado para entrada na transformação
│   ├── interim        <- dados intermediários, e.g., resultado de transformação
│   ├── processed      <- dados finais usados para a modelagem
│   └── raw            <- dados originais sem modificações
│
├── notebooks          <- Jupyter notebooks ou equivalentes
│
├── slides             <- arquivo de slides em formato PDF
│
├── src                <- fonte em linguagem de programação ou sistema (e.g., Cytoscape)
│   └── README.md      <- instruções básicas de instalação/execução
│
└── assets             <- mídias usadas no projeto
~~~

# Etapa 04 - Análises com o Segundo Modelo Lógico

* Jupyter: <br>
[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/MatheusCod/Kinda_SUS-MC536_2s2020/main)

## Slides da Apresentação da Etapa

> Coloque um link para o arquivo dos slides da apresentação que estão na pasta `slides`.

## Modelo Conceitual Atualizado

![Modelos Conceituais](./assets/modelos_conceituais.png)

## Modelos Lógicos Atualizados

> Coloque aqui os dois modelos lógicos dos bancos de dados relacionados aos modelos conceituais. O modelo lógico da etapa anterior pode ser copiado ou apresentado revisado. Para o modelo relacional, sugere-se o formato a seguir. Para outros modelos lógicos o formato é livre, pode ser adotado aqueles apresentados em sala.

> Exemplo de modelo lógico relacional
~~~
Análise obesidade(LocationAbbr,  Mortalidade_homens, Mortalidade_mulheres, Obesidade_homens, Obesidade_mulheres)

Consumo de alimentos(Gênero, Idade, Carne vermelha, Frutas, Vegetais, Proteína, Bebidas açucaradas)

Disponibilidade(Açúcares, Frutas, Carne Vermelha, Vegetais)

Obesidade e Nutrição Global(Country, Obesity, Fruit, Non-starchy vegetables, Unprocessed red meats, Total protein, Sugar-sweetened beverages)
~~~

## Programa de extração e conversão de dados atualizado

> Coloque um link para o arquivo do notebook que executa a extração e conversão de dados. Ele estará dentro da pasta `notebook`. Se por alguma razão o código não for executável no Jupyter, coloque na pasta `src`. Se a extração e conversão envolverem queries executadas atraves de uma interface de um SGBD não executável no Jupyter, como o Cypher, apresente na forma de markdown. <br>
[Extracao Consumo Alimentos](./notebooks/extracaoConsumoAlimentosipynb) <br>
[Extracao Global Obesity](./notebooks/extracaoGlobalObesity.ipynb)

## Conjunto de queries de dois modelos

> Acrescente um link para o arquivo do notebook que executa o segundo conjunto de queries. Ele estará dentro da pasta `notebook`. Se por alguma razão o código não for executável no Jupyter, coloque na pasta `src`. Se as queries forem executadas atraves de uma interface de um SGBD não executável no Jupyter, como o Cypher, apresente na forma de markdown.
> O link para queries da etapa 3 também deve aparecer aqui e as queries poderão ser revisadas. <br>
### Grupo 1
Nesse primeiro conjunto de _queries_ são agrupados países que possuem uma taxa de obesidade semelhante aos EUA, país alvo da nossa análise, e ao Japão, um dos países
com menor taxa de obesidade.

>LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/MatheusCod/Kinda_SUS-MC536_2s2020/main/stage04/data/processed/treated_global_obesity.csv' AS line  
CREATE (:Country {name: line.Country, data: line.BothSexes})

>CREATE INDEX ON :Country(name)

>MATCH (us:Country {name: "United States of America"})  
MATCH (similar:Country)  
WHERE toInteger(similar.data) >= toInteger(us.data) - 10 and similar.name <> "United States of America"   
CREATE (similar)-[:Relates]->(us)  

>MATCH (japan:Country {name: "Japan"})  
MATCH (similar:Country)  
WHERE toInteger(similar.data) <= toInteger(japan.data) + 5 and similar.name <> "Japan"  
CREATE (similar)-[:Relates]->(japan)  

***

### Grupo 2
Aqui foram agrupados os países que possuem um consumo de carne vermelha e bebidas açucaradas semelhantes aos EUA, com o objetivo de, em conjunto com as _queries_ do grupo 1, visualizar a possível correlação entre os dois grupos.

>LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/MatheusCod/Kinda_SUS-MC536_2s2020/main/stage04/data/processed/consumo_alimentos_etapa4.csv' AS line  
CREATE (:CountryFood {name: line.countryname, Fruits: line.Fruits, Vegetables: line.Vegetables, BeansAndLegumes: line.BeansAndLegumes, UnprocessedRedMeats: line.UnprocessedRedMeats, SugarSweetenedBeverages: line.SugarSweetenedBeverages})  

>CREATE INDEX ON :CountryFood(name)  

>MATCH (us:CountryFood {name: "United States of America"})  
MATCH (similar:CountryFood)  
WHERE toInteger(similar.UnprocessedRedMeats) >= toInteger(us.UnprocessedRedMeats) - 10 and toInteger(similar.UnprocessedRedMeats) <= toInteger(us.UnprocessedRedMeats) + 10 and similar.name <> "United States of America"   
CREATE (similar)-[:MeatDiet]->(us)  

>MATCH (us:CountryFood {name: "United States of America"})  
MATCH (similar:CountryFood)  
WHERE toInteger(similar.SugarSweetenedBeverages) >= toInteger(us.SugarSweetenedBeverages) - 20 and similar.name <> "United States of America"   
CREATE (similar)-[:Sugar]->(us)  

>MATCH (c1)-[:Relates]->(us:Country {name:"United States of America"})  
MATCH (c2)-[:Sugar]->(us2:CountryFood {name:"United States of America"})  
MATCH (c3)-[:MeatDiet]->(us2:CountryFood {name:"United States of America"})  
WHERE (c1.name = c2.name) OR (c1.name = c3.name)  
RETURN c2, c3  

***

### Grupo 3
Neste grupo de _queries_ foram criados um grafo que relaciona os países que possuem taxa de obesidade semelhante entre si e outro que agrupa os países com consumo de carne vermelha parecido. Espera-se criar com isso uma visualização mais geral entre esses grupos

>MATCH (c1:Country)  
MATCH (c2:Country)  
WHERE toInteger(c1.data) >= toInteger(c2.data) - 1 AND toInteger(c1.data) <= toInteger(c2.data) + 1 AND c1.name <> c2.name  
MERGE (c1)<-[d:Relates]->(c2)  

>MATCH (c1:CountryFood)  
MATCH (c2:CountryFood)  
WHERE toInteger(c1.UnprocessedRedMeats) >= toInteger(c2.UnprocessedRedMeats) - 1 and toInteger(c1.UnprocessedRedMeats) <= toInteger(c2.UnprocessedRedMeats) + 1 and c1.name <> c2.name  
MERGE (c1)<-[:MeatDiet]->(c2)  

>MATCH (n:CountryFood)  
MATCH (c1)-[:Relates]->(c2:Country)  
RETURN c1, c2, n  

***
### Grupo 4
Para este grupo foi aplicado o conceito de community para agrupar os países com taxa de obesidade semelhante, baseado nas relações criadas no grupo 3.

>CALL gds.graph.create(  
    'communityGraph',  
    'Country',  
    {  
        Relates: {  
            orientation: 'UNDIRECTED'  
            }        
    }   
)

>CALL gds.louvain.stream('communityGraph')  
YIELD nodeId, communityId  
RETURN gds.util.asNode(nodeId).name AS name, communityId  
ORDER BY communityId ASC

>CALL gds.louvain.stream('communityGraph')  
YIELD nodeId, communityId  
RETURN gds.util.asNode(nodeId).name AS name, communityId  
#### Stage 03
[Análise Obesidade](../stage03/notebook/analiseObesidade.ipynb)

## Bases de Dados
> Elencar as bases de dados utilizadas no projeto. Trata-se de uma atualização daquelas apresentadas na Etapa 3.

título da base | link | breve descrição
----- | ----- | -----
Obesity | http://dbpedia.org/page/Obesity | Página da DBpedia com informações úteis e definições sobre Obesidade.
Obesity Stats | https://www.kaggle.com/adu47249/obesity-stats | Estatística sobre peso nos Estados Unidos do ano 2011 até 2016.
Heart Disease Mortality | https://catalog.data.gov/dataset/heart-disease-mortality-data-among-us-adults-35-by-state-territory-and-county-d31fc/resource/5974720b-7972-4272-8eb1-277dfdc538c2 | Mortalidade por doenças do coração em adultos (+35 anos) nos EUA.
Prevalence of obesity among adults | https://apps.who.int/gho/data/node.main.BMI30C?lang=en | Prevalência de obesidade entre adultos, BMI ≥ 30, estimativas brutas por país.
Food Availability | https://www.ers.usda.gov/data-products/food-availability-per-capita-data-system/ | Disponibilidade de comida per capita nos EUA.
Global Dietary Database | https://www.globaldietarydatabase.org/ | Consumo de grupos de alimentos em diversos países, separado por faixa etária, nível de educação, entre outros.

## Arquivos de Dados
> Elencar os arquivos usados no projeto que estão disponíveis no Github do projeto (manter os da Etapa 3 e acrescentar os da Etapa 4).

nome do arquivo | link | breve descrição
----- | ----- | -----
all_cnty_ac_yr_2015.csv | https://github.com/MatheusCod/Kinda_SUS-MC536_2s2020/blob/main/stage04/data/interim/all_cnty_ac_yr_2015.csv | Filtro do dataset do Global Diatery Data para o ano de 2015.
NCD_BMI_30C.csv | https://github.com/MatheusCod/Kinda_SUS-MC536_2s2020/blob/main/stage04/data/processed/NCD_BMI_30C.csv | Taxa de obesidade global.
consumo_alimentos_etapa4.csv | https://github.com/MatheusCod/Kinda_SUS-MC536_2s2020/blob/main/stage04/data/processed/consumo_alimentos_etapa4.csv | Consumo de alimentos global
global_obesity.csv | https://github.com/MatheusCod/Kinda_SUS-MC536_2s2020/blob/main/stage04/data/processed/global_obesity.csv | descrição
treated_global_obesity.csv  | https://github.com/MatheusCod/Kinda_SUS-MC536_2s2020/blob/main/stage04/data/processed/treated_global_obesity.csv | descrição 
GDD 2015 Codebook_Feb 3 2020.xlsx | https://github.com/MatheusCod/Kinda_SUS-MC536_2s2020/blob/main/stage04/data/raw/GDD%202015%20Codebook_Feb%203%202020.xlsx | descrição
NCD_BMI_30C.csv  | https://github.com/MatheusCod/Kinda_SUS-MC536_2s2020/blob/main/stage04/data/raw/NCD_BMI_30C.csv | descrição
HeartDisease.csv | https://github.com/MatheusCod/Kinda_SUS-MC536_2s2020/blob/main/stage03/CSVs/HeartDisease.csv | Mortalidade por Doenças de Coração
ObesityStats_2013.csv | https://github.com/MatheusCod/Kinda_SUS-MC536_2s2020/blob/main/stage03/CSVs/ObesityStats_2013.csv | Dados sobre Obesidade nos EUA - para o ano de 2013
ObesityStats.csv | https://github.com/MatheusCod/Kinda_SUS-MC536_2s2020/blob/main/stage03/CSVs/ObesityStats.csv | Dados sobre Obesidade nos EUA 
