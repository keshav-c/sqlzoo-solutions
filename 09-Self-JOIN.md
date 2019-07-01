# Self JOIN

The Edinburgh Buses database consists of two tables: `stops` and `routes`.

### `stops`

This is a list of areas served by buses. The detail does not really include each actual bus stop - just areas within Edinburgh and whole towns near Edinburgh.

Field | Type | Notes
-----|-----|-----
id | INTEGER | Arbitrary value
name | CHAR(30) | The name of an area served by at least one bus

### `route`

A route is the path through town taken by a bus.

Field | Type | Notes
-----|-----|-----
num | CHAR(5) | The number of the bus - as it appears on the front of the vehicle. Oddly these numbers often include letters
company | CHAR(3) | Several bus companies operate in Edinburgh. The main one is Lothian Region Transport - LRT
pos | INTEGER | This indicates the order of the stop within the route. Some routes may revisit a stop. Most buses go in both directions.
stop | INTEGER | This references the stops table

As different companies use numbers arbitrarily the num and the company are both required to identify a route.

1. How many stops are in the database.

        SELECT COUNT(*) FROM stops;

2. Find the id value for the stop 'Craiglockhart'

        SELECT id FROM stops WHERE name = 'Craiglockhart';

3. Give the id and the name for the stops on the '4' 'LRT' service.

        SELECT stops.id, stops.name 
        FROM route 
        JOIN stops ON route.stop = stops.id 
        WHERE num = '4' AND company = 'LRT';

4. Show the number of routes that visit either London Road (149) or Craiglockhart (53). Notice the services that link these stops have a count of 2. Show the services that link both these stops.

        SELECT company, num, COUNT(*)
        FROM route 
        WHERE stop=149 OR stop=53
        GROUP BY company, num
        HAVING COUNT(*) = 2;

5. The self join shown gives all the places you can get to from Craiglockhart (53), without changing routes in the column `b.stop`.

        SELECT a.company, a.num, a.stop, b.stop
        FROM route a JOIN route b ON
            (a.company=b.company AND a.num=b.num)
        WHERE a.stop=53;

    Show the services from Craiglockhart to London Road

        SELECT a.company, a.num, a.stop, b.stop
        FROM (
            SELECT num, company, name, stop 
            FROM route JOIN stops ON id = stop
            ) AS a 
            JOIN (
                SELECT num, company, name, stop 
                FROM route JOIN stops ON id = stop
            ) AS b 
            ON (a.company = b.company AND a.num = b.num) 
        WHERE a.name = 'Craiglockhart' AND 
            b.name = 'London Road';

6. Another way to do the same thing as above, but using cascading joins, not nested select. Find the services between 'Fairmilehead' and 'Tollcross'.

        SELECT a.company, a.num, stopa.name, stopb.name
        FROM route a JOIN route b ON
                (a.company=b.company AND a.num=b.num)
            JOIN stops stopa ON (a.stop=stopa.id)
            JOIN stops stopb ON (b.stop=stopb.id)
        WHERE stopa.name='Fairmilehead' 
            AND stopb.name = 'Tollcross';

7. Give a list of all the services which connect stops 115 and 137 ('Haymarket' and 'Leith')

        SELECT DISTINCT a.company, a.num 
        FROM route a JOIN route b 
            ON (a.num = b.num AND a.company = b.company) 
        WHERE a.stop = 115 AND b.stop = 137;

8. Give a list of the services which connect the stops 'Craiglockhart' and 'Tollcross'.

        SELECT a.company, a.num 
        FROM
            route a
            JOIN route b ON (a.num = b.num AND a.company = b.company)
            JOIN stops stopa ON a.stop = stopa.id 
            JOIN stops stopb ON b.stop = stopb.id 
        WHERE stopa.name = 'Craiglockhart' AND 
            stopb.name = 'Tollcross';

9. Give a distinct list of the stops which may be reached from 'Craiglockhart' by taking one bus, including 'Craiglockhart' itself, offered by the LRT company. Include the company and bus no. of the relevant services.

        SELECT DISTINCT stopb.name, a.company, a.num 
        FROM route a 
            JOIN route b ON (a.num = b.num AND a.company = b.company)
            JOIN stops stopa ON a.stop = stopa.id 
            JOIN stops stopb ON b.stop = stopb.id
        WHERE stopa.name = 'Craiglockhart';

10. Find the routes involving two buses that can go from Craiglockhart to Lochend. Show the bus no. and company for the first bus, the name of the stop for the transfer, and the bus no. and company for the second bus.

    On modifying **8** to find the list of services connecting 'Craiglockhart' and 'Lochend', no matching records are returned.

        SELECT 
            from_craiglockhart.num, from_craiglockhart.company, from_craiglockhart.name, to_lochend.num, to_lochend.company
        FROM
            (SELECT b.num, b.company, stopb.name 
            FROM 
                route a JOIN route b 
                ON a.num = b.num AND 
                    a.company = b.company 
                JOIN stops stopa ON a.stop = stopa.id 
                JOIN stops stopb ON b.stop = stopb.id 
            WHERE stopa.name = 'Craiglockhart'
            ) AS from_craiglockhart
            JOIN
            (SELECT a.num, a.company, stopa.name 
            FROM 
                route a JOIN route b 
                ON a.num = b.num AND 
                    a.company = b.company 
                JOIN stops stopa ON a.stop = stopa.id 
                JOIN stops stopb ON b.stop = stopb.id 
            WHERE stopb.name = 'Lochend'
            ) AS to_lochend
            ON from_craiglockhart.name = to_lochend.name;
