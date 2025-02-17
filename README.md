# Analyzing NBA Data from the Last Decade with SQL - The 3 pointer and it's affect on the game
## Introduction
We have been given a dataset containing comprehensive game stats, player stats, and team stats for every game from the 2013-14 season up until the 2022-23 season. This dataset was downloaded from Kaggle.com. [NBA Boxscore Dataset](https://www.kaggle.com/datasets/lukedip/nba-boxscore-dataset) The NBA has seen a huge increase in 3-point shots both made and attempted in the last decade. We aim to use this data to answer some unique questions regarding 3-pointers that will give more insight into how much the 3-pointer really affects winning games. 

#### Questions

What was the average number of 3s taken per game by a team in the 2013-2014 vs the 2022-2023 season?

Which teams has taken the most 3s across this entire decade? What is their overall winning percentage compared to the teams with the least amount of 3s taken?

Do more 3PA (3-point attempts) lead to higher TS% (True shooting percentage)? 

Does having a center that can shoot 3s boost win %?

What results in more wins? Outrebounding opponents vs making more 3-pointers.


#### Question 1 - Average number of 3s: 2013/14 vs 2022/23

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

