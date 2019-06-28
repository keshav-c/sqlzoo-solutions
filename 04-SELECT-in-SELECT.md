# `SELECT` with `SELECT`

Using `SELECT` statements within `SELECT` statements to perform more complex queries.

Using the table `world`(`name`, `continent`, `area`, `population`, `gdp`)

1. List each country name where the population is larger than that of 'Russia'.

        SELECT name FROM world
        WHERE population > (SELECT population FROM world
            WHERE name='Russia');

2. Show the countries in Europe with a per capita GDP greater than 'United Kingdom'.

        SELECT name FROM world 
        WHERE continent = 'Europe' AND 
        gdp/population > (SELECT gdp/population FROM world 
            WHERE name = 'United Kingdom');

3. List the name and continent of countries in the continents containing either Argentina or Australia. Order by name of the country.

        SELECT name, continent FROM world 
        WHERE continent IN (SELECT continent FROM world 
            WHERE name in ('Argentina', 'Australia')) 
        ORDER BY name;

4. Which country has a population that is more than Canada but less than Poland? Show the name and the population.

        SELECT name, population FROM world WHERE 
        population > (SELECT population FROM world 
            WHERE name = 'Canada') 
        AND population < (SELECT population FROM world 
            WHERE name = 'Poland');

5. Show the name and the population of each country in Europe. Show the population as a percentage of the population of Germany.

        SELECT name, 
            CONCAT(ROUND(population / (SELECT population FROM world     WHERE name = 'Germany') * 100, 0), '%'
            ) AS perc_of_germany 
        FROM world WHERE continent = 'Europe';

_Note: Keyword ALL allows >= or > or < or <= to act over a list_

6. Which countries have a GDP greater than every country in Europe? [Give the name only.] (Some countries may have NULL gdp values)

        SELECT name FROM world 
        WHERE gdp > ALL(SELECT gdp FROM world 
            WHERE continent = 'Europe' AND gdp > 0);

_Note: We can refer to values in the outer SELECT within the inner SELECT. We can name the tables so that we can tell the difference between the inner and outer versions._

7. Find the largest country (by area) in each continent, show the continent, the name and the area

        SELECT continent, name, area 
        FROM world x WHERE area >= ALL 
            (SELECT area FROM world y
            WHERE y.continent=x.continent
            AND area > 0);

_The above example is known as a correlated or synchronized sub-query._

_A correlated subquery works like a nested loop: the subquery only has access to rows related to a single record at a time in the outer query. The technique relies on table aliases to identify two different uses of the same table, one in the outer query and the other in the subquery._

8. List each continent and the name of the country that comes first alphabetically.

        SELECT continent, name FROM world x 
        WHERE name = (SELECT name FROM world y 
            WHERE y.continent = x.continent 
            ORDER BY name 
            LIMIT 1) ;

9. Find the continents where all countries have a population <= 25000000. Then find the names of the countries associated with these continents. Show name, continent and population.

        SELECT name, continent, population FROM world x 
        WHERE 25000000 >= ALL (
            SELECT population FROM world y 
            WHERE y.continent = x.continent 
            AND population > 0);

10. Some countries have populations more than three times that of any of their neighbours (in the same continent). Give the countries and continents.

        SELECT name, continent FROM world x 
        WHERE population > ALL 
            (SELECT population * 3 FROM world y 
            WHERE y.continent = x.continent 
                AND population > 0 
                AND y.name <> x.name
            ); 