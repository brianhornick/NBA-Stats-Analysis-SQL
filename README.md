# Analyzing NBA Data from the Last Decade with SQL - The 3 pointer and it's affect on the game
### Introduction
We have been given a dataset containing comprehensive game stats, player stats, and team stats for every game from the 2013-14 season up until the 2022-23 season. This dataset was downloaded from Kaggle.com. [NBA Boxscore Dataset](https://www.kaggle.com/datasets/lukedip/nba-boxscore-dataset) The NBA has seen a huge increase in 3-point shots both made and attempted in the last decade. We aim to use this data to answer some unique questions regarding 3-pointers that will give more insight into how much the 3-pointer really affects winning games. We will be using SQLite to discover the answers to these questions. The goal is to answer these questions in as few steps as possible.

### Questions

What was the average number of 3s taken per game by a team in the 2013-2014 vs the 2022-2023 season?

Which teams has taken the most 3s across the most recent 3 years in this dataset? What is their overall winning percentage compared to the teams with the least amount of 3s taken?

Do more 3PA (3-point attempts) lead to higher TS% (True shooting percentage)? 

Does having a center that can shoot 3s boost win %?

What results in more wins? Outrebounding opponents vs making more 3-pointers.


### Question 1 - Average number of 3s: 2013/14 vs 2022/23

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

### Question 2 - The winning percentage of high vs low 3-point volume teams

In this section, we aim to take the winning % of the top 5 teams with the highest volume of 3s attempted and compare it to the winning % of the teams with the least amount of 3's attempted.

Part 1: Finding the top and bottom 5 teams in total 3PA

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







