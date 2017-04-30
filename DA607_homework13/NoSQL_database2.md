Migrating database from MySQL to Neo4j
================
Yun Mai
April 30, 2017

NoSQL migration
---------------

Assignment:

For this assignment, you should take information from a relational database and migrate it to a NoSQL database of your own choosing.

For the relational database, you might use the flights database, the tb database, the "data skills" database your team created for Project 3, or another database of your own choosing or creation.

For the NoSQL database, you may use MongoDB (which we introduced in week 7), Neo4j, or another NoSQL database of your choosing.

Your migration process needs to be reproducible. R code is encouraged, but not required. You should also briefly describe the advantages and disadvantages of storing the data in a relational database vs. your NoSQL database.

Overview:

I will migrate the movie\_rating database generated in week\_2 homework to Neo4j. Nodes will be created first followed by buiding relationships. At last the graph will be viewed with visualization tool.

``` r
devtools::install_github("nicolewhite/Rneo4j") 
install.packages("RMySQL")
```

``` r
library(RMySQL)
```

    ## Warning: package 'RMySQL' was built under R version 3.3.3

    ## Loading required package: DBI

``` r
library(RNeo4j)
library(knitr)
```

Get the Data from MySQL
-----------------------

``` r
# connect to MySQL local database
movie_con <- dbConnect(MySQL(), user="root", password=mysql_pw, dbname = "movie_nosql", host = "localhost")
```

``` r
#Send query to MySQL and retrieve the movie data from MySQL.
query.movie <- dbSendQuery(movie_con, "SELECT * FROM movie")
movie.data <- dbFetch(query.movie, n=-1)

#Send query to MySQL and retrieve the friends data from MySQL. Friends are the people who rated the movies
query.friends <- dbSendQuery(movie_con, "SELECT * FROM friends")
friends.data <- dbFetch(query.friends, n=-1)

#Send query to MySQL and retrieve the rating data from MySQL.
query.rating <- dbSendQuery(movie_con, "SELECT * FROM rating")
rating.data <- dbFetch(query.rating, n=-1)

#send query to MySQL and retrieve the actor data from MySQL
query.actor <- dbSendQuery(movie_con, "SELECT * FROM actor")
actor.data <- dbFetch(query.actor, n=-1)

kable(head(rating.data, n=5), caption = "rating - the main table")
```

| row\_names |  MovieID| MovieName                    |  FriendID| FriendName |  FriendRating|
|:-----------|--------:|:-----------------------------|---------:|:-----------|-------------:|
| 1          |        1| The Fate of the Furious      |         1| Ming       |           2.0|
| 2          |        2| Star Wars: The Last Jedi     |         1| Ming       |           4.0|
| 3          |        3| Guardians of the Galaxy Vol. |         1| Ming       |           4.0|
| 4          |        4| Thor: Ragnarok               |         1| Ming       |           1.5|
| 5          |        5| Beauty and the Beast         |         1| Ming       |           4.0|

``` r
kable(head(movie.data, n=5),caption = "movie")
```

| row\_names |  MovieID| MovieName                    |
|:-----------|--------:|:-----------------------------|
| 1          |        1| The Fate of the Furious      |
| 2          |        2| Star Wars: The Last Jedi     |
| 3          |        3| Guardians of the Galaxy Vol. |
| 4          |        4| Thor: Ragnarok               |
| 5          |        5| Beauty and the Beast         |

``` r
kable(head(friends.data, n=5), caption = "friends")
```

| row\_names |  FriendsID| FriendsName |
|:-----------|----------:|:------------|
| 1          |          1| Ming        |
| 2          |          2| Hao         |
| 3          |          3| Alison      |
| 4          |          4| Eran        |
| 5          |          5| Orshi       |

``` r
kable(head(actor.data, n=5),caption = "actor")
```

| row\_names | actors          | title                    |  actorID|
|:-----------|:----------------|:-------------------------|--------:|
| 1          | Scott Eastwood  | The Fate of the Furious  |        1|
| 2          | Charlize Theron | The Fate of the Furious  |        2|
| 3          | Dwayne Johnson  | The Fate of the Furious  |        3|
| 4          | Vin Diesel      | The Fate of the Furious  |        4|
| 5          | Tom Hardy       | Star Wars: The Last Jedi |        5|

**Export data as .csv files**

``` r
write.csv(movie.data, "movie.csv")
write.csv(friends.data, "friends.csv")
write.csv(rating.data, "rating.csv")
write.csv(actor.data, "actor.csv")
```

**Disconnect from MySQL**

``` r
dbDisconnect(movie_con)
```

    ## Warning: Closing open result sets

    ## [1] TRUE

Load the data to Neo4j
----------------------

**Download .csv files**

``` r
friends_d <- read.csv(file="https://raw.githubusercontent.com/YunMai-SPS/DA607/master/DA607_homework13/friends.csv", header=TRUE, sep=",", stringsAsFactors = F)

movie_d <- read.csv(file="https://raw.githubusercontent.com/YunMai-SPS/DA607/master/DA607_homework13/movie.csv", header=TRUE, sep=",", stringsAsFactors = F)

rating_d <- read.csv(file="https://raw.githubusercontent.com/YunMai-SPS/DA607/master/DA607_homework13/rating.csv", header=TRUE, sep=",", stringsAsFactors = F)

actor_d <- read.csv(file="https://raw.githubusercontent.com/YunMai-SPS/DA607/master/DA607_homework13/actor.csv", header=T, sep=",", stringsAsFactors = F)
```

Migration to Neo4j
------------------

1. Create graph in Neo4j
------------------------

``` r
#connect to Neo4j
graph = startGraph("http://localhost:7474/db/data", username = "neo4j", password = neo4j.pw)

#could be used to delete all nodes, relationships, indexes, and constraints from the graph database
clear(graph, input = F)

# two kinds of nodes will be created: movie and person. the same nodes will be merged.

query = "
         MERGE (movie:movie {name: {title1}, id:{movieid1}})

         CREATE (audience:person {name:{audname},id1:{audid},id2:{movieid2}, rating:{friendrating}})
         CREATE (audience)-[r:RATED]->(movie)
         SET r.rating = {friendrating}
"

tx = newTransaction(graph)
for (i in 1:nrow(rating_d)){
    appendCypher(tx,query,
                title1 = rating_d$MovieName[i],
                movieid1 = rating_d$MovieID[i],
                audname = rating_d$FriendName[i],
                audid = rating_d$FriendID[i],
                movieid2 = rating_d$MovieID[i],
                friendrating = rating_d$FriendRating[i])
               
}
commit(tx)


query = "
         CREATE (actor:person {name:{actname}, id:{actorid}), title:{movietitle})
         MERGE (movie:movie, {movie:{title1}})
         CREATE (actor)-[a:ACTED_IN]->(movie)
"
for (i in 1:nrow(actor_d)){
                query
                actname = actor_d$actors[i]
                actorid = actor_d$actorID[i]
                movietitle = actor_d$title[i]
                title1 = actor_d$title[i]
}

summary(graph)
```

    ##     This    To  That
    ## 1 person RATED movie

**The actor table and the rating(main) table could not be merged into one since they have different length. So I tried a different way to create graphes for audiences-rate-movies, actors-act-movies graphes separately.**

2.1 Create Nodes with RNeo4j functions
--------------------------------------

``` r
for (i in 1:nrow(rating_d)){
    movie.node <- createNode(graph, "movie", name=rating_d$MovieName[i], id=rating_d$MovieID[i])
}

for (i in 1:nrow(rating_d)){
    audience.node <- createNode(graph, "audience", name=rating_d$FriendName[i], id=rating_d$FriendID[i], movieid = rating_d$MovieID[i])
}

for (i in 1:nrow(actor_d)){
    actor.node <- createNode(graph, "actor", name=actor_d$actors[i], id=actor_d$actorID[i], movietitle = actor_d$title[i])
}
```

2.2Create the relationship with RNeo4j functions
------------------------------------------------

\*\* 2.2.1. relationship between audiences and movies\*\*

``` r
audience_query = "MATCH (a:movie), (b:audience) WHERE (a.id) = (b.movieid)  RETURN a"
for (i in 1:nrow(rating_d)) {
    movie_rated <- getSingleNode(graph, audience_query, name=rating_d$MovieName[i])
    createRel(audience.node, "RATED", movie_rated, rating = rating_d$FriendRating[i])
}
```

**2.2.2 relationship between actors and movies**

``` r
actor_query = "MATCH (a:movie), (c:actor) WHERE (a.name) = (c.movietitle)  RETURN c"
for (i in 1:nrow(rating_d)) {
    movie_related <- getSingleNode(graph, actor_query, name=actor_d$actors[i])
    createRel(actor.node, "ACTED_IN", movie_related)
}

summary(graph)
```

    ##       This       To  That
    ## 1 audience    RATED movie
    ## 2   person    RATED movie
    ## 3    actor ACTED_IN actor

tabular results could be returned by using Cypher. As introduced in Nicole White's GitHub RNeo4j repo "<https://github.com/nicolewhite/RNeo4j#nodes>"

``` r
query = "
MATCH (people:Person)-[relatedTo]-(:Movie {title: 'Beauty and the Beast'}) 
RETURN people.name, Type(relatedTo), relatedTo"

cypher(graph, query)
```

``` r
query = "
MATCH (Ming:person)-[r:RATED]->(m:movie)
WHERE Ming.name = 'Ming'
RETURN Ming.name, r.rating, m.title
"
cypher(graph, query)
```

    ##   Ming.name r.rating m.title
    ## 1      Ming      2.0      NA
    ## 2      Ming      4.0      NA
    ## 3      Ming      4.0      NA
    ## 4      Ming      1.5      NA
    ## 5      Ming      4.0      NA
    ## 6      Ming      5.0      NA

When returning relative more complicated results:

``` r
query = "
MATCH (Ming:person)-[r:RATED]->(m:movie)
WHERE Ming.name = 'Ming'
RETURN Ming.name, COLLECT(m.movie) AS seen
"

cypherToList(graph, query)
```

    ## [[1]]
    ## [[1]]$Ming.name
    ## [1] "Ming"
    ## 
    ## [[1]]$seen
    ## list()

Parameters can be passed

``` r
query = "
MATCH (p1:Person)-[r:RATED]->(m1:movie)
WHERE p1.name = {name1} AND m1:movie = {title1}
RETURN p1.name, r.weight, m1:movie
"

cypher(graph, query, name1="Hao", name2="The Fate of the Furious")
```

Browse Neo4j Graph
------------------

``` r
graph = startGraph("http://localhost:7474/db/data/", username="neo4j", password=neo4j.pw)
browse(graph)
```

My Rstudio somehow cannot show Neo4j browser. Cypher clauses will be put in Neo4j to build the database.
