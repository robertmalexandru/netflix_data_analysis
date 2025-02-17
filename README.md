# Netflix Data Analysis

![Project Logo](logo.png)

## Table of Contents

- [Overview](#overview)
- [Objectives](#objectives)
- [Dataset](#dataset)
- [Queries](#queries)
- [Findings and Conclusion](#findings-and-conclusion)

## Overview

This project comprehensively analyzes Netflix's movie and TV show catalog using SQL. The primary aim is to extract valuable insights and address various business questions about the dataset.

## Objectives

- Analyze the distribution between movies and TV shows.
- Identify prevalent content ratings.
- Examine content based on release years, countries, and durations.
- Categorize content using specific criteria and keywords.

## Dataset

The dataset utilized for this analysis is sourced from Kaggle:

- **Dataset Link:** [Netflix Movies and TV Shows](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Queries

```sql

-- Create the table
DROP TABLE IF EXISTS netflix_titles;
CREATE TABLE netflix_titles
(
    show_id VARCHAR(10),
    type VARCHAR(10),
    title VARCHAR(150),
    director VARCHAR(250),
    actor VARCHAR(1000),
    country VARCHAR(150),
    date_added VARCHAR(50),
    release_year INT,
    rating VARCHAR(10),
    duration VARCHAR(15),
    listed_in VARCHAR(100),
    description VARCHAR(250)
);

-- Q1: Determine the distribution of content types on Netflix.
SELECT type, COUNT(*) 
FROM netflix_titles
GROUP BY type;

-- Q2: Identify the most frequently occurring rating for each type of content.
SELECT type, rating, counts
FROM (
    SELECT type, rating, COUNT(*) AS counts, 
           RANK() OVER (PARTITION BY type ORDER BY COUNT(*) DESC) AS ranking
    FROM netflix_titles
    GROUP BY type, rating
) AS t1
WHERE ranking = 1;

-- Q3: Retrieve all movies released in a specific year.
SELECT * FROM netflix_titles 
WHERE type = 'Movie' AND release_year = 2020;

-- Q4: Identify the top 5 countries with the highest number of content items.
SELECT TRIM(UNNEST(STRING_TO_ARRAY(country, ','))) AS country, COUNT(*) AS titles
FROM netflix_titles
GROUP BY country
ORDER BY titles DESC
LIMIT 5;

-- Q5: Find the movie with the longest duration.
SELECT title, 
       MAX(CAST(SPLIT_PART(duration, ' ', 1) AS INT)) AS max_len
FROM netflix_titles
WHERE type = 'Movie' AND duration ~ '^[0-9]+ min$'
GROUP BY title
ORDER BY max_len DESC
LIMIT 1;

-- Q6: Retrieve content added to Netflix in the last 5 years.
SELECT * 
FROM netflix_titles
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';

-- Q7: List all content directed by Christopher Nolan.
SELECT * FROM netflix_titles WHERE director ILIKE '%Christopher Nolan%';

-- Q8: Identify TV shows with more than 5 seasons.
SELECT title, SPLIT_PART(duration, ' ', 1)::INT AS seasons
FROM netflix_titles
WHERE type = 'TV Show' AND duration ~ '^[0-9]+ Season' AND SPLIT_PART(duration, ' ', 1)::INT > 5;

-- Q9: Count the number of content items in each genre.
SELECT TRIM(UNNEST(STRING_TO_ARRAY(listed_in, ','))) AS genre, COUNT(*)
FROM netflix_titles
GROUP BY genre
ORDER BY COUNT(*) DESC;

-- Q10: Calculate and rank years by the average number of content releases in the United States. Return the top 5 years with the highest average.
SELECT year, COUNT(*) AS num_releases
FROM (
    SELECT EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) AS year
    FROM netflix_titles
    WHERE country ILIKE '%United States%'
) AS subquery
GROUP BY year
ORDER BY num_releases DESC
LIMIT 5;

-- Q11: Retrieve all movies classified as documentaries.
SELECT * FROM netflix_titles WHERE listed_in ILIKE '%Documentaries%';

-- Q12: List content that does not have a director.
SELECT * FROM netflix_titles WHERE director IS NULL;

-- Q13: Count the number of movies featuring Leonardo DiCaprio in the last 10 years.
SELECT * FROM netflix_titles
WHERE actor ILIKE '%Leonardo DiCaprio%' AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;

-- Q14: Identify the top 10 actors with the most appearances in United States movies.
SELECT TRIM(UNNEST(STRING_TO_ARRAY(actor, ','))) AS actor, COUNT(*)
FROM netflix_titles
WHERE country ILIKE '%United States%'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;

-- Q15: Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.
SELECT 
    CASE 
        WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad Content'
        ELSE 'Good Content'
    END AS category,
    COUNT(*) AS total_content
FROM netflix_titles
GROUP BY category;
```

## Findings and Conclusion

- **Content Distribution:** The dataset contains various movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by the United States highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.
