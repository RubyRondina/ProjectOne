# Project 2: Rolling 30 Day Retention

<p align="center"><img width="500" height="" src="https://github.com/RubyRondina/SQL_Project_30DayRetention/blob/main/TablesAndCharts/Retention.jpg"></p>


The purpose of this project is to use **SQL** to:

- Combine data in a relational database
- Develop complex queries to analyze data and answer relevant questions

And to take advantage of **Sheets** to:

- Facilitate the data analysis process
- Create clear visualizations of aggregated data


## The Background

As an analyst for a mobile game company, the task is to investigate player retention on the game's one year anniversary.  There is a rich store of four tables as follows:

-Match information, including the players who matched against each other, and the outcome,

-Player information, including information like the player's age and when they joined,

-Item information, including the item ID and the price

-Purchase information, including what player ID bought what item ID, and on what day.


The Schema is as follows:

<p align="center"><img width="500" height="" src="https://github.com/RubyRondina/SQL_Project_30DayRetention/blob/main/TablesAndCharts/Schema.png"></p>


### Question 1: Calculate the rolling 30-day retention as a fraction of the total playerbase at the time. 

Rolling 30-day retention asks the following question: 
Did a given player play a match 30 days after he or she joined? 
A player is either retained or not retained with respect to this retention metric. 


### Solution: Rolling 30 Day Retention by Day

My first step was imagining a chart that had 365 days in the x-axis.

Then I drew a table in google sheets with the 4 following columns for each day:

-The total number of players who joined that day

-Of the players who joined that day, how many were retained

-The fractional retention (the third column divided by the second column).

![Imagined Table for Retention](https://github.com/RubyRondina/SQL_Project_30DayRetention/blob/main/TablesAndCharts/Imagined%20Table%201.png)

I decided to start by writing a query for the most difficult column, which is the number of players retained per day by using the Max function to call on the last day played by each player.  I then subtracted their “joined” day from their max “day.”  In order to do this, I first had to join the `Matches` table to the `Player_Info` table on the player_id key.

Then I counted the number of player_ids per day ONLY IF their “Max Day” played minus “joined” was greater than 30.

Then I used the ROUND and SAFE_DIVIDE functions to populate the “Fractional Retention” column.

For each query that I tested, I made sure that every result was tied to “Joined” (aka day number) since the final table had to be anchored to this column.

The final query is as follows using 3 select statements:

```sql
SELECT
 day AS Day,
 Number_of_Players_Joined,
 Number_of_Players_Retained,
 Round (safe_divide (Number_of_Players_Retained,
     Number_of_Players_Joined),2) AS Fractional_Retention
FROM (
 SELECT
   joined AS Day,
   count (player_id) AS Number_of_Players_Joined,
   countif (Last_Day_Played - joined > 30) AS Number_of_Players_Retained
 FROM (
     SELECT
       M.player_id,
       joined,
       MAX(day) AS Last_Day_Played
     FROM
       `big-query-project-1-329514.Game_Company_Data.Matches` M
     JOIN
       `big-query-project-1-329514.Game_Company_Data.Player Info` PLIN
     ON
       M.player_id = PLIN.player_id
     GROUP BY
       player_id,
       joined)
       GROUP BY
   joined)
```


![Chart of Fractional Retention](https://github.com/RubyRondina/SQL_Project_30DayRetention/blob/main/TablesAndCharts/Chart%201.png)

The data visualization shows fairly consistent up and down rolling 30-day-retention throughout the year.
According to my research, a 40% fractional retention for 30-day retention is decent.  Since the numbers here are higher, it shows a pretty strong performance for the game.


### Question 2: Calculate 30 day retention by Age Group

I approached this question in 2 ways.  

First, I sorted the players by age groups and then separated them by “retained” or “not retained” after 30 days.  

### Solution A: Rolling 30 Day Retention by Age (Retained vs. Not Retained by Age)

```sql
SELECT
 age,
 COUNT(Retained) AS Number_of_Retained,
 Count (Not_Retained) AS Number_of_Not_Retained,
FROM (
 SELECT
   player_id,
   age,
   CASE
     WHEN Retention > 30 THEN 1
   ELSE
   NULL
 END
   AS Retained,
   CASE
     WHEN Retention <= 30 THEN 1
   ELSE
   NULL
 END
   AS Not_Retained
 FROM (
   SELECT
     M.player_id,
     age,
     joined,
     max (day) AS Last_Day_Played,
     max (day) - joined AS Retention
   FROM
     `big-query-project-1-329514.Game_Company_Data.Player Info` PLIN
   JOIN
     `big-query-project-1-329514.Game_Company_Data.Matches` M
   ON
     PLIN.player_id = M.player_id
   GROUP BY
     M.player_id,
     age,
     joined))
GROUP BY
 age
ORDER BY
 age
```

The results are visualized in the chart below:

![Chart of Retention by Age Group](https://github.com/RubyRondina/SQL_Project_30DayRetention/blob/main/TablesAndCharts/Chart%202.png)


This visualization looks pretty boring since it shows that more players were retained than not, in every age group.  And this chart just showed that 20-year-olds played the game the most.

So I decided to calculate the Retention Fraction by age group.  And that’s where the data was more interesting. 


### Solution B: Retention Fraction by Age (Retained Divided by Total, per Age)

To start off, I imagined the table below:

![Imagined Table for Fractional Retention by Age](https://github.com/RubyRondina/SQL_Project_30DayRetention/blob/main/TablesAndCharts/Imagined%20Table%202.png)


My innermost query is to call on each player_id, their age, what day they joined, their last day playing a match and their retention value as  “Max Day” played minus “joined”

I then nested this query into another SELECT statement that called for distinct ages, a count of players with those ages, and a count of players with those ages ONLY IF their Retention value was greater than 30.

Then my final and outermost query is using the SAFE_DIVIDE function to divide  “Count of Retained” by “Count of Players” to populate the “Fractional Retention” column.

```sql
SELECT
 Age,
 Number_of_Players,
 Number_Retained,
 ROUND(safe_divide (Number_Retained,
     Number_of_Players),2) AS Fractional_Retention
FROM (
 SELECT
   DISTINCT age,
   COUNT(player_id) AS Number_of_Players,
   COUNTIF(Retention > 30) AS Number_Retained
 FROM (
   SELECT
     M.player_id,
     age,
     joined,
     max (day) AS Last_Day_Played,
     max (day) - joined AS Retention
   FROM
     `big-query-project-1-329514.Game_Company_Data.Player Info` PLIN
   JOIN
     `big-query-project-1-329514.Game_Company_Data.Matches` M
   ON
     PLIN.player_id = M.player_id
   GROUP BY
     M.player_id,
     age,
     joined)
 GROUP BY
   age
 ORDER BY
   age)
```

These results are visualized in the chart below:

![Chart of Fractional Retention by Age](https://github.com/RubyRondina/SQL_Project_30DayRetention/blob/main/TablesAndCharts/Chart%203.png)


It’s important to disregard the youngest and oldest age groups here since only 1 or 2 players are in each of those age groups, so the fractional retention of 0 is not really a fair assessment.

Otherwise, the data visualization shows a fairly consistent fractional retention over most age groups.  

It’s interesting to note that fractional retention is highest in the 29-year-olds and 31-year-olds, but dips sharply in the middle with the 30-year-olds.

