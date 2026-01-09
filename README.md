# Netflix and TV Shows Data Analysis using SQL  

![Netflix logo](https://raw.githubusercontent.com/rathoredivyansh2022-web/Netflix_sql_project-/main/logo.png)

## Overview
This project analyzes Netflix movies and TV shows data using SQL to extract insights and solve real-world business problems.

## Objectives
- Analyze Movies vs TV Shows distribution  
- Identify common ratings  
- Analyze content by year, country, duration  
- Categorize content using keywords  

## Dataset
Source: Kaggle – Netflix Movies and TV Shows Dataset 
-- 1. Drop & Create Table
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix (
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

-- 2. Count Movies vs TV Shows
SELECT 
    type,
    COUNT(*) AS total_count
FROM netflix
GROUP BY type;

-- 3. Most Common Rating for Movies & TV Shows
WITH rating_counts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
ranked_ratings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rnk
    FROM rating_counts
)
SELECT 
    type,
    rating AS most_common_rating
FROM ranked_ratings
WHERE rnk = 1;

-- 4. Movies Released in 2020
SELECT *
FROM netflix
WHERE type = 'Movie'
  AND release_year = 2020;

-- 5. Top 5 Countries with Most Content
SELECT 
    country,
    COUNT(*) AS total_content
FROM (
    SELECT UNNEST(STRING_TO_ARRAY(country, ',')) AS country
    FROM netflix
) t
WHERE country IS NOT NULL
GROUP BY country
ORDER BY total_content DESC
LIMIT 5;

-- 6. Longest Movie
SELECT *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC
LIMIT 1;

-- 7. Content Added in Last 5 Years
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';

-- 8. Content Directed by Rajiv Chilaka
SELECT *
FROM netflix
WHERE director ILIKE '%Rajiv Chilaka%';

-- 9. TV Shows with More Than 5 Seasons
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;

-- 10. Content Count by Genre
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY genre;

-- 11. Top 5 Years with Highest Indian Content Release
SELECT 
    release_year,
    COUNT(*) AS total_release
FROM netflix
WHERE country = 'India'
GROUP BY release_year
ORDER BY total_release DESC
LIMIT 5;

-- 12. Documentary Movies
SELECT *
FROM netflix
WHERE listed_in ILIKE '%Documentaries%';

-- 13. Content Without Director
SELECT *
FROM netflix
WHERE director IS NULL;

-- 14. Salman Khan Movies in Last 10 Years
SELECT *
FROM netflix
WHERE casts ILIKE '%Salman Khan%'
  AND release_year >= EXTRACT(YEAR FROM CURRENT_DATE) - 10;

-- 15. Top 10 Actors in Indian Movies
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*) AS total_movies
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY total_movies DESC
LIMIT 10;

-- 16. Categorize Content Based on Violence
SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' 
              OR description ILIKE '%violence%' 
            THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) t
GROUP BY category;

## Findings & Conclusion

Netflix has a balanced mix of movies and TV shows.

Certain ratings dominate specific content types.

India is a major contributor to Netflix content.

Keyword-based categorization helps identify content nature.

This project demonstrates strong SQL querying, data analysis, and business insight generation.

