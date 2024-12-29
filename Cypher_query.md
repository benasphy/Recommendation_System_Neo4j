# Data Model

This document provides a detailed description of the Neo4j data model used in the project, including Cypher queries to create and manage the database structure, as well as recommendation algorithms.

## Adding Persons

```cypher
MERGE (p1:Person {name: "Alice", age: 30, location: "New York"})
MERGE (p2:Person {name: "Bob", age: 25, location: "London"})
MERGE (p3:Person {name: "Charlie", age: 35, location: "Paris"})
```

## Adding Genres and Movies

### Genres
```cypher
MERGE (g1:Genre {name: "Sci-Fi"})
MERGE (g2:Genre {name: "Romance"})
```

### Movies
```cypher
MERGE (m1:Movie {title: "The Matrix", rating: 4.8, release_year: 1999})
MERGE (m2:Movie {title: "Inception", rating: 4.7, release_year: 2010})
MERGE (m3:Movie {title: "Titanic", rating: 4.1, release_year: 1997})
MERGE (m4:Movie {title: "Interstellar", rating: 4.9, release_year: 2014})
MERGE (m5:Movie {title: "The Notebook", rating: 4.3, release_year: 2004})
```

## Creating Relationships

### BELONGS_TO
```cypher
MERGE (m1)-[:BELONGS_TO]->(g1)
MERGE (m2)-[:BELONGS_TO]->(g1)
MERGE (m3)-[:BELONGS_TO]->(g2)
MERGE (m5)-[:BELONGS_TO]->(g2)
MERGE (m4)-[:BELONGS_TO]->(g1)
```

### WATCHED
```cypher
MERGE (p1)-[:WATCHED {rating: 5}]->(m1)
MERGE (p1)-[:WATCHED {rating: 4}]->(m3)
MERGE (p2)-[:WATCHED {rating: 4}]->(m2)
MERGE (p2)-[:WATCHED {rating: 3}]->(m5)
MERGE (p3)-[:WATCHED {rating: 5}]->(m3)
MERGE (p3)-[:WATCHED {rating: 4}]->(m4)
```

### FRIENDS_WITH
```cypher
MERGE (p1)-[:FRIENDS_WITH]->(p2)
MERGE (p2)-[:FRIENDS_WITH]->(p3)
MERGE (p1)-[:FRIENDS_WITH]->(p3)
```

## Recommendation Queries

### Collaborative Filtering Recommendation
Recommend movies that friends have watched but the user hasn’t.
```cypher
MATCH (user:Person {name: "Alice"})-[:FRIENDS_WITH]->(friend)-[watched:WATCHED]->(movie:Movie)
WHERE NOT (user)-[:WATCHED]->(movie)
WITH movie, COALESCE(watched.rating, 0) AS friend_rating
RETURN movie.title AS RecommendedMovies, avg(friend_rating) AS AverageFriendRating
ORDER BY AverageFriendRating DESC
LIMIT 5
```

### Content-Based Filtering
Recommend movies based on genres the user has liked.
```cypher
MATCH (user:Person {name: "Alice"})-[:WATCHED]->(watched:Movie)-[:BELONGS_TO]->(genre)
MATCH (genre)<-[:BELONGS_TO]-(movie:Movie)
WHERE NOT (user)-[:WATCHED]->(movie)
RETURN movie.title AS RecommendedMovies, movie.rating AS MovieRating
ORDER BY MovieRating DESC
LIMIT 5
```

### Hybrid Filtering
Combine collaborative and content-based filtering for better recommendations.
```cypher
MATCH (user:Person {name: "Alice"})-[:FRIENDS_WITH]->(friend)-[:WATCHED]->(friend_movie:Movie)
WHERE NOT (user)-[:WATCHED]->(friend_movie)
WITH user, friend_movie, count(friend) AS friend_count
MATCH (friend_movie)-[:BELONGS_TO]->(genre)<-[:BELONGS_TO]-(user_movie:Movie)<-[:WATCHED]-(user)
RETURN friend_movie.title AS RecommendedMovies, friend_count AS Popularity, avg(user_movie.rating) AS GenreRelevance
ORDER BY Popularity DESC, GenreRelevance DESC
LIMIT 5
```


