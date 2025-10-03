# Spotify Data Analysis Project

![Spotify Logo](spotify.JPG)

**Project Overview**

- **This project analyzes a Spotify dataset using PostgreSQL to derive insights into music streaming trends. It covers data cleaning, exploratory analysis, and advanced SQL queries including window functions, CTEs, and aggregations.**

- **The aim is to simulate real-world data analytics tasks and demonstrate SQL proficiency for a Data Analyst portfolio.**

**Create Table**

```sql
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
**Exploratory Data Analysis (EDA)**

**1 Total rows in dataset**

```sql
SELECT COUNT(*) FROM spotify;
```
**2 Number of distinct artists, albums and album_type**
```sql
SELECT COUNT(DISTINCT(artist)) FROM spotify;

SELECT COUNT(DISTINCT(album)) FROM spotify;

SELECT DISTINCT(album_type) FROM spotify;
```
**3 Longest & shortest track duration**
```sql
SELECT MAX(duration_min) FROM spotify;

SELECT MIN(duration_min) FROM spotify;
```
**4 Invalid records cleanup**
```sql
SELECT * FROM spotify WHERE duration_min=0;

DELETE FROM spotify
WHERE duration_min=0;
```
**5 Unique channels and platforms**

```sql
SELECT DISTINCT(channel) FROM spotify;

SELECT DISTINCT(most_played_on) FROM spotify;
```

**Analysis Easy Level**

**Q1 Retrieve the names of all tracks that have more than 1 billion streams.**

```sql
	SELECT track,stream FROM spotify
	WHERE stream > 1000000000;
```
	
**Q2 List all albums along with their respective artists.**

```sql
	SELECT 
	DISTINCT album, artist 
	FROM spotify
	ORDER BY album;
```
	
**Q3 Get the total number of comments for tracks where licensed = TRUE.**

```sql
	SELECT SUM(comments) AS total_comments FROM spotify
	WHERE licensed = TRUE;
```
**Q4 Find all tracks that belong to the album type single.**

```sql
	SELECT *
	FROM spotify
	WHERE album_type='single';
```
	
**Q5 Count the total number of tracks by each artist.**

```sql
	SELECT 
		artist,
		COUNT(*) AS total_no_songs
	FROM spotify
	GROUP BY artist;
```

**Analysis Medium Level**

**Q6 Calculate the average danceability of tracks in each album.**

```sql
	SELECT 
		album,
		AVG(danceability) AS avg_danceability
	FROM spotify
	GROUP BY album
	ORDER BY avg_danceability DESC;
```	
**Q7 Find the top 5 tracks with the highest energy values.**

```sql
	SELECT 
		track,
		MAX(energy) AS highest_energy
	FROM spotify
	GROUP BY track
	ORDER BY highest_energy DESC
	LIMIT 5;
```
	
**Q8 List all tracks along with their views and likes where official_video = TRUE.**

```sql
	SELECT 
		track,
		SUM(views) AS total_views,
		SUM(likes) AS total_likes
	FROM spotify
	WHERE official_video = TRUE
	GROUP BY track
	ORDER BY total_views DESC;
```
	
**Q9 For each album, calculate the total views of all associated tracks.**

```sql
	SELECT 
		album,
		SUM(views) AS total_views
	FROM spotify
	GROUP BY album
	ORDER BY total_views DESC;
```
	
**Q10 Retrieve the track names that have been streamed on Spotify more than YouTube.**

```sql
SELECT *
	FROM (
	SELECT 
		track,
		COALESCE( SUM(CASE WHEN most_played_on ='Youtube' THEN stream END),0) AS streamed_on_youtube,
		COALESCE(SUM(CASE WHEN most_played_on ='Spotify' THEN stream END),0) AS streamed_on_spotify
	FROM spotify
	GROUP BY track
) AS t1
WHERE streamed_on_spotify>streamed_on_youtube
		AND
		streamed_on_youtube <>0;
```

**Analysis Advanced Level**

**Q11 Find the top 3 most-viewed tracks for each artist using window functions.**

```sql
WITH ranking_artist
AS 
	(
		SELECT 
			artist,
			track,
			SUM(views) AS total_view,
			DENSE_RANK() OVER(PARTITION BY artist ORDER BY SUM(views) DESC) AS rank_views
		FROM spotify
		GROUP BY artist,track
		ORDER BY artist,total_view DESC
	)
SELECT * FROM ranking_artist
WHERE rank_views <=3
```
	
**Q12 Write a query to find tracks where the liveness score is above the average.**

```sql
		SELECT
			track,
			liveness
		FROM spotify
		WHERE liveness > (SELECT AVG(liveness)
							FROM spotify)
```
	
**Q13 Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album.**

```sql
WITH diff_energy
AS (
	SELECT 
		album,
		MAX(energy) AS highest_energy,
		MIN(energy) AS lowest_energy
	FROM spotify
	GROUP BY album
)
SELECT 
	album,
	highest_energy - lowest_energy AS energy_diff
FROM diff_energy
ORDER BY energy_diff DESC;
```
**Q14 Find tracks where the energy-to-liveness ratio is greater than 1.2.**

```sql
	SELECT 
		track,
		energy,
		liveness,
		(energy / liveness) AS energy_liveness_ratio
	FROM spotify
	WHERE energy/liveness > 1.2;
```

**Q15 Calculate the cumulative sum of likes for tracks ordered by the number of views, using window functions**

```sql
	SELECT
		track,
		views,
		likes,
		SUM(likes) OVER(ORDER BY views DESC,track) AS cumulative_likes
	FROM spotify
```	

**Query Optimization Technique**

- **To enhance query performance in the Spotify Data Analysis Project, we applied a systematic optimization process as outlined below:**

```sql
EXPLAIN ANALYZE
	SELECT 
		artist,
		track,
		views
	FROM spotify
	WHERE artist = 'Gorillaz'
		AND
		most_played_on = 'Spotify'
	ORDER BY stream DESC LIMIT 25
```

- **1. Initial Query Performance Analysis (EXPLAIN)**

- **First evaluated the query performance using the EXPLAIN function.**

- **The query retrieved tracks based on the artist column, with the following metrics:**

- **Before Optimization**
	- **Planning Time: 0.360ms**
	- **Execution Time: 10.921ms**

- Below is the **screenshot** of the `EXPLAIN` result before optimization:
      ![Analysis Before Index](Analysis_Before_Optimization.JPG)
	![Query Plan Before Index](Query_Plan_Before_Optimization.JPG)
	
- **Graphical view (Before Optimization):**

	![Graphical Before Index](Graphical_Before_Optimization.JPG)
	  

- **2. Index Creation on the artist Column**

	- **To optimize retrieval speed, we created an index on the artist column.**

	- **Indexing allows the database to quickly locate rows without scanning the entire table.**

```sql
CREATE INDEX artist_idx ON spotify(artist);
```

- **3. Performance Analysis After Indexing**

	- **After applying the index, re-ran the same query and observed a significant performance boost:**

- **After Optimization**
	- **Planning Time: 0.434ms**
	- **Execution Time: 0.573ms**

- Below is the **screenshot** of the `EXPLAIN` result after optimization:
      ![Analysis Before Index](Analysis_After_Optimization.JPG)

	  ![Query Plan Before Index](Query_Plan_After_Optimization.JPG)

- **Graphical view (After Optimization):**
	  ![Graphical Before Index](Graphical_After_Optimization.JPG)

## Conclusion:

- **This optimization demonstrates how indexing can drastically reduce query execution time, leading to more efficient and scalable database operations in the Spotify Data Analysis Project.**

## Spotify Data Analysis Project – Key Insights

- **Global Hit Tracks**
    - **385 tracks have crossed 1 billion streams, showing their massive global popularity and dominance on streaming platforms.**

- **Impact of Official Videos**
    - **Tracks with official videos gather 3x more views than non-official uploads, highlighting how video content boosts fan engagement and visibility.**

- **Energy-to-Liveness Ratio**
    - **Tracks where energy-to-liveness ratio > 1.2 are mostly studio-recorded EDM tracks, dominating over live recordings and showing the production style of high-energy tracks.**

- **Platform Preference Trends**
    - **On average, Spotify streams exceed YouTube views for newer artists, indicating a trend where listeners prefer Spotify for recent releases.**


