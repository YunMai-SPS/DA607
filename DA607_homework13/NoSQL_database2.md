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
```

    ## Warning in .local(conn, statement, ...): Unsigned INTEGER in col 3 imported
    ## as numeric

``` r
actor.data <- dbFetch(query.actor, n=-1)

#send query to MySQL and retrieve the director data from MySQL
query.director <- dbSendQuery(movie_con, "SELECT * FROM director")
director.data <- dbFetch(query.director, n=-1)
#send query to MySQL and retrieve the writer data from MySQL
query.writer <- dbSendQuery(movie_con, "SELECT * FROM writer")
writer.data <- dbFetch(query.writer, n=-1)

#send query to MySQL and retrieve the genre data from MySQL
query.genre <- dbSendQuery(movie_con, "SELECT * FROM genre")
genre.data <- dbFetch(query.genre, n=-1)

#send query to MySQL and retrieve the country data from MySQL
query.country <- dbSendQuery(movie_con, "SELECT * FROM country")
country.data <- dbFetch(query.country, n=-1)

#send query to MySQL and retrieve the country data from MySQL
query.omdb <- dbSendQuery(movie_con, "SELECT * FROM omdb")
omdb.data <- dbFetch(query.omdb, n=-1)


kable(head(rating.data, n=7), caption = "Rating - the main table")
```

|  row\_names|  MovieID| MovieName                |  FriendID| FriendName |  FriendRating|
|-----------:|--------:|:-------------------------|---------:|:-----------|-------------:|
|           1|        1| The Fate of the Furious  |         1| Ming       |           2.0|
|           7|        1| The Fate of the Furious  |         2| Hao        |           1.0|
|          13|        1| The Fate of the Furious  |         3| Alison     |           4.5|
|          19|        1| The Fate of the Furious  |         4| Eran       |           3.0|
|          25|        1| The Fate of the Furious  |         5| Orshi      |           2.5|
|           2|        2| Star Wars: The Last Jedi |         1| Ming       |           4.0|
|           8|        2| Star Wars: The Last Jedi |         2| Hao        |           4.5|

``` r
kable(head(movie.data, n=6),caption = "Movie Table")
```

|  row\_names|  MovieID| MovieName                    |
|-----------:|--------:|:-----------------------------|
|           1|        1| The Fate of the Furious      |
|           2|        2| Star Wars: The Last Jedi     |
|           3|        3| Guardians of the Galaxy Vol. |
|           4|        4| Thor: Ragnarok               |
|           5|        5| Beauty and the Beast         |
|           6|        6| Logan                        |

``` r
kable(head(friends.data, n=5), caption = "Audience Table")
```

|  row\_names|  FriendsID| FriendsName |
|-----------:|----------:|:------------|
|           1|          1| Ming        |
|           2|          2| Hao         |
|           3|          3| Alison      |
|           4|          4| Eran        |
|           5|          5| Orshi       |

``` r
kable(head(actor.data, n=5),caption = "Actor Table")
```

| row\_names | actors          | title                    |  actorID|
|:-----------|:----------------|:-------------------------|--------:|
| 1          | Scott Eastwood  | The Fate of the Furious  |        1|
| 2          | Charlize Theron | The Fate of the Furious  |        2|
| 3          | Dwayne Johnson  | The Fate of the Furious  |        3|
| 4          | Vin Diesel      | The Fate of the Furious  |        4|
| 5          | Tom Hardy       | Star Wars: The Last Jedi |        5|

``` r
kable(head(director.data, n=5),caption = "Director Table")
```

|  row\_names| directors     | title                        |  directorID|
|-----------:|:--------------|:-----------------------------|-----------:|
|           1| F. Gary Gray  | The Fate of the Furious      |           1|
|           2| Rian Johnson  | Star Wars: The Last Jedi     |           2|
|           3| James Gunn    | Guardians of the Galaxy Vol. |           3|
|           4| Taika Waititi | Thor: Ragnarok               |           4|
|           5| Bill Condon   | Beauty and the Beast         |           5|

``` r
kable(head(writer.data, n=5),caption = "Writer Table")
```

|  row\_names| writers             | title                        |  writerID|
|-----------:|:--------------------|:-----------------------------|---------:|
|           1| Chris Morgan        | The Fate of the Furious      |         1|
|           2| Gary Scott Thompson | The Fate of the Furious      |         2|
|           3| Rian Johnson        | Star Wars: The Last Jedi     |         3|
|           4| George Lucas        | Star Wars: The Last Jedi     |         4|
|           5| James Gunn          | Guardians of the Galaxy Vol. |         5|

``` r
kable(head(genre.data, n=3),caption = "Genre Table")
```

|  row\_names| genres   | title                   |  genreID|
|-----------:|:---------|:------------------------|--------:|
|           1| Action   | The Fate of the Furious |        1|
|           2| Crime    | The Fate of the Furious |        2|
|           3| Thriller | The Fate of the Furious |        3|

``` r
kable(head(country.data, n=3),caption = "Country Table")
```

|  row\_names| countries | title                   |  countryID|
|-----------:|:----------|:------------------------|----------:|
|           1| USA       | The Fate of the Furious |          1|
|           2| France    | The Fate of the Furious |          2|
|           3| Canada    | The Fate of the Furious |          3|

``` r
kable(head(omdb.data, n=3),caption = "Omdb Table")
```

|  row\_names| Title                          |  Year| imdbID    | genre                      | actor                                                     | country                    | director     | writer                                                                                                                                                                                                                                                                                           |  MovieID|
|-----------:|:-------------------------------|-----:|:----------|:---------------------------|:----------------------------------------------------------|:---------------------------|:-------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------:|
|           1| The Fate of the Furious        |  2017| tt4630562 | Action,Crime,Thriller      | Scott Eastwood,Charlize Theron,Dwayne Johnson,Vin Diesel  | USA,France,Canada,UK,Samoa | F. Gary Gray | Chris Morgan,Gary Scott Thompson (based on characters created by)                                                                                                                                                                                                                                |        1|
|           2| Star Wars: The Last Jedi       |  2017| tt2527336 | Action, Adventure, Fantasy | Tom Hardy, Daisy Ridley, Adam Driver, Mark Hamill         | USA                        | Rian Johnson | Rian Johnson (screenplay), George Lucas (characters)                                                                                                                                                                                                                                             |        2|
|           3| Guardians of the Galaxy Vol. 2 |  2017| tt3896198 | Action, Adventure, Sci-Fi  | Chris Pratt, Karen Gillan, Vin Diesel, Sylvester Stallone | USA                        | James Gunn   | James Gunn, Dan Abnett (based on the Marvel comic book by), Andy Lanning (based on the Marvel comic book by), Stan Lee (characters), Jack Kirby (characters), Gene Colan (characters), Arnold Drake (characters), Steve Englehart (characters), Steve Gan (characters), Jim Starlin (characters) |        3|

![MySQL diagram](https://raw.githubusercontent.com/YunMai-SPS/DA607/master/DA607_homework13/MySQL_diagram.png)

**Export data as .csv files**

``` r
write.csv(movie.data, "movie.csv")
write.csv(friends.data, "friends.csv")
write.csv(rating.data, "rating.csv")
write.csv(actor.data, "actor.csv")
write.csv(director.data, "director.csv")
write.csv(writer.data, "writer.csv")
write.csv(genre.data, "genre.csv")
write.csv(country.data, "country.csv")
write.csv(omdb.data, "omdb.csv")
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

director_d <- read.csv(file="https://raw.githubusercontent.com/YunMai-SPS/DA607/master/DA607_homework13/director.csv", header=TRUE, sep=",", stringsAsFactors = F)

writer_d <- read.csv(file="https://raw.githubusercontent.com/YunMai-SPS/DA607/master/DA607_homework13/writer.csv", header=TRUE, sep=",", stringsAsFactors = F)

genre_d <- read.csv(file="https://raw.githubusercontent.com/YunMai-SPS/DA607/master/DA607_homework13/genre.csv", header=TRUE, sep=",", stringsAsFactors = F)

country_d <- read.csv(file="https://raw.githubusercontent.com/YunMai-SPS/DA607/master/DA607_homework13/country.csv", header=TRUE, sep=",", stringsAsFactors = F)

omdb_d <- read.csv(file="https://raw.githubusercontent.com/YunMai-SPS/DA607/master/DA607_homework13/omdb.csv", header=TRUE, sep=",", stringsAsFactors = F)
```

Migration to Neo4j
------------------

1. Create graph in Neo4j
------------------------

``` r
#connect to Neo4j
graph = startGraph("http://localhost:7474/db/data", username = "neo4j", password = neo4j.pw)

#delete all nodes, relationships, indexes, and constraints from the graph database
clear(graph, input = F)

# nodes to be created: 
# Nodes:
#   movie              
#   audience          
#   actor             
#   director                
#   writer              
#   genre
#   country
# the same nodes will be merged.


query = "
         MERGE (movie:Movie {name: {title1}, id:{movieid1}})
         CREATE (audience:Audience {name:{audname},id1:{audid},id2:{movieid2}, rating:{friendrating}})
         CREATE (audience)-[r:RATED]->(movie)
         SET r.rating = {friendrating}
         RETURN audience,movie         
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

summary(graph)
```

    ##       This    To  That
    ## 1 Audience RATED Movie

I used MATCH clauses in the above query to create relationships but error occured. It seemed that Neo4j did not allow to have multiple statements within a query. Then I removed MATCH clauses but only kept CREAT clause to create relationship.

Then I attempted to create relationships and nodes with different lables in separate queries using the same approach but did not work.

I tried to fix the porblem but without sucess. Sometime the error message is: the newTransaction(graph) has expired.

This following code chunk did not work. The error is: Error in appendCypher.transaction(tx, query) : Neo.ClientError.Statement.SyntaxError Invalid input ')': expected whitespace, '.', node labels, '\[', "=~", IN, STARTS, ENDS, CONTAINS, IS, '^', '\*', '/', '%', '+', '-', '=', "&lt;&gt;", "!=", '&lt;', '&gt;', "&lt;=", "&gt;=", AND, XOR, OR, ',' or '}' (line 2, column 50 (offset: 60)) "CREATE (actor:Actor {name:{actname}, id:{actorid}), title:{movietitle})" ^".

So I put them as statements for further troubleshooting in the future.

``` r
# query = "
#         MATCH (movie:Movie {id:{movieid1}})
#         MATCH (audience:Audience {id1:{audid})
#         CREATE (audience)-[r:RATED]->(movie)
#         SET r.rating = {friendrating}
#         RETURN audience,movie         
#"


#tx = newTransaction(graph)
#for (i in 1:nrow(actor_d)){
#  appendCypher(tx,query,
#                actname = actor_d$actors[i],
#                actorid = actor_d$actorID[i],
#                movietitle = actor_d$title[i],
#                title1 = actor_d$title[i])
#}
#commit(tx)

# query = "
#         CREATE (actor:Actor {name:{actname}, id:{actorid}), #title:{movietitle})
#         MATCH (movie:Movie {name: {title1}})
#         MATCH (actor:Actor {name:{actname}})
#         CREATE (actor)-[a:ACTED_IN]->(movie)
#"
#tx = newTransaction(graph)
#appendCypher(tx,query)
#commit(tx)
```

**Since I could not create relationship and more nodes. I tried a different way to create graphes.**

2.1 Create Nodes with RNeo4j functions
--------------------------------------

``` r
# nodes to be created: 
# Nodes:
#   movie              
#   audience          
#   actor             
#   director                
#   writer              
#   genre
#   country
# the same nodes will be merged.

# Relationships
#  movie            -[:belongs_to]-> genre
#  movie            -[:produced_in]-> country
#  audience         -[:rated_in]->   movie
#  actor            -[:acted_in]-> movie
#  director         -[:directed]-> movie
#  writer           -[:writed_scripts]->  movie

#connect to Neo4j
graph = startGraph("http://localhost:7474/db/data", username = "neo4j", password = neo4j.pw)

#delete all nodes, relationships, indexes, and constraints from the graph database
clear(graph, input = F)

for (i in 1:nrow(rating_d)){
    movie.node <- createNode(graph, "Movie", name=rating_d$MovieName[i], id=rating_d$MovieID[i])
}

for (i in 1:nrow(rating_d)){
    audience.node <- createNode(graph, "Audience", name=rating_d$FriendName[i], id1=rating_d$FriendID[i],id2=rating_d$MovieID[i],moviename=movie_d$MovieName[i])
}

for (i in 1:nrow(actor_d)){
    actor.node <- createNode(graph, "Actor", name=actor_d$actors[i], id=actor_d$actorID[i], movietitle = actor_d$title[i])
}

for (i in 1:nrow(director_d)){
    director.node <- createNode(graph, "Director", name=director_d$directors[i], id=director_d$directorID[i], movietitle = director_d$title[i])
}

for (i in 1:nrow(writer_d)){
    writer.node <- createNode(graph, "Writer", name=writer_d$writers[i], id=writer_d$writerID[i], movietitle = writer_d$title[i])
}

for (i in 1:nrow(genre_d)){
    genre.node <- createNode(graph, "Genre", name=genre_d$genres[i], id=genre_d$genreID[i], movietitle = genre_d$title[i])
}

for (i in 1:nrow(country_d)){
    country.node <- createNode(graph, "Country", name=country_d$countries[i], id=country_d$countryID[i], movietitle = country_d$title[i])
}
```

2.2 Create the relationship with RNeo4j functions
-------------------------------------------------

\*\* 2.2.1. Relationship between audiences and movies\*\*

``` r
movie_query = "
MATCH(a:Movie) WHERE a.id={id}
RETURN a
"
audience_query = "
MATCH(b:Audience) WHERE b.id2={id2} AND b.name = {name}
RETURN b
"

for (i in 1:nrow(rating_d)) {
    movie_rated <- getSingleNode(graph, movie_query, id=rating_d$MovieID[i])
    audience_rated <- getSingleNode(graph, audience_query, id2=rating_d$MovieID[i], name=rating_d$FriendName[i])
    createRel(audience_rated, "RATED", movie_rated, rating = rating_d$FriendRating[i])
}
summary(graph)
```

    ##       This    To  That
    ## 1 Audience RATED Movie

**2.2.2 Relationship between actors and movies**

``` r
movie_query = "
MATCH(a:Movie) WHERE a.name={name}
RETURN a
"
actor_query = "
MATCH(b:Actor) WHERE b.movietitle={movietitle} AND b.name={name}  
RETURN b 
"
 
for (i in 1:nrow(actor_d)){
    movie_rated <- getSingleNode(graph, movie_query, name=rating_d$MovieName[i])
    actor_related  <- getSingleNode(graph, actor_query, name=actor_d$actors[i], id=actor_d$id[i], movietitle=actor_d$title[i])
    createRel(actor_related, "ACTED_IN", movie_rated)
}
summary(graph)
```

    ##       This       To  That
    ## 1    Actor ACTED_IN Movie
    ## 2 Audience    RATED Movie

**2.2.3 Relationship between directors and movies**

``` r
movie_query = "
MATCH(a:Movie) WHERE a.name={name}
RETURN a
"
director_query = "
MATCH(b:Director) WHERE b.movietitle={movietitle} AND b.name={name}  
RETURN b
"
for (i in 1:nrow(director_d)){
    movie_rated <- getSingleNode(graph, movie_query, name=rating_d$MovieName[i])
    director_related  <- getSingleNode(graph, director_query, name=director_d$directors[i], movietitle=director_d$title[i])
    createRel(director_related, "DIRECTED", movie_rated)
}
summary(graph)
```

    ##       This       To  That
    ## 1 Director DIRECTED Movie
    ## 2    Actor ACTED_IN Movie
    ## 3 Audience    RATED Movie

**2.2.4 Relationship between writers and movies**

``` r
movie_query = "
MATCH(a:Movie) WHERE a.name={name}
RETURN a
"
writer_query = "
MATCH(b:Writer) WHERE b.movietitle={movietitle} AND b.name={name}  
RETURN b
"
for (i in 1:nrow(writer_d)){
    movie_rated <- getSingleNode(graph, movie_query, name=rating_d$MovieName[i])
    writer_related  <- getSingleNode(graph, writer_query, name=writer_d$writers[i], movietitle=writer_d$title[i])
    createRel(writer_related, "WROTE", movie_rated)
}
summary(graph)
```

    ##       This       To  That
    ## 1   Writer    WROTE Movie
    ## 2 Director DIRECTED Movie
    ## 3    Actor ACTED_IN Movie
    ## 4 Audience    RATED Movie

**2.2.5 Relationship between genres and movies**

``` r
movie_query = "
MATCH(a:Movie) WHERE a.name={name}
RETURN a
"
genre_query = "
MATCH(b:Genre) WHERE b.movietitle={movietitle} AND b.name={name}  
RETURN b
"
for (i in 1:nrow(genre_d)){
    movie_rated <- getSingleNode(graph, movie_query, name=rating_d$MovieName[i])
    genre_related  <- getSingleNode(graph, genre_query, name=genre_d$genres[i], movietitle=genre_d$title[i])
    createRel(movie_rated, "BELONGS_TO", genre_related)
}
summary(graph)
```

    ##       This         To  That
    ## 1   Writer      WROTE Movie
    ## 2 Director   DIRECTED Movie
    ## 3    Actor   ACTED_IN Movie
    ## 4 Audience      RATED Movie
    ## 5    Movie BELONGS_TO Genre

**2.2.6 Relationship between countries and movies**

``` r
movie_query = "
MATCH(a:Movie) WHERE a.name={name}
RETURN a
"
country_query = "
MATCH(b:Country) WHERE b.movietitle={movietitle} AND b.name={name}  
RETURN b
"
for (i in 1:nrow(country_d)){
    movie_rated <- getSingleNode(graph, movie_query, name=rating_d$MovieName[i])
    country_related  <- getSingleNode(graph, country_query, name=country_d$countries[i], movietitle=country_d$title[i])
    createRel(movie_rated, "PRODUCED_IN", country_related)
}
summary(graph)
```

    ##       This          To    That
    ## 1   Writer       WROTE   Movie
    ## 2 Director    DIRECTED   Movie
    ## 3    Actor    ACTED_IN   Movie
    ## 4 Audience       RATED   Movie
    ## 5    Movie  BELONGS_TO   Genre
    ## 6    Movie PRODUCED_IN Country

tabular results could be returned by using Cypher. As introduced in Nicole White's GitHub RNeo4j repo "<https://github.com/nicolewhite/RNeo4j#nodes>"

``` r
query = "
MATCH (audience:Audience)-[r:RATED]->(movie:Movie)
WHERE audience.name = 'Ming'
RETURN audience.name, r.rating, movie.name
"
cypher(graph, query)
```

    ##   audience.name r.rating                    movie.name
    ## 1          Ming      2.0       The Fate of the Furious
    ## 2          Ming      4.0      Star Wars: The Last Jedi
    ## 3          Ming      4.0 Guardians of the Galaxy Vol. 
    ## 4          Ming      1.5                Thor: Ragnarok
    ## 5          Ming      4.0          Beauty and the Beast
    ## 6          Ming      5.0                         Logan

When returning relative more complicated results:

``` r
query = "
MATCH (audience:Audience)-[r:RATED]->(movie:Movie)
WHERE audience.name = 'Ming'
RETURN audience.name, COLLECT(movie.movie) AS seen
"

cypherToList(graph, query)
```

    ## [[1]]
    ## [[1]]$audience.name
    ## [1] "Ming"
    ## 
    ## [[1]]$seen
    ## list()

``` r
query = "
MATCH (audience:Audience)-[r:RATED]-(movie:Movie {name:'Beauty and the Beast'}) 
RETURN audience.name, movie.name,r.rating"

cypher(graph, query)
```

    ##   audience.name           movie.name r.rating
    ## 1         Orshi Beauty and the Beast        4
    ## 2          Eran Beauty and the Beast        5
    ## 3        Alison Beauty and the Beast        5
    ## 4           Hao Beauty and the Beast        4
    ## 5          Ming Beauty and the Beast        4

Parameters can be passed

``` r
query = "
MATCH (audience:Audience)-[r:RATED]-> (movie:Movie)
WHERE audience.name = {name1} AND movie.name = {name2}
RETURN audience.name, r.rating, movie.name
"

cypher(graph, query, name1="Hao", name2="The Fate of the Furious")
```

    ##   audience.name r.rating              movie.name
    ## 1           Hao        1 The Fate of the Furious

Browse Neo4j Graph
------------------

``` r
graph = startGraph("http://localhost:7474/db/data/", username="neo4j", password=neo4j.pw)
browse(graph)
```

My Rstudio somehow cannot show Neo4j browser. Cypher clauses will be put in Neo4j to build the database. One of the graphes is attached(atudience-rate-movie).

![Neo4j graph](https://raw.githubusercontent.com/YunMai-SPS/DA607/master/DA607_homework13/rating_graph.png)
