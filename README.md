# 🎬 Apple Store Analytics Case Study

## 📌 1.0 Project Context

### Business Task

We (Leadership team) have been asked to support an aspiring app developer who needs data-driven insights to decide what type of app to build. They are seeking answers to questions such as:  
1. What app categories are most popular?  
2. What price should I set?  
3. How can I maximize user ratings?

## 📌 2.0 Data Combining

I will be uploading the data and running the SQL Queries on BigQuery.  The Data is split into 5 CSV files.  UNION ALL will be used to seamlessly combine our four separate tables into a single one:

````sql
CREATE TABLE `applestore_data.appleStore_description_combined` AS
SELECT * FROM `applestore_data.table_1`
UNION ALL
SELECT * FROM `applestore_data.table_2`
UNION ALL
SELECT * FROM `applestore_data.table_3`
UNION ALL
SELECT * FROM `applestore_data.table_4`;
````
*** 

## 📌 3.0 Exploratory Data Analysis

Before we dive into problem-solving, let's explore the data!

#### 1.
> Check the number of unique apps in both tables.  A disceprency could mean missing data in either of the two tabels, so this ensures we are dealing with the same set of applications in both data sets.


````sql
SELECT COUNT(DISTINCT id) AS Unique_App_IDs
FROM `applestore-data-project.applestore_data.applestore`;
````
![Screenshot 2023-08-09 at 10 28 37 AM](https://github.com/discobuggie/AppleStore-Data-Exploration/assets/108239917/e204d9a7-c343-4ac0-8874-371d1ae0713d)

````sql
SELECT COUNT(DISTINCT id) AS Unique_App_IDs
FROM `applestore-data-project.applestore_data.appleStore_description_combined`
````
![Screenshot 2023-08-09 at 10 28 37 AM](https://github.com/discobuggie/AppleStore-Data-Exploration/assets/108239917/46fbda92-1853-40cc-8fa8-6ea7bd49cd72)

As we can see, there are no missing data between the two tables!

#### 2.
> Check for any missing values in key fields.

````sql
SELECT COUNT (*) AS Missing_Values
FROM `applestore-data-project.applestore_data.applestore`
WHERE track_name IS NULL OR user_rating IS NULL OR prime_genre IS NULL;
````
![Screenshot 2023-08-09 at 3 05 14 PM](https://github.com/discobuggie/AppleStore-Data-Exploration/assets/108239917/ed20b02a-f52e-4521-a95f-229fa8d90fc3)

````sql
SELECT COUNT (*) AS Missing_Values
FROM `applestore-data-project.applestore_data.appleStore_description_combined`
WHERE app_desc IS NULL
````
![Screenshot 2023-08-09 at 3 05 14 PM](https://github.com/discobuggie/AppleStore-Data-Exploration/assets/108239917/5cb564cf-bce2-4b0e-b418-e698c14369b3)

No missing values! All tables are clean and there are no **data quality issues**.

#### 3. 
>  Find out the number of apps per genre.  This will give us an overview of the types distribution in the Apple Store, helping us identify **dominant genres**.

````sql
SELECT prime_genre, 
  COUNT (*) AS Num_apps
FROM `applestore-data-project.applestore_data.applestore`
GROUP BY prime_genre
ORDER BY Num_apps DESC
````
![Screenshot 2023-08-09 at 3 14 01 PM](https://github.com/discobuggie/AppleStore-Data-Exploration/assets/108239917/3af225ca-4608-4241-a737-5a795ca440cc)

Out of 23 rows, Games and Entertainment genres are clearly leading with a huge number of apps. 

#### 4.
> Get an overview of apps' ratings.

````sql
SELECT 
  min(user_rating) AS Min_Rating,
  max(user_rating) As Max_Rating,
  avg(user_rating) AS Avg_Rating
FROM `applestore-data-project.applestore_data.applestore`
````
![Screenshot 2023-08-09 at 3 16 53 PM](https://github.com/discobuggie/AppleStore-Data-Exploration/assets/108239917/235842e7-f7d0-4416-b164-359c4da86f56)

***

## 📌 4.0  Analysis

Here are some interesting insights for our stakeholder!

#### 1. 
> First I want to determine whether paid apps have higher ratings than free apps.

````sql
SELECT CASE
  WHEN price > 0 THEN "Paid" ELSE "Free"
  END AS App_Type,
  avg(user_rating) AS Avg_Rating
FROM `applestore-data-project.applestore_data.applestore`
GROUP BY App_Type;
````

![Screenshot 2023-08-09 at 3 21 50 PM](https://github.com/discobuggie/AppleStore-Data-Exploration/assets/108239917/aaec515c-93f1-4b05-ae6d-9ce7a8da4177)

The rating of paid apps is *slightly* higher compared to free apps.

#### 2.

> Check if apps with more supported languages have higher ratings.

````sql
SELECT CASE 
  WHEN lang_num < 10 THEN "<10 Languages"
  WHEN lang_num BETWEEN 10 AND 30 THEN "10-30 Languages"
  ELSE ">30 Languages"
  END AS language_bucket,
  avg(user_rating) AS Avg_Rating
FROM `applestore-data-project.applestore_data.applestore`
GROUP BY language_bucket
ORDER BY Avg_Rating DESC;
````
![Screenshot 2023-08-09 at 3 24 02 PM](https://github.com/discobuggie/AppleStore-Data-Exploration/assets/108239917/07c6427e-7ef9-4819-a6c4-bcea4fed5bd0)

The "10-30 Langauges" language bucket has higher average user ratings, meaning we don't necessarily need to work on integrating so many languages.  Instead, effort can be allocated towards other aspects of the app. 

#### 3.

> Check Genres with low ratings to analyze market gaps.

````sql
SELECT prime_genre,
  avg(user_rating) AS Avg_Rating
FROM `applestore-data-project.applestore_data.applestore`
GROUP BY prime_genre
ORDER BY Avg_Rating ASC
LIMIT 10;
````
![Screenshot 2023-08-09 at 3 29 09 PM](https://github.com/discobuggie/AppleStore-Data-Exploration/assets/108239917/a21a77ac-f1c0-4751-b6d8-a0effeff3902)

The first few categories recieved very low user ratings meaning that the users ar enot satisfied.  Therefore, this could be a good opportunity to create an app in this space. 

#### 4.

> Check if there is a correlation between the length of the app description and the user rating.  We will perform a JOIN on  `applestore-data-project.applestore_data.appleStore_description_combined`

````sql
SELECT CASE
   WHEN length(b.app_desc) <500 THEN "Short"
    WHEN length(b.app_desc) BETWEEN 500 AND 1000 THEN "Medium"
    ELSE "Long"
  END AS description_length_bucket,
  avg(user_rating) AS Avg_Rating
FROM
  `applestore-data-project.applestore_data.applestore` AS A
JOIN
  `applestore-data-project.applestore_data.appleStore_description_combined` AS B
ON 
    a.id = b.id
GROUP BY description_length_bucket
ORDER BY Avg_Rating DESC;
````
![Screenshot 2023-08-09 at 3 33 02 PM](https://github.com/discobuggie/AppleStore-Data-Exploration/assets/108239917/e48fc7fb-6d72-4cc4-98cc-616fae3d1f11)

The *longer* the description, the *better* the user rating on average! 

#### 5.

> Check the top-rated apps for each genre. We will perform a Window Function. It retrieves specific information about apps, including their prime genre, track name, and user rating, while applying ranking based on user rating and total rating count within each prime genre. Using PARTITION BY prime_genre, a distinct window is established for each unique genre, and within these windows, rows are ranked in descending order by user_rating, with tiebreaks resolved by rating_count_tot. As a result, the inner subquery computes the ranking of each app within its corresponding prime genre, primarily considering descending user rating and subsequently descending total rating count. The RANK() window function with the PARTITION BY clause ensures that the ranking is performed within each prime genre category. The outer query then filters the results from the subquery, selecting only the records where the calculated rank is 1. This effectively fetches the top-rated app within each prime genre.

The resulting dataset provides valuable insights into the highest-ranked app within each genre, allowing for targeted analysis of user ratings and app popularity across different genres.
````sql
SELECT
  prime_genre,
  track_name,
  user_rating
FROM ( 
  SELECT
  prime_genre,
  track_name,
  user_rating,
  RANK() OVER (PARTITION BY prime_genre ORDER BY user_rating DESC, rating_count_tot DESC) AS rank
  FROM 
  `applestore-data-project.applestore_data.applestore`
) AS a
WHERE
a.rank = 1;
````
![Screenshot 2023-08-09 at 3 39 45 PM](https://github.com/discobuggie/AppleStore-Data-Exploration/assets/108239917/abce55ef-36e6-4558-9f62-4f824809b407)

This table provides valuable insights for our stakeholders to identify top-performing apps. Consequently, it guides them toward potential candidates to emulate and prioritize.

***

## 📌 5.0  Act

Lets summarize the final reccomendations for our client:

1. Paid vs. Free apps
- Our Data analysis has shown that paid apps generally achieves slightly higher ratings than free apps.
- Users who pay for an app may have higher engagement and percieved more value, leading to the better ratings.
- If users percieve the quality of the app as good, it may be worth it to consider charging a certain amount for the app.

2. Language Support
- Apps supporting between 10 and 30 languages have better ratings.
- The emphasis lies not on the quantity of supported languages but rather on selecting the appropriate languages that align with the app's purpose.

3. Finance and Book apps have low ratings
- This suggests that user needs are not being fully met.
- Presents a realm of opportunity, as crafting a high-quality app within these categories to effectively address user needs could lead to substantial user ratings and significant market penetration.

4. Apps with longer descriptions have btter ratings
- Users likely appreciate having a clear understanding of the app's features and capabilities before they download.
- A detailed and well-crafted app description can set the expectation and eventually increase the satisfaction of users.

5. A new app should aim for an average rating above 3.5
- In order to stand out from the crowd, we should aim for a rating higher than the average 3.5.

6. Game and Entertainment genre have high competition.
- These categories have a very high volume of apps, suggesting the market may be saturated.
- Entering these spaces might be challenging due to high competition.
- However, it also suggests a high user demand in these spaces.
