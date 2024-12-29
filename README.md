# Recommendation System with Neo4j and Machine Learning (Naive Bayes Algorithm)

This repository demonstrates how to build a **Recommendation System** using Neo4j as the database and Python for machine learning, Mainly I used **Naive Bayes** algorithm which is suitable for prediction. It includes a detailed example of how to structure data in Neo4j using Cypher queries, alongside a machine learning-based recommendation model implemented with Python.

## Features

1. **Data Model in Neo4j:**
   - `Person` nodes with attributes like `name`, `age`, and `location`.
   - `Movie` nodes with properties such as `title`, `rating`, and `release_year`.
   - `Genre` nodes to categorize movies.
   - Relationships:
     - `WATCHED`: Tracks which movies a person has watched and their ratings.
     - `FRIENDS_WITH`: Connects people who are friends.
     - `BELONGS_TO`: Links movies to their genres.

2. **Recommendation Approaches:**
   - **Collaborative Filtering**: Suggests movies based on what friends have watched.
   - **Content-Based Filtering**: Recommends movies based on genres a user likes.
   - **Hybrid Filtering**: Combines collaborative and content-based techniques for improved recommendations.

3. **Machine Learning Integration:**
   - Implements a **Naive Bayes** classifier for predicting user ratings on unseen movies.
   - Uses real data fetched from Neo4j for model training and evaluation.

4. **Cypher Query Integration:**
   - Includes a **`cypher_query.md`** file with all relevant Cypher queries for setting up the database and running recommendation queries directly in Neo4j.

---

## Getting Started

### Prerequisites

- Python 3.8+
- Neo4j database (with AuraDB or local installation)
- Required Python libraries:
  ```bash
  pip install neo4j pandas scikit-learn numpy
  ```

### Database Setup

1. **Initialize the Neo4j Database:**
   Use the `cypher_query.txt` file to populate the Neo4j database with data. The file contains Cypher queries for:
   - Creating `Person`, `Movie`, and `Genre` nodes.
   - Establishing relationships like `WATCHED`, `FRIENDS_WITH`, and `BELONGS_TO`.

   Run the queries in the Neo4j Browser:
   ```cypher
   :source cypher_query.txt
   ```

2. **Connection Configuration:**
   Update the following credentials in your Python script:
   ```python
   NEO4J_URI = "neo4j+s://e3c132c9.databases.neo4j.io"
   NEO4J_USER = "neo4j"
   NEO4J_PASSWORD = "c4GoxNxc7ZU4sv8reohTvAh9RH3rf1GXXmielB2TZps"
   ```

---

## Python Script Overview

### Key Components

1. **Fetch Data from Neo4j:**
   Retrieves user, movie, and rating data from the Neo4j database.

   ```python
   MATCH (p:Person)-[w:WATCHED]->(m:Movie)
   RETURN p.name AS user, m.title AS movie, w.rating AS rating
   ```

2. **Machine Learning Model:**
   - Uses a **Naive Bayes Classifier** to predict ratings.
   - Evaluates the model using **Root Mean Squared Error (RMSE)**.

3. **Recommendation Generation:**
   - Predicts ratings for unseen movies.
   - Provides top recommendations for a user.

4. **Neo4j Integration:**
   Writes recommendations back into Neo4j as `RECOMMENDED` relationships.

   ```python
   MATCH (u:Person {name: 'Alice'})
   MATCH (m:Movie {title: 'Inception'})
   MERGE (u)-[:RECOMMENDED {predicted_rating: 4.5}]->(m)
   ```

---

## Recommendation Queries in Cypher

### Collaborative Filtering
Recommends movies that a user's friends have watched but the user hasn't.
```cypher
MATCH (user:Person {name: "Alice"})-[:FRIENDS_WITH]->(friend)-[watched:WATCHED]->(movie:Movie)
WHERE NOT (user)-[:WATCHED]->(movie)
WITH movie, COALESCE(watched.rating, 0) AS friend_rating
RETURN movie.title AS RecommendedMovies, avg(friend_rating) AS AverageFriendRating
ORDER BY AverageFriendRating DESC
LIMIT 5
```

### Content-Based Filtering
Suggests movies based on genres the user likes.
```cypher
MATCH (user:Person {name: "Alice"})-[:WATCHED]->(watched:Movie)-[:BELONGS_TO]->(genre)
MATCH (genre)<-[:BELONGS_TO]-(movie:Movie)
WHERE NOT (user)-[:WATCHED]->(movie)
RETURN movie.title AS RecommendedMovies, movie.rating AS MovieRating
ORDER BY MovieRating DESC
LIMIT 5
```

### Hybrid Filtering
Combines collaborative and content-based filtering for more accurate recommendations.
```cypher
MATCH (user:Person {name: "Alice"})-[:FRIENDS_WITH]->(friend)-[:WATCHED]->(friend_movie:Movie)
WHERE NOT (user)-[:WATCHED]->(friend_movie)
WITH user, friend_movie, count(friend) AS friend_count
MATCH (friend_movie)-[:BELONGS_TO]->(genre)<-[:BELONGS_TO]-(user_movie:Movie)<-[:WATCHED]-(user)
RETURN friend_movie.title AS RecommendedMovies, friend_count AS Popularity, avg(user_movie.rating) AS GenreRelevance
ORDER BY Popularity DESC, GenreRelevance DESC
LIMIT 5
```

---

---

## How to Use

1. **Set Up Neo4j:**
   - Import the `cypher_query.txt` into your Neo4j database.

2. **Run the Python Script:**
   - Ensure the Python script is connected to the Neo4j database.
   - Execute `main.py` to generate recommendations.

3. **Access Recommendations:**
   - View the recommendations directly in Neo4j under the `RECOMMENDED` relationship.
   - Alternatively, retrieve them from the Python output.

---

## License
This project is licensed under the MIT License. Feel free to use, modify, and share.

---

## Contributing
Contributions are welcome! Please feel free to open issues or submit pull requests.

