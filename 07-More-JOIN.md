# More `JOIN` operations

More exercises using the _movie_ database, which consists of three tables `movie`, `actor` and `casting`. This database features two entities (movies and actors) in a many-to-many relation. Each entity has its own table. A third table, `casting`, is used to link them. The relationship is many-to-many because each film features many actors and each actor has appeared in many films.

The `movie` table

Field name | Type | Notes
--------|--------|--------
id | INTEGER | An arbitrary unique identifier
title | CHAR(70) | The name of the film - usually in the language of the first release.
yr | DECIMAL(4) | Year of first release.
director | INT | A reference to the actor table.
budget | INTEGER | How much the movie cost to make (in a variety of currencies unfortunately).
gross | INTEGER | How much the movie made at the box office.

The `actor` table

Field name | Type | Notes
--------|--------|--------
id | INTEGER | An arbitrary unique identifier
name | CHAR(36) | The name of the actor (the term actor is used to refer to both male and female thesps.)

The `casting` table

Field name | Type | Notes
--------|--------|--------
movieid | INTEGER | A reference to the movie table.
actorid | INTEGER | A reference to the actor table.
ord | INTEGER | The ordinal position of the actor in the cast list. The star of the movie will have ord value 1 the co-star will have value 2, ...

_Note: The actor table actually refers to the director of the film. The actor.id refers to the movie.director field_

1. List the films where the yr is 1962 [Show `id`, `title`]

        SELECT id, title FROM movie WHERE yr=1962;

2. Give year of release for 'Citizen Kane'.

        SELECT yr FROM movie WHERE title = 'Citizen Kane';

3. List all of the Star Trek movies, include the `id`, `title` and `yr` (all of these movies include the words Star Trek in the title). Order results by year.

        SELECT id, title, yr FROM movie 
        WHERE title LIKE '%Star Trek%' ORDER BY yr;

4. What id number does the actor 'Glenn Close' have?

        SELECT id FROM actor WHERE name = 'Glenn Close';

5. What is the id of the film 'Casablanca'?

        SELECT id FROM movie WHERE title = 'Casablanca';

6. Obtain the cast list for 'Casablanca'.

    - First using nested SELECT statements

            SELECT actor.name FROM actor WHERE 
                actor.id IN (SELECT casting.actorid 
                    FROM casting 
                    WHERE casting.movieid = (SELECT movie.id 
                        FROM movie 
                        WHERE movie.title = 'Casablanca'
                    )
                );

    - Using JOINs

            SELECT actor.name 
            FROM movie JOIN casting ON movie.id = casting.movieid       JOIN actor ON actorid = actor.id 
            WHERE movie.title = 'Casablanca';

7. Obtain the cast list for the film 'Alien'.

        SELECT actor.name 
        FROM movie JOIN casting ON movie.id = casting.movieid 
            JOIN actor ON actorid = actor.id 
        WHERE movie.title = 'Alien';

8. List the films in which 'Harrison Ford' has appeared.

        SELECT movie.title FROM
        actor JOIN casting on actor.id = casting.actorid 
            JOIN movie ON casting.movieid = movie.id
        WHERE actor.name = 'Harrison Ford';

9. List the films where 'Harrison Ford' has appeared - but not in the starring role. [_Note: the `ord` field of casting gives the position of the actor. If `ord = 1` then this actor is in the starring role_]

        SELECT movie.title FROM 
        actor JOIN casting on actor.id = casting.actorid 
            JOIN movie ON casting.movieid = movie.id
        WHERE actor.name = 'Harrison Ford' 
            AND casting.ord > 1;

10. List the films together with the leading star for all 1962 films.

    - Solution with nested SELECT and JOIN

        SELECT movie.title, starring_actor.name 
        FROM (SELECT 
                casting.movieid, actor.name 
                FROM actor JOIN casting 
                    ON actor.id = casting.actorid 
                WHERE casting.ord = 1
            ) AS starring_actor 
            JOIN movie 
            ON starring_actor.movieid = movie.id 
        WHERE movie.yr = 1962;

11. Which were the busiest years for 'John Travolta', show the year and the number of movies he made each year for any year in which he made more than 2 movies.

    - Default solution

            SELECT yr,COUNT(title) FROM
            movie JOIN casting ON movie.id=movieid
                JOIN actor   ON actorid=actor.id
            WHERE name = 'John Travolta'
            GROUP BY yr
            HAVING 
                COUNT(title) = (
                    SELECT MAX(c) FROM
                        (SELECT yr,COUNT(title) AS c 
                        FROM movie JOIN casting ON movie.id = movieid
                            JOIN actor ON actorid = actor.id
                        WHERE name = 'John Travolta'
                        GROUP BY yr) AS t
                )

    - My Solution

            SELECT movie.yr, COUNT(movie.title) 
            FROM movie JOIN casting ON movie.id = casting.movieid
                JOIN actor ON actor.id = casting.actorid 
            WHERE actor.name = 'John Travolta' 
            GROUP BY movie.yr 
            HAVING COUNT(movie.title) > 2;

12. List the film title and the leading actor for all of the films 'Julie Andrews' played in.

        SELECT movie.title, actor.name 
        FROM 
            actor JOIN casting ON casting.actorid = actor.id 
                JOIN movie ON movie.id = casting.movieid 
        WHERE 
            movie.id IN 
                (SELECT movie.id 
                FROM actor JOIN casting ON actor.id = casting.actorid
                    JOIN movie ON movie.id = casting.movieid 
                WHERE actor.name = 'Julie Andrews'
                ) 
            AND casting.ord = 1;

13. Obtain a list, in alphabetical order, of actors who've had at least 30 starring roles.

        SELECT actor.name 
        FROM actor JOIN casting ON actor.id = casting.actorid 
        WHERE casting.ord = 1 
        GROUP by actor.name 
        HAVING COUNT(casting.ord) >= 30 
        ORDER BY actor.name;

14. List the films released in the year 1978 ordered by the number of actors in the cast, then by title.

        SELECT movie.title, crew_count.total_cast 
        FROM 
            movie JOIN (
                SELECT casting.movieid, 
                    COUNT(casting.actorid) AS total_cast 
                FROM casting 
                GROUP BY casting.movieid
            ) AS crew_count 
            ON movie.id = crew_count.movieid 
        WHERE movie.yr = 1978 
        ORDER BY crew_count.total_cast DESC, movie.title;

15. List all the people who have worked with 'Art Garfunkel'.

        SELECT actor.name 
        FROM actor JOIN casting ON actor.id = casting.actorid 
        WHERE 
            casting.movieid IN (
                SELECT casting.movieid 
                FROM casting JOIN actor ON casting.actorid = actor.id
                WHERE actor.name = 'Art Garfunkel'
            ) 
            AND actor.name <> 'Art Garfunkel';