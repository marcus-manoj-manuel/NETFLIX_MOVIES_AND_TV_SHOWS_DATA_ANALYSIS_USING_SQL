# NETFLIX MOVIES AND TV SHOWS DATA ANALYSIS USING SQL

![Netflix Logo](https://github.com/marcus-manoj-manuel/Netflix-Movies-TV-Shows-SQL-Data-Analysis-Project/blob/main/logo.png)
<!-- Replace with your own Netflix logo image URL or remove this line -->

## Overview

This project presents a comprehensive SQL-based analysis of Netflix's movies and TV shows dataset. Using PostgreSQL 17 and pgAdmin, I explored the Netflix catalog to uncover meaningful insights about content distribution, ratings, regional trends, and genre patterns. The project showcases practical SQL skills including data manipulation, aggregation, string functions, and complex querying techniques essential for data analyst roles.

## Objectives

- Analyze the distribution of content types (movies vs TV shows) on Netflix
- Identify the most common ratings for different content types
- Investigate content based on release years, countries, and durations
- Explore content added in recent years and identify trends
- Categorize and filter content based on specific criteria and keywords
- Extract insights about directors, actors, and genres

## Dataset

The dataset for this project is sourced from Kaggle and contains information about Netflix's movies and TV shows:

- **Dataset Link:** [Netflix Movies and TV Shows Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)
- **Dataset Description:** Contains details about Netflix titles including type, director, cast, country, release year, rating, duration, genre, and description

## Schema
```sql
DROP TABLE IF EXISTS netflix;

CREATE TABLE netflix (
    show_id      VARCHAR(6),
    type         VARCHAR(10),
    title        VARCHAR(150),
    director     VARCHAR(208),
    casts        VARCHAR(1000),
    country      VARCHAR(150),
    date_added   VARCHAR(50),
    release_year INT,
    rating       VARCHAR(10),
    duration     VARCHAR(15),
    listed_in    VARCHAR(100),
    description  VARCHAR(250)
);
```

## Business Problems and Solutions

### 1. Count the Number of Movies vs TV Shows

**Objective:** Determine the distribution of content types on Netflix.
```sql
SELECT 
    type,
    COUNT(*) AS total_content
FROM netflix
GROUP BY type;
```

### 2. Find the Most Common Rating for Movies and TV Shows

**Objective:** Identify the most frequently occurring rating for each type of content.
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

### 3. List All Movies Released in a Specific Year (e.g., 2020)

**Objective:** Retrieve all movies released in a specific year.
```sql
SELECT * 
FROM netflix
WHERE release_year = 2020
  AND type = 'Movie';
```

### 4. Find the Top 5 Countries with the Most Content on Netflix

**Objective:** Identify the top 5 countries with the highest number of content items.
```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
    COUNT(*) AS total_content
FROM netflix
WHERE country IS NOT NULL
GROUP BY country
ORDER BY total_content DESC
LIMIT 5;
```

### 5. Identify the Longest Movie

**Objective:** Find the movie with the longest duration.
```sql
SELECT 
    title,
    duration
FROM netflix
WHERE type = 'Movie'
  AND duration IS NOT NULL
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC
LIMIT 1;
```

### 6. Find Content Added in the Last 5 Years

**Objective:** Retrieve content added to Netflix in the last 5 years.
```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

**Objective:** List all content directed by 'Rajiv Chilaka'.
```sql
SELECT *
FROM netflix
WHERE director LIKE '%Rajiv Chilaka%';
```

### 8. List All TV Shows with More Than 5 Seasons

**Objective:** Identify TV shows with more than 5 seasons.
```sql
SELECT 
    title,
    duration
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

### 9. Count the Number of Content Items in Each Genre

**Objective:** Count the number of content items in each genre.
```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY genre
ORDER BY total_content DESC;
```

### 10. Find Each Year and the Average Number of Content Releases by India on Netflix

**Objective:** Calculate and rank years by the average number of content releases from India, returning the top 5 years.
```sql
SELECT 
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country LIKE '%India%')::numeric * 100, 2
    ) AS avg_release_percentage
FROM netflix
WHERE country LIKE '%India%'
GROUP BY release_year
ORDER BY total_release DESC
LIMIT 5;
```

### 11. List All Movies that are Documentaries

**Objective:** Retrieve all movies classified as documentaries.
```sql
SELECT * 
FROM netflix
WHERE type = 'Movie'
  AND listed_in LIKE '%Documentaries%';
```

### 12. Find All Content Without a Director

**Objective:** List content that does not have a director.
```sql
SELECT * 
FROM netflix
WHERE director IS NULL;
```

### 13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

**Objective:** Count the number of movies featuring 'Salman Khan' in the last 10 years.
```sql
SELECT * 
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10
  AND type = 'Movie';
```

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

**Objective:** Identify the top 10 actors with the most appearances in Indian-produced movies.
```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*) AS total_appearances
FROM netflix
WHERE country LIKE '%India%'
  AND type = 'Movie'
GROUP BY actor
ORDER BY total_appearances DESC
LIMIT 10;
```

### 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords

**Objective:** Categorize content as 'Bad' if it contains 'kill' or 'violence' keywords and 'Good' otherwise. Count the number of items in each category.
```sql
SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category;
```

## Findings and Conclusion

Through this analysis, I discovered several key insights about Netflix's content library:

- **Content Distribution:** The dataset reveals a diverse collection of movies and TV shows, with distinct patterns in how content is categorized and distributed across different types.
- **Rating Patterns:** The most common ratings analysis provides valuable insights into Netflix's target audience demographics and content strategy.
- **Geographic Trends:** The analysis of top countries producing content and India's content release patterns over the years highlights regional content preferences and Netflix's global expansion strategy.
- **Content Characteristics:** Investigating longest movies, TV shows with multiple seasons, and genre distribution reveals Netflix's focus on varied content to cater to different viewer preferences.
- **Director and Actor Insights:** Identifying prolific directors and actors, especially in regional content like Indian movies, showcases the platform's investment in regional talent.
- **Content Categorization:** The keyword-based categorization demonstrates how content can be filtered for specific themes, which is useful for content recommendation systems.

This project demonstrates the power of SQL in extracting actionable insights from large datasets. The analysis can inform content acquisition strategies, help understand viewer preferences, and guide decision-making for streaming platforms.

## Author

**Marcus Manoj Manuel**

This project is part of my data analyst portfolio, showcasing SQL skills essential for data analysis roles. The project demonstrates proficiency in PostgreSQL, data manipulation, aggregation functions, and extracting business insights from real-world datasets.

### Connect with Me

- **LinkedIn:** [Marcus Manoj Manuel](https://in.linkedin.com/in/marcus-manoj-manuel-a489642a9)

Feel free to reach out for any questions, feedback, or collaboration opportunities!

---

*Thank you for reviewing my project!*
