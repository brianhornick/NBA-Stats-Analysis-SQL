# Analyzing NBA Data with SQL - The Three-Pointer and its Effect on the Game - by Brian Hornick
### Introduction
Anyone who knows anything about basketball will know that the NBA has seen a huge increase in 3-point shots both made and attempted in the last decade. However, just how much has the game changed, and how important is the three-ball really for winning games? To investigate this, we will be using a large dataset containing comprehensive game stats, player stats, and team stats for every game from the 2013-14 season up until the 2022-23 season. This dataset was downloaded from Kaggle.com. [NBA Boxscore Dataset](https://www.kaggle.com/datasets/lukedip/nba-boxscore-dataset) SQLite will be the tool to investigate this dataset and the goal is to answer each question using only 1 query.

### Questions

What was the average number of 3s taken per game by a team in the 2013-2014 vs the 2022-2023 season?

What is the year-over-year change in average 3s taken per game by a team?

Which teams has taken the most 3s across the most recent 3 years in this dataset? What is their overall winning percentage compared to the teams with the least amount of 3s taken?

Do more 3-Point Attempts impact winning when adjusted for 3-Point Percentage?

What results in more wins?  Outrebounding opponents vs shooting a higher 3-Point %


### Question 1 - Average Number of 3s: 2013/14 vs 2022/23

This first question allows us to get a first, general look at how much the game has changed over the last decade. The problem seems straightforward, however, it does require a few functions to calculate. The table where the 3PA (3-point attempts) are outlined is held in the team_stats table, where the season each game is played is held in the game_info table. Therefore we must join the tables on their matching "game_id" column to find 3-point attempts filtered by specific seasons. The AVG function is used, of course, to find the average number of 3s taken. ROUND and AS functions are used to make the output a little cleaner and the FILTER function is used to filter both the outputs by the respective seasons being compared.

```
SELECT 
ROUND(AVG([3PA]) FILTER (WHERE season = 2223),2) AS '2223_3pa_avg',
ROUND(AVG([3PA]) FILTER (WHERE season = 1314),2) AS '1314_3pa_avg'
FROM team_stats ts
  JOIN game_info gi
  ON ts.game_id = gi.game_id;
```
Upon executing the code, the output looks like this:

![image_alt](https://github.com/brianhornick/NBA-Stats-Analysis-SQL/blob/main/Images/Screenshot%202025-02-17%20133036.png?raw=true)

From this output, we can see a huge difference in the number of 3pa for the 2022-23 season vs the 2013-14 season, almost a 60% increase. 

### Question 2 - Year-over-year change in average 3s taken per game

We've seen a glaring difference in the amount of 3s taken per game, comparing the 2013-14 vs the 2022-23 seasons, however, we don't know if this was a gradual change or if there were there certain seasons where there were massive spikes. By looking at the year-over-year change difference in average 3s taken per game, we can get the answer to this question.

To find this, first, we create a CTE (Common Table Expression) which establishes a temporary table that takes the season and average 3PA per game, per team. A similar method is used as the first problem, obtaining our averages using the AVG function, rounding our answers to 2 decimal places with the ROUND function, using JOIN to get the season variable, and in this case, using GROUP BY to get the average 3PA per each season. Then, from this temporary table, we query the season again and then take the average 3PA per season subtracted by the previous year (using the LAG function) to get our year-over-year change in 3PA per game.
```
WITH avg_3pa_calc AS (
SELECT season, ROUND(AVG([3PA]),2) AS avg_3pa
FROM team_stats ts
  JOIN game_info gi
  ON ts.game_id = gi.game_id
GROUP BY season)

SELECT season, avg_3pa - LAG(avg_3pa, 1) OVER (ORDER BY season) AS YOY_3pa_change
FROM avg_3pa_calc;
```
Here is what this code executes:

![image_alt](https://github.com/brianhornick/NBA-Stats-Analysis-SQL/blob/main/Images/Screenshot%202025-02-19%20155000.png?raw=true)

As we can see the largest increase in 3PA happened between the 2016-2017 - 2019-2020 season where there was a combined total cumulative increase of over 10 3s taken a game! 
2022-23 was actually the first season in the last 10 years to see a decrease in 3s taken per game.

### Question 3 - The Winning Percentage of High vs Low 3-Point Volume Teams

In this section, we aim to take the winning % of the top 5 teams with the highest volume of 3s attempted and compare it to the winning % of the teams with the least amount of 3's attempted. This analysis will specifically look at the most recent 3 seasons. 

```
WITH three_ranker AS (
SELECT team, ROW_NUMBER () OVER (ORDER BY SUM([3PA]) DESC) AS rank,
    SUM(CASE WHEN team = [home_team] AND [result] = 1 THEN 1
        WHEN team = [away_team] AND result = 0 THEN 1
        ELSE 0 END) AS total_wins,
    SUM(CASE WHEN team = [home_team] OR team = [away_team] THEN 1 ELSE 0 END) AS total_games_played
FROM team_stats ts
  JOIN game_info gi
  ON ts.game_id = gi.game_id
WHERE season = 2223 OR season = 2122 OR season = 2021
GROUP BY team)

SELECT ROUND(SUM(total_wins) FILTER (WHERE [rank] < 6) * 100.0 / (total_games_played *5),2) AS top5_winperc,
       ROUND(SUM(total_wins) FILTER (WHERE [rank] > 25) * 100.0 / (total_games_played *5),2) AS bottom5_winperc
FROM three_ranker;
```
First, we select our teams and then use a ROW_NUMBER window function to create a ranking that ranks all the teams by total 3 pointers attempted. Finally, 2 more columns are needed, one that totals wins using the SUM and CASE functions and another that uses the same functions to sum the total games played for each team over the 3 seasons. We need to add up games won and played this way as these tables don't have a specific column that totals wins but the game_info table does have a 'result' column that will display a '1' if the home team wins. Next, we use the JOIN to join in the game_info table and the WHERE function to filter by the most recent 3 seasons. Finally, we GROUP BY team. 

The output structure of this first part is shown below:

![image_alt](https://github.com/brianhornick/NBA-Stats-Analysis-SQL/blob/main/Images/Screenshot%202025-02-21%20142835.png?raw=true)

Now we need to put this query into a CTE so that we can calculate our win percentages off of it. Then, SUM and FILTER functions are used to add up the wins of both groups of 5 teams. We divide each by the total games * 5 as there are 5 teams and use the ROUND function as well as multiplying each numerator by 100.0 to obtain 2 clean percentages rounded to 2 decimal places as shown below:

![image_alt](https://github.com/brianhornick/NBA-Stats-Analysis-SQL/blob/main/Images/Screenshot%202025-02-21%20141710.png?raw=true)

As clearly shown, there is a stark difference in win % between the teams that shoot the most 3s and those that shoot the least. 

A correlation definitely exists, however, correlation does not equal causation and it's important for us to look at possible confounding variables.

### Question 4 - Adjusting for 3-Point Percentage

One confounding variable that is almost definitely leading teams to shoot more 3-pointers is that they are simply better at shooting them and thus will shoot more. In this section, we will be comparing the 3PA and wins of teams that shoot around the league average in 3P% (3-Point Make Percentage). Again, we will look at this dataset's most recent 3 seasons.

The first part of this code is obtaining the league average for 3P% over the last 3 years. We will use this figure in the next part of our query to find teams that shoot around this average. We have nested this figure in a CTE so that we can easily refer to it in the next part of our query.

Following this, the column 'team', as well as aggregate calculations of rounded average 3 point attempts per game and rounded average 3P% per game are performed so that we can compare the 3PA and ensure that all these teams being compared have nearly the same 3P%. Lastly, a SUM and CASE function is used again to add up all the home and away wins for each team. 

Next, we join our two tables once again, so that we can use filter (WHERE clause) by season and then GROUP BY team. Now, to only select teams around the league average, we must use the HAVING clause (as aggregate functions are being performed here), followed by a nested query that obtains our league average 3P% calculation. We take this calculation and subtract it from each team's 3P%. Using the BETWEEN function, only teams that shoot within 1% above or below the league average will be included. 

```
WITH league3P_avg as (
SELECT ROUND(avg([3Pp]),2) AS league_3Pp
FROM team_stats ts
    JOIN game_info gi
    ON ts.game_id = gi.game_id
WHERE season = 2223 OR season = 2122 OR season = 2021)

SELECT team, ROUND(AVG([3PA]),2) AS average_3pa, ROUND(AVG([3Pp]),4) * 100 AS average_3Pperc,
SUM(CASE WHEN team = [home_team] AND [result] = 1 THEN 1
WHEN team = [away_team] AND result = 0 THEN 1
ELSE 0 END) AS total_wins
FROM team_stats ts
    JOIN game_info gi
    ON ts.game_id = gi.game_id
WHERE season = 2223 OR season = 2122 OR season = 2021
GROUP BY team
HAVING 
(SELECT league_3Pp
FROM league3P_avg) - ROUND(AVG([3Pp]),2) BETWEEN -0.01 AND 0.01
ORDER BY [average_3pa];
```
Here is what the above query executes: 

![image_alt](https://github.com/brianhornick/NBA-Stats-Analysis-SQL/blob/main/Images/Screenshot%202025-02-20%20151446.png?raw=true)

8 teams fall in this +/- 1% of league average 3P%

Our results show a pretty inconclusive answer as the total wins, across the 3 seasons seem to be uncorrelated with 3PA, when adjusted for 3P%. There are likely many other factors at play, such as defense, rebounding, turnovers, ect. and our sample size is pretty small.

Let's take a look at rebounding and compare its impact on winning vs 3P%.

### Question 5 - Comparing the Winning Percentage of Outrebounding Opponents Vs Shooting A Higher 3-Point Percentage

```
SELECT 

ROUND((SELECT COUNT(*)
FROM team_stats g1
JOIN team_stats g2 
    ON g1.game_id = g2.game_id 
    AND g1.team <> g2.team
WHERE g1.TRB > g2.TRB AND g1.PTS > g2.PTS)
* 100.0 / COUNT(DISTINCT game_id),2) AS Pct_Games_Won_Outrebounded,

ROUND((SELECT COUNT(*) 
FROM team_stats g1
JOIN team_stats g2 
    ON g1.game_id = g2.game_id 
    AND g1.team <> g2.team
WHERE g1.[3Pp] > g2.[3Pp] AND g1.PTS > g2.PTS)
* 100.0 / COUNT(DISTINCT game_id),2) AS Pct_Games_Won_More3Ppct

FROM team_stats
```
To solve this problem, rather than join other tables, we must perform a self-join. This is because each game_id has 2 rows, one for each team. By performing a self-join on the condition that the teams are not the same (AND g1.team <> g2.team), we can compare the two rows for each game_id much more easily. For each game, each team is now being directly compared to their opponent. 

Next, a WHERE clause is used to select only the instances where a team had more points scored (PTS) and either total rebounds (TRB) (First sub-query) or a higher 3-Point Percentage (Second Sub-query). We then need to divide each output from our subquery by the total games played (COUNT(DISTINCT game_id)) and again, use the ROUND function as well as multiply the numerator by 100.0 to get a clean 2 decimal, percentage. 

From this code, here is what we get:

![image_alt](https://github.com/brianhornick/NBA-Stats-Analysis-SQL/blob/main/Images/Screenshot%202025-02-23%20140055.png?raw=true)

As we can see, shooting a higher 3-point percentage (69.88%) does seem to lead to more wins than outrebounding opponents(64.11%). However, the difference isn't huge and both do show a significant correlation with winning games.

### Conclusion

From looking at 3-point attempts and percentages through these 5 different problems, here are the conclusions we can draw:

1. There has certainly been a significant change in 3-pointers attempted per game over the past decade, most of this increase occurring between 2016 and 2020.
2. Teams that attempt more 3s do on average win more games, however, this is more because teams will attempt more 3s when they have players capable of shooting higher percentages, not that taking more 3s directly causes more wins
3. Shooting higher 3-point percentages does lead to more wins than outrebounding opponents but rebounds are still important
