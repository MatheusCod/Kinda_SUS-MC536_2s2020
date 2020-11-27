LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/MatheusCod/Kinda_SUS-MC536_2s2020/main/stage04/CSVs/treated_global_obesity.csv' AS line
CREATE (:Country {name: line.Country, data: line.BothSexes})

MATCH (us:Country {name: "United States of America"})
MATCH (similar:Country)
WHERE toInteger(similar.data) >= toInteger(us.data) - 10 and similar.name <> "United States of America" 
CREATE (similar)-[:Relates]->(us)

MATCH (japan:Country {name: "Japan"})
MATCH (similar:Country)
WHERE toInteger(similar.data) <= toInteger(japan.data) + 5 and similar.name <> "Japan"
CREATE (similar)-[:Relates]->(japan)

***

LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/MatheusCod/Kinda_SUS-MC536_2s2020/main/stage04/CSVs/consumo_alimentos_etapa4.csv' AS line
CREATE (:CountryFood {name: line.countryname, Fruits: line.Fruits, Vegetables: line.Vegetables, BeansAndLegumes: line.BeansAndLegumes, UnprocessedRedMeats: line.UnprocessedRedMeats, SugarSweetenedBeverages: line.SugarSweetenedBeverages})

CREATE INDEX ON :Country(name)
CREATE INDEX ON :CountryFood(name)

MATCH (us:CountryFood {name: "United States of America"})
MATCH (similar:CountryFood)
WHERE toInteger(similar.UnprocessedRedMeats) >= toInteger(us.UnprocessedRedMeats) - 10 and toInteger(similar.UnprocessedRedMeats) <= toInteger(us.UnprocessedRedMeats) + 10 and similar.name <> "United States of America" 
CREATE (similar)-[:MeatDiet]->(us)

MATCH (us:CountryFood {name: "United States of America"})
MATCH (similar:CountryFood)
WHERE toInteger(similar.SugarSweetenedBeverages) >= toInteger(us.SugarSweetenedBeverages) - 20 and similar.name <> "United States of America" 
CREATE (similar)-[:Sugar]->(us)

MATCH (c1)-[:Relates]->(us:Country {name:"United States of America"})
MATCH (c2)-[:Sugar]->(us2:CountryFood {name:"United States of America"})
MATCH (c3)-[:MeatDiet]->(us2:CountryFood {name:"United States of America"})
WHERE (c1.name = c2.name) OR (c1.name = c3.name)
RETURN c2, c3

***

MATCH (c1:Country)
MATCH (c2:Country)
WHERE toInteger(c1.data) >= toInteger(c2.data) - 1 AND toInteger(c1.data) <= toInteger(c2.data) + 1 AND c1.name <> c2.name
MERGE (c1)<-[d:Relates]->(c2)

MATCH (c1:CountryFood)
MATCH (c2:CountryFood)
WHERE toInteger(c1.UnprocessedRedMeats) >= toInteger(c2.UnprocessedRedMeats) - 1 and toInteger(c1.UnprocessedRedMeats) <= toInteger(c2.UnprocessedRedMeats) + 1 and c1.name <> c2.name
MERGE (c1)<-[:MeatDiet]->(c2)

MATCH (n:CountryFood)
MATCH (c1)-[:Relates]->(c2:Country)
RETURN c1, c2, n
