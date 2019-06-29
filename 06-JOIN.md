# The `JOIN` operation

`JOIN` allows you to use data from two or more tables. The tables contain all matches and goals from UEFA EURO 2012 Football Championship in Poland and Ukraine.

- `game` table: `id`, `mdate`, `stadium`, `team1`, and `team2`
- `goal` table: `matchid`, `teamid`, `player`, and `gtime`
- `eteam` table: `id`, `teamname`, and `coach`

1. Show the `matchid` and `player` name for all goals scored by Germany. To identify German players, check for: `teamid = 'GER'`

        SELECT matchid, player FROM goal WHERE teamid = 'GER';

2. Show `id`, `stadium`, `team1`, `team2` for just game `1012`

        SELECT id,stadium,team1,team2 FROM game WHERE id = 1012;

3. Show the `player`, `teamid`, `stadium` and `mdate` for every German goal.

        SELECT goal.player, goal.teamid, game.stadium, game.mdate 
        FROM game JOIN goal ON (game.id = goal.matchid) 
        WHERE goal.teamid = 'GER';

4. Show the `team1`, `team2` and `player` for every goal scored by a player called Mario

        SELECT game.team1, game.team2, goal.player 
        FROM game JOIN goal ON (goal.matchid = game.id) 
        WHERE goal.player LIKE 'Mario%';

5. Show `player`, `teamid`, `coach`, `gtime` for all goals scored in the first 10 minutes.

        SELECT goal.player, goal.teamid, eteam.coach, goal.gtime
        FROM goal JOIN eteam ON (goal.teamid = eteam.id)
        WHERE gtime <= 10;

6. List the the dates of the matches and the name of the team in which 'Fernando Santos' was the team1 coach.

        SELECT game.mdate, eteam.teamname 
        FROM game JOIN eteam ON (game.team1 = eteam.id) 
        WHERE eteam.coach = 'Fernando Santos';

7. List the player for every goal scored in a game where the stadium was 'National Stadium, Warsaw'

        SELECT goal.player 
        FROM goal JOIN game ON (goal.matchid = game.id) 
        WHERE game.stadium = 'National Stadium, Warsaw';

8. Show the name of all players who scored a goal against Germany.

        SELECT DISTINCT goal.player
        FROM goal JOIN game ON (goal.matchid = game.id) 
        WHERE (game.team1 = 'GER' OR game.team2 = 'GER') 
            AND goal.teamid <> 'GER';

9. Show teamname and the total number of goals scored.

        SELECT eteam.teamname, COUNT(*) AS goals
        FROM eteam JOIN goal ON (eteam.id = goal.teamid)
        GROUP BY teamname;

10. Show the stadium and the number of goals scored in each stadium.

        SELECT game.stadium, COUNT(*) AS goals
        FROM game JOIN goal ON (game.id = goal.matchid)
        GROUP BY stadium;

11. For every match involving 'POL', show the matchid, date and the number of goals scored.

        SELECT goal.matchid, game.mdate, COUNT(*) AS goals
        FROM game JOIN goal ON goal.matchid = game.id 
        WHERE (team1 = 'POL' OR team2 = 'POL') 
        GROUP BY goal.matchid, game.mdate;

12. For every match where 'GER' scored, show matchid, match date and the number of goals scored by 'GER'

        SELECT game.id, game.mdate, COUNT(*) AS goals 
        FROM game JOIN goal ON (game.id = goal.matchid) 
        WHERE goal.teamid = 'GER' 
        GROUP BY game.id, game.mdate;

13. List every match with the goals scored by each team as shown.

        SELECT 
            game.mdate, 
            game.team1, 
            SUM(CASE WHEN goal.teamid = game.team1 THEN 1 ELSE 0 END) AS score1,
            game.team2, 
            SUM(CASE WHEN goal.teamid = game.team2 THEN 1 ELSE 0 END) AS score2
        FROM game LEFT JOIN goal ON game.id = goal.matchid
        GROUP BY game.mdate, game.team1, game.team2
        ORDER BY game.mdate, game.id, game.team1, game.team2;

In the above solution we use a `LEFT JOIN` because we also want those rows from the `game` table, where there is no corresponding record in the `goal` table, i.e the final score was _0-0_.

## The Music Database

- album ( asin, title, artist, price, release, label, rank )
- track ( album, dsk, posn, song )

The phrase `FROM album JOIN track ON album.asin = track.album` represents the join of the tables album and track. This JOIN has one row for every track. In addition to the track fields (`album`, `disk`, `posn` and `song`) it includes the details of the corresponding album (`title`, `artist` etc.).

1. Find the title and artist who recorded the song 'Alison'.

        SELECT album.title, album.artist
        FROM album JOIN track ON (album.asin = track.album)
        WHERE track.song = 'Alison';

2. Which artist recorded the song 'Exodus'?

        SELECT album.artist 
        FROM album JOIN track ON album.asin = track.album 
        WHERE song = 'Exodus';

3. Show the song for each track on the album 'Blur'

        SELECT track.song 
        FROM track JOIN album ON track.album = album.asin 
        WHERE album.title = 'Blur';

4. For each album show the title and the total number of track.

        SELECT album.title, COUNT(*)
        FROM album JOIN track ON (album.asin = track.album)
        GROUP BY album.title;

5. For each album show the title and the total number of tracks containing the word 'Heart'.

        SELECT album.title, COUNT(*)
        FROM album JOIN track ON (album.asin = track.album)
        WHERE track.song LIKE '%Heart%'
        GROUP BY album.title;

6. A "title track" is where the song is the same as the title. Find the title tracks.

        SELECT track.song AS title_track 
        FROM track JOIN album ON track.album = album.asin 
        WHERE track.song = album.title;

7. An "eponymous" album is one where the title is the same as the artist (for example the album 'Blur' by the band 'Blur'). Show the eponymous albums.

        SELECT title AS eponymous_album FROM album 
        WHERE title = artist;

8. Find the songs that appear on more than 2 albums. Include a count of the number of times each shows up.

        SELECT track.song, COUNT(DISTINCT album.title) 
        FROM track JOIN album ON track.album = album.asin 
        GROUP BY track.song 
        HAVING COUNT(DISTINCT album.title) > 2;

9. A "good value" album is one where the price per track is less than 50 pence. Find the good value album - show the title, the price and the number of tracks.

        SELECT album.title, album.price, COUNT(track.song) 
        FROM album JOIN track ON album.asin = track.album 
        GROUP BY album.title, album.price 
        HAVING album.price / COUNT(track.song) < 0.50;

10. List albums so that the album with the most tracks is first. Show the title and the number of tracks. Where two or more albums have the same number of tracks you should order alphabetically.

        SELECT album.title, COUNT(track.song) 
        FROM album JOIN track ON album.asin = track.album 
        GROUP BY album.title 
        ORDER BY COUNT(track.song) DESC, album.title;