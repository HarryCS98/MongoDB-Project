# MongoDB Project for Premier League data

### Contents of project
- Data Set
- Database creation
- Database Query's

## Data Set

 - Data Set:
   https://github.com/HarryCS98/MongoDB-Project/blob/main/EPL_dataset.csv
   Data Set Key:
   https://github.com/HarryCS98/MongoDB-Project/blob/main/EPL_DataSet_Key.txt

## Database Creation

To create the database, I used mongoDB compass compared to Robot3T (which I am using to query the database) as Robot3T does not support the importing of CSV files. So, to import the data set I first created a database in mongoDB compass. I then used the add data button to import the csv file to create the database.

## Database Query's



## 1.

### Question:

Show all the EPL teams involved in the season.

### Method:

To display all EPL teams involved in the season we need to find all unique teams. Now both HomeTeam and AwayTeam fields contain a list of all the team names as all teams must play at least one home and away match so we can simply use the distinct function to get all distinct values and pass in the HomeTeam field.

### Final query:

    db.epl.distinct("HomeTeam");


### Screenshot of answer:

[![Image from Gyazo](https://i.gyazo.com/31f406986688f1561f8e76e34bb19ec1.png)](https://gyazo.com/31f406986688f1561f8e76e34bb19ec1)

## 2.

### Question:

How many matches were played on Mondays?

### Method:

To find how many matches where played on a Monday we first group all the documents together by day to do this we set the _id equal to our new field called day. Then we create this field then we set its value to the day of the week to do this we use the dayOfWeek function but this only takes in a date data type so we first need to turn our string date into ISO date format and a date data type to do this we use the dateFromString function we then pass in our date field and then set the format the date is in so in our case its in day then month then year. Then we need to count how many documents are in each group (each group is grouped on the day of the week the match was played). So, we then create a new field called amount and we set this equal to sum:1 which just counts up by one every time a document is added to that group. Now we have done that we now have the output of how many matches where played on every day but we only want to see how many matches where played on a Monday so we then use the match function and pass in the day field using _id.day and set this to 2 which is the value for Monday. (The days of the week function outputs days in numerical form 1 = Sunday, 2= Monday, 3=Tuesday, 4= Wednesday, 5=Thursday, 6= Friday,7= Saturday). This then only gives us the amount of matches that was played on a Monday.

### Final query:

    db.epl.aggregate([{$group: {_id:{"day": {$dayOfWeek:{"$dateFromString": { "dateString": "$Date","format": "%d/%m/%Y"}}}}, amount:{$sum:1}}}, {$match:{"_id.day": 2}}]);

### Screenshot of answer:

[![Image from Gyazo](https://i.gyazo.com/1176de382c9ddafda10361a7d32b47be.png)](https://gyazo.com/1176de382c9ddafda10361a7d32b47be)



## 3.

### Question:

Display the total number of goals “Liverpool” had scored and conceded in the season.

### Method:

To calculate the number of goals Liverpool has scored and conceded in the season. We first need to get all the matches that Liverpool played in so we use the match command combined with an or operator we used this to match to all the documents that have Liverpool as the hometeam or the away team. Then we used the facet command this allows us to have two aggregation pipelines so we can calculate the amount of goals scored and conceded for Liverpool as a hometeam and as an away team separately. First, we create a pipeline called totalAmountHome this will calculate goals scored and conceded when Liverpool was playing home games. So first we match to any to liverpool. Then we use the project command to create two new fields one called HomeGoals and another called AwaysGoals as liverpool is playing at home in this pipleline homegoals will be goals scored and awaygoals will be goals conceded to get these values we use the toint function on the FTHG(full time home goals) and toint on FTAG (full time away goals). We then group all these documents together adding up all HomeGoals and Awaygoals into new fields called totalHomeGoals and TotalawayGoals. After this we moved on to all the away matches Liverpool played. We again match to liverpool and then we use the project function again to create the homegoals and awaygoals fields these again use the toint function to get the values from FTAG and FTHG but this time we put the FTAG in the HomeGoals field and the FTHG in the awaygoals field as liverpool where playing away therefor a homegoal would actually be an awaygoal. then again group the data adding all the Homegoals together into a new field called TotalHomeGoals and then Awaygoals into a field called TotalAwayGoals. Next we need to use another project command this is then where we group all the goals scored and conceded by liverpool in both home and away matches. To do this we create a new field called totalGoalsScored and then use the concatArrays function to add the two arrays together and to access those arrays we use the dot notation. So we first get the homegoals from the homegames using totalAmountHome.totalHomeGoals and then we get all the goals liverpool scored away by using the command totalAmountAway.totalHomeGoals (remembering that home goals is actually the goals liverpool scored while playing as the away team as we switch FTHG and FTAG). Then we do the same for goals conceded by creating a new field totalGoalsConceded and using the concatArrys function on totalawaygoals from both home and away games (rembering we switch the FTHG and FTAG on the away team pipeline). This a list of documents each holding the number of either goals scored or conceded for each match but we now need to add them together and get a total number to do this we first have to unwind both our totalGoalsScored and totalGoalsConceded arrays using the unwind function. Then we group totalGoalsscored into a field called GoalsScored and totalGoalsConceded into a new field called GoalsConceded adding all the goals together in the process using the sum function. Then the results where for some reason doubled so then we used the addFields function to override the GoalsConceded and GoalsScored fields and set the value to the original GoalsConceded and GoalsScored divided by two.

### Final query:

    db.epl.aggregate([{$match: {$or: [{"HomeTeam": "Liverpool"},{"AwayTeam": "Liverpool"}]}},{$facet: {totalAmountHome: [{$match: {"HomeTeam": "Liverpool"}},{$project: {HomeGoals: {$toInt: "$FTHG"},AwayGoals: {$toInt: "$FTAG"}}},{$group: {_id: null,totalHomeGoals: {$sum: "$HomeGoals"},totalawayGoals: {$sum: "$AwayGoals"}}},],totalAmountAway: [{$match: {"AwayTeam": "Liverpool"}},{$project: {HomeGoals: {$toInt: "$FTAG"},AwayGoals: {$toInt: "$FTHG"}}},{$group: {_id: null,totalHomeGoals: {$sum: "$HomeGoals"},totalawayGoals: {$sum: "$AwayGoals"}}},]},},{"$project": {"totalGoalsScored": {"$concatArrays": ["$totalAmountHome.totalHomeGoals","$totalAmountAway.totalHomeGoals"]},"totalGoalsConceded": {"$concatArrays": ["$totalAmountHome.totalawayGoals","$totalAmountAway.totalawayGoals"]},},},{$unwind: "$totalGoalsScored"},{$unwind: "$totalGoalsConceded"},{$group: {"_id": "null","GoalsConceded": {$sum: "$totalGoalsConceded"},"GoalsScored": {$sum: "$totalGoalsScored"}}},{"$addFields": {"GoalsConceded": {"$divide": ["$GoalsConceded",2]},"GoalsScored": {"$divide": ["$GoalsScored",2]}}}])


### Screenshot of answer:
[![Image from Gyazo](https://i.gyazo.com/c886ee677d4810e253eb24a58df45c13.png)](https://gyazo.com/c886ee677d4810e253eb24a58df45c13)




## 4.
### Question:

Which teams have the most and least shots in the season?

### Method:

The first step in finding which team had the most and least shots in a season is to use the facet function this allows us to do multiple piplines in one query. Here we do two pipelines one called totalAmountHome and one called totalAmountAway. TotalAmountHome pipeline is used to group all hometeams by team name and create a new field called shots which uses the sum function to add up all teams shots to do this we pass in the HS (home shots) to the sum function after passing it to the toint function to turn it from a string to an int. The process is then repeated for the totalAmountAway pipeline we grouped the teams based on team name and then added the each teams total shots for away games up into a new field called shots using the sum function and the AS (away shots) field. After this we use the project command to create a new field called totalshots and then use the concatArrays function to combine the totalAmountHome and totalAmountAway arrays together then this new array totalshots in unwinded using the unwind function then the a group function is applied grouping the data on team name using the totalshots._id (_id now holds the team name) and then a new field called Shots is created this field value is set to each teams shots combined using the sum function then the shots are ordered from biggest to smallest and then finally a final group is done and we use the first and last functions to get the first team in the array which have the most shots and the last team which have the least shots.

### Final query:

    db.epl.aggregate([{$facet: { totalAmountHome: [{ $group: { _id: "$HomeTeam"  , shots:{$sum:{$toInt:"$HS"}} }},],totalAmountAway: [{ $group: { _id: "$AwayTeam", shots:{$sum:{$toInt:"$AS"}}}},] , },} ,{ "$project" : {'totalshots' : { "$concatArrays" : [ '$totalAmountHome', '$totalAmountAway' ] },},  },  {$unwind: '$totalshots'},{$group : {_id : "$totalshots._id",Shots : { $sum : "$totalshots.shots"},}},{$sort: {"Shots" : -1}},{$group : {_id : "",MostShotsTeam : {$first : "$_id"},MostShots: { $first : "$Shots"},LeastShotsTeam : {$last : "$_id"},LeastShots: {$last : "$Shots"}}},  ])

### Screenshot of answer:

[![Image from Gyazo](https://i.gyazo.com/e720749709234d93d7129b5a2adf9679.png)](https://gyazo.com/e720749709234d93d7129b5a2adf9679)



## 5.

### Question:

Who refereed the most matches?

### Method:

To find who refereed the most matches we first group all documents together on the referee field this then group all the matches that where refereed by the same referee together we then create a new field this is called AmountOfMatches this uses sum:1 this then increase the value by one every time a document is grouped into a group so this leaves us with the total amount of documents in each group and as each group is one referee we can simply then sort the groups using AmountOfMatches field and then limit this to 1 giving us the referee with the highest amount of matches refereed.

### Final query:

    db.epl.aggregate([{$group: {_id : "$Referee", AmountOfMatches: {$sum: 1}}},{$sort: {AmountOfMatches: -1}},{$limit: 1}]);

### Screenshot of answer:

[![Image from Gyazo](https://i.gyazo.com/ae90abc4bd09016e4d85092c1339bffa.png)](https://gyazo.com/ae90abc4bd09016e4d85092c1339bffa)



## 6.

### Question:

How many matches “Arsenal” won as the away team?

### Method:

To find out how many matches Arsenal won as an away team we can simply use the find function to find AwayTeams that have a value of Arsenal and also have a FTR (full time result) value of A and if both of these match then the away team was Arsenal and with a full time result of A which means away team won we have all the away matches Arsenal won and then finally we can simply use the count function to count how many documents we matched to.

### Final query:

    db.epl.find({AwayTeam: "Arsenal",FTR : "A" }).count();

### Screenshot of answer:

[![Image from Gyazo](https://i.gyazo.com/15da57e4415d4448fd3808036abf08ec.png)](https://gyazo.com/15da57e4415d4448fd3808036abf08ec)



## 7.

### Question:

Display all the matches that “Man United” lost.

### Method:

To find all the matches Man United lost we need to search both the HomeTeam and AwayTeam fields for all the matches Man United played in then check to see if the FTR or full time result was the opposite so if Man United was the home team we needed the FTR to be equal to A as this mean the away team won and then if Man United was the away team we needed the FTR to be set to H as this means the home team won. To do this we used the find function then we use the or operator this means if either condition is met it will match to the document the first condition we added was that Man United was the Home team and the FTR was A and then for our second condition we check to see if Man United was the away team and the FTR was H.

### Final query:

    db.epl.find({$or:[{"HomeTeam": "Man United", "FTR" : "A"},{"AwayTeam": "Man United", "FTR" : "H"}]});

### Screenshot of answer:
[![Image from Gyazo](https://i.gyazo.com/5b3db86b13e48290cdd27c7dda88f8ce.png)](https://gyazo.com/5b3db86b13e48290cdd27c7dda88f8ce)


## 8.

### Question:

Display all matches that “Liverpool” won but were down in the first half

### Method:

To display all matches Liverpool won but were down in the first half we again used the find function we then passed in the or operator as we need to check the home and away games to see if Liverpool won but was down in the first half. For our first condition in our find function we checked to see if Liverpool was the home team the FTR (full time result) was equal to H as we need to see games that Liverpool won and as H means the Home team won this means Liverpool won. Then we check to see if the HTR (half time result) was equal to A this means the away team was leading at half time which would mean the other team was winning at half time. Then for our second condition we checked to see if Liverpool was the away team and then the FTR (full time result) was A as this would mean Liverpool won the game and then we check to see if the HTR (half time result) was equal to H as this would mean the home team was winning at half time.

### Final query:

    db.epl.find({$or:[{"HomeTeam": "Liverpool", "FTR" : "H", "HTR": "A"},{"AwayTeam": "Liverpool", "FTR" : "A", "HTR": "H"}]});

### Screenshot of answer:

[![Image from Gyazo](https://i.gyazo.com/26f0ddc9b0316ee9a397c9bcb57e7922.png)](https://gyazo.com/26f0ddc9b0316ee9a397c9bcb57e7922)



## 9.

### Question:

Write a query to display the final ranking of all the teams based on their total points.

### Method:

To Determine the amount of points each team had and then rank them we first need to calculate the points each team received to doing this we will give 3 points to a team that won 1 point to a team that drawed and finally 0 points to teams who lost. To do this we first use the facet function which allows us to do multiple pipelines in one query. First we calculate the points for home match wins we do this my matching to FTR(full time result) equal to H then we group on the hometeam field (the name of the teams) we then create a new fields called points which adds up home many times a team has won using the sum:1 method. Then we use the addfields method to replace the value in the points field to a new value which is the original points value multiplied by 3 as that’s how many points are given for a win. The process is then repeated for the AwayMatches pipeline apart from this time we match to any FTR that are equal to A and then group on the awayteam field we then go through the same process and the homematches pipeline and multiply the amount of matches each team won by 3. We now need to calculate points for drawed matches to do this we create two new pipelines called AwayMatchesDraw and HomeMatchesDraw these then both match to all FTR that have a value of D for draw but AwayMatchesDraw groups on the AwayTeam field and records the amount of times each of these team drawed as an away team in the created points field and the HomeMatchesDraw group on the HomeTeam field which again keep track of how many matches each team drawed as a home team in the points field. These do not need to be multiplied by anything as you get a single point for a draw and losses don’t need to be calculated as no points are allocated for a loss. Next we use a project function to create a new field called TeamPoints this uses the concatArrays to join HomeMatches, AwayMatches, AwayMatchesDraw and finally HomeMatchesDraw all together we then unwind this new TeamPoints array and then finally group this data by team name using TeamPoints._id and then add all the teams points up in a new Points field by using the sum function which when each team is grouped together adds there points up. Then finally we use the Sort method pass in the Points field and use the value of -1 this sorts the results from highest amount of points to the lowest amounts of points so the team with the highest amount of points will be at the top of the list and the team with the lowest amount of points will be at the bottom of the list.

### Final query:

    db.epl.aggregate([{$facet:{HomeMatches:[{$match:{"FTR":"H"}},{$group:{_id:"$HomeTeam", Points:{$sum: 1}}}, {"$addFields":{"Points" :{$multiply:["$Points", 3]}}}],AwayMatches:[{$match:{"FTR":"A"}},{$group:{_id:"$AwayTeam", Points:{$sum: 1}}}, {"$addFields":{"Points" :{$multiply:["$Points", 3]}}}],AwayMatchesDraw:[{$match:{"FTR":"D"}},{$group:{_id:"$AwayTeam", Points:{$sum: 1}}}],HomeMatchesDraw:[{$match:{"FTR":"D"}},{$group:{_id:"$HomeTeam", Points:{$sum: 1}}}]}},{ "$project" : {  'TeamPoints' : { "$concatArrays" : [ '$HomeMatches', '$AwayMatches', '$AwayMatchesDraw', '$HomeMatchesDraw'  ] },}},  {$unwind: '$TeamPoints'},{$group : {_id : "$TeamPoints._id",Points : { $sum:{$toInt:"$TeamPoints.Points"}},}},  {$sort: {"Points": -1}}]);

### Screenshot of answer:

[![Image from Gyazo](https://i.gyazo.com/b5cba569c01d201b6b9d909c67c63253.png)](https://gyazo.com/b5cba569c01d201b6b9d909c67c63253)
