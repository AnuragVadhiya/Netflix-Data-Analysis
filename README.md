
# Netflix Content Analysis with SQL

## Introduction

This project, **Netflix Content Analysis**, utilizes SQL to perform a detailed exploration of Netflix's movies and TV shows dataset. The primary aim is to derive actionable insights that address key business questions related to content strategy, audience targeting, and platform performance. By analyzing attributes such as content type, ratings, release years, genres, and geographic origins, this project uncovers trends and patterns that can inform Netflix’s content acquisition, production, and marketing decisions.

The dataset, stored in a PostgreSQL table named `netflix`, includes comprehensive details about each title, such as show ID, type, title, director, cast, country, release year, rating, duration, genres, and description. The accompanying SQL script (`netflix_titles.sql`) provides a robust set of queries for data exploration and analysis, designed to be both insightful and adaptable to similar datasets.

This repository showcases practical SQL techniques for extracting meaningful business insights from complex datasets.

## Dataset Description

The `netflix` table contains the following columns:

| Column Name   | Data Type     | Description                                      |
|---------------|---------------|--------------------------------------------------|
| show_id       | VARCHAR(5)    | Unique identifier for each title                 |
| type          | VARCHAR(10)   | Content type (Movie or TV Show)                  |
| title         | VARCHAR(250)  | Title of the movie or TV show                    |
| director      | VARCHAR(550)  | Director(s) of the content                       |
| casts         | VARCHAR(1050) | Cast members involved in the content             |
| country       | VARCHAR(550)  | Country(ies) of production                       |
| date_added    | VARCHAR(55)   | Date the content was added to Netflix            |
| release_year  | INT           | Year the content was released                    |
| rating        | VARCHAR(15)   | Content rating (e.g., TV-MA, PG-13)              |
| duration      | VARCHAR(15)   | Duration (e.g., 120 min, 3 Seasons)              |
| listed_in     | VARCHAR(250)  | Genres or categories (e.g., Drama, Comedy)       |
| description   | VARCHAR(550)  | Brief description of the content                 |

**Source**: The dataset is sourced from Kaggle: [Netflix Shows Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download).

### Query Descriptions

#### Table Creation
```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```
- **Purpose**: Drops any existing `netflix` table and creates a new one with the appropriate schema.

#### Q.1: Count the Number of Movies vs TV Shows
```sql
SELECT 
    type,
    COUNT(*)
FROM netflix
GROUP BY 1;
```
- **Objective**: Quantify the proportion of movies versus TV shows in Netflix’s catalog.

#### Q.2: Find the Most Common Rating for Movies and TV Shows
```sql
WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```
- **Objective**: Determine the most prevalent content rating for movies and TV shows.

#### Q.3: List All Movies Released in a Specific Year (e.g., 2020)
```sql
SELECT * 
FROM netflix
WHERE release_year = 2020;
```
- **Objective**: Retrieve all movies released in 2020.

#### Q.4: Find the Top 5 Countries with the Most Content on Netflix
```sql
SELECT * 
FROM
(
    SELECT 
        UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
        COUNT(*) AS total_content
    FROM netflix
    GROUP BY 1
) AS t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```
- **Objective**: Identify the five countries producing the most Netflix content.

#### Q.5: Identify the Longest Movie
```sql
SELECT 
    *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```
- **Objective**: Find the movie with the longest runtime.

#### Q.6: Find Content Added in the Last 5 Years
```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```
- **Objective**: List content added to Netflix since June 2020 (based on June 16, 2025).

#### Q.7: Find All Movies/TV Shows by specific Director 
```sql
SELECT *
FROM (
    SELECT 
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS t
WHERE director_name = 'Suhas Kadav';
```
- **Objective**: Retrieve all content directed by a specified director.

#### Q.8: List All TV Shows with More Than 3 Seasons
```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 3;
```
- **Objective**: Identify TV shows with over 3 seasons(or any number of seasons).

#### Q.9: List All Movies that are Documentaries
```sql
SELECT * 
FROM netflix
WHERE listed_in LIKE '%Documentaries';
```
- **Objective**: Retrieve all documentary movies.


#### Q.10: Find All Content Without a Director
```sql
SELECT * 
FROM netflix
WHERE director IS NULL;
```
- **Objective**: List content lacking a director.

#### Q.11: Find Movies Featuring '---------'
```sql
SELECT * 
FROM netflix
WHERE casts LIKE '%Aamir Khan%';
```
- **Objective**: Retrieve all movies casting specified actor.

#### Q.12: Categorize Content Based on 'Kill' and 'Violence' Keywords
```sql
SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'A/R rated'
            ELSE 'U/A Rated'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category;

```
- **Objective**: Classify content as 'A/R rated' (contains 'kill' or 'violence') or 'U/A rated' and count each category.


This analysis equips Netflix with data-driven insights to optimize its content strategy, enhance viewer engagement, and strengthen its global presence.

## Author

**Anurag Vadhiya**

This project is a testament to my SQL proficiency and passion for data analysis, crafted to demonstrate my ability to derive business insights from complex datasets. For inquiries, feedback, or collaboration opportunities, please open an issue on GitHub or contact me directly.

---

*Last Updated: June 16, 2025*
