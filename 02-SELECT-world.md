# SELECT from `world` table

More SELECT exercises

1. Show the `name`, `continent` and `population` of all countries.

        SELECT name, continent, population FROM world;

2. Show the name for the countries that have a population of at least 200 million.

        SELECT name FROM world WHERE population >= 200000000;

3. Give the `name` and the `per capita GDP` for those countries with a population of at least 200 million.      

        SELECT name, gdp/population AS per_capita_gdp FROM world
        WHERE population >= 200000000;

4. Show the name and population in millions for the countries of the continent 'South America'.

        SELECT name, population/1000000 FROM world 
        WHERE continent = 'South America';

5. Show the name and population for France, Germany, Italy.

        SELECT name, population FROM world 
        WHERE name IN ('France', 'Germany', 'Italy');

6. Show the countries which have a name that includes the word 'United'

        SELECT name FROM world WHERE name LIKE '%United%';

_Two ways to be big: A country is big if it has an area of more than 3 million sq km or it has a population of more than 250 million._

7. Show the countries that are big by area or big by population. Show name, population and area.

        SELECT name, population, area  FROM world 
        WHERE area > 3000000 OR population > 250000000;

_Note: XOR One or the other (but not both)_

8. Show the countries that are big by area or big by population but not both. Show name, population and area.

        SELECT name, population, area FROM world 
        WHERE area > 3000000 XOR population > 250000000;

_Note: `ROUND(f,p)` returns `f` rounded to `p` decimal places._

9. Show the name and population in millions and the GDP in billions for the countries of the continent 'South America'.

        SELECT name, 
            ROUND(population/1000000, 2) AS pop_in_millions, 
            ROUND(gdp/1000000000, 2) AS gdp_in_billions 
        FROM world WHERE continent = 'South America';

10. Show the name and per-capita GDP for those countries with a GDP of at least one trillion. Round this value to the nearest 1000.

        SELECT name, ROUND(gdp/population, -3) FROM world 
        WHERE gdp >= 1000000000000;

_Note: `LENGTH(s)` returns the number of characters in string `s`._

11. Show the name and capital where the name and the capital have the same number of characters.

        SELECT name, capital FROM world
        WHERE LENGTH(name) = LENGTH(capital);

_Note: `LEFT(s,n)` allows you to extract `n` characters from the start of the string `s`_

_Note: `<>` is the NOT EQUALS operator_

12. Show the name and the capital where the first letters of each match. Don't include countries where the name and the capital are the same word.

        SELECT name, capital FROM world
        WHERE LEFT(name, 1) = LEFT(capital, 1) AND name <> capital;

13. Find the country that has all the vowels and no spaces in its name.

        SELECT name FROM world
        WHERE (name LIKE '%A%' OR name LIKE '%a%') AND
            (name LIKE '%E%' OR name LIKE '%e%') AND
            (name LIKE '%I%' OR name LIKE '%i%') AND
            (name LIKE '%O%' OR name LIKE '%o%') AND
            (name LIKE '%U%' OR name LIKE '%u%') AND
            (name NOT LIKE '% %');