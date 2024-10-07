To make the project description more original, I'll rephrase the content and remove any potential plagiarism while maintaining the structure and technical accuracy. Here's a cleaned-up version:

---

# Netflix Movies and TV Shows Data Analysis Using SQL

## Overview

This project involves a detailed analysis of Netflix's catalog of movies and TV shows using SQL. The aim is to derive meaningful insights and address key business questions by working with the dataset. Below is an outline of the project's goals, data-driven questions, solutions, findings, and conclusions.

## Objectives

- Examine the distribution between movies and TV shows on Netflix.
- Determine the most common content ratings for both movies and TV shows.
- Analyze content based on release year, countries, and durations.
- Explore content categories and keywords to identify patterns.

## Dataset

The dataset used in this analysis is available on Kaggle:

- **Dataset Link:** [Netflix Movies and TV Shows Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema

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

## Business Questions and SQL Solutions

### 1. Distribution of Movies vs. TV Shows

```sql
SELECT 
    type,
    COUNT(*)
FROM netflix
GROUP BY type;
```

**Goal:** To understand the balance between the number of movies and TV shows available on Netflix.

### 2. Identify the Most Common Ratings for Movies and TV Shows

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

**Goal:** To find out the most frequently assigned rating for both types of content.

### 3. Movies Released in a Specific Year (e.g., 2020)

```sql
SELECT * 
FROM netflix
WHERE release_year = 2020;
```

**Goal:** Retrieve all movies released in a specified year (e.g., 2020).

### 4. Top 5 Countries with the Most Content

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

**Goal:** Identify the top 5 countries with the most content available on Netflix.

### 5. The Longest Movie on Netflix

```sql
SELECT 
    *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```

**Goal:** Find the movie with the longest duration on the platform.

### 6. Content Added to Netflix in the Last 5 Years

```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

**Goal:** Retrieve all content added to Netflix in the last 5 years.

### 7. Movies/TV Shows Directed by Rajiv Chilaka

```sql
SELECT *
FROM (
    SELECT 
        *,
        UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name
    FROM netflix
) AS t
WHERE director_name = 'Rajiv Chilaka';
```

**Goal:** List all movies and TV shows directed by Rajiv Chilaka.

### 8. TV Shows with More Than 5 Seasons

```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

**Goal:** Identify TV shows that have more than 5 seasons.

### 9. Number of Content Items in Each Genre

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY genre;
```

**Goal:** Count the number of content items in each genre.

### 10. Average Number of Releases in India by Year

```sql
SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'India')::numeric * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'India'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```

**Goal:** Rank the top 5 years based on average content releases from India.

### 11. Documentaries on Netflix

```sql
SELECT * 
FROM netflix
WHERE listed_in LIKE '%Documentaries';
```

**Goal:** Retrieve all movies categorized as documentaries.

### 12. Content Without a Director

```sql
SELECT * 
FROM netflix
WHERE director IS NULL;
```

**Goal:** List all content that doesn't have a director assigned.

### 13. Movies Featuring Salman Khan in the Last 10 Years

```sql
SELECT * 
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
```

**Goal:** Count how many movies Salman Khan has appeared in over the last 10 years.

### 14. Top 10 Actors Appearing in Indian Movies

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;
```

**Goal:** Find the top 10 actors with the most movie appearances in Indian productions.

### 15. Content Categorized by 'Kill' or 'Violence' Keywords

```sql
SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Violent'
            ELSE 'Non-violent'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category;
```

**Goal:** Classify content as 'Violent' if descriptions contain the words 'kill' or 'violence,' and 'Non-violent' otherwise.

## Findings and Conclusion

- **Content Distribution:** The platform offers a diverse selection of both movies and TV shows across various ratings and genres.
- **Ratings Insight:** Understanding the most frequent ratings helps target the primary audience for each type of content.
- **Regional Insights:** Analysis of countries, especially content production in India, highlights key geographic trends.
- **Content Classification:** Categorizing based on specific keywords like 'violence' provides insight into the nature of content on Netflix.
