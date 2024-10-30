# Spotify Advanced SQL Project and Query Optimization
Project Category: Advanced
[Click Here to get Dataset](https://www.kaggle.com/datasets/sanjanchaudhari/spotify-dataset)

![Spotify Logo](https://github.com/Azmary413/Spotify-Advanced-SQL-Project-and-Query-Optimization/blob/main/spotify%20background.jpg)

## Overview
This project involves analyzing a Spotify dataset with various attributes about tracks, albums, and artists using **SQL**. It covers an end-to-end process of normalizing a denormalized dataset, performing SQL queries of varying complexity (easy, medium, and advanced), and optimizing query performance. The primary goals of the project are to practice advanced SQL skills and generate valuable insights from the dataset.

## 🛠 Project Steps

### 1. Data Exploration
Before diving into SQL, it’s important to understand the dataset thoroughly. The dataset contains attributes such as:
- `Artist`: The performer of the track.
- `Track`: The name of the song.
- `Album`: The album to which the track belongs.
- `Album_type`: The type of album (e.g., single or album).
- Various metrics such as `danceability`, `energy`, `loudness`, `tempo`, and more.

## 2. Table Creation


```sql
-- create table
DROP TABLE IF EXISTS spotify;
CREATE TABLE spotify (
    artist VARCHAR(255),
    track VARCHAR(255),
    album VARCHAR(255),
    album_type VARCHAR(50),
    danceability FLOAT,
    energy FLOAT,
    loudness FLOAT,
    speechiness FLOAT,
    acousticness FLOAT,
    instrumentalness FLOAT,
    liveness FLOAT,
    valence FLOAT,
    tempo FLOAT,
    duration_min FLOAT,
    title VARCHAR(255),
    channel VARCHAR(255),
    views FLOAT,
    likes BIGINT,
    comments BIGINT,
    licensed BOOLEAN,
    official_video BOOLEAN,
    stream BIGINT,
    energy_liveness FLOAT,
    most_played_on VARCHAR(50)
);
```

### 3. Querying the Data
After the data is inserted, various SQL queries can be written to explore and analyze the data. Queries are categorized into **easy**, **medium**, and **advanced** levels to help progressively develop SQL proficiency.

**Easy Queries:** Basic retrieval, filtering, and aggregation.
**Medium Queries:** Involves grouping, joins, and aggregation functions.
**Advanced Queries:** Uses nested subqueries, window functions, and CTEs.
  
---

## 15 Practice Questions

### Data Analysis : Easy Category

1. **Retrieve the names of all tracks that have more than 1 billion streams.**

```sql
SELECT * FROM spotify
WHERE stream > 1000000000;
```
   
2. **List all albums along with their respective artists.**
  
```sql
SELECT album, artist
FROM spotify
ORDER BY 1;
```
   
3. **Get the total number of comments for tracks where `licensed = TRUE`.**

```sql
SELECT SUM(comments) as total_comments
FROM spotify
WHERE licensed = 'true';
```
   
4. **Find all tracks that belong to the album type `single`.**

```sql
SELECT *
FROM spotify
WHERE album_type = 'single';
```
   
5. **Count the total number of tracks by each artist.**

```sql
SELECT artist, COUNT(*) AS total_songs
FROM spotify
GROUP BY 1;
```

### Data Analysis : Medium Category

6. **Calculate the average danceability of tracks in each album.**

```sql
SELECT album, avg(danceability) as avg_danceability
FROM spotify
GROUP BY 1
ORDER BY 2 DESC;
```

7. **Find the top 5 tracks with the highest energy values.**

```sql
SELECT track, MAX(energy)
FROM spotify
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;

```

8. **List all tracks along with their views and likes where `official_video = TRUE`.**

```sql
SELECT track, SUM(views) AS total_views, SUM(likes) AS total_likes
FROM spotify
WHERE official_video = 'true'
GROUP BY 1
ORDER BY 2;

```

9. **For each album, calculate the total views of all associated tracks.**

```sql
SELECT album, track, SUM(views) AS total_views
FROM spotify
GROUP BY 1,2
ORDER BY 3 DESC;
```

10. **Retrieve the track names that have been streamed on Spotify more than YouTube**.

```sql
SELECT *
FROM (
    SELECT
        track,
        COALESCE(SUM(CASE WHEN most_played_on = 'YouTube' THEN stream END), 0) AS streamed_on_youtube,
        COALESCE(SUM(CASE WHEN most_played_on = 'Spotify' THEN stream END), 0) AS streamed_on_spotify
    FROM spotify
    GROUP BY 1
) AS tl
WHERE streamed_on_spotify > streamed_on_youtube
    AND 
	streamed_on_youtube <> 0;

```


### Advanced Level
11. Find the top 3 most-viewed tracks for each artist using window functions.

```sql
WITH ranking_artist AS
(
	SELECT 
	artist, 
	track, 
	SUM(views) AS total_views,
	DENSE_RANK() OVER(PARTITION BY artist ORDER BY SUM(views) DESC ) AS rank
	FROM spotify
	GROUP BY 1,2
	ORDER BY 1,3 DESC
)
SELECT *
FROM ranking_artist
WHERE rank <= 3;

```

12. Write a query to find tracks where the liveness score is above the average.

```sql
SELECT 
artist,
track,
liveness
FROM spotify
WHERE liveness > (
	SELECT 
	AVG(liveness) AS avg_score
	FROM spotify
);

```

13. **Use a `WITH` clause to calculate the difference between the highest and lowest energy values for tracks in each album.**

```sql
WITH cte
AS
(SELECT 
	album,
	MAX(energy) as highest_energy,
	MIN(energy) as lowest_energery
FROM spotify
GROUP BY 1
)
SELECT 
	album,
	highest_energy - lowest_energery as energy_diff
FROM cte
ORDER BY 2 DESC
```
   
14. Find tracks where the energy-to-liveness ratio is greater than 1.2.

```sql

SELECT
track,
ROUND(CAST(energy/liveness AS numeric), 2) AS energy_liveness_ratio
FROM spotify
WHERE energy/liveness > 1.2
ORDER BY 2 DESC;

```

15. Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions.

```sql
SELECT 
	track,
	SUM(likes) OVER(ORDER BY views) AS cumulative_sum
FROM Spotify
	ORDER BY cumulative_sum DESC;
```

Here’s an updated section for your **Spotify Advanced SQL Project and Query Optimization** README, focusing on the query optimization task you performed. You can include the specific screenshots and graphs as described.

---

## Query Optimization Technique 

To improve query performance, we carried out the following optimization process:

- **Initial Query Performance Analysis Using `EXPLAIN`**
    - We began by analyzing the performance of a query using the `EXPLAIN` function.
    - The query retrieved tracks based on the `artist` column, and the performance metrics were as follows:
        - Execution time (E.T.): **10.9 ms**
        - Planning time (P.T.): **1.16 ms**
    - Below is the **screenshot** of the `EXPLAIN` result before optimization:
      ![EXPLAIN Before Index](https://github.com/Azmary413/Spotify-Advanced-SQL-Project-and-Query-Optimization/blob/main/spotify_explain_before_index.jpg)

- **Index Creation on the `artist` Column**
    - To optimize the query performance, we created an index on the `artist` column. This ensures faster retrieval of rows where the artist is queried.
    - **SQL command** for creating the index:
      ```sql
      CREATE INDEX idx_artist ON spotify_tracks(artist);
      ```

- **Performance Analysis After Index Creation**
    - After creating the index, we ran the same query again and observed significant improvements in performance:
        - Execution time (E.T.): **0.144 ms**
        - Planning time (P.T.): **1.396 ms**
    - Below is the **screenshot** of the `EXPLAIN` result after index creation:
      ![EXPLAIN After Index](https://github.com/Azmary413/Spotify-Advanced-SQL-Project-and-Query-Optimization/blob/main/spotify_explain_after_index.jpg)

- **Graphical Performance Comparison**
    - A graph illustrating the comparison between the initial query execution time and the optimized query execution time after index creation.
    - **Graph view** shows the significant drop in both execution and planning times:
      ![Performance Graph](https://github.com/Azmary413/Spotify-Advanced-SQL-Project-and-Query-Optimization/blob/main/analysis_after_index.jpg)
      ![Performance Graph](https://github.com/Azmary413/Spotify-Advanced-SQL-Project-and-Query-Optimization/blob/main/graphical_view_before_index.jpg)
      ![Performance Graph](https://github.com/Azmary413/Spotify-Advanced-SQL-Project-and-Query-Optimization/blob/main/graphical_view_after_index.jpg)

This optimization shows how indexing can drastically reduce query time, improving the overall performance of our database operations in the Spotify project.
---

## Technology Stack
- **Database**: PostgreSQL
- **SQL Queries**: DDL, DML, Aggregations, Joins, Subqueries, Window Functions
- **Tools**: pgAdmin 4 (or any SQL editor), PostgreSQL (via Homebrew, Docker, or direct installation)

## 🚀 How to Run the Project
Set Up: Install PostgreSQL and pgAdmin (if not installed).
Create Schema: Build tables using the normalization structure.
Insert Data: Populate tables with sample data.
Run Queries: Execute SQL queries to analyze and optimize.
Optimization: Use indexing and performance review for large datasets.

---

## Next Steps
- **Visualize the Data**: Use a data visualization tool like **Tableau** or **Power BI** to create dashboards based on the query results.
- **Expand Dataset**: Add more rows to the dataset for broader analysis and scalability testing.
- **Advanced Querying**: Dive deeper into query optimization and explore the performance of SQL queries on larger datasets.

---

## Contributing
If you would like to contribute to this project, feel free to fork the repository, submit pull requests, or raise issues.

---

## License
This project is licensed under the MIT License.
