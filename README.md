-- ============================================
-- Netflix Data Analysis | SQL Project
-- Author: Marcus Manoj Manuel
-- Tools: PostgreSQL 17 + pgAdmin
-- ============================================


-- 🔹 Create Table Schema
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


-- ============================================
-- 📍 BUSINESS PROBLEMS & SQL SOLUTIONS
-- ============================================

-- 1️⃣ Count the number of Movies vs TV Shows
SELECT 
    type AS content_type,
    COUNT(*) AS total_titles
FROM netflix
GROUP BY type
ORDER BY total_titles DESC;


-- 2️⃣ Most common ratings for Movies vs TV Shows
WITH RatingCounts AS (
    SELECT type, rating, COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT type, rating, rating_count,
           RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating,
    rating_count
FROM RankedRatings
WHERE rank = 1;


-- 3️⃣ List all Movies released in a specific year (Example: 2020)
SELECT *
FROM netflix
WHERE type = 'Movie'
  AND release_year = 2020
ORDER BY title;


-- 4️⃣ Top 5 countries with the most content
SELECT 
    country_trim AS country,
    COUNT(*) AS total_content
FROM (
    SELECT 
        TRIM(UNNEST(STRING_TO_ARRAY(country, ','))) AS country_trim
    FROM netflix
    WHERE country IS NOT NULL
) AS t
WHERE country_trim <> ''
GROUP BY country_trim
ORDER BY total_content DESC
LIMIT 5;


-- 5️⃣ Identify the longest movie
SELECT *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC
LIMIT 1;


-- 6️⃣ Content added in the last 5 years
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';


-- 7️⃣ Content by Director ‘Rajiv Chilaka’
SELECT *
FROM (
    SELECT 
        n.*,
        TRIM(UNNEST(STRING_TO_ARRAY(director, ','))) AS director_name
    FROM netflix AS n
) AS t
WHERE director_name = 'Rajiv Chilaka';


-- 8️⃣ TV Shows with more than 5 seasons
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;


-- 9️⃣ Count content per Genre (split by commas)
SELECT 
    TRIM(UNNEST(STRING_TO_ARRAY(listed_in, ','))) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY genre
ORDER BY total_content DESC;


-- 🔟 Top 5 Years with Highest % of Content Released by India
SELECT 
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country LIKE '%India%')::numeric * 100,
        2
    ) AS percentage_of_indian_titles
FROM netflix
WHERE country LIKE '%India%'
GROUP BY release_year
ORDER BY percentage_of_indian_titles DESC
LIMIT 5;


-- 1️⃣1️⃣ All Documentaries
SELECT *
FROM netflix
WHERE listed_in ILIKE '%Documentaries%';


-- 1️⃣2️⃣ Content without a Director
SELECT *
FROM netflix
WHERE director IS NULL OR director = '';


-- 1️⃣3️⃣ Movies in Last 10 Years Featuring Salman Khan
SELECT *
FROM netflix
WHERE casts ILIKE '%Salman Khan%'
  AND release_year >= EXTRACT(YEAR FROM CURRENT_DATE) - 10
  AND type = 'Movie';


-- 1️⃣4️⃣ Top 10 Actors in Indian Movies on Netflix
SELECT 
    TRIM(UNNEST(STRING_TO_ARRAY(casts, ','))) AS actor,
    COUNT(*) AS total_titles
FROM netflix
WHERE country LIKE '%India%'
  AND type = 'Movie'
GROUP BY actor
ORDER BY total_titles DESC
LIMIT 10;


-- 1️⃣5️⃣ Categorize Content as Good / Bad based on Violent Keywords
SELECT 
    category,
    type,
    COUNT(*) AS content_count
FROM (
    SELECT *,
        CASE 
            WHEN description ILIKE '%kill%' 
              OR description ILIKE '%violence%' 
            THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category, type
ORDER BY type, category;

-- ===========================
-- END OF FILE 📌
-- ===========================
