# Analyzing NBA Data from the Last Decade with SQL - The 3 Pointer and its Effect on the Game
### Introduction
We have been given a dataset containing comprehensive game stats, player stats, and team stats for every game from the 2013-14 season up until the 2022-23 season. This dataset was downloaded from Kaggle.com. [NBA Boxscore Dataset](https://www.kaggle.com/datasets/lukedip/nba-boxscore-dataset) The NBA has seen a huge increase in 3-point shots both made and attempted in the last decade. We aim to use this data to answer some unique questions regarding 3-pointers that will give more insight into how much the 3-pointer really affects winning games. We will be using SQLite to discover the answers to these questions.

### Questions

What was the average number of 3s taken per game by a team in the 2013-2014 vs the 2022-2023 season?

What is the year-over-year change in average 3s taken per game by a team?

Which teams has taken the most 3s across the most recent 3 years in this dataset? What is their overall winning percentage compared to the teams with the least amount of 3s taken?

Do more 3PA (3-point attempts) lead to higher TS% (True shooting percentage)? 

What results in more wins?  Outrebounding opponents vs making more 3-pointers.


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

To find this, first, we create a CTE (Common Table Expression) which establishes a temporary table that takes the season and average 3PA per game, per team. A similar method is used as the first problem, obtaining our averages using the AVG function, rounding our answers to 2 decimal places with the ROUND function, using JOIN to get the season variable, and in this case, using GROUP BY to get the average 3PA per each season. Then, from this temporary table, we query the season again and then take the average 3PA per season subtracted by the previous year (using the LAG function) to get our 
```
WITH avg_3pa_calc AS (
SELECT season, ROUND(AVG([3PA]),2) AS avg_3pa
FROM team_stats ts
  JOIN game_info gi
  ON ts.game_id = gi.game_id
GROUP BY season)

SELECT season, avg_3pa - LAG(avg_3pa, 1) OVER (ORDER BY season) AS YOY_3pa_change
FROM avg_3pa_calc
```
Here is what this code executes:

![image_alt](https://github.com/brianhornick/NBA-Stats-Analysis-SQL/blob/main/Images/Screenshot%202025-02-19%20155000.png?raw=true)

As we can see the largest increase in 3PA happened between the 2016-2017 - 2019-2020 season where there was a cumulative increase of over 10 3s a game! 
The most recent season was actually the first season in the last 10 years to see a decrease in 3s per game.

### Question 3 - The Winning Percentage of High vs Low 3-Point Volume Teams

In this section, we aim to take the winning % of the top 5 teams with the highest volume of 3s attempted and compare it to the winning % of the teams with the least amount of 3's attempted.

Part 1: Finding the Top and Bottom 5 Teams in Total 3PA

To find the top and bottom 5 teams in terms of total 3-point attempts for the last 3 years, we must first select each team and their respective total 3PA output over the past 3 years. SUM, GROUP BY, and ORDER BY functions will allow us to get the total 3s, grouped by each team ordered by the amount of 3s they attempted. To filter by the last 3 seasons, the same join function must be used as the previous question, however this time, we will use a where clause to filter the last 3 years.

```
SELECT team, SUM([3PA]) AS total_3PA
FROM team_stats ts
  JOIN game_info gi
  ON ts.game_id = gi.game_id
WHERE season = 2223 OR season = 2122 OR season = 2021
GROUP BY team
ORDER BY total_3PA DESC;
```
Here is the output for the above code:

![image_alt](https://github.com/brianhornick/NBA-Stats-Analysis-SQL/blob/main/Images/Screenshot%202025-02-18%20143407.png?raw=true) ![image_alt](https://github.com/brianhornick/NBA-Stats-Analysis-SQL/blob/main/Images/Screenshot%202025-02-18%20152021.png?raw=true)

As shown, the top 5 teams we will be looking at are Golden State, Utah, Dallas, Boston and Milwaukee and the bottom 5 teams are Chicago, Washington, New Orleans, San Antonio and Cleveland

Part 2: Calculating and Comparing Win %

Now to calculate the win% of the top 5 and bottom 5 teams, we must add up all the wins for each of these groups and divide by the total number of games played. Unfortunately, this isn't as straightforward to calculate as expected, as wins and losses in this database are categorized by home team vs away team. When the home team wins the 'result' column will display a 1, when the away team wins the 'result' column displays a 0. The COUNT function is used to tally up all the wins, followed by the FILTER function which filters the result by the 'result' we are looking for (1 when adding home wins, 0 when adding away wins) and the teams we are interested in. 

Total home and away wins are added together and then divided by the total number of games played by all 5 teams, which again we need to segment away teams and home teams. This is because if away and home teams are added together in the same FILTER block, occurrences where 2 of the teams of interest played each other will only count as one game played, when in fact it should be 2 (1 game played for each of the 2 teams). The ROUND function as well as multiplying the numerator by 100.0 ensures each result is a percentage number nicely rounded to 2 decimal places. The WHERE clause is used to ensure only results from the most recent 3 seasons of this database are used.

```
SELECT 
ROUND((COUNT(home_team) FILTER (WHERE result = 1 AND (home_team = 'GSW' OR home_team = 'DAL' OR home_team = 'UTA' OR home_team = 'BOS' OR home_team = 'MIL')) +
COUNT(away_team) FILTER (WHERE result = 0 AND (away_team = 'GSW' OR away_team = 'DAL' OR away_team = 'UTA' OR away_team = 'BOS' OR away_team = 'MIL'))) * 100.0
/ (COUNT(game_id) FILTER (WHERE home_team = 'GSW' OR home_team = 'DAL' OR home_team = 'UTA' OR home_team = 'BOS' OR home_team = 'MIL') 
+ COUNT(game_id) FILTER (WHERE away_team = 'GSW' OR away_team = 'DAL' OR away_team = 'UTA' OR away_team = 'BOS' OR away_team = 'MIL')),2) AS top_5winperc,

ROUND((COUNT(home_team) FILTER (WHERE result = 1 AND (home_team = 'CHI' OR home_team = 'WAS' OR home_team = 'NOP' OR home_team = 'SAS' OR home_team = 'CLE')) +
COUNT(away_team) FILTER (WHERE result = 0 AND (away_team = 'CHI' OR away_team = 'WAS' OR away_team = 'NOP' OR away_team = 'SAS' OR away_team = 'CLE'))) * 100.0
/ (COUNT(game_id) FILTER (WHERE home_team = 'CHI' OR home_team = 'WAS' OR home_team = 'NOP' OR home_team = 'SAS' OR home_team = 'CLE') 
+ COUNT(game_id) FILTER (WHERE away_team = 'CHI' OR away_team = 'WAS' OR away_team = 'NOP' OR away_team = 'SAS' OR away_team = 'CLE')),2) AS bottom_5winperc

FROM game_info
WHERE season = 2223 OR season = 2122 OR season = 2021;
```
Here is our result:

![image_alt](https://github.com/brianhornick/NBA-Stats-Analysis-SQL/blob/main/Images/Screenshot%202025-02-18%20163344.png?raw=true)

The difference here is very apparent as the top 5 teams have almost a 60% winning percentage compared to only a 45% winning percentage of the bottom 5 teams. 
Clearly, there is some correlation between taking more 3s and winning games, however, the relationship between the 3-pointer and winning games still requires further investigation. 
As many know, correlation does not always mean causation.


