# 🏈 Fantasy Football League Database

**Team Name:** Group 1

## Team Members

| Name | GitHub |
| --- | --- |
| Elher Zemihret | https://github.com/Ezemihret/SQLGROUPPPROJECT |
| Allison Davis | https://github.com/AllisonDavis149/GP1?tab=readme-ov-file#gp1 |
| Coen Cardelli | https://github.com/CoenCardelli/SQL |
| Charlotte Holzapfel | https://github.com/charlotteholzapfel/Group-Project-1 |

---

## Scenario Description

Our database models a Fantasy Football League platform — a system where users join leagues, manage teams, draft real NFL players, and compete based on those players' weekly real-life performance.

Each League hosts multiple Users, and each User manages one or more Teams. Teams are built through a Draft, where players (real NFL athletes) are selected in rounds. Once the season begins, WeeklyStats track each player's points scored per week, which determines how teams perform in head-to-head Games. Users can also make Transactions (adding, dropping, or trading players) throughout the season.

The database supports tracking of league activity, draft history, player performance, team standings, and transaction records — everything a league manager would need to run and analyze a fantasy football season.

---

## Data Model

![Data Model](data_model.png)

### Explanation of the Data Model



**DraftPicks** records which Team selected which Player, in what round and pick number. It links Team and Player (M:1 to each).


**Games** records a head-to-head matchup between two Teams for a given week, storing the score for each side and the winner. The `homeTeamID`, `awayTeamID`, and `winnerTeamID` are all foreign keys back to Teams.


**Leagues** sits at the top of the hierarchy. A league has many Teams, but each Team belongs to exactly one League (1:M). A league is identified by a unique `leagueID` and stores the league name and season year.


**Players** represents a real NFL athlete. A Player can appear in many WeeklyStats records (one per week), many DraftPicks (across different leagues/seasons), and many Transactions (all 1:M). Players also have a `teamID` (FK to Teams), an `nflTeam` text label storing the real NFL franchise name (e.g., "Chiefs"), and a self-referencing `captainID` that can designate a team captain relationship.


**Teams** is the central entity — it connects a User to a League via foreign keys `userID` and `leagueID`. Each Team participates in many Games, initiates many Transactions, and makes many DraftPicks (all 1:M).


**Transactions** logs adds, drops, or trades. Each transaction is tied to a Team and a Player, along with the type and the week it occurred.


**Users** represents any person participating in the platform. A single User can manage multiple Teams across different leagues (1:M). Users are identified by `userID` and store basic contact info (name, email).


**WeeklyStats** captures a Player's fantasy points for a specific week. Each record is tied to one Player and one week number, and stores `pointsScored`.






### What the database supports:
- Tracking users, teams, leagues, and seasons
- Recording full draft history
- Storing player performance data week by week
- Logging game results and standings
- Managing roster transactions (adds, drops, trades)
- Identifying team captains via self-referencing relationship on Players

### What the database does NOT support:
- Real-time data feeds or live scoring
- Multiple positions per player per week
- Salary cap or auction draft formats
- Playoff bracket structures (beyond regular season games)

---

## Data Dictionary

---

### DraftPicks

| Column Name | Description | Data Type | Size | Format | Key? |
| --- | --- | --- | --- | --- | --- |
| pickID | Unique sequential number identifying each draft pick | Numeric | | | PK |
| roundNumber | The round number within the draft in which the pick was made (e.g., 1–15) | Numeric | | | |
| pickNumber | The overall pick number within the draft | Numeric | | | |
| playerID | Indicates the player who was selected with this pick | Numeric | | | FK (ref. Players) |
| teamID | Indicates the fantasy team that made this draft pick | Numeric | | | FK (ref. Teams) |

---

### Games

| Column Name | Description | Data Type | Size | Format | Key? |
| --- | --- | --- | --- | --- | --- |
| gameID | Unique sequential number identifying each game | Numeric | | | PK |
| weekNumber | The NFL week number during which the game was played (1–18) | Numeric | | | |
| homeTeamID | Indicates the home team participating in the matchup | Numeric | | | FK (ref. Teams) |
| awayTeamID | Indicates the away team participating in the matchup | Numeric | | | FK (ref. Teams) |
| homeScore | Total fantasy points scored by the home team during the game | Numeric | | | |
| awayScore | Total fantasy points scored by the away team during the game | Numeric | | | |
| leagueID | Indicates the league in which the game was played | Numeric | | | FK (ref. Leagues) |
| winnerTeamID | Indicates the team that won the game | Numeric | | | FK (ref. Teams) |

---

### Leagues

| Column Name | Description | Data Type | Size | Format | Key? |
| --- | --- | --- | --- | --- | --- |
| leagueID | Unique sequential number identifying each fantasy league | Numeric | | | PK |
| leagueName | The name given to the fantasy football league | Text | 45 | | |
| seasonYear | The NFL season year the league is associated with | Text | 4 | 9999 | |

---

### Players

| Column Name | Description | Data Type | Size | Format | Key? |
| --- | --- | --- | --- | --- | --- |
| playerID | Unique sequential number identifying each NFL player | Numeric | | | PK |
| playerName | The full name of the real NFL player | Text | 45 | | |
| position | The player's position on the field (QB, RB, WR, TE, K, DEF) | Text | 45 | | |
| teamID | Indicates the fantasy team the player is currently rostered on | Numeric | | | FK (ref. Teams) |
| captainID | Indicates the player designated as team captain; references itself | Numeric | | | FK (ref. Players) |

---

### Teams

| Column Name | Description | Data Type | Size | Format | Key? |
| --- | --- | --- | --- | --- | --- |
| teamID | Unique sequential number identifying each fantasy team | Numeric | | | PK |
| teamName | The name given to the fantasy football team | Text | 45 | | |
| userID | Indicates the user who owns and manages this team | Numeric | | | FK (ref. Users) |
| leagueID | Indicates the league that this team belongs to | Numeric | | | FK (ref. Leagues) |

---

### Transactions

| Column Name | Description | Data Type | Size | Format | Key? |
| --- | --- | --- | --- | --- | --- |
| transactionID | Unique sequential number identifying each roster transaction | Numeric | | | PK |
| transactionType | The type of transaction performed on the roster: ADD, DROP, or TRADE | Text | 45 | | |
| weekNumber | The NFL week number during which the transaction occurred | Numeric | | | |
| teamID | Indicates the team that initiated the transaction | Numeric | | | FK (ref. Teams) |
| playerID | Indicates the player involved in the transaction | Numeric | | | FK (ref. Players) |

---

### Users

| Column Name | Description | Data Type | Size | Format | Key? |
| --- | --- | --- | --- | --- | --- |
| userID | Unique sequential number identifying each platform user | Numeric | | | PK |
| firstName | The first name of the user | Text | 45 | | |
| lastName | The last name of the user | Text | 45 | | |
| email | The email address of the user | Text | 45 | aaaaaaaaa@aaaaa.aaa | |

---

### WeeklyStats

| Column Name | Description | Data Type | Size | Format | Key? |
| --- | --- | --- | --- | --- | --- |
| statsID | Unique sequential number identifying each weekly stats record | Numeric | | | PK |
| weekNumber | The NFL week number the stats were recorded for (1–18) | Numeric | | | |
| pointsScored | The total fantasy points the player scored during that week | Numeric |  | | |
| playerID | Indicates the player that these weekly stats belong to | Numeric | | | FK (ref. Players) |

---

## Ten Queries

### Query Feature Matrix

| Feature | Q1 | Q2 | Q3 | Q4 | Q5 | Q6 | Q7 | Q8 | Q9 | Q10 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Multiple Table Join | X | X | X |  | X | X | X |  | X | X |
| Subquery |  |  | X |  |  | X |  |  |  | X |
| Correlated Subquery |  |  |  |  |  |  |  |  |  | X |
| GROUP BY | X | X | X | X |  | X | X | X | X |  |
| GROUP BY with HAVING |  | X |  |  |  |  | X |  |  |  |
| Multi-condition WHERE |  |  |  | X | X |  |  | X |  | X |
| Built-in Functions / Calculated Field | X | X | X | X |  | X | X | X | X | X |
| REGEXP |  |  |  |  |  |  |  | X |  |  |
| NOT EXISTS |  |  |  |  | X |  |  |  |  |  |

---

### Q1 — Top 5 Players by Average Weekly Points (Simple)

**Description:** Retrieve the top 5 NFL players ranked by their average fantasy points scored per week.

**Managerial Justification:** League managers and users want to know which players are the most consistently valuable. This query helps identify which players to target in trades or on the waiver wire.

```sql
SELECT Players.playerName, Players.position,
       AVG(WeeklyStats.pointsScored) AS avgPoints
FROM Players
JOIN WeeklyStats ON Players.playerID = WeeklyStats.playerID
GROUP BY Players.playerID, Players.playerName, Players.position
ORDER BY avgPoints DESC
LIMIT 5;
```

**Result:**

```
+ --------------- + ------------- + -------------- +
| playerName      | position      | avgPoints      |
+ --------------- + ------------- + -------------- +
| Christian McCaffrey | RB            | 43.5000        |
| Josh Allen      | QB            | 32.1000        |
| Lamar Jackson   | QB            | 30.4000        |
| Patrick Mahomes | QB            | 29.9000        |
| CeeDee Lamb     | WR            | 29.4000        |
+ --------------- + ------------- + -------------- +
5 rows
```

---

### Q2 — Teams with More Than 3 Transactions in a Season (Complex)

**Description:** Find all fantasy teams that have made more than 3 transactions (adds, drops, or trades) within a league.

**Managerial Justification:** Highly active teams may signal engaged users or desperate roster management. Commissioners can use this to monitor trade activity and flag potential abuse.

```sql
SELECT Teams.teamName,
       Users.firstName,
       Users.lastName,
       COUNT(Transactions.transactionID) AS totalTransactions
FROM Teams
JOIN Users ON Teams.userID = Users.userID
JOIN Transactions ON Teams.teamID = Transactions.teamID
GROUP BY Teams.teamID, Teams.teamName, Users.firstName, Users.lastName
HAVING COUNT(Transactions.transactionID) > 3
ORDER BY totalTransactions DESC;
```

**Result:**

```
+ ------------- + -------------- + ------------- + ---------------------- +
| teamName      | firstName      | lastName      | totalTransactions      |
+ ------------- + -------------- + ------------- + ---------------------- +
| End Zone Empire | Sofia          | Nguyen        | 5                      |
| Blitz Brigade | James          | Carter        | 4                      |
| Pocket Passers | Marcus         | Williams      | 4                      |
| Red Zone Raiders | Priya          | Patel         | 4                      |
| Hail Mary Heroes | Derek          | Johnson       | 4                      |
+ ------------- + -------------- + ------------- + ---------------------- +
5 rows
```

---

### Q3 — Players Drafted Who Scored Below League Average (Complex)

**Description:** Identify drafted players whose average weekly points fall below the overall league average for all players.

**Managerial Justification:** Helps managers evaluate draft efficiency — if a team consistently drafts underperforming players, it signals poor draft strategy and could explain losing records.

```sql
SELECT Players.playerName,
       Players.position,
       Teams.teamName,
       AVG(WeeklyStats.pointsScored) AS avgPoints
FROM DraftPicks
JOIN Players ON DraftPicks.playerID = Players.playerID
JOIN Teams ON DraftPicks.teamID = Teams.teamID
JOIN WeeklyStats ON Players.playerID = WeeklyStats.playerID
GROUP BY Players.playerID, Players.playerName, Players.position, Teams.teamName
HAVING AVG(WeeklyStats.pointsScored) < (
    SELECT AVG(pointsScored)
    FROM WeeklyStats
)
ORDER BY avgPoints ASC;
```

**Result:**

```
+ --------------- + ------------- + ------------- + -------------- +
| playerName      | position      | teamName      | avgPoints      |
+ --------------- + ------------- + ------------- + -------------- +
| Gus Edwards     | RB            | Red Zone Raiders | 8.4000         |
| Gus Edwards     | RB            | Hail Mary Heroes | 8.4000         |
| Dalton Kincaid  | TE            | Pocket Passers | 10.0000        |
| Dalton Kincaid  | TE            | Fourth & Forever | 10.0000        |
| Amari Cooper    | WR            | End Zone Empire | 12.2000        |
| Amari Cooper    | WR            | Snap Judgments | 12.2000        |
| Sam LaPorta     | TE            | Blitz Brigade | 12.3000        |
| Tony Pollard    | RB            | Blitz Brigade | 13.2000        |
| Tony Pollard    | RB            | Sack Pack     | 13.2000        |
| Breece Hall     | RB            | Red Zone Raiders | 16.0000        |
| Stefon Diggs    | WR            | Hail Mary Heroes | 17.1000        |
| Mark Andrews    | TE            | Fourth & Forever | 18.3000        |
| Derrick Henry   | RB            | Snap Judgments | 20.9000        |
| Davante Adams   | WR            | End Zone Empire | 22.1000        |
+ --------------- + ------------- + ------------- + -------------- +
14 rows
```

---

### Q4 — Weekly Points Scored by Each Team for a Specific Week (Simple)

**Description:** Show the total fantasy points scored by each team during a given week (e.g., Week 5).

**Managerial Justification:** This is the core metric for determining weekly game outcomes. Managers use this data to compare performance and adjust rosters heading into the next week.

```sql
SELECT Teams.teamName,
       SUM(WeeklyStats.pointsScored) AS totalPoints
FROM Teams
JOIN DraftPicks ON Teams.teamID = DraftPicks.teamID
JOIN WeeklyStats ON DraftPicks.playerID = WeeklyStats.playerID
WHERE WeeklyStats.weekNumber = 5
  AND WeeklyStats.pointsScored > 0
GROUP BY Teams.teamID, Teams.teamName
ORDER BY totalPoints DESC;
```

**Result:**

```
+ ------------- + ---------------- +
| teamName      | totalPoints      |
+ ------------- + ---------------- +
| Blitz Brigade | 77               |
| Pocket Passers | 76               |
| End Zone Empire | 73               |
| Snap Judgments | 71               |
| Sack Pack     | 63               |
| Hail Mary Heroes | 59               |
| Fourth & Forever | 58               |
| Red Zone Raiders | 46               |
+ ------------- + ---------------- +
8 rows
```

---

### Q5 — Teams That Have Never Made a Transaction (Simple)

**Description:** Find all teams that have not made any adds, drops, or trades during the season.

**Managerial Justification:** Inactive teams hurt league competitiveness. Commissioners can use this to identify disengaged users and reach out to improve participation.

```sql
SELECT Teams.teamName,
       Users.firstName,
       Users.lastName,
       Users.email
FROM Teams
JOIN Users ON Teams.userID = Users.userID
WHERE NOT EXISTS (
    SELECT 1
    FROM Transactions
    WHERE Transactions.teamID = Teams.teamID
);
```

**Result:**

```
+ ------------- + -------------- + ------------- + ---------- +
| teamName      | firstName      | lastName      | email      |
+ ------------- + -------------- + ------------- + ---------- +
| Double Trouble | James          | Carter        | jcarter@email.com |
| Second Season | Sofia          | Nguyen        | snguyen@email.com |
| Snap Judgments | Tyler          | Brooks        | tbrooks@email.com |
| Sack Pack     | Hannah         | Scott         | hscott@email.com |
+ ------------- + -------------- + ------------- + ---------- +
4 rows
```

---

### Q6 — League Standings Based on Win/Loss Record (Complex)

**Description:** Calculate each team's wins and losses across all games within a league and sort by wins descending.

**Managerial Justification:** Standings are the primary measure of success in a fantasy league. This query powers leaderboards and determines playoff seeding.

```sql
SELECT Teams.teamName,
       COUNT(CASE 
                WHEN Games.winnerTeamID = Teams.teamID THEN 1 
            END) AS wins,
       COUNT(CASE 
                WHEN Games.winnerTeamID != Teams.teamID
                     AND (Games.homeTeamID = Teams.teamID 
                          OR Games.awayTeamID = Teams.teamID)
                THEN 1 
            END) AS losses
FROM Teams
JOIN Games 
  ON Teams.teamID = Games.homeTeamID 
  OR Teams.teamID = Games.awayTeamID
WHERE Teams.leagueID = (
    SELECT Leagues.leagueID 
    FROM Leagues 
    LIMIT 1
)
GROUP BY Teams.teamID, Teams.teamName
ORDER BY wins DESC;
```

**Result:**

```
+ ------------- + --------- + ----------- +
| teamName      | wins      | losses      |
+ ------------- + --------- + ----------- +
| Blitz Brigade | 5         | 0           |
| Sack Pack     | 5         | 0           |
| End Zone Empire | 4         | 1           |
| Red Zone Raiders | 2         | 3           |
| Hail Mary Heroes | 2         | 3           |
| Pocket Passers | 1         | 4           |
| Snap Judgments | 1         | 4           |
| Fourth & Forever | 0         | 5           |
+ ------------- + --------- + ----------- +
8 rows
```

---

### Q7 — Draft Round Efficiency: Average Points by Round (Complex)

**Description:** Calculate the average fantasy points scored by players grouped by the draft round in which they were selected.

**Managerial Justification:** Reveals whether early round picks justify their draft position. If late-round picks outperform early ones, it signals poor draft strategy across the league — valuable insight for future drafts.

```sql
SELECT DraftPicks.roundNumber,
       COUNT(DISTINCT DraftPicks.playerID) AS playersDrafted,
       AVG(WeeklyStats.pointsScored) AS avgPointsPerRound
FROM DraftPicks
JOIN WeeklyStats ON DraftPicks.playerID = WeeklyStats.playerID
GROUP BY DraftPicks.roundNumber
HAVING AVG(WeeklyStats.pointsScored) > 5
ORDER BY DraftPicks.roundNumber ASC;
```

**Result:**

```
+ ---------------- + ------------------- + ---------------------- +
| roundNumber      | playersDrafted      | avgPointsPerRound      |
+ ---------------- + ------------------- + ---------------------- +
| 1                | 8                   | 30.5000                |
| 2                | 8                   | 19.6500                |
| 3                | 4                   | 10.9500                |
+ ---------------- + ------------------- + ---------------------- +
3 rows
```

---

### Q8 — Find Players at Skill Positions (QB, RB, WR, TE) Using REGEXP (Complex)

**Description:** Retrieve all players whose position matches one of the four primary skill positions using a regular expression pattern.

**Managerial Justification:** Fantasy leagues often score skill positions differently. Filtering by position group lets managers compare position scarcity and depth across the league.

```sql
SELECT Players.playerName,
       Players.position,
       AVG(WeeklyStats.pointsScored) AS avgPoints
FROM Players
JOIN WeeklyStats ON Players.playerID = WeeklyStats.playerID
WHERE Players.position REGEXP '^(QB|RB|WR|TE)$'
  AND WeeklyStats.pointsScored > 0
GROUP BY Players.playerID, Players.playerName, Players.position
ORDER BY Players.position, avgPoints DESC;
```

**Result:**

```
+ --------------- + ------------- + -------------- +
| playerName      | position      | avgPoints      |
+ --------------- + ------------- + -------------- +
| Josh Allen      | QB            | 32.1000        |
| Lamar Jackson   | QB            | 30.4000        |
| Patrick Mahomes | QB            | 29.9000        |
| Dak Prescott    | QB            | 23.5000        |
| Christian McCaffrey | RB            | 43.5000        |
| Saquon Barkley  | RB            | 27.0000        |
| Derrick Henry   | RB            | 20.9000        |
| Breece Hall     | RB            | 16.0000        |
| Tony Pollard    | RB            | 13.2000        |
| Gus Edwards     | RB            | 8.4000         |
| Travis Kelce    | TE            | 22.8000        |
| Mark Andrews    | TE            | 18.3000        |
| Sam LaPorta     | TE            | 12.3000        |
| Dalton Kincaid  | TE            | 10.0000        |
| CeeDee Lamb     | WR            | 29.4000        |
| Tyreek Hill     | WR            | 28.1000        |
| Justin Jefferson | WR            | 27.8000        |
| Davante Adams   | WR            | 22.1000        |
| Stefon Diggs    | WR            | 17.1000        |
| Amari Cooper    | WR            | 12.2000        |
+ --------------- + ------------- + -------------- +
20 rows
```

---

### Q9 — Users Who Manage Teams in Multiple Leagues (Simple)

**Description:** Find users who are participating in more than one fantasy league simultaneously.

**Managerial Justification:** Power users who manage multiple teams are highly engaged. Identifying them helps platform administrators target premium feature offerings or provide support.

```sql
SELECT Users.userID,
       Users.firstName,
       Users.lastName,
       Users.email,
       COUNT(Teams.teamID) AS numberOfTeams
FROM Users
JOIN Teams ON Users.userID = Teams.userID
GROUP BY Users.userID, Users.firstName, Users.lastName, Users.email
HAVING COUNT(Teams.teamID) > 1
ORDER BY numberOfTeams DESC;
```

**Result:**

```
+ ----------- + -------------- + ------------- + ---------- + ------------------ +
| userID      | firstName      | lastName      | email      | numberOfTeams      |
+ ----------- + -------------- + ------------- + ---------- + ------------------ +
| 1           | James          | Carter        | jcarter@email.com | 2                  |
| 2           | Sofia          | Nguyen        | snguyen@email.com | 2                  |
+ ----------- + -------------- + ------------- + ---------- + ------------------ +
2 rows
```

---

### Q10 — Players Outperforming Their Own Season Average in the Latest Week (Complex)

**Description:** Use a correlated subquery to find players whose most recent week's score exceeds their own personal season average.

**Managerial Justification:** Hot players — those trending above their own baseline — are prime trade targets and waiver wire pickups. This query surfaces players with current upward momentum.

```sql
SELECT Players.playerName,
       Players.position,
       WeeklyStats.weekNumber,
       WeeklyStats.pointsScored AS latestWeekPoints
FROM Players
JOIN WeeklyStats ON Players.playerID = WeeklyStats.playerID
WHERE WeeklyStats.weekNumber = (
    SELECT MAX(weekNumber)
    FROM WeeklyStats
)
AND WeeklyStats.pointsScored > (
    SELECT AVG(WeeklyStats2.pointsScored)
    FROM WeeklyStats WeeklyStats2
    WHERE WeeklyStats2.playerID = Players.playerID
)
ORDER BY WeeklyStats.pointsScored DESC;
```

**Result:**

```
+ --------------- + ------------- + --------------- + --------------------- +
| playerName      | position      | weekNumber      | latestWeekPoints      |
+ --------------- + ------------- + --------------- + --------------------- +
| Josh Allen      | QB            | 10              | 36                    |
| CeeDee Lamb     | WR            | 10              | 36                    |
| Lamar Jackson   | QB            | 10              | 34                    |
| Saquon Barkley  | RB            | 10              | 33                    |
| Patrick Mahomes | QB            | 10              | 30                    |
| Davante Adams   | WR            | 10              | 28                    |
| Dak Prescott    | QB            | 10              | 26                    |
| Mark Andrews    | TE            | 10              | 21                    |
| Stefon Diggs    | WR            | 10              | 19                    |
| Sam LaPorta     | TE            | 10              | 15                    |
| Dalton Kincaid  | TE            | 10              | 12                    |
+ --------------- + ------------- + --------------- + --------------------- +
11 rows
```

---

## Database Information

- **Database Name:** fantasy_football_db
- **Platform:** MySQL
- All queries are bookmarked as stored procedures named `TP_Q1` through `TP_Q10`

## Additional Notes

- Each table is populated with at least 10+ rows of sample data to ensure queries return meaningful result sets
- Foreign key constraints are enforced to maintain referential integrity
- The data model was designed to be scalable across multiple seasons by including `seasonYear` in the Leagues table
- `captainID` in the Players table is a self-referencing foreign key allowing designation of a team captain
