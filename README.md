🏈 Fantasy Football League Database
Team Name: Group [Your Group Number]
Team Members
NameGitHubElher Zemihret[Link to GitHub]Allison Davis[Link to GitHub]Coen Cardellihttps://github.com/CoenCardelli/SQLCharlotte Holzapfel[Link to GitHub]

Scenario Description
Our database models a Fantasy Football League platform — a system where users join leagues, manage teams, draft real NFL players, and compete based on those players' weekly real-life performance.
Each League hosts multiple Users, and each User manages one or more Teams. Teams are built through a Draft, where players (real NFL athletes) are selected in rounds. Once the season begins, WeeklyStats track each player's points scored per week, which determines how teams perform in head-to-head Games. Users can also make Transactions (adding, dropping, or trading players) throughout the season.
The database supports tracking of league activity, draft history, player performance, team standings, and transaction records — everything a league manager would need to run and analyze a fantasy football season.

Data Model
Show Image
Explanation of the Data Model
Leagues sits at the top of the hierarchy. A league has many Teams, but each Team belongs to exactly one League (1:M). A league is identified by a unique leagueID and stores the league name and season year.
Users represents any person participating in the platform. A single User can manage multiple Teams across different leagues (1:M). Users are identified by userID and store basic contact info (name, email).
Teams is the central entity — it connects a User to a League via foreign keys userID and leagueID. Each Team participates in many Games, initiates many Transactions, and makes many DraftPicks (all 1:M).
Players represents a real NFL athlete. A Player can appear in many WeeklyStats records (one per week), many DraftPicks (across different leagues/seasons), and many Transactions (all 1:M). Players also have a teamID (the NFL team they play for) and a self-referencing captainID that can designate a team captain relationship.
WeeklyStats captures a Player's fantasy points for a specific week. Each record is tied to one Player and one week number, and stores pointsScored.
DraftPicks records which Team selected which Player, in what round and pick number. It links Team and Player (M:1 to each).
Games records a head-to-head matchup between two Teams for a given week, storing the score for each side and the winner. The homeTeamID, awayTeamID, and winnerTeamID are all foreign keys back to Teams.
Transactions logs adds, drops, or trades. Each transaction is tied to a Team and a Player, along with the type and the week it occurred.
What the database supports:

Tracking users, teams, leagues, and seasons
Recording full draft history
Storing player performance data week by week
Logging game results and standings
Managing roster transactions (adds, drops, trades)
Identifying team captains via self-referencing relationship on Players

What the database does NOT support:

Real-time data feeds or live scoring
Multiple positions per player per week
Salary cap or auction draft formats
Playoff bracket structures (beyond regular season games)


Data Dictionary
Leagues
ColumnData TypeKeyDescriptionleagueIDINTPKUnique identifier for each leagueleagueNameVARCHAR(45)Name of the fantasy leagueseasonYearVARCHAR(4)The NFL season year (e.g., '2024')
Users
ColumnData TypeKeyDescriptionuserIDINTPKUnique identifier for each userfirstNameVARCHAR(45)User's first namelastNameVARCHAR(45)User's last nameemailVARCHAR(45)User's email address
Teams
ColumnData TypeKeyDescriptionteamIDINTPKUnique identifier for each fantasy teamteamNameVARCHAR(45)Name of the fantasy teamuserIDINTFK (Users)The user who owns this teamleagueIDINTFK (Leagues)The league this team belongs to
Players
ColumnData TypeKeyDescriptionplayerIDINTPKUnique identifier for each NFL playerplayerNameVARCHAR(45)Full name of the NFL playerpositionVARCHAR(45)Player's position (QB, RB, WR, TE, K, DEF)teamIDINTFK (Teams)The NFL team the player is rostered oncaptainIDINTFK (Players)Self-referencing key to designate a captain relationship
WeeklyStats
ColumnData TypeKeyDescriptionstatsIDINTPKUnique identifier for each stats recordweekNumberINTNFL week number (1–18)pointsScoredINTFantasy points scored that weekplayerIDINTFK (Players)The player these stats belong to
DraftPicks
ColumnData TypeKeyDescriptionpickIDINTPKUnique identifier for each draft pickroundNumberINTThe draft round (e.g., 1–15)pickNumberINTThe overall pick numberplayerIDINTFK (Players)The player who was draftedteamIDINTFK (Teams)The team that made this pick
Games
ColumnData TypeKeyDescriptiongameIDINTPKUnique identifier for each gameweekNumberINTThe week the game was playedhomeTeamIDINTFK (Teams)The home/first teamawayTeamIDINTFK (Teams)The away/second teamhomeScoreINTFantasy points scored by home teamawayScoreINTFantasy points scored by away teamleagueIDINTFK (Leagues)The league this game belongs towinnerTeamIDINTFK (Teams)The team that won the game
Transactions
ColumnData TypeKeyDescriptiontransactionIDINTPKUnique identifier for each transactiontransactionTypeVARCHAR(45)Type: ADD, DROP, or TRADEweekNumberINTThe week the transaction occurredteamIDINTFK (Teams)The team making the transactionplayerIDINTFK (Players)The player involved in the transaction

Ten Queries
Query Feature Matrix
FeatureQ1Q2Q3Q4Q5Q6Q7Q8Q9Q10Multiple Table JoinXXXXXXXXSubqueryXXXCorrelated SubqueryXGROUP BYXXXXXXXXGROUP BY with HAVINGXXMulti-condition WHEREXXXXBuilt-in Functions / Calculated FieldXXXXXXXXXREGEXPXNOT EXISTSX

Q1 — Top 5 Players by Average Weekly Points
Description: Retrieve the top 5 NFL players ranked by their average fantasy points scored per week.
Managerial Justification: League managers and users want to know which players are the most consistently valuable. This query helps identify which players to target in trades or on the waiver wire.
sqlSELECT Players.playerName, Players.position,
       AVG(WeeklyStats.pointsScored) AS avgPoints
FROM Players
JOIN WeeklyStats ON Players.playerID = WeeklyStats.playerID
GROUP BY Players.playerID, Players.playerName, Players.position
ORDER BY avgPoints DESC
LIMIT 5;
Result: (paste query output here)

Q2 — Teams with More Than 3 Transactions in a Season
Description: Find all fantasy teams that have made more than 3 transactions (adds, drops, or trades) within a league.
Managerial Justification: Highly active teams may signal engaged users or desperate roster management. Commissioners can use this to monitor trade activity and flag potential abuse.
sqlSELECT Teams.teamName,
       Users.firstName,
       Users.lastName,
       COUNT(Transactions.transactionID) AS totalTransactions
FROM Teams
JOIN Users ON Teams.userID = Users.userID
JOIN Transactions ON Teams.teamID = Transactions.teamID
GROUP BY Teams.teamID, Teams.teamName, Users.firstName, Users.lastName
HAVING COUNT(Transactions.transactionID) > 3
ORDER BY totalTransactions DESC;
Result: (paste query output here)

Q3 — Players Drafted Who Scored Below League Average
Description: Identify drafted players whose average weekly points fall below the overall league average for all players.
Managerial Justification: Helps managers evaluate draft efficiency — if a team consistently drafts underperforming players, it signals poor draft strategy and could explain losing records.
sqlSELECT Players.playerName,
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
Result: (paste query output here)

Q4 — Weekly Points Scored by Each Team for a Specific Week
Description: Show the total fantasy points scored by each team during a given week (e.g., Week 5).
Managerial Justification: This is the core metric for determining weekly game outcomes. Managers use this data to compare performance and adjust rosters heading into the next week.
sqlSELECT Teams.teamName,
       SUM(WeeklyStats.pointsScored) AS totalPoints
FROM Teams
JOIN DraftPicks ON Teams.teamID = DraftPicks.teamID
JOIN WeeklyStats ON DraftPicks.playerID = WeeklyStats.playerID
WHERE WeeklyStats.weekNumber = 5
  AND WeeklyStats.pointsScored > 0
GROUP BY Teams.teamID, Teams.teamName
ORDER BY totalPoints DESC;
Result: (paste query output here)

Q5 — Teams That Have Never Made a Transaction
Description: Find all teams that have not made any adds, drops, or trades during the season.
Managerial Justification: Inactive teams hurt league competitiveness. Commissioners can use this to identify disengaged users and reach out to improve participation.
sqlSELECT Teams.teamName,
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
Result: (paste query output here)

Q6 — League Standings Based on Win/Loss Record
Description: Calculate each team's wins and losses across all games within a league and sort by wins descending.
Managerial Justification: Standings are the primary measure of success in a fantasy league. This query powers leaderboards and determines playoff seeding.
sqlSELECT Teams.teamName,
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
Result: (paste query output here)

Q7 — Draft Round Efficiency: Average Points by Round
Description: Calculate the average fantasy points scored by players grouped by the draft round in which they were selected.
Managerial Justification: Reveals whether early round picks justify their draft position. If late-round picks outperform early ones, it signals poor draft strategy across the league — valuable insight for future drafts.
sqlSELECT DraftPicks.roundNumber,
       COUNT(DISTINCT DraftPicks.playerID) AS playersDrafted,
       AVG(WeeklyStats.pointsScored) AS avgPointsPerRound
FROM DraftPicks
JOIN WeeklyStats ON DraftPicks.playerID = WeeklyStats.playerID
GROUP BY DraftPicks.roundNumber
HAVING AVG(WeeklyStats.pointsScored) > 5
ORDER BY DraftPicks.roundNumber ASC;
Result: (paste query output here)

Q8 — Find Players at Skill Positions (QB, RB, WR, TE) Using REGEXP
Description: Retrieve all players whose position matches one of the four primary skill positions using a regular expression pattern.
Managerial Justification: Fantasy leagues often score skill positions differently. Filtering by position group lets managers compare position scarcity and depth across the league.
sqlSELECT Players.playerName,
       Players.position,
       AVG(WeeklyStats.pointsScored) AS avgPoints
FROM Players
JOIN WeeklyStats ON Players.playerID = WeeklyStats.playerID
WHERE Players.position REGEXP '^(QB|RB|WR|TE)$'
  AND WeeklyStats.pointsScored > 0
GROUP BY Players.playerID, Players.playerName, Players.position
ORDER BY Players.position, avgPoints DESC;
Result: (paste query output here)

Q9 — Users Who Manage Teams in Multiple Leagues
Description: Find users who are participating in more than one fantasy league simultaneously.
Managerial Justification: Power users who manage multiple teams are highly engaged. Identifying them helps platform administrators target premium feature offerings or provide support.
sqlSELECT Users.userID,
       Users.firstName,
       Users.lastName,
       Users.email,
       COUNT(Teams.teamID) AS numberOfTeams
FROM Users
JOIN Teams ON Users.userID = Teams.userID
GROUP BY Users.userID, Users.firstName, Users.lastName, Users.email
HAVING COUNT(Teams.teamID) > 1
ORDER BY numberOfTeams DESC;
Result: (paste query output here)

Q10 — Players Outperforming Their Own Season Average in the Latest Week
Description: Use a correlated subquery to find players whose most recent week's score exceeds their own personal season average.
Managerial Justification: Hot players — those trending above their own baseline — are prime trade targets and waiver wire pickups. This query surfaces players with current upward momentum.
sqlSELECT Players.playerName,
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
Result: (paste query output here)

Database Information

Database Name: fantasy_football_db
Platform: MySQL
All queries are bookmarked as stored procedures named TP_Q1 through TP_Q10

Additional Notes

Each table is populated with at least 10+ rows of sample data to ensure queries return meaningful result sets
Foreign key constraints are enforced to maintain referential integrity
The data model was designed to be scalable across multiple seasons by including seasonYear in the Leagues table
captainID in the Players table is a self-referencing foreign key allowing designation of a team captain
