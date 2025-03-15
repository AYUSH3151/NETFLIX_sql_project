# Netflix Movies and TV Shows Data Analysis using SQL

![](https://github.com/najirh/netflix_sql_project/blob/main/logo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

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

## Business Problems and Solutions

### 1. Count the number of Movies vs TV Shows--
``` 
SELECT 
     TYPE, 
	 COUNT(*) AS TOTAL_COUNT
FROM NETFLIX_DATASET
GROUP BY TYPE;
```
### 2. Find the most common rating for movies and TV shows--
```
SELECT * FROM NETFLIX_DATASET

SELECT TYPE, RATING
FROM (SELECT 
      RATING,
      TYPE, 
	  COUNT (*),
	  RANK() OVER (PARTITION BY TYPE 
				   ORDER BY COUNT(*) DESC )
FROM NETFLIX_DATASET
GROUP BY RATING, TYPE) AS RANK_1
WHERE RANK =1
```
### 3. List all movies released in a specific year (e.g., 2020)---
```
SELECT * FROM NETFLIX_DATASET

SELECT *
FROM NETFLIX_DATASET
WHERE TYPE='Movie'
AND RELEASE_YEAR = 2019
```
### 4. Find the top 5 countries with the most content on Netflix--
```
SELECT * FROM NETFLIX_DATASET

SELECT COUNT(*),UNNEST(STRING_TO_ARRAY(COUNTRY, ','))
FROM NETFLIX_DATASET
GROUP BY 2
ORDER BY COUNT(*) DESC
LIMIT 5
```
### 5. Identify the longest movie or TV show duration--
```
SELECT * FROM NETFLIX_DATASET

SELECT * 
FROM  (SELECT TYPE, DURATION, TITLE
       FROM NETFLIX_DATASET
       WHERE TYPE = 'Movie'
       AND DURATION IS NOT NULL
       ORDER BY  2 DESC
       LIMIT 1) A
 UNION ALL
SELECT *
FROM (SELECT TYPE, DURATION, TITLE
      FROM NETFLIX_DATASET
      WHERE TYPE = 'TV Show'
      AND DURATION IS NOT NULL
      ORDER BY  2 DESC
      LIMIT 1) B
```
### 6. Find content added in the last 5 years--
```
SELECT * FROM NETFLIX_DATASET

SELECT DATE_ADDED, TYPE, DURATION
FROM NETFLIX_DATASET
WHERE 
     TO_DATE(DATE_ADDED, 'MONTH DD, YYYY') >= CURRENT_DATE - INTERVAL '5 YEARS'

--------- OR-----------	 

SELECT DATE_ADDED, TYPE, DURATION
FROM NETFLIX_DATASET
WHERE CAST(DATE_ADDED AS DATE) >= CURRENT_DATE - INTERVAL '5 Years'
```	 
### 7. Find all the movies/TV shows by director 'Rajiv Chilaka'!--
```
SELECT * FROM NETFLIX_DATASET

SELECT 
	  TYPE, 
	  DIRECTOR,
	  TITLE
FROM NETFLIX_DATASET
WHERE DIRECTOR ILIKE '%Rajiv Chilaka%'
```
### 8. List all TV shows with more than 5 seasons--
```
SELECT * FROM NETFLIX_DATASET

SELECT 
      TYPE,
	  DURATION
FROM NETFLIX_DATASET
WHERE 
      TYPE = 'TV Show'
      AND CAST(SPLIT_PART(DURATION, ' ', 1)AS INTEGER) > 5
```
### 9. Count the number of content items in each genre--
```
SELECT * FROM NETFLIX_DATASET

SELECT NEW_LISTED_IN, COUNT(NEW_LISTED_IN) TOTAL_CONTENT
FROM (SELECT 
      SHOW_ID,
	  UNNEST (STRING_TO_ARRAY(LISTED_IN, ',')) NEW_LISTED_IN
FROM NETFLIX_DATASET) A
GROUP BY 1
-------- OR --------
SELECT 
	  UNNEST (STRING_TO_ARRAY(LISTED_IN, ',')) NEW_LISTED_IN,
	  COUNT(SHOW_ID) A 
FROM NETFLIX_DATASET
GROUP BY 1
```
### 10.Find each year and the average numbers of content release in India on netflix. --
### return top 5 year with highest avg content release!--
```
SELECT * FROM NETFLIX_DATASET 

SELECT
      EXTRACT (YEAR FROM TO_DATE(DATE_ADDED, 'MONTH DD,YYYY'))AS YEAR, 
	  COUNT(*) AS TOTAL_CONTENT,
	  ROUND (COUNT(*):: NUMERIC/ (SELECT COUNT(*) FROM NETFLIX_DATASET WHERE COUNTRY = 'India') * 100::NUMERIC) AS AVG_CONTENT
FROM NETFLIX_DATASET
WHERE COUNTRY = 'India'
GROUP BY 1
```
### 11. List all movies that are documentaries--
```
SELECT * FROM NETFLIX_DATASET

SELECT TYPE, LISTED_IN, TITLE
FROM NETFLIX_DATASET
WHERE TYPE = 'Movie' AND LISTED_IN ILIKE '%Documentaries%'
```
### 12. Find all content without a director--
```
SELECT * FROM NETFLIX_DATASET

SELECT *
FROM NETFLIX_DATASET
WHERE DIRECTOR IS NULL
```
### 13. Find how many movies actor 'Salman Khan' appeared in last 5 years!--
```
SELECT * FROM NETFLIX_DATASET

SELECT *
FROM NETFLIX_DATASET
WHERE 
     TYPE = 'Movie'
	 AND CAST_MEMBER ILIKE '%Salman Khan%'
	 AND CAST (DATE_ADDED AS DATE) > CURRENT_DATE - INTERVAL '5 YEARS'

----------- OR -------------
     
SELECT *
FROM NETFLIX_DATASET
WHERE 
     TYPE = 'Movie'
	 AND CAST_MEMBER ILIKE '%Salman Khan%'
	 AND RELEASE_YEAR:: NUMERIC >= EXTRACT ( YEAR FROM CURRENT_DATE) - 10
``` 
### 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.--
```
SELECT * FROM NETFLIX_DATASET

SELECT 
      UNNEST(STRING_TO_ARRAY(CAST_MEMBER, ','))ACTORS,
	  COUNT(*) AS MOVIES_DONE
FROM NETFLIX_DATASET
WHERE COUNTRY ILIKE '%India%'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10
```
### 15. Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. 
### Label content containing these keywords as 'Bad' and all other 
### content as 'Good'. Count how many items fall into each category.--
```
SELECT * FROM NETFLIX_DATASET

WITH NEW_TABLE 
AS
(SELECT 
      TYPE,
      TITLE,
	  DESCRIPTION,
	  (CASE 
	      WHEN DESCRIPTION ILIKE '%kill%'
		   OR DESCRIPTION ILIKE '%VIOLENCE%' THEN 'BAAAAAAD'
		  ELSE 'GOOOOOOD' END) RATING
FROM NETFLIX_DATASET)
SELECT
      RATING,
	  COUNT(RATING) 
FROM NEW_TABLE
GROUP BY 1
```
**Objective:** Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.

## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.

### Stay Updated and Join the Community

- **LinkedIn**: [Connect with me professionally](https://www.linkedin.com/in/ayush-kumar-3365b322a/)
- **EMAIL**: (ayushofficial3151@gmail.com)
  
Thank you for your support, and I look forward to connecting with you!
