# Project One - SQL


## Part 1 - Rolling 30 Day Retention by Day

My first step was imagining a chart that had 365 days in the x-axis.
Then I drew a table in google sheets with the 4 columns outlined in the project instructions:

![Imagined Table for Retention](Imagined%20Table%201.png)

I decided to start by writing a query for the most difficult column, which is the number of players retained per day.  

Thanks to the hint in the project description, I used the Max function to call on the last day played by each player.  I then subtracted their “joined” day from their max “day.”  In order to do this, I first had to join the `Matches` table to the `Player_Info` table on the player_id key.

Then I counted the number of player_ids per day ONLY IF their “Max Day” played minus “joined” was greater than 30.

Then I used the ROUND and SAFE_DIVIDE functions to populate the “Fractional Retention” column.

For each query that I tested, I made sure that every result was tied to “Joined” (aka day number) since the final table had to be anchored to this column.

![Chart of Fractional Retention](Chart%201.png)

The data visualization looks like waves or something akin to a lifesign monitor.
It shows fairly consistent up and down rolling 30-day-retention throughout the year.
According to my research, a 40% fractional retention for 30-day retention is decent.  Since the numbers here are higher, it shows a pretty strong performance for the game.


## Part 2 - Retention by Age Group

I approached this question in 2 ways.  

First, I sorted the players by age groups and then separated them by “retained” or “not retained” after 30 days.  It looked like the chart below:

![Chart of Retention by Age Group](Chart%202.png)


It looked pretty boring to me since it showed that more players were retained than not, in every age group.  And this chart just showed that 20-year-olds played the game the most.

So I decided to calculate the Retention Fraction by age group.  And that’s where the data was more interesting. 

To start off, I imagined the table below:

![Imagined Table for Fractional Retention by Age](Imagined%20Table%202.png)


My innermost query is to call on each player_id, their age, what day they joined, their last day playing a match and their retention value as  “Max Day” played minus “joined”

I then nested this query into another SELECT statement that called for distinct ages, a count of players with those ages, and a count of players with those ages ONLY IF their Retention value was greater than 30.

Then my final and outermost query is using the SAFE_DIVIDE function to divide  “Count of Retained” by “Count of Players” to populate the “Fractional Retention” column.

![Chart of Fractional Retention by Age](Chart%203.png)


It’s important to disregard the youngest and oldest age groups here since only 1 or 2 players are in each of those age groups, so the fractional retention of 0 is not really a fair assessment.

Otherwise, the data visualization shows a fairly consistent fractional retention over most age groups.  

It’s interesting to note that fractional retention is highest in the 29-year-olds and 31-year-olds, but dips sharply in the middle with the 30-year-olds.  This could be just a random coincidence since this is data made-up from an imaginary company.

