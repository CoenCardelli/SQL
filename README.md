# 🏈 Fantasy Football League Database

## Team Name: Group [Your Group Number]

### Team Members
| Name | GitHub |
|------|--------|
| Elher Zemihret | [Link to GitHub] |
| Allison Davis | [Link to GitHub] |
| Coen Cardelli | [Link to GitHub] |
| Charlotte Holzapfel | [Link to GitHub] |

---

## Scenario Description

Our database models a **Fantasy Football League** platform — a system where users join leagues, manage teams, draft real NFL players, and compete based on those players' weekly real-life performance.

Each **League** hosts multiple **Users**, and each User manages one or more **Teams**. Teams are built through a **Draft**, where players (real NFL athletes) are selected in rounds. Once the season begins, **WeeklyStats** track each player's points scored per week, which determines how teams perform in head-to-head **Games**. Users can also make **Transactions** (adding, dropping, or trading players) throughout the season.

The database supports tracking of league activity, draft history, player performance, team standings, and transaction records — everything a league manager would need to run and analyze a fantasy football season.

---

## Data Model

![Data Model Diagram](data_model.png)

> **Note:** Replace `data_model.png` with your actual ER diagram image uploaded to this repo.

### Explanation of the Data Model

**League** sits at the top of the hierarchy. A league has many **Teams**, but each Team belongs to exactly one League *(1:M)*. A league is identified by a unique `leagueID` and stores the season year.

**User** represents any person participating in the platform. A single User can manage multiple Teams across different leagues *(1:M)*. Users are identified by `userID` and store basic contact info (name, email).

**Team** is the central entity — it connects a User to a League. Each Team participates in many **Games**, initiates many **Transactions**, and makes many **DraftPicks** *(all 1:M)*.

**Player** represents a real NFL athlete. A Player can appear in many **WeeklyStats** records (one per week), many **DraftPicks** (across different leagues/seasons), and many **Transactions** *(all 1:M)*.

**WeeklyStats** captures a Player's fantasy points for a specific week. Each record is tied to one Player and one week number, and stores `pointsScored`.

**DraftPick** records which Team selected which Player, in what round and pick number. It links Team and Player *(M:1 to each)*.

**Game** records a head-to-head matchup between two Teams for a given week, storing the score for each side and the winner.

**Transaction** logs adds, drops, or trades. Each transaction is tied to a Team and a Player, along with the type and the week it occurred.

#### What the database supports:
- Tracking users, teams, leagues, and seasons
- Recording full draft history
- Storing player performance data week by week
- Logging game results and standings
- Managing roster transactions (adds, drops, trades)

#### What the database does NOT support:
- Real-time data feeds or live scoring
- Multiple positions per player per week
- Salary cap or auction draft formats
- Playoff bracket structures (beyond regular season games)

---

## Data Dictionary

### League
| Column | Data Type | Key | Description |
|--------|-----------|-----|-------------|
| leagueID | INT | PK | Unique identifier for each league |
| leagueName | VARCHAR(100) | | Name of the fantasy league |
| seasonYear | INT | | The NFL season year (e.g., 2024) |

### User
| Column | Data Type | Key | Description |
|--------|-----------|-----|-------------|
| userID | INT | PK | Unique identifier for each user |
| firstName | VARCHAR(50) | | User's first name |
| lastName | VARCHAR(50) | | User's last name |
| email | VARCHAR(100) | | User's email address |

### Team
| Column | Data Type | Key | Description |
|--------|-----------|-----|-------------|
| teamID | INT | PK | Unique identifier for each team |
| teamName | VARCHAR(100) | | Name of the fantasy team |
| userID | INT | FK (User) | The user who owns this team |
| leagueID | INT | FK (League) | The league this team belongs to |

### Player
| Column | Data Type | Key | Description |
|--------|-----------|-----|-------------|
| playerID | INT | PK | Unique identifier for each player |
| playerName | VARCHAR(100) | | Full name of the NFL player |
| position | VARCHAR(10) | | Player's position (QB, RB, WR, TE, K, DEF) |
| nflTeam | VARCHAR(50) | | Real NFL team the player plays for |

### WeeklyStats
| Column | Data Type | Key | Description |
|--------|-----------|-----|-------------|
| statsID | INT | PK | Unique identifier for each stats record |
| weekNumber | INT | | NFL week number (1–18) |
| pointsScored | DECIMAL(5,2) | | Fantasy points scored that week |
| playerID | INT | FK (Player) | The player these stats belong to |

### DraftPick
| Column | Data Type | Key | Description |
|--------|-----------|-----|-------------|
| pickID | INT | PK | Unique identifier for each draft pick |
| roundNumber | INT | | The draft round (e.g., 1–15) |
| pickNumber | INT | | The overall pick number |
| teamID | INT | FK (Team) | The team that made this pick |
| playerID | INT | FK (Player) | The player who was drafted |

### Game
| Column | Data Type | Key | Description |
|--------|-----------|-----|-------------|
| gameID | INT | PK | Unique identifier for each game |
| weekNumber | INT | | The week the game was played |
| homeTeamID | INT | FK (Team) | The home/first team |
| awayTeamID | INT | FK (Team) | The away/second team |
| homeScore | DECIMAL(6,2) | | Fantasy points scored by home team |
| awayScore | DECIMAL(6,2) | | Fantasy points scored by away team |
| winnerTeamID | INT | FK (Team) | The team that won the game |
| leagueID | INT | FK (League) | The league this game belongs to |

### Transaction
| Column | Data Type | Key | Description |
|--------|-----------|-----|-------------|
| transactionID | INT | PK | Unique identifier for each transaction |
| transactionType | VARCHAR(20) | | Type: ADD, DROP, or TRADE |
| weekNumber | INT | | The week the transaction occurred |
| teamID | INT | FK (Team) | The team making the transaction |
| playerID | INT | FK (Player) | The player involved in the transaction |

---

## Ten Queries

### Query Feature Matrix

| Feature | Q1 | Q2 | Q3 | Q4 | Q5 | Q6 | Q7 | Q8 | Q9 | Q10 |
|---------|----|----|----|----|----|----|----|----|----|----|
| Multiple Table Join | X | X | X | | X | X | X | | X | X |
| Subquery | | | X | | | X | | | X | |
| Correlated Subquery | | | | | | | | | | X |
| GROUP BY | X | X | X | X | | X | X | X | X | |
| GROUP BY with HAVING | | X | | | | | X | | | |
| Multi-condition WHERE | | | | X | X | | | X | | X |
| Built-in Functions / Calculated Field | X | X | X | X | | X | X | X | X | X |
| REGEXP | | | | | | | | X | | |
| NOT EXISTS | | | | | X | | | | | |

---

### Q1 — Top 5 Players by Average Weekly Points

**Description:** Retrieve the top 5 NFL players ranked by their average fantasy points scored per week.

**Managerial Justification:** League managers and users want to know which players are the most consistently valuable. This query helps identify which players to target in trades or on the waiver wire.

```sql
SELECT p.playerName, p.position, p.nflTeam,
       AVG(ws.pointsScored) AS avgPoints
FROM Player p
JOIN WeeklyStats ws ON p.playerID = ws.playerID
GROUP BY p.playerID, p.playerName, p.position, p.nflTeam
ORDER BY avgPoints DESC
LIMIT 5;
```

**Result:** *(paste query output here)*

---

### Q2 — Teams with More Than 3 Transactions in a Season

**Description:** Find all fantasy teams that have made more than 3 transactions (adds, drops, or trades) within a league.

**Managerial Justification:** Highly active teams may signal engaged users or desperate roster management. Commissioners can use this to monitor trade activity and flag potential abuse.

```sql
SELECT Team.teamName,
       User.firstName,
       User.lastName,
       COUNT(Transaction.transactionID) AS totalTransactions
FROM Team
JOIN User ON Team.userID = User.userID
JOIN Transaction ON Team.teamID = Transaction.teamID
GROUP BY Team.teamID, Team.teamName, User.firstName, User.lastName
HAVING COUNT(Transaction.transactionID) > 3
ORDER BY totalTransactions DESC;
```

**Result:** *(paste query output here)*

---

### Q3 — Players Drafted Who Scored Below League Average

**Description:** Identify drafted players whose average weekly points fall below the overall league average for all players.

**Managerial Justification:** Helps managers evaluate draft efficiency — if a team consistently drafts underperforming players, it signals poor draft strategy and could explain losing records.

```sql
SELECT Player.playerName,
       Player.position,
       Team.teamName,
       AVG(WeeklyStats.pointsScored) AS avgPoints
FROM DraftPick
JOIN Player ON DraftPick.playerID = Player.playerID
JOIN Team ON DraftPick.teamID = Team.teamID
JOIN WeeklyStats ON Player.playerID = WeeklyStats.playerID
GROUP BY Player.playerID, Player.playerName, Player.position, Team.teamName
HAVING AVG(WeeklyStats.pointsScored) < (
    SELECT AVG(pointsScored)
    FROM WeeklyStats
)
ORDER BY avgPoints ASC;
```

**Result:** *(paste query output here)*

---

### Q4 — Weekly Points Scored by Each Team for a Specific Week

**Description:** Show the total fantasy points scored by each team during a given week (e.g., Week 5).

**Managerial Justification:** This is the core metric for determining weekly game outcomes. Managers use this data to compare performance and adjust rosters heading into the next week.

```sql
SELECT Team.teamName,
       SUM(WeeklyStats.pointsScored) AS totalPoints
FROM Team
JOIN DraftPick ON Team.teamID = DraftPick.teamID
JOIN WeeklyStats ON DraftPick.playerID = WeeklyStats.playerID
WHERE WeeklyStats.weekNumber = 5
  AND WeeklyStats.pointsScored > 0
GROUP BY Team.teamID, Team.teamName
ORDER BY totalPoints DESC;
```

**Result:** *(paste query output here)*

---

### Q5 — Teams That Have Never Made a Transaction

**Description:** Find all teams that have not made any adds, drops, or trades during the season.

**Managerial Justification:** Inactive teams hurt league competitiveness. Commissioners can use this to identify disengaged users and reach out to improve participation.

```sql
SELECT Team.teamName,
       User.firstName,
       User.lastName,
       User.email
FROM Team
JOIN User ON Team.userID = User.userID
WHERE NOT EXISTS (
    SELECT 1
    FROM Transaction
    WHERE Transaction.teamID = Team.teamID
);
```

**Result:** *(paste query output here)*

---

### Q6 — League Standings Based on Win/Loss Record

**Description:** Calculate each team's wins and losses across all games within a league and sort by wins descending.

**Managerial Justification:** Standings are the primary measure of success in a fantasy league. This query powers leaderboards and determines playoff seeding.

```sql
SELECT Team.teamName,
       COUNT(CASE 
                WHEN Game.winnerTeamID = Team.teamID THEN 1 
            END) AS wins,
       COUNT(CASE 
                WHEN Game.winnerTeamID != Team.teamID
                     AND (Game.homeTeamID = Team.teamID 
                          OR Game.awayTeamID = Team.teamID)
                THEN 1 
            END) AS losses
FROM Team
JOIN Game 
  ON Team.teamID = Game.homeTeamID 
  OR Team.teamID = Game.awayTeamID
WHERE Team.leagueID = (
    SELECT League.leagueID 
    FROM League 
    LIMIT 1
)
GROUP BY Team.teamID, Team.teamName
ORDER BY wins DESC;
```

**Result:** *(paste query output here)*

---

### Q7 — Draft Round Efficiency: Average Points by Round

**Description:** Calculate the average fantasy points scored by players grouped by the draft round in which they were selected.

**Managerial Justification:** Reveals whether early round picks justify their draft position. If late-round picks outperform early ones, it signals poor draft strategy across the league — valuable insight for future drafts.

```sql
SELECT DraftPick.roundNumber,
       COUNT(DISTINCT DraftPick.playerID) AS playersDrafted,
       AVG(WeeklyStats.pointsScored) AS avgPointsPerRound
FROM DraftPick
JOIN WeeklyStats ON DraftPick.playerID = WeeklyStats.playerID
GROUP BY DraftPick.roundNumber
HAVING AVG(WeeklyStats.pointsScored) > 5
ORDER BY DraftPick.roundNumber ASC;
```

**Result:** *(paste query output here)*

---

### Q8 — Find Players at Skill Positions (QB, RB, WR, TE) Using REGEXP

**Description:** Retrieve all players whose position matches one of the four primary skill positions using a regular expression pattern.

**Managerial Justification:** Fantasy leagues often score skill positions differently. Filtering by position group lets managers compare position scarcity and depth across the league.

```sql
SELECT Player.playerName,
       Player.position,
       Player.nflTeam,
       AVG(WeeklyStats.pointsScored) AS avgPoints
FROM Player
JOIN WeeklyStats ON Player.playerID = WeeklyStats.playerID
WHERE Player.position REGEXP '^(QB|RB|WR|TE)$'
  AND WeeklyStats.pointsScored > 0
GROUP BY Player.playerID, Player.playerName, Player.position, Player.nflTeam
ORDER BY Player.position, avgPoints DESC;
```

**Result:** *(paste query output here)*

---

### Q9 — Users Who Manage Teams in Multiple Leagues

**Description:** Find users who are participating in more than one fantasy league simultaneously.

**Managerial Justification:** Power users who manage multiple teams are highly engaged. Identifying them helps platform administrators target premium feature offerings or provide support.

```sql
SELECT User.userID,
       User.firstName,
       User.lastName,
       User.email,
       COUNT(Team.teamID) AS numberOfTeams
FROM User
JOIN Team ON User.userID = Team.userID
GROUP BY User.userID, User.firstName, User.lastName, User.email
HAVING COUNT(Team.teamID) > 1
ORDER BY numberOfTeams DESC;
```

**Result:** *(paste query output here)*

---

### Q10 — Players Outperforming Their Own Season Average in the Latest Week

**Description:** Use a correlated subquery to find players whose most recent week's score exceeds their own personal season average.

**Managerial Justification:** Hot players — those trending above their own baseline — are prime trade targets and waiver wire pickups. This query surfaces players with current upward momentum.

```sql
SELECT Player.playerName,
       Player.position,
       Player.nflTeam,
       WeeklyStats.weekNumber,
       WeeklyStats.pointsScored AS latestWeekPoints
FROM Player
JOIN WeeklyStats ON Player.playerID = WeeklyStats.playerID
WHERE WeeklyStats.weekNumber = (
    SELECT MAX(weekNumber)
    FROM WeeklyStats
)
AND WeeklyStats.pointsScored > (
    SELECT AVG(WeeklyStats2.pointsScored)
    FROM WeeklyStats WeeklyStats2
    WHERE WeeklyStats2.playerID = Player.playerID
)
ORDER BY WeeklyStats.pointsScored DESC;
```

**Result:** *(paste query output here)*

---

## Database Information

- **Database Name:** `fantasy_football_db`
- **Platform:** MySQL
- **All queries are bookmarked as stored procedures** named `TP_Q1` through `TP_Q10`

---

## Additional Notes

- Each table is populated with at least 10+ rows of sample data to ensure queries return meaningful result sets
- Foreign key constraints are enforced to maintain referential integrity
- The data model was designed to be scalable across multiple seasons by including `seasonYear` in the League table
